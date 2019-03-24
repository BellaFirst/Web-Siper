## PYTHON网络爬虫与信息提取
## 1.1 Requests库
#example1
import requests
r=requests.get("http://www.baidu.com")
print(r.status_code)##200
type(r)##<class 'requests.models.Response'>

##Response对象属性
type(r)
<class 'requests.models.Response'>
r.encoding
'ISO-8859-1'
r.apparent_encoding
'utf-8'
r.encoding='utf-8'##利用此编码替换编码
r.text
##说明：r.encoding 从HTTP header中猜测的相应内容编码方式 如果header中不存在charset 则认为编码为ISO-8859-1
##r.apparent_encoding 从内容中分析出相应的编码方式（备选编码方式） 分析内容得到编码信息
##request库异常
##r.raise_for_status() 如果不是200 产生异常requests.HTTPError

##[！！！爬取网页的通用代码框架！！！]
import requests
def getHTMLText(url):
    try:
        r=requests.get(url,timeout=30)
        r.raise_for_status()##如果状态不是200 引发HTTPError异常
        r.encoding=r.apparent_encoding
        return r.text
    except:
        return "Error"
if __name__=="__main__":
    url="http://www.baidu.com"
    print(getHTMLText(url))

##HTTP协议 Hypetext Transfer Protocol超文本传输协议 基于“请求与相应”模式的、无状态的应用层协议
##URL格式 http://host[:port][path] host：合法的INTERNET主机域名或IP地址 port:端口号，缺省端口为80 path：请求资源的路径
##URL是通过HTTP协议存取资源的Internet路径，一个URL对一个数据资源
##操作 get head post put patch delete

##requests.request(method,url,*kwargs) method:请求方法，对应get/put/post等7种 url:获取页面链接 kwargs:控制参数

## 1.2 网络爬虫“盗亦有道”
#Robots协议

## 1.3网络爬虫实例
#京东商品页面爬虫
import requests
url="https://item.jd.com/27378844116.html"
try:
    r=requests.get(url)
    print(r.raise_for_status())
    r.encoding=r.apparent_encoding
    print(r.rext[:10000])
except:
    print("Fail!")
#没有成功，发现网页不允许爬虫，修改程序如下：
import requests
url="https://item.jd.com/100000986052.html"
try:
    kv={'user-agent':'Mozilla/5.0'}#修改user-agent 模拟为网络访问
    r=requests.get(url,headers=kv)
    print(r.raise_for_status())
    r.encoding=r.apparent_encoding
    print(r.text[:2000])
except:
    print("Fail!")

##百度搜索
>>> import requests
>>> kv={'wd':'Python'}
>>> r=requests.get("http://www.baidu.com/s",params=kv)
>>> r.status_code
200
>>> r.request.url
'http://www.baidu.com/s?wd=Python'
>>> len(r.text)
451074
#规范代码框架  又错了。。。框架就有错，别的就没问题。。。
import requests
keyword="Python"
try:
    kv={'wd':keyword}
    r=requests.get("http://www.baidu.com/s",params=kv)
    print(r.requests.url)
    r.raise_for_status()
    print(len(r.text))
except:
    print("Fail!!")


##网络图片的爬取和存储
import requests
import os
url="https://b-ssl.duitang.com/uploads/item/201603/23/20160323161700_4mE5j.jpeg"
root="E://"
path=root+url.split('/')[-1]
try:
    if not os.path.exists(root):
        os.mkdir(root)
    if not os.path.exists(path):
        r=requests.get(url)
        with open(path,'wb')as f:
            f.write(r.content)
            f.close()
            print("Successed Saved！")
    else:
        print("Fail Saved!")
except:
    print("Fail!")

##IP地址归属地的查询
import requests 
url="http://m.ip138.com/ip.asp?ir="
try:
    r=requests.get(url+'202.113.112.30')
    r.raise_for_status()
    r.encoding=r.apparent_encoding
    print(r.text[-500:])
except:
    print("Fail!")

## 2.1 BeautifulSoup库 bs4
#from bs4 import BeautifulSoup
#soup=BeautifulSoup("<html>data<html>","html.parser")
#soup2=BeautifulSoup(open("文件地址")，“html.parser”)

BeautifulSoup类的基本元素
Tag             标签 最基本的信息组成单元 分别用<></>表明开头和结尾
Name            标签的名字，<p>...</p>的名字是'p'，格式：<tag>.name
Attributes      标签的属性，字典形式组织，格式：<tag>.attrs
NavigableString 标签内废属性字符串，<>...</>中文符串，格式：<tag>.String
Comment         标签内字符串的注释部分，一种特殊的comment类型
#Example
import requests
r=requests.get("http://python123.io/ws/demo.html")
demo=r.text
from bs4 import BeautifulSoup
soup=BeautifulSoup(demo,"html.parser")
print(soup.title)
print(soup.a)
print(soup.a.name)#获取标签名字
print(soup.a.parent.name)
tag=soup.a
print(tag.attrs)#获取标签属性
print(tag.attrs['class'])#标签属性为字典，获取某一类的属性
print(tag.attrs['href'])#连接属性
print(type(tag.attrs))#标签属性
print(type(tag))
    
#基于bs4库的HTML内容遍历方法
#HTML基本格式
##遍历方法：下行遍历、上行遍历、平行遍历
1.标签树的下行遍历
.contents   子节点的列表，将<tag>所有儿子节点存入列表
.children   子节点的迭代类型，与.contents类似，用于遍历儿子节点
.descendants子孙节点的迭代类型，包含所有子孙节点，用于循环遍历
eg：for child in soup.body.contents:
        print(child)
2.标签树的上行遍历
.parent  节点的父亲标签
.parents 节点先辈标签的迭代类型，用于循环遍历先辈节点（用循环的方式获取）
eg：for parent in soup.a.parents:
        if parent is None:
           print(parent)
       else:
           print(parent.name)  
3.平行遍历（**必须发生在同一个父节点下的各节点间）
.next_sibling       返回按照HTML文本顺序的下一个平行节点标签
.previous_sibling   。。。。。。。。。。。上。。。。。。。。
.next_siblings      迭代类型，返回按照HTML文本顺序的后续所有平行节点标签
.previous_siblings  。。。。。。。。。。。。。。。。前续。。。。。。。。

#基于bs4库的HTML格式化和编码
soup.prettify()
soup.a.prettify()

     
## 2.2信息组织与提取
#信息标记的三种方式
HTML（Hyper Text Markup Language）是www的信息组织方式,通过预定义的<>...</>标签形式组织不同类型的信息
1.XML <name> </name> <name/> <!--name--> Internet 上的信息交互与传递
2.JSOM 有类型的键值对 "key":"value"   "name":[“value1”,“value2”]   “key”：{“subkey”：“subvalue”} 移动应用云端和节点的信息通信，无注释
3.YAML 无类型的键值对 key：name        各系统的配置文件，有注释易读
                     key:#comment
                     -value1
                     -value2
                     key:
                        subkey:subvalue
#信息提取的一般方式
for link in soup.find_all('a'):
    print(link.get('href'))
#bs4库的HTML内容查找方法
<>.find_all(name,attrs,recursive,string,** kwarg)返回一个类表类型，存储查找结果
name 对标签名称的检索字符串
attrs 对标签属性值的检索字符串，课标注属性检索
recursive 是否对子孙全部检索，默认为True
string <>...</>中字符串区域的检索字符串

## 2.3中国大学排名
import requests
from bs4 import BeautifulSoup
import bs4
 
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""
 
def fillUnivList(ulist, html):
    soup = BeautifulSoup(html, "html.parser")
    for tr in soup.find('tbody').children:
        if isinstance(tr, bs4.element.Tag):
            tds = tr('td')
            ulist.append([tds[0].string, tds[1].string, tds[3].string])
 
def printUnivList(ulist, num):
    print("{:^10}\t{:^6}\t{:^10}".format("排名","学校名称","总分"))
    for i in range(num):
        u=ulist[i]
        print("{:^10}\t{:^6}\t{:^10}".format(u[0],u[1],u[2]))
     
def main():
    uinfo = []
    url = 'http://www.zuihaodaxue.cn/zuihaodaxuepaiming2016.html'
    html = getHTMLText(url)
    fillUnivList(uinfo, html)
    printUnivList(uinfo, 20) # 20 univs
main()

#完善程序
import requests
from bs4 import BeautifulSoup
import bs4
 
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""
 
def fillUnivList(ulist, html):
    soup = BeautifulSoup(html, "html.parser")
    for tr in soup.find('tbody').children:
        if isinstance(tr, bs4.element.Tag):
            tds = tr('td')
            ulist.append([tds[0].string, tds[1].string, tds[3].string])
 
def printUnivList(ulist, num):
    tplt = "{0:^10}\t{1:{3}^10}\t{2:^10}"
    print(tplt.format("排名","学校名称","总分",chr(12288)))
    for i in range(num):
        u=ulist[i]
        print(tplt.format(u[0],u[1],u[2],chr(12288)))
     
def main():
    uinfo = []
    url = 'http://www.zuihaodaxue.cn/zuihaodaxuepaiming2016.html'
    html = getHTMLText(url)
    fillUnivList(uinfo, html)
    printUnivList(uinfo, 20) # 20 univs
main()

## 3.1 Re正则表达式
##表示无穷或特别多不易枚举，通用字符串表达方式，以及判断查找匹配
##正则表达式常用操作符
.  表示任何单个字符
[] 字符集，对单个字符给出取值范围 [a-z]表示a到z的单个字符
[^]非字符集，对单个字符给出排除范围 [^abc]表示非a或b或c的单个字符
*  前一个字符0次或无限次扩展 abc* 表示ab。abc，abcc等
+  前一个字符1次或无限次扩展 abc+表示abc。abcc等
？ 前一个字符0次或1次扩展 abc？表示ab、abc
|  左右表达式任意一个 abc|def表说abc、def
{m} 扩展一个字符m次  ab{2}表示abb
{m,n} 扩展前一个字符m到n次 ab{1,2}c表示abc、abbc
^ 匹配字符串开头
$ 匹配字符串结尾
() 分组标记，内部只能用|操作符  (abc)表示abc，（abc|def）表示abc、def
\d 数字，等价于[0-9]
\w 单词字符，等价于[A-Za-z0-9]
##经典实例
^[A-Za-z]+$   26个字母组成的字符串
^[A-Za-z0-9]+$ 26个字母和数字组成的字符串
^-?\d+$        整数形式的字符串
^[0-9]*[1-9][0-9]*$ 正整数形式的字符串
[1-9]\d{5}     中国邮政编码，6位
[\u4e00-\u9fa5] 匹配中文字符
\d+.\d+.\d+.\d+. IP地址字符串形式的正则表达式
\d{1,3}.\d{1,3}.\d{1,3}.\d{1,3}
精确写法 0-99：[1-9]?\d    100-199:1\d{2}   200-249：2[0-4]\d  250-255:25[0-5]
(([1-9]?\d | 1\d{2} | 2[0-4]\d | 25[0-5]).){3}([1-9?\d | 1\d{2} | 2[0-4]\d | 25[0-5])

##Re库 用于字符串匹配 采用raw string类型即原生字符串类型
#当[正则表达式]包含<转义符>时，使用Raw String
Re库主要功能函数
re.search() 在一个字符串中搜素匹配正则表达式的第一个位置，返回match对象
re.match()  从一个字符串的开始位置起匹配正则表达式，返回match对象
re.findall() 搜索字符串，以列表类型返回全部能匹配的子串
re.split() 讲一个字符串按照正则表达式匹配结果进行分割，返回列表类型
re.finditer() 搜索字符串，返回一个匹配结果的迭代类型，每个迭代元素就是match对象
re.sub()     在一个字符串中替代所有匹配正则表达式的子串，返回替换后的字符串
#Re函数式用法：一次性操作 rst=re.search(r'[1-9]\d{4}','BIT 10081')
#Re面向对象用法：编译后的多次操作 pat=re.compile(r'[1-9]\d(4))  rst=pat.search('BIT 10081')

##Match对象的属性
.string  待匹配的文本
.re      匹配时使用的pattren对象（正则表达式）
.pos     正则表达式搜素文本的开始位置
.endpos  正则表达式搜索文本的结束位置
##Match对象的方法
.group(0)  获得匹配后的字符串
.start()   匹配字符串在原始字符串的开始位置
.end()     。。。。。。。。。。。。结束位置
.span()    返回（.start(),.end())

##Re库贪婪匹配和最小匹配
#默认贪婪匹配，输出最长字符串
#最小匹配操作符
* ？   前一个字符0次或无限次扩展，最小匹配
+？     。。。。1.。。。。。。。。。。。。
？？    。。。。。0次或1次扩展，最小匹配
{m，n} 扩展前一个字符m或n次，最小匹配

##淘宝商品信息定向爬虫（程序没问题，应该是淘宝不允许访问）
import requests
import re
def getHTMLText(url):#获得页面
    try:
        r=requests.get(url,timeout=30)
        r.raise_for_status()
        r.encoding=r.apparent_encoding
        return r.text
    except:
        return " "

def parsePage(ilt,html):#对获得的页面进行解析 获取名称和价格
    try:
        plt=re.findall(r'\"view_price\"\:\"[\d\.]*\"',html)#获得商品价格以及前面view_price标识
        tlt=re.findall(r'\"raw_title\"\:\".*?\"',html)#获得商品名称
        for i in range(len(plt)):
            price=eval(plt[i].split(':')[1])
            title=eval(tlt[i].split(':')[1])
            ilt.append([price,title])
    except:
        print("")
    
def printGoodsList(ilt):#将商品信息输出到屏幕上
    tplt="{:4}\t{:8}\t{:16}"
    print(tplt.format("序号","价格","名称"))
    count=0
    for g in ilt:
        count=count+1
        print(tplt.format(count,g[0],g[1]))
    
def main():
    goods='书包'
    depth=2  #爬取深度为2页
    start_url='https://s.taobao.com/search?q='+goods
    infoList=[] #输出结果
    for i in range(depth):
        try:
            url=start_url+'&s='+str(44*i) #对每个页面进行设计
            html=getHTMLText(url)
            parsePage(infoList,html)
        except:
            continue
    printGoodsList(infoList)

main()





















