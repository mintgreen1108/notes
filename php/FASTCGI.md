GI 和 FASTCGI 小结：

在看关于队列的知识，发现讲述队列博客的开始提到一个名词fastCGI。fastCGI记得好像只有在配置 nginx 服务器的时候看到过，所以目前对这个知识点一无所知，所以翻阅了几片>博客好像读懂了点，先做个小结。以后有更深入的了解再补充。

## 基本概念

CGI：wiki上写到是描述服务器(web server)和请求处理程序(php 解释器)间的一种标准。>按我的理解，通俗来讲，就是规定了 web server 发送给 php 解释器数据的格式和PHP解释器返回给浏览器的数据格式。

fastCGI: CGI的增强版。

## 执行原理


