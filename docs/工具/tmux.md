# tmux

## session操作

### 新建session

```shell
tmux new -s <session-name> 
```

### 离开session

```shell
# ctrl + b d
tmux detach
```

### 进入session

```shell
tmux attach -t <session-name>
```

### 关闭session

```shell
tmux kill-session -t <session-name>
```

### 切换session

```shell
tmux switch -t <session-name>
```

## 窗格操作

切割窗格的快捷键 ctrl+b % 可以快速的左右切割，ctrl+b " 可以快速的上下进行切割。

快捷键 ctrl+b z ,将会放大当前操作的窗格，继续触发该快捷键将会还原当前的窗格。

快捷键 ctrl+b t 将会把在当前的窗格当中显示时钟。

在vim按键模式下 ctrl+b h/j/k/l 用来在不同的窗格之间切换。

ctrl+b [ 在当前窗格中进行滚屏操作

点击shift并使用鼠标操作可恢复终端默认鼠标功能，复制粘贴会方便一些。

ctrl+b q 显示窗格编号。

ctrl+b alt+方向键调整窗格长宽。

## 窗口操作

创建窗口的快捷键ctrl+b c, 可以通过快捷键快速的创建一个窗口出来。

ctrl+b n 快速切换到下一个窗口；ctrl+b p 快速切换到上一个窗口。

ctrl+b num 快速切换到指定序号窗口。

ctrl+b , 该快捷键可以重新命名窗口

## 配置文件

配置文件为~/.tmux.conf，以下是一个简易的配置文件实例：

```
# 基础设置
set -g default-command "/bin/bash"
set -g default-terminal "screen-256color"
set -g display-time 3000
set -g escape-time 0
set -g history-limit 65535
set -g base-index 1
set -g pane-base-index 1
set -g status-style bg=blue

# 选中窗口
bind-key k select-pane -U
bind-key j select-pane -D
bind-key h select-pane -L
bind-key l select-pane -R

# copy-mode 将快捷键设置为 vi 模式
setw -g mode-keys vi

# 启用鼠标(Tmux v2.1)
set -g mouse on
```
