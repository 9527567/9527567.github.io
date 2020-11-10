---
title: on my zsh以及vimplus配置符号字体的问题
date: 2020-11-01 19:19:59
tags:
	-tecnology
	-fonts
---

今天在安装zsh和vimplus的过程中符号一直未能正常显示，经历波折后最终找到原因，特此记录。

##### 安装zsh和vimplus的主要流程

1. 安装on my zsh和vimplus
2. Linux和Windows分别安装powerline字体
3. 设置Windows终端字体为Droid Sans Mono Nerd Font

问题出在最后的符号显示上，网上的解决方案都是安装powerline字体，但是此方法并不起作用。

**应该在https://github.com/ryanoasis/nerd-fonts 下载Windows专用字体Droid Sans Mono Nerd Font Complete Mono Windows Compatible.otf而非Droid Sans Mono Nerd Font Complete.otf，而后者是绝大部分网络上所提供的版本。**



