---
title: "使用less命令查看文件"
subtitle: "「linux」- less指令"
layout: post
author: "WynnZuo"
header-style: text
tags:
  - linux
---

### 适用场景
less指令适合查看大文件，或者只想查看文件中的某几行。
#### 打开文件
```
# 打开文件
less xx.log
# 定位到100行
less +100g xx.log
# 定位到最后一行
less +GG xx.log
# 定位到第100个字节的位置
less +100P xx.log
# 直接定位到100%的位置
less +100p xx.log
```
#### 移动
```
j        向前移动一行
k        向后移动一行
G        移动到最后一行
g        移动到第一行
ctrl+f   向前移动一屏
ctrl+b   向后移动一屏
ctrl+d   向前移动半屏
ctrl+u   向后移动半屏
```
#### 退出
```
q/ZZ  退出less命令
```
#### 搜索
```
/   使用一个模式进行搜索，并定位到下一个匹配的文本
n   向前查找下一个匹配的文本
N   向后查找前一个匹配的文本
```