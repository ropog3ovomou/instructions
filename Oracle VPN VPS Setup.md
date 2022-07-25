# Personal VPN with free Oracle cloud, Linux Mint
You will configure two VPN tunnels:
- **Wireguard** as the main protocol
- fast **SSH tunnelling** as an alternative, on-demand VPN.

Both tunnels will be switchable via the standard visual tools on your desktop.

## Prepare SSH keys
> **Note**
> If you are not familiar with Linux, see [Mint Setup for beginners](https://github.com/ropog3ovomou/instructions/blob/main/Mint%20Setup.md)

1. If you haven't used SSH on your machine before, create your main SSH key pair:
    ````sh
    ssh-keygen
    ````
2. Create an additional key pair for the ssh tunnel:
   ````sh
   ssh-keygen -f .ssh/id_rsa_vpn -P ''
   ````

## Create cloud instance
1. [Login](https://cloud.oracle.com/) to your oracle cloud account
2. Click Get Started -> Launch Resources -> [Create a VM instance](https://cloud.oracle.com/compute/instances/create)
    - Name: **"vpn"**
    - Placement
        - Availability Domain: **AD-1** (Frankfurt)
    - Image and shape 
        - Image: **Canonical Ubuntu**
        - Shape: **VM.Standard.E2.1.Micro**
    - Add SSH keys -> **(x) Paste public keys**
        - SSH keys: **paste output of:**
        ```sh
        cat ~/.ssh/id_rsa.pub
        ```
    - Click Create
3. Wait until instance turns green
5. Public IP address -> **Copy**
1. Run the following in your terminal, replacing `<IP>` with the copied address: 
    > Your string must look like `sudo sed -i '$a111.11.1.111 vpn' /etc/hosts`
    ````sh
    sudo sed -i '$a<IP> vpn' /etc/hosts
    ````
2. Connect to the vpn server with SSH:
    > **Note**
    > If you are new to SSH, please see a [quick intro](https://www.youtube.com/watch?v=qWKK_PNHnnA).
    ````sh
	  ssh ubuntu@vpn
    
    ````
👍 Contratulations, now you have a machine in the cloud. 
    
## Setup SSH-based VPN
> **Note** 
> All commands you run in the client terminal.
1. Run following commands on the client:
    ````sh
    # Enable SSH tunneling on the server
    ssh ubuntu@vpn "sudo sed -i 's/^.*PermitTunnel no/PermitTunnel yes/' /etc/ssh/sshd_config" &&
    # Enable kernel packer forwarding:
    ssh ubuntu@vpn "sudo su -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'" &&
    # Allow special key to connect as root (required for kernel tunnelling)
    cat ~/.ssh/id_rsa_vpn.pub |ssh ubuntu@vpn 'sudo su -c "cat >> /root/.ssh/authorized_keys"'
    # Install SSH tunnelling plugin
    sudo apt-get install network-manager-ssh &&
    # Add network manager VPN connection
    nmcli con add connection.type vpn vpn.service-type org.freedesktop.NetworkManager.ssh vpn.data \
    'auth-type = key, key-file = ~/.ssh/id_rsa_vpn, local-ip = 172.16.40.2, 
    netmask = 255.255.255.252, remote = vpn, remote-ip = 172.16.40.1' \
    ipv4.dns 1.1.1.1,8.8.8.8 connection.autoconnect no connection.id SSH
    
    ````
#### Switch on the connection
1. Click **SSH** under network connections tray icon (see the image below)

![image](https://user-images.githubusercontent.com/107844943/180663363-50b7c58f-c2c3-4916-8938-d707bb02769f.png)

## Setup WireGuard VPN
1. Go to the server:
    ````sh
    ssh ubuntu@vpn
### On the server
Perform following operations from ssh
1. Setup WireGuard VPN:
    ````sh
    # Install virtualenv:
    sudo apt install -y --no-install-recommends python3-virtualenv
    # Download algo and activate virtual environment
    git clone https://github.com/trailofbits/algo.git &&
    cd algo &&
    python3 -m virtualenv --python="$(command -v python3)" .env &&
    source .env/bin/activate &&
    python3 -m pip install -U pip virtualenv &&
    python3 -m pip install -r requirements.txt
    # Disable IPSec
    sed -i 's/ipsec_enabled: true/ipsec_enabled: false/' config.cfg
    # Start the installation
    ansible-playbook main.yml -e "provider=local role=local \
    sever=localhost ondemand_cellular=false ondemand_wifi=false \
    dns_adblocking=true ssh_tunneling=true store_pki=true endpoint=`curl ifconfig.me`"
1. Clear IPTables and reboot
    ````sh
    sudo iptables -F && sudo reboot
    
    ````
### Allow wireguard through the cloud firewall:
We have to create new security list in the network and add it to the subnet
1. [Login](https://cloud.oracle.com/) to your oracle cloud account
1. Virtual Cloud Networks -> vcn-***** -> Security Lists -> Create Security List
    - Name: **vpn**
    - +Another ingres rule 
        - Source CIDR: "**0.0.0.0/0**"
        - IP Protocol: **UDP**
        - Destination Port Range: "**500,4500,51820**"
    - Save list
2. Go to Virtual Cloud Networks -> vcn-***** -> subnet-***** -> Add Security List -> Security List: **vpn**
### On the client
````sh
# Install required build tools
sudo apt install -y wireguard git dh-autoreconf libglib2.0-dev intltool build-essential \
     libgtk-3-dev libnma-dev libsecret-1-dev network-manager-dev resolvconf &&
# Download network manager plugin
git clone https://github.com/max-moser/network-manager-wireguard &&
cd network-manager-wireguard &&
# Build and install
./autogen.sh --without-libnm-glib &&
./configure --without-libnm-glib --prefix=/usr --sysconfdir=/etc --libdir=/usr/lib/x86_64-linux-gnu \
    --libexecdir=/usr/lib/NetworkManager --localstatedir=/var &&
make &&
sudo make install &&
(
# Read vpn config into variables
source <(ssh ubuntu@vpn 'cat algo/configs/localhost/wireguard/laptop.conf |
         sed -e "/\[/d" -e "/^$/d" -e "s/ //g"') &&
# Work around a bug that prevents wiregard from working without explicit mask
[[ "$Address" == *"/"* ]] || Address=$Address/32 &&
# Add network manager connection
nmcli con add connection.type vpn vpn.service-type org.freedesktop.NetworkManager.wireguard vpn.data \
"connection-dns = $DNS, local-ip4 = $Address, local-private-key = $PrivateKey, peer-allowed-ips = 0.0.0.0/0, 
peer-endpoint = $Endpoint, peer-preshared-key = $PresharedKey, peer-public-key = $PublicKey" \
connection.autoconnect no connection.id WG
)

````

## Test network speed
sudo apt install speedtest-cli
speedtest
> VPN speed should be significantly lower than your normal speed. 


 
## Sources:

> Adapted from [setting up personal wireguard vpn on oracle cloud compute instance](https://pswalia2u.medium.com/setting-up-personal-wireguard-vpn-on-oracle-cloud-compute-instance-1d90d56d4b8b)
