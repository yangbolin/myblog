title: iframe在IE下面session失效了
date: 2014-04-19 18:13:16
tags: web
categories: web开发
---

####一.问题
记得之前做一个项目的时候，把我们自己web系统的中的一个表单提交页面嵌入到其他产品线web系统的一个页面中，在IE浏览器下面出现表单提交失败的场景，查看日志是由于csrfToken失效导致。

<!-- more -->

把上面的问题抽象一下，系统A中有一个页面a,以iframe的方式嵌入到系统B的b页面中，a页面中有一个form表单，此时用户访问b页面后，然后提交表单，在IE浏览器下面表单没法提交，这里的IE浏览器指所有IE系列的浏览器。

####二.分析
通过后台日志分析，发现是csrfToken失效导致，因此我们怀疑在b页面中提交表单后，session中不存在csrfToken导致，debug后事实的确如此。然后我们把a页面从b页面中剥离出来后，发现一切正常，此时只有iframe值得怀疑了，网上搜索一把，发现在IE浏览器中iframe存在session/cookie丢失的问题，因为a页面对b页面来说是第三方网站，IE限制存储第三方网站的session和cookie，当然这个限制可以在浏览器客户端进行设置，把这个限制去除掉，不过我们不能要求用户这么去做。网上搜索的结果也告诉我们相关的解决方案，需要我们在服务端设置一下Servlet的response即可，具体解决办法如下：
```java
httpServletResponse.addHeader("P3P","CP='CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR'");
```
在后台java代码中获取iframe请求的response，然后增加P3P的头，关于P3P可以参考百度百科的解释，这里就不详细介绍了。
[P3P介绍](http://baike.baidu.com/view/722330.htm)

####三.聊聊CSRF攻击以及csrfToken的原理
* CSRF攻击介绍
下面这张图详细描述了CSRF攻击的过程
![CSRF攻击过程](http://bolinyoung.qiniudn.com/csrf.png)
第4第5就是所谓csrf攻击。举个简单的例子，假如xx银行提供了一个http的get请求，这个http的get请求可以修改用户银行账户余额，用户用自己的账号登录了xx银行的web系统，然后再登录一个不可信的钓鱼网站，此时这个钓鱼网站模拟用户发出一个修改用户银行余额的http请求，这样用户账户余额就被钓鱼网站给修改了，当然这只是举一个简单的例子，银行系统肯定不会这么干的，除非...此处略去N个字。

* csrfToken原理
为了防止上面描述的csrf攻击，我们可以在页面上生成一个csrfToken,然后把这个csrfToken也写入到session，浏览器发起get/post请求的时候必须要带上这个csrfToken，不然就认为请求不合法，同时要是get/post请求中带了这个token我们就去和session中存储的token进行比较，要是一致我们才做相关的事情。那么此时我们很容易想到要是钓鱼网站也知道了这个token的话，就一样可以模拟用户发起get/post请求了，不过这个token一般是很难获取到的，这个token只对用户是可见的，钓鱼网站很难获取到这个token，当然这个办法不是万能的。
