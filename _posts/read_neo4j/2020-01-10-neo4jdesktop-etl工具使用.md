---
title: "neo4j-desktop安装使用系列问题-etl工具使用"
subtitle: "「neo4j-desktop」-etl工具"
layout: post
author: "Dxufe"
header-style: text
tags:
  - neo4j
---
1. **安装neo4j etl tool之后，在my project中打开**   
![avatar](/img/neo4j/neo4j-etl-01.png)  
2. **连接关系型数据库-mysql为例**  
输入连接名、主机地址、端口、选择数据类型、数据库名称，自动生成connectionurl,但该url不能正确连接，要将？好之后的内容手动改成```serverTimezone=GMT```，完成后test and save connection，进入下一步.  
![avatar](/img/neo4j/neo4j-etl-02.png) 
![avatar](/img/neo4j/neo4j-etl-03.png) 
3. **选择建好的数据库连接以及图谱项目，进行mapping**  
mapping操作是将关系型数据库中的数据进行实体和关系的抽象，形成一张本体网络。  
![avatar](/img/neo4j/neo4j-etl-04.png) 
4. **本体网络生成，属性编辑，关系名称修改**  
上一步mapping成功后，将自动生成本体网络，如图所示，可以对节点表和关系表进行编辑操作，包括修改属性类型，或者选择忽略某个属性，或者直接忽略某张表。  
需注意的是：  
neo4j etl tool在进行关系型数据库转换的过程中会将拥有一个主键的表识别成node，两个外键的表识别成 relations，多个外键的表，会识别成多对关系；所以要对mysql中的实体表、关系表设置好主键，外键。  
mapping结束后会在C:\Users\Administrator\AppData\Local\Temp\生成临时json文件，用以表示etl匹配过程，后续写csv也是根据这个json文件来进行。  
![avatar](/img/neo4j/neo4j-etl-05.png) 
5. **开始导入数据-项目未启动:bulk-import**  
在项目，此处是graphtest处于stop状态下，只有这一种数据导入方式可选，这执行的是neo4j-admin import,进行大批量的数据导入，这种方式是最快最有效的。   
![avatar](/img/neo4j/neo4j-etl-06.png) 
这会首先在 C:\Users\Administrator\AppData\Local\Temp\csv-001 目录下自动创建csv的临时文件以及neo4j-admin-import命令。  
其中，csv文件将数据分成了headers和data两部分，以符合neo4j的命名规范：  
![avatar](/img/neo4j/neo4j-etl-07.png) 
import命令参数文件是自动生成的import导数据命令，即将csv文件导入到目标图数据库：    
![avatar](/img/neo4j/neo4j-etl-08.png) 
6. **开始导入数据-项目启动中:load-csv**  
 首先启动项目，其次修改loadcsv的相关参数，否则会报错```Couldn't load the external resource at: file:```。在manage页面的setting中进行修改，首先将```dbms.directories.import=import```参数注释掉，这个参数的意思是导入的csv文件必须在项目路径的import文件夹下，然后再把参数```bindbms.security.allow_csv_import_from_file_urls=true```放开注释，即允许通过文件url地址来进行csv的导入，要这样设置后，以load-csv方式导入才能成功。详见下图：  
![avatar](/img/neo4j/neo4j-etl-loadcsv-01.png) 
![avatar](/img/neo4j/neo4j-etl-loadcsv-02.png) 
然后按照1-5步进行操作，在第五步中，选择导入方式为online Import Load CSV  
同样，首先会生成和5中描述一致的临时csv文件，以及导入csv的cypher语句，语句示例如下：  
![avatar](/img/neo4j/neo4j-etl-loadcsv-03.png)  
执行这些cypher语句将把csv文件导入到目标图数据库。
7. **注意csv文件中的非法字符**  
若报错指向临时csv文件```there's a field starting with a quote and whereas it ends that quote there seems to be characters in that field```.说明csv文件中有非法字符，需要在源数据库中进行清洗，从新进行连库mapping导数据操作，常见的非法字符如``` "",\ ```.
8. **关于中文编码问题**  
上述5、6中介绍的方式生成的csv文件为系统默认的编码非utf8,所以在目标图谱中会出现乱码。   
![avatar](/img/neo4j/neo4j-graph-01.png)  
解决方法为，将csv文件编码改成 utf-8，无BOM编码格式（如notepad），再执行neo4j import 中的导入命令，或cypher语句中的导入命令。例如改了csv文件编码后，将cypher文件中的语句copy至browser命令行运行导入：  
![avatar](/img/neo4j/neo4j-graph-02.png)  
然后可以发现中文正常显示：  
![avatar](/img/neo4j/neo4j-graph-03.png)


