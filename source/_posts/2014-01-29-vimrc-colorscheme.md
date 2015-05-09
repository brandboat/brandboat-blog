---
layout: post
title: "vim 設置 colorscheme"
date: 2014-01-29
comments: true
categories: vim
---

[**vim-railscasts-theme**][2]
 
- **``cd ~/.vim/colors``**
若無此資料夾，請自行新建

- **``vim railscasts.vim``**
複製 [**vim-railscasts-theme**][2] 中的 railscasts.vim 到裡頭

- **``vim ~/.vimrc``**
加入此行 
``colorscheme railscasts`` 
或者 
``colo railscasts``
來設置 colorscheme


![](http://sunaku.github.io/vim-256color-bce-xterm.png)<br/><br/>
(圖片來源自：<http://sunaku.github.io/vim-256color-bce-xterm.png>)
<br/><br/>
如果使用了``colorscheme``而在``tmux``開``vim``時出現每一行底色跟背景顏色不一致的情況，這是因為設置了``TERM=xterm-256color``而衍生出來的問題，解決辦法 ：  [disable Background Color Erase][3] 。

- 在``~/.vimrc``中加入

``set t_ut=``


[1]: https://github.com/tomasr/molokai
[2]: https://github.com/jpo/vim-railscasts-theme
[3]: http://sunaku.github.io/vim-256color-bce.html
