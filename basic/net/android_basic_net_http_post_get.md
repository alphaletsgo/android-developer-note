# HTTP GET和POST区别

### HTTP协议格式

为了理解两者在传输过程中的不同，我们先看一下HTTP协议的格式：

```xml
<request line> <!-->请求行<-->
<headers> <!-->头部信息<-->
<blank line>	<!-->空行<-->
<request-body>	<!-->请求体<-->
```

在HTTP请求中，第一行必须是一个请求行（request line），用来说明请求**类型**、**要访问的资源**以及**使用的HTTP版本**。紧接着是一个首部（header）小节，用来说明服务器要使用的附加信息。在首部之后是一个空行，再此之后可以添加任意的其他数据[称之为主体(body)]。

GET与POST方法实例：

**GET**

```
GET /books/?sex=man&name=Professional HTTP/1.1
Host: www.wrox.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
Gecko/20050225 Firefox/1.0.1
Connection: Keep-Alive
```

**POST**

```
POST / HTTP/1.1
Host: www.wrox.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
Gecko/20050225 Firefox/1.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 40
Connection: Keep-Alive
     （----此处空一行----）
[name=Professional%20Ajax&publisher=Wiley]
```

有了以上对HTTP请求的了解和示例，我们再来看两种提交方式的区别：

- **GET提交**：请求的数据会附在URL之后（就是把数据放置在请求行（request line）中），以``?``分割URL和传输数据，多个参数用&连接；例如：``login.action?name=hyddd&password=idontknow&verify=%E4%BD%A0 %E5%A5%BD``。Url的编码格式采用的是**ASCII码**，而不是Unicode，这也就是说你不能在Url中包含任何非ASCII字符，所有非ASCII字符均需要编码再传输，关于Url编码可参考：[http://kb.cnblogs.com/page/133765/](http://kb.cnblogs.com/page/133765/)。

  **POST提交**：把提交的数据放置在是HTTP包的包体中。上文示例中```[]```中的数据就是实际的传输数据，因此，GET提交的数据会在地址栏中显示出来，而POST提交，地址栏不会改变


- 传输数据的大小：首先声明：HTTP协议没有对传输的数据大小进行限制，HTTP协议规范也没有对URL长度进行限制。
  而在实际开发中存在的限制主要有：

  GET:特定浏览器和服务器对URL长度有限制，例如IE对URL长度的限制是2083字节(2K+35)。对于其他浏览器，如Netscape、FireFox等，理论上没有长度限制，其限制取决于操作系统的支持。

  因此对于GET提交时，传输数据就会受到URL长度的限制。

  POST:由于不是通过URL传值，理论上数据不受限。但实际各个WEB服务器会规定对post提交数据大小进行限制，Apache、IIS6都有各自的配置。

 ### 安全性

POST的安全性要比GET的安全性高。注意：这里所说的安全性和上面GET提到的“安全”不是同个概念。上面“安全”的含义仅仅是不作数据修改，而这 里安全的含义是真正的Security的含义，比如：通过GET提交数据，用户名和密码将明文出现在URL上，因为

(1)登录页面有可能被浏览器缓存。

(2)其他人查看浏览器的历史纪录，那么别人就可以拿到你的账号和密码了，除此之外，使用GET提交数据还可能会造成Cross-site request forgery攻击。

### Http get、post、soap

1）get：请求参数是作为一个key/value对的序列（查询字符串）附加到URL上的，查询字符串的长度受到web浏览器和web服务器的限制（如IE最多支持2048个字符），不适合传输大型数据集同时，它很不安全。

2）post：请求参数是在http标题的一个不同部分（名为entity body）传输的，这一部分用来传输表单信息，因此必须将Content-type设置为:application/x-www-form-urlencoded。post设计用来支持web窗体上的用户字段，其参数也是作为key/value对传输。
​      但是：它不支持复杂数据类型，因为post没有定义传输数据结构的语义和规则。
3）soap：是http post的一个专用版本，遵循一种特殊的xml消息格式（Content-type设置为: text/xml   任何数据都可以xml化）。

 

### HTTP响应 

1．HTTP响应格式：

```xml
<status line>
<headers>
<blank line>
[<response-body>]
在响应中唯一真正的区别在于第一行中用状态信息代替了请求信息。状态行（status line）通过提供一个状态码来说明所请求的资源情况。 
 HTTP响应实例：
HTTP/1.1 200 OK
Date: Sat, 31 Dec 2005 23:59:59 GMT
Content-Type: text/html;charset=ISO-8859-1
Content-Length: 122
＜html＞
＜head＞
＜title＞Wrox Homepage＜/title＞
＜/head＞
＜body＞
＜!-- body goes here --＞
＜/body＞
＜/html＞
2．最常用的状态码有：
◆200 (OK): 找到了该资源，并且一切正常。
◆304 (NOT MODIFIED): 该资源在上次请求之后没有任何修改。这通常用于浏览器的缓存机制。
◆401 (UNAUTHORIZED): 客户端无权访问该资源。这通常会使得浏览器要求用户输入用户名和密码，以登录到服务器。
◆403 (FORBIDDEN): 客户端未能获得授权。这通常是在401之后输入了不正确的用户名或密码。
◆404 (NOT FOUND): 在指定的位置不存在所申请的资源。
```
 ![image](/imgs/http_head_struct.png)

### 完整示例

例子：

**HTTP GET** 
发送

```
GET /DEMOWebServices2.8/Service.asmx/CancelOrder?UserID=string&PWD=string&OrderConfirmation=string HTTP/1.1
Host: api.efxnow.com
```

回复

```Xml
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8
Content-Length: length
<?xml version="1.0" encoding="utf-8"?>
<objPlaceOrderResponse xmlns="https://api.efxnow.com/webservices2.3">
<Success>boolean</Success>
<ErrorDescription>string</ErrorDescription>
<ErrorNumber>int</ErrorNumber>
<CustomerOrderReference>long</CustomerOrderReference>
<OrderConfirmation>string</OrderConfirmation>
<CustomerDealRef>string</CustomerDealRef>
</objPlaceOrderResponse>
```

**HTTP POST** 
发送

```
POST /DEMOWebServices2.8/Service.asmx/CancelOrder HTTP/1.1
Host: api.efxnow.com
Content-Type: application/x-www-form-urlencoded
Content-Length: length

UserID=string&PWD=string&OrderConfirmation=string
```

回复

```xml
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8
Content-Length: length
<?xml version="1.0" encoding="utf-8"?>
<objPlaceOrderResponse xmlns="https://api.efxnow.com/webservices2.3">
<Success>boolean</Success>
<ErrorDescription>string</ErrorDescription>
<ErrorNumber>int</ErrorNumber>
<CustomerOrderReference>long</CustomerOrderReference>
<OrderConfirmation>string</OrderConfirmation>
<CustomerDealRef>string</CustomerDealRef>
</objPlaceOrderResponse>
```

**SOAP 1.2** 
发送

```Xml
POST /DEMOWebServices2.8/Service.asmx HTTP/1.1
Host: api.efxnow.com
Content-Type: application/soap+xml; charset=utf-8
Content-Length: length

<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
<soap12:Body>
    <CancelOrder xmlns="https://api.efxnow.com/webservices2.3">
      <UserID>string</UserID>
      <PWD>string</PWD>
      <OrderConfirmation>string</OrderConfirmation>
    </CancelOrder>
</soap12:Body>
</soap12:Envelope>
```

回复

```xml
HTTP/1.1 200 OK
Content-Type: application/soap+xml; charset=utf-8
Content-Length: length
<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
<soap12:Body>
    <CancelOrderResponse xmlns="https://api.efxnow.com/webservices2.3">
      <CancelOrderResult>
        <Success>boolean</Success>
        <ErrorDescription>string</ErrorDescription>
        <ErrorNumber>int</ErrorNumber>
        <CustomerOrderReference>long</CustomerOrderReference>
        <OrderConfirmation>string</OrderConfirmation>
        <CustomerDealRef>string</CustomerDealRef>
      </CancelOrderResult>
    </CancelOrderResponse>
</soap12:Body>
</soap12:Envelope>
```

