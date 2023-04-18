## 爬虫爬取豆瓣评分电影Top250的爬虫代码
+  爬取的内容：**电影详情链接，图片链接，影片中文名，影片外国名，评分，评价数，概况，相关信息。**
+  爬虫的代码：
~~~
# -*- codeing = utf-8 -*-  #常见的字符编码包括 ASCII、UTF-8、UTF-16 等。不同的字符编码有不同的字符集和二进制编码方式，所以需要根据实际情况选择合适的字符编码。
from bs4 import BeautifulSoup  #bs4 是一个 Python 库，它提供了一组工具来解析 HTML 和 XML 文档，并提供简单的方式来检索文档中的信息。BeautifulSoup 是 bs4 库中最常用的类,可以方便地从 HTML 或 XML 文档中提取所需的信息。
import re  # 表示导入 Python 标准库中的 re 模块，该模块提供了正则表达式的支持，可以用来进行字符串匹配、查找、替换等操作。
import urllib.request, urllib.error  # 其中 urllib.request 子模块提供了一个 urlopen() 方法用于获取指定 URL 的页面内容，而 urllib.error 子模块则提供了一些常见的错误异常类，例如 URLError、HTTPError 等，用于在出现错误时进行处理。
import xlwt  # import xlwt 表示在 Python 代码中导入了 xlwt 模块，用于对 Excel 文件进行读写操作。xlwt 是一个第三方库，需要先安装才能使用
#import sqlite3  # 使用 sqlite3 模块可以完成 SQLite3 数据库的创建、连接、表的创建和数据的插入、查询、修改和删除等操作。

findLink = re.compile(r'<a href="(.*?)">')  # findLink 是一个正则表达式对象，用于匹配 HTML 中的链接,它被赋值为 re.compile(r'<a href="(.*?)">')
findImgSrc = re.compile(r'<img.*src="(.*?)"', re.S)  # <img.*是一个正则表达式，用于匹配HTML文档中的img标签；src="(.*?)"用于匹配 HTML 文档中的图片标签（<img> 标签）中的 src 属性值；re.S是re模块中的一个标志参数，表示在匹配时将.也匹配换行符（\n）
findTitle = re.compile(r'<span class="title">(.*)</span>') # class是一种用于给元素设置分类的属性，一个元素可以有多个class，各class之间用空格分隔，此处的class指代的是标题元素，</span>是HTML代码中的结束标签
findRating = re.compile(r'<span class="rating_num" property="v:average">(.*)</span>') # 在这里，property属性的值为"v:average"，它是指定网页中某个标签的属性值
findJudge = re.compile(r'<span>(\d*)人评价</span>') # 该段代码使用正则表达式在HTML文本中匹配包含“<span>数字人评价</span>”格式的字符串，并将这些字符串保存在一个列表中
findInq = re.compile(r'<span class="inq">(.*)</span>') # 该正则表达式可以匹配HTML中class属性为"inq"的span标签中的内容。
findBd = re.compile(r'<p class="">(.*?)</p>', re.S) # 这段代码主要用于提取页面中电影的相关描述信息，如导演、主演、剧情介绍等




def main():
    baseurl = "https://movie.douban.com/top250?start="  #要爬取的网页链接
    # 1.爬取网页
    datalist = getData(baseurl)
    savepath = "豆瓣电影Top250.xls"    #当前目录新建XLS，存储进去
    # dbpath = "movie.db"              #当前目录新建数据库，存储进去
    # 3.保存数据
    saveData(datalist,savepath)      #2种存储方式可以只选择一种
    # saveData2DB(datalist,dbpath)



# 爬取网页
def getData(baseurl):
    datalist = []  #用来存储爬取的网页信息
    for i in range(0, 10):  # 调用获取页面信息的函数，10次
        url = baseurl + str(i * 25)
        html = askURL(url)  # 保存获取到的网页源码
        # 2.逐一解析数据
        soup = BeautifulSoup(html, "html.parser")
        for item in soup.find_all('div', class_="item"):  # 查找符合要求的字符串
            data = []  # 保存一部电影所有信息
            item = str(item)
            link = re.findall(findLink, item)[0]  # 通过正则表达式查找
            data.append(link)
            imgSrc = re.findall(findImgSrc, item)[0]
            data.append(imgSrc)
            titles = re.findall(findTitle, item)
            if (len(titles) == 2):
                ctitle = titles[0]
                data.append(ctitle)
                otitle = titles[1].replace("/", "")  #消除转义字符
                data.append(otitle)
            else:
                data.append(titles[0])
                data.append(' ')
            rating = re.findall(findRating, item)[0]
            data.append(rating)
            judgeNum = re.findall(findJudge, item)[0]
            data.append(judgeNum)
            inq = re.findall(findInq, item)
            if len(inq) != 0:
                inq = inq[0].replace("。", "")
                data.append(inq)
            else:
                data.append(" ")
            bd = re.findall(findBd, item)[0]
            bd = re.sub('<br(\s+)?/>(\s+)?', "", bd)
            bd = re.sub('/', "", bd)
            data.append(bd.strip())
            datalist.append(data)

    return datalist


# 得到指定一个URL的网页内容
def askURL(url):
    head = {  # 模拟浏览器头部信息，向豆瓣服务器发送消息
        "User-Agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36"
    }
    # 用户代理，表示告诉豆瓣服务器，我们是什么类型的机器、浏览器（本质上是告诉浏览器，我们可以接收什么水平的文件内容）

    request = urllib.request.Request(url, headers=head)
    html = ""
    try:
        response = urllib.request.urlopen(request)
        html = response.read().decode("utf-8")
    except urllib.error.URLError as e:
        if hasattr(e, "code"):
            print(e.code)
        if hasattr(e, "reason"):
            print(e.reason)
    return html


# 保存数据到表格
def saveData(datalist,savepath):
    print("save.......")
    book = xlwt.Workbook(encoding="utf-8",style_compression=0) #创建workbook对象
    sheet = book.add_sheet('豆瓣电影Top250', cell_overwrite_ok=True) #创建工作表
    col = ("电影详情链接","图片链接","影片中文名","影片外国名","评分","评价数","概况","相关信息")
    for i in range(0,8):
        sheet.write(0,i,col[i])  #列名
    for i in range(0,250):
        # print("第%d条" %(i+1))       #输出语句，用来测试
        data = datalist[i]
        for j in range(0,8):
            sheet.write(i+1,j,data[j])  #数据
    book.save(savepath) #保存

# def saveData2DB(datalist,dbpath):
#     init_db(dbpath)
#     conn = sqlite3.connect(dbpath)
#     cur = conn.cursor()
#     for data in datalist:
#             for index in range(len(data)):
#                 if index == 4 or index == 5:
#                     continue
#                 data[index] = '"'+data[index]+'"'
#             sql = '''
#                     insert into movie250(
#                     info_link,pic_link,cname,ename,score,rated,instroduction,info)
#                     values (%s)'''%",".join(data)
#             # print(sql)     #输出查询语句，用来测试
#             cur.execute(sql)
#             conn.commit()
#     cur.close
#     conn.close()


# def init_db(dbpath):
#     sql = '''
#         create table movie250(
#         id integer  primary  key autoincrement,
#         info_link text,
#         pic_link text,
#         cname varchar,
#         ename varchar ,
#         score numeric,
#         rated numeric,
#         instroduction text,
#         info text
#         )
#
#
#     '''  #创建数据表
#     conn = sqlite3.connect(dbpath)
#     cursor = conn.cursor()
#     cursor.execute(sql)
#     conn.commit()
#     conn.close()

# 保存数据到数据库
~~~






if __name__ == "__main__":  # 当程序执行时
    # 调用函数
     main()
    # init_db("movietest.db")
     print("爬取完毕！")






