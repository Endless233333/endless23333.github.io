# linux 多种终端复用器简介


Linux 终端复用器允许用户在单个窗口中创建或启用多个终端, 通常在连接到远程服务器时会使用, 即便链接意外中断, 也可以通过终端复用器的相关功能恢复到之前的工作状态。

本文将简要介绍 screen, tmux 和 zellij 三款终端复用器, 其中 screen 和 tmux 在 ubuntu server 22.04 中自带, 而 zellij 的使用比较方便。

## screen
Screen 是一个全屏窗口管理器, 它在交互式 shell 之间多路传输物理终端。在用户离开它管理的窗口时, 窗口内的程序仍会运行。

创建一个会话可以使用以下三种方式:
- `screen`
  - 直接创建并以 linux 的 hostname 命名
- `screen -S <name>`
  - 创建指定名字的会话
- `screen -R <name>`
  - 先尝试进入指定会话, 如果没有创建的话则创建它

创建的会话是可以重名的, 不过它们的 pid 不同, 可以通过这来区分, 建议使用第三种方式创建。

在进入绘画后使用 `ctrl-a d` 分离会话。

使用 `screen -ls` 可以查看所有会话的 pid, 名字, 状态。

使用 `screen -r [pid/name]` 或 `screen -R [pid/name]` 重新进入会话。

在会话中 `exit` 或 `ctrl-d` 终止此会话, 也可以在外面通过 `screen -R [pid/name] -X quit` 终止。

## tmux
tmux 可以管理多个会话, 一个会话中可以有多个窗口, 一个窗口中可以划分称多个面板。

tmux 中的快捷键都需要先 `ctrl-b` 才能使用, `ctrl+b ?` 获取帮助。

使用 `tmux` 或者 `tmux new -s <name>` 创建一个新的会话, 后者可以指定名字。

`ctrl+b d` 或者 `tmux detach` 分离会话。

`tmux ls` 列出列出所有会话。

`tmux attach -t [id/name]` 进入会话。

`tmux kill-session -t [id/name]` 删除会话。

## zellij
zellij 使用 rust 开发, 默认情况下进入 zellij 后快捷键都会在屏幕下方提示, 所以入门门槛比较低。

直接 `zellij` 就可以进入 zellij, 使用 `ctrl-s d` 分离会话。

在会话中 `ctrl-o w ctrl-r` 重命名会话。

`zellij ls` 列出所有会话。

`zellij a <name>` 进入会话。

`zellij d <name>` 删除会话。
