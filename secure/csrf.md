# CSRF攻击：跨站请求伪造

1、登陆网站A，并在本地生成Cookie

2、在不登出A的情况下，访问危险网站B


## 预防方法：
合理使用GET、POST方法。
不同的表单包含一个不同的伪随机值。
使用restful。
禁用CORS(cross-origin request) 跨域请求。

## laravel中CSRF验证原理：
1、开启session时，生成一个token值并存放在session中。
2、通过中间件verifyToken，先isreeding()判断请求是否CRSF验证，过滤掉自定义数组except中的请求。最后调用token match方法验证请求的token值与session中的token值是否匹配。

## 总结：
利用web中身份验证的漏洞，只保证请求事来自用户信任的浏览器，而非用户自愿发出的请求。


