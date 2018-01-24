## 第二章 HTTP与Web网络
超文本传输协议可能是使用最广泛的应用层协议。起初它是帮助学者
共享HTML文档开发的。如今它是数不胜数的互联网应用的核心协议，
且是广域网的根本协议。

本章包含如下主题：

- HTTP协议结构
- 通过HTTP协议使用python与服务交互
- 下载文件
- HTTP功能，如压缩与cookie
- 错误处理
- 链接
- Python标准库 **urllib**
- Kenneth Retiz的第三方库 **Requests** 
**urllib库** 是处理HTTP任务的首选标准库。标准库中还包一个底层模块
叫**http** 。尽管它可以处理协议的各个方面，但它不适合日常使用。**urllib** 有更简洁的接口，且可处理接下来这章的所有内容。
第三方库**Requests** 是除**urllib** 非常流行的第二选择。它有着典雅
的接口和强大的功能集，是简化HTTP工作流程的极好工具。在本章最后我们会讨论它如何来代替**urllib**。

### 请求与响应
HTTP是应用层协议，绝大多数运行在TCP协议之上。HTTP协议有意设计成可
读的信息格式，但仍可用来传送符意的字节串数据。
一个HTTP数据交流包包含两个要素。由客户端发起的通过指定的URL链接请求服务器特定资源的**请求** ，和服务器提供客户端请求资源的**响应**  ，若报务器无法提供客户端请求的数据，响应会包含错误的相关信息。

这种事件顺序在HTTP中是固定的。所有交互都于客户端发起。除非客户端明确请求，服务端不会给客户端发送任何东西。

这一章会教你如何使用Python完成HTTP客户端。我们会学习如何发起请求，如何解释服务器的响应信息。在***第九章 Web应用*** 我们会学习编写服务器端程序。

到现在为止，应用最广的HTTP版本由 `RFC 7230` 到 `RFC 7235` 定义的是1.1版。HTTP2是最新的版本，在本书付印前正式批准的。绝大多数语义语法与1.1版相同，主要变化是TCP连接的实现。目前为止HTTP 2尚未广泛支持，所以本书以1.1版讲解。若你想了解更多HTTP 2相关信息，可参阅方档`RFC 7540`和 `RFC 7541`。

HTTP 1.0版，由`RFC 1945`定义，仍被一些较老的软件使用。1.1版向扣兼容1.0版，**`urllib`**和**`Requests`** 库都支持HTTP 1.1，我们不用担心会连接HTTP1.0的报务器。只是有些高级特性不可用。几乎现今所有服务都使用1.1版，这里不在介绍与1.0的区别。若你要了解更多信息（这真是一个好的开始），*stack overflow* 关于这一[问题](http://stackoverflow.com/questions/246859/http-1-0-vs-1-1 "http1.0 vs 1.1")给出了答案。

### 使用urllib发送请求
在 ***第一章 网络编程与Python*** 中讨论`RFC`文档下载时，我们看到了HTTP数据交换的一些例子。**`urllib`** 包根据使用HTTP时不同的任务拆分成多个子模块。当发送请求与接受响应时，我们使用**`urllib.request`** 模块。

使用**`urllib`** 获取URL内容时流程很简单。打开Pyton解释器，如下操作：

	>>> from urllib.request import urlopen
	>>> response = urlopen('http://www.debian.org')
	>>> response
	<http.client.HTTPResponse object at 0x000002D3EEABBB00>
	>>> response.readline()
	b'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">\n'

我们使用`urllib.request.urlopen()`函数发送请求和接收请求`http://www.debian.org`资源的响应，这个例子中为一个HTML页面，接下来打印出我们接收的HTML第一行内容。
 
### 响应

我们在仔细研究一下响应对象。前面的例子我们看到`urlopen()`返回一个`http.client.HTTPResponse`实例。响应象使我们能获取请求的资源，和响应的属性与元数据。如下可查看前一节响应的网址：

	>>> response.url
	'https://www.debian.org/'

通过`readline()`和`read()`方法，以类似文件接口的方式获取请求资源的数据。前面一节中我们看到了`readline()`方法。下面是使用`read()`方法：
	
	>>> response = urlopen('http://www.debian.org')
	>>> response.read(50)
	b'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" '

`read()`方法返回指定数量的字节数据。这个例子中是50个字节。通过无参数调用`read()`方法会一口气返回所有数据。

类文件接口方法是有限制的。一旦数据被读取后，前面提到的函数将不能再资读取数据。为了证明这一点，试如下操作：

	>>> response = urlopen('http://www.debian.org')
	>>> response.read()
	b'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">\n'
	...
	>>> response.read()
	b''

我们可以看到在第二次调用`read()`函数时，返回一个空字符串。这里没有`seek()`和`rewind()`方法来重置位置。所以最好保存`read()`的输出到变量中。

`readline()`和`read()`函数都会返回字节串对象，无论是`http`和`urllib`都不会将收到的数据解码成Unicode编码。本章后面，在`Requests`库帮助下我们会查找一种处理这种问题的办法。

### 状态代号

假如我们想要知道我们的请求是否发生了任何不可预测的问题？或者我们想知道在我们读取数据出来前响应中是否包含任何信息？或许我们在等待一个很长的响应，我们相要不用读取整个响应来快速判断请求是否成功。

HTTP响应通过**状态码** 提供这样一种方式。通过读取响应的状态属性可以获取它的状态码。

	>>> response.status
	200

状态码是一些整数，它们告诉了我们请求运行情况。代号`200`通知我们一切运行良好。

这里有一些代号，每一个表达不同的意义。根据第一个数字，状态码归为以下几组：

- 100：信息性质类
- 200：成功
- 300：重定向
- 400：客户端错误
- 500：报务器错误

一些常遇到的状态码和它们的信息如下：

- 200：OK 成功
- 404：Not Found 未找到
- 500：Interal Server Error 报务器内部错误

通过IANA官方维护的状态列表可从[这里](https://www.iana.org/assignments/http-status-codes "http status codes")找到。这章里我们会看到多种不同的状态码。

### 错误处理

状态码帮助我们判断响应是否成功。在`200`范围内的任一状态码表明成功，同理在`400`和`500`范围内的任一状态码表明失败。

状态码总应该被检查，以确保我们的程序在发生错误时能正确反应。**`urllib`** 通过在碰到问题时抛出一个异常来帮助我们检查状态码。

让我们尝试如何捕捉和有效处理这些异常。尝试如下的命令块：

	>>> import urllib.error
	>>> form urllib.request import urlopen
	>>> try:
	...   urlopen('http://www.ietf.org/rfc/rfc0.txt')
	... except urllib.error.HTTPError as e:
	... print('status', e.code)
	... print('reason', e.reason)
	... print('url', e.url)
	...
	status:404
	reason: Not Found
	url: http://www.ieft.org/rfc/rfc0.txt

在这里我们请求了RFC0，然而它并不存在，所以服务器返回`404`状态码，`urllib`检测到了这一消息并抛出了`HTTPError`异常。

可以看到`HTTPError`提供了很多关于请求的有用的属性。在前面的例子中，我们使用了`status`,`reason`和`utl`属性来获取请求的一些信息。

如果在网络协议的更底层发生了错误，相应的模块会抛出一个异常。`urllib`库会捕捉这些异常并把它们打包成`URLError`。比如，我们可能会指定一个不存在的IP地址或主机名，如下所示：

	>>> urlopen('http://192.0.2.1/index.html')
	...
	urllib.error.URLError: <urlopen error [Errno 110] Connection timed out>

在这个例子中我们从`192.0.2.1`主机上请求`index.html`页面。`192.0.2.0/24`范围内IP地址被保留为文档专用，所以你永远不会碰到前面用到的IP地址的主机。因此，TCP连接超时，`socket`抛出超时异常，`urllib`捕捉后重新包装再次抛出给我们。我们可以像之前例子那样来捕捉这些异常。

### HTTP头

请求和响应由两个主要部分组成，`headers头部`和`body主体`。在***第一章 网络编程和Python***中的`TCP RFC下载器`中我们简略的看到了HTTP头部。`头部headers`为通过TCP连接传送的原始消息开始几行的协议相关的信息。`主体body`为消息的剩余部分。两者通过一个空白行隔开。如下为一个HTTP请求例子：

	GET / HTTP/1.1
	Accept-Encoding: identity
	Host: www.debian.com
	Connection: close
	User-Agent: Python-urllib/3.4

第一行称为请求行。它由请求方法`GET`,资源路径`/`和HTTP版本`1.1`组成。剩下的几行为请求头部。每一行由一个头名字，一个冒号和一一个头名字的值组成。前面例子输出中的请求只包含了头部，没有包含主体。

头部有多种用途。在请求中，它们可用来传递额外的数据，比如`cookie`和认证证书及请求报务器回复首选格式的资源。

比如，一个重要的头部为`Host`头信息。很多web服务器提供在同一个服务器上使用同一IP地址，安装多个网站的功能。
DNS别名为多个网站域名设置，所以它们全部指向同一IP地址。实际上，web服务器提供多个域名，每一个对应一个它运行的网站。IP和TCP（HTTP运行在其上）不能用来告诉服务器客户端要连哪个域名，因为它们两个都单纯地运行于IP地址之上。HTTP协议允许客户端在HTTP请求中通过包含`Host`头信息提供要访问的域名。

接下来一节我们会看到更多的请求头部。

如下是一个响应的例子：

	HTTP/1.1 200 OK
	Date: Sun, 07 Sep 2014 19:58:48 GMT
	Content-Type: text/html
	Content-Length: 4729
	Server: Apache
	Content-Language: en

	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  
  	"http://www.w3.org/TR/html4/strict.dtd">\n...

第一行包含了协议的版本，状态码和状态消息。下面的行包含了头部信息和一个空行，然后是主体。在响应中，服务器可用头部信息来告知客户端一些信息，比如主体的长度，响应主体包含内容的格式，客户端要保存的cookie数据。

按如下命令来查看一个响应对象的头部：

	>>> response = urlopen('http://www.debian.org')
	>>> response.getheaders()
	[('Date', 'Mon, 01 Jan 2018 08:53:00 GMT'), ('Server', 'Apache'), ('Content-Location', 'index.en.html'), ('Vary', 'negotiate,accept-language,Accept-Encoding'), ('TCN', 'choice')
	...

`getheaders()`方法以元组列表`(header name,header value)`的格式返回头部信息.`RFC 7231`有`HTTP1.1`头部信息和它们含义的完整列表.让我们看一下在请求和响应中如何使用一些头部信息。

### 定制请求

若要使用头部提供的功能，在发送请求前我们要把头部信息加入到请求中。要这样做，我们不能只使用 `urlopen()`。我们要按如下步骤：

- 创建一个`Request`对象
- 把头加入`Request`对象中
- 使用`urlopen()`发送请求对象

我们接下来学习如何定制一个请求来获取`Debian`瑞典语版本的主页。我们会用`Accept-Language`头部，它告诉服务器我们想要的返回资源的首选语言。注意的是，不是所有的服务器都有多国语言版本的资源，所以不是所有服务器都会响应`Accept-Language`头部。

首先，我们创建一个`Request`对象：

	>>> from urllib.request import Request
	>>> req = Request('http://www.debian.org')

接下来添加头信息：

	>>> req.add_header('Accept-Language','sv')

`add_header()`以头信息的名字和内容为参数。 `Accept-Language` 头信息使用两个 `ISO 639-1`语言代号字母。瑞典语的代号为`sv`。
最后我们通过`urlopen()`提交定制后的请求:

	>>> response = urlopen(req)

我们可以通过打印响应消息开始的几行来查看响应是否为瑞典语。

	>>> response.readlines()[:5]
	>>> [b'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">\n', 
	b'<html lang="sv">\n', 
	b'<head>\n', 
	b'  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">\n', 
	b'  <title>Debian -- Det universella operativsystemet </title>\n']

`Accept-Language`头已经通知了服务器我们首选的响应内容的语言。

要查看请求的头部信息，可如下操作：

	>>> req = Request('http://www.debian.org')
	>>> req.add_header('Accept-Language', 'sv')
	>>> req.header_items()
	[('Accept-language', 'sv')]

`urlopen()`方法在我们运行它时会添加一些自己的头部信息：

	>>> response = urlopen(req)
	>>> req.header_items()
	[('Host', 'www.debian.org'), ('User-agent', 'Python-urllib/3.6'), ('Accept-language', 'sv')]

一个添加头部的快捷方式为在创建请求对象的同时添加它们，如下所示：

	>>> headers = {'Accept-Language': 'sv'}
	>>> req = Request('http://www.debian.org', headers=headers)
	>>> req.header_items()
	[('Accept-language', 'sv')]

我们把头信息以字典的格式传递给`Request`对象构造器关键词`headers`参数。通过这种方式，我们可以通过添另更多条目到字典中一次性添加多条头信息。

让我们看一下关于头部我们还有哪些更多操作。

#### 内容压缩

请求头字段`Accept-Encoding`和响应头字段`Content-Encoding`可一起工作，以暂时允许网络传输时响应的主体内容进行编码，这样可减小传输的数据量。

这一流程如下几步：

- 客户端通过在`Accept-Encoding`头字段添加可接受的编码表后发送请求
- 服务器选择一种它支持的编码方法
- 服务器使用这一编码方式编码主体数据
- 报务器发送响应，并通过`Content-Encoding`头字段指定它使用的编码方式
- 客户通过指定的编码方式解码响应主体数据

让我们讨论如何请求一个文档，使服务器以`gzip`格式压缩响应主体。首先来构建请求：

	>>> req = Request('http://www.debian.org')

接下来，加下`Accept-Encoding`头字段：
	
	>>> req.add_header('Accept-Encoding', 'gzip')

然后，通过`urlopen()`提交请求：

	>>> response = urlopen(req)

通过查看响应的`Content-Encoding`头字段来检查服务器是否使用了`gzip`压缩：

	>>> response.getheader('Content-Encoding')
	'gzip'

利用`gzip`模块可解压缩主体数据：

	>>> import gzip
	>>> content = gzip.decompress(response.read())
	>>> content.splitlines()[:5]
	[b'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">', 
	b'<html lang="en">', 
	b'<head>', 
	b'  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">', 
	b'  <title>Debian -- The Universal Operating System </title>']

编码通过`IANA`注册。现在的编码方式有：`gzip`,`compress`,`deflate`，`identity`。前三个指向特定的压缩方法。最后一个允许客户端表时它不想要任何编码作用于内容上。

让我们看一下通过指定`identity`编码请求不要压缩会发生什么：

	>>> req = Request('http://www.debian.org')
	>>> req.add_header('Accept-Encoding','identity')
	>>> response = urlopen(req)
	>>> print(response.getheader('Content-Encoding'))
	None

当服务器使用`identity`编码方式时，响应中不包含`Content-Encoding`头字段。

#### 多值

在头字段`Accept-Encoding`中加入更多的值，通过逗号分隔，来告诉服务器我们可接受不只一种编码方式。让我们试一下。创建我们的`Request`对象：

	>>> req = Request('http://www.debian.org')

然后，添加我们的头字段，这次添加更多的编码方式：

	>>> encodings = 'deflate, gzip, identity'
	>>> req.add_header('Accept-Encoding',encodings)

现在提交请求并查看响应的编码：

	>>> response = urlopen(req)
	>>> response.getheader('Content-Encoding')
	'gzip'

若有需要，通过添加一个`q`值可以给指定的编码赋予相对的权重：

	>>> encodings = 'gzip, deflate;q=0.8, identity;q=0.0'

`q`值放在编码名称的后面，并通过分号分隔。`q`的最大值是1.0，也是`q`的值未指定时的默认值。所以前面的语句可以理解为我的首选编码方式为`gzip`,第二选择是`deflate`,若没有其它可用编码了，第三选择为`identity`。

### 内容协商

通过头字段`Accept-Encoding`指定内容压缩方式，`Accept-Language`选择语言都是内容协商的例子，客户端考虑它请求的资源格式和内容指明它的首选项。如下的头字段也可作为这种用途：

- `Accept`:请求首选的文件格式
- `Accept-Charset`:通过首选的字符集来请求资源

有更多的其它方面的内容协商技巧，但是由于它们受支持的不一致性会变的很复杂，这章就不全部讲解这些内容了。`RFC 7231`包含了你想要的所有细节。查看`3.4`,`5.3`,`6.4.1`,`6.5.6`等小节，你会发现的的程序会用到它。

#### 内容类型

`HTTP`可用来传送任何类型的文件或数据。服务器可以通过响应的头字段`Content-Type`内容来通知客户端它发送的主体容的数据类型。这是`HTTP`客户端决定如何处理服务器返回主体数据的首要方法。

通过检查响应的头字段来查看内容类型，如下所示：

	>>> response = urlopen('http://www.debian.org')
	>>> response.getheader('Content-Type')
	'text/html'

这一字段中的值取自由`IANA`维护的一份列表。这些值有各种名字：`内容类型`，`互联网媒体类型`，`MIME类型`（*MIME代表多用途互联网邮件扩展，这一规格说明书首次确定了这一约定）*。这一完整列表可在[这里](http://www.iana.org/assignments/media-types "Content Types")找到。

有很多注册了媒体类型用来在网络中传输的各种数据类型，如下是一些常用类型：

| 媒体类型  | 描述 |
| :------: | :---: |
|text/html | HTML文档|
| text/plain| 无格式文本文档 |
|image/jpeg|JPG图像|
|application/pdf|PDF文档|
|application/json|JSON数据|
|application/xhtml+xml|XHTML文档|

还有一种有趣的媒体类型是`application/octet-stream`，在实际使用中，它用来表示没有一种可适用的媒体类型。这种情况的一个例子如腌制后(`pickled`)的Python对象 。同样可用来表示服务器未知的文件类型。若要正确处理这一类型的媒体文件，我们要通过其它方式来发现它的格式。可能的途径如下：

- 检查下载资源（若存在）的后缀名。`mimetypes` 模块可以用来确定它的媒体类型 *（参见 **第三章 功能API**）* 。
- 下载数据后使用文件类型分析工具。使用Python标准库`imghdr`模块可分析图片，第三方库`python-magic`包，或`GUN  `文件命令可用在其它类型文件上。

内容类型的值可包含附加的参数来提供更多的关于类型的信息。这通常用来提供数据使用的字符集。比如：

    Content-Type: text/html; charset=UTF-8.

这个例子中，它告诉了我们文档的字符集为`UTF-8`。这个参数添加到一个分号后面，并总是键-值对的方式。

让我们讨论一个例子，下载`Python`的主页，并使用它返回的`Content-Type`值。首先提交请求：

    >>> response = urlopen('http://www.python.org')

然后检查响应的`Content-Type`值，提取字符集信息：

    >>> format, params = response.getheader('Content-Type').split(';')
    >>> params
    ' charset=utf-8'
    >>> charset = params.split('=')[1]
    >>> charset
    'utf-8'

最后我们使用提供的字符集解码响应内容：

    >>> content = response.read().decode(charset)

要经常注意的是，服务器或不提供`Content-Type`头字段的`charset`，或提供一个错误的值。因此这一值只能做为一个建议。这也是本章后面关注`Requests`库的原因之一。它会自动收集这些提示并找到解码响应主体的字符集，并为我们做出最优的预测。


### 客户代理

另一个值得了解的头字段是`User-Agent`。任何使用`HTTP`沟通的客户端都可称之为**客户代理**。`RFC 7231`建议用户端应当使用`User-Agent`头字段在每次请求时表明自己的身份。它包含的内容一般由发送请求的软件飞决定，它一般由一个包含表明程序和版本号的字串构成，可能还会包含操作系统和运行它的硬件信息。比如，我现在使用的Firefox客户端版本如下：
    Mozilla/5.0 (X11; Linux x86_64; rv:24.0) Gecko/20140722  
    Firefox/24.0 Iceweasel/24.7.0

这是一行较长的字符串这里分成了两行。你可能会猜到，我在64位Liunx系统上运行`Iceweasel`*(Firefox的Debian版本)* 版本号24。用户代理字段不是用来表明特定的用户的。它们只是用来表明发起请求的产品。

我们可以查看`urllib`使用的用户代理。执行如下几步：

    >>> req = Request('http://www.python.org')
    >>> urlopen(req)
    >>> req.get_header('User-agent')
    'Python-urllib/3.6'

这里我们建立了一个请求并用`urlopen`提交，`urlopen`把客户代理头字段添加到了请求中。我们可通过使用`get_header()`方法还检查这一头字段。这一头字段和它的值添加到了通过`urllib`发送的的每一次请求中，这样我们发起请求的每台服务器都会知道我们在使用Python 3.6和`urllib`库。

网站管理中央党校可以检查请求的客户端，使用这一信息可应用到各种事情上，包括如下：

- 为他们的网站统计访问分类
- 阻止包含特定客户代理字串的访问
- 为已知客户端的问量发送资源的可选版本，比如解释特定语言如CSS时的错误，或不支持一些语言，如JavaScript

最后两条会引发问题，因为它们会停止或阻碍我们获取想要的内容。要避开这一点，我们可以通过尝试设置我们的客户代理让它来假扮一个有名的浏览器。称之为**欺骗**，如下所下：

    >>> req = Request('http://www.debian.org')
    >>> req.add_header('User-Agent', 'Mozilla/5.0 (X11; Linux x86_64;  
    rv:24.0) Gecko/20140722 Firefox/24.0 Iceweasel/24.7.0')
    >>> response = urlopen(req)

服务器会把我们的程序当作普通的FireFox客户端来响应。不同浏览器的客户端代理字串可在网上找到。我尚未遇到关于此的全面的源资，但是Google一下浏览器和它的版本号通常会发现一些东西。或者你可以使用`Wireshark`来捕捉一个由你想模仿的浏览器发起的`HTTP`请求，再查看它的请求包含的客户代理头字段。

### Cookie文件

`Cookie`是服务器发送的响应头字段`Set-Cookie`中的一小段数据。客户端保存到本地后，未来发送给服务器的请求都会添加这一字段。

服务器在很多地方用到`Cookie`。它们可以给`Cookie`添加一个独特的ID，以便记录客户端到达了网站的不同区域。它们可以存储登陆标记，这允许服务器自动使客户端登入，即使客户离开之后又再次登入。也可用来存储用户的参数数据和用户个人信息片断等等。

`Cookie`是很有必要的，因为在两次请求中间服务器没有办法追踪客户端。`HTTP`被称为无状态协议。它没有明确定的机制来确保两个请求来自于同一个客户端。若不使用`Cookie`来允许服务器添加一个独特的标识给请求的话，像购物车（这就是开发`Cookie`时想要解决的）将不可能实现，因为服务器不知道哪个购物车属于哪个请求。

在`Python`中我们也要处理`Cookie`,没有它，一些网站可能不会以想要的方式工作。在使用`Python`时我们可能要进入需要登陆的页面，登陆会话通常也由`Cookie`完成。

#### Cookie处理

我们将如何通过`urllib`库来处理`Cookie`.首先我们创建一个变量用来存储服务器发送的`Cookie`：

    >>> from http.cookiejar import CookieJar
    >>> cookie_jar = CookieJar()
    
接下来创键一个称为`urllib opener`的对象。它会自己解析服务器发送回来的`Cookie`，并存储到我们的`Cookie`容器中：

    >>> from urllib.request import build_opener, HTTPCookieProcessor
    >>> opener = build_opener(HTTPCookieProcessor(cookie_jar))
    
然后，我们便可使用这个打开器来执行`HTTP`请求:

    >>> opener.open('http://www.github.com')

最后，我们检查一下服务器发送给我们的`Cookie`:
    
    >>> len(cookie_jar)
    2

任何时侯我们要用`opener`执行更多请求时，`HTTPCookieProcessor`函数都会检查`cookie_jar`是否包含这个网站的`Cookie`,若存在它会自动包含到我们的请求中。同时，若有其它的`Cookie`它也会再加入到`cookie_jar`中。

`http.cookiejar`模块也包含一个`FileCookieJar`类，它的工作方式和`CookieJar`相似，但它提供了一个可以把`Cookie`存储到文件中的附加功能。这允许跨多个`Python`会话的`Cookie`保存。

#### 了解你的Cookie文件

详细了解`Cookie`的属性是有意义的。让我们检查一下前面一节例子中`GitHub`发送给我们的`Cookie`。

要了解它，我们要从`Cookie`的容器中取出它。`CookieJar`模块不允许我们直接获取它们，但它支持了迭代协议。所以可以用它创建一个列表来快速的取出它们：

    >>> cookies = list(cookie_jar)
    >>> cookies
    [Cookie(version=0, name='logged_in', value='no', ...),
    Cookie(version=0, name='_gh_sess', value='eyJzZxNzaW9uX...', ...)
    ]

可以看到我们有两个`Cookie`对象。现在让我们从第一个中获取一些信息：

    >>> cookies[0].name
    'logged_in'
    >>> cookies[0].value
    'no'
    
`Cookie`的名字使服务器可以快速的参考它。这个`Cookie`明显是`GitHub`用来判断我们是否登陆的机制中的一部分。接下来，我们这样做：

    >>> cookies[0].domain
    'github.com'
    >>> cookies[0].path
    '/'

域名和路径表明了`Cookie`的作用范围，我们的`urllib`打开器在发送给`www.github.com`和它的子域名的请求中都会加入这些`Cookie`，因为它的子域名都在根目录之下。

接下来让我们看一下这一`Cookie`的生存期：

    >>> cookies[0].expires
    2060882017

这是一个`Unix`时间戳，我们可把它转换成日期和时间：

    >>> import datetime
    >>> datetime.datetime.fromtimestamp(cookies[0].expires)
    datetime.datetime(2035, 4, 22, 20, 13, 37)
    
我们的`Cookie`会在2035年4月22日过期。过期日期是服务器想要客户端保存`Cookie`的时间。一旦超过过期日期，客户端会扔掉它并在下一请求时，服务器发送新的`Cookie`给它。当然，客户端可以立即把它扔掉，虽然这会给依靠`Cookie`工作的功能带来响影响。

我们看一下常用的两个`Cookie`标志：

    >>> print(cookies[0].get_nonstandard_attr('HttpOnly'))
    None
    
有多种方式来获取保存在客户端的`Cookie`：

- 通过客户端`HTTP`请求和响应响应序列
- 通过运行在客户端的脚本，如：`JaveScript`
- 通过客户端其它进程，如：`Flash`

`HttpOnly`标志表示客户端只允许获取`Cookie`做为`HTTP`请求和响应的一分部分时才能使用。其它方式会被拒绝。这会保护客户端免遭跨站点脚本的攻击（*详见 第九章 Web编程*）。这是一个重要的安全特性，当服务器这样设置时，我们的客户端也要相应的按此工作。

这里也有一个安全标志：
    
    >>> cookies[0].secure
    Ture
    
若它的值为真，这个安全标志表示，`Cookie`数据只能通过安全的连接传送，比如：`HTTPS`。同样的，我们也要尊敬这一设置，若这一标志被设为真，我们的程序发送包含`Cookie`的请求时，只会发送给`HTTPS`加密的`URL`链接。

你可能会觉得这里有些矛盾。我们的`URL`链接通过`HTTP`发送请求，服务器给我们发送了`Cookie`，但这一`Cookie`中却要求只能通过安全的链接发送。网站设计人员真的会留下像这样一个安全漏洞吗？放心吧，它们不会的。响应其它是通过`HTTPS`发送的。这是怎么发生的呢？答案就是：**重定向** 。

### 重定向

有时服务器把它的内容挪来挪去。有时也会废弃一些内容并把新内容放到一个不同的新地方。有时服务器相要我们使用更安全的`HTTPS`协议来替换`HTTP`。所有这些情况下，它们会有访问旧链接的流量，这时服务器更想给客户自动发送最新的内容。

在`300`范围内的状态码就是设计用来满足这一需求的。这些状态码表示需要更多的操作来完成请求。最常见的动作就是重试另一个`URL`链接。这称为**重定向** 。

我们将会看到当使用`urllib`这是如何工作的。发起一个请求：

    >>> req = Request('http://www.gmail.com')
    >>> response = urlopen(req)
    
很简单，现在我们看一下响应的`URL`地址：

    >>> response.url
    'https://accounts.google.com/ServiceLogin?service=mail&passive=true&r  
    m=false...'
这并不是我们请的的地址!若我们在浏览器中打开这个`URL`地址会发现，它其它是`Google`的登陆页面*(若你已经缓存了`Google`的登陆会话，则要清除缓存再试)* 。`Google`把我们从`http://www.google.com`重定向到它的登陆页面，`urllib`自动跟踪了重定向。有时我们可能会被重定向不只一次。看我们请求对象的`redirect_dict`属性：

    >>> req.redirect_dict
    {'https://accounts.google.com/ServiceLogin?service=...': 1,  
    'https://mail.google.com/mail/': 1}
    
`urllib`库把我们重定向的每个`URL`链接添加到了这个字典中。可以看到我们被重定向了两次，第一次到`https://mail.google.com`,第二个到登陆页面。

当我们发送第一个请求，服务器返回包含重定向的响应状态码，状态码为`301`,`302`,`303`,`307`之一。它们都表示一个重定向。这个响应包含一个位置`Location`头字段，它包含了要重定向的新地址。`urllib`会提交一个新请求到这一新地址，如前所述，它会接收另一个新地址使它指向`Google`的登陆面面。

`urllib`为我们跟踪了重定向，它们通常不会影响我们，但知道一个 `urllib`返回响应的`URL`不同于我们请求的地址是有意义的。同时，在一个请求中，若遇到了太多重定向*（对`urllib`来说多余10次）* ，`urllib`会放弃并抛出`urllib.error.HTTPError`异常。

### URL链接

统一资源定位符，或`URL`是网络运行的基础，它们的正式定义在`RFC 3986`中。`URL`代表了给定主机的资源。`URL`如何链接到远程主机的资源上完全由系统管理员决定。`URL`可以指向服务器上的文件，或当请求发生时动态生成的资源。`URL`链接到什么不重要，因为它只在我们请求它时才发生作用。

`URL`由多个部分组成。`Python`使用`urllib.parse`模块来处理`URL`链接。让我们用`Python`把一个`URL`拆分成它的各组成部分：

    >>> from urllib.parse import urlparse
    >>> result = urlparse('http://www.python.org/dev/peps')
    >>> result
    ParseResult(scheme='http', netloc='www.python.org', path='/dev/peps',  
    params='', query='', fragment='')
    
`urllib.parse.urlparse()`函数解释了我们的`URL`，并把`http`作为（`scheme`）**方式**，`www.python.org`作为**网络地址**，`/dev/peps'作为 **路径** 。我们可以解析结果的属性来获得这些组成部分：

    >>> result.netloc
    'www.python.org'
    >>> result.path
    '/dev/peps'

几乎网络上的所有资源，我们都会使用`http`和`https`方式。在这两个方式中，要定位特定的资源，我们需要知道它位于的主机和要连接的`TCP`端口*（两者合起来组成了网络地址）* ，且我们要知道资源在主机上的路径*（它就是路径部分）* 。

可以在主机名后面显式的添加指定的端品号。它和主机名以冒号分隔。让我们看一下使用`urlparse`处理后会发生什么：

    >>> urlparse('http://www.python.org:8080/')
    ParseResult(scheme='http', netloc='www.python.org:8080', path='/',  
    params='', query='', fragment='')

`urlparse`方法只是把它作为网络地址的一部分。这样正好，这正是调用者如`urllib.request.urlopen()`希望的格式。

若我们不提供一个端口号（通常都是这样），默认的端口号就会被使用，`80`端口用于`http`，`443`用于`https`。这正是我们通常希望的，这些正是`HTTP`和`HTTPS`协议的标准端口。

#### 路径和相对资源定位符

路径是跟在主机名和端口号后的任何东西。路径通常以斜线(**/**) 开头，当只跟着一个斜线时，称为根目录。运行如何命令可以看出：

    >>> urlopen('http://www.python.org/')
    ParseResult(scheme='http', netloc='www.python.org', path='/',  
    params='', query='', fragment='')

若请求中没有指定路径，则`urllib`默认请求根目录。

当`URL`中包含**方式**和**主机名** *（如前面的例子中）* ，这个`URL`称为**绝对资源定位符**。对应的，可能会存在**相对资源定位符**，它只包含路径部分，如下所示：
    
    >>> urlparse('../images/tux.png')
    ParseResult(scheme='', netloc='', path='../images/tux.png',  
    params='', query='', fragment='')

可以看出解析结果只包含了路径。若我们想要使用相对资源定位符来请求资源，需要提供丢失的**方式**，**主机名** 和 **基础路径** 。

通常我们遇到相对资源定位符的情况是，我们已经从一个资源定位符那里得到了资源。我们仅需要将源资的定位符来填充到丢失的部分。让我们看一个例子。

假如我们已经取得了一个`URL` `http:/www.debian.org`,且在网上源码中我们发相了**关于**页面的相对资源定位符。它的相对源定位符为`intro/about`。

我们可利用`urllib.parse.urljoin()`函数，通过之前原始页面的`URL`来创建绝对`URL`。让我们看是如何完成的：

    >>> form urllib.parse import urljoin
    >>> urljoin('http://www.debian.org', 'intro/about')
    'http://www.debian.org/intro/about'

当把基本`URL`和相对`URL`提供给`urljoin`后，我们创建了一个新的绝对`URL`.

这里要注意的是，`urljoin`如何在主机名和路径中添加了斜线。只有当基本`URL`不包含路径时，`urljoin`才会为我们添加斜线，如前面例子那样。让我们看看若基本`URL`包含路径时会发生什么。

    >>> urljoin('http://www.debian.org/intro/', 'about')
    'http://www.debian.org/intro/about'
    >>> urljoin('http://www.debian.org/intro', 'about')
    'http://www.debian.org/about'

这会给我们不同的结果。注意`urljoin`在基本`URL`以斜线结尾时直接附加到路径后，但若基本`URL`不以斜线结尾时，它直接把最后的路径部分替换掉。

我们可以通过在路径前添加一个斜线来强制替换基本`URL`所有的路径部分。如下：

    >>> urljoin('http://www.debian.org/intro/about', '/News')
    'http://www.debian.org/News'

如何导航到父目录呢？让我们试一下标准的点号语句，如下所示：

    >>> urljoin('http://www.debian.org/intro/about/', '../News')
    'http://www.debian.org/intro/News'
    >>> urljoin('http://www.debian.org/intro/about/', '../../News')
    'http://www.debian.org/News'
    >>> urljoin('http://www.debian.org/intro/about', '../News')
    'http://www.debian.org/News'

它正如我们期望的那样运行。要注意基本`URL`结尾以斜线结束和没有斜线的两者之间的区别。

最后，假如‘相对’`URL`正好是绝对`URL`呢：

    >>> urljoin('http://www.debian.org/about', 'http://www.python.org')
    'http://www.python.org'

相对`URL`完全替换了基本`URL`。这是很方便的，因为在用`urljoin`作于于`URL`前不用再检测它是否为相对的。

#### 查询字符串

`RFC 3986`定义了`URL`的另一属性。它们可以包含以键值对格式出现在路径后面的额外参数。以一个问号和路径分隔，如下所示：

    http://docs.python.org/3/search.html?q=urlparse&area=default

这个参数字符串称为查询字符串。多个参数通过与符号(&)分隔。让我们看一下`urlparse`如何处理它：

    >>> urlparse('http://docs.python.org/3/search.html?q=urlparse&area=default')
    ParseResult(scheme='http', netloc='docs.python.org',  
    path='/3/search.html', params='', query='q=urlparse&area=default',  
    fragment='')

所以，`urlparse`识别到了查询字符串做为查询部分。

查询字符串用来给我们想要获取的资源提供参数，通常这以某种方式来定制资源。在前面提到 的例子中，我们的查询字符串告诉`Python`文档搜索页面，我们想要搜索`urlparse`字段。

`urllib.parse`模块有一个功能帮助我们把`urlparse`返回的查询部分变成更有用的信息：

    >>> from urllib.parse import parse_qs
    >>> result = urlparse('http://docs.python.org/3/search.html?q=urlparse&area=default')
    >>> parse_qs(result.query)
    {'area': ['default'], 'q': ['urlparse']}

`parse_qs`方法读取查询字串并肥它转换成一个字典。为什么字典的值是以列表的行式存储的？这是因为一个查询字串中参数可能出现不只一次。试一下用重复的参数：

    >>> result = urlparse('http://docs.python.org/3/search.html?q=urlparse&q=urljoin')
    >>> parse_qs(result.query)
    {'q': ['urlparse', 'urljoin']}

为什么两个值都被添加到了列表中呢？它取决于服务器如何来解析它。若我们发送这个查询字符串，它可以只选值中的一个来使用，而忽略重复。你只能试一下，看会发生什么。

通常你可以通过在浏览器的网页界面中提交一个查询来找出我们要在查询字串中添加什么，通过查看结果页面的`URL`。你应该能认出你搜索的文字，从而推断出相应查询文字的键。通常 查询字符串里的很多其它参数对于获得基本的结果是不必要的。试试只能过搜索文字来请求页面会发生什么。若没有按预期的正常工作再添加其它参数。

如果以这种形式提交到了一个页面但结果页面的`URL`并不包含查询字符串，这是页面使用了其它一种方式来发送这种形式的数据。我们会在 *HTTP 方法* 一节中讨论`POST`方法时看到种方式。


#### URL编码

`URL`地址的表示被限制在`ASCII`字符集中，在这一字符集中，在不两只的`URL`组成部分中很多字符被保留使用从而要避免使用。我们通过称为`URL`编码的方式来避开它们。因它使用百分号标志作为转义字符，因此通常称为**百分号编码`percent encoding`** 。让我们使用`URL`编码一个字符串：

    >>> from urllib.parse import quote
    >>> quote('A duck?')
    'A%20duck%3F'

特殊字符`' '` 和 `'?'`被转义序列替换了。转义序列中的数字为字符的`ASCII`码的16进制表示。

需要转义的保留字符的完整规则定义在`RFC 3986`中，`urllib`提供给我们一对方法帮助构建`URL`地址。这意味着我们根本不用记住这些。

我们只需要完成：

- `URL`编码路径
- `URL`编码查询字符串
- 通过`urllib.parse.urlunparse()`函数来组合它们

让我们看一下如何用代码实现以上几步。首先，编码路径：

    >>> path = 'pypi'
    >>> path_enc = quote(path)

然后，编码查询字串：

    >>> from urllib.parse import urlencode
    >>> query_dict =  {':action': 'search', 'term': 'Are you quite sure  
    this is a cheese shop?'}
    >>> query_enc = urlencode(query_dict)
    >>> query_enc
    '%3Aaction=search&term=Are+you+quite+sure+this+is+a+cheese+shop%3F'

最后，我们把它们组合成一个`URL`:

    >>> from urllib.parse import urlunparse
    >>> netloc = 'pypi.python.org'
    >>> urlunparse(('http', netloc, path_enc, '', query_enc, ''))
    'http://pypi.python.org/pypi?%3Aaction=search&term=Are+you+quite+sure  
    +this+is+a+cheese+shop%3F'

`quote()`函数专门用设计来编码路径。默认情况下，它忽略斜线字符并不编码它们。在这前一例子中并不明显，让我们试试接下来的它是如何工作的：

    >>> from urllib.parse import quote
    >>> path = '/images/users/+Zoot+/'
    >>> quote(path)
    '/images/users/%2BZoot%2B/'

可以看到它跳过了斜线，但没有忽略加号`+`。这对路径来说是最合适的。

`urlencode()`函数同样专门用来从字典变量中编码查询字符串。注意它是如何正确的百分号编码我们的值然后把它们通过与号`&`连接起来，以此来构建查询字符串。

最后，`urlunparse()`方法需要一个和`urlparse()`输出结果的组成部分对应的6个元组，也包含两个空字符串。

对路径的编码有一个地方要注意。若路径自身包含斜线，可能会引发问题。下面的命令中可以看出：

    >>> username = '+Zoot/Dingo+'
    >>> path = 'images/users/{}'.format(username)
    >>> quote(path)
    'images/user/%2BZoot/Dingo%2B'

怎么用户名中的斜线没有跳过呢？这会被错误的解释成多余一层文件结构，这不是我们想要的。想要避开这一问题，首先我们要单独地避开包含斜线的符何路径元素，然后再把它们手动连接起来：

    >>> username = '+Zoot/Dingo+'
    >>> user_encoded = quote(username, safe='')
    >>> path = '/'.join(('','images', 'users', username))
    '/images/users/%2BZoot%2FDingo%2B'

现在看用户名的斜线是否百分号编码了呢？我们把用户名单独编码，通过`safe=''`参数告诉`quote`不要忽略斜线，它覆盖了默认忽略`/`的列表。然后我们能过简单的`join()`功能把路径元素结合起来。

这里，值得强调的是通过网线传送的主机名必须是`ASCII`格式，然而`socket`和`http`模块支持`Unicode`编码的主机名到`ASCII`兼容编码的透明转换，在现实使用中，我们不用担主机名的编码。在`codecs`模块文档`encodings.idna`一节中有更详细的说明。

#### URL小结

在前面章节中，我们使用了很多功能函数。让我们重新复习一下我们使用的每个模块是用来做什么的。所有这些功能都可在`urllib.parse`模块中找到。它们如下：

- 把`URL`拆分成它的各个组成部分：`urlparse`
- 把绝对`URL`和相对`URL`组合起来：`urljoin`
- 把查询字符串解析到一个字典中：`parse_qs`
- `URL`编码一个路径：`quote`
- 从字典构建一个`URL`编码的查询字符串：`urlencode`
- 从各个组成部分创建`URL`*（`urlparse逆过程）*：`urlunparse`

### HTTP方法

到目前为止我们已经使用请求服务器来给我们发送网站资源，但`HTTP`提供给了我们更多的活动空间。在请求行中的`GET`是一个`HTTP`方法，还有多个方法，如`HEAD`,`POST`,`OPTION`,`PUT`,`DELETE`,`TRACE`,`CONNECT`和`PATCH`。

在下一章中我们会详细了解它们的细节，但这里有两个方法，我们要快速的查看一下。
    

#### HEAD方法

`HEAD`方法和`GET`一样。惟一区别是服务器在响应中不会加入主体，即使请求的`URL`中包含可用的资源。`HEAD`方法用来检查资源是否存在或发生了变化。注意有些服务器没有实现这一个方法，若它们实现了，这将会节省大量的带宽。

在使用`urllib`时它的代替方法是在创建请求对象时提供方法名称：

    >>> req = Request('http://www.google.com', methon = 'HEAD')
    >>> response = rulopen(req)
    >>> response.status
    200
    >>> response.read()
    b''

这里服务器返回了成功`200`响应,然而如我们所料主体是空的。

#### POST方法

`POST`方法在一定意义上是`GET`方法的反向。我们用`POST`方法给服务器发送数据。然而服务器仍然可以给我们发送完整的响应。`POST`方法用来提交用户在`HTML`表单中的输入和给服务器上传文件。

使用`POST`方法时，我们想要发送的数据会包含到请求的主体里。我们可以发送任何字节数据并通过请求中添加头字段`Content-Type`的正确`MIME`类型来表明它的类型。

让我们来看个通过`POST`发送`HTML`表单数据给服务器的例子，正如我们在网站上提交一份表单时，浏览器那样做的。表单数据都由键值对构成;`urllib`允许我们平常使用字典那样来实现它*（在下面一节中我们会看到这一数据来自哪里）* ：

    >>> data_dict = {'P':'Python'}

在投递`HTML`表单数据时，表单数据要像`URL`中的查询字符串一样来格式化，且必须是`URL`编码的。`Content-Type`头字段也必须设置为特殊的`MIMI`类型: `application/x-www-form-urlencoded`。

由于这一格式不同于查询字符串，我们可以在我们的字典上使用`urlencode()`功能来准备这些数据：

    >>> data = urlencode(data_dict).encode('utf-8')

这里我们附代把结果编码成了字节，因为它会做为主体发送。在这一例子中，我们使用了`UTF-8`字符集。

接下来，我们构建我们的请求：

    >>> req = Request('http://search.debian.org/cgi-bin/omega',data=data)

通过把我们的数据做为`data`关键字的参数，这会告诉`urllib`我们想要把我们的数据做为请求的主体来发送。这会使请求以`POST`方式来发送请求，而不是以`GET`来发送。

接下来，我们添加`Content-Type`头字段：
    
    >>> req.addheader('Content-Type', 'application/x-www-form-urlencode;charset=UTF-8')
    
最后提交请求：

    >>> response = urlopen(req)

若我们把响应数据保存到文件中，并用浏览器打开它，我们会发现一些`Debian`网站关于`Python`的搜索结果。
    
 

### 表单检查

在前面一节我们使用了`URL` `http://search.debian.org/cgibin/omega`，和一个字典`data_dict = {'P': 'Python'}`.但这些是哪来的呢？

我们通过访问以前要靠手动提交来获得结果的网页来获得数据。然后我们检查这一页面的`HTML` 代码。若我们在浏览器中执行像之前那样的搜索，我们最可能会在网站`http://www.debian.org/` 右上角的搜索输入框中输入要搜索的内容，然后点击搜索。

绝大多数现代浏览器允许我们直接检查任何元素的源码。只需在我检查的元素上右击，这个例子中就是在搜索框中右击，然后选择**检查网页元素** 选项，如下面的截图：

![](https://i.imgur.com/3B23UnR.png)

源代码会在一部分弹出窗口中显示。前面的截图中是在屏屏右侧显示。你会看到像这样的代码：

    <form name="p" method="get" action="https://search.debian.org/cgi-bin/omega">
		<p>
            <input type="hidden" name="DB" value="zh-cn">
			<input name="P" value="" size="27">
			<input type="submit" value="搜索">
		</p>
	</form>

你会看到第二个`<input>`高亮了。这是对应搜索文本输入框的标签。高亮的`input`标签中`name`属性的值就是我们在`data_dict`字典中的键，在这里也就是 `P` 。在`data_dict`字典中的值也就是我们想要搜索的内容。

要得到`URL`，我们要看高亮`<input>`行上面的`<form>`标签。这里我们要的`URL`就是`action`属性的值：`http://search.debian.org/cgi-bin/omega`。这一页面的源码已经被下载下来，以防`Debian`页面在我们们到时发生了改变。

这一过程适用于绝大部分`HTML`页面。查找对应输入文本框的`<input>`属性，然后找到`<form>`标签。若你不熟悉`HTML`，可能会有一个试错的过程。在下一章我们会看到更多解析`HTML`的方法。
 
一但我们有了输入框的名字和`URL`链接，就可以构建和提交`POST`请求，正如前面一节看到的。

### HTTPS

除非有些要受保护，所有的`HTTP`请求和响应是明文发送的。可以访问数据传送这条网络的任何人都可能会截获我们的通信并无障碍读出。

由于网络用于传送相当多的敏感数据，阻止读取窃听通信数据者的方法已经有了，即使他们能截取数据。这些方法绝大部分都是实现某种形式加密。

加密`HTTP`流量的标准方法称为`HTTP`加密或`HTTPS` 。它使用一种称为`TLS/SSL` 的加密方式，它作用到`HTTP`运行基础的`TCP`协议上。`HTTPS`一般是使用`TCP`协议的`443`端口，对应的`HTTP`使用`80`端口。

对大部分使用者来说，这一过程基本是‘透明’的。原理上我们只需要把`URL`地址中的`http`改为`https` 。`urllib`是支持`HTTPS`的，我们的`Python`客户端也可以这样更改。

要注意的是，并不是所有的服务器都支持`HTTPS`，只是简单的把`URL`组合改成`https`并不能保证可运行于所有的站点上。若是这种情况，连接尝试可能会以多种方式失败，包括`socket`超时，连接复位错误，或者可能发生`HTTP`错误，比如状态码`400`或`500`范围内的错误。已经有越来越多的站点开始支持`HTTPS` 了。很多人切换到`HTTPS`上，并把它做为默认是协议，这会使你的应用给用户更多安全保障，所以检查它是否可用是值得的。

### Requests库

这是使用`urllib`库的介绍。如你所见，对于绝大多数`HTTP` 任务来说，标准库已经可以完全胜任了。我们尚未接触它所有功能。仍有众多的处理类我们没有讨论，更不用说‘打开器’ `opener`接口是可扩充的了。

然而，它的`API`不是最简练的，有多个尝试来提升它。其中之一就是非常流行的第三方库：`Requests`。在`PyPi`上以`requests` 库提供。它可以通过`Pip`来安装，也可以从[这里](http://docs.python-requests.org/ "requests库")下载，它提供相关文档。

我们看到的很多任务，`Requests`库都可以自动化和简化实现。说明它最快的方法就是试一些例子。

使用`Requests`库获取`URL`链接的命令和使用`urllib`完成是相似的：如下：

    >>> import request
    >>> response = request.get('http://www.debian.org')

我们可以查看响应对象的属性。试一下：
    
    >>> response.status_code
    200
    >>> reponse.reason
    'OK'
    >>> response.url
    'http://www.debian.org/'
    >>> response.headers['content-type']
    'text/html'

注意前面命令中的头字段名字是小写的。`Requests`响应对象的头字段属性的键是区分大小写的。

响应对象中添加了一些方便的属性：
    >>> response.ok
    True

`ok`属性表明请求是否成功了。表明请求包含`200`范围内的状态码。同样

    >>> response.is_redirected
    False

`is_redirected`属性表明请求是否重定向了。我们也可以通过响应对象来获取请求属性：

    >>> response.request.headers
    {'User-Agent': 'python-requests/2.3.0 CPython/3.4.1 Linux/3.2.0-4-  
    amd64', 'Accept-Encoding': 'gzip, deflate', 'Accept':     '*/*'}

要注意，`Requests`自动为我们处理了压缩。在`Accept-Encoding`字段中包含了`gzip`和`deflate`。若我们看响应的`Content-Encoding`会看到响应使用了`gzip`压缩，`Requests`自动为我们解压缩了：

    >>> response.headers['content-encoding']
    gzip

可以多种方式来查看响应的内容。像我们从`HTTPResponse`对象获得的字节流一样，可如下操作：

    >>> response.content
    b'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  
    "http://www.w3.org/TR/html4/strict.dtd">\n<html lang="en">...

`Requests`同样也会自动为我们执行解压缩。这样就可得到解码的内容：

    >>> response.text
    '<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  
    "http://www.w3.org/TR/html4/strict.dtd">\n<html  
    lang="en">\n<head>\n
    ...

注意现在是字符串了，不再是字节流。`Requests`库使用头字段中的值来选择一个字符集，并帮我们把内容解码为`Unicode`字符集。若它不能从头字段中得到字符集，它会使用[`chardet`库](http://pypi.python.org/pypi/chardet "chardet库") 通过容来做判断。我们看下这里`Requests`选择了哪种编码：

    >>> response.encoding
    'ISO-8859-1'

我们甚至可以要求它改变使用的编码方式：

    >>> response.encoding = 'utf-8'
    
在更改编码后，随后对响应的`text`属性的调用都会返回使用新编码解码的内容。

`Requests`库自动处理了`cookies`。尝试下面代码：

    >>> response = request.get('http://www.github.com')
    >>> print(response.cookies)
    <<class 'requests.cookies.RequestsCookieJar'>
    [<Cookie logged_in=no for .github.com/>,
    <Cookie _gh_sess=eyJzZxNz... for ..github.com/>]>

`Requests`库也有一个`Session`类，它允放`cookie`的重复利用，和使用`http`模块的`CookieJar`及`urllib`模块的`HTTPCookieHandler`对象相似。可像下面这样在后面的请求中重用`cookies`：

    >>> s = requests.Session()
    >>> s.get('http://www.google.com')
    >>> response = s.get('http://google.com/preferences')

`Session`对象与`Requests`模块有相似的接口，可以像使用`requests.get()`方法一样使用它的`get()`方法。现在所有碰到的`cookie`都保存在了`Session`对象中，在以后我们使用`get()`方法时，它们会自己随相应请求一起发送。

重定向同样自动跟踪，也和我们使用`urllib`库时一样，任何重定向请求都被捕捉存放在`history`属性中。

不同的`HTTP`方法易于使用，它们有自己的函数：

    >>> response = requests.head('http://www.google.com')
    >>> response.status_code
    200
    >>> response.text
    ''

添加指定的头字段到请求中与使用`urllib`时的添加方式相同：

    >>> headers = {'User-Agent': 'Mozilla/5.0 Firefox 24'}
    >>> response = requests.get('http://www.debian.org', headers = headers)

发送带查询字符串的请求是很直接简洁的：

    >>> params = { {':action': 'search', 'term': 'Are you quite sure this  
    is a cheese shop?'}}
    >>> response = requests.get('http://pypi.python.org/pypi',  
    params=params)
    >>> response.url
    'https://pypi.python.org/pypi?%3Aaction=search&term=Are+you+quite+sur  
    e+this+is+a+cheese+shop%3F'

`Requests`库为我们很好的处理了编码和格式的问题。

虽然我们使用了关键字`data`参数，`Post`投递同样也很简单：

    >>> data = {'P', 'Python'}
    >>>  response = requests.post('http://search.debian.org/cgi-  
    bin/omega', data=data)

#### 使用Requests库处理错误

在`Requests`中处理错误与在`urllib`中略有不同。让我们尝试一些错误情况并看它是如何工作的。如下产生一个`404`错误：

    >>> response = requests.get('http://www.google.com/notawebpage')
    >>> response.status_code
    404

在这种情况下，`urllib`会抛出异常，但`Requests`却没有。`Requests`库可以检查状态码并抛出相应的异常，但我们必须要求它这样做：

    >>> response.raise_for_status()
    ...
    requests.exceptions.HTTPError: 404 Client Error

现在试一个成功的请求：

    >>> r = requests.get('http://www.google.com')
    >>> r.status_code
    200
    >>> r.raise_for_status()
    None

它不做什么，在大多数情况下会让我们的程序退出`try/except`块并像我们希望的那样继续运各行。

若在更低的协议栈内发生了错误会怎样呢？试试如下：

    >>> r = requests.get('http://192.0.2.1')
    ...
    requests.exceptions.ConnectionError: HTTPConnectionPool(...

我们请求了一个不存在的主机，一旦超时，我们会得到一个`ConnectionError`异常。

在`Python`中使用`HTTP`时，与`urllib`库想比，`Requests`库减轻了我们的工作量。除非有要求必须用`urllib`，我建议在你的项目中使用`Requests`库。

### 小结


