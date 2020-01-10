---
title: "neo4j-desktop安装使用系列问题-安装"
subtitle: "「neo4j-desktop」-安装 "
layout: post
author: "Dxufe"
header-style: text
tags:
  - neo4j
---
1. 安装包下载  
   下载链接 https://neo4j.com/download/neo4j-desktop/?edition=desktop&flavour=winstall64&release=1.2.4&offline=true ，填写邮箱姓名等信息进入下载页面，将注册码copy到粘贴板，安装完成后会要求输入。  
2. 执行安装  
   按提示进行安装，输入注册码进行注册激活，然后便可以正常使用  
3. 新建图谱  
   add graph、create a local graph。
4. 安装图应用  
   提供如neo4j browser,neo4j etl tool,neo4j bloom等工具，可对已建的图谱进行操作管理。安装后要在my project 中进行add application.更多应用参见 https://install.graphapp.io/ 。  
5. 安装应用失败  
   若安装ETL TOOL等工具的过程中，提示 ```A JavaScript error occurred in the main process```,安装失败，此时，可以，电脑左下角 win 按钮 搜索 %appdata%，进入路径后将 neo4j文件夹删除，再重新试夏是否可以安装。  
