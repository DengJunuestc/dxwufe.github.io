---
title: "python-crawler-webdriver+xpath+pymysql"
subtitle: "「sql」- optimize"
layout: post
author: "Dxufe"
header-style: text
tags:
  - crawler
---

 1. 目标
		科创板开板以来，申报企业数量不断增加，目前已逾150家，拟通过爬虫获取所有申报企业的基本信息，例如审核状态、省份、行业、保荐机构、律师事务所、会计师事务所、保荐人、签字会计师、签字律师等信息。
 2. 网站分析
 		打开http://kcb.sse.com.cn/renewal/ 进入上交所科创板股票发行上市审核项目动态页面，可以看到页面已有比较清晰格式化的信息展示，包括全称、审核状态、行业、保荐机构、律师事务所、会计师事务所、更新日期、受理日期等信息。![在这里插入图片描述](https://img-blog.csdnimg.cn/20190816164650504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
 		翻页页面url不变，右键网页源代码中也无上述表格中的相关信息，故尝试通过requests获取html不能得到上述表格中相关信息，所以拟采取 selenium 中的webdriver模拟浏览器进行访问
(webdriver安装可参考https://blog.csdn.net/Alisen39/article/details/82930220)。
```
from selenium import webdriver
from lxml import etree
browser = webdriver.Chrome()
browser.get('http://kcb.sse.com.cn/renewal/') # page_source获取网页的源代码，然后可以使用正则表达式，css，xpath，bs4等来解析网页
html = etree.HTML(browser.page_source)  #得到html
```
 3. 页面解析元素提取
 		在F12中通过定位相关字段，右键查看xpath获取目标元素的路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190816165634416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
定义函数 pagedataget(html)，以提取每一页的关键信息，其中部分字段存在换行等情况，不能在一个路径下提取完整，需要加以判断整合。完整代码如下。

```
'''获取每页的数据'''
def pagedataget(html):
    items = html.xpath('//*[@id="dataList1_container"]/tbody/tr')
    num1=[];issuer_ful1=[];status1=[];province1=[];csrc_ind1=[];compinfourl=[];
    sponsor_org1=[];lawfirm1=[];accountfirm1=[];update1=[];acceptdate1=[];
    issuer_sec1=[];financing_amount1=[];sponsor1=[];accountant1=[];
    lawyer1=[];assessor_org1=[];assessor1=[]
    for item in items :
        num=item.xpath('./td[@class="td_no_break text-align-center"]/text()')
        if num:
            num1.append(num)
        issuer_ful=item.xpath('./td[@class="td_break_word_7 "]/a/text()[1]')
        b=item.xpath('./td[@class="td_break_word_7 "]/a/text()[2]')
        if issuer_ful and b:
            issuer_ful=issuer_ful[0]+b[0]
            issuer_ful1.append(issuer_ful)    
        else:
            if issuer_ful:
                issuer_ful=issuer_ful[0][0:]
                issuer_ful1.append(issuer_ful)    
        url=item.xpath('./td[@class="td_break_word_7 "]/a/@href')
        if url:
            compinfourl.append('http://kcb.sse.com.cn'+url[0])
        status=item.xpath('./td[@class="td_break_word_2 "]/a/text()[1]')
        if status:
            status1.append(status[0])
        province=item.xpath('./td[@class="td_no_break pd_style "]/a/text()[1]')
        if province:
            province1.append(province[0])
        csrc_ind=item.xpath('./td[@class="td_break_word_3 "]/a/text()[1]')
        b=item.xpath('./td[@class="td_break_word_3 "]/a/text()[2]')
        if csrc_ind and b:
            csrc_ind=csrc_ind[0]+b[0]
            csrc_ind1.append(csrc_ind)
        else:
            if csrc_ind:
                csrc_ind1.append(csrc_ind[0])
        sponsor_org=item.xpath('./td[@class="td_break_word_9 "]/a/text()[1]')
        b=item.xpath('./td[@class="td_break_word_9 "]/a/text()[2]')
        if sponsor_org and b:
            sponsor_org=sponsor_org[0]+b[0]     
            sponsor_org1.append(sponsor_org)
        else:
            if sponsor_org:
                if len(sponsor_org)==2:
                    sponsor_org1.append(sponsor_org[0]+','+sponsor_org[1])
                else:
                    sponsor_org1.append(sponsor_org[0])
        lawfirm=item.xpath('./td[@class="td_break_word_8 "]/a/text()[1]')
        b=item.xpath('./td[@class="td_break_word_8 "]/a/text()[2]')
        if lawfirm and b:
            lawfirm=lawfirm[0]+b[0]
            lawfirm1.append(lawfirm)
        else:
            if b:
                lawfirm1.append(lawfirm[0])
        accountfirm=item.xpath('./td[@class="td_break_word "]/a/text()[1]')
        b=item.xpath('./td[@class="td_break_word "]/a/text()[2]')
        if accountfirm and b:
            accountfirm=accountfirm[0]+b[0]  
            accountfirm1.append(accountfirm)
        else:
            if accountfirm:
                accountfirm1.append(accountfirm[0])
        date=item.xpath('./td[@class="td_no_break "]/text()')
        if date:
            update1.append(date[0])
            acceptdate1.append(date[1])
        if issuer_ful:
            (issuer_sec,financing_amount,sponsor,accountant,lawyer,assessor_org,assessor)=compdataget(str(issuer_ful))    #调用公司数据获取函数得到额外的信息
            issuer_sec1.append(issuer_sec)
            financing_amount1.append(financing_amount)
            sponsor1.append(sponsor)
            accountant1.append(accountant)
            lawyer1.append(lawyer)
            assessor_org1.append(assessor_org)
            assessor1.append(assessor)
    KCBINFO=pd.concat([DataFrame(issuer_ful1),DataFrame(status1),DataFrame(province1),
                       DataFrame(csrc_ind1), DataFrame(sponsor_org1),DataFrame(lawfirm1),
                       DataFrame(accountfirm1),DataFrame(update1),DataFrame(acceptdate1),
                       DataFrame(compinfourl),DataFrame(issuer_sec1),DataFrame(financing_amount1),
                       DataFrame(sponsor1),DataFrame(accountant1),DataFrame(lawyer1),
                       DataFrame(assessor_org1),DataFrame(assessor1)],axis=1) #信息拼接
    KCBINFO.columns=['issuer_ful','status','province','csrc_ind','sponsor_org','lawfirm','accountfirm','update','acceptdate','compinfourl',
                     'issuer_sec','financing_amount','sponsor','accountant','lawyer','assessor_org','assessor']
    return KCBINFO
```
 4. 公司详情页信息提取
		除了页面的基本信息外，还需要访问公司详情页面获取一些额外信息,可以通过访问pagedataget函数中提取到的url来实现这些进一步信息的提取。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190816170635549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
因此需要定义针对某家公司的这些额外信息进行提取的函数，传入参数为公司名称，此处通过browser.find_element_by_partial_link_text模拟浏览器点击公司名称进入详情页面，而不通过访问url进入公司页面，完整代码如下

```
def compdataget(issuer_ful):
    aa = browser.find_element_by_partial_link_text(issuer_ful[0:4])   #从公司名称关键字找到链接点进去
    aa.click()   #点击公司进入该公司页
    window=browser.window_handles 
    browser.switch_to_window(window[1]) #浏览器切换到当前公司的页面
    html = etree.HTML(browser.page_source) #获取该公司的数据
    issuer_sec=str(html.xpath('//*[@id="issuer_sec"]/text()')) #得到发行人简称
    financing_amount=str(html.xpath('//*[@id="type"]/text()')) #得到融资金额
    sponsor=str(html.xpath('//*[@id="sponsor"]/text()')) #得到保荐人
    accountant=str(html.xpath('//*[@id="accountant"]/text()')) #得到签字会计师
    lawyer=str(html.xpath('//*[@id="lawyer"]/text()')) #得到签字律师
    assessor_org=html.xpath('//*[@id="assessor_org"]/a/text()') #得到评估机构
    if len(assessor_org)==0:
        assessor_org='-'
    if len(assessor_org)==2:
        assessor_org=assessor_org[0]+'、'+assessor_org[1]
    else:
        if len(assessor_org)==1:
            assessor_org=assessor_org[0]
    assessor=html.xpath('//*[@id="assessor"]/text()') #得到签字评估师
    if len(assessor)==2:
        assessor=assessor[0]+'、'+assessor[1]
    else:
        if len(assessor)==1:
            assessor=assessor[0]
    browser.close()
    browser.switch_to_window(window[0])      #浏览器切换回主页面
    return issuer_sec,financing_amount,sponsor,accountant,lawyer,assessor_org,assessor

```
在定义好每一页的数据提取函数和每个公司的数据提取函数后，因为切换页面时，url不变，因此也需要通过模拟浏览器点击“下一页”进行换页操作，主函数如下

```
if __name__ == '__main__':
    import pandas as pd 
	from pandas import DataFrame
	import time
	import pymysql
	from sqlalchemy import create_engine
    start = time.time()
    browser = webdriver.Chrome()
    browser.get('http://kcb.sse.com.cn/renewal/')
    html = etree.HTML(browser.page_source)
    KCBINFO=pagedataget(html)     #第一页的数据
    '''获得页数'''
    pagenum=html.xpath('//*[@id="dataList1_container_pagination"]/div/span[@class="paging_input"]/text()[1]')[0][2] 
    for i in range(1,int(pagenum)):
        '''翻页'''
        a = browser.find_element_by_link_text('下一页') 
        a.click()
        '''该页的数据'''
        html = etree.HTML(browser.page_source)
        KCBINFO=pd.concat([KCBINFO,pagedataget(html)])   
    KCBINFO.to_csv('kcbinfo.csv',sep=',',encoding='utf-8',index=None)
    end = time.time()
    print ('一共有',len(KCBINFO),'家公司，'+'运行时间为',(end-start)/60,'分钟')
```
爬取得到的信息如下，若遇到爬取得到的信息仍有格式不规范存在冗余的信息，可继续进行清洗。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190816172316791.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
 5. 入库
 	将爬取得到的信息自动建表入到本地mysql库中，可以选择先在本地mysql中建好表再插入，或者直接在程序中建表写入。本次采用后者，自动建表入库，由于不同时间爬取得到的信息不一样，所以表名与时间挂钩以示区分，建表后通过sqlalchemy 将data导入到mysql中，定义save_to_db函数如下。
```
def save_to_db(data):
    conn = pymysql.connect(host='localhost', port=3306, user='root', password='xxx', db='xxx', charset="utf8") 
    cursor = conn.cursor()
    '''建表'''
    tablename='sse_kcbinfo_'+time.strftime("%Y_%m_%d_%H_%M_%S")
    create_sql="""CREATE TABLE """+tablename+""" (`id` int(11) NOT NULL AUTO_INCREMENT,
    `issuer_ful` varchar(255) DEFAULT NULL COMMENT '发行人全称',
    `status` varchar(255) DEFAULT NULL COMMENT '审核状态',
    `province` varchar(255) DEFAULT NULL COMMENT '省份',
    `csrc_ind` varchar(255) DEFAULT NULL COMMENT '证监会行业',
    `sponsor_org` varchar(255) DEFAULT NULL COMMENT '保荐机构',
    `lawfirm` varchar(255) DEFAULT NULL COMMENT '律师事务所',
    `accountfirm` varchar(255) DEFAULT NULL COMMENT '会计师事务所',
    `update` varchar(255) DEFAULT NULL COMMENT '更新日期',
    `acceptdate` varchar(255) DEFAULT NULL COMMENT '受理日期',
    `compinfourl` varchar(255) DEFAULT NULL COMMENT '链接',
    `issuer_sec` varchar(255) DEFAULT NULL COMMENT '公司简称',
    `financing_amount` varchar(255) DEFAULT NULL COMMENT '融资额（亿元）',
    `sponsor` varchar(255) DEFAULT NULL COMMENT '保荐人',
    `accountant` varchar(255) DEFAULT NULL COMMENT '签字会计师',
    `lawyer` varchar(255) DEFAULT NULL COMMENT '签字律师',
    `assessor_org` varchar(255) DEFAULT NULL COMMENT '资产评估机构',
    `assessor` varchar(255) DEFAULT NULL COMMENT '签字资产评估师',
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=153 DEFAULT CHARSET=utf8;"""
    cursor.execute(create_sql)
    '''写入数据'''
    try:
        connect = create_engine('mysql+pymysql://root:password@localhost:3306/name?charset=utf8')#password为本地mysql密码，name为库名
        pd.io.sql.to_sql(data,tablename, connect, schema=name, if_exists='append',index=None)  
        print('save to db success!')
    except:
        print('save to db failure!')
    cursor.close()  
```
执行save_to_db（KCBINFO）后，通过Navicat查看本地mysql中的数据情况，发现确有新增表及数据，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190816173239400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190816173354402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTExMjkx,size_16,color_FFFFFF,t_70)
说明自动建表入库成功