# Настройка текстового интерфейса в Debian / Ubuntu / Mint

## Общее
```sh
sudo apt install -y neovim git tmuxp fzf vifm powerline ripgrep bat stow &&
git clone https://github.com/hamvocke/dotfiles.git &&
cd dotfiles &&

# base16 colors
git clone https://github.com/chriskempson/base16-shell.git ~/.config/base16-shell &&
(cat >>~/.bashrc<<END
# Base16 Shell
BASE16_SHELL="\$HOME/.config/base16-shell/"
[ -n "\$PS1" ] && \\
    [ -s "\$BASE16_SHELL/profile_helper.sh" ] && \\
        eval "\$("\$BASE16_SHELL/profile_helper.sh")"
END
) &&

# tmux
stow tmux &&
sed -i -e 's/^set -g default-terminal.*$/\#\0/' -e 's/select-pane -.$/\0Z/' ~/.tmux.conf &&
(cat >> ~/.tmux.conf<<END
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel "xclip -i -f -selection primary | xclip -i -selection clipboard"
bind -n M-F11 resize-pane -Z
bind -n S-PgUp copy-mode -u
run-shell 'powerline-config tmux setup'
END
) &&

# bash
cat >> ~/.bashrc<<END
if [ -f /usr/share/powerline/bindings/bash/powerline.sh ]; then
powerline-daemon -q
POWERLINE_BASH_CONTINUATION=1
POWERLINE_BASH_SELECT=1
source /usr/share/powerline/bindings/bash/powerline.sh
fi
END

```
## Neovim
```sh
cat >~/.vimrc<<END
"keep history of hidden buffers
se hidden

"visual setup
sy on
color slate
set gfn=SourceCodeProForPowerline-ExtraLight:h18

"indentation
se ts=4 sts=4 sw=4 et

"search
se hlsearch
se ignorecase
set isfname+=32 "filenames have spaces. 

"Alternative language
set keymap=russian-jcukenwin "add Russian
set iminsert=0 "start with English
"Change language on alt-;
imap <esc>; <c-^>
map <esc>; a<c-^><esc>

map <space> :up\|b#<c-M>
"save on and alt-s
nnoremap <esc>s :up<c-M>
"capitalize last word
inoremap <c-F> <esc>bgUeea
"ls buffers on alt-l
nnoremap <esc>l :ls<c-M>:b
"don't skip wrapped lines
nnoremap j gj
nnoremap k gk
END
mkdir -p ~/.config/tmuxp/
cat >~/.config/tmuxp/tests-shell-editor.yml <<END
session_name: "${APP}"
shell_command_before: "nvm use default"
start_directory: "~/src/${APP}"
windows:
- focus: 'true'
  layout: 2878,156x67,0,0{56x67,0,0[56x33,0,0,1,56x33,0,34,3],99x67,57,0,2}
  options:
    automatic-rename: 'off'
  panes:
  - vifm 
  - git status 
  - focus: 'true'
    shell_command: nvim "-c '0"
  window_name: dev
 END
cat >>~/.bashrc<<END
alias ms="tmuxp load -y"
wo()
{
    APP=\$1 NODE_VERSION=\${2:-default} ms tests-shell-editor
}
END
```

## Zsh
```sh



```
## Sources
Adapted from:
- https://betterprogramming.pub/how-to-use-tmuxp-to-manage-your-tmux-session-614b6d42d6b6
- https://erikzaadi.com/2020/02/17/how-to-use-zsh-and-tmuxp-to-speed-up-your-day-to-day-workflow/
- https://github.com/hamvocke/dotfiles
- https://www.freecodecamp.org/news/tmux-in-practice-integration-with-system-clipboard-bcd72c62ff7b/
- 
