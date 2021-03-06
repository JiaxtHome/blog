---
title: hexo+github搭建个人博客
categories: Hexo
tags: [hexo,next,个人博客]
---

# 配置github端

## 准备
github配置前提：
1. 已申请github账号
2. 本机已安装git
3. 本机ssh key已配置到github

以上操作，不明白的可自行网上搜索

## 创建GitHub Pages
（1）新建一个repository库（名称随便写，没必要一定是xxx.github.io）
![image](hexo/new_rep.png)
（2）配置github pages
在settings面板中选择将"master"分支作为github pages站点
![image](hexo/github_pages.png)
（3）新建一个backup分支，后面备份的时候会用到
![image](hexo/new_branch.png) 
github端配置完成～

---

# 配置hexo端
## 安装Node.js和npm
Node.js是hexo的运行环境，npm是Node.js的包管理器，这两个是hexo运行的必要工具
（1）去[**Node.js中文官网**](http://nodejs.cn/) 下载最新版本
![image](hexo/nodejs.png)
（2）解压tar.xz格式并移动到/usr/local/node

	$ xz -d node-v6.10.3-linux-x64.tar.xz
	$ tar -xvf node-v6.10.3-linux-x64.tar
	$ sudo mv node-v6.10.3-linux-x64 /usr/local/node
	
（3）配置环境变量

	$ sudo gedit /etc/profile
	
在最后添加如下一行

	export PATH=/usr/local/node/bin:$PATH
	
重启电脑或导入环境变量

	$ source /etc/profile	
	
（4）检测

	$ node -v
	   v6.10.3
	$ npm -v
	   3.10.10
	
如果版本显示正常，则代表安装完成。

## 安装hexo
直接命令行安装

	$ sudo npm install -g hexo-cli

新建站点目录，初始化hexo

	$ cd blog
	$ hexo init //初始化hexo到当前目录
	$ hexo g //生成静态网页
	$ hexo s //启动本地服务器

此时，在浏览器输入 http://localhost:4000 就可以在本地浏览效果了。

---

# 联调与发布
下面我们需要将本地静态网页发布到github上。
（1）配置站点配置文件_config.yml
>配置文件有两个：一个是站点配置文件，位于站点主目录下；另一个是主题配置文件，位于themes目录对应主题下
	
	deploy:
  		type: git
  		repository: git@github.com:JiaxtHome/blog.git
  		branch: master
	# URL
		## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
		url: https://jiaxthome.github.io/blog/
		root: /blog
	
注：URL需要配置，否则下面更换主题无法生效
其他详细配置可参考[**官方配置文档**](https://hexo.io/docs/configuration.html)

（2）安装自动发布模块

	$ npm install hexo-deployer-git --save
	
（3）发布到github

	$ hexo clean //清除缓存
	$ hexo g //生成静态网页
	$ hexo d //发布到github
	
此时在浏览器中输入https://jiaxthome.github.io/blog/ 就可以看到真正的在线效果了。

（4）发布一篇新博文

	$ hexo n "title" //new 一篇新文章
	
此时，会在source/_posts目录下会生成title.md文件，当然也可以直接复制一个已有的.md文件到该目录下，编辑后重新发布即可。

---

# 更换主题
以Next主题为例
（1）下载主题

	$ cd your-hexo-site
	$ git clone https://github.com/iissnan/hexo-theme-next themes/next
	
（2）修改站点配置文件_config.yml

	theme: next
	
此时重新发布后就可以看到next主题效果了。

（3）主题配置
各个主题为了满足个性化，都有自己的主题配置文件_config.yml，位于themes目录对应主题下，可以参考各主题的官方文档配置。比如，Next提供了三套样式可供选择：

	# Schemes
	scheme: Muse
	#scheme: Mist
	#scheme: Pisces

其他详细配置可以参考[**Next主题官方文档**](http://theme-next.iissnan.com/)。

---

# 图片存储
关于MarkDown文件引用图片的存储问题可以使用"图床"解决，但存在不安全因素。这里介绍hexo提供的方案：
（1）修改站点配置文件_config.yml

	post_asset_folder: true
	
（2）安装 hexo-asset-image插件

	$ npm install https://github.com/CodeFalling/hexo-asset-image --save
	
此时通过 hexo n 命令生成博文的同时，会创建一个同名的文件夹，引用的图片资源就可以放在这里了。如果是拷贝的.md文件，手动创建一个同名文件夹就可以。

（3）引用图片

	![logo](imagefold/logo.jpg)
	
这样，我们的图片资源就随着静态网页一起发布到github上了。

---

# 备份与恢复
个人博客的所有资源都需要我们自己维护的，如果文章和图片全放在本地，会有两大弊端：

- 一旦本地文件丢失，不可恢复
- 只能在本地电脑发布，无法协同工作

要解决这个问题，我们需要：

1. 将本地资源备份到backup分支，用git做同步管理
2. 在需要协同工作的多个电脑上都安装hexo环境

这样就实现了backup分支管理资源，master分支发布网站，具体步骤如下：
（1）将backup分支设置为默认分支
![image](hexo/default_branch.png)
注：如果之前忘记建backup分支，现在新建backup分支会包含master分支内容，同步比较麻烦，建议直接重建仓库。

（2）新建.gitignore文件

	/.deploy_git //部署用的文件，不需要备份
	/public //生成的网站内容，每次部署都会更新，不需要备份
	db.json //一些缓存数据，每次部署都会更新，不需要备份

（3）初始化本地仓库

	$ git init
	$ git remote add origin git@github.com:JiaxtHome/blog.git

（4）上传本地资源
	
	$ git add .
	$ git commit -m "init"
	$ git push origin backup
	
（5）新电脑同步资源

	$ git fetch --all
	$ git checkout -b backup remotes/origin/backup

重新编辑发布后，push到远程同步。

