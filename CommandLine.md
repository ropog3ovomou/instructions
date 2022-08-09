# Настройка текстового интерфейса в Debian / Ubuntu / Mint

## На основе Bash
```sh
sudo apt install -y tmux powerline
cat > ~/.tmux.conf<<END
run-shell 'powerline-config tmux setup'
set-window-option -g mode-keys vi
END
cat >> ~/.bashrc<<END
if [ -f /usr/share/powerline/bindings/bash/powerline.sh ]; then
powerline-daemon -q
POWERLINE_BASH_CONTINUATION=1
POWERLINE_BASH_SELECT=1
source /usr/share/powerline/bindings/bash/powerline.sh
fi
END

```

## На основе Zsh
```sh

```
