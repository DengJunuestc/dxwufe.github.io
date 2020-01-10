---
title: "neo4j-desktop安装使用系列问题-数据导入neo4j-admin import以及数据规范"
subtitle: "「neo4j」-数据导入 "
layout: post
author: "Dxufe"
header-style: text
tags:
  - neo4j
---
1. neo4j各种数据导入方式对比  
![avatar](/img/neo4j/neo4j-dataimport-01.png)  
其中属官方import方式最快，在此进行import方式的试验。  
2. 修改项目bin目录下的ps1文件  
![avatar](/img/neo4j/neo4j-dataimport-02.png)  
![avatar](/img/neo4j/neo4j-dataimport-03.png)  
将四个ps1文件的import moudule修改为该项目的bin目录，否则的话在cmd中运行import命令会提示无效。  
 ![avatar](/img/neo4j/neo4j-dataimport-06.png)  
3. 确保处于项目关闭状态-cmd 确保```data\database\graph.db```文件夹下没有文件（初次没有该文件夹，导入会自动创建），确保neo4j.conf中的importcsv相关的设定。进入该项目的bin目录  
4. 运行导入命令指定好分隔符，每个数据文件包括headers和数据两个文件，以etl工具中以bulk import 方式生成的```neo4j-admin-import-params```文件中的命令为参考，示例如下：  
  ```neo4j-admin import --mode=csv --database=graph.db --ignore-duplicate-nodes=true --ignore-missing-nodes=true --nodes "C:\Users\ADMINI~1\AppData\Local\Temp\csv-001\graphtest\NODE_graphtest.address_0ad8b883-2945-41d8-bd5e-a6fd14f32215_headers.csv,C:\Users\ADMINI~1\AppData\Local\Temp\csv-001\graphtest\NODE_graphtest.address_0ad8b883-2945-41d8-bd5e-a6fd14f32215.csv" --delimiter , --array-delimiter ; --id-type STRING --multiline-fields true```  
![avatar](/img/neo4j/neo4j-dataimport-04.png)  
![avatar](/img/neo4j/neo4j-dataimport-05.png)   
导入成功，十分迅速。  
5. 关于csv 的数据规范  
   **若使用etl工具生成的csv,则表头与数据分开成了两个文件，在写命令的时候需要两个文件都写**  
  ```--nodes "C:\Users\ADMINI~1\AppData\Local\Temp\csv-001\graphtest\NODE_graphtest.address_0ad8b883-2945-41d8-bd5e-a6fd14f32215_headers.csv,C:\Users\ADMINI~1\AppData\Local\Temp\csv-001\graphtest\NODE_graphtest.address_0ad8b883-2945-41d8-bd5e-a6fd14f32215.csv"```  
  其中实体headers表的命名规范为:  
  ```:ID,attr1name:attr1type,attr2name:attr2type,...,:LABEL```  
  其中关系headers表的命名规范为:  
  ```:START_ID,:END_ID,:TYPE,attr1name:attr1type,attr2name:attr2type```    
  其中属性类型如string不是必要的。  
  **若自定义表，可无需将表头分开**  
  ```neo4j-admin import --mode=csv --database=graph.db --ignore-duplicate-nodes=true --ignore-missing-nodes=true --nodes:company C:\Users\Administrator\Desktop\company.csv --relationships:invest C:\Users\Administrator\Desktop\company_invest_company.csv ```  
  其中实体headers表的命名规范为,不需要指定label:  
  ```:ID,attr1name,attr2name```  
  其中关系headers表的命名规范为，不需要指定type:  
  ```:START_ID,:END_ID,attr1name,attr2name```    

  