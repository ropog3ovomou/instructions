# Настройка текстового интерфейса в Debian / Ubuntu / Mint

## Общее
```sh
sudo apt install -y neovim git tmuxp fzf fasd vifm powerline ripgrep bat stow chafa \
fortune cowsay lolcat sl cmatrix &&

# colors
ln -s `which batcat` ~/.local/bin/bat
git clone https://github.com/chriskempson/base16-shell.git ~/.config/base16-shell &&
(
cd ~/.config/base16-shell &&
git reset --hard ae84047d378700bfdbabf0886c1fb5bb1033620f
mkdir -p scripts/best/dark scripts/best/light &&
(cd scripts/best/light && for s in github gruvbox-light-soft; do ln -s ../../base16-$s.sh .;done) &&
(cd scripts/best/dark && for s in material-palenight material-darker default-dark chalk brewer bright \
harmonic-dark greenscreen heetch irblack outrun-dark papercolor-dark phd pop rebecca summerfruit-dark
do ln -s ../../base16-$s.sh .;done)
) &&
tmp=`mktemp`
(
echo '# Best themes'
for c in light dark; do
  tail -n7 ~/.config/base16-shell/profile_helper.sh|sed -e "s/scripts/scripts\/best\/$c/" -e "s/base16_/16$c-/"
done
) > $tmp
cat $tmp >> ~/.config/base16-shell/profile_helper.sh
(cat >>~/.rc<<END
export MANPAGER="sh -c 'sed -e s/.\\\\x08//g | bat -l man -p'"
export EDITOR=vi
# Base16 Shell
BASE16_SHELL="\$HOME/.config/base16-shell/"
[ -n "\$PS1" ] && \\
    [ -s "\$BASE16_SHELL/profile_helper.sh" ] && \\
        eval "\$("\$BASE16_SHELL/profile_helper.sh")"
END
) &&
# bash
cat >> ~/.bashrc<<END
if [ -f $HOME/.rc ]; then
  source $HOME/.rc;
fi
END

```

## Tmux
```sh
git clone https://github.com/hamvocke/dotfiles.git &&
cd dotfiles &&
stow tmux &&
sed -i -e 's/\<C-a\>/M-a/' -e 's/^set -g default-terminal.*$/\#\0/' -e 's/select-pane -.$/\0Z/' ~/.tmux.conf &&
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm &&
(cat >> ~/.tmux.conf<<END
# Shortcuts
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel "xclip -i -f -selection primary | xclip -i -selection clipboard"
bind -n M-F11 resize-pane -Z
bind -n S-PgUp copy-mode -u
# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
# Automatically restore active sessions on reboot
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @resurrect-strategy-vim 'session'
set -g @resurrect-strategy-nvim 'session'
set -g @resurrect-capture-pane-contents 'on'
set -g @continuum-restore 'on'
set -g @continuum-boot 'on'
run-shell 'powerline-config tmux setup'
run '~/.tmux/plugins/tpm/tpm'
END
) &&

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
sudo apt install zsh powerline
chsh -s $(which zsh)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $HOME/.oh-my-zsh/custom/themes/powerlevel10k
mkdir -p ~/.fonts
cd ~/.fonts/
for x in Regular Bold Italic Bold%20Italic; do wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20$x.ttf\;done
cd -
fc-cache -f -v
gconftool-2 --set /apps/gnome-terminal/profiles/Default/font --type string "Ubuntu Mono derivative Powerline 11"
gconftool-2 --set /apps/gnome-terminal/profiles/Default/use_system_font --type=boolean false
sed -i 's/POWERLEVEL9K_DIR_BACKGROUND=.*$/POWERLEVEL9K_DIR_BACKGROUND=31/' ~/.p10k.zsh
cd ~/.oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions.git
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
git clone https://github.com/unixorn/fzf-zsh-plugin.git
cd -
sed -i '/^plugins/s/)/ zsh-autosuggestions zsh-syntax-highlighting fzf-zsh-plugin)/' ~/.zshrc
(cat >>~/.zshrc<<END
source ~/.rc
END
) &&
```
## Sources
Adapted from:
- https://betterprogramming.pub/how-to-use-tmuxp-to-manage-your-tmux-session-614b6d42d6b6
- https://erikzaadi.com/2020/02/17/how-to-use-zsh-and-tmuxp-to-speed-up-your-day-to-day-workflow/
- https://github.com/hamvocke/dotfiles
- https://www.freecodecamp.org/news/tmux-in-practice-integration-with-system-clipboard-bcd72c62ff7b/
- https://man7.org/linux/man-pages/man1/tmux.1.html
- https://github.com/sharkdp/bat/issues/523
