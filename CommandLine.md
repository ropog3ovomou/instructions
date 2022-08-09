# Настройка текстового интерфейса в Debian / Ubuntu / Mint

## Bash
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

## Zsh
```sh
sudo apt install -y tmuxp &&
mkdir -p ~/.config/tmuxp/
cat >~/.config/tmuxp/tests-shell-editor.yml <<END
session_name: "\${APP}"
shell_command_before: "nvm use ${NODE_VERSION:-default}"
start_directory: "~/src/\${APP}"
windows:
  - panes:
      - vim
    window_name: editor
  - panes:
      - git status
    window_name: shell
  - layout: even-horizontal
    panes:
      - yarn test --watch
    window_name: tests
END
cat >>~/.bashrc<<END
alias ms="tmuxp load -y"
wo()
{
    APP=$1 NODE_VERSION=${2:-default} ms tests-shell-editor
}
```
