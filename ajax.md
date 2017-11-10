# Ajax

## 0. 跨越携带Cookies

跨域请求设置withCredentials （主域名相同，子域名不同）

> 默认情况下，标准的跨域请求是不会发送cookie等用户认证凭据的，XMLHttpRequest 2的一个重要改进就是提供了对授信请求访问的支持。

`$.ajaxSetup({crossDomain:true,xhrFields:{withCredentials:true}});`

`xhr.withCredentials = true;`

然后设置服务端响应头

`Access-Control-Allow-Credentials: true`

并且服务端需要制定一个域名而不能是泛型 *

`Access-Control-Allow-Origin:www.github.com` 而不能是  `Access-Control-Allow-Origin:*`

## 1. jQuery ajax

`$.ajax(settings)`

| oPTIO                    | 作用                   | 默认值（或者描述）                              |
| ------------------------ | -------------------- | -------------------------------------- |
| async                    | 同步或者异步               | true                                   |
| beforeSend(xhr)          | 发送请求前                | 返回false可取消请求                           |
| cache                    |                      | true，dataType为script和jsonp时默认为false    |
| complete(xhr,ts)         |                      |                                        |
| contentType              | *发送*信息时的编码类型         | application/x-www-form-urlencoded      |
| context                  | 设置ajax回调函数的上下文       | 让回调内的this指向这个对象 context: document.body |
| data                     |                      |                                        |
| dataFilter(data,type)    | 给ajax返回的原始数据进行预处理的函数 |                                        |
| dataType                 | 预期*服务器*返回的数据类型       |                                        |
| error(xhr,err,errObject) |                      |                                        |
| jsonp                    | 在jsonp请求中重写回调函数的名字   |                                        |
| jsonpCallback            | 为jsonp指定一个回调函数名      |                                        |
| password                 | 用来响应 HTTP 访问认证请求的密码  |                                        |
| success                  |                      |                                        |
| traditional              | 使用传统方式来序列化数据         |                                        |
| timout                   | 超时时间                 |                                        |



## 2. 原生 ajax



## 3. 一个 ajax 发送了2次请求

*当你在跨域的时候发送一个非简单请求时*  (http://www.cnblogs.com/huangyuezhen/p/6868740.html)

浏览器会预先发送一个OPTIONS请求，来查明这个跨站请求对于目的站点是不是可安全接受的。

> 当服务端对 OPTIONS 请求返回表示支持跨域请求的 Origin, method, headers 时, 浏览器才会发送你所需要的真正的跨域请求

什么是简单请求	

1. 只使用 GET，HEAD，或者POST等请求方法，如果使用POST向服务端传送数据，则数据类型只能是`application/x-www-form-urlencoded`, `multipart/form-data 或 text/plain`中的一种。
2. 不包含自定义的请求头