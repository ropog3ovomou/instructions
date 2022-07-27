# Mint setup for beginners
Follow instructions one by one, skipping the optional if you want. Copy-paste and run the code in your Shell (see [how to do that](https://www.youtube.com/watch?v=_cu7hWs-zrQ)). 
## Basic setup
#### Update all softwre to the recent versions
> **Warning**
> Takes ~ half an hour
```sh
sudo apt update && sudo apt upgrade -y
```

#### Add mousewheel support
Install
```sh
sudo apt install imwheel
```
Enable
```sh
cat >~/.imwheelrc<<EOF
".*"
None,      Up,   Button4, 3
None,      Down, Button5, 3
Control_L, Up,   Control_L|Button4
Control_L, Down, Control_L|Button5
Shift_L,   Up,   Shift_L|Button4
Shift_L,   Down, Shift_L|Button5
EOF
mkdir -p ~/.config/autostart/
cat >~/.config/autostart/imwheel.desktop<<EOF
[Desktop Entry]
Version=1.0
Name=Imwheel
Comment=Enable mouse scroll
Icon=mouse
Exec=imwheel
Terminal=false
Type=Application
EOF
chmod a+x ~/.config/autostart/imwheel.desktop
imwheel

```
#### Remove password from the screensaver (optional)
menu -> preferences -> screensaver -> Screensaver settings -> Delay before starting the screensaver: **Never**

## Software (optional)

#### Google Chrome
```sh
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb &&\
sudo dpkg -i google-chrome-stable_current_amd64.deb &&\
rm google-chrome-stable_current_amd64.deb

```
> Launch it from Menu -> Internet -> Google Chrome

#### Qbit bittorent client
```sh
sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable &&\
sudo apt update &&\
sudo apt install qbittorrent -y

```
> Launch it from Menu -> Internet -> qBittorrent

#### lantern VPN
```sh
wget https://getlantern.org/lantern-installer-64-bit.deb &&\
sudo dpkg -i lantern-installer-64-bit.deb &&\
rm lantern-installer-64-bit.deb

```
> Launch it from Menu -> Other -> Lantern

#### RDP Client Remmina
```sh
sudo apt-add-repository ppa:remmina-ppa-team/remmina-next &&\
sudo apt update &&\
sudo apt install remmina remmina-plugin-rdp remmina-plugin-secret

```
#### Brave browser
```sh
sudo apt install apt-transport-https curl -y &&\
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg &&\
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg arch=amd64] https://brave-browser-apt-release.s3.brave.com/ stable main"|sudo tee /etc/apt/sources.list.d/brave-browser-release.list &&\
sudo apt update &&\
sudo apt install brave-browser -y

```


