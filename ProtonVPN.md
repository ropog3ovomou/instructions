# Установка Proton VPN на Mint
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
# Установка пакетов
sudo apt-get update &&
sudo apt-get install -y protonvpn-cli bat

```
#### Первый запуск (установка соединения вручную)
При необходимости [зарегистрируйтесь](https://protonvpn.com/free-vpn/linux). 
> **Warning**
> Учетная запись VPN будет храниться на компьютере постоянно. Не используйте учетные записи, о которых другие не должны знать или связывать с вами. Создайте отдельную учетку.
> 
> **Note** Бесплатный логин распространяется на одно устройтсво. Если учетка уже использована на другом устройтсве, подключение длится долго и оканчивается неудачей. Повторите запуск под другой учеткой.
```sh
# Логин
clear &&
echo -e '\033[3B\033[1;37m1. Бесплатная регистрация: \033[36mhttps://protonvpn.com/free-vpn/linux\033[0m\n' &&
echo -n '2. Enter your proton username here> ' &&
read uname &&
protonvpn-cli logout >/dev/null || true &&
protonvpn-cli ks --off >/dev/null &&
protonvpn-cli login "$uname" &&
# Установа соединения
protonvpn-cli c -f || protonvpn-cli c --cc NL || protonvpn-cli c -r &&
protonvpn-cli ks --on &&
protonvpn-cli r

```
#### Проверка скорости подключения
```sh
protonvpn-cli s | batcat -p -l c -H 5 &&
echo 'Тестовое скачивание (Ctrl-C чтобы прервать):'
wget -nv --show-progress -O /dev/null http://speedtest.wdc01.softlayer.com/downloads/test100.zip

```
##### Переподключение к потенциально более быстрому серверу
```sh
sudo apt-get install -y vramsteg
js="$HOME/.cache/protonvpn/cached_serverlist.json"
ss="$(jq -r '.LogicalServers[]|.Name' $js|grep NL-FREE)"
protonvpn-cli ks --off
echo Comparing server speedsp...
t=$(vramsteg --now)
best=$(for s in $ss; do
  vramsteg -t -s $t -m 0 -c $((n++)) -x $(echo $ss|wc -w) >&2 &&
  sp=$(protonvpn-cli c $s >/dev/null 2>&1 && curl -so /dev/null -w '%{speed_download}' https://bit.ly/3SJ7EP3)
  echo $s ${sp:-0}
done|sort -rnk2 | head -n1 | cut -d' ' -f1)
until protonvpn-cli c NL-FREE#128; do echo Reconnecting...; done
protonvpn-cli ks --on

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
protonvpn-cli c ${srv:=-f} || protonvpn-cli c ${srv:=-f} || protonvpn-cli c --cc NL || protonvpn-cli c -r
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
