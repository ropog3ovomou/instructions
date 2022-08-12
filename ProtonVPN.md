# Установка Proton VPN на Linux Mint
В результате выполнения данной инструкции VPN
всегда будет включен, что предотвратит утечку пакетов.
Преимущества **Proton VPN** это высокая скорость и бесплатный базовый план. 
Сервис разработан в Швейцарии при поддержке Европейского Союза. 

## Подключение
#### Установка
```sh
# Установка источника пакета
wget https://protonvpn.com/download/protonvpn-stable-release_1.0.1-1_all.deb &&
sudo dpkg -i protonvpn-stable-release_1.0.1-1_all.deb &&
rm protonvpn-stable-release_1.0.1-1_all.deb &&
# Установка пакета
sudo apt-get update &&
sudo apt-get install -y protonvpn-cli

```
#### Первый запуск
При необходимости [зарегистрируйтесь](https://protonvpn.com/free-vpn/linux). 
> **Warning**
> Если Вы пользуетесь другими сервисами Протона -- не используйте для VPN учетные записи, о которых другие не должны знать или связывать с вами. Создайте отдельную учетку.
> 
> **Note** Бесплатный логин распространяется на одно устройтсво. Если учетка уже использована на другом устройтсве, подключение длится долго и оканчивается неудачей. Повторите запуск под другой учеткой.
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
#### Проверка скорости подключения
```sh
sudo apt install -y bat speedtest-cli &&
protonvpn-cli s | batcat -p -l c -H 5 &&
echo -n Проверяем скорость... &&
speedtest-cli --simple --bytes --single --secure |batcat -p -l c

```
> **Note**
> Если появляется `...ERROR: HTTP Error 403: Forbidden`, повторите проверку через две минуты

##### Альтернативная проверка скорости
```sh
wget -O /dev/null http://speedtest.wdc01.softlayer.com/downloads/test100.zip

```
##### Переподключение к потенциально более быстрому серверу
```sh
protonvpn-cli c --cc NL &&
protonvpn-cli s | batcat -p -l c -H 5 

```
#### Автоматизация запуска
1. Подключитесь к серверу
2. Выполните скрипт, приведенный ниже
3. Перезагрузите компьютер и убедитесь, что VPN подключается автоматически.
4. Поменяйте источники обновлений для Mint (см. [Установка улучшенных серверов обновления](%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0%20Mint.md#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D1%83%D0%BB%D1%83%D1%87%D1%88%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%BE%D0%B2-%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BD%D0%B5%D0%BE%D0%B1%D1%8F%D0%B7%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE) в начальной настройке)
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

## Отключение
Иногда возникает необходимость временного доступа в Интернет напрямую. Ниже следуют инструкции для такого случая.
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
