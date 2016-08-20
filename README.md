#代码编写

###分析页面信息：
1. 我需要获取的是每一个「公司」的详情页面链接 和 分页按钮链接；
2. 统一存储获取到的链接，提供给多个 spider 爬取；
3. 多个 spider 共享一个 redis list 中的链接；

#代码说明：
###juzi_spider.py
1. class 继承了RedisCrawlSpider 而不是CrawlSpider
2. start_urls 改为一个自定义的 itjuziCrawler:start_urls,这里的itjuziCrawler:start_urls 就是作为所有链接存储到 redis 中的 key,scrapy_redis 里也是通过redis的 lpop方法弹出并删除链接的；

###db_util.py
使用 SQLAlchemy 作为 ORM 工具，当表结构不存在时，自动创建表结构

###middlewares.py
增加了很多 User-Agent，每一个请求随机使用一个，防止防止网站通过 User-Agent 屏蔽爬虫

###settings.py
配置middlewares.py scrapy_redis redis 链接相关信息

#部署：
###Dockerfile
   1.使用 python3.5作为基础镜像
   2.将/usr/local/bin设置环境变量
   3.映射 host 和 container 的目录
   4.安装 requirements.txt
   5.特别要说明的是COPY spiders.py /usr/local/lib/python3.5/site-packages/scrapy_redis，将 host 中的 spiders.py 拷贝到container 中的 scrapy_redis 安装目录中，因为 lpop 获取redis 的值在 python2中是 str 类型，而在 python3中是 bytes 类型，这个问题在 scrapy_reids 中需要修复，spiders.py 第84行需要修改；
启动后立即执行爬行命令 scrapy crawl itjuzi_dis

###docker-compose.yml
  1. 使用第2版本的 compose 描述语言
  2. 定义了 spider 和 redis 两个 service
  3 .spider默认使用当前目录的 Dockerfile 来创建，redis使用 redis:latest 镜像创建，并都映射6379端口


#启动 container
  1. docker-compose up #从 docker-compose.yml 中创建 `container` 们
  2. docker-compose scale spider=4 #将 spider 这一个服务扩展到4个，还是同一个 redis


#现在给 redis 中放入 start_urls:
   lpush itjuziCrawler:start_urls http://www.itjuzi.com/company
   
   4个爬虫都动起来了，一直爬到start_urls为空
