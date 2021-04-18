**前言**

经过反复调查，chexx 的检测系统在今天 block 了我的ip，幸运的是没有被封号，但我也不知道为什么会被ban ip，反正发生了无论如何也要记录一下发生了什么，以便未来升级我的自动化程序。



**下面是 Chexx Block了我ip的迹象**

Chexx 官网

![image-20210419005329794](https://img.manesec.com/00/13.png)

毫无违和感的笑容，告诉你 呵呵呵呵呵。

![image-20210419005432978](https://img.manesec.com/00/14.png)

![image-20210419005441912](https://img.manesec.com/00/15.png)

官网搜索均没有问题，**但是点进去题目的时候，无论注册了还是没注册都会出现这个提示**：

![image-20210419005519238](https://img.manesec.com/00/16.png)

用的是 [perimeterx](https://www.perimeterx.com/whywasiblocked/) 的一家防机器人，自动化的企业，尝试寻找如何解除封锁，首先看到了上面的提示：

- `Access to this page has been denied because we believe you are using automation tools to browse the website.` 意思是拒绝你访问是因为你正在使用自动化的工具来获取里面的答案，我的猜想是他用了人工智能去判断我是否是机器人，因为 Chrome 的插件和网页的 JavaScript 是分割开的，这种网页一般不太可能扫到我的插件，所以我相信他不可能可以知道我在用插件提取答案。

- `Javascript is disabled or blocked by an extension (ad blockers for example)` Javascript是开启的，所以不可能会被禁用掉。
- `Your browser does not support cookies  `Cookies也一样是开启的。

这两个我都有开，ADBlock 没安装。

![image-20210419011030072](https://img.manesec.com/00/17.png)

直接上开发者观察发生了*肾么事*，从图片可以看到，发出去的第一个 GET 请求就被block了，也就是你的IP要是被ban了，第一个请求就是 403，这个时候你要是关掉插件还是什么的也没有机会去访问chexx的内容了，我还以为是那种简单的用 JavaScript 来判断你是否被block了，然后删掉div不给你答案，或者自动跳转等。

**大致的架构：**

```
You -> PerimeterX Bot Defender -> Chexx
```



**Mane Chexx Answer 返回结果**

![1](https://img.manesec.com/00/18.png)



**解决方案**

最后，强行换了一个IP就没了，IP多的很，不缺。



**总结**

其实可以看出，这种算法有一个很致命的缺点，就是开卷考试的时候，写个脚本拼命攻击 `PerimeterX Bot Defender`，考试ban了大家一起不要用，至于怎么实现，我就不说了。当然可以开个代理避免被人家攻击，但并不是所有人都会准备好代理去考试的。
