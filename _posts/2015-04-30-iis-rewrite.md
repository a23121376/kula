---
layout: postlayout
title: 独立主机IIS6.0环境下配置rewrite（discuz为例）
categories: [seo]
tags: [iis,web]
---

独立主机IIS6.0环境下配置rewrite（discuz为例）

discuz论坛，网址静态化对网站收录以及优化都有着重要的意义！那么，怎么实现经验化呢？下面，我们就详细的操作一下让大家看看！
1、进入seo设置页面，将会面的全部打钩钩！然后确认！

![](http://s4.sinaimg.cn/middle/9cbcfbcctx6Czzn8nV9c3&690)
 
2、查看当前的 Rewrite 规则，根据自己的实际使用情况选择合适自己的代码！（本人使用的是windows2003,IIS6.0的服务器！）

![](http://s12.sinaimg.cn/middle/9cbcfbcctx6Czzn9Rurbb&690)

 3、复制代码，在文本文档中另存为！　
windows 主机 保存为   httpd.ini
linux虚拟主机 保存为    htaccess
然后上传至根目录！

4、空间需要支持开通rewrite重写！
http://pan.baidu.com/share/link?shareid=279966250&uk=1460636505  点击下载软件！
（以下文章，引自http://www.hexiaojun.com/143.html，感谢无私分享）
　　下面来一步步开始安装：
　　我们先将将下载的 IIS Rewrite 组件解压，放到适当的目录（如C:\ISAPI_Rewrite3）下。
![](http://s5.sinaimg.cn/large/9cbcfbcctx6CzASz1pqb4&690)

5、解压完毕后，您需要给C:\ISAPI_Rewrite3目录加上Users的读和运行权限，不然可能会造成IIS无法启动。
 
6、设置好权限后，在 IIS 管理器里选择网站，右键选择“属性”，如下图所示：
![](http://s15.sinaimg.cn/large/9cbcfbcctx6CzB3Et7o4e&690)


7、然后选择ISAPI筛选器，然后点击“添加” 选型卡 如下图
![](http://s15.sinaimg.cn/large/9cbcfbcctx6CzB2MlgG1e&690)

8、　　点击添加选项卡之后，在筛选器名称填写iiswrite，可执行文件选取：C:\ISAPI_Rewrite3\ISAPI_Rewrite.dll ，也就是解压isapi_rewrite 3的文件夹路径。如下图：
![](http://s9.sinaimg.cn/large/9cbcfbcctx6CzB6mxegc8&690)

9、点击“确定” 按钮。
![](http://s5.sinaimg.cn/large/9cbcfbcctx6CzBd6D7m24&690)

10、　　重新启动 IIS
![](http://s3.sinaimg.cn/large/9cbcfbcctx6CzBeX1bsf2&690)

11、　　点击确定。
![](http://s10.sinaimg.cn/large/9cbcfbcctx6CzBhDMoNb9&690)

12、　　重新选择网站 => 右键“属性”=> “ISAPI 筛选器”，如果看到状态为向上的绿色箭头，就说明 IISRewrite 模块安装成功了。
![](http://s10.sinaimg.cn/large/9cbcfbcctx6CzBhDMoNb9&690)

　　注意：我们服务器系统是win2003，安装的都是IIS的web服务器，不是Apache的，按照本教程配置好VPS的伪静态后，您只需要把您网站的静态化规则文件命名为httpd.ini文件放在网站根目录也就是web目录下就可以了，无须通过网站后台设置，那是在Apache服务器下才那样设置的，有的shopex的程序需要先把httpd.ini放web下再在后台启用下伪静态设置。