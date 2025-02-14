# 协议与伪协议 

网络协议 是通信计算机双方约定好的规则  遵守约定 计算机双方便能进行通信

**伪协议** 

并不是共用的协议 往往是某个语言或某个程序自身 为了解决自己内部通信需求自行编制的一个协议

PHP伪协议利用

1. **跟踪输入点**

2. **输入点进入文件操作函数 readfile  file 这种**

3. **能够控制参数的开头**        

   ```
   <?php
   $input = $_GET['input'];
   
   file_get_contents($input."dada"); //可撸
   
   file_get_contents("dada".$input); //不可撸 ../../
   
   ?>
   ```

   ![image-20231115125016288](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231115125016288.png)

![](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231114100909886.png![image-20231115125026967](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231115125026967.png)

![image-20231114215028867](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231114215028867.png)



# **CSRF的原理**

**csrf攻击的是受害者的pc，帮助我们发起攻击的是访问⻚⾯的受害者的浏览器。**

**利用点：构造恶意参数诱骗用户 拿取凭据进行非授权操作**

![image-20231114213515515](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231114213515515.png)



## CSRF的利用

攻击者通过欺骗用户在不知情的情况下执行恶意操作，以利用用户已经在某个网站上进行身份验证的事实 以执行未经授权的操作

关键点 1.伪造数据包(burp可以生成POC) 2.无过滤防护/有过滤防护能饶过 3.受害者需要触发（诱惑）

### 常规的检测

 1.**同源策略** 的基本思想是一个页面的脚本只能访问与该页面具有相同源的资源，防止恶意网站窃取用户在其他网站的信息

2.**来源检测Referer字段****：** 来源检测是通过检查HTTP请求头中的Referer字段来验证请求的来源。Referer字段包含了发送请求的页面的URL。在CSRF防护中，服务器可以检查请求的Referer字段，确保请求来自合法的源，而不是从攻击者的网站发起的请求

### 严谨的检测

1.全部对比：服务器会对比请求中的所有可能来源，包括Referer字段、Origin头、自定义头部等，确保它们都与预期的来源一致

​	对抗点：结合XSS攻击或文件上传漏洞来绕过同源策略，比如通过XSS在受害者浏览器上执行脚本，或者上传	恶意文件，以触发伪造的请求，同时保证请求似乎来自同一来源

2.**不严谨的对比**：在不严谨的对比中，服务器可能只部分检查请求的来源，例如只检查域名而忽略协议和端口。这种情况下，攻击者可能找到绕过检测的方法

### 不严谨的CSRF检测：

1. **逻辑问题：** 在不严谨的检测中，可能忽略对逻辑问题的检测，导致攻击者可以利用某些逻辑漏洞绕过CSRF防护。比如检查 两字段其中一个

2. **无来源为True（请求中没有包含来源（Origin）头或者Referer字段，某些不严谨的检测机制可能会将这个请求视为合法）** 不严谨的检测容许请求不包含来源信息，从而容易受到CSRF攻击。

3. **双地址 ：** 不严谨的检测可能不会检查请求的URL格式，从而容许一些绕过手段，如双地址，通过使用两个不同的地址来实现攻击，伪装

   **原理同源策略允许某些跨域行为的限制，所以通过两个不同的地址来绕过浏览器的同源策略**

   ```
   比如https://example.com/change-password?redirect=http://malicious.com， 一个受攻击的网站（`https://example.com`）提供了一个更改密码的功能，同时接受一个名为 `redirect` 的查询参数作为重定向目标。攻击者可以构造一个恶意的URL，将 `redirect` 参数设置为恶意网站（`http://malicious.com`）的地址，当用户访问该URL并进行密码更改操作时，会将用户重定向到恶意网站
   ```

   

绕同源策略的技巧： 跨域资源共享（CORS）或代理服务器（同源检测是在浏览器端 代理是在服务器端）。发送跨域请求，从而绕过同源限制

# CSRF token

CSRFtoken令牌是一种用于增加网站安全性的防护措施。它通过在用户请求中包含一个唯一的令牌，来确保请求是由合法用户发起的，而不是通过 CSRF 攻击伪造的

利用技巧：

1. **复用：** 如果攻击者成功获取了某个用户的 CSRF 令牌，他们可以尝试将这个令牌用于伪造请求。有效的 CSRF 防护机制应该能够检测到令牌的重复使用，并拒绝这样的请求。
2. **删除：** 如果攻击者能够删除用户的 CSRF 令牌，就意味着在接下来的请求中，用户的请求将不包含正确的令牌，从而绕过了 CSRF 防护。（无法用token区分是正常请求还是恶意 ）因此，保护 CSRF 令牌免受删除攻击是重要的。
3. **置空：** 有时攻击者可能尝试将用户的 CSRF 令牌置空，即将其值设置为空。在下一次请求中，如果服务器未能检测到空值，可能会认为请求是合法的，从而造成安全漏洞。

# SSRF原理与攻击目标 

原理：web的服务器提供了向其他服务器请求资源的参数（协议），但未对用户的输入进行好过滤 **导致了恶意请求参数以web服务器为跳板，利用通信协议**对内部区域进行探针或攻击  （本地资源探测和内网资源探测 反带）

目标：SSRF攻击目标是外网无法访问的内部系统(正因请求是由服务端发起的，所以服务端能请求到与自身相连而与外网隔离的内部系统）对内部系统进行探针窥探 当内部某个组件存在缺陷的时候 我们就可以发布攻击





![image-20231114213535860](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231114213535860.png)



## 微服务与RPC架构让SSRF流行

首先 SSRF支持跨协议 从http/https到其他协议

其次 微服务和 RPC 架构的流行性增强了 SSRF 攻击，因为它们支持跨协议的攻击。



![image-20231114110210145](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231114110210145.png)

 

微服务和RPC（远程过程调用）的流行与软件开发和部署模式的演变有关

微服务架构是一种软件设计方法，将应用程序拆分为小的、独立的服务单元，这些服务单元可以独立开发、部署和扩展。

RPC是一种通信协议，用于在不同的计算机上执行程序的过程调用，

**微服务的流行原因：**

1. **灵活性和可扩展性：** 微服务允许团队根据需要独立开发和部署服务，使得系统更容易扩展和维护。
2. **技术多样性：** 不同的微服务可以使用不同的技术栈，选择适合其目标的最佳工具和语言。
3. **故障隔离：** 单个微服务的故障不会影响整个系统，提高了系统的鲁棒性

**RPC的流行原因：**

1. **简化分布式系统开发：** RPC允许远程调用，使得开发者可以像本地调用一样调用远程服务，简化了分布式系统的开发。
2. **性能：** 相对于其他通信方式，RPC通常具有较低的开销，使得它在性能敏感的应用中很有吸引力。

**微服务和RPC对SSRF漏洞的影响：**

1. **内部网络访问：** 微服务架构中的服务通常需要相互通信，而RPC是实现这种通信的一种方式。如果未正确配置和验证输入，攻击者可能通过SSRF漏洞从一个服务访问另一个服务，甚至是内部网络。

   **跨协议通信：** 微服务架构通常涉及多个服务之间的协同工作，这些服务可能使用不同的通信协议，例如 HTTP、RPC 或者其他自定义协议。攻击者可以通过构造特定的请求，跨协议地访问其他服务，从而绕过一些安全机制

   **RPC**

2. **本质是远程函数调用：** RPC是一种通信协议，它允许在网络上的不同计算机上执行远程过程调用。这意味着应用程序可以通过网络调用远程服务器上的函数，使得应用程序与其他系统或服务进行交互。

3. **直接指定函数和目标服务器：** 在构建RPC请求时，应用程序通常会直接指定要调用的函数和目标服务器的地址。这给了攻击者机会，如果应用程序在构建RPC请求时未进行适当的验证和过滤用户输入，攻击者可以通过构造恶意的请求来利用SSRF漏洞。

4. **未验证用户输入的危险性：** 如果应用程序在构建RPC请求时允许用户输入未经验证的数据，攻击者可能会通过在RPC请求中指定恶意的目标服务器地址或远程函数调用来触发SSRF攻击。攻击者可以尝试访问其本不应该访问的资源，包括内部网络资源。





# *SSRF的利用*

**curi支持的ssrf协议  28种 在代审时候 遇到某个ssrf漏洞是调用类似CURL的模块去进行访问的，以上协议都可以利用，**![image-20231115075958057](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231115075958057.png)列如PHP调用curl的时候

![image-20231115075928469](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231115075928469.png)

程序员的本意是获取我们输入指定的文件，但ssrf漏洞的存在，帮我们构造了⼀个内部环境的代理agent，通过agent，我们可以对内部环境 进⾏窥探，当内部某个组件存在缺陷的时候，我们就可以攻击。

```
<?php

$file = $_GET['file'];

$resp = file_get_contents($file);

echo $resp; ?>

?>
```

DICT  GOPHER

## 访问内部web

利用协议：http/https

最初级的ssrf利⽤了，通过ssrf去访问⼀个开启在127.0.0.1的web应⽤或者开放在内⽹的web应⽤

可能开放在内⽹的服务没有进⾏⽐较强的鉴权，那么就可能有⼀些撸点，常⻅的利⽤是，内⽹**存在其他可以直接rce的web服务**，可以先通过ssrf探测web指纹，来确定是 否存在特定应⽤。

当然也有⼀种利⽤⽅式是，直接对内⽹的web服务直接，盲打各种rce，也有许多常⻅案例。 这种利⽤⽅式在早期的⽹络攻防环境乃⾄现在的⽹络安全环境中，仍然很适⽤。原因是什么？

引申到⽹络安全体系的部署⽅式 边界网络安全  但内网中的服务往往认为自己处于安全的环境中，因此可能在安全措施上（如鉴权）做得不够严格，这就给了攻击者更多的机会

复杂的网络环境：大型组织的网络环境通常非常复杂，涉及多个子网、内部服务和应用程序。管理和保护这些网络资源可能存在挑战，导致一些内部Web服务没有得到适当的鉴权和访问控制

## 内⽹端⼝扫描

利用协议 任意 常规 http/https

这个与上⾯那个类似，利⽤的是⼀种差分攻击（差别分析攻击） 参考密码学

“端⼝开放情况下与端⼝不开放情况下”，得到返回的时间会差别很⼤。以此，可以⽤来判断端 ⼝是否开启



## ssrf转任意文件读取

file //⼏乎通⽤

ldap、zlib、phar、tar、rar //php

 jar //java

 jar war tar

通过ssrf对本地⽂件进⾏读取，可以⽤来做代码审计、或者配置⽂件读取、密码读取。等等， 也是⼀个⾮常好⽤的协议



## ssrf攻击内部应⽤

利用协议 dict  gopher

这两个协议也被称之为ssrf中的万⾦油协议了，因为他们都是封装协议（裸协议），可以⽤来 封装其他协议

其实就是通过dict或者gopher协议去构造其他应⽤的通信协议，与其他应⽤通信。然后攻击其 他应⽤

常被攻击的内部应用 redis   fastCGI/php-fpm

备注：POST 中断连接 Redis 



## ssrf的检测绕过

### **短网址配合SSRF**

攻击者首先会创建一个指向他们想要访问的内部资源的短网址。然后，他们将这个短网址发送到有SSRF漏洞的服务器。服务器会对这个短网址进行解析，实际上访问的是攻击者指定的内部资源，从而完成了一次SSRF攻击

```
**短网址配合SSRF：**

1. 首先，你发现目标服务器内部有一个管理界面，其URL是：http://192.168.1.1/admin。
2. 你不能直接从外部访问这个地址，但你可以通过SSRF漏洞让服务器自己去访问它。
3. 你通过一个短网址服务（例如bit.ly）将这个长URL转换成短URL，例如http://bit.ly/123。
4. 然后，你在有SSRF漏洞的地方输入这个短URL，服务器会解析这个短URL，然后访问http://192.168.1.1/admin
5. 通过这种方式，你就可以让服务器访问到内部的资源，实现了SSRF攻击。
```

### **自建网址让服务器访问：**

1.隐藏真实的攻击目标 

2.攻击者提供更多的控制可能

比如，攻击者可以在自建的网址上放置恶意代码，当服务器访问这个网址时，就会执行这些恶意代码

- 攻击者在`http://attacker.com`部署了一个恶意网站。
- 利用SSRF漏洞，攻击者让服务器访问`http://attacker.com/malicious`，从而触发恶意操作。

### 域名访问

1. 攻击者构造一个恶意请求，其中包含对`malicious-example.com`的访问。

2. 由于存在SSRF漏洞，攻击者利用SSRF漏洞在受害服务器上构造恶意请求，从而控制服务器发起对`malicious-example.com`或其他目标的实际请求。

3. 攻击者可能将`malicious-example.com`指向自己的恶意服务器，以获取敏感信息、执行远程命令等恶意操作。

   配合前面，脑补一下 如果malicious-example.com本身就是我的网址，并且做了短网址重定向，并且我在上面写入了DOM-XSS和CSRF的恶意代码，SSRF漏洞的服务器被我引导过来，那么后续的用户不也一样？  隐蔽 社工  信息窃取  三漏洞结合

   或者 网页使用您提供的一些信息自动创建 PDF，您可以**插入一些 JS，这些 JS 将在创建 PDF 时由 PDF 创建者**本身（服务器）执行，您将能够滥用 SSRF，利用XSS恶意代码（代码触发了一个向内部网络发起请求的SSRF漏洞。）生成PDF，让服务器解析 最终执行SSRF

   ```
   1. 在你的`evil.com`服务器上，你设置了一个DNS A记录指向你想要访问的内网IP，例如192.168.1.1（这是你无法直接访问的内部资源）。
   2. 在存在SSRF漏洞的地方，你输入`http://evil.com`。服务器将会解析这个域名，并实际上访问192.168.1.1。
   3. 通过这种方式，你就利用SSRF漏洞让服务器访问了内部资源。这种方法的优点是你可以随时改变DNS记录来访问不同的内部资源
   ```

由于域名的所有权和使用可能更难追踪，攻击者可以更容易地隐藏其真实身份和位置，增加了攻击的隐蔽性





## Python中的SSRF

在python中，ssrf并没有那么容易利⽤

### **urllib**

urllib是python⾃带的协议处理和流处理⼯具，也有许多程序⽤他来做http请求的客户端。但是 ⽇渐式微，主要是⽤起来太过麻烦

主要支持协议

![image-20231115081150501](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231115081150501.png)

这⾥有两个协议我们可以利⽤，⼀个是file协议，可以⽤来读取本地⽂件。 ⼀个是data协议，⽐较有趣⼀点，可以嵌⼊⼀些我们的任意输⼊，但是现在⽹络上似乎没有什 么利⽤点，这个⼤家可以研究下

### **requests**

requests是⽬前⽤到的最多的http动作库，基本代替了urllib在⽇常产品中的份额。

![image-20231115081238175](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231115081238175.png)

听过动调跟报错，我们可以看到，其实这⾥requests库默认只⽀持http跟https协议，这就注 定，他不能与php和java那样，⽅便的进⾏ssrf

如何查看一个库支持的协议类型

```
1.故意让程序报错

2.通过报错得到执⾏流

3.在关键流程打断点

4.关注断点的关键变量
```

## SSRF黑盒关注点

黑盒探针：  -业务功能点  1.社交分享功能：获取超链接的标题等内容进行显示  2.转码服务：通过 URL 地址把原地址的网页内容调优使其适合手机屏幕浏览  3.在线翻译：给网址翻译对应网页的内容  4.图片加载/下载：例如富文本编辑器中的点击下载图片到本地；通过 URL 地址加载或下 载图片  5.图片/文章收藏功能：主要其会取 URL 地址中 title 以及文本的内容作为显示以求一 个好的用具体验  6.云服务厂商：它会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的 信息，就可以进行 ssrf 测试  7.网站采集，网站抓取的地方：一些网站会针对你输入的 url 进行一些信息采集工作  8.数据库内置功能：数据库的比如 mongodb 的 copyDatabase 函数  9.邮件系统：比如接收邮件服务器地址  10.编码处理, 属性信息处理，文件处理：比如 ffpmg，ImageMagick，docx，pdf，

xml 处理器等  11.未公开的 api 实现以及其他扩展调用 URL 的功能：可以利用 google 语法加上这些 关键字去寻找 SSRF 漏洞   -

URL 关键参数  share  wap  url  link  src  source  target  u  display  sourceURl
