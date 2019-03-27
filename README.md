# web安全问题
参考网站：http://web.jobbole.com/92875/
-------------------------
## 1、XSS攻击
>XSS是跨站脚本攻击（Cross-Site Scripting）的简称，其原因是浏览器错误的将攻击者提供的用户输入数据当做JavaScript脚本给执行了。
>>XSS有几种不同的分类办法，例如:按照恶意输入的脚本是否在应用中存储，XSS被划分为：
>>>* 存储型XSS
>>>* 反射型XSS

>>按照是否和服务器有交互，可以分为：
>>>* Server Side XSS
>>>* DOM based XSS

>攻击者可以利用XSS漏洞来窃取包括用户身份信息在内的各种敏感信息、修改Web页面以欺骗用户，甚至控制受害者浏览器，或者和其他漏洞结合起来形成蠕虫攻击，等等。

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1554275800&di=2dc473bee17dc4edfbde20d9450b131f&imgtype=jpg&er=1&src=http%3A%2F%2Fnew.51cto.com%2Ffiles%2Fuploadimg%2F20080415%2F1445300.gif) 

### 如何防御
>防御XSS最佳的做法就是对数据进行严格的输入编码，使得攻击者提供的数据不再被浏览器认为是脚本而被误执行。例如`<script>`在进行HTML编码后变成了`&lt;script&gt;`，而这段数据就会被浏览器认为只是一段普通的字符串，而不会被当做脚本执行了。

>编码也不是件容易的事情，需要根据输出数据所在的上下文来进行相应的编码。例如刚才的例子，由于数据将被放置于HTML元素中，因此进行的是HTML编码，而如果数据将被放置于URL中，则需要进行URL编码，将其变为%3Cscript%3E。此外，还有JavaScript编码、CSS编码、HTML属性编码、JSON编码等等。好在现如今的前端开发框架基本上都默认提供了前端输出编码，这大大减轻了前端开发小伙伴们的工作负担。

>其他的防御措施，例如设置CSP HTTP Header、输入验证、开启浏览器XSS防御等等都是可选项，原因在于这些措施都存在被绕过的可能，并不能完全保证能防御XSS攻击。不过它们和输出编码却可以共同协作实施纵深防御策略。

## 2、iframe攻击
>iframe中的内容是由第三方来提供的，默认情况下他们不受我们的控制，他们可以在iframe中运行JavaScirpt脚本、Flash插件、弹出对话框等等，这可能会破坏前端用户体验。

>如果说iframe只是有可能会给用户体验带来影响，看似问题不大，那么iframe中的域名过期后被恶意攻击者抢注，或者第三方被黑客攻破，iframe中的内容被替换掉了，从而利用用户浏览器中的安全漏洞下载安装木马、恶意勒索软件等等，这问题可就大了。

### 如何防御
>iframe有了一个叫做sandbox的安全属性，通过它可以对iframe的行为进行各种限制，充分实现“最小权限“原则。使用sandbox的最简单的方式就是只在iframe元素中添加上这个关键词就好，就像下面这样：
```javascript
<iframe sandbox src="..."> ... </iframe>
```

>sandbox还忠实的实现了“Secure By Default”原则，也就是说，如果你只是添加上这个属性而保持属性值为空，那么浏览器将会对iframe实施史上最严厉的调控限制，基本上来讲就是除了允许显示静态资源以外，其他什么都做不了。比如不准提交表单、不准弹窗、不准执行脚本等等，连Origin都会被强制重新分配一个唯一的值，换句话讲就是iframe中的页面访问它自己的服务器都会被算作跨域请求。

>另外，sandbox也提供了丰富的配置参数，我们可以进行较为细粒度的控制。一些典型的参数如下：
>> * allow-forms：允许iframe中提交form表单
>> * allow-popups：允许iframe中弹出新的窗口或者标签页（例如，window.open()，showModalDialog()，target=”_blank”等等）
>> * allow-scripts：允许iframe中执行JavaScript
>> * allow-same-origin：允许iframe中的网页开启同源策略

## 3、点击劫持
>我们自己做的页面可能被不法分子放到他们精心构造的iframe或者frame当中，进行点击劫持攻击。

>这是一种欺骗性比较强，同时也需要用户高度参与才能完成的一种攻击。通常的攻击步骤是这样的：
>> * 1、攻击者精心构造一个诱导用户点击的内容，比如Web页面小游戏
>> * 2、将我们的页面放入到iframe当中
>> * 3、利用z-index等CSS样式将这个iframe叠加到小游戏的垂直方向的正上方
>> * 4、把iframe设置为100%透明度
>> * 5、受害者访问到这个页面后，肉眼看到的是一个小游戏，如果受到诱导进行了点击的话，实际上点击到的却是iframe中的我们的页面

>点击劫持的危害在于，攻击利用了受害者的用户身份，在其不知情的情况下进行一些操作。如果只是迫使用户关注某个微博账号的话，看上去仿佛还可以承受，但是如果是删除某个重要文件记录，或者窃取敏感信息，那么造成的危害可就难以承受了。

### 如何防御
>有多种防御措施都可以防止页面遭到点击劫持攻击，例如Frame Breaking方案。一个推荐的防御方案是，使用X-Frame-Options：DENY这个HTTP Header来明确的告知浏览器，不要把当前HTTP响应中的内容在HTML Frame中显示出来。

## 4、浏览器错误推断内容
>想象这样一个攻击场景：某网站允许用户在评论里上传图片，攻击者在上传图片的时候，看似提交的是个图片文件，实则是个含有JavaScript的脚本文件。该文件逃过了文件类型校验（这涉及到了恶意文件上传这个常见安全问题，但是由于和前端相关度不高因此暂不详细介绍），在服务器里存储了下来。接下来，受害者在访问这段评论的时候，浏览器会去请求这个伪装成图片的JavaScript脚本，而此时如果浏览器错误的推断了这个响应的内容类型（MIME types），那么就会把这个图片文件当做JavaScript脚本执行，于是攻击也就成功了。

### 如何防御
>浏览器根据响应内容来推断其类型，本来这是个很“智能”的功能，是浏览器强大的容错能力的体现，但是却会带来安全风险。要避免出现这样的安全问题，办法就是通过设置X-Content-Type-Options这个HTTP Header明确禁止浏览器去推断响应类型。

>同样是上面的攻击场景，后端服务器返回的Content-Type建议浏览器按照图片进行内容渲染，浏览器发现有X-Content-Type-OptionsHTTP Header的存在，并且其参数值是nosniff，因此不会再去推断内容类型，而是强制按照图片进行渲染，那么因为实际上这是一段JS脚本而非真实的图片，因此这段脚本就会被浏览器当作是一个已经损坏或者格式不正确的图片来处理，而不是当作JS脚本来处理，从而最终防止了安全问题的发生。

## 5、使用了不安全的框架、依赖包等
>应用使用了第三方代码，不论应用自己的代码的安全性有多高，一旦这些来自第三方的代码有安全漏洞，那么对应用整体的安全性依然会造成严峻的挑战。（想起了2018年貌似圣诞节还是哪天的一个bug，多个网站用了同一个框架，而该框架一声不吭内置了个功能，在圣诞节当天按钮上方自动出现雪堆图案，简直狗血彩蛋，各公司纷纷半夜叫起各种程序员各种调各种八阿哥，奋战好几个小时，在第二天天亮之后，框架才发文承认对此次“恐怖”事件负责。。手动滑稽）

![](http://dingyue.nosdn.127.net/ATRhfaXjirBjoVORe6TnQTlxoBWRdOrf9On3G07ccMgq01545803391120.jpeg) 
![](http://dingyue.nosdn.127.net/cUQYpeNqjqqjul9f1k38zvteHro55ylv8D5VfOTxEjWwa1545805415819.jpeg) 
![](http://dingyue.nosdn.127.net/faAVu73xi73K2BHbrQjsOwFVMV6XzeUiJFjMPsfCII5P91545803632813.jpeg) 
![](http://dingyue.nosdn.127.net/03X4MLAdqtOZYBdI6CjMGctGWIAjVn8gcpMhf1BQi67q51545803640129.jpeg) 

### 如何防御
>手动检查这些第三方代码有没有安全问题是个苦差事，主要是因为应用依赖的这些组件数量众多，手工检查太耗时，好在有自动化的工具可以使用，比如NSP(Node Security Platform)，Snyk等等。

## 6、操作https
>为了保护信息在传输过程中不被泄露，保证传输安全，使用TLS或者通俗的讲，使用HTTPS已经是当今的标准配置了。然而事情并没有这么简单，即使是服务器端开启了HTTPS，也还是存在安全隐患，黑客可以利用SSL Stripping这种攻击手段，强制让HTTPS降级回HTTP，从而继续进行中间人攻击。

>问题的本质在于浏览器发出去第一次请求就被攻击者拦截了下来并做了修改，根本不给浏览器和服务器进行HTTPS通信的机会。大致过程如下，用户在浏览器里输入URL的时候往往不是从`https://`开始的，而是直接从域名开始输入，随后浏览器向服务器发起HTTP通信，然而由于攻击者的存在，它把服务器端返回的跳转到HTTPS页面的响应拦截了，并且代替客户端和服务器端进行后续的通信。由于这一切都是暗中进行的，所以使用前端应用的用户对此毫无察觉。

### 如何防御
>解决这个安全问题的办法是使用HSTS（HTTP Strict Transport Security），它通过下面这个HTTP Header以及一个预加载的清单，来告知浏览器在和网站进行通信的时候强制性的使用HTTPS，而不是通过明文的HTTP进行通信：

```javasscript
Strict-Transport-Security: max-age=; includeSubDomains; preload
```
>这里的“强制性”表现为浏览器无论在何种情况下都直接向服务器端发起HTTPS请求，而不再像以往那样从HTTP跳转到HTTPS。另外，当遇到证书或者链接不安全的时候，则首先警告用户，并且不再让用户选择是否继续进行不安全的通信。

## 7、本地存储数据泄密
>以前，对于一个Web应用而言，在前端通过Cookie存储少量用户信息就足够支撑应用的正常运行了。然而随着前后端分离，尤其是后端服务无状态化架构风格的兴起，伴随着SPA应用的大量出现，存储在前端也就是用户浏览器中的数据量也在逐渐增多。

>前端应用是完全暴露在用户以及攻击者面前的，在前端存储任何敏感、机密的数据，都会面临泄露的风险，就算是在前端通过JS脚本对数据进行加密基本也无济于事。

>举个例子来说明，假设你的前端应用想要支持离线模式，使得用户在离线情况下依然可以使用你的应用，这就意味着你需要在本地存储用户相关的一些数据，比如说电子邮箱地址、手机号、家庭住址等PII（Personal Identifiable Information）信息，或许还有历史账单、消费记录等数据。

### 如何防御
>尽管有浏览器的同源策略限制，但是如果前端应用有XSS漏洞，那么本地存储的所有数据就都可能被攻击者的JS脚本读取到。如果用户在公用电脑上使用了这个前端应用，那么当用户离开后，这些数据是否也被彻底清除了呢？前端对数据加密后再存储看上去是个防御办法，但其实仅仅提高了一点攻击门槛而已，因为加密所用到的密钥同样存储在前端，有耐心的攻击者依然可以攻破加密这道关卡。

>所以，在前端存储敏感、机密信息始终都是一件危险的事情，推荐的做法是尽可能不在前端存这些数据。

## 8、缺乏静态资源完整性校验
>出于性能考虑，前端应用通常会把一些静态资源存放到CDN（Content Delivery Networks）上面，例如Javascript脚本和Stylesheet文件。这么做可以显著提高前端应用的访问速度，但与此同时却也隐含了一个新的安全风险。

>如果攻击者劫持了CDN，或者对CDN中的资源进行了污染，那么我们的前端应用拿到的就是有问题的JS脚本或者Stylesheet文件，使得攻击者可以肆意篡改我们的前端页面，对用户实施攻击。这种攻击方式造成的效果和XSS跨站脚本攻击有些相似，不过不同点在于攻击者是从CDN开始实施的攻击，而传统的XSS攻击则是从有用户输入的地方开始下手的。

### 如何防御
>防御这种攻击的办法是使用浏览器提供的SRI（Subresource Integrity）功能。顾名思义，这里的Subresource指的就是HTML页面中通过和元素所指定的资源文件。

>每个资源文件都可以有一个SRI值，就像下面这样。它由两部分组成，减号（-）左侧是生成SRI值用到的哈希算法名，右侧是经过Base64编码后的该资源文件的Hash值。

` <script src="“https://example.js”" integrity="“sha384-eivAQsRgJIi2KsTdSnfoEGIRTo25NCAqjNJNZalV63WKX3Y51adIzLT4So1pk5tX”"/>`

>浏览器在处理这个script元素的时候，就会检查对应的JS脚本文件的完整性，看其是否和script元素中integrity属性指定的SRI值一致，如果不匹配，浏览器则会中止对这个JS脚本的处理。

#### 第一次写自述不知道怎么写结语，但不写结语又太丑，，，so，，，，
