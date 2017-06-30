---
layout: post
title: Python爬虫之Github项目爬取
categories:
- 随笔
---

<div class="message">
	应该有好几个月没有写博客了，原因好几个吧，一是由于课题的原因，二是自己状态的原因，写在本月末，激情的自己又回来了。代码已上传至Github <a target="_blank" href="https://github.com/conghaoyuan/GithubCrawler"><font color="red">GithubCrawler</font></a>
</div>
## 1、题目介绍

	题目：爬取2017年1月1日到如今，github上，最流行的20个（100个？）大数据相关的项目。要求：请自定义/量化“最流行”


## 2、需求分析

	爬取项目为github上开源项目

## 3、要求

*	时间为2017年1月1日至今项目
*	自定义爬取个数（20个）
*	自定义爬取内容（大数据相关）
*	自定义项目排序（最流行、复刻/克隆次数对多、最近发布等）

## 4、项目流程

从github上爬取处理的流程大致为：根据定义的量化信息爬取页面、解析html页面取到相关内容信息、进行内容存储。

<img width="600px" src="/images/170630/crawler_1.png"/>

## 5、爬虫流程图

<img width="600px" src="/images/170630/crawler_2.png"/>

## 6、代码设计

编写爬虫类，量化信息可作为变量输入
<img width="600px" src="/images/170630/crawler_class.png"/>
<img width="600px" src="/images/170630/crawler_3.png"/>

发送URL请求，获取页面方法
<img width="600px" src="/images/170630/crawler_4.png"/>

解析页面HTML方法
<img width="600px" src="/images/170630/crawler_5.png"/>

内容存储
<img width="600px" src="/images/170630/crawler_6.png"/>

## 7、完整代码

	# -*- coding:utf-8 -*-
    from re import search
    __author__ = 'Conghaoyuan'
    import urllib
    import urllib2
    import re
    import thread
    import time
    import sys
    defaultencoding = 'utf-8'
    if sys.getdefaultencoding() != defaultencoding:
        reload(sys)
        sys.setdefaultencoding(defaultencoding)
     
    #github爬虫类
    class GHCrawler:
    #初始化方法，定义一些变量
    def __init__(self):
        self.pageIndex = 1
        self.user_agent = 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'
        #初始化headers
        self.headers = { 'User-Agent' : self.user_agent }
        #存放段子的变量，每一个元素是每一页的段子们
        self.stories = []
        #存放程序是否继续运行的变量
        self.enable = False
    #传入某一页的索引获得页面代码
    def getPage(self,pageIndex,sort_options,search_content):
        #print project_number,sort_options,search_content+"========="
        
        try:
            #url = 'http://www.qiushibaike.com/hot/page/' + str(pageIndex)
            #url = 'https://github.com/search?o=desc&q=big+data&s=stars&p='+str(pageIndex)
            url = 'https://github.com/search?'+str(sort_options)+'&q='+str(search_content)+'&p='+str(pageIndex)
            print url
            #构建请求的request
            request = urllib2.Request(url,headers = self.headers)
            #利用urlopen获取页面代码
            response = urllib2.urlopen(request)
            #将页面转化为UTF-8编码
            pageCode = response.read().decode('utf-8')
            return pageCode
        except urllib2.URLError, e:
            if hasattr(e,"reason"):
                print u"连接github失败,错误原因",e.reason
                return None

    #传入某一页代码，返回本页不带图片的段子列表
    def getPageItems(self,pageIndex,sort_options,search_content):
        #print project_number,sort_options,search_content+"===="
        pageCode = self.getPage(pageIndex,sort_options,search_content)
        #print pageCode
        if not pageCode:
            print "页面加载失败...."
            return None
        #reg = '<div.*?author">.*?<a.*?<img.*?>(.*?)</a>.*?<div.*?content">(.*?)<!--(.*?)-->.*?</div>(.*?)<div class="stats.*?class="number">(.*?)</i>'
        reg = '<a.*?v-align-middle">(.*?)</a>.*?<p.*?pr-4">(.*?)</p>.*?<p.*?Updated.*?>(.*?)</relative-time>.*?<div.*?pt-2">.*?<span.*?repo-language-color.*?</span>(.*?)</div>.*?<div.*?</svg>(.*?)</a>'
        pattern = re.compile(reg,re.S)
        items = re.findall(pattern,pageCode)
        #print items
        #用来存储每页的段子们
        pageStories = []
        #遍历正则表达式匹配的信息
        for item in items:
            #item[0]github项目名称，item[1]是项目简介，item[2]是发布时间,item[3]是语言，item[4]是星数量
            pageStories.append([item[0].strip(),item[1].strip(),item[2].strip(),item[3].strip(),item[4].strip()])
        return pageStories

    #加载并提取页面的内容，加入到列表中
    def loadPage(self,pageIndex,sort_options,search_content):
        #print project_number,sort_options,search_content
        pageStories = self.getPageItems(self.pageIndex,sort_options,search_content)
        for pageStorie in pageStories:
            self.stories.append(pageStorie)

    #取规定数量个数内容
    def getNumberStory(self,pageStories):
        f = file("data.txt","w+")
        for story in pageStories:
            print "项目名称："+story[0]+"\n"
            print "项目简介："+story[1]+"\n"
            print "发布时间："+story[2]+"\n"
            print "开发语言："+story[3]+"\n"
            print "流行度："+story[4]+"\n"
            print "\n"
            str = "项目名称："+story[0]+"\n项目简介："+story[1]+"\n发布时间："+story[2]+"\n开发语言："+story[3]+"\n流行度："+story[4]+"\n"
            f.writelines(str+"\n")
        f.close()
    
    #开始方法
    def start(self,page_number,sort_options,search_content):
        print u"正在读取github页面，搜索内容为："+search_content
        #print project_number,sort_options,search_content
        #使变量为True，程序可以正常运行
        self.enable = True
        #先加载一页内容
        while self.pageIndex <= page_number:
            self.loadPage(self.pageIndex,sort_options,search_content)
            self.pageIndex += 1
        self.getNumberStory(self.stories)
        
	spider = GHCrawler()

	def getSortOptions(argument):
	    switcher = {
	        0: "o=desc&s=stars",
	        1: "o=asc&s=stars",
	        2: "o=desc&s=forks",
	        3: "o=asc&s=forks",
	        4: "o=desc&s=updated",
	        5: "o=asc&s=updated",
	    }
	    return switcher.get(argument, "nothing")
	#自定义爬取个数
	project_number = 20
	page_number = project_number/10
	#自定义/量化（0：最流行，1：不流行，2：复刻/克隆最多，3：克隆最少，4：最近更新，5：近期最少更新）
	options = 0
	sort_options = getSortOptions(options)
	print sort_options
	#自定义搜索内容
	content = "big data"
	search_content = content.replace(" ", "+")
	spider.start(page_number,sort_options,search_content)


## 8、问题

Github做了反爬虫机制，如果定义的爬取项目数量较多，需要请求多个页面，由于URL请求次数过多，Github会返回由于多次请求错误。准备下一版做一个反爬虫机制来解决。

