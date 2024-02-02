# zsh 提示符


在 [zshparam(1)](https://man.archlinux.org/man/zshparam.1) 中可以看到 zsh 的各种提示符可以通过修改 PROMPT, PROMPT2, PROMPT3, PROMPT4, RPROMPT 等变量的值进行设置, 它们的基本语法相同, 其中 PROMPT 是主要的提示符, 本文也以它为例。

## zsh PROMPT 转义字符
zshparam(1) 中关于 PS1 的说明 (zsh 中 PROMPT, prompt, PS1 等价) 指向了 [zshmisc(1)](https://man.archlinux.org/man/zshmisc.1)。

在 zshmisc(1) 的 [SIMPLE PROMPT ESCAPES](https://man.archlinux.org/man/zshmisc.1#SIMPLE_PROMPT_ESCAPES) 一节中可以看到到所有的转义字符，以下列出一些我认为有用的:
- 登陆信息:
  - `%M`: 完整的主机名
  - `%n`: 用户名
- Shell 状态:
  - `%#`: 特权模式下是一个井号否则是一个百分号
  - `%?`: 上一个命令的返回状态
  - `%d` 或 `%/`: 当前目录
  - `%~`: 当前目录，若以 $HOME 起始，则将之替换为一个波浪号
  - `%j`: 作业数
- 时间和日期:
  - `%D`: 年-月-日
  - `%T`: 时:分
  - `%*`: 时:分:秒
- 样式:
  - `%B (%b)`: 开始 (结束) 粗体
  - `%F{red} (%f)`: 开始 (结束) 颜色
- 条件子串:
  - `%(x.true-text.false-text)`: x 为条件, true-text 为条件为真时显示的内容, false-text 为条件为假时显示的内容。x 有特定的选择, 当 x 为 ? 时，表示上一个命令的返回状态, 默认 0 为真, 其他为假。

## git 当前分支名
在 [PRO Git 的 Git in Zsh](https://git-scm.com/book/en/v2/Appendix-A%3A-Git-in-Other-Environments-Git-in-Zsh) 一节中可以看到一个在右侧显示分支名的实例:
```sh
autoload -Uz vcs_info
precmd_vcs_info() { vcs_info }
precmd_functions+=( precmd_vcs_info )
setopt prompt_subst
RPROMPT='${vcs_info_msg_0_}'
# PROMPT='${vcs_info_msg_0_}%# '
zstyle ':vcs_info:git:*' formats '%b'
```
在 `zstyle ':vcs_info:git:*' formats '%b'` 一句中 `'%b'` 指定了显示的样式，`%b` 表示分支名。


## 我的 PROMPT 配置
推荐一个网站 [zsh-prompt-generator](https://zsh-prompt-generator.site/) 用于生成简单的 PROMPT。

首先我希望提示符分为两行，这只需要加一个换行即可:
```sh
export PROMPT='┌──
└'

┌──
└
```
在第一行中显示当前的时间和当前目录:
```sh
export PROMPT='┌──(%T)─[%~]
└'

┌──(20:00)─[~]
└
```
在第二行中, 若上一个命令的返回状态不为 0 则显示此返回状态, 否则按照 `%#` 显示:
```sh
export PROMPT='┌──(%T)─[%~]
└(%(?.%#.%?))─'

┌──(20:00)─[~]
└(%)─false
┌──(20:00)─[~]
└(1)─
```
在第二行显示当前分支名:
```sh
autoload -Uz vcs_info
precmd_vcs_info() { vcs_info }
precmd_functions+=( precmd_vcs_info )
setopt prompt_subst
zstyle ':vcs_info:git:*' formats '[%b]─'

export PROMPT='┌──(%T)─[%~]
└(%(?.%#.%?))─${vcs_info_msg_0_}'

┌──(21:27)─[~/git-test]
└(%)─[master]─
```
最后设置一下颜色:
```sh
autoload -Uz vcs_info
precmd_vcs_info() { vcs_info }
precmd_functions+=( precmd_vcs_info )
setopt prompt_subst
zstyle ':vcs_info:git:*' formats '%F{10}[%f%b%F{10}]─%f'

export PROMPT='%F{10}┌──(%f%F{11}%T%f%F{10})─[%f%F{15}%~%f%F{10}]
└(%f%(?.%F{14}%#%f.%F{9}%?%f)%F{10})─%f${vcs_info_msg_0_}'
```
