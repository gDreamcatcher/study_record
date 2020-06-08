# vim-go
## 安装zsh

### 预安装
```shell
yum install -y wget curl git make cmake gcc gcc-c++ python3 python3-devel
```
### yum 安装
```shell
yum install -y zsh
```

### 源码安装
centos7 使用yum安装的的zsh版本太低, 可以选择源码编译安装
```
wget https://nchc.dl.sourceforge.net/project/zsh/zsh/5.8/zsh-5.8.tar.xz
xz -d zsh-5.8.tar.xz
tar -xvf zsh-5.8.tar
cd zsh-5.8
./configure
make -j && make install
```

### 安装oh-my-zsh
```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 配置个性化.zshrc
cat >> ~/.zshrc <<EOF
# key bindings
bindkey "\e[1~" beginning-of-line
bindkey "\e[4~" end-of-line

alias ls='ls --color=auto'
alias la='ls -A --color=auto'
alias lh='ls -lh --color=auto'
alias lha='ls -lhA --color=auto'
alias ll='ls -l --color=auto'
alias lla='ls -Al --color=auto'

alias psf='ps -ef|grep -v grep|grep'
pkl(){
  ps -ef |grep $1 |grep -v grep
  ps -ef |grep $1 |grep -v grep|cut -c 9-15 |xargs kill -9
  echo 'kill $1'
}
export TERM=xterm-256color 
export LANG="zh_CN.UTF-8"
export TZ='Asia/Shanghai'
EOF

```

## 安装vim

```shell
yum install -y vim
# 或者
PYTHON="～/package/miniconda3/envs/py36/bin/python" ./configure --with-features=huge --enable-python3interp=dynamic --enable-python3interp --prefix=～/local/vim

# 配置vim
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
curl -fLo ~/.vimrc https://raw.githubusercontent.com/BroQiang/vim-go-ide/master/vimrc
# add set encoding=utf-8 in .vimrc
# 打开.vimrc then input :PlugInstall

```
### 插件配置
上面一起安装了很多个插件，有些插件要单独配置，记录到下面

#### vim-go
这个是 go 语言支持插件，上面插件完成后还需要安装很多个 Go 的包才能正常工作，
由于国内下载go模块比较慢，所以添加proxy
```shell
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```
然后在 vim 中执行下面命令：`:GoInstallBinaries`

出现 `vim-go: installing finished!` 安装成功，可以使用 Go 包的相关功能了

#### YouCompleteMe
这个插件是用来自动完成的，不过需要手动做一些额外的配置

```shell
# 1、安装以来关系
sudo apt install build-essential cmake python3-dev
# 2、编译，并加入 go 的支持
cd ~/.vim/plugged/YouCompleteMe
python3 install.py --go-completer 
```

## **安装tmux**

```shell
yum install -y tmux
```

### **plugin插件安装**
```shell
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```
将下面的内容添加到`.tmux.conf`，然后执行`prefix+I`，等待插件安装完就可以了
```shell
# sets prefix to Ctrl-A
set -g prefix C-a
unbind C-b

# sets shell do the default user shell
set -g default-shell /usr/bin/zsh
setw -g automatic-rename off
setw -g allow-rename off

# 256 colors
set -g default-terminal "xterm-256color"

bind -n End send-key C-e
bind -n Home send-key C-a

# reloads .tmux.conf
unbind r
bind r source-file ~/.tmux.conf

# for mouse
set -g mouse on
# Sane scrolling
set -g terminal-overrides 'xterm*:smcup@:rmcup@'

bind c new-window -c "#{pane_current_path}"
bind | split-window -v -c  "#{pane_current_path}"
bind - split-window -h -c  "#{pane_current_path}"

set -sg escape-time 1
# 让窗口索引从 1 开始
set -g base-index 1
# 让面板索引从 1 开始
setw -g pane-base-index 1

# 调整移动键
bind h select-pane -L # 左
bind j select-pane -D # 下
bind k select-pane -U # 上
bind l select-pane -R # 右

set -g status-fg white
set -g status-bg black

setw -g monitor-activity on
set -g visual-activity on

# 设置状态栏左侧的内容和颜色
set -g status-left-length 40
set -g status-left "[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P"

# 设置状态栏右侧的内容和颜色
# 15% | 28 Nov 18:15
set -g status-right "#(~/battery Discharging) | #[fg=cyan]%d %b %R"

# 每 60 秒更新一次状态栏
set -g status-interval 60

# windows 中间显示
set -g status-justify centre

source-file ${HOME}/.tmux/plugins/tmux-themepack/powerline/default/green.tmuxtheme

# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'jimeh/tmux-themepack'

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
```