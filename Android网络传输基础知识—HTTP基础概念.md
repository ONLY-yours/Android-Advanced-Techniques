# HTTP基础概念

### 报文格式

##### 1. 请求报文

请求行：请求方法+路径+版本号
 请求头：host，User-Agent等。下面具体介绍
 请求体：内容

```cpp
GET https://www.baidu.com/ HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Linux; Android 7.0; m3 note) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Mobile Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
ip=172.56.168.66&imei=86521684611
```

##### 2. 响应报文

状态行：版本号+状态码+状态信息
 响应头：Content-Type，Content-Length等
 响应体：返回内容

```dart
HTTP/1.1 200 OK
Server: nginx/1.8.1
Date: Thu, 03 Jan 2019 07:31:02 GMT
Content-Type: text/plain;charset=UTF-8
Content-Length: 93
Connection: keep-alive
Accept-Charset: big5, big5-hkscs, euc-jp, euc-kr, gb18030, gb2312, gbk, ibm-thai, ibm00858, ibm01140, ibm01141

{"adlist":{"kaiping":{"ad":{}},"yunying":{"ad":{}}},"showadtime":"10","getadlisttime":"1200"}
```

### 请求方法和响应码以及Header

##### 1. Request Method请求方法

- GET      获取资源，没有body
- POST     增加或修改资源，有body
- PUT      修改资源，有body（GET和PUT是幂等执行多次和一次是一样的，post增加多次就会有多个增加）
- DELETE   删除资源，没有body
- HEAD     和GET几乎相同，但是不返回body，一般用于下载前看看下载内容是否存在，大小，是否支持断点续传

##### 2. 响应状态码

###### 状态码类别

|      | 类别                           | 原因短语                   |
| ---- | ------------------------------ | -------------------------- |
| 1xx  | Informational(信息性状态码)    | 接收的请求正在处理         |
| 2xx  | Success(成功状态码)            | 请求正常处理完毕           |
| 3xx  | Redirection(重定向状态码)      | 需要进行附加操作以完成请求 |
| 4xx  | Client Error(客户端错误状态码) | 服务器无法处理请求         |
| 5xx  | Server Error(服务器错误状态码) | 服务器处理请求出错         |

- 1 : 临时性消息 100（继续发送）101（正在切换协议）Upgrade:h2c 返回101用HTTP2，发回200还用http/1.1
- 2 : 200成功 201创建成功	
- 3 : 重定向 301（永远移动）302（暂时移动）304（内容未改变）缓存问题	
- 4 : 客户端错误 400（Bad Request请求错误，参数不对等）401（认证失败，未登录等）403（内容被禁止，访问权限，访问授权问题）404（内容找不到）	
- 5 : 服务器错误

##### 3. Header(http消息的元数据)

- host：服务器主机地址  不是网络上用于寻址（DNS），而是目标服务器上用于定位子服务器
- content-length:指Body长度（主要用于二进制数据，确定内容长度）
- content-type:
   1.text/html  请求web页面的返回响应类型，Bady中返回html文本
   2.application/x-www-form-urlencoded 纯文本普通表单
   3.multitype/form-data 含有二进制内容的提交方式
   4.application/json 用于web Api的响应或者post/put的请求
   5.image/jpg application/zip 提交单文件 (网页不常用)
- Transfer-Encoding:chunked 用于当响应发起时，内容长度还没有确定下来，用途是尽早给出响应，减少用户等待。最后传输0表示传输介绍
- Location: 指定重定向的url地址，请求[http://www.baidu.com](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.baidu.com)重定向到[https://www.baidu.com](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.baidu.com)

```cpp
HTTP/1.1 307 Internal Redirect
Location: https://www.baidu.com/
Non-Authoritative-Reason: HSTS
```

- User-Agent：用户代理，实际发送请求，接受响应的浏览器和手机。（返回手机页面还是pc页面）
- Accept-Range:byte 响应报文中出现，表示服务器支持分段下载
- Range: bytes=0-15000 请求报文中出现，表示要取那段数据
   （Range/Accept-Range作用：断点续传，多线程下载）
- Content-Range:<start>-<end>/total 响应报文中出现，表示发送的是那段数据
- Accept: 通知服务器，用户代理能够处理的媒体类型以及媒体类型的相对优先级
- Accept-Charset:通知服务器用户代理支持的字符集及字符集的相对优先顺序
- Accept-Encoding:告知服务器用户代理支持的内容编码及内容编码的优先级顺序
- Accept-Language:告知服务器用户代理能够处理的自然语言集（指中文或英文等），以及自然语言集相对优先级。
- Content-Encoding：响应报文中压缩类型
- Cache 在客户端或中间网络节点缓存数据，降低从服务器取数据的频率，提高网络性能
- Cache-Control：缓存响应指令 no-cache（下次使用时，询问一下）,no-store(不要缓存),max-age（未过期直接使用） public （可向任意一方提供响应缓存）private （仅向特定用户返回响应）
- Last-Modified：服务器返回一个过期时间
- If-Modified-Since：客户端请求时带上这个时间
- Etag:  服务器发回来一个唯一标志符

```swift
HTTP/1.1 304 Not Modified
Cache-Control: max-age=0, must-revalidate
Date: Thu, 03 Jan 2019 08:22:51 GMT
Etag: 2098a650d25ba787abb9038281fd287a
```

- If-None-Match ：请求时带上这个唯一标志符

```undefined
GET /hm.js?0c0e9d9b1e7d617b3e6842e85b9fb068 HTTP/1.1
Host: hm.baidu.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36
Accept: */*
Referer: https://www.jianshu.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
If-None-Match: 2098a650d25ba787abb9038281fd287a
```

- Cache-Control: private(中间节点不能缓存数据，个性信息)/public(中间节点能缓存数据，公有信息)