---
title: Web安全
date: 2017-09-05 09:19:25
tags: Web安全
---

所谓**攻击**，即黑客利用网站操作系统的漏洞和Web服务程序的SQL注入漏洞等得到Web服务器的控制权限，轻则篡改网页内容，重则窃取重要内部数据，更为严重的则是在网页中植入恶意代码，使得网站访问者受到侵害。

**常见的Ｗeb攻击技术：**
1. XSS（Cross-Site Scripting，跨站脚本攻击）：指通过在存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或JavaScript进行的一种攻击．**重点是跨域和客户端执行**
2. SQL注入攻击
3. CSRF（Cross-Site Request Forgeries，跨站点请求伪造）：指攻击者通过设置好的陷阱，强制对已完成的认证用户进行非预期的个人信息或设定信息等某些状态更新．
CSRF的核心就是请求伪造，通过伪造身份提交POST和GET请求来进行跨域的攻击。完成CSRF需要两个步骤：
- 登陆受信任的网站A，在本地生成cookie
- 在不登出A的情况下，或者本地cookie没有过期的情况下，访问危险网站B

## 1. XSS的原理
XSS其实就是Html的注入问题，攻击者的输入没有经过严格的控制进入了数据库，最终显示给来访的用户，导致可以在来访用户的浏览器里以浏览用户的身份执行Html代码，数据流程如下：攻击者的Html输入—>web程序—>进入数据库—>web程序—>用户浏览器。

## 2. XSS的攻击方式
1.反射型 Reflected XSS
发出请求时，XSS代码出现在URL中，作为输入提交到服务器端，服务器端解析后响应，XSS代码随响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。这个过程像一次反射，故叫反射型。
```js
router.get('/', function(req, res, next) {
  res.set('X-XSS-Protection',0);//禁止浏览器对XSS进行拦截
  res.render('index', { title: 'Express',xss:req.query.xss});//获取url中的参数做解析xss
});
```
```html
<body>
    <h1><%= title %></h1>
    <p>Welcome to <%= title %></p>
    <div class="">
    	<%- xss %> <!--执行xss脚本-->
    </div>
  </body>
```
浏览器地址栏输入格式如`http://localhost:3000/?xss=' '`

- 输入xss等于一个<img>标签，图片会显示在页面中  弹出1
![img攻击.png](http://upload-images.jianshu.io/upload_images/3188930-100348c70eed509f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 如果xss脚本是一iframe，可以植入广告
![iframe.png](http://upload-images.jianshu.io/upload_images/3188930-f09aee07a28cea33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.存储型 Stored XSS
存储型XSS和反射型XSS的差别仅在于，提交的代码会存储在服务器端(数据库、内存、文件系统等)，下次请求目标页面时不用再次提交XSS代码。
```js
  res.render('index', { title: 'Express',xss:sql()});//sql语句操作
```
3.基于DOM或本地的XSS攻击  DOM-based or local XSS
一般是提供一个免费的wifi，但是提供免费wifi的网关会往你访问的任何页面插入一段脚本或者直接返回一个钓鱼页面，从而植入恶意脚本。这种直接存在于页面，无须经过服务器返回就是基于本地的XSS攻击。

## 3. XSS的防御措施
1. 编码
对用户输入的数据进行HTML Entity编码
2. 过滤
移除用户上传的DOM属性，如onerror等
移除用户上传的Style节点、Script节点、Iframe节点等。
3. 校正
避免直接对HTML Entity解码
使用DOM Parse转换，校正不配对的DOM标签

## 4. 攻击目的和手段
攻击者使被攻击者在浏览器中执行脚本后，如果需要收集来自被攻击者的数据(如cookie或其他敏感信息)，可以自行架设一个网站，让被攻击者通过JavaScript等方式把收集好的数据作为参数提交，随后以数据库等形式记录在攻击者自己的服务器上。
1.  盗用 cookie ，获取敏感信息。
2. 利用植入 Flash ，通过 crossdomain 权限设置进一步获取更高权限;或者利用Java等得到类似的操作。
3. 利用 iframe、frame、XMLHttpRequest或上述Flash等方式，以(被攻击)用户的身份执行一些管理动作，或执行一些一般的如发微博、加好友、发私信等操作。
4. 利用可被攻击的域受到其他域信任的特点，以受信任来源的身份请求一些平时不允许的操作，如进行不当的投票活动。
5. 在访问量极大的一些页面上的XSS可以攻击一些小型网站，实现DDoS攻击的效果。
6. 破坏页面正常结构，插入恶意内容．

## 5. 模仿XSS攻击
https://github.com/BubbleM/xss_demo
