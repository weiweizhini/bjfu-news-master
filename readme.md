# 北京林业大学绿色新闻搜索引擎

## 整体过程
1. 爬虫爬取以 http://news.bjfu.edu.cn/lsxy/ 打头的所有新闻页面的内容，并将新闻页面的标题（newsTitle）、链接地址(newsUrl)、发表时间(newsPublishTime)、浏览次数(newsClickCount)、来源部门(newsSource)以及新闻内容（newsContent）等属性抓取下来。
2. 关系类数据库不适用于爬虫数据存储，因此使用非关系类数据库MongoDB。数据库可用可视化工具方便查看，例如Mongo Management Studio。
3. 使用Whoosh全文索引库建立文档索引，使用jieba分词引用ChineseAnalyzer进行中文分词,并实现对中文查询提取关键字进行搜索。
4. 搭建人机交互的搜索引擎Web界面。使用flask框架进行HTTP响应

## 使用方法
1. 启动mongodb，并在main.py中的main函数中设置连接mongodb方式，默认是5000端口，数据库无密码， [mongodb安装教程-仅供参考](https://www.runoob.com/mongodb/mongodb-window-install.html)
2. 运行``pip install -r requirements.txt`` 安装依赖库，这里推荐豆瓣的源``-i https://pypi.doubanio.com/simple``
3. 运行mian.py，当第二次运行时可以注释掉main函数，只启动http（main函数的主要功能是爬取数据并保存到数据库）

## 目录结构
```
├─static            网页目录用来存储网页
│  └─css
│  └─js
│  index.html       主页
│  
├─code              python代码文件夹
│  └─baseLib        通用基础类文件夹，用来存放不是针对本工程的通用类
│  BjfuHtml.py      针对本工程的python文件包括GetBjfuHtmls和BjfuHtmlPage连个类
│
└─main.py           主函数
└─requirements.txt
```
## python类介绍
其中基础类是通用类可以用到其他爬虫相关的项目上

### GetBjfuHtmls
- 针对绿色新闻网的爬虫，用一个起始url进行初试化
- 调用start启动爬虫进行连续爬取，知道爬取完全部数据停止
- 主要成员： Scheduler（调度器）
- 若要实现爬取其他网站可以重新实现start方法

### BjfuHtmlPage
- 继承自HtmlPage类，自己实现页面分析get_datas方法就可以
- BjfuHtmlPage是针对绿色新闻网的分析类，可以访问指定url，并对该页面进行分析

### URLList（基础类）
- 一个url容器
- URLList是一个简单的url容器，当url数据量变多时要重新实现URLlist
- 提供增删查三种方法

### Scheduler（基础类）
- 一个简单的调度器，容器采用的URLList
- 只提供两个方法：insert和get，使用者无需关心是否回插入重复的url，也就是说调度器自动避免访问重复的页面
- 同时可以用len方法查看调度器中还有多少url要访问
- 原理：同时有两个容器一个保存要访问的url一个保存历史url,url被get之后会自动从要访问的url容器放入历史容器中

### HtmlPage（基础类）
- HtmlPage:页面分析类，用来对一个页面进行分析
- 使用方式：
    1. 用一个url初试化
    2. 调用request进行请求
    3. 分析页面
    - get_html方法获取整个html（shtring类型）
    - et_all_url方法获取页面中的全部url
    - 子类可以利用self.soup（BeautifulSoup(self.html_text, 'lxml') 的返回）对页面进行分析
    
### MyWhoosh（基础类）
- 对whoosh的简单封装
- 提供增删查3种方法，可以插入一篇文章，然后通过关键字调用find查找这篇文章

2.爬虫爬取北京林业大绿色新闻网页
(1)获取目标网址信息

(2)提取目标网址信息
谷歌浏览器可以直接通过检查查看到我们需要的信息在哪里

(3)获取文章内容

所需要的文章内容信息都在class="article_con"的div中，得到所在位置，采用css选择器的方法进行元素定位，下面通过代码来获取。

(4)获取其他项（发表时间，来源，作者）


找到发表时间、来源和作者所在的标签，发现它的类别为span，进行提取。下面通过代码来获取。

(5)获取标题

所需要的文章标题信息都在<h2>标签中，下面通过代码来获取。

(6)获取点击次数

所需要的文章点击次数都在js中，下面通过代码来获取。



(7)爬取以 http://news.bjfu.edu.cn/lsxy/ 打头的所有新闻页面的内容
上面（1）-（6）只是对于某一个新闻页面的信息爬取，给定绿色新闻的网址，能自动爬取上面所有的网址，爬虫获取到目标网页的翻页地址翻页，这里需要两步：1.识别翻页的<a>标签2.提取网址获取新闻链接地址。
URLList（基础类）来实现URLList是一个简单的url容器，当url数据量变多时要重新实现URLlist提供增删查三种方法。

(8)将爬取的信息存储到数据库中

3.实现数据库
首先安装MongoDB和Robo3实现可视化工具方便查看。



Python中实现数据库的连接和数据的插入实现代码如下：
数据库连接

插入数据


4.Whoosh全文索引的实现
MyWhoosh（基础类）对whoosh的简单封装。MyWhoosh继承whoosh实现使用jieba分词引用ChineseAnalyzer进行中文分词,并实现对中文查询提取关键字进行搜索。
我们需要创建索引文件存储信息至mkdir目录，创建一个MyWhoosh索引擎。

提供增删查3种方法，可以插入一篇文章，然后通过关键字调用find查找这篇文章。
插入方法代码：


find方法代码：


5.搜索引擎Web界面
使用flask框架进行HTTP响应。导入安装Flask库



使用python运行后访问localhost:5000就能看到网页显示搜索。

