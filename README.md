# XsRead
自己学习写书源的过程，碰到的问题解决方式，作为备份。

本文是给自己做个备份，以免忘记香色闺阁的书源制作方法，APP版本是V2.56.1，使用的是EDGE浏览器（google也是一样的）。KDE connect同步iphone和PC的剪切板内容。

找到一个软件，可以在PC端读取和编辑香色闺阁的书源：https://github.com/LinShengwzp/source-reader/releases/tag/v1.0.0

所有书源的制作实际上都是模拟网页访问过程，并截取网页中需要的内容，阅读、香色等均是如此。总体过程是：
告诉服务器你需要什么内容；2. 服务器向设备返回网页内容；3. 从网页内容中截取我们需要的信息。 
	无论是搜索书籍、获取书籍基本信息、目录、正文，都是上面的过程。获取视频、音频、漫画均是类似的过程。

以下以正常网页访问过程为顺序说明香色闺阁的制作过程。

第一步是进入书源制作页面，香色闺阁只能在手机上制作，打开手机APP，点击左上角文件夹图标，在弹出的菜单中选择“站点管理”，每一行均表示一个书源，向左滑会出现“副本”和“编辑”，首先点击副本备份，然后点击编辑进入书源编辑页面。
	书源编辑页面中必须配置的选项：名称、host、httpHeaders、书籍搜索、章节列表、章节内容。其他的选填。

以下以https://www.bqgbi.com/为例进行说明

## 打开资源网站

正常访问网页一般会在浏览器的地址栏输入网址，例如`https://www.bqgbi.com`。这个过程对应书源编辑页面中的“基础信息”：
- 名称：资源网站的名称，例如笔趣阁。
	可以自定义，加emoj之类的，方便查看和分类书源。
- host：网址，例如`https://www.bqgbi.com`。
	注意，这里一般会去除com后面的斜杠/，我没试过加上会怎么样，最好去掉。
- httpHeaders：包含给`https://www.bqgbi.com`这个网址的服务器发送的信息。可以理解为，向服务器发送一些要求，告诉服务器需要什么。
	填入以下信息：
```
{"Referer":"https://www.bqgbi.com","User-Agent":"Mozilla\/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/73.0.3683.75 Safari\/537.36"}
```
把Referer后面的网址替换为需要的网址，其他的不用管。
这个过程在正常访问网页时也有，但是是浏览器在后台完成的，不可见。

## 搜索书籍

正常搜索书籍一般会在页面的搜索栏中输入搜索关键字，例如“都市”，然后点击搜索按钮即可出现搜索结果。这个过程对应书源编辑页面中的“书籍搜索”：

- 请求信息
	对应输入关键字点击搜索按钮的过程。
	填入以下信息：
```
@js:
let url=config.host+"/s"; 

let hh= {
  "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/604.3.5 (KHTML, like Gecko) Version/13.0 Safari/604.1"
};

let hp= {'q':params.keyWord};

return {'url':url, 'POST':false, 'httpParams':hp
, "httpHeaders": config.httpHeaders,forbidCookie:true};
```
或者
```
/s?q=%@keyWord
```

推荐第一种方式

我们需要通过如下方式获取需要的信息以填写上述代码中的某些信息。
![image](https://github.com/ckstc/XsRead/blob/main/img/Pasted%20image%2020230905134319.png)

![image](https://github.com/ckstc/XsRead/blob/main/img/Pasted%20image%2020230905135006.png)

有时候浏览器会显示链接为`https://www.bqgbi.com/s?q=都市`，有时为`https://www.bqgbi.com/s?q=%E9%83%BD%E5%B8%82`，可以看到`%E9%83%BD%E5%B8%82`和都市是同一个东西，只是编码问题，可自行百度。

下面解释一下如何根据上图中的信息修改代码：
第一句：不用改。“@js:”代表以下使用JavaScript语法。
第二句：必须改/s。`config.host`代表前面输入的host（这个例子中为`https://www.bqgbi.com`），`host+'/s'`后的完整的网址url为`https://www.bqgbi.com/s`。这里的url实际上就是`https://www.bqgbi.com/s?q=%E9%83%BD%E5%B8%82`中？之前的部分。在另一个网站中，网址为`http://www.duichongwang.com.cn/ar.php?keyWord=%E9%83%BD%E5%B8%82`，则“/s”应该被替换为`/ar.php`。
第三句：不用改。
第四句：必须要改'q'。这句对应？之后的部分，也就是上图“负载”显示的部分，在这个例子中参数为q，值为都市，图中表示为`q：都市`，在我们写的代码中以`'q':params.keyWord`表示。在另一个网站中，网址为`http://www.duichongwang.com.cn/ar.php?keyWord=%E9%83%BD%E5%B8%82`，则应该将q替换为？之后的keyWord。
第五句：必须改'POST'后面的值，根据情况写ture或者false。在第一个图中的红框中可以看到`请求方法：GET`，此时代码应该写成`'POST':false`，代表请求方法为GET；如果此处显示`请求方法：POST`，则代码应该写成`'POST':true`。

- 请求参数编码方式和响应编码方式，一般为utf-8，如果不行则换成GBK。

以上修改可保证能够正常进行搜索了，如果想了解其他参数的含义，可参考文末尾的链接。

- 响应规则
	也就是完成搜索后已经获得了搜索结果页面，如何提取这个页面中的信息。

*注：写响应规则的时候搞错了，搞成了http://www.duichongwang.com.cn，无所谓，实际内容没有本质区别。*

我们需要通过如下方式获取需要的信息以填写响应规则。

![image](https://github.com/ckstc/XsRead/blob/main/img/Pasted%20image%2020230905141521.png)

![image](https://github.com/ckstc/XsRead/blob/main/img/Pasted%20image%2020230905150535.png)

上图第3和第4步是为了定位我们检索到的书籍列表对应的代码的，相应代码在“元素”标签下面。可以看到，代码前面有个小三角，这表示代码是一层一层展开的，类似于文件夹的结构。上一级称为父节点，下一级称为子节点。每行代码都有一对小括号`<></>`，分别表示开始和结束。`<>`中，div、h2、ul、li、span均为标签，`<>`中标签后的内容为标签的属性，如class表示标签的类，id表示标签的唯一身份标识（类似于身份证），括号中间的黑色字体就是我们在网页中看到的文字内容，也就是我们最终需要的内容。

我们需要填写的内容实际上是为了从这种代码中提取出我们想要的信息，比如作品分类，名称，最新章节，作者。

列表（list）
表示整个列表，一个网页有非常多信息，我们需要先定位到搜索结果列表，才能获取想要的信息。此处代码为`//div[@class="row"]//ul/li`。可以点击“元素”下面的任意代码，然后按住ctrl+F，在弹出的搜索框中输入上述代码，可以直接定位到符合以上代码条件的标签，如果没有，则表明标签不正确。

 “//”表示：从当前节点选择子节点或者孙节点。比如，标签`<div class=“row”>`和标签`<ul class="txt-list txt-list-row5">`之间，还有一个节点`<div class="layout layout2 layout-co18">`，我们直接以“//ul”可以定位到ul标签所在的位置，即可以直接定位到孙节点。

 `div[@class="row"]`表示：定位到属性class为row的div标签，@表示选取属性。如果网页中有多个class为row的div标签，则可以通过`div[@class="row"][1]`、`div[@class="row"][2]`……来进行定位。

 “//ul”表示：定位到ul标签。由于`div[@class="row"]`后面均为其子孙标签，如果只有一个ul标签，则可不写ul的属性，当然，我们也可以写ul的属性，完整的代码为`//div[@class="row"]//ul[@class="txt-list txt-list-row5"]/li`，但是一般没必要。如果有多个属性相同的ul标签，则用`//div[@class="row"]//ul[1]`、`//div[@class="row"]//ul[2]`……进行定位。

 “/li”表示：定位到li标签。“/”只有一个斜杠表示子节点，不会定位孙节点，此处只会定位到li这一层级，不会定位到li的下一层级。可以看到，li标签下一层级的内容（span标签对于的内容）就是列表的内容，则li本身就表示列表，可以理解为表格中的一行。当然，也可以用另外一种写法，同样可以定位`//ul[@class="txt-list txt-list-row5"]/li`
	
  书名（bookName）
	上图中，书名在标签<span class="s2">下面，代码如下`//span[@class=“s2”]/a/text()`，由于我们已经定位到列表list了，此处可以直接定位标签li下的内容。span[@class=“s2”]表示定位到属性class为s2的span标签；span的下一级标签为a，所需要“/a”；/text()表示获取标签a对应的文字内容，也就是书名。
	
  作者（author）	`//span[@class=“s4”]/text()`
	
  类别（cat）	`//span[@class=“s1”]/text()`
	
  最后一章标题（lastChapterTitle）	`//span[@class=“s3”]/a/text()`，这里实际上就是最新章节。
  
  书本详情页地址（detailUrl）	`//span[@class=“s2”]/a/@href`，此处标签a的属性包含了一个地址`href=/ldks/44008/`，前面说过@表示取属性，也就是说@href表述取出了`/ldks/44008/`，然后与`config.host`拼接，即`config.host+/ldks/44008/`，也就是书籍的完整网址为`http://www.duichongwang.com.cn/ldks/44008/`。

  响应解析方式`格式化为DOM（html）`，以上列表中的代码均为xPath规则的，如果不选择这个解析方式会出错

## 书籍详情

- 请求信息`%@result`就是书本详情页地址（detailUrl）对应的页面
- 图标`//div[@class="detail-box"]//img/@src`
- 简介`//div[@class="detail-box"]//div[@class="desc xs-hidden"]/text()`
- 响应解析方式`格式化为DOM（html）`

## 章节列表

- 请求信息`%@result`，就是书本详情页地址（detailUrl）对应的页面
- 列表`//div[@class="section-box"][2]/ul/li`
标题`//a/text()`
下一级界面地址`//a/@href`
- 下一页地址`//div[@class="listpage"]/span[3]/a/@href`，有些网站章节分为多页，此时需要找到“下一页”的链接
- 最后一章更新时间`//div[@class="detail-box"]//p[5]`
- 响应解析方式`格式化为DOM（html）`
- moreKeys`{"maxPage":999}`，在设置了下一页地址的情况下，必须设置下一页的最大数目。

## 章节内容
- 请求信息`%@result`，就是章节列表中下一级界面地址
- 正文（content）`//div[@class="reader-main"]/div[@class="content"]/text()`
- 下一页地址`//div[@class="section-opt m-bottom-opt"]/a[contains(., \"下一页\")]/@href`
- 响应解析方式`格式化为DOM（html）`
- moreKeys`{"maxPage":2}`


## 缺陷：

1. 类别（cat）有时候包含多余的字符，需要用正则表达式去掉，如何操作还不会。不太影响阅读。//部分解决，使用js语法的replace()函数。

2. maxPage可以用js语法控制而非设置固定值，固定值不能适用于所有情况，暂时不会。数值设置过大会影响响应速度，设置过小又显示不全。//已解决，利用xpath自带的过滤方式。比如`//a[cantains(.,"下一页")]/@href`，则只会获取到a标签的内容包含下一页的链接，此时可将maxPage设为999。

3. 忘了页面的编码方式在哪查看了。//已解决，在网页源代码的head标签里查看`charset`。

4. 书籍分类还不知道怎么弄。//完成了最简单的写法
```
新建分类后
请求信息如下：
@js:
let url =config.host+"/"+params.filters.type +"?" + params.pageIndex;

let hh= {
  "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/604.3.5 (KHTML, like Gecko) Version/13.0 Safari/604.1"
}
return {'url':url,"httpHeaders":hh};

morekeys如下，只有一级页面：
{"pageSize":20,"requestFilters":[{"key":"type","items":[{"title":"玄幻奇幻","value":"xh"},{"title":"修真武侠","value":"xz"},{"title":"都市言情","value":"ds"},{"title":"穿越历史","value":"cy"},{"title":"网游竞技","value":"wy"},{"title":"科幻灵异","value":"kh"},{"title":"完结","value":"wj"},{"title":"排行榜","value":"ph"}]}]}
```
有时分类得到的detailUrl和搜索情况下得到的detailUrl不一样，如搜索的detailUrl有时没有config.host而，分类得到的detailUrl则有，此时需要分情况：
```
@js:
if(params.queryInfo.detailUrl.match("http")=="http")//如果detailUrl带有http，也就是包含config.host
{return params.queryInfo.detailUrl+"all_"+params.pageIndex+"/";}else //直接返回detailUrl
{let url = config.host+params.queryInfo.detailUrl+"all_"+params.pageIndex+"/";//否则，加上config.host。
return url;}
```

5. 搜索书籍时响应页面只有“加载中”而无内容，是因为网页时动态记载的。//已解决，需要在请求头的return中增加`webView:true,webViewJsDelay:3`

6. 控制台的“负载”显示`searchkey：（无法对值进行解码）`，且预览页面为乱码。//已解决，需要将书籍搜索的响应编码方式改为`简体中文（gb2312）`，尝试不同的解码方式。

7. 如果目录页面需要点击“查看更多”，怎么处理不知道。//一个成功的示例为，在“章节列表”的请求信息处不用`%@result`，而是用以下代码代替。
```
@js:
let r = result.replace(\".htm\",\"/\")
return r;
```

8. xbsrebuild使用方法//windows使用方式，（1）下载`xbsrebuild-windows-386.zip`，解压到特定目录，将exe重命名为`xbsrebuild.exe`；（2）windows11下，在该目录下右键并“在终端中打开”；（3）按照作者的说明，输入`.\xbsrebuild server`启动服务，不要关闭终端窗口；（4）将香色书源文件放在xbsrebuild.exe同一目录下，再次在该目录下右键并“在终端中打开”，打开另一个终端；（5）按照作者的说明，输入`.\xbsrebuild xbs2json -i xx.xbs -o xx.json`或者`.\xbsrebuild json2xbs -i xx.json -o xx.xbs`进行转换，其中“xx”需要自己改为对应名称，转换后的文件在同一目录下。`*.json`文件可以在`https://www.bejson.com/jsoneditoronline/`中格式化编辑。如果已经熟练了编写规则，可以使用这种方式在电脑上编辑书源完成大部分工作后，再在手机上测试调整。

9. 不知道怎么用js语句replace()调整result的内容。由于Xpath只能定位和提取内容，所有对内容的操作只能由js进行，虽然网上有很多js教程，但是我暂时不会在香色中写js，照着写的很多时候完全不生效。//错误引用了，xpath语句之后js开头应该是|@js:，也就是只有一个|而非两个

## 参考：

1. [XsRead/png/README.md at main · Fantuan-cell/XsRead (github.com)](https://github.com/Fantuan-cell/XsRead/blob/main/png/README.md)
2. [p1书籍搜索_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1na411q77A?p=1)
3. 【【阅读3.0】书源制作过程(基础语法)-哔哩哔哩】 https://b23.tv/TUMG3e5
4. [ne1llee/xbsrebuild: 香色闺阁xbs书源加解密工具 (github.com)](https://github.com/ne1llee/xbsrebuild)
5. [学爬虫利器XPath,看这一篇就够了](https://zhuanlan.zhihu.com/p/29436838)
6. [手把手教你学会Xpath高级用法](https://zhuanlan.zhihu.com/p/32187820)
7. [Xpath库](https://github.com/zhegexiaohuozi/JsoupXpath/blob/master/README.md)


```
一、格式化GET和POST请求
以下文字直接给chatgpt：

我现在需要通过网站自带的搜索方法（GET或POST方法）进行搜索，并提取搜索结果的信息。（1）GET方法。我会给你一个网址，你只需要将其中的关键词替换为自定义的%@keyWord，页码替换为自定义的%@pageIndex，其他内容保持不变。例如,我给你https://www.xbiquwx.la/search.html?searchkey=都市&page=1&Submit=submit&t_botton=t ，你只需要替换其中的searchkey和page，你应该给我输出https://www.xbiquwx.la/search.html?searchkey=%@keyWord&page=%@pageIndex&Submit=submit&t_botton=t。请注意，searchkey也可能叫q、key等等；page必然是数字，当然page也可能叫index等等。你只需要给我输出替换后的网址，不需要你解释，网址单独一行显示，不要给网址加上双引号。（2）POST方法。我会给你一个网址和请求参数，你需要将格式化为指定形式。例如，我给你网址https://m.youdian5.com/s.php，请求参数为：keyword: 都市 t: 1；或者keyword=都市&t=1。其中，https://m.youdian5.com替换为自定义的config.host，然后将参数格式化为'keyword':params.keyWord,'t':'1'。格式化后以如下形式显示@js: let url=config.host+"/s.php"; let hh= {"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/604.3.5 (KHTML, like Gecko) Version/13.0 Safari/604.1" }; let hp= {'keyword':params.keyWord,'t':'1'}; return { 'url':url,'POST':true,'httpParams':hp,"httpHeaders": config.httpHeaders, "forbidCookie":true, //'requestParamsEncode':'', //'responseFormatType':'json', //'cacheTime':3600, //'tryCount':1, //"webView":true, //'webViewJs':'document.documentElement.outerHTML', //'webViewSkipUrls':['https://www.baidu.com'], //'sourceRegex':'.*\\.(mp3|m4a).*' }

二、提取书籍信息

以下文字直接给chatgpt：
我需要提取网页HTML源码中的书籍信息，包括：列表list；书名bookName；作者author；图标cover；简介desc；类别cat；状态status；字数wordCount；最后一章标题lastChapterTitle；书籍详情页地址detailUrl。不同网站对列表所使用的html标签有区别，通常列表使用的列表有ul、ol、dt、table，对应的列表项的标签有li、dd、tr，但是也有的网站仅适用div标签的id或class属性设为list、searchresult，有的甚至设置为其他不常见的字符如col。有两种方式处理html代码：第一种，通过xpath获得书籍信息，例如：list://ul[@class=="fk"]；bookName://a[2]/text()；author://a[3]/text()；cat://a[1]/text()；其中，所有书籍信息都位于list所在的标签之内，所以bookName等信息省略了list的表达式//ul[@class=="fk"]。第二种，通过JavaScript的正则表达式获取书籍信息。functionfunctionName(config,params,result){letlist=[];letreg=/%3Cli%3E<a(?:\S|\s)*?>(.*?)<\/a><ahref="(.*?)"(?:\S|\s)*?>(.*?)<\/a>(?:\S|\s)*?<ahref="(.*?)"(?:\S|\s)*?>(.*?)<\/a><\/li>/gim;while(tem=reg.exec(result)){letbookInfo={};bookInfo.cat=tem[1];bookInfo.detailUrl=tem[2];bookInfo.bookName=tem[3];bookInfo.author=tem[4];list.push(bookInfo)}return{'list':list}}其中config,params,result均为自定义参数，result即为html网页代码。接下来，我会给你HTML代码，请分别给我xpth和JavaScript代码。对于xpath，你只需要像我一样给出xpath表达式即可；对于JavaScript，你需要严格按照我给你的示例代码进行代码编写，不需要扩展。我在前面列了10项具体信息，如果我给你的HTML代码中没有，仍然显示，但是留空。
```

```
原生能力支持使用params.nativeTool，有以下几个功能：

log(obj); // 打印log，key使用时间截
logWithKey(obj, strKey); // 打印log并自定义key

stringByObject(obj); // 将任意对象转换为字符串

deviceId(); // 默认的本地设备id，32位md5小写
deviceIdWithTemplateWithSeparator(strTemplate, strSeparator); // 自定义格式的本地设备id，strTemplate为模版，aaa-aa-aaaa，这里使用-分为3段，每段第一个字符将标识该段类型：0为纯数字，a为纯字母小写，A为纯字母大写，b为字符(数字+字母)小写，B为字符(数字+字母)大写，默认的deviceId模版即为：bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb

base64Decode(str); // base64解码，返回字符串
base64Encode(str); // base64编码，返回字符串
base64EncodeWithData(data); // 对二进制流(NSData)base64编码，返回字符串

readFile(strPath); // 从path中读取文件，返回二进制流
readTxtFile(strPath); // 从path中读取文件，返回字符串
unzipFile(strPath); // 解压zip文件，返回目录path
unzipFileWithPassword(strPath, strPassword); // 使用密码解压zip文件，返回目录path
allFilesAtPath(strDirPath); // 获取path目录下所有的文件path，返回数组:arr(path)

getCache(strKey); // 获取全局缓存对象
setCache(strKey, obj); // 设置全局缓存对象

sha1Encode(str); // 返回sha1
md5Encode(str); // 返回md5

cookieByKey(str); // 返回字符串
cookiesByUrl(url); // 返回数组

XPathParserWithSource(str); // 创建XPath解析器，可用于下面XPath解析器专用接口


XPath解析器接口有：
raw(); // 返回原始html
content(); // 返回内容
tagName(); // 返回字符串
attributes(); // 返回字典
queryWithXPath(strXPath); // 返回查询结果，以数组保存
```

```
看到一串代码：
$.path ||@js: let ids = params.nativeTool.dataByAesDecryptWithBase64StringWithKeyWithIv(result, "f041c49714d39908", "0123456789abcdef") return params.nativeTool.stringByObject(ids)

香色似乎还有其他加密解密函数？
```
