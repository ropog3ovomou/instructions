# Настройка Linux Mint после установки
Выполняйте инструкции по очереди. Помеченные как необязательные можете пропускать. Строки необходимо запускать в терминале (см. [командная строка для новичков](https://www.youtube.com/watch?v=qwopGsaNF_Q)).
## Начальная настройка
#### Обновление версий ПО

> **Note**
> Эта операция занимает примерно полчаса
```sh
sudo apt update && sudo apt upgrade -y
```

#### Добавление поддержки колесика мыши
> **Note** 
> Без этого приложения слегка реагируют лишь на интенсивное прокручивание.
##### Установка
```sh
sudo apt install imwheel
```
##### Активация
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
#### Снятие пароля с экранной заставки (необязательно)
меню -> Параметры -> Экранная заставка -> Настройки заставки -> Задержка [уже] начатой заставки: Никогда

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

#### Proton VPN в режиме постоянной защиты
##### Установка
```sh
# Установка репозитория
wget https://protonvpn.com/download/protonvpn-stable-release_1.0.1-1_all.deb &&
sudo dpkg -i protonvpn-stable-release_1.0.1-1_all.deb &&
rm protonvpn-stable-release_1.0.1-1_all.deb &&
№ Установка программы
sudo apt-get update &&
#sudo apt install -y protonvpn gnome-shell-extension-appindicator gir1.2-appindicator3-0.1
sudo apt-get install -y protonvpn-cli &&
# Логин
clear &&
echo -e '\033[3B\033[1;37m1. Бесплатная регистрация: \033[36mhttps://protonvpn.com/free-vpn/linux\033[0m\n' &&
read -p '2. Enter your proton username here> ' uname &&
protonvpn-cli login "$uname" &&
№ Установа соединения
protonvpn-cli killswitch --on &&
protonvpn-cli connect --fastest

```

##### Настройка автозапуска
```sh
srv=`protonvpn-cli status|grep Server:|cut -d: -f2-`
(cat >~/.config/protonvpn/autostart.sh<<END
#!/bin/bash
protonvpn-cli ks --off
protonvpn-cli ks --on
protonvpn-cli c $srv
END
) &&
(cat >~/.config/autostart/protonvpn-cli.desktop<<END
[Desktop Entry]
Type=Application
Name=ProtonVPN CLI
Icon=lock
Exec=$HOME/.config/protonvpn/autostart.sh
Terminal=false
END
) &&
chmod a+x ~/.config/autostart/protonvpn-cli.desktop ~/.config/protonvpn/autostart.sh &&
echo = Success =

```
> Отключение автозапуска: Меню ➜ Параметры ➜ Автозагрузка ➜ **Proton VPN CLI**.
> См. также [использование](https://protonvpn.com/support/linux-vpn-tool/#cli) вручную.


#### lantern VPN
```sh
# Установка
wget https://getlantern.org/lantern-installer-64-bit.deb &&\
sudo dpkg -i lantern-installer-64-bit.deb &&\
rm lantern-installer-64-bit.deb &&
# Запускать автоматически
cat >~/.config/autostart/lantern.desktop<<END
[Desktop Entry]
Type=Application
Name=lantern
Exec=lantern
Icon=lantern
X-GNOME-Autostart-enabled=true
NoDisplay=false
Hidden=false
X-GNOME-Autostart-Delay=0
END

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

