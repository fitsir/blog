---
title: 使用Scrapy爬取论坛信息
date: 2016-09-18 16:32:23
tags:
    - Python
    - Scrapy
    - MySQL
    - Flask
    
categories: Python
toc: true
---

参考[buptbill220/bbsspider](https://github.com/buptbill220/bbsspider)的论坛爬虫

## 开发环境
开发环境是Windows 10 64bit + Python3.5.2 + MySQL5.7, eclipse+pydev.

<!--more-->

## 配置MySQL
在`mysql`中创建`byr`数据库, 库中三个表创建如下:


```sql
CREATE TABLE `article` (                       
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,        
  `url` varchar(80) NOT NULL,                              
  `text` text,                                             
  PRIMARY KEY (`id`),                                      
  UNIQUE KEY `url` (`url`)                                 
) ENGINE=InnoDB AUTO_INCREMENT=7536 DEFAULT CHARSET=utf8;



CREATE TABLE `article_list` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `uptime` date DEFAULT '2016-05-19',
  `hot` int(10) unsigned NOT NULL DEFAULT '0',
  `author` varchar(50) NOT NULL,
  `title` varchar(100) NOT NULL,
  `url` varchar(80) NOT NULL DEFAULT 'http://bbs.byr.cn',
  PRIMARY KEY (`id`),
  UNIQUE KEY `url` (`url`),
  KEY `author` (`author`)
) ENGINE=InnoDB AUTO_INCREMENT=28425 DEFAULT CHARSET=utf8;

CREATE TABLE `section` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `url` varchar(60) NOT NULL,
  `name` varchar(50) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=451 DEFAULT CHARSET=utf8;
```

## 创建爬虫

### 创建项目
在开始爬取之前，您必须创建一个新的Scrapy项目。 进入您打算存储代码的目录中，运行下列命令:
```bash
$ scrapy startproject byr_scrapy
```
### 创建第一个爬虫
在`spiders`目录下创建一个`ByrSectionSpider.py`文件, 创建爬虫,初始化信息如下
```python
class ByrSectionSpider(scrapy.Spider):
    name = "byr-section"
    allowed_domains = ["bbs.byr.cn"]
    cookiejar = {}
    db = None
    headers = {'User-Agent': 'Mozilla/5.0', 'Host': 'bbs.byr.cn',
               'X-Requested-With': 'XMLHttpRequest', 'Connection': 'keep-alive'}
    pat = r'href=\"(.*?)\" title=\"(.*?)\"'
    section_pat = r'^/section/(.*?)$'

    start_urls = ['http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-0',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-1',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-2',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-3',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-4',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-5',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-6',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-7',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-8',
                  'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-9']
```
其中
* `class ByrSectionSpider(scrapy.Spider):`新建的爬虫类,集成自`scrapy.Spider`
* `name = "byr-section"`设置爬虫的名称,唯一, 使用如: `$ scrapy crawl byr-section`
* `start_urls` 设置爬虫开始爬的网址

### 模拟登陆

```python
def start_requests(self):
    self.db = MySQL('127.0.0.1', '******', '******', 'byr', 3306, 'utf8', 5, '')
    return [scrapy.FormRequest("http://bbs.byr.cn/user/ajax_login.json",
                    formdata={'id': '**', 'passwd': '******', 'mode': '0', 'CookieDate': '0'},
                        meta={'cookiejar': 1}, headers=self.headers, callback=self.logged_in)]
def logged_in(self, response):
    print('\n\n', response.body_as_unicode(), '\n\n')
    self.cookiejar = response.meta['cookiejar']
    for url in self.start_urls:
        yield scrapy.Request(url, meta={'cookiejar': response.meta['cookiejar']}, 
            headers=self.headers, callback=self.parse)
```

### 分析网页

首先通过正则找到板块列表中每个板块的链接, 形如`/board/Python`, 将其和`title`字段存储在数据表`section`中,如果链接地址为`/section/***`则代表是个二级目录,则进入该目录再次进行`parse`
```python
    def parse(self, response):
        body = response.body_as_unicode()
        try:
            data = body.replace(r'\"', '"')
            results = re.findall(self.pat, data, re.I)
            for result in results:
                url = result[0]
                title = result[1]
                print('url [%s], title [%s]' % (url, title))
                rs = re.findall(self.section_pat, url, re.I)
                if rs:
                    nurl = 'http://bbs.byr.cn/section/ajax_list.json?uid=ae&root=sec-%s' % rs[0]
                    print(nurl)
                    yield scrapy.Request(nurl, meta={'cookiejar': self.cookiejar}, 
                        headers=self.headers, callback=self.parse)
                else:
                    self.store_data({'url': url, 'name': title})
        except:
            print('parse json [%s] failed' % body)
```
