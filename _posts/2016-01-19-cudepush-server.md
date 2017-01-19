---
layout: post
title:  react native codepush之搭建自己的更新服务器
date:   2016-01-19 11:36:34 +0800
categories: jekyll update
---


[参考文章-code-push-server](https://github.com/lisong/code-push-server)

感谢上文作者的辛苦付出

本文简历在已经成功运行 微软 codepush热更新，并且了解codepush 相关指令的基础上。 

[参考文章-iOS](http://www.jianshu.com/p/19f23d66286f)

[参考文章-android](http://www.jianshu.com/p/eafe0136d3a3)

#### 简介
code-push-server是一个开源项目，基于 nodejs + mysql 搭建自己的热更新服务器

#### 环境

macOS Sierra 10.12.1

nodejs v4.3.1

mysql 5.6

#### 一、安装mysql（其他环境自行对应mysql安装）

推荐安装 `mysql 5.6`

[mysql 5.6下载地址](http://dev.mysql.com/downloads/mysql/5.6.html#downloads)

一键安装 毫无压力

##### 设置mysql密码

进入mysql安装目录，命令分步执行

	cd /usr/local/mysql/bin
	
	./mysql -u root -p  //这一步是登录root用户 回车即可，5.6默人密码为空
	
	修改密码
	
	set password = password('输入你的新密码');  //引号不能省略
	
	

##### 启动mysql服务

打开 系统设置，在面板下会出现一个mysql的图标，点击进入并启动即可。

<image src="http://ok0ekjrod.bkt.clouddn.com/mysql.jpeg"></image> 

#### 二、本地安装code-push-serve

作者发布了两种安装方式（npm安装或源码安装），在此我推荐使用源码安装，因为后期我们要基于这个服务修改自己的网页，源码安装方便些。

首先进入项目准备安装的目录执行以下命令（如果没有安装git 则可以去git上download下来解压）

	git clone https://github.com/lisong/code-push-server.git
	
clone完毕后执行

	cd code-push-server && npm install
	
修改`config/config.js` 文件，在 **db** 对象中添加数据库信息，参考如下配置，对应自己的用户名密码，数据库名称

	db: {
	    username: "root",	//
	    password: "123456",
	    database: "codepush",
	    host: "127.0.0.1",
	    port: 3306,
	    dialect: "mysql"
	  }
	  
初始化服务，项目根目录（code-push-server）下执行命令
	
	./bin/db init --dbhost localhost --dbuser root --dbpassword #初始化mysql数据库
	
	
上述无报错即可进行下一步

#### 三、配置服务器-存储在本地

修改`config/config.js` 

将 **common** 对象中的 **storageType**改为 **local** 

新建文件存储目录 `data`，`storage`，并修改配置文件

	local: {
	    //此地址为以上新建的文件夹，自己对应自己的路径
	    storageDir: "/Users/luozhong/work/reactNative/server/storage",
	    //ip地址改成自己设备对应的ip 这是下载地址 
	    downloadUrl: "http://192.168.201.113:3000/download"
	  }
	  
	 common: {
 		//此地址为以上新建的文件夹，自己对应自己的路径
	    dataDir: "/Users/luozhong/work/reactNative/server/data",
	    storageType: "local"	//选择存储类型，目前支持local和qiniu配置
	  }

启动服务

	./bin/www	//无报错信息即为正常启动，可以在浏览器中输入 http://127.0.0.1:3000查看，默认用户名密码是 admin 123456
	
#### 四、项目与服务建立链接

进入reactnative 项目根目录执行命令查看当前是否登录，因为是新服务，所以要先保证没有别的账号正在登录

	 code-push whoami
	 
如果报错如下，表示没有登录

	[Error]  You are not currently logged in. Run the 'code-push login' command to authenticate with the CodePush server.
	
如果没有报错 并且显示邮箱账号，则表示已经登录账户，则我们要先注销当前账号

	code-push logout
	
成功注销后执行登录指令，浏览器会自动打开本地服务登录页面，命令行中会提示输入key

	code-push login http://localhost:3000

输入账号和密码 `admin` `123456` 登录后点击按钮 `获取token` 并复制token到命令行中，并回车确认

	Successfully logged-in. //提示此表示登录成功
	
至此我们已经将codepush和我们自建的服务器关联起来了.

#### 五、注册应用

项目根目录下执行

	code-push app add Trip-ios	//项目名+iOS/android后缀

#### 六、项目中修改对应的key

查看key

	code-push deployment ls Trip-ios

将`Staging `值修改到对应的iOS或安卓项目中并重新打包安装

#### 七、发布更新

进入项目根目录执行以下命令，表示打包并发布，默认发布在开发环境

	 code-push release-react Trip-ios ios
	 
成功后即可打开app验证。
	 


