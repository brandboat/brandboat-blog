---
layout: post
title: "bash redirect : &>"
comments: true
date: 2015-02-09
category: bash
---

```
unalias fs &> /dev/null
```
這指令是在看 hadoop 安裝指南 時注意到的，主要是第一次看到 &> 這樣的寫法感到好奇才特別去查。

之前曾經學過有關於 shell script 訊息導向(redirection)的部份，但看到的多是

```
// 1. stderr redirect to /dev/null
unalias fs 2> /dev/null 

// 2. stderr redirect to stdout
grep * 2>&1
```

這邊稍微解釋一下，1代表 standard output，2代表standard error。

而 &> 其實就是將 stderr 與 stdout 導向至檔案，其意思等同於

```
> file 2>&1
```

舉個例子，我們要將標題的指令中的所有訊息全部導向至 /dev/null 這個大黑洞，原本要打
```
unalias fs > /dev/null 2>&1
```
現在只要換成
```
unalias fs &> /dev/null
```
或者
```
unalias fs >& /dev/null
```
但這樣簡便的寫法只有在 bash 可以用，csh，tcsh，sh 則沒有。
