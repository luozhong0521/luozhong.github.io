---
layout: post
title:  react native iOS 热更新（真机运行，打包文件，上传 ）
date:   2016-01-19 11:36:34 +0800
categories: jekyll update
---

### 本文使用的是 微软codepush cloud 服务。

### 热更新环境以及插件的安装

#### 这份文档建立在已经能在iOS模拟器上正常运行项目的基础上

[参考文章1-code-push](http://microsoft.github.io/code-push/docs/react-native.html#link-4)

#### 使用codepush做热更新

#### 运行环境

macOS Sierra 10.12.1

iOS10

xcode Version 8.1 (8B62)

#### CodePush简介

CodePush 是微软提供的一套用于热更新 React Native 和 Cordova 应用的服务。
CodePush 是提供给 React Native 和 Cordova 开发者直接部署移动应用更新给用户设备的云服务。CodePush 作为一个中央仓库，开发者可以推送更新 (JS, HTML, CSS and images)，应用可以从客户端 SDK 里面查询更新。CodePush 可以让应用有更多的可确定性，也可以让你直接接触用户群。在修复一些小问题和添加新特性的时候，不需要经过二进制打包，可以直接推送代码进行实时更新。

#### 安装与注册CodePush

使用CodePush之前首先要安装CodePush客户端，我的机器是OS Sierra 10.12.1

涉及到的指令：

	npm install -g code-push-cli //安装codepush客户端
	
	code-push register	//注册codepush账号
	
	code-push login	//登录
	
	code-push loout	//注销
	
	code-push access-key ls		//列出登陆的token
	
	code-push access-key rm <accessKye>	//删除某个 access-key
	

管理 CodePush 账号需要通过 NodeJS-based CLI，如果未安装则可以输入

	npm install -g code-push-cli	//-g表示全局安装

我的版本是

<image src="http://ok0dwebt7.bkt.clouddn.com/codepus.jpeg" />


#### 创建一个CodePush 账号

	code-push register

浏览器会自动打开注册页面，我这里用的是git账号授权的。

<image  src="http://ok0dwebt7.bkt.clouddn.com/reg.png" />

授权通过之后，CodePush会展示“access key”，复制此key到终端即可完成注册。

登录，登陆成功后，session文件将会写在 /Users/你的用户名/.code-push.config 中

	code-push login
	
	
至此 codepush客户端相关工作完成

#### 在CodePush服务器注册app（项目）

涉及相关命令

	code-push app add <appName> //在账号里面添加一个新的app
	
	code-push app rm <appName>	在账号里移除一个app
	
	code-push app rename <appName> //重命名一个存在app
	
	code-push app ls //列出账号下面的所有app
	
	code-push app transfer //把app的所有权转移到另外一个账号
	
		

为了让codepush关联上我们需要热更的app  则需要向codepush注册目标 app，我的app名字是Trip，请自行对应自己的app名称

	code-push app add Trip
	
	
#### 集成CodePush SDK 到 iOS项目中（目前我只做了iOS版）

##### 安装CodePush插件到项目中

我的项目目录结构是

<image  src="http://ok0dwebt7.bkt.clouddn.com/mulu.jpg" />

在项目根目录下执行指令，此时会在node_modules文件夹中生成react-native-code-push文件夹

	npm install --save react-native-code-push
	
##### 使用cocoapods导入code-push到iOS项目中 [参考文档](https://microsoft.github.io/code-push/docs/react-native.html#link-4)

cocoapods：简单理解一个iOS的依赖管理工具，有点类似前端的 [bower](https://bower.io/)

##### 首先安装cocoapods

	sudo gem install cocoapods
	
	如果报错

	ERROR:  While executing gem ... (Gem::DependencyError)
	Unable to resolve dependencies: cocoapods requires cocoapods-core (= 1.1.1), cocoapods-downloader (< 2.0, >= 1.1.2), cocoapods-trunk (< 2.0, >= 1.1.1), xcodeproj (< 2.0, >= 1.3.3)
	
	尝试运行
	
	sudo gem install cocoapods --pre 

在项目`ios`文件夹中新建`Podfile`文件内容如下，[参考文档](https://guides.cocoapods.org/contributing/contribute-to-cocoapods.html)

	platform :ios, '8.0'

	target 'Trip' do	//app名字自行修改
	    pod 'CodePush', :path => '../node_modules/react-native-code-push'//路径根据自己的文件目录修改
	end

项目`Podfile `同级目录下运行指令，即可将codepush插件集成到项目

	pod install




在xcode中打开项目

项目目录下有 *iOS* 文件夹，里面有一个叫 *xxxx.xcodeproj* 的文件 点击就可用xcode打开，打开后我的目录结构是这样的

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode1.jpg" />

在文件夹中打开项目文件夹，找到/node_modules/react-native-code-push/ios/CodePush.xcodeproj  这个文件

将他拖到xcode项目中的Libraries文件夹中

目的是为了将codepush集成到iOS项目中使用相关功能

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode2.jpeg" />

在xcode中 点击蓝色的项目名称呼出中间的操作栏，点击操作栏上面的 ***build phases*** 栏 打开***Link Binary With Libraries***

将 `Libraries/CodePush.xcodeproj/Products` 中的 `libCodePush.a`文件拖入`Link Binary With Libraries`中

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode3.jpeg" />

点击***Link Binary With Libraries***的加号，选择 ***libz.tbd*** 添加进去，添加上了，iOS编译出来的包中就会包含这些库。

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode4.jpeg" />

选择操作栏上部 ***Build Settings*** 栏目 在***Header Search Paths***那一项中加入 

	$(SRCROOT)/../node_modules/react-native-code-push
	
并且这一项的后面必须选择 ***recursive***  （用力敲黑板），这是让iOS知道去递归查找此依赖 否则有可能找不到
	
<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode5.jpeg" />

####代码中配置CodePush插件

打开xcode 在项目中找到 ***AppDelegate.m***文件，并单击左键打开 ，一般在项目名文件夹中 找不到就仔细点找

在代码头部加入代码并`cmd + s`保存，如果加入后报错，则对照上面的步骤仔细检查是否疏漏了，找不到此文件表明前面的依赖引入没有成功

	#import "CodePush.h"

继续在此文件中找到 `jsCodeLocation ` 关键词，将源码替换成以下代码

PS：让iOS判断是否是debug 和 release模式 对应加载不同的入口文件。我们热更时需要以 `release`方式运行

	#ifdef DEBUG
	    jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?	platform=ios&dev=true"];
	#else
	    jsCodeLocation = [CodePush bundleURL];
	#endif

启动命令窗口输入命令获取 `Staging ` key，复制 `Staging ` 的值（因为是在生产环境 所以使用 ***Staging***）

	code-push deployment ls  <AppName> --displayKeys
	
<image  src="http://ok0dwebt7.bkt.clouddn.com/codepush1.jpeg" />

打开xcode 将此值填入 ***info.plist***文件中 

添加 ***CodePushDeploymentKey*** 键 

将 ***Bundle versions string, short*** 值改成 ***1.0.0*** 因为iOS中codepush只支持这样的格式 

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode6.jpeg" />

xcode中点击项目名 调出中间的操作栏 点击 ***general*** 如果下面的 ***build*** 值为空  则改为1 （我这里是1 自己对应自己的项目）

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode7.jpeg" />

####react native 中使用插件

打开iOS的入口文件 我这里是 ***index.ios.js***

在代码中引入插件

	import codePush from "react-native-code-push";
	
在 父组件的 ***componentDidMount*** 方法中调用热更新方法

	componentDidMount(){
    	codePush.sync();
  	}
***以上就完成了基本的配置***

###react native 打包

[相关命令](https://github.com/facebook/react-native/blob/master/local-cli/bundle/bundleCommandLineArgs.js)

	react-native bundle
	
常用参数解释
	
	--entry-file //ios或android入口的文件名称，一般叫 index.ios.js或 index.android.js
	
	--platform //平台名称(ios/android)
	
	--dev //设置为false的时候会对JavaScript代码进行优化处理。
	
	--bundle-output //生成的jsbundle文件的名称(包含路径)，比如./ios/bundle/index.ios.jsbundle
	
	--assets-dest //图片以及其他资源存放的目录,比如./ios/assets
	
我的打包命令如下，我的静态资源放到 ios/Trip/assets 中的（依照个人的路径为准）

***特别注意：bundle 文件夹 需要事先手动建好***

	react-native bundle --entry-file index.ios.js --platform ios --dev false --bundle-output ./ios/bundle/index.ios.jsbundle --assets-dest ./ios/Trip/assets

此时成功会在***ios***目录下看到一个***bundle***目录，里面是我们的打包文件

打开xcode ，选中项目中和项目名一样的黄色文件夹，右键选择***Add Files to "RNIos"***

找到我们的***bundle***，注意是文件夹，不是文件，然后不要猴急猴急的点add按钮，请看下一步

在***option***中选择***Create folder references*** 然后点击 add

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode11.jpeg" />

完成后即可进行下一步的真机安装了


### iOS真机测试热更新

首先需要在apple[开发者网站](https://developer.apple.com/)去注册开发者账号（个人测试可以不交99的保护费）

xcode 中打开项目（参考上文如何打开项目）

点击项目，在操作栏中选择*** general*** 在team中选择自己的开发者账户

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode8.jpeg" />

按下 `cmd+shift+,`键打开配置窗口，在***build configration*** 中可以选择项目运行方式（debug/release）

debug：真机调试用，当修改代码后可以在真机上时时看到效果

releae：使用上文指定的bundle文件运行，等于与本地代码时时修改没有关系了（热更新需要使用这个模式调试）

将手机插上 ***usb*** 链接 ***mac***，点击左上角手机图标，即可看到我们的真机机器名，选择它，然后点击左边的三角形按钮运行项目。

坐等几分钟，不出意外的话即可在手机上安装app ，成功或失败 xcode会弹出提示。

<image  src="http://ok0dwebt7.bkt.clouddn.com/xcode10.jpeg" />

上述操作成功在手机上安装app后，就可以继续下面的打包发布了。


    
### 打包 bundle 文件

	react-native bundle --entry-file index.ios.js --platform ios --dev false --bundle-output ./ios/bundle/index.ios.jsbundle --assets-dest ./ios/Trip/assets
	
### 向codepush发布打包代码

随意修改代码，使其表现和手机上显示不一致即可

然后重新执行打包命令，生成一个新的bundle文件

执行codepush的发布命令，将新文件发布到***staging*** 环境中

相关命令

	code-push release <appName> <updateContents> <targetBinaryVersion> --mandatory //发布
	
	code-push deployment history Trip Staging  //查询发布历史
	
参数解释
	
	<appName> //app名称
	
	<updateContents> //Bundle文件所在目录（完整）
	
	<targetBinaryVersion>	// 需要热更的app 版本
	
	--mandatory //此参数非必须  如果有则表示app强制更新
	
我的命令如下

	code-push release Trip ./ios/bundle/index.ios.jsbundle 1.0.0 

成功后重启app 则会看到更新（是重启app 需要结束程序后再打开）

cocoapods用来管理依赖，类似前端的bower

一般一分钟成功

	sudo gem install cocoapods
	
	如果报错
	
	ERROR:  While executing gem ... (Gem::DependencyError)
    Unable to resolve dependencies: cocoapods requires cocoapods-core (= 1.1.1), cocoapods-downloader (< 2.0, >= 1.1.2), cocoapods-trunk (< 2.0, >= 1.1.1), xcodeproj (< 2.0, >= 1.3.3)
    
    尝试运行
    
    sudo gem install cocoapods --pre 

