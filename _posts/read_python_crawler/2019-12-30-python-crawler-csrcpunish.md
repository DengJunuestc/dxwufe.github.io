---
title: "python-crawler-requests+xpath"
subtitle: "「crawler」- 证监会行政处罚决定书"
layout: post
author: "Dxufe"
header-style: text
tags:
  - crawler
---
 1. 目标  
 		证监会行政处罚决定书为证监会发布的对证券期货市场违法违规主体进行行政处罚的相关文书。可以将这些文书按照所涉及的行为进行分类，如涉嫌财务造假、市场操纵、未尽勤勉职责等，作为相应标签下的黑样本进一步进行其他研究。本文将尝试对这些行政处罚决定书进行爬取以格式化存储。
 2. 网页分析  
  		进入证监会网站，信息披露，按体裁文种查看，点击行政处罚决定，可以查看到下述页面，初步分析可用requests直接取得页面源码再进行元素提取，但发现如此得到的源码中仅含左侧的标题栏，不含右侧的具体内容等信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817185144175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
右键发现，此页面不仅包含网页源代码，还存在一个框架源代码的选项
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817185552656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
即网页源代码只包含左侧的标题栏目，而框架源代码中才有具体的行政处罚决定书等内容，点击打开框架源代码，发现出现了一个新的链接。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817185809446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
单独进入此链接,发现此时进入了仅包含行政处罚决定信息发布的页面，此时便可通过，requests得到网页源代码再进行关键信息的提取
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817190059704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
	单击下一页，发现链接在变，在第一页的链接后面直接加了一个页面数字，因此可通过变换链接进行换页而不需要模拟浏览器进行点击下一页操作。
 3. 爬取思路  
	通过以上分析得出以下爬虫思路：在每一页通过xpath提取页面的文书名称、发文日期、文号以及文书内容url，然后访问该url得到文书的文本内容；然后更换页面url进行换页操作，更换页面url需要获取页面总页数，但是通过requests得到的首页源码中不含有总页数信息，可能是有些信息隐藏了，所以对首页采用webdrvier访问得到页面html获取包含总页数在内的关键信息。然后更换页面url后直接通过requests进行访问获取页面源码进行信息提取。
 4. 进行爬取  
 实际过程中会发现几个页面提取之后会报connection error的错，这是爬虫被限制了，可以通过变化headers 的形式进行访问，每次随机选择一个headers。
 
 5. 代码  
 ps:每个文书的文本内容提取还存在不足，包括未清洗干净未提取完全等问题。
 

```
from selenium import webdriver
from lxml import etree
import pandas as pd 
from pandas import DataFrame
import time
import requests
import random

def pagedataget(html):
    items=html.xpath('//*[@class="row"]')
    title=[];publish_date=[];docnum=[];docurl=[];text=[]
    for item in items :
        name=item.xpath('./li[@class="mc"]/div/a/text()')[0]
        if name:
            title.append(name)
        date=item.xpath('./li[@class="fbrq"]/text()')[0]
        publish_date.append(date)
        num=item.xpath('./li[@class="wh"]/text()')[0]
        docnum.append(num)
        url='http://www.csrc.gov.cn/pub/zjhpublic/'+item.xpath('./li[@class="mc"]/div/a/@href')[0][6:]
        docurl.append(url)       
        text.append(doctext(url,header))  csrcxzcf=pd.concat([DataFrame(title),DataFrame(publish_date),DataFrame(docnum),DataFrame(docurl),DataFrame(text)],axis=1)
    csrcxzcf.columns=['title','publish_date','docnum','docurl','text']
    return csrcxzcf

def doctext(url,header):
    wb=requests.get(url,headers=header)
    wb.encoding='utf-8'
    text = etree.HTML(wb.text)
    a=str(text.xpath('//*[@id="ContentRegion"]/div/p/span/text()'))
    if len(a)==2:
        a=str(text.xpath('//*[@id="ContentRegion"]/div/div/p/span/text()'))
    a=a.replace("'","").replace(",","")
    a=''.join(a.split())    
    a=a.replace("。","。\n  ")    
    return a            
    
if __name__ == '__main__':
    start = time.time()
    browser = webdriver.Chrome()
    browser.get('http://www.csrc.gov.cn/pub/zjhpublic/3300/3313/index_7401.htm')  
    html = etree.HTML(browser.page_source) #获取网页
    user_agent_list = ["Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36",
                    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36",
                    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:60.0) Gecko/20100101 Firefox/61.0",
                    "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36",
                    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36",
                    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36",
                    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0)",
                    "Mozilla/5.0 (Macintosh; U; PPC Mac OS X 10.5; en-US; rv:1.9.2.15) Gecko/20110303 Firefox/3.6.15",
                    ]
    header={'User-Agent':'Mozilla/5.0'}
    header['User-Agent'] = random.choice(user_agent_list)
    csrcxzcf=pagedataget(html)     #第一页的数据
    pagenum=int(html.xpath('//*[@title="尾页"]/a/text()')[0])
    for i in range(1,pagenum):
        pageurl='http://www.csrc.gov.cn/pub/zjhpublic/3300/3313/index_7401'+'_'+str(i)+'.htm'
        header['User-Agent'] = random.choice(user_agent_list)
        tx=requests.get(pageurl,headers=header)
        tx.encoding='utf-8'
        html = etree.HTML(tx.text)
        csrcxzcf=pd.concat([csrcxzcf,pagedataget(html)])    
    end = time.time()
    print ('运行时间：', end-start)
```

 6. 结果

涉及页数较多，爬取需要一定时间，大约五六分钟，结果如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190817194228997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)

	


 
