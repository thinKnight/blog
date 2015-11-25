title: Python网络爬虫（一）
date: 2014-10-13 21:48:33
tags: Python
---
null
<!-- more -->

## 一、遍历网站的全部页面

### 思路
要遍历一个网站的全部页面,要做的就是先打开目标网站的源码,从中提取所有的URL,然后再逐个遍历,并保存已处理过的URL。  

#### 1.提取URL
从一堆HTML中提取可用的URL是一件轻松的事,处理的方法也有很多。  

- 正则表达式提取  
- BeautifulSoup提取
- 自带库sgmlib中的SGMLParser类  

这里就试试第三种方法。  

#### 2.存储URL
提取出URL后,分开存储刚刚提取出来的URL和已经处理过的URL就是接下来要解决的问题。  

一开始想过存在列表里面,但是从中提取和pop出URL的顺序又成了问题,所以这里采用Python自带的**队列**数据结构。  

然后处理完的URL就直接存入文件,并且计数,即为已经访问到的页面数量。  

#### 3.实现函数 
接下来的任务就是运行函数来处理URL了,但是我现在用的只是最笨的**递归**,效率低下不说,对内存也是一个很大的考验。所以之后会通过多线程编程来解决这个问题,把URL放入内存池中,规定每次允许运行的线程数,这样就能在一定程度上提升效率和速度了。  


#### 4.具体代码
```Python
# -*- coding: UTF-8 -*-
# 搜寻现成的爬虫代码，弄明白怎样遍历一个网站的全部页面，编码实现：
# 能够遍历一个网站的大部分页面，保存输出可遍历页面的URL，并统计访问到的页面数量。
import re
import Queue
import urllib2
from sgmllib import SGMLParser

url_queue = Queue.Queue(0)
url_num = 0

class find_url(SGMLParser):
	"""docstring for find_url
		store the urls into url_new

	"""
	def __init__(self):
		SGMLParser.__init__(self)
		self.url_new = []

	def start_a(self, attrs):
		href = [v for k, v in attrs if k=='href'] 
		
		if re.match(r'^https?:/{2}\w.+$', "".join(href)):
			self.url_new.extend(href)

def open_url():
	url = url + 1

	url_given = url_queue.get()

	url_traversed.write(url_given + "\n") 

	content = urllib2.urlopen(url_given).read()	
	result  = find_url()
	result.feed(content) 
	for urls in result.url_new:
		# print i                 
		url_queue.put(urls)

	while not url_queue.empty():
		open_url()


if __name__ == '__main__':

	url = "http://movie.douban.com"

	url_traversed = open('URLSTORE.txt', 'w')

	url_queue.put(url)

	open_url()

	url_traversed.closed()

	print "The number of the traversed URL is %d" % url_num
```  

## 二、向百度提交搜索
#### 1.提交搜索
我现在还是用的最笨的方法,即直接打开包含需要搜索内容的URL,就能得到搜索页面的源码。  

用POST和GET提交的方法下次再用。  

#### 2.处理结果  
用BeautifulSoup查找<div>标签间的内容,但是这个还是只能大概地过滤,并不能很精准地返回搜索内容。  

#### 3.待解问题
1. 用POST和GET方法提交搜索。  
2. 细致地处理返回的搜索结果。  
3. 遍历所有的搜索结果。  

```Python
# -*-coding: UTF-8 -*-
# 2.用百度设置内的高级搜索功能，在指定网站中搜索URL中包含？的结果。编程实现：
# 	自动向百度提交搜索，在指在指定网站中搜索URL中包含？的结果，提取百度搜索结果并输出到文件。
#   例如，搜索nuist.edu.cn，其实就是想百度提交搜索字符串site:(nuist.edu.cn) ?

# http://www.baidu.com/s?wd=
import sys
import urllib2
from bs4 import BeautifulSoup

reload(sys)   
sys.setdefaultencoding('utf8')  

def search_baidu():
    url = urllib2.urlopen("http://www.baidu.com/s?wd=site:(nuist.edu.cn)%20?")

  
    urltmp = url.read()

    # urltmp = urltmp.decode("UTF-8").encode("UTF-8")

    soup = BeautifulSoup(urltmp)

    res = soup.find(name='div').getText('\n')

    ss = open('ss.txt', 'w')
    ss.write(res)
    ss.close()

search_baidu()
```


