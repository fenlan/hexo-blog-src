---
title: GET vs POST
date: 2018-04-13 15:18:23
categories: Web
tags:
  - GET
  - POST
  - HTTP
---

在综合考虑下，决定复习一些Web方向的东西，以备不时之需。。。。

## What is HTTP?
HTTP全称`Hypertext Transfer Protocol`(就是中文常说的`超文本传输协议`)，这个协议是为了客户端和服务端能进行通信。HTTP是客户端和服务端进行请求响应过程的协议，客户端发送请求，服务端响应这个请求并返回信息。通常一个web浏览器作为一个客户端，而一台提供web服务的机器作为服务端。

## Two HTTP Request Methods: GET and POST
两个非常常用的请求响应方法 : GET and POST。
- `GET`- Requests data from a specified resource
- `POST`- Submits data to be processed to a specified resource

### The GET Method
GET请求方法会将请求信息放在URL中
``` http
/test/demo_form.php?name1=value1&name2=value2
```
GET其他特性：
- GET请求能被缓存
- GET请求会保留在浏览器的历史记录中
- GET请求可以被标为书签
- GET请求不应该用在处理敏感数据时候
- GET请求有长度限制
- GET只应该用在取回数据

> 总的来说，由于GET请求数据被放在URL中，以及它能缓存，便导致了上述的其他特性。诸如`GET请求会保留在浏览器的历史记录中` `GET请求可以被标为书签` `GET请求不应该用在处理敏感数据时候`这三条就是因为GET请求会被缓存，且请求数据在URL中，又因为浏览器或者说操作系统对URL的长度有要求，所以GET请求有长度限制。

<!--more-->
### The POST Method
POST请求包含的数据放在了HTTP数据报中
``` http
POST /test/demo_form.php HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
```
POST其他特性：
- POST请求不会被缓存
- POST请求不会保留在浏览器历史记录中
- POST请求不会被标记为书签
- POST请求对数据长度没有限制

## Compare GET vs. POST
|             |  GET    |  POST  |
|-------------|---------|--------|
| 返回/回退    |  没有任何影响(因为有缓存机制)  | 数据会被重新提交(浏览器会提醒用户数据会被重新提交) |
| 书签         | 可以作为书签 | 不能作为书签 |
| 缓存         | 可以被缓存 | 不能被缓存 |
| 编码类型     | application/x-www-form-urlencoded | application/x-www-form-urlencoded or multipart/form-data. Use multipart encoding for binary data |
| 历史记录     | 参数会保留在浏览器历史记录中 | 参数不会保留在历史记录中 |
| 数据长度限制  | GET数据会放在URL中，URL的最大长度为2048个字符 | 没有限制 |
| 数据类型限制  | 只允许ASCII字符 | 没有限制 |
| 安全性       | GET数据安全性低于PSOT，因为它将数据放入URL(一定不用GET发送密码或者其他敏感数据) | POST安全性略高于GET，它的数据不会保留在浏览器历史中，也不会保留在服务器日志文件中(但不是绝对安全，其他人可以截获数据包，分析数据包得到数据) |

## Other HTTP Request Methods
| Method | Description |
|--------|--------|
| HEAD | Same as GET but returns only HTTP headers and no document body |
| PUT  | Uploads a representation of the specified URI |
| DELETE | Deletes the specified resource |
| OPTIONS | Returns the HTTP methods that the server supports |
| CONNECT | Converts the request connection to a transparent TCP/IP tunnel |

## REST 方法使用标准
下列是常用的REST方法定义：

- GET（SELECT）：从服务器取出资源（一项或多项）
- POST（CREATE）：在服务器新建一个资源
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）
- PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）
- DELETE（DELETE）：从服务器删除资源

使用例子：

- GET /zoos：列出所有动物园
- POST /zoos：新建一个动物园
- GET /zoos/ID：获取某个指定动物园的信息
- PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息
- PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
- DELETE /zoos/ID：删除某个动物园
- GET /zoos/ID/animals：列出某个指定动物园的所有动物
- DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

## 状态码(Status Code)
- 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的(Idempotent)
- 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
- 202 Accepted - [\*]：表示一个请求已经进入后台排队(异步任务)
- 204 NO CONTENT - [DELETE]：用户删除数据成功。
- 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
- 401 Unauthorized - [\*]：表示用户没有权限（令牌、用户名、密码错误）。
- 403 Forbidden - [\*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
- 404 NOT FOUND - [\*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
- 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
- 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
- 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
- 500 INTERNAL SERVER ERROR - [\*]：服务器发生错误，用户将无法判断发出的请求是否成功。

## 参考链接
- [HTTP Methods: GET vs. POST](https://www.w3schools.com/tags/ref_httpmethods.asp)
- [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)