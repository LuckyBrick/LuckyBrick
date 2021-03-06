# 选择HTTP状态码－让这件事变得简单
> 2015，12，4  michael kropat

还有比返回HTTP状态码更简单的事情吗?页面正常,那就返回200;页面不存在?那就返回404。

如果想重定向至另外一个页面，那就返回302，或者301。

我喜欢把HTTP状态码看作CB10码（这段谚语我不会翻译。。。）

到现在为止一切都很好，直到有一天有人告诉你，你做的事情一点儿也不符合REST风格。然后你就夜不能寐，绞尽脑汁考虑你的新资源的返回有没有遵从RFC协议（Roy-Fielding 推荐使用的状态码）。这里真的该返回200吗？还是它其实应该返回 204 no content? 不不不，这里绝对应该返回202 Accepted…又或是201 created？

让人纠结的是：官方的HTTP/1.1协议，那个RFC文档最初写于1997年。那个年代你还在用 Netscape浏览器，以33.6kps的网速上网。就好比非要将孙子兵法用于现代企业经营策略一样。永恒的忠告：（凡火攻有五）永远没法帮助我们做市场验证。

![pic](http://racksburg.com/wp-content/uploads/2015/12/win98-rfc2068-annotated.png)

如果，有一种决策树可以帮助迅速识别与相关的HTTP状态码，那我们就可以忽略掉无关项了，于许多状态码中选择出最合适的那一个了。

亲爱的，你可真幸运，这一天已经到来！
## 从何说起

![pic1](http://ww4.sinaimg.cn/mw690/867560c1jw1ezz54eqfc8j20o10dq763.jpg)

是不是非常明显呢？但是作者看到许多人在有关”这个返回是应该选择 503 service unavailable 还是404 not found”中纠结。停！如果你发现你总在不同的返回类中思忖返回何种http 状态码，就大错特错！再看看上面的流程图。

### 在此之前，有些注意事项：
¬	你完全可以不必听作者在扯什么，直接阅读RFC 7231 和 httpstatuses.com
¬	我更多得是想说给正在做符合REST风格的网站的用户听
⎫	返回正确的返回码对于网站来说锦上添花
⎫	略过了代理服务器相关的返回码
¬	作者将返回状态码归为三大类：
![pic1](http://ww4.sinaimg.cn/mw690/867560c1jw1ezz54atjn6j210c0e8q5x.jpg)

### 另外一个重要声明：
我比其他人知道得多一些只是因为我在一个每天都制造有用的API的地方工作，如果你认为我的说法是错的，或者是轻视了你喜爱的状态码，可以与我探讨告诉我我是如何大错特错的。

2XX/3XX

4XX

5XX

###  尾声：状态码为什么有用
其实我不能完全肯定他们有用。

在facebook有一群聪明的人，会写只返回200状态码的api。

反对费心思考特定状态码的观点是：现有的状态码对于现代化的网站和api太笼统了。为了客户的方便，返回的信息必须包含一些特定格式的详细信息（比如哪些字段验证失败了、为什么失败了），那么为什么要会花时间在冗余的、未为可用的状态码上？

当被追问为甚么使用这些特定的状态码很重要？一个常见的解释是：HTTP是一个分层系统，有意义的响应码会使所有在客户端和服务器之间的代理、高速缓存或HTTP库将更好地工作。我并不觉得这个说法令人信服，因为：当所有人都用HTTPS，已经禁止使用不在服务器控制下的代理和缓存了。

### 我将给出三个使用特定状态码的理由：
1.	客户端已经可以很容易的处理不同状态码：
Φ	301 moved permantly 和 302 Found google和其他搜索引擎已经对他们做了优化
Φ	301 moved permantly 默认缓存，429 Too Many Resuests 默认不缓存，等等
Φ	客户端可以支持429 Too Many Resuests暂不加载、在一定延迟后重试
Φ	客户端同样可以处理503 Service Unavilable
2.	尽管没有详尽的现代需求,许多状态码代表着一种值得特殊处理的情况（所以为什么不用标准状态码呢？）
Φ	如果Api返回404而不是405 Method Not Allowed 我就会想：“是我输入的url地址有误还是用错HTTP方法了呢？”；
Φ	如果有不与500 Internal Server Error 混淆的502 Bad Gateway（一种上游的问题），我们就节省了珍贵的调试找问题时间。
3.	现在，广泛使用的apis正逐步建立一种约定：返回类似201 Created, 429 Too Many Requests和 503 Service Unavilable。如果你遵循这个惯例,你的用户将更容易使用你的网站/ API、解决他们可能遇到的任何问题。

这一切中最困难的部分是决定返回哪些状态码，但正确的流程图使得选择一个有意义的代码变得容易得多。
