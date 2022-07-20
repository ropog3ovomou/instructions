# Mint setup for beginners
Follow instructions one by one, skipping optional if you want. Copy-paste the code in your Shell (see [noobs lab](https://www.youtube.com/watch?v=ppP5i7f47ao)).
## Initial setup
#### Update all softwre to the recent versions

```sh
sudo apt update && sudo apt upgrade -y
```
> **Warning**
> Takes around half an hour

#### Enable mousewheel
```sh
sudo apt install imwheel
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
## Install Software (optional)

#### lantern VPN
```sh
wget https://getlantern.org/lantern-installer-64-bit.deb
sudo dpkg -i lantern-installer-64-bit.deb
```


#### Google Chrome
```sh
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb

```


#### RDP Client Remmina
```sh
sudo apt-add-repository ppa:remmina-ppa-team/remmina-next
sudo apt install remmina remmina-plugin-rdp remmina-plugin-secret

```


#### Brave browser
```sh
sudo apt install apt-transport-https curl
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg arch=amd64] https://brave-browser-apt-release.s3.brave.com/ stable main"|sudo tee /etc/apt/sources.list.d/brave-browser-release.list
sudo apt update
sudo apt install brave-browser -y

```

# Qbit bittorent client
```sh
udo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable
sudo apt update
apt install qbittorrent -y

```

