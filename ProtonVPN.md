# Установка Proton VPN на Linux Mint
В результате выполнения данной инструкции VPN
всегда будет включен, что предотвратит утечку пакетов.
Преимущества **Proton VPN** это высокая скорость и бесплатный базовый план. 
Сервис разработан в Швейцарии при поддержке Европейского Союза. 

## Настройка
#### Установка
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
#### Первый запуск
При необходимости [зарегистрируйтесь](https://protonvpn.com/free-vpn/linux). 
> **Warning**
> Если Вы пользуетесь другими сервисами Протона -- не используйте для VPN учетные записи, о которых не должны знать или связывать с вами. Создайте отдельную учетку.

```sh
# Логин
clear &&
echo -e '\033[3B\033[1;37m1. Бесплатная регистрация: \033[36mhttps://protonvpn.com/free-vpn/linux\033[0m\n' &&
read -p '2. Enter your proton username here> ' uname &&
protonvpn-cli logout || true &&
protonvpn-cli login "$uname" &&
# Установа соединения
protonvpn-cli c -f || protonvpn-cli c -r &&
protonvpn-cli ks --on &&
protonvpn-cli r

```
> **Note** Бесплатный логин распространяется на одно устройтсво. Если учетка уже использована на другом устройтсве, подключение длится долго и оканчивается неудачей. Повторите запуск под другой учеткой.

#### Автоматизация запуска
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

## Оптимизация
##### Проверка скорости
```sh
sudo apt install -y speedtest-cli &&
speedtest-cli

```
> **Note**
> Если появляется `...ERROR: HTTP Error 403: Forbidden`, повторите `speedtest-cli` через две минуты

##### Альтернативная проверка скорости
```sh
wget -O /dev/null http://speedtest.wdc01.softlayer.com/downloads/test100.zip

```

##### Переподключение к наиболее быстрому серверу
```sh
protonvpn-cli d &&
protonvpn-cli c -f

```
> Если быстрый сервер найден, заново настройте автоматический запуск. 
##### Переподключение к случайному серверу
```sh
protonvpn-cli d &&
protonvpn-cli c -r

```

## Удаление
##### Восстановление незащищенного доступа в Интернет
```sh
protonvpn-cli ks --off ||
# если вышеуазанное невозможно
nmcli device del pvpnksintrf0

```
##### Отключение автозагрузки
Пуск ➜ Параметры ➜ Автозагрузка ➜ Proton VPN CLI ➜ **[ВЫК]**

## Ссылки
См. [использование](https://protonvpn.com/support/linux-vpn-tool/#cli) на сайте производителя.
