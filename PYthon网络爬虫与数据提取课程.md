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

## 2.1BeautifulSoup库 bs4
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
#<html>——<body>——<p>——<a>   遍历方法：下行遍历、上行遍历、平行遍历
#<head>   <p>     <a>
#<title>  <b>
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

     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
    
    
    
    
    
    
    
    











