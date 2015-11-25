title: Python网络爬虫（二）
date: 2014-10-23 18:51:36
tags: Python
---
null
<!-- more -->

# 上文回顾
上次写到了**遍历网站的全部页面**和**向百度提交搜索**，但是其中还存在着许多的问题。  

## 解析HTML
在上次的方法中，由于以前都是用简单的正则表达式来解析HTML，所以为了尝鲜我就使用了BeautifulSoup和SGMLParser两种方法。但是经过使用下来发现还是BeautifulSoup好那么一点，而且官方的[文档](http://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#)也很详实，以后用起来也会更加方便。  

当然了，这两者在解析的过程中都有自己的局限性，所以还得配合正则表达式使用。  

## 循环与递归
由于上次处理的只是遍历一个页面的URL，所以总的来说工作量比较小，然后我就用了**递归**这种最笨的方法。  

但是显而易见，递归是一个非常耗内存的差方法，用递归写过输出斐波那契数列的人都知道，从第十几个数字后就开始慢的不行了，而且最近还听说某厂面试一个应届生的时候因为他用递归处理斐波那契就直接拒了他......所以还是不用的好。  

由于Python中自带队列数据结构，所以通过队列实现迭代循环是目前较为理想的方案。  

## 多线程
在爬虫程序中，当我提交了请求之后CPU需要等待网站相应后才能进一步计算，也就是需要等待`urllib2.urlopen()`得到相应之后才能read网页的内容，所以这就需要等待一段时间，所以为了提高爬虫的效率，就需要开启多线程进程抓取。

## 健壮
在爬虫运行的时候，如果因为被网站的防爬虫机制禁止了爬取行为，那就会导致整个爬虫程序的意外退出，所以就必须把`urllib2`的行为包起来。  

另外，如果同一个IP在短时间内对一个网站进行大量访问，可能会被网站的防爬虫措施制裁，比如豆瓣...所以为了避免爬虫挂掉，就得设置一个时间间隔，也就是让线程暂时阻塞，等时间到了之后再加入线程队列中。  

## Bloom Filter
在上次的遍历一个URL中的所有URL任务中，虽然一次能抓取到几千个URL，但是并不能保证这些URL都是不重复的，如果在这些URL中有环路的话，爬虫就会先入死循环中，所以对抓取到的URL进行去重就是一个要面临的问题。  

当需要处理的数据很少的时候，可以用`set`集合来解决，但是当数据量变大的时候，就得靠**Bloom Filter**（布隆过滤器）了。BF的算法不算非常复杂，不过好歹有现成的[轮子](https://github.com/jaybaird/python-bloomfilter/blob/master/pybloom/pybloom.py)，用起来也方便了许多。  

## Demo
```Python
>>> from pybloom import BloomFilter
>>> f = BloomFilter(capacity=10000, error_rate=0.001)
>>> for i in range_fn(0, f.capacity):
... _ = f.add(i)
...
>>> 0 in f
True
>>> f.capacity in f
False
>>> len(f) <= f.capacity
True
>>> (1.0 - (len(f) / float(f.capacity))) <= f.error_rate + 2e-18
True
>>> from pybloom import ScalableBloomFilter
>>> sbf = ScalableBloomFilter(mode=ScalableBloomFilter.SMALL_SET_GROWTH)
>>> count = 10000
>>> for i in range_fn(0, count):
... _ = sbf.add(i)
...
>>> sbf.capacity > count
True
>>> len(sbf) <= count
True
>>> (1.0 - (len(sbf) / float(count))) <= sbf.error_rate + 2e-18
True
```  

------------

# 新任务
>自动向百度提交搜索请求，搜索nuist.edu.cn中包含？的URL，从返回的结果页面中，提取每一个分页中的URL，并将结果写入一个文件中。**这次强调所有结果有多少页就爬取多少页**！

## 实现
```Python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import re
import time
import Queue
import urllib2
import threading 
from bs4 import BeautifulSoup
import sys
import urllib
from pybloom import BloomFilter
import time

# use Bloom Filter
bf = BloomFilter(1000000, 0.01)

# translate the default code
reload(sys)
sys.setdefaultencoding("utf-8")

# define a queue
url_wait = Queue.Queue(0)

class MyThread(threading.Thread):
    def __init__(self, url, num):
        threading.Thread.__init__(self)
        self.url = url
        # self.tnum = num
    def run(self):
        # traverse the whole url
        time.sleep(5)
        traverse(self.url)
        # print "This is thread-%d" % self.tnum

def find_nextpage(new_url):
    try:
        tmp = urllib2.urlopen(new_url)
        content = tmp.read()
    except:
        pass

    soup = BeautifulSoup(content)

    for link in soup.find_all(href=re.compile("rsv_page=1")):
        tmp_link = link.get('href')
        real_url = "http://www.baidu.com" + tmp_link
        return real_url  

def traverse(url):
    
    fp = open("all_url.txt", "a")

    url_wait.put(url)
     
    while not url_wait.empty():
        url = url_wait.get()
        if url not in bf:
            try:
                content = urllib2.urlopen(url).read() 
                soup = BeautifulSoup(content)                                     
                for urls in soup.find_all(href=re.compile("http")):                     
                    link = urls.get('href')
                    url_wait.put(link)
            except:
                pass

            bf.add(url)
            fp.write( url + '\n\n')   

    fp.close()  


def main():
    num = 0
    fp = open("target.txt", "a")
    url_pool = Queue.Queue(0)
    start_url = "http://www.baidu.com/s?wd=site:(nuist.edu.cn)%20?"
    
    url_pool.put(start_url)

    try:
        while not url_pool.empty():
            new_url = url_pool.get()
            fp.write(new_url + "\n\n")

            nextpage = find_nextpage(new_url)
            url_pool.put(nextpage)
        
            Thread = MyThread(new_url, num)
            num += 1
            Thread.start()
    except:
        pass
    fp.close()  


if __name__ == '__main__':  
    main()  


```

