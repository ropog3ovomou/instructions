# Бесплатный VPN c Oracle и Linux Mint
Мы настроим два варианта VPN:
- **Wireguard** в качестве главного
- и **SSH** как запасной

Подключениями можно будет управлять визуально, через иконку сетевых соединений

# Генерация ssh-ключей
> **Note**
> Для знакомства с SSH смотрите 
> 1. [для чего нужен SSH](https://www.youtube.com/watch?v=Ah4ayHYnrwc) 
> 2. [командная строка для новичков](https://www.youtube.com/watch?v=qwopGsaNF_Q) 
> 3. [основы SSH и SCP в одном видео](https://youtu.be/Ns24-dCXDDg)


1. Создайте основную пару ключей:
    ````sh
    ssh-keygen
    ````
2. Создайте отдельную пару для VPN (понадобится позже):
   ````sh
   ssh-keygen -f .ssh/id_rsa_vpn -P ''
   ````

# Создание машины в облаке
1. [Войдите](https://cloud.oracle.com/) в учетную запись Oracle (зарегистрируйтесь при необходимости)
2. Нажмите Get Started -> Launch Resources -> [Create a VM instance](https://cloud.oracle.com/compute/instances/create)
    - Name: **"vpn"**
    - Placement
        - Availability Domain: **AD-1** (Frankfurt)
    - Image and shape 
        - Image: **Canonical Ubuntu**
        - Shape: **VM.Standard.E2.1.Micro**
    - Add SSH keys -> **(x)** Paste public keys
        - SSH keys: **запустите следующую команду и вставьте результат:**
        ```sh
        cat ~/.ssh/id_rsa.pub
        ```
    - Нажмите Create
3. Подождите, пока индикатор станет зеленым
5. Public IP address -> **Copy**
1. Замените `<IP>` в следующей команде скопированным адресом и выполните ее: 
    > Команда должна выглядеть примерно так - `sudo sed -i '$a111.11.1.111 vpn' /etc/hosts`
    ````sh
    sudo sed -i '$a<IP> vpn' /etc/hosts
    ````
2. Передайте пробную команду на сервер через SSH:
    ````sh
    ssh ubuntu@vpn uname -a
    
    ````
👍 Теперь у вас есть собственный сервер в облаке
    
# Настройка VPN на основе SSH
## На клиенте
Выполните в терминале следующие команды:
````sh
# Включить SSH туннель на сервере
ssh ubuntu@vpn "sudo sed -i 's/^.*PermitTunnel no/PermitTunnel yes/' /etc/ssh/sshd_config" &&
# Включить перенаправление пакетов в ядре:
ssh ubuntu@vpn "sudo su -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'" &&
# Добавить специальный ключ для пользователя root (необходимо для туннелирования)
cat ~/.ssh/id_rsa_vpn.pub |ssh ubuntu@vpn 'sudo su -c "cat >> /root/.ssh/authorized_keys"'
# Установить плагин туннелирования SSH
sudo apt-get install network-manager-ssh &&
# Добавить VPN соединение в сетевой менеджер
nmcli con add connection.type vpn vpn.service-type org.freedesktop.NetworkManager.ssh vpn.data \
'auth-type = key, key-file = ~/.ssh/id_rsa_vpn, local-ip = 172.16.40.2, 
netmask = 255.255.255.252, remote = vpn, remote-ip = 172.16.40.1' \
ipv4.dns 1.1.1.1,8.8.8.8 connection.autoconnect no connection.id SSH

````
👍 VPN на основе SSH готов. Как пользоваться см. [Проверка соединения](https://github.com/ropog3ovomou/instructions/new/main#%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%B0-%D1%81%D0%BE%D0%B5%D0%B4%D0%B8%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F).
# Настройка VPN WireGuard
## На сервере
1. Войдите на сервер:
    ````sh
    ssh ubuntu@vpn
1. Установите WireGuard VPN:
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
1. Обнулите iptables и перезагрузите сервер
    ````sh
    sudo iptables -F && sudo reboot
    
    ````
## На сайте Oracle
Настроим firewall. Для этого нужно создать список правил в сети (network) и подключить его к подсети (subnet).
1. [Войдите](https://cloud.oracle.com/) в учетную запись Oracle
1. Virtual Cloud Networks -> vcn-***** -> Security Lists -> Create Security List
    - Name: **vpn**
    - +Another ingres rule 
        - Source CIDR: "**0.0.0.0/0**"
        - IP Protocol: **UDP**
        - Destination Port Range: "**500,4500,51820**"
    - Save list
2. Перейдите в Virtual Cloud Networks -> vcn-***** -> subnet-***** -> Add Security List -> Security List: **vpn**
## На клиенте
````sh
# Установить инструменты сборки
sudo apt install -y wireguard git dh-autoreconf libglib2.0-dev intltool build-essential \
     libgtk-3-dev libnma-dev libsecret-1-dev network-manager-dev resolvconf &&
# Загрузить плагин сетевого менеджера
git clone https://github.com/max-moser/network-manager-wireguard &&
cd network-manager-wireguard &&
# Скомпилировать и установить
./autogen.sh --without-libnm-glib &&
./configure --without-libnm-glib --prefix=/usr --sysconfdir=/etc --libdir=/usr/lib/x86_64-linux-gnu \
    --libexecdir=/usr/lib/NetworkManager --localstatedir=/var &&
make &&
sudo make install &&
(
# Загрузить данные с сервера в переменные
source <(ssh ubuntu@vpn 'cat algo/configs/localhost/wireguard/laptop.conf |
         sed -e "/\[/d" -e "/^$/d" -e "s/ //g"') &&
# Поправить баг с неработающим соединением при неуказанной маске
[[ "$Address" == *"/"* ]] || Address=$Address/32 &&
# Добавить соединение в сетевой менеджер
nmcli con add connection.type vpn vpn.service-type org.freedesktop.NetworkManager.wireguard vpn.data \
"connection-dns = $DNS, local-ip4 = $Address, local-private-key = $PrivateKey, peer-allowed-ips = 0.0.0.0/0, 
peer-endpoint = $Endpoint, peer-preshared-key = $PresharedKey, peer-public-key = $PublicKey" \
connection.autoconnect yes connection.id WG
)

````
👍 Поздравляем, настройка VPN завершена.
# Проверка соединения
### Подключение
1. Проверьте внешний IP адрес:
    ````sh
    curl https://ipinfo.io/`curl -s ifconfig.me`
    ````
2. Нажмите **SSH** или **WG** в системном лотке
 
![image](https://user-images.githubusercontent.com/107844943/180772075-89822abc-8e9c-4e23-b334-4d29483b6f29.png)
    
3. Проверьте IP еще раз
    ````sh
    curl https://ipinfo.io/`curl -s ifconfig.me`
    ````
    
![image](https://user-images.githubusercontent.com/107844943/180773495-3fb5376c-06cf-4051-ab9f-5c68af5f5fc2.png)

### Скорость
````sh
sudo apt install -y speedtest-cli &&
speedtest-cli

````
# Как подключаться к WG VPN автоматически
При использовании VPN в целях безопасности важно избегать случаев, когда он отключен. Следующее заставит VPN подключаться автоматически
> **Note**
> При неудачном соединении с VPN сетевое соединение отключается. Это делает работу безопасной
1. Выбрать **Сетевые соединения** в системном лотке
2. Отрыть главное соединение
3. Основное -> **[x]** Автоматически подключаться к VPN -> **WG**

<img src=https://user-images.githubusercontent.com/107844943/180794674-f9b5473a-30fd-4fb4-bbfb-e7f19d26b76b.png width=200> <img src=https://user-images.githubusercontent.com/107844943/180785955-0c352592-f82c-4677-b29e-6bff2cfa5029.png height=300 > <img src=https://user-images.githubusercontent.com/107844943/180786161-4f5f354f-5aa9-4a0d-9729-1f78b8bfe2d9.png height=300 >

 
# Источники

> Adapted from 
> - [setting up personal wireguard vpn on oracle cloud compute instance](https://pswalia2u.medium.com/setting-up-personal-wireguard-vpn-on-oracle-cloud-compute-instance-1d90d56d4b8b)
> - [Network Manager VPN plugin for WireGuard](https://github.com/max-moser/network-manager-wireguard)
> - [How to configure WireGuard VPN client with NetworkManager GUI](https://www.xmodulo.com/wireguard-vpn-network-manager-gui.html)
> - [Algo VPN](https://github.com/trailofbits/algo/blob/master/README.md)
> - [Network Manager - SSH](https://github.com/danfruehauf/NetworkManager-ssh)
