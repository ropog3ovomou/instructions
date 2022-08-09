G# Настройка Linux Mint после установки
Выполняйте инструкции по очереди. Помеченные как необязательные можете пропускать. Строки необходимо запускать в терминале (см. [командная строка для новичков](https://www.youtube.com/watch?v=qwopGsaNF_Q)).

> **Warning**
> Mint предлагает выполнить часть этой же настройки с помощью визуальных средств. Не запускайте предлагаемые шаги параллельно с этой настройкой.
> Появляющиеся автоматически окна можно закрыть.

## Начальная настройка

#### Установка улучшенных серверов обновления (необязательно)
```sh
sudo mintsources

```
> В появившемся окне необходимо поменять зеркала: **Основной (...)** и **Базовый (...)**, после чего закрыть окно.
> Списки автоматически сортируются по скорости. Желательно выбрать самый быстрый сервер. Для этого подождите, пока список отсортируется и выберите первый пункт.

#### Обновление версий ПО

> **Note**
> Эта операция занимает примерно полчаса
```sh
sudo apt update && sudo apt upgrade -y

```

#### Обновление основной версии Mint
###### Резервное копирование
> **Warning**
> Занимает порядка 10 минут и прогресс может не отображаться. Не прерывайте процесс.
```sh
sudo timeshift --create

```
###### Запуск диалога обновления
```sh
sudo apt install -y mintupgrade && sudo mintupgrade && sudo reboot

```
###### После перезагрузки - удаление резервных копий (необязательно)
```sh
sudo timeshift --delete-all

```

#### Добавление поддержки колесика мыши
> **Note** 
> Без этого приложения реагируют на прокручивание случайным образом.
###### Установка
```sh
sudo apt install imwheel
```
###### Активация
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
## Безопасность
#### Снятие пароля с экранной заставки (необязательно)
Пуск -> Параметры -> Экранная заставка -> Настройки -> Задержка перед запуском заставки: Никогда
#### Установка разблокировки связки ключей при автоматическом входе
```sh
echo > tmp.sh 'echo -n mint|gnome-keyring-daemon -l' &&
sudo cp tmp.sh /etc/profile.d/unlock_keyring.sh && rm tmp.sh

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
sudo apt install qbittorrent -y

```
> Запускается из меню -> Интернет

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
sudo apt install remmina remmina-plugin-rdp remmina-plugin-secret

```
> Запускается из меню -> Интернет

#### Brave browser (содержит Tor)
```sh
sudo apt install apt-transport-https curl -y &&\
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg &&\
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg arch=amd64] https://brave-browser-apt-release.s3.brave.com/ stable main"|sudo tee /etc/apt/sources.list.d/brave-browser-release.list &&\
sudo apt update &&\
sudo apt install brave-browser -y

```
> Запускается из меню -> Интернет
