---
title:  Huginn 指南：为任意网站制作 RSS
date: 2022-05-08
lastmod: 2022-05-08
description: 又一篇介绍使用 Huginn 制作 RSS 的教程🕶️
categories:
- 技术
- 指南
tags:
- Huginn
- RSS
- 爬虫
- CSS
- 技术
---

<!-- # Huginn 指南：为任意网站制作 RSS -->

Huginn 使用多个不同功能的 Agent 组合搭配来实现一系列功能，一个 Agent 可以执行特定的操作，并产生一个 Event，你可以指定他产生的 Event 由哪个 Agent 接收处理。

比如我们需要使用 WebSite Agent 来爬取某个网站上的列表内容，爬取后列表的每一项会生成一个 Event，我们可以指定一个 Output Agent 来接收这些 Event，并将其转换为 RSS 的格式进行输出，复制 Output Agent 返回给你的 URL 就可以进行订阅啦，Huginn 爬取网站生成 RSS 的基本原理就是这样。

## Huginn 爬取普通网页制作 RSS



右键网页打开开发者工具，屏幕会分出一部分空间显示开发者工具窗口，点击左上角的按钮再把鼠标移动到页面上可以选择页面的某一个元素，比如这里我们要爬取推荐文章列表，推荐文章列表的每一项都有同样的样式，我们可以使用 CSS 选择器来指定爬取该项

![image-20220508160659378](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207138.png)



该元素可以用 CSS 选择器`.meiwen`选择到。

**Tips:** class=xxx 则使用`.xxx`,如果为 id=xxx 则使用`#xxx`

该标签是 a 标签，我们可以继续限定一下条件，使用`a.meiwen`

如果要限制必须是 h2 标签下的带有 meiwen 的 class 属性的 a 标签，可以使用`h2 a.meiwen`

如果要再加一个限制，是列表元素 li 下面的 h2 标签里带有 meiwen 的 class 属性的 a 标签，可以使用`li h2 a.meiwen`

添加其他附加限定条件以此类推

你也可以在右侧 Style 面板里点击加号添加一个自定义样式，输入 CSS 选择器，浏览器会自动高亮符合条件的网页元素，你可以使用这个功能来检验你写的 CSS 选择器是否正确以及是不是提取的你想要的内容

![image-20220508163359806](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207716.png)





接下来在 Huginn 里创建一个 Scenario，然后点击新建一个 Agent，类型选择 WebSite Agent，填写一个名称，并在最下面的配置处指定你要爬取的网页元素

> ###### **Website Agent**
>
> The Website Agent scrapes a website, XML document, or JSON feed and creates Events based on the results.

```json
url: {
	css: li h2 a.meiwen
	value: @href
}
```

以爬取 url 为例，css 表示使用 css 选择器选择网页元素，value 表示从哪里获取对应的值，对于刚才我们选取的一个元素来说

```html
<a class="meiwen" href="/article/58030.html" title="缘起则聚，缘灭则散">缘起则聚，缘灭则散</a>
```

href 属性里面的内容是他的链接，title 属性里面的内容则是他的标题。接下来在 value 中填写`@href`指定从 href 属性提取内容，就可以取到链接地址了

> **Pro Tips:**
>
> 如果是纯文本节点或是想提取标签内嵌的文本，value 里可以填写`string(.)`
>
> 如果想删除多余的空格，可以使用`normalize-space(.)`

> **Pro Tips 2:**
>
> 字符串处理函数和标签属性值变量可以一起使用，如`normalize-space(@title)`可以获取该标签的 title 属性值并删除多余的空白字符

![image-20220508163714342](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207717.png)

接下来点击 Dry Run 按钮进行测试，不出意外我们会得到一个 json 的输出，里面包括我们爬取到的每一项他的 url 和 title。

如果没有成功，你可能需要删掉上面没有使用到的 hovertext 节点，因为该项指定的内容在我们刚才的网页中并不存在。

![image-20220508164259295](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207718.png)



点击 Save 保存，我们开始下一步输出 RSS。在刚才的 Scenario 中再新建一个 Agent，选择 Data Output Agent，并设置他的名字和 Source Agent

> ###### Data output Agent
>
> The Data Output Agent outputs received events as either RSS or JSON. Use it to output a public or private stream of Huginn data.

![image-20220508164941647](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207719.png)



**Propagate immediately**是指即时处理来自 Source Agent 的 Event，启用他方便我们调试，但会略微增加服务器负载，你可以自行决定是否使用。

![image-20220508165345792](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207720.png)

在 secret 字段中为你的这个 RSS 标注一个英文的名字，修改 title 字段标注你的 RSS 的名字。item 字段是每条文章会有的属性，一般来说主要就 title 和 link，分别设置为上文我们提取的值的变量名。这里添加一个 guid 字段，这是一篇文章的唯一标识符，避免 RSS 阅读器读到的文章标题不同但是内容相同，常见于某篇文章的标题被修改，这会导致 RSS 阅读器内出现多篇重复文章。

此外建议增加一个 link 字段，值设置为与爬取的网站的主域名一致，避免网站内使用相对链接开头的资源无法正常加载。

![image-20220508170038722](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207721.png)



![image-20220508170115192](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207722.png)

点击 Save 保存，回到 Scenario 界面，第一次需要手动点击运行一下刚才的 Website Agent。稍等片刻后台会进行爬取，右上角会显示产生了多少个 Event。再点开刚才设置的 Data Output Agent 查看详情，vola！右侧就会显示生成的 RSS 链接了，复制以 xml 结尾的链接到 RSS 阅读器中就可以订阅啦🎉

![image-20220508170212112](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207723.png)





如果你的配置正确但是 WebSite Agent 又没有输出任何内容的话，那么你要爬取的站点的内容应该是通过 JavaScript 在你带开浏览器时动态生成的。

> **Pro Tips:**
>
> 你可以右键网页在开发人员菜单中点击显示源代码，然后用 Ctrl+F 搜索你在页面上见到的想爬取的某一条内容，如果有说明该网站的页面时静态的，否则说明你要爬取的内容并不是静态的。

要爬取这样的站点，我们需要模拟浏览器操作，执行页面上的 JavaScript 脚本，待数据生成完毕后再使用上面的方法去爬取内容。要模拟浏览器操作，这里介绍两个方案：一个是利用 browserless 的无头浏览器，另一个是使用在线 PhatomJS 服务。



## 使用 Huginn+Browserless 为动态网页生成 RSS



部署了[RSS Man X](https://github.com/Colin-XKL/RSSmanX)的完整版的话默认已经包含 huginn 和 browserless 服务，无需重复安装。且 huginn 可以通过`service.browserless:3000`访问到 browserless 服务。

总体步骤与上文一样，不过因为被抓取的内容是通过 JavaScript 动态生成的，所以直接使用 WebSite Agent 是抓取不到东西的，我们需要先使用 Post Agent 向 browserless 服务发送指令，让他帮我们模拟浏览器操作，再将最终完整的网页返回给我们，再使用 WebSite Agent 进行后续处理。

> ###### **Post Agent**
>
> A Post Agent receives events from other agents (or runs periodically), merges those events with the Liquid-interpolated contents of payload, and sends the results as POST (or GET) requests to a specified url. To skip merging in the incoming event, but still send the interpolated payload, set no_merge to true.

接下来以[这个网站](https://pccz.court.gov.cn/pcajxxw/pcgg/ggdh?lx=0)为例介绍一下，这个列表页的内容是由 JavaScript 动态生成的

![image-20220508205159783](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207724.png)

按照以下内容设置你的 Post Agent

```json
{
  "post_url": "http://service.browserless:3000/content",
  "expected_receive_period_in_days": "1",
  "content_type": "json",
  "method": "post",
  "payload": {
    "url": "https://pccz.court.gov.cn/pcajxxw/pcgg/ggdh?lx=0"
  },
  "headers": {
    "Cache-Control": "no-cache",
    "Content-Type": "application/json"
  },
  "emit_events": "true",
  "no_merge": "false",
  "output_mode": "clean"
}
```

其中`post_url`为你的 browserless 实例地址，如果你使用了 RSS MAN X 里的 Huginn 可以直接向上面一样填写。`payload`中的`url`字段填写你需要的网页地址。注意 emit_events 要设置为`true`，这样才方便我们后续使用 WebSite Agent 操作。

点击 Dry Run，如果能返回一个带有`body`字段且里面有文本内容说明调用成功。

调用不成功检查一下配置，以及是不是我们的爬虫被目标网站拦截了。若是爬虫被拦截可参考文末的解决方案。

![image-20220508195731732](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207725.png)

接下来点击 Save 保存，再新建一个 Website Agent，Source 设置为刚才的 Post Agent。

```json
{
  "expected_update_period_in_days": "2",
  "data_from_event": "{{ body }}",
  "type": "html",
  "mode": "on_change",
  "extract": {
    "url": {
      "css": "ul#wslb li a",
      "value": "normalize-space(@href)"
    },
    "title": {
      "css": "ul#wslb li a",
      "value": "@title"
    },
    "author": {
      "css": "ul#wslb li div.center",
      "value": "normalize-space(.)"
    }
  }
}
```



注意修改`data_from_event`的值，其他地方与爬取普通网站一样。再新建并配置一下 Output Agent，RSS 的链接就出来了

![image-20220508205856407](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207726.png)



## 使用 PhantomJS 为动态页面生成 RSS

首先需要到[Phantom Js Cloud 的官网](https://phantomjscloud.com)注册一个账户，每个账户官方提供了一定的免费额度，每天大概可以爬取 500 个页面，对于做 RSS 来说妥妥的够用了。注册之后可以得到一个 API KEY，这个待会会使用到。

接下来新建一个 Phantom JS Cloud Agent，填写基本的名称、API KEY 和目标 URL，render type 选择 html。底下 User Agent 建议改用真实的浏览器 UA，减少触发反爬虫的可能性。如果不知道自己的 UA 可以到这个网站查看当前浏览器的 UA，复制字符串即可。如果懒得查也可以用这个

```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.81 Safari/537.36
```

> ###### Phantom JS Cloud Agent
>
> This Agent generates PhantomJs Cloud URLs that can be used to render JavaScript-heavy webpages for content extraction.
>
> URLs generated by this Agent are formulated in accordance with the PhantomJs Cloud API. The generated URLs can then be supplied to a Website Agent to fetch and parse the content. Sign up to get an api key, and add it in Huginn credentials.

点击 Dry Run 后会返回一条 url，返回格式类似这样

```json
[
  {
    "url": "https://phantomjscloud.com/api/browser/xxxxxxx"
  }
]
```

之后创建一个 WebSite Agent，Source Agent 设置为上面的 Phantom JS Cloud Agent，然后稍微改一下 url 的部分

```json
{
  "expected_update_period_in_days": "2",
  "url_from_event": "{{url}}",
  "type": "html",
  "mode": "on_change",
  "extract": {...}
}  
```

注意将原来的`url:??`的部分更改为`"url_from_event": "{{url}}"`,这样就指定使用 Phantom JS Cloud 为我们获取的完整网页，接下来的操作就大同小异了。配置好要爬取的字段和规则后点击 Dry Run 就可以看到结果

![image-20220508215636780](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207727.png)



这里是刚才的 Agent 产生的一个 Event。这里除了 url 和 title 我还提取了 date 字段。在 Output Agent 中，可以配置为每一条文章输出发布时间的信息。对于这种规整的日期字母串，可以使用`{{ date | date: %F }}`进行解析并格式化输出为 rss 规范的时间格式如果没有配置的话，每篇文章的发布时间显示的都是一样的，为 Huginn 爬虫更新的时间。

```json
"item": {
      "title": "{{title}}",
      "link": "{{url}}",
      "pubDate": "{{ date | date: %F }}"
    },
```

> **Pro Tips:**
>
> Huginn 解析日期
>
> 注意`{{ date | date: %F }}`里面，第一个`date`是上面的 Website Agent 产生的 Event 里面的变量名，第二个`date`是 Huginn 使用的 Liquid 模版引擎语法里内置的一个 filter，可以理解为有特定功能的函数





![image-20220508220438696](https://blog-1301127393.file.myqcloud.com/BlogImgs/202205082207637.png)





## FAQ

* **为什么启动 docker 容器后访问 Huginn 显示网络错误？**

  Huignn 冷启动较慢，需要等待三五分钟。如果还是不行，检查端口映射和防火墙设置

* **为什么抓取到的包含相对路径的结果，网页上可以点击访问，但是生成的 RSS 不能正常使用？**

  检查 link 的设置，有些网站只是域名，有些网站有子目录，具体查看该网页源码中 head 节点里 base url 的设置

* **如何对爬取到的某一项的字符串做更高级更复杂的处理？**

  可以参考[Hugnn 官方对 Liquid 语法的文档](https://github.com/huginn/huginn/wiki/Formatting-Events-using-Liquid)以及[Shopify 官方关于 Liquid 模板的语法文档](https://shopify.dev/api/liquid)

* **为什么部署了 Huginn，等了很长时间都无法访问后台页面？**

  若部署后某个应用一直无法通过浏览器访问，请检查是否绑定到了 6000/6666 等特殊端口，浏览器会拦截对这些端口的访问参见[这里](https://blog.colinx.one/posts/docker-compose%E7%9A%84%E9%94%99%E8%AF%AF%E4%BD%BF%E7%94%A8%E5%A7%BF%E5%8A%BF/)

* **Huginn 爬虫访问目标网站被拦截了怎么解决**  
  介绍几个基本的反反爬虫策略：
  1. 带上 User Agent，要求是真实的浏览器 UA。
  2. Browserless 去掉默认会和请求一起发送的可以被识别为爬虫的特征参数，设置 browserless 的环境变量`DEFAULT_HEADLESS=false`
  3. 使用随机代理 IP，适用于因爬取频率达到爬虫阈值


## 扩展阅读

* [Huginn 官方文档](https://github.com/huginn/huginn/wiki/)
* [Liquid 模板引擎语法官方文档](https://shopify.dev/api/liquid/filters/)
* [RSS Man X 部署指南](https://blog.colinx.one/posts/rssmanx%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97/)
* [docker compose 的错误使用姿势](https://blog.colinx.one/posts/docker-compose%E7%9A%84%E9%94%99%E8%AF%AF%E4%BD%BF%E7%94%A8%E5%A7%BF%E5%8A%BF/)

