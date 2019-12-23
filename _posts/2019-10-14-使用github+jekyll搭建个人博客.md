---
layout: post
title: "github+jekyll搭建个人博客"
date: 2019-10-12 
description: "github+jekyll搭建个人博客"
tag: 博客 
---

　　最近发现github上可以免费的搭建个人博客，在网上找了很多教程之后，也算完成了。因为找了很多的教程，也遇到了很多的坑，现在打算自己写一篇简单易懂的博客。 

**1.拥有github账号**（已经有github账号及git的请跳过）  

​		若要使用github搭建免费博客，**github账号是必要的**，而本地的git工具并非必要。在浏览器上进入[https://github.com/](https://github.com/)**登录自己的github账号**。接下来的**不想麻烦的就可以直接去第2步**找自己主题了，若不想安装git工具则可跳过这步。想必作为一个程序员，自己的电脑上一般都会有安装git工具，如果没有就**请**去安装一个。对于github搭建的博客而言，主要用于更新和修改博客。因为在github上搭建的博客属于静态网页，需要在本地修改了之后push到远程仓库，进而实现博客的更新。  当然也可以在github网页上进行更新，这就是不需要本地git工具的理由之一。

​		此处附三个git安装教程：https://blog.csdn.net/qq_32786873/article/details/80570783 ，https://blog.csdn.net/weixin_43127338/article/details/83965053 ，https://www.cnblogs.com/bestchenyan/p/9298410.html 。如果还是有问题的话，请教一下同学同事什么的。  

**2.找自己喜欢的主题**  

​		可以去 [http://jekyllthemes.org/](http://jekyllthemes.org/) 找自己喜欢的博客模板。这里我随便挑了一个模板，点击Homepage进入该项目的仓库。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106145315.png">  
</div>

​		进入该项目的仓库后，先看看项目根目录是否有_post文件夹，这个文件夹主要是用来存放博客文章的。如果没有这个文件夹，说明这个项目不能发布文章。_image文件夹用来放图片，这个文件夹可能在根目录，也可能是某个子目录。确定是它了之后，就**直接fork**，简单粗暴。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106145730.png">  
</div>

​		在点击fork之后，可能会需要一点时间，稍等。fork成功之后如下所示。里面的很多东西都还需要修改，这里先不管。**点击Setting**，修改几个重要配置。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106151028.png">  
</div>

​		在Setting中**修改仓库名**，要想使用github搭建博客，这个仓库名就**必须是 自己账户名.github.io** （如：我的就是 hubeans.github.io）。非要问为什么的话，我也不知道。不过想想也有道理，大家的账户名都不一样，这样博客地址就不会一样了。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106151818.png">  
</div>

​		在修改了仓库名之后，往下拉找到 GitHub Pages 部分，就可以在这里看到自己的博客地址了。

​		到此，自己的博客已经算是搭建好了。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106152843.png">  
</div>

​		如果想要删除这个项目，不想要这个博客了。可以在Setting中继续往下拉，拉到Danger Zone （危险区），选择Delete this repository 根据弹出的提示就可以删除这个仓库。点击之后需要你确认是否删除当前仓库，需要在里面的文本框中再次输入这个仓库的名字，然后才能删除。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106153654.png">  
</div>

​		另外，如果不是很喜欢自己的博客地址，可以去买个域名。比如我就是买的阿里云的域名（hu12340.vip），双十一的时候买的，好像是12块一年。因为之前没有实名认证，所以搞得域名一直无法绑定解析成功。当时我在网上找了很多攻略，而攻略说的很简单（其实是真的简单，我没搞定是因为没实名认证），所以搞了很久才搞好。

​		此处以阿里云域名为例，进行域名绑定。

​		首先进入自己的阿里云域名控制台（控制台-域名）。

<div align="center">
	<img src="/images\posts\博客搭建\域名控制台.png">  
</div>

​		然后点击解析，进入域名解析控制台。

<div align="center">
	<img src="/images\posts\博客搭建\域名解析控制台.png">  
</div>

​		在添加记录之前，可以先去获取自己的博客的ip地址（在终端中ping自己的博客地址）。打开终端，然后输入ping 自己的博客地址 （如：ping hu12340.github.io）。

win：

<div align="center">
	<img src="/images\posts\博客搭建\win-终端.png">  
</div>

linux：

<div align="center">
	<img src="/images\posts\博客搭建\linux-终端.png">  
</div>

​		点击添加记录，完善各项属性。各项属性都已经给出了解释，如@表示不需要前缀等。可以自己去了(bai)解(du)各项属性值的意义。

<div align="center">
	<img src="/images\posts\博客搭建\添加记录.png">  
</div>

​		我自己的添加了两个记录，@和www。这样直接访问 hu12340.vip 和 www.hu12340.vip 都可以进入我的博客。

​		在添加了记录之后，需要在自己博客项目的根目录下添加一个CNAME文件。文件的名字就叫CNAME，然后内容就是自己的域名。

<div align="center">
	<img src="/images\posts\博客搭建\添加CNAME.png">  
</div>

​		提交后，github就会自动解析该文件，将其中的域名地址作为你的github博客地址（如你直接输入 自己的博客地址.github.io 会自动转到 自己的域名上）。

------

​		博客搭建好了之后，试着**在_config.yml中修改一些参数**，主要就是博客名及各种连接之类的。  如果电脑上没有安装git工具，就需要在仓库中修改配置，发布文章等。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106154642.png">  
</div>

尝试修改里面的一些配置：

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106155011.png">  
</div>

​		点击文件就可以进入该文件界面，然后进行操作，如修改等。点击about.md 后，如图所示：

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106155207.png">  
</div>

​		点击修改，随意修改里面的内容。这个文件是博客中的关于我页面，就写点关于自己的话就行。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106160043.png">  
</div>

​		修改完毕之后，下拉选择提交。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106160245.png">  
</div>

​		修改了about.md之后，可以试着去修改_config.yml 文件。这个文件里面主要是一些核心配置，因为是这个博客是英文的，我也是尝试着修改，然后对比变化。自己也尝试修改，每个人的想法都会不一样，不要扼杀自己的想法。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106162159.png">  
</div>

下面是我修改了这些属性后，博客的变化：

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106162458.png">  
</div>

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106162519.png">  
</div>



​		还不错吧，个人根据自己喜好去研究，去修改。.md有很多语法，如前面空两格、后面空两格、‘#’号、‘*’号等，会文章的展示都会有影响，有需要自己去学习一下。我自己是使用Typora写文章的。

​		个人比较推荐在电脑上安装git工具，方便管理。

​		电脑上如果有git工具，找个文件夹用于放博客文件目录。右键点击-选择Git Base Here，然后使用git clone把自己远程仓库上的博客项目拉下来，其中有一个_posts文件夹，里面放的就是文章。那么，我们就可以在这里对博客上的文章进行增删查改了。改完之后add、commit、push到远程仓库就行了。  

​		在仓库中复制自己的项目地址，然后在本地文件夹中clone：

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106163203.png">  
</div>

​		右键点击-选择Git Base Here，输入clone命令 git clone 项目地址，然后回车，就可以把项目拉到本地。

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106163345.png">  
</div>

​		如下图所示，项目clone成功，文件夹中会多出一个文件夹，这个文件夹就是你的博客项目。可以在里面进行增删改查，然后git add . -> git commit -m "修改描述" -> git push -u origin master 这样就可以把本地项目提交到远程仓库。如果对git命令不是很了解，可以看下我写的关于git命令的文章 [git常用命令](https://hu12340.github.io/2019/10/git%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4/)

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106163620.png">  
</div> 

<div align="center">
	<img src="/images\posts\博客搭建\QQ图片20191106163649.png">  
</div>

​		现在的电脑还没有push权限，需要添加配置ssh后才能将本地代码push到远程仓库。

<div align="center">
	<img src="/images\posts\博客搭建\添加SSH.png">  
</div>

​		将本地电脑中的 .ssh文件夹下的id_rsa.pub文件的内容复制到key文本框中。.ssh文件夹是一个隐藏文件夹，需要开启可查看隐藏文件夹才能看到。一般linux系统，该文件夹在～目录下（即用户目录）；而windows系统的.ssh文件夹在C:\Users\Administrator\.ssh。

<div align="center">
	<img src="/images\posts\博客搭建\Add-SSH.png">  
</div>

​		至于博客的更新，网上也有很多教程。如：https://www.cnblogs.com/wxyww/p/xiaoshujiang.html 这个教程中是使用了**小书匠**进行更新。我之前也试了下，百度直接搜索小书匠就能搜到。其需要绑定token值、github仓库和分支。成功后，即可直接在小书匠上更新文章。因为它绑定了你的github仓库，所以保存和另存为的文章直接就同步到了你的github仓库，比较方便。前提是绑定成功了的话，我在自己的电脑上绑定倒是挺顺利的。不过在工作电脑上绑定时，一直绑定不成功，右上角的提示里啥也没有。再重新申请一个token去绑定后倒是成功了。不过绑定好后，里面github同步文件的时候load出错，同事是再申请一次token，再重新绑定才成功的。感觉比较麻烦，所以我还是比较倾向使用**typora**编辑器。windows下载地址： https://www.typora.io/#windows  。在本地更新好文章后，直接add、commit、push三连就可更新博客上。如果push后博客还是没变化，请稍等，多刷新几次，应该是缓存或者网络原因。另外，**文章格式很重要**，最好先在原有文章上修改，然后另存。比如我使用的这个模板，它的命名就必须是：年-月-日-文章名 这种格式（如2019-10-14-使用github+jekyll搭建个人博客.md）。之前跟着原有文章一直该，可始终无法显示的博客中，后来才发现是文件命名的问题。

​		至于插入图片，对html有了解的朋友应该比较熟悉。那就是使用div和img，将图片插入到网页中。其中，所使用的图片也需要放入项目的image文件夹下，一起push到git仓库。

<div align="center">
	<img src="/images/QQ图片20191014142149.png" />  
</div> 

​		如果对html不熟，也不想使用这种方法，那我们可以使用typora直接插入图片地址。上传图片到自己的github上（当然也可直接用网上的图片地址），在项目中找到该图片并复制图片地址。

<div align="center">
	<img src="/images/posts/博客搭建/1571617967820.png">  
</div>

​		在typora中右键选择插入-图像，然后选择输入图片地址，粘贴即可。

<div align="center">
	<img src="/images/posts/博客搭建/1571618156316.png">  
</div>

​		不过我发现在项目中找到该图片并复制图片地址这种方法，很多时候图片加载不出来，加载很慢。不是很推荐这种做法。

希望这篇文章对你能有所帮助。

<div align="center">
	<img src="/images/路飞.jpg">  
</div> 

