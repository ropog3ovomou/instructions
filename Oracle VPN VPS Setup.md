# Personal Mint / Ubuntu double-VPN via Oracle cloud
You will create a free instance in the Oracle cloud and use it as the VPN server for your Linux machine.
You will configure Wireguard as the main protocol, and SSH tunnel on-demand. 
Both will be switchable via standard network connections icon on your desktop.

## Prepare your SSH keys
> **Note**
> If you are not familiar with Linux, see [Mint Setup for beginners](https://github.com/ropog3ovomou/instructions/blob/main/Mint%20Setup.md)

1. If you havent' used SSH on your machine earlier, create your main SSH key pair:
    ````sh
    ssh-keygen
    ````
2. Create an additional key pair for ssh tunnel:
   ````sh
   ssh-keygen -f .ssh/id_rsa_vpn -P ''
   ````

## Create Oracle cloud VPS instance
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
3. Click Create and wait until instance turns green
5. Instance access -> Public IP address -> **Copy**
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
 Contratulations, now your server is up and running. 
    
## Setup SSH tunnel-based VPN

 
## Sources:

> Adapted from [setting up personal wireguard vpn on oracle cloud compute instance](https://pswalia2u.medium.com/setting-up-personal-wireguard-vpn-on-oracle-cloud-compute-instance-1d90d56d4b8b)
