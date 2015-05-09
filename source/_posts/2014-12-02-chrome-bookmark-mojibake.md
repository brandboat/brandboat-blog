---
layout: post
title: "Chrome 標籤亂碼"
description:
date: 2014-12-02
category: [Ubuntu, chrome]
---

Ubuntu 解決辦法如下：

`sudo vim /etc/fonts/conf.d/49-sansserif.conf`
內容如下
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<!--
If the font still has no generic name, add sans-serif
-->
  <match target="pattern">
    <test qual="all" name="family" compare="not_eq">
      <string>sans-serif</string>
    </test>
    <test qual="all" name="family" compare="not_eq">
      <string>serif</string>
    </test>
    <test qual="all" name="family" compare="not_eq">
      <string>monospace</string>
    </test>
    <edit name="family" mode="append_last">
      <string>sans-serif</string>
    </edit>
  </match>
</fontconfig>

```
將
```
<edit name="family" mode="append_last">
  <string>sans-serif</string>
</edit>
```

中的 `sans-serif` 改為你想要的中文字型，若不知道就填寫 `Ubuntu`。

- 其他關於字型可參考的文章 ： http://scar.simcz.tw/article/2014/04/22/fix-ubuntu-14-04-lts-zh-font-selector/
