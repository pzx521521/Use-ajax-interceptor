# Use-ajax-interceptor
# How to Use ajax-interceptor
# 背景简介
+ 随着Web应用/网页的不断增加, 随着[restful](https://www.runoob.com/w3cnote/restful-architecture.html)的发展, springboot的实现简单, 以及不注意加密等等因素影响,客户端越来越容易模拟数据
# 举例:
+ pandownload 等通过加载httpClient模拟浏览器登录, 记录Cookie以实现再次打开免登陆的操作
+ [修改一些网页展示数据-> 原作者示例](https://juejin.im/post/5c8286855188257fdd0bd08f)
+ 前端开发在后台没有做好的时候, 终于不用读取本地json文件.到时候在修改文件了.也不用特地叮嘱后台跨域了, 后台也不用修改跨域的代码了
+ 嗯...  还是很鸡肋的
# 什么是 [ajax-interceptor](https://github.com/slorber/ajax-interceptor)
+ 可以拦截页面上的 ajax 请求，并把返回结果替换成任意文本。它对 mock数据、排查一些线上问题等会有很大帮助。（当然 chales 等抓包软件也可以做到，然而使用起来比较繁琐，做成 chrome 插件的形式会方便许多）
# 实现原理
+ 页面加载时往页面上注入 js 代码，这段 js 会生成一个 XMLHttpRequest 的代理对象，并把 window.XMLHttpRequest 替换成这个对象。该对象又会对 onreadystatechange 和 onload 两个回调函数做特殊处理，在它们执行时把 responseText 和 response 的值覆盖为你设置的值。其它就是这段页面上的 js 代码和右侧弹出的 iframe (用户界面)以及插件后台代码的一些数据交互和存储上的实现，这部分比较杂就不介绍了([引用了作者的原话](https://github.com/slorber/ajax-interceptor))
#作者 ajax-interceptor 的局限性
+ 仅适用于单post包含要修改的全部数据(无法仅仅修改部分返回数据)
+ 没有办法携带Cookie拦截(即设置任意cookie到本地)
# 给我们的提示
+ 做这种ajax/fetch(前端术语)其实是restful接口的时候, 重要数据一定要进行验证!!!!!!!!!!!!!!!!!!!!!!!!!!
+ 验证的方法有以下几种: restful请求中一定要  jwt/cookie/token加密 等等, 注意一定要加时间加密, 这样即使带参传过了时间也会失效
+ 验证的原理:
都是通过后台加密, 把加密后的验证串(以下统称token)写到前端, 每次前端请求的时候都带上token-> 到后台验证
**所有的前端加密都是不靠谱的**


#ajax-interceptor 的安装使用教程
+  [在这里](https://github.com/slorber/ajax-interceptor)下载到源码并解压
![在这里下载到源码](https://i.loli.net/2019/04/12/5cb090b05bd7a.jpg)
+ 在Chrome中加载这个插件(当然如果你可以**翻** **墙**可以直接去[chrome商店](https://link.juejin.im/?target=https%3A%2F%2Fchrome.google.com%2Fwebstore%2Fdetail%2Fajax-interceptor%2Fnhpjggchkhnlbgdfcbgpdpkifemomkpg)中下载)
![在Chrome中加载这个插件](https://i.loli.net/2019/04/12/5cb091f492a72.jpg)
+ 添加拦截规则
![添加拦截规则](https://i.loli.net/2019/04/12/5cb09275a4b8b.jpg)
#最最重要的 - - 
+ 上面写那么多其实都是废话.....既然你看到这里了, 我就告诉你-> 我的目的其实是去看小电影
小电影上有个post去验证返回结果是false, 不让我看= =, 我就用这个改为了然后可以看了, 然后感觉还不错= = 
分享一下...

 
#ajax-interceptor 的使用示例:
+ [示例网站](http://javbus.91thd.vip/)
+ 分析 每次跳转播放后都会进行验证-> 验证不通过就不让播放
+ 拦截这两个地址
```
http://javbus.91thd.vip/isLogin
http://javbus.91thd.vip/isValid
```
结果改为```true```即可, 如上图 添加拦截规则

![带cookie验证](https://i.loli.net/2019/04/12/5cb09f62e2440.jpg)
![返回结果](https://i.loli.net/2019/04/12/5cb09f630dd5b.jpg)
# 分析:
+ 上面确实带cookie 验证了, 但是返回结果太简单了, 没有包含如视频地址等等, 加密的关键信息
+ 想了一下, 确实没有好的办法
+ 其实是他设计的有问题
+ 先说下他的原理
  + 获取到视频的m3u8地址, 然后有播放器控制是否播放-> 这本身就是有问题的, 因为播放器是前端的, 修改起来其实挺方便的
  + 其实他是对m3u8地址内容进行了加密了, 用其他m3u8是解析不出来的
  + 理论上应该会员才能获取到m3u8地址, 这才是正确的方式
  + 但是由于他要给试看的功能, 这就很尴尬了,通常网站是放30s的任意观看的地址, 完整视频, 通过会员的cookie(用户名密码)进行重新获取, 但是这个网站省懒或者说为了给观看者更好的体验?然后采用了这种有问题的方式.................
  + 非要实现这种效果, 也是可以的,正确的方式是播放器是获取不同视频段的时候返回地址是用token验证
  + 其实也挺难的,因为 m3u8 是一个固定的格式, 固定的解析....网上通用的一套没办法使用, 还是觉得m3u8地址暴漏了就完了 主要是没有技术搞一套自己的播放器吧,.为了搞一套新的技术给用户体验, 却懒得/技术不够 去实现这个东西



