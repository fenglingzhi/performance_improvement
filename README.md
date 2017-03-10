# performance_improvement

原文： [Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html)

译文：雅虎前端优化35条规则翻译

译者：@creeperyang

如何让web页面更快，雅虎团队实践总结了7类35条规则，下面一一列出。

## 1. Content

#### 1.1 Make Fewer HTTP Requests
##### Minimize HTTP Requests减少/最小化 http 请求数。
到终端用户的响应时间80%花在前端：大部分用于下载组件（js/css/image/flash等等）。减少组件数就是减少渲染页面所需的http请求数。这是更快页面的关键。

减少组件数的一个方法就是简化页面设计。保持富内容的页面且能减少http请求，有以下几个技术：

* Combined files。合并文件，如合并js，合并css都能减少请求数。如果页面间脚本和样式差异很大，合并会更具挑战性。
* CSS Sprites。雪碧图可以合并多个背景图片，通过background-image 和 background-position 来显示不同部分。
* Image maps。合并多个图片到一个图片，一般用于如导航条。由于定义坐标的枯燥和易错，一般_不推荐_。
* Inline images。使用data:url scheme来內连图片。

减少请求数是为第一次访问页面的用户提高性能的最重要的指导。

#### 1.2 Reduce DNS Lookups

减少DNS查询。

就像电话簿，你在浏览器地址栏输入网址，通过DNS查询得到网站真实IP。

DNS查询被缓存来提高性能。这种缓存可能发生在特定的缓存服务器（ISP/local area network维护），或者用户的计算机。DNS信息留存在操作系统DNS缓存中（在windows中就是 DNS Client Serve ）。大多浏览器有自己的缓存，独立于操作系统缓存。只要浏览器在自己的缓存里有某条DNS记录，它就不会向操作系统发DNS解析请求。

IE默认缓存DNS记录30分钟，FireFox默认缓存1分钟。

当客户端的DNS缓存是空的，DNS查找次数等于页面中的唯一域名数。

减少DNS请求数可能会减少并行下载数。避免DNS查找减少响应时间，但减少并行下载数可能会增加响应时间。指导原则是组件可以分散在至少2个但不多于4个的不同域名。这是两者的妥协。

#### 1.3 Avoid Redirects

避免跳转。

跳转用301或302状态码来达成。一个301响应http头的例子：

```
HTTP/1.1 301 Moved Permanently
Location: http://example.com/newuri
Content-Type: text/html
```
浏览器自动跳转到Location指定的路径。跳转所需的所有信息都在http头，所以http主体一般是空的。301302响应一般不会被缓存，除非有额外的头部信息，比如Expires或Cache-Control指定要缓存。meta刷新标签或 JavaScript 也可以跳转，但如果真要跳转，3xx跳转更好，主要是保证返回键可用。

跳转显然拖慢响应速度。在跳转的页面被获取前浏览器没什么能渲染，没什么组件能下载。

最浪费的跳转之一发生在url尾部slash（/）缺失。比如[http://astrology.yahoo.com/astrology](http://note.youdao.com/)会301跳转到[http://astrology.yahoo.com/astrology/](http://note.youdao.com/)。这可以被Apache等服务器修复，用Alias，mod\_rewrite等等。

#### 1.4 Make Ajax Cacheable

让Ajax可缓存。

使用ajax的好处是可以向用户提供很快的反馈，因为它是向后台异步请求数据。但是，这些异步请求不保证用户等待的时间——异步不意味着瞬时。

提高ajax性能的最重要的方法是让响应被缓存，即在[Add an Expires or a Cache-Control](https://developer.yahoo.com/performance/rules.html#expires=) Header中讨论的 Expires 。其它方法是：

* gzip组件
* 减少DNS查找
* 压缩JS
* 避免跳转
* 设置ETags

#### 1.5 Post-load Components

延迟加载组件。

再看看你的页面然后问问自己，“什么是页面初始化必须的？”。剩下的内容和组件可以延迟。

JavaScript是理想的（延迟）候选者，可以切分到onload事件之前和之后。比如拖放的js库可以延迟，因为拖动必须在页面初始化之后。其它可延迟的包括隐藏的内容，折叠起来的图片等等。

#### 1.6 Preload Components

预加载组件。

预加载看起来与延迟加载相反，但它的确有个不同的目标。通过预加载你可以利用浏览器的空闲时间来请求你将来会用到的组件。这样当用户访问下一个页面时，你会有更多的组件已经在缓存中，这样会极大加快页面加载。

有几种预加载类型：
* 无条件预加载：一旦onload触发，你立即获取另外的组件。比如谷歌会在主页这样加载搜索结果页面用到的雪碧图。
* 有条件预加载：基于用户动作，你推测用户下一步会去哪里并加载相应组件。
* 预期的预加载：在发布重新设计（的网站）前提前加载。在旧网页预加载新网页的部分组件，那么切换到新网页时就不会是没有任何缓存了。

#### 1.7 Reduce the Number of DOM Elements

减少dom数。

一个复杂的页面意味着更多的内容要下载，以及更慢的dom访问。比如在有500dom数量的页面添加事件处理就和有5000dom数量的不同。

如果你的页面dom元素很多，那么意味着你可能需要删除无用的内容和标签来优化。

#### 1.8 Split Components Across Domains

把组件分散到不同的域名。

把组件分散到不同的域名允许你最大化并行下载数。由于DNS查询的副作用，最佳的不同域名数是2-4。

#### 1.9 Minimize the Number of iframes

最小化iframe的数量。

iframe允许html文档被插入到父文档。

<iframe>优点：
* 帮助解决缓慢的第三方内容的加载，如广告和徽章
* 安全沙盒
* 并行下载脚本

<iframe>缺点：
* 即使空的也消耗（资源和时间）
* 阻塞了页面的onload
* 非语义化（标签）

#### 1.10 No 404s

不要404。

http请求是昂贵的，所以发出http请求但获得没用的响应（如404）是完全不必要的，并且会降低用户体验。

一些网站会有特别的404页面提高用户体验，但这仍然会浪费服务器资源。特别坏的是当链接指向外部js但却得到404结果。这样首先会降低（占用）并行下载数，其次浏览器可能会把404响应体当作js来解析，试图从里面找出可用的东西。

## 2. Server

#### 2.1 Use a Content Delivery Network

使用CDN。

用户接近你的服务器会减少响应时间。把你的内容发布到多个，地理上分散的服务器可以让页面加载更快。但怎么开始？

首先不要试图把你的架构重新设计成分布式架构。因为可能引进更多复杂性和不可控。

记住80-90%的终端用户响应时间花费在下载页面中的所有组件：图片、样式、脚本、falsh等等。这是_Performance Golden Rule_。不要从困难的重新设计后台架构开始，最好首先分发你的静态内容。这不仅可以减少响应时间，用CDN还很容易来做。

CDN是一群不同地点的服务器，可以更高效地分发内容到用户。一些大公司有自己的CDN。

#### 2.2 Add an Expires or a Cache-Control Header

加Expires或者Cache-Control头部。

这条规则有两个方面：

* 对静态组件：通过设置Expires头部来实现“永不过期”策略。
* 对动态组件：用合适的Cache-Control头部来帮助浏览器进行有条件请求。
页面越来越丰富，意味着更多脚本，样式，图片等等。第一次访问的用户可能需要发出多个请求，但使用Expires可以让这些组件被缓存。这避免了访问子页面时没必要的http请求。Expires一般用在图片上，但应该用在所有的组件上。

浏览器（以及代理）使用缓存来减少http请求数，加快页面加载。服务器使用http响应的Expires头部来告诉客户端一个组件可以缓存多久。比如下面：

```
Expires: Thu, 15 Apr 2010 20:00:00 GMT //2010-04-15之前都是稳定的 
```
**注意**，如果你设置了Expires头部，当组件更新后，你必须更改文件名。

#### 2.3 Gzip Components

传输时用gzip等压缩组件。

http请求或响应的传输时间可以被前端工程师显著减少。终端用户的带宽，ISP，接近对等交换点等等没法被开发团队控制，但是，压缩可以通过减少http响应的大小减少响应时间。

从`HTTP/1.1`开始，客户端通过http请求中的`Accept-Encoding头`部来提示支持的压缩：

```
Content-Encoding: gzip
```

gzip一般可减小响应的70%。尽可能去gzip更多（文本）类型的文件。html，脚本，样式，xml和json等等都应该被gzip，而图片，pdf等等不应该被gzip，因为它们本身已被压缩过，gzip它们只是浪费cpu，甚至增加文件大小。

#### 2.4 Configure ETags

实体标记（Entity tags，ETag）是服务器和浏览器之间判断浏览器缓存中某个组件是否匹配服务器端原组件的一种机制。实体就是组件：图片，脚本，样式等等。ETag被当作验证实体的比最后更改`（last-modified）`日期更高效的机制。服务器这样设置组件的ETag：

```
HTTP/1.1 200 OK
Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
ETag: "10c24bc-4ab-457e1c1f"
Content-Length: 12195
```

之后，如果浏览器要验证组件，它用If-None-Match头部来传ETag给服务器。如果ETag匹配，服务器返回304：

```
GET /i/yahoo.gif HTTP/1.1
Host: us.yimg.com
If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
If-None-Match: "10c24bc-4ab-457e1c1f"
HTTP/1.1 304 Not Modified
```
ETag的问题是它们被构造来使它们对特定的运行这个网站的服务器唯一。浏览器从一个服务器获取组件，之后向另一个服务器验证，ETag将不匹配。然而服务器集群是处理请求的通用解决方案。

如果不能解决多服务器间的ETag匹配问题，那么删除ETag可能更好。

#### 2.5 Flush the Buffer Early

早一点刷新buffer（尽早给浏览器数据）。

当用户请求一个页面，服务器一般要花200-500ms来拼凑整个页面。这段时间，浏览器是空闲的（等数据返回）。在php，有个方法flush()允许你传输部分准备好的html响应给浏览器。这样的话浏览器就可以开始下载组件，而同时后台可以继续生成页面剩下的部分。这种好处更多是在忙碌的后台或轻前端网站可以看到。

一个比较好的flush的位置是在head之后，因为浏览器可以加载其中的样式和脚本文件，而后台继续生成页面剩余部分。

```
<!-- css, js -->
</head>
<?php flush(); ?>
<body>
<!-- content -->
```
#### 2.6 Use GET for AJAX Requests

ajax请求用get。

[Yahoo! Mail](https://login.yahoo.com/?.src=ym&.intl=us&.lang=en-US&.done=https%3a//mail.yahoo.com)团队发现当使用`XMLHttpRequest`，POST 被浏览器实现为两步：首先发送头部，然后发送数据。所以使用GET最好，仅用一个TCP包发送（除非cookie太多）。IE的url长度限制是2K。

POST但不提交任何数据根GET行为类似，但从语义上讲，获取数据应该用GET，提交数据到服务器用POST。

#### 2.7 Avoid Empty Image src

避免空src的图片。

空src属性的图片的行为可能跟你预期的不一样。它有两种形式：

1. html标签：`<img src="">`
2. js：`var img = new Image(); img.src = ""`;
两种都会造成同一种后果：浏览器会向你的服务器发请求。

* IE，向页面所在的目录发请求。
* Safari和Chrome，请求实际的页面。
* FireFox3及之前和Safari/Chrome一样，但从3.5开始修复问题，不再发请求。
* Opera遇到空图片src不做任何事。

#### 为什么这种行为很糟糕？

1. 由于发送大量的意料之外的流量，会削弱服务器，尤其那些每天pv上百万的页面。
2. 浪费服务器计算周期取生成不会被浏览的页面。
3. 可能会破坏用户数据。如果你在跟踪请求状态，通过cookie或其它，你可能会破坏数据。即使image的请求不会返回图片，但所有的头部数据都被浏览器读取了，包括cookie。即使剩下的响应体被丢弃，破坏可能已经发生。

这种行为的根源是uri解析发生在浏览器。RFC 3986 定义了这种行为，空字符串被当作相对路径，Firefox, Safari, 和 Chrome都正确解析，而IE错误。总之，浏览器解析空字符串为相对路径的行为被认为是符合预期的。

html5在_4.8.2_添加了对标签src属性的描述，指导浏览器不要发出额外的请求。

> The src attribute must be present, and must contain a valid URL referencing a non-interactive, optionally animated, image resource that is neither paged nor scripted. If the base URI of the element is the same as the document's address, then the src attribute's value must not be the empty string.

幸运的是将来浏览器不会有这个问题了（在图片上）。不幸的是，`<script src="">`和`<link href="">`没有这样的规范。

## 3 Cookie

#### 3.1 Reduce Cookie Size

http cookie的使用有多种原因，比如授权和个性化。cookie的信息通过http头部在浏览器和服务器端交换。尽可能减小cookie的大小来降低响应时间。

* 消除不必要的cookie。
* 尽可能减小cookie的大小来降低响应时间。
* 注意设置cookie到合适的域名级别，则其它子域名不会被影响。
* 正确设置Expires日期。早一点的Expires日期或者没有会尽早删除cookie，优化响应时间。

#### 3.2 Use Cookie-free Domains for Components

用没有cookie的域名提供组件。

当浏览器请求静态图片并把cookie一起发送到服务器时，cookie此时对服务器没什么用处。所以这些cookie只是增加了网络流量。所以你应该保证静态组件的请求是没有cookie的。可以创建一个子域名来托管所有静态组件。

比如，你域名是`www.example.org`，可以把静态组件托管在`static.example.org`。不过，你如果把cookie设置在顶级域名`example.org`下，这些`cookie`仍然会被传给`static.example.org`。这种情况下，启用一个全新的域名来托管静态组件。

另外一个用没有cookie的域名提供组件的好处是，某些代理可能会阻止缓存待cookie的静态组件请求。

## 4. CSS

#### 4.1 Put Stylesheets at the Top

把样式放在顶部。

研究雅虎网页性能时发现把样式表移到`<head>`里会让页面更快。这是因为把样式表移到`<head>`里允许页面逐步渲染。

关注性能的前端工程师希望页面被逐步渲染，这时因为，我们希望浏览器尽早渲染获取到的任何内容。这对大页面和网速慢的用户很重要。给用户视觉反馈，比如进度条的重要性已经被大量研究和记录。在我们的情况中，`HTML`页面就是进度条。当浏览器逐步加载页面头部，导航条，logo等等，这些都是给等待页面的用户的视觉反馈。这优化了整体用户体验。

把样式表放在文档底部的问题是它阻止了许多浏览器的逐步渲染，包括IE。这些浏览器阻止渲染来避免在样式更改时需要重绘页面元素。所以用户会卡在白屏。

[HTML](https://www.w3.org/)规范清楚表明样式应该在`<head>`里。

#### 4.2 Avoid CSS Expressions

避免CSS表达式。

CSS表达式是强大的（可能也是危险的）设置动态CSS属性的方法。IE5开始支持，IE8开始不赞成使用。例如，背景颜色可以设置成每小时轮换：

```
background-color: expression( (new Date()).getHours()%2 ? "#B8D4FF" : "#F08A00" );
```

CSS表达式的问题是它们可能比大多数人预期的计算的更频繁。它们不仅在页面载入和调整大小时重新计算，也在滚动页面甚至是用户在页面上移动鼠标时计算。比如在页面上移动鼠标可能轻易计算超过10000次。

要避免CSS表达式计算太多次，可以在它第一次计算后替换成确切值，或者用事件处理函数而不是CSS表达式。

#### 4.3 Choose `<link>` over `@import`

选择`<link>`而不是`@import`。

之前的一个最佳原则是说CSS应该在顶部来允许逐步渲染。

在IE用`@import`和把CSS放到页面底部行为一致，所以最好别用。

#### 4.4 Avoid Filters

避免使用（IE）过滤器。

IE专有的`AlphaImageLoader`过滤器用于修复IE7以下版本的半透明真彩色PNG的问题。这个过滤器的问题是它阻止了渲染，并在图片下载时冻结了浏览器。另外它还引起内存消耗，并且它被应用到每个元素而不是每个图片，所以问题（的严重性）翻倍了。

最佳做法是放弃`AlphaImageLoader`，改用PNG8来优雅降级。

## 5. JavaScript

#### 5.1 Put Scripts at the Bottom

把脚本放到底部。

脚本引起的问题是它们阻塞了并行下载。[HTTP1.1](https://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.1.4)规范建议浏览器每个域名下不要一次下载超过2个组件。如果你的图片分散在不同服务器，那么你能并行下载多个图片。**但当脚本在下载，浏览器不会再下载其它组件，即使在不同域名下。**

有些情况下把脚本移动到底部并不简单。比如，脚本中用了`document.write`来插入内容，它就不能被移动到底部。另外有可能有作用域问题。但大多数情况，有方法可以解决这些问题。

一个替代建议是使用异步脚本。defer属性表明脚本不包含`document.write`，是提示浏览器继续渲染的线索。不幸的是，Firefox不支持。如果脚本能异步，那么也就可以移动到底部。

#### 5.2 Make JavaScript and CSS External

使用外部JS和CSS。

这里的很多性能规则涉及外部组件怎么管理。但你首先要明白一个基本问题：JS和CSS是应该包含在外部文件还是內连在页面本身？

真实世界中使用外部文件一般会加快页面，因为JS和CSS文件被浏览器缓存了。內连的JS和CSS怎在每次HTML文档下载时都被下载。內连减少了http请求，但增加了HTML文档大小。另一方面，如果JS和CSS被缓存了，那么HTML文档可以减小大小而不增加HTTP请求。

核心因素，就是JS和CSS被缓存相对于HTML文档被请求的频率。尽管这个因素很难被量化，但可以用不同的指标来计算。如果网站用户每个session有多个pv，许多页面重用相同的JS和CSS，那么有很大可能用外部JS和CSS更好。

许多网站用这些指标计算后在中间位置。对这些网站来说，最佳方案还是用外部JS和CSS文件。唯一例外是內连更被主页偏爱，如**http://www.yahoo.com/。主页每个session可能只有少量的甚至一个pv，这时候內连可能更快。**

对多个页面的首页来说，可以通过技术减少（其它页面的）http请求。在首页用內连，初始化后动态加载外部文件，接下来的页面如果用到这些文件，就可以使用缓存了。

#### 5.3 Minify JavaScript and CSS

压缩JS和CSS。

压缩就是删除代码中不必要的字符来减小文件大小，从而提高加载速度。当代码压缩时，注释删除，不需要的空格（空白，换行，tab）也被删除。

混淆是对代码可选的优化。它比压缩更复杂，并且可能产生bug。在对美国top10网站的调查，压缩可减小21%，而混淆可减小25%。

除了外部脚本和样式，內连的脚本和样式同样应该被压缩。

#### 5.4 Remove Duplicate Scripts

删除重复的脚本。

在页面中引入相同的脚本两次会伤害性能。可能超出你的预料，美国top10网站的2家有重复脚本引入。两个主要因素造成同一页面引入相同脚本：团队大小和脚本数量。当确实引入重复脚本，会发出不必要的http请求和浪费js执行时间。

发出不必要的http请求发生在IE而不是Firefox。在IE，如果外部脚本引入两次且没有缓存，它会发出2个请求。即使脚本被缓存，刷新时也会发出额外请求。

除了增加http请求，时间被浪费在执行脚本多次上。不管IE还是Firefox都会执行多次。

一种避免多次引入脚本的方法是在模板系统实现一个脚本管理模块。

#### 5.5 Minimize DOM Access

最小化DOM访问。

用JS访问DOM元素是缓慢的，所以为了响应更好的页面，你应该：

* 缓存访问过的元素的引用
* 在DOM树外更新节点，然后添加到DOM树
* 避免用JS实现固定布局

#### 5.6 Develop Smart Event Handlers

开发聪明的事件处理

有时候页面看起来不那么响应（响应速度慢），是因为绑定到不同元素的大量事件处理函数执行太多次。这是为什么使用_事件委托_是一种好方法。

另外，你不必等到`onload`事件来开始处理DOM树，`DOMContentLoaded`更快。大多时候你需要的只是想访问的元素已在DOM树中，所以你不必等到所有图片被下载。

## 6 Images

#### 6.1 Optimize Images

优化图片

在设计师建好图片后，在上传图片到服务器前你仍可以做些事：

* 检查gif图片的调色板大小是否匹配图片颜色数。
* 可以把gif转成png看看有没有变小。除了动画，gif一般可以转成png8。
* 运行`pngcrush`或其它工具压缩png。
* 运行`jpegtran`或其它工具压缩jpeg。
* 

#### 6.2 Optimize CSS Sprites

优化CSS雪碧图

* 把图片横向合并而不是纵向，横向更小。
* 把颜色近似的图片合并到一张雪碧图，这样可以让颜色数更少，如果低于256就可以用png8.
* "Be mobile-friendly"并且合并时图片间的间距不要太大。这对图片大小影响不是太大，但客户端解压时需要的内存更少。100×100是10000个像素，1000×1000是1000000个像素。

#### 6.3 Don't Scale Images in HTML

不要在html中缩放图片

不要因为你可以设置图片的宽高就去用比你需要的大得多的图片。如果你需要

```
<img width="100" height="100" src="mycat.jpg" alt="My Cat" /> 
```
那么，就用100x100px的图片，而不是500x500px的。

#### 6.4 Make favicon.ico Small and Cacheable

favicon.ico小且缓存

favicon.ico是在你服务器根路径的图片。邪恶的是即使你不关心它，浏览器仍然会请求它。所以最好不要响应404。另外由于在同一服务器，每次请求favicon.ico时也会带上cookie。这个图片还会影响下载顺序，比如在IE，如果你在`onload`时下载额外的组件，fcvicon会在这些组件之前被下载。

怎么减轻favicon.ico的缺点？

* 小，最好1K以下
* 设置Expires头部。也许可以安全地设置为几个月。

## 7 Mobile

#### 7.1 Keep Components under 25K

保持组件小于25K

这个限制与iPhone不缓存大于25K的组件相关。注意，这是非压缩（uncompressed）的文件大小。在这里minification（压缩，不要与compress混淆）很重要，因为gzip无法满足（iPhone）。

#### 7.2 Pack Components into a Multipart Document

打包组件到一个多部父文档

打包组件到一个多部父文档类似于带附件的邮件。它帮助你在一个http请求中获取多个组件，但注意，iPhone不支持。



