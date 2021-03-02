### Chrome抓包和Postman

打开Chrome的Developer Tools，可以看到网页HTTP的请求包

![1614688478452](/assets/1614688178452.png)

在右侧的“Headers”选项卡中，可以分别查看请求头和响应头，以及请求参数，在“Response”选项卡可以查看响应内容。

通过“view source”，可以查看未经解析的响应内容，可以将Request Header中的这些文本复制，去Postman中执行，这样可以很容易地修改参数，定制请求。

![1614688810957](/assets/1614688810957.png)

点击Postman的Headers设置的“Bulk Edit”按钮，将复制的请求头文本粘贴进去，就可以完成请求头的批量设置。

点击Postman右侧的“Code”按钮，可以将请求转换为多种语言（curl、Java、Python等等）代码：

![1614689165532](/assets/1614689165532.png)

某些网站（可能是因为采用了HTTP/2协议），请求头会比较特别：

![1614689290266](/assets/1614689290266.png)

比如，其中有 “:” 开头的请求头参数。

使用PostMan或者Python的`Request`模块执行这种请求，会提示有不合法的header名称；对于某些网站，将 “:” 删除或者将整个以 “:” 开头的参数删除，请求还是可以正常发送的，但是某些网站不行。