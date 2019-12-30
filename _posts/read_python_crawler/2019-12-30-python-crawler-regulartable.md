---
title: "python-crawler-read_html-vs-requests+lxml+xpath for regular table"
subtitle: "「crawler」- 规范化的表格数据"
layout: post
author: "Dxufe"
header-style: text
tags:
  - crawler
---  
为爬取网页表格数据，较容想到的常规方式是通过**requests请求以及lxml解析定位**获取元素，此外还可以通过pandas库的**read_html**直接获取表格数据，得到的将是目标网页所有table表格的list集合。那么后者看似简单的方式是不是在时间上更有效率呢，在此做一个对比分析。

 1. 目标网页  
 		以证监会官网披露的IPO表格为例，见链接[证监会IPO申报披露信息统计](http://eid.csrc.gov.cn/ipo/checkClick.action?choice=info#)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190821150235780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
共198页，每页信息二十条。
 2. requests请求+lxml解析+xpath定位  
 

```
from lxml import etree
import pandas as pd 
from pandas import DataFrame
import time
import requests
import random
from sqlalchemy import create_engine


def pagedataget(html):
    items=html.xpath('//*[@class="timeborder"]')
    compname=[];publish_date=[];ssdbk=[];pubtype=[];pubinfo=[]
    for item in items :
        name=str(item.xpath('./td/text()')[0]).replace('\r','').replace('\n','').replace('\t','')
        name=''.join(name.split())  
        if name:
            compname.append(name)
        date=str(item.xpath('./td/text()')[1]).replace('\r','').replace('\n','').replace('\t','')
        publish_date.append(date)
        sb=str(item.xpath('./td/text()')[2]).replace('\r','').replace('\n','').replace('\t','')
        ssdbk.append(sb)
        pt=str(item.xpath('./td/text()')[3]).replace('\r','').replace('\n','').replace('\t','')
        pubtype.append(pt)
        pinf=item.xpath('./td/a/text()')
        if len(pinf)==2:
            pinf=str(pinf[0]).replace('\r','').replace('\n','').replace('\t','')+'、'+str(pinf[1]).replace('\r','').replace('\n','').replace('\t','')
        else:
            pinf=str(pinf[0]).replace('\r','').replace('\n','').replace('\t','')
        pubinfo.append(pinf)       
    ipopub=pd.concat([DataFrame(compname),DataFrame(publish_date),DataFrame(ssdbk),DataFrame(pubtype),DataFrame(pubinfo)],axis=1)
    ipopub.columns=['compname','publish_date','ssdbk','pubtype','pubinfo']
    return ipopub
if __name__ == '__main__':
    start = time.time()
    user_agent_list = ["Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36",
                    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36",
                    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:60.0) Gecko/20100101 Firefox/61.0",
                    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36",
                    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36"]
    header={'User-Agent':'Mozilla/5.0'}
    ipopub=DataFrame(data=None,columns=['compname','publish_date','ssdbk','pubtype','pubinfo'])
    for i in range(1,199):
        header['User-Agent'] = random.choice(user_agent_list)
        pageurl='http://eid.csrc.gov.cn/ipo/infoDisplay.action?pageNo='+str(i)+'&temp=&temp1=&blockType=byTime'
        header['User-Agent'] = random.choice(user_agent_list)
        tx=requests.get(pageurl,headers=header)
        tx.encoding=tx.apparent_encoding
        html = etree.HTML(tx.text)
        ipopub=pd.concat([ipopub,pagedataget(html)])  
    ipopub.index=range(0len(ipopub)+1)
    '''写入数据'''
    tablename='csrc_ipo_pub'
    try:
        connect = create_engine('mysql+pymysql://root:password@localhost:3306/crawler?charset=utf8')
        pd.io.sql.to_sql(ipopub,tablename, connect, schema='crawler', if_exists='replace')  
        print('save to db success!')
    except:
        print('save to db failure!')  
    end = time.time()
    print ('一共有',len(ipopub),'家公司，'+'运行时间为',(end-start)/60,'分钟')
```
![结果显示共用158秒。](https://img-blog.csdnimg.cn/20190821151442283.png)
 3. pd.read_html  
 

```
import time
start = time.time()
import pandas as pd
from pandas import DataFrame
pubtable=DataFrame(data=None,columns=['compname','publish_date','ssdbk','pubtype','pubinfo'])
for i in range(1,199): 
    url ='http://eid.csrc.gov.cn/ipo/infoDisplay.action?pageNo='+str(i)+'&temp=&temp1=&blockType=byTime'
    data = pd.read_html(url,header=0,encoding='utf-8')[2] 
    pub = data.iloc[0:len(data)-1,0:-1]
    pub.columns=['compname','publish_date','ssdbk','pubtype','pubinfo']
    pubtable=pd.concat([pubtable,pub])  
end = time.time()
print ('一共有',len(pubtable),'家公司，'+'运行时间为',(end-start)/60,'分钟')
```
![结果显示共用119秒。](https://img-blog.csdnimg.cn/2019082115214371.png)
可见直接通过read_html获取表格数据速度更快，省了40秒。

 4. read_html的参数如下，io可为目标网址，match 为欲匹配的正则表达式，flavor为解析器，header为指定行标题，encoding为读入的编码格式，正确设置才能正确提取表格，否则会出现乱码。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190821152455716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)