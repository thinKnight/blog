title: 豆瓣爬虫
date: 2014-08-28 15:08:40
tags: Python
---
可抓取豆瓣读书、电影、音乐中任意标签下内容
<!-- more -->

在我刚刚入门Python爬虫的时候，无论怎样都很难找到一个适当的实例让我参考。  

看过很多别人的例子，但都觉得不得要领，所以在折腾很久后写了这个简单的例子。  

---

### 源码

```Python
# -*- coding: UTF-8 -*-
# 如果要在python2的py文件里面写中文，则必须要添加一行声明文件编码的注释，否则python2会默认使用ASCII编码。  

import re 
import urllib2

def douban_crawler(url_head, target):
    for page in range(0, 1000, 20):
    #这个1000是检索的条目数量，可以按需设定
        url_rear = "?start=%d&type=T" % page
        url_use = url_head + url_rear
        #两段合成真正的url
        content = urllib2.urlopen(url_use).read()
        content = content.decode("UTF-8").encode("UTF-8")
        
        content = content.replace(r'title="去FM收听"', "")
        content = content.replace(r'title="去其他标签"', "")
        
        name = re.findall(r'title="(\S*?)"', content, re.S)
        #正则表达式捕获标题
        num  = re.findall(r'<span\s*class="rating_nums">([0-9.]*)<\/span>', content)
        #正则表达式捕获分数
        
        doc = zip(name, num)
        #将标题和分数打包成([ , ][ , ]...)的形式
        if target == "book":
            dou = open("doc_book.txt", 'a')
        elif target == "music":
            dou = open("doc_music.txt", 'a')
        elif target == "movie":
            dou = open("doc_movie.txt", 'a')

        for i in doc:
            dou.write(i[0] + " " + i[1] + "\n")
            #写入
    dou.close()

if __name__ == '__main__':
    target = raw_input("豆瓣 book movie music，你想爬哪一个? ")

    tag   = raw_input("请输入你想要检索的标签: ")

    url_head  = "http://%s.douban.com/tag/%s" % (target, tag)

    douban_crawler(url_head, target)
    
    print "抓取完毕"
```


### 抓取结果
- **豆瓣读书-小说**

> 月亮和六便士 9.0  
百年孤独 9.2  
解忧杂货店 8.7  
追风筝的人 8.8  
霍乱时期的爱情 9.0  
平凡的世界（全三部） 9.0  
围城 8.9  
活着 9.1  
一九八四 9.3  
人生的枷锁 9.0  
陆犯焉识 8.7  
...  

- **豆瓣电影-悬疑**

> 盗梦空间 9.2  
寒战 7.4  
嫌疑人X的献身 7.4  
七宗罪 8.7  
致命ID 8.5  
云图 8.0  
禁闭岛 8.5  
蝴蝶效应 8.6  
致命魔术 8.8  
恐怖游轮 8.2  
...

- **豆瓣音乐-pop**

> 十二新作 8.2  
Alright,Still 7.7  
范特西 8.5  
Apologize 8.9  
逆光 7.3  
Spin 8.5  
阿岳正传 8.3  
感官/世界 8.7  
八度空间 7.5  
PCD 8.4  
...


