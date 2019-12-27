---
title: "使用git克隆管理github项目"
subtitle: "「git」- 基本指令"
layout: post
author: "Dxufe"
header-style: text
tags:
  - git-operation
---

## git安装
下载并安装git  

## 拉取项目  
### 新建文件夹 project  
进入文件夹，右键，选择 git bash here    
输入git init 初始化  
### 配置本地仓库账号邮箱  
git config --global user.name "Your Name"  
git config --global user.email "Your email"  
### 配置ssh登录  
ssh-keygen -t rsa -C "your email"  
回车，设置密码，再次数输入密码  
前往C:\Users\Administrator\.ssh文件夹打开id_rsa.pub文件，将内容复制到粘贴板  
前往自己的github项目 clone or download中选择ssh，add new public key ,填写title，并将刚刚复制的粘贴板内容粘贴到key中，保存  
在终端键入 git clone 项目地址  即为clone or download中的地址

## git操作
### 查看项目状态  
cd 进入项目目录  键入 git status  
### 修改内容
直接在本地project文件夹下面的项目中进行修改，修改完后，git status 会显示哪些文件被修改  
### 将文件修改提交到本地暂存区
修改完毕后键入 git add .
### 提交当前修改备注 
键入 git commit -m "备注内容"
### 提交修改
键入 git push  
注意需要输入ssh登录的密码

