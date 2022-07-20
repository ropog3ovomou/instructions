# Настройка Mint для новичков и не только
Выполняйте инструкции одру за другой. Помеченные как необязательные можете пропускать. Строки необходимо скопировать и запускать в терминале (см. [командная строка для новичков](https://www.youtube.com/watch?v=qwopGsaNF_Q)).
## Начальная настройка
#### Обновить имеющиеся программы

> **Warning**
> Операция занимает примерно полчаса
```sh
sudo apt update && sudo apt upgrade -y
```

#### Добавление поддержки колесика мыши
Установка
```sh
sudo apt install imwheel
```
Настройка и запуск
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
## Программы (необязательно)

#### Google Chrome
```sh
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb &&\
sudo dpkg -i google-chrome-stable_current_amd64.deb &&\
rm google-chrome-stable_current_amd64.deb

```
> Запускается из меню -> Интернет

#### Qbit (клиент bittorent)
```sh
sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable &&\
sudo apt update &&\
sudo apt install qbittorrent -y

```
> Запускается из меню -> Интернет

#### lantern VPN
```sh
wget https://getlantern.org/lantern-installer-64-bit.deb &&\
sudo dpkg -i lantern-installer-64-bit.deb &&\
rm lantern-installer-64-bit.deb

```
> Запускается из меню -> Прочие

#### RDP Client Remmina
```sh
sudo apt-add-repository ppa:remmina-ppa-team/remmina-next &&\
sudo apt update &&\
sudo apt install remmina remmina-plugin-rdp remmina-plugin-secret

```
> Запускается из меню -> Интернет

#### Brave browser
```sh
sudo apt install apt-transport-https curl -y &&\
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg &&\
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg arch=amd64] https://brave-browser-apt-release.s3.brave.com/ stable main"|sudo tee /etc/apt/sources.list.d/brave-browser-release.list &&\
sudo apt update &&\
sudo apt install brave-browser -y

```
> Запускается из меню -> Интернет

