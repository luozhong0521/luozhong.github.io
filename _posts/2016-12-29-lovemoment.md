---
layout: post
title:  react native iOS 端常见问题与解决方案
date:   2016-12-29 11:36:34 +0800
categories: jekyll update
---

[react-react-native-的es5-es6写法对照表](http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8)

### 一、搭建环境
[参考地址-中文](http://reactnative.cn/)

[参考地址-官网](http://facebook.github.io/react-native/)

#### 需要资源

硬件 

	mac 电脑
##### 软件

	xcode 	
	nodejs
	Homebrew //Mac系统的包管理器，用于安装NodeJS和一些其他必需的工具软件。
	Yarn	//React Native的命令行工具（react-native-cli）
	Watchman	//由Facebook提供的监视文件系统变更的工具。安装此工具可以提高开发时的性能（packager可以快速捕捉文件的变化从而实现实时刷新）。
	Flow	//静态的JS类型检查工具

#### 安装环境

##### 安装Homebrew

	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

在Max OS X 10.11（El Capitan)版本中，homebrew在安装软件时可能会碰到/usr/local目录不可写的权限问题。可以使用下面的命令修复：

	sudo chown -R `whoami` /usr/local
	
##### 配置node源

	npm config set registry https://registry.npm.taobao.org --global
	npm config set disturl https://npm.taobao.org/dist --global
	
##### 安装Yarn

	npm install -g yarn react-native-cli
	
如果你看到`EACCES: permission denied`这样的权限报错，那么请参照上文的homebrew译注，修复/usr/local目录的所有权：

	sudo chown -R `whoami` /usr/local
	
##### 安装xcode

[下载xcode](https://developer.apple.com/xcode/)

下载完成后执行默认安装即可

##### 安装Watchman

	brew install watchman

##### 安装Flow

	brew install flow
	
### 二、新建工程

在目标文件夹中输入命令，初次执行此命令时间稍长

	react-native init AwesomeProject //初始化一个名为 AwesomeProject 的项目

### 模拟器运行项目

##### 启动项目（两种方式）

1、命令启动方式进入项目目录执行  `react-native run-ios`

	cd AwesomeProject && react-native run-ios

2、xcode启动项目

双击打开工程目录下 `ios/AwesomeProject.xcodeproj`文件，这个文件是iOS工程文件。

功能简介：

* 🔨图标表示运行设备，模拟器或者真机，需要手动选择（目前只能选择模拟器）
* 左上角三角形`run` 
* 黑色正方形：停止run

选择一个模拟器

点击黑色三角形即可运行项目

<image src="http://ois2zrmyi.bkt.clouddn.com/xcode1.jpeg"/>


### 三、调试

聚焦到模拟器上

选择左上角 `Hardware/Shake Gesture` 此时会在模拟器上呼出调试面板

 <image src="http://ois2zrmyi.bkt.clouddn.com/xcode2.jpeg"/>
 
 
 <image width="300px" src="http://ois2zrmyi.bkt.clouddn.com/xcode3.jpeg"/>

* 点击 **Debug JS** 浏览器会自动打开 `http://localhost:8081/debugger-ui` 页面，任何在react antive中的console即可在这个页面的控制台展示
* **Enable Live Reload** 开启自动刷新页面功能，项目中保存后页面自动刷新

### 四、模拟器基本操作快捷键

* `cmd + r`键刷新模拟器页面
* `cmd + shift + h` 回到主页（相当于真机按下home键）
* `cmd + shift + h + h` 快速点击两次 h （相当于双击home键）

### 真机运行

* [注册开发者账号](https://idmsa.apple.com/IDMSWebAuth/login?appIdKey=891bd3417a7776362562d2197f89480a8547b108fd934911bcbea0110d07f757&path=%2Faccount%2F&rv=1)
* 双击打开工程目录下 `ios/AwesomeProject.xcodeproj`文件启动xcode
* 将手机连接到电脑，并在手机弹框上点击 **信任** 
* 单击左侧栏工程名，打开控制面板，选择 `General`下的 `Signing`，勾选`Autimatically manage signing`
* 在 `Team` 选项中选择 `add an account` 添加自己的开发者账号
<image src="http://ois2zrmyi.bkt.clouddn.com/xcode4.jpeg"/>

* 在左上角黑色正方形旁边的设备中选择连接电脑的iphone名称，中间顶部状态栏会展示当前设备准备情况。
<image src="http://ois2zrmyi.bkt.clouddn.com/xcode5.jpeg"/>
* 当设备准备就绪后 点击黑色三角形或者按下`cmd + r`键运行程序，随后手机上即可安装
* 安装ok后点击图标启动app，如果弹出需要信任证书则进入 `系统设置-通用-设备管理`中找到相关证书，点击验证即可*
* 进入app界面后摇晃手机即可呼出开发者面板，即可选择**自动刷新**或者 **调试模式**

***温馨提示：摇晃手机记得拿稳***

### 五、真机运行-切换运行模式

运行模式分为**debug** 和 **release** 两种

##### debug即程序调试时候，此时应用运行基于我们的未打包的js文件，可实时修改预览

##### release即发布，xcode会自动打包js文件

同时按下 `cmd + shift + ,`三个键 唤起设置界面 

<image src="http://ois2zrmyi.bkt.clouddn.com/xcode11.jpeg"/>


#### 真机调试报错情况

##### 1. 在运行release模式的时候 出现如下错误，删除这个test文件再运行即可
 
<image width="300px" src="http://ois2zrmyi.bkt.clouddn.com/xcode8.jpeg"/>

##### 2. 运行debug时出现以下错误，需要clean，同时按下 `shift + cmd + k` 完成清除

<image width="300px" src="http://ois2zrmyi.bkt.clouddn.com/xcode12.jpeg"/>

### 五、修改icon，appname和启动画面

#### 修改icon

* appstore 搜索 **Prepo** 下载安装
* 启动**Prepo** 将需要的图标（一张原图）拖放到软件中即可生成全部类型的icon，导出即可
* xcode中打开项目
<image width="100%" src="http://ois2zrmyi.bkt.clouddn.com/xcode6.jpeg"/>
* 将导出的icon拖到对应的地方，2x、3x表示倍数。大小为像素，例如第一个图标为 20x2=40像素 的图标
* 重新运行项目即可看到图标
 


#### 修改appName

* 修改info.plist文件，新建名为 `Bundle display name` 的key，值为 app的名字，这个值在发布 ipa包的时候需要
* 修改 `Bundle name` 值为 app的名字，在真机运行的时候需要 
<image width="100%" src="http://ois2zrmyi.bkt.clouddn.com/xcode7.jpeg"/>
* 再次运行项目即可看到更新的应用名


#### 修改App Transport Security Settings

添加方法参考上文  ***修改appName***

iOS 10 添加了https安全策略 为了继续使用 http 需要做以下修改

	App Transport Security Settings 添加 Allow Arbitrary Loads 布尔值 YES

<image width="100%" src="http://ois2zrmyi.bkt.clouddn.com/xcode9.jpeg"/>

#### WebView调用相机闪退

在info.plist文件下添加如下key

	相机权限

	<key>NSCameraUsageDescription</key>
	
	<string>cameraDesciption</string>	 
	
	相册权限
	
	<key>NSPhotoLibraryUsageDescription</key>
	
	<string>photoLibraryDesciption</string>

<image width="100%" src="http://ois2zrmyi.bkt.clouddn.com/xcode10.jpeg"/>

