# Установка Proton VPN на Linux Mint
Преимущества **Proton VPN** это высокая скорость и бесплатный базовый план. Сервис разработан в Швейцарии при поддержке Европейского Союза. 

Данная инструкция предписывает установку Proton VPN в режиме "всегда включен" и предотвращает утечку любых пакетов

### Установка
```sh
# Установка репозитория
wget https://protonvpn.com/download/protonvpn-stable-release_1.0.1-1_all.deb &&
sudo dpkg -i protonvpn-stable-release_1.0.1-1_all.deb &&
rm protonvpn-stable-release_1.0.1-1_all.deb &&
# Установка пакета
sudo apt-get update &&
#sudo apt install -y protonvpn gnome-shell-extension-appindicator gir1.2-appindicator3-0.1 &&
sudo apt-get install -y protonvpn-cli

```
##### Первый запуск
```sh
# Логин
clear &&
echo -e '\033[3B\033[1;37m1. Бесплатная регистрация: \033[36mhttps://protonvpn.com/free-vpn/linux\033[0m\n' &&
read -p '2. Enter your proton username here> ' uname &&
protonvpn-cli login "$uname" &&
# Установа соединения
protonvpn-cli ks --on &&
protonvpn-cli c -f

```

##### Автоматизация запуска скорости
```sh
srv=`protonvpn-cli status|grep Server:|cut -d: -f2-` &&
(cat >~/.config/protonvpn/autostart.sh<<END
#!/bin/bash
protonvpn-cli ks --off
protonvpn-cli ks --permanent
protonvpn-cli c ${srv:=-f}
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
echo -e '\n\x1b[1m=SUCCESS=\n'

```
> Настраивается с текущим или самым быстрым сервером, если подключение отсутствует.
> 
> Отключение: Пуск ➜ Параметры ➜ Автозагрузка ➜ **Proton VPN CLI**.

### Проверка скорости
```sh
sudo apt install -y speedtest-cli &&
speedtest-cli

```
> Если появляется `...ERROR: HTTP Error 403: Forbidden`, повторите `speedtest-cli` через две минуты

##### Переподключение к более быстрому серверу (при необходимости)
```sh
protonvpn-cli d &&
protonvpn-cli c -f

```
> Если быстрый сервер найден, заново настройте автоматический запуск. См. также [использование](https://protonvpn.com/support/linux-vpn-tool/#cli) на сайте производителя.

##### Альтернативная проверка
```sh
wget -O /dev/null http://speedtest.wdc01.softlayer.com/downloads/test100.zip

```
