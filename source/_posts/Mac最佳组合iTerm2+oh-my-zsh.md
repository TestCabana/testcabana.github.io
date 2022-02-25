---
title: Mac最佳组合iTerm2+oh-my-zsh
categories: Mac实用工具
tags:
	- iTerm2
	- oh-my-zsh
---

> 工欲善其事，必先利其器，作为一个长期使用 `Vim` 作为主力编程开发工具的我来说，在命令行下面工作是必不可少的一件事情了。虽然 `Mac` 系统自带了 `Terminal` 工具，但是功能实在是太弱，而且长的太丑了，长此用下去，必定是不能忍的。于是，我开始使用 `iTerm2+oh-my-zsh` 的最佳组合。



![最佳组合iTerm2+oh-my-zsh](https://www.escapelife.site/images/learn-iterm2-terminal.png)



------

## 1. 最佳组合介绍

> 主要介绍，iTerm2 和 oh-my-zsh 的特点和功能。

### 1.1 iTerm2

- **[-] 主要特点**
  - => [iTerm2 官方网站地址](https://www.iterm2.com/index.html)
  - `iTerm2` 是一款完全免费的，专为 `MacOS` 用户打造的命令行应用
  - `iTerm2` 本身支持自定义配色，自定义快捷键，自定义标签配置等等
  - `iTerm2` 本身支持方便的水平和垂直分屏功能，便捷的快速搜索等等
  - 毫不避讳的讲，`iTerm2` 是我用过 `MacOS` 下最好的终端工具也不为过
- **[1] 拆分窗格**



![](https://www.iterm2.com/img/screenshots/split_panes_full.png)



- **[2] 热键窗口**



![](https://www.iterm2.com/img/screenshots/hotkeywindow_full.png)



- **[3] 屏幕搜索**



![](https://www.iterm2.com/img/screenshots/search.png)



- **[4] 自动补全**



![最佳组合iTerm2+oh-my-zsh](https://www.iterm2.com/img/screenshots/autocomplete.png)



- **[5] 历史记录**



![最佳组合iTerm2+oh-my-zsh](https://www.iterm2.com/img/screenshots/pastehistory.png)



- **[6] 即时重播**



![最佳组合iTerm2+oh-my-zsh](https://www.iterm2.com/img/screenshots/instantreplay.gif)



- **[7] 标记配置**



![最佳组合iTerm2+oh-my-zsh](https://www.iterm2.com/img/screenshots/profiles1_full.png)



------

### 1.2 oh-my-zsh

- **[-] 主要特点**
  - => [oh-my-zsh 官方网站地址](https://github.com/robbyrussell/oh-my-zsh)
  - 使用上 `iTerm2` 终端利器后，当然还得配上一款顺手的 `Shell` 解释器
  - 虽然使用最广泛的时 `Bash` 解释器，但是可定制性和可扩展性不够强大
  - 跟 `Bash` 相比，`Zsh` 的功能强大了许多，可以自动补全命令且支持插件
  - 但是 `Zsh` 的默认配置极其复杂繁琐，让人望而却步，很难新手直接上路
  - 开源项目 `oh-my-zsh` 让 `Zsh` 的配置降到 `0` 门槛，且兼容 `Bash` 解释器
- **[1] 进程 ID 号补全**



![最佳组合iTerm2+oh-my-zsh](https://xiaozhou.net/pics/iterm/zsh1.gif)



- **[2] 路径快速跳转**



![最佳组合iTerm2+oh-my-zsh](https://xiaozhou.net/pics/iterm/zsh2.gif)



- **[3] 目录简写补全**



![最佳组合iTerm2+oh-my-zsh](https://xiaozhou.net/pics/iterm/zsh3.gif)



- **[4] 命令参数补全**



![最佳组合iTerm2+oh-my-zsh](https://xiaozhou.net/pics/iterm/zsh4.gif)



------

## 2. 组合安装配置

> 这里主要说下，两款工具的安装和基本配置。

### 2.1 组合安装

- iTerm2
  - 在官网直接下载安装就可以使用了，很方便



![最佳组合iTerm2+oh-my-zsh](https://www.escapelife.site/images/learn-iterm2-terminal-1.png)



- oh-my-zsh
  - 安装之前需要确保系统安装了`git`和`wget`工具



```bash
# 1.安装brew工具
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# 2.安装git和wget工具
$ brew install git
$ brew install wget

# 3.安装oh-my-zsh工具
$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

------

### 2.2 组合配置

- [1] iTerm2 主题配置
  - 支持许多的主题配色，可以自定义也可以参考网上现成的主题配色

| 编号 | 主题配色              | 对应链接地址                                         |
| :--- | :-------------------- | :--------------------------------------------------- |
| 1    | **Solarized 的配色**  | `https://github.com/altercation/solarized`           |
| 2    | **iTerm2 的配色合集** | `http://iterm2colorschemes.com/`                     |
| 3    | **GitHub 的配色合集** | `https://github.com/mbadolato/iTerm2-Color-Schemes/` |



![最佳组合iTerm2+oh-my-zsh](https://www.escapelife.site/images/learn-iterm2-terminal-2.png)



- [2] 安装 Powerline 字体
  - 为了终端下能正确的显示 `fancy` 字符，需要安装 `powerline` 字体

| 编号 | 主题字体           | 对应链接地址                         |
| :--- | :----------------- | :----------------------------------- |
| 1    | **Powerline 字体** | `https://github.com/powerline/fonts` |



![最佳组合iTerm2+oh-my-zsh](https://www.escapelife.site/images/learn-iterm2-terminal-3.png)



- [3] 切换 Shell 解释器
  - 如果你是用的`MacOS`系统的话，默认应该自带了 `Zsh` 解释器了



```bash
# 查看系统的解释器
$ cat /etc/shells
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh

# 切换Shell解释器
$ chsh -s /bin/zsh
```

- [4] 安装 Zsh 常用插件
  - `Zsh` 支持插件，通过插件扩展可以实现许多方便的功能

| 编号 | 常用插件推荐                  | 实现的功能介绍                 |
| :--- | :---------------------------- | :----------------------------- |
| 1    | **`autojump`**                | 一个目录直接快速跳转的效率工具 |
| 2    | **`web-search`**              | 终端搜索工具支持常用的搜索引擎 |
| 3    | **`zsh-autosuggestions`**     | 一个十分好用的命令自动提示插件 |
| 4    | **`zsh-syntax-highlighting`** | 终端输入时命令可以高亮语法着色 |



![最佳组合iTerm2+oh-my-zsh](https://xiaozhou.net/pics/iterm/zsh5.gif)



------

## 3. iTerm2 快捷键

> 使用快捷键可以极大地效率化，我们的日常操作。

### 3.1 基本操作

- **移动删除操作**

| 编号 | 命令按键    | 命令解释说明           |
| :--: | :---------- | :--------------------- |
| 1    | **`⌃ + a`** | 将光标移动到行首       |
| 2    | **`⌃ + e`** | 将光标移动到行首       |
| 3    | **`⌃ + k`** | 从光标处删至命令行尾   |
| 4    | **`⌃ + u`** | 从光标处删至命令行首   |
| 5    | **`⌃ + p`** | 递归显示上一条命令     |
| 6    | **`⌃ + n`** | 递归显示下一条命令     |
| 7    | **`⌃ + w`** | 从光标处删至单词首     |
| 8    | **`⌃ + h`** | 从光标处前删除一个字母  |
| 9    | **`⌃ + d`** | 从光标处后删除一个字母  |
| 10   | **`⌃ + y`** | 粘贴至光标后           |
| 11   | **`⌃ + r`** | 搜索命令历史           |

- **窗口标签操作**

| 编号 | 命令                               | 效果                       |
| :--: | :--------------------------------- | :------------------------- |
| 1    | **`⌘ + n`**                        | 新建窗口                   |
| 2    | **`⌘ + t`**                        | 新建标签页                 |
| 3    | **`⌘ + w`**                        | 关闭当前页                 |
| 4    | **`⌥⌘ + 数字`**                    | 切换窗口                   |
| 5    | **`⌘ + [`和`⌘ + ]`**         | 切换窗口                   |
| 6    | **`⌘ + 数字`*或*`⌘ + 方向键`**   | 切换标签页                 |
| 7    | **`⌘ + /`**                        | 显示光标位置               |
| 8    | **`⌘ + enter`**                    | 切换全屏                   |
| 9    | **`⌘ + r`或`⌃ + l`**         | 清屏                       |
| 10   | **`⌘ + Click`**                    | 可以打开文件，文件夹和链接    |

------

### 3.2 高级操作

- **分屏选择操作**

| 编号 | 命令         | 效果     |
| :--- | :----------- | :------- |
| 1    | **`⌘ + d`**  | 左右分屏 |
| 2    | **`⇧⌘ + d`** | 上下分屏 |

- **其他特殊功能**

| 编号 | 命令         | 效果                     |
| :--- | :----------- | :----------------------- |
| 1    | **`⌘ + ;`**  | 自动补全历史记录         |
| 2    | **`⇧⌘ + h`** | 自动补全剪贴板历史       |
| 3    | **`⌥⌘ + e`** | 查找所有来定位某个标签页 |
| 4    | **`⌥⌘ + b`** | 历史回放                 |
| 5    | **`⌘ + f`**  | 查找                     |

------

## 4. 工具完整配置

> 详细的配置，请参考我的 [`Github`](https://github.com/EscapeLife/DotFiles/blob/master/zshrc) 地址，这里不多说明。

- 配置共分为三段，分别为插件配置、环境配置、别名配置
- 最后附上一张我现在开发环境的配置样式截图，O(∩_∩)O 哈哈哈~



![最佳组合iTerm2+oh-my-zsh](https://www.escapelife.site/images/learn-iterm2-terminal-4.png)

------

