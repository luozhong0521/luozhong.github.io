---
layout: post
title:  react native android 热更新（真机运行，打包文件，上传 ）
date:   2016-01-19 11:36:34 +0800
categories: jekyll update
---


### 运行环境

macOS Sierra 10.12.1

jdk 1.8.0_101

### 打包 apk

#### 生成签名秘钥

`keytool ` java证书管理工具

相关参数
	
	-genkey	//在用户目录中创建一个默认文件”.keystore”,还会产生一个mykey的别名，mykey中包含用户的公钥、私钥和证书
	
	-alias	//产生别名 每个keystore都关联这一个独一无二的alias，这个alias通常不区分大小写
	
	-keyalg	//指定密钥的算法 (如 RSA DSA，默认值为：DSA)
	
	-keystore	//指定密钥库的名称(产生的各类信息将不在.keystore文件中)
	
	-validity	//指定创建的证书有效期多少天(默认 90)
	
终端进入**工程目录**输入签名命令

	keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
	
这条命令会要求你输入密钥库（keystore）和对应密钥的密码，然后设置一些发行相关的信息。最后它会生成一个叫做my-release-key.keystore的密钥库文件

<image src="http://ok0ecqynu.bkt.clouddn.com/sing1.jpeg"/>

在运行上面这条语句之后，密钥库里应该已经生成了一个单独的密钥，有效期为10000天。--alias参数后面的别名是你将来为应用签名时所需要用到的，所以记得记录这个别名。

秘钥文件默认生在当前目录（mac是如此，windows不清楚）

**注意：请记得妥善地保管好你的密钥库文件，不要上传到版本库或者其它的地方。**

#### 设置gradle变量

1. 将`my-release-key.keystore`文件（根据自己命名不同）移动到工程`android/app`中
2. 编辑`/android/gradle.properties`文件，添加以下的代码

		MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
		MYAPP_RELEASE_KEY_ALIAS=my-key-alias
		MYAPP_RELEASE_STORE_PASSWORD=这里写刚才输入的密码
		MYAPP_RELEASE_KEY_PASSWORD=这里写刚才输入的密码

上面的这些会作为全局的gradle变量，在后面的步骤中可以用来给app签名。

***一旦你在Play Store发布了你的应用，如果想修改签名，就必须用一个不同的包名来重新发布你的应用（这样也会丢失所有的下载数和评分）。所以请务必备份好你的密钥库和密码。***

#### 添加签名到项目的gradle配置文件

打开项目目录下的`android/app/build.gradle`文件

在 **android** 字段中添加如下代码

	signingConfigs {
        release {
            storeFile file(MYAPP_RELEASE_STORE_FILE)
            storePassword MYAPP_RELEASE_STORE_PASSWORD
            keyAlias MYAPP_RELEASE_KEY_ALIAS
            keyPassword MYAPP_RELEASE_KEY_PASSWORD
        }
    }	
    
 在 `buildTypes`字段的`release `中添加
 
 	signingConfig signingConfigs.release
 
 添加好后就像下面代码的样子（省略号表示还有其他的默认代码）
 
	android {
	   
	    defaultConfig { ... }
	    signingConfigs {
	        release {
	            storeFile file(MYAPP_RELEASE_STORE_FILE)
	            storePassword MYAPP_RELEASE_STORE_PASSWORD
	            keyAlias MYAPP_RELEASE_KEY_ALIAS
	            keyPassword MYAPP_RELEASE_KEY_PASSWORD
	        }
	    }
	    buildTypes {
	        release {
	            ...
	            signingConfig signingConfigs.release
	        }
	    }
	}

#### 生成发行APK包

在项目中`android `目录下运行

	./gradlew assembleRelease
	
`./gradlew assembleRelease`表示执行当前目录下的名为gradlew的脚本文件，运行参数为assembleRelease

最后apk文件生成在`android/app/build/outputs/`文件夹中，我的叫做 **app-release.apk**，拷贝到手机上即可安装

### 集成codepush到安卓项目中

[参考文档1](https://microsoft.github.io/code-push/docs/react-native.html#link-5)

[参考文档2](https://github.com/Microsoft/react-native-code-push#android-setup)

codepush有两种方式导入，我这里选择的是[rnpm](https://www.npmjs.com/package/rnpm)的方式

#### 安装rnpm

终端输入
	
	npm i -g rnpm
	
#### 导入配置到项目中
	
进入项目中 `android`文件夹运行命令导入配置

	rnpm link react-native-code-push
	
修改`android/settings.gradle`文件，添加如下代码（貌似插件导入后会自动添加进入）

	include ':app', ':react-native-code-push'
	project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
	
修改 `android/app/build.gradle`文件 （貌似也会自动添加）

	...
	dependencies {
	    ...
	    compile project(':react-native-code-push')
	}
	
继续修改 `android/app/build.gradle`文件 ，添加代码（貌似还是会自动添加）

	apply from: "../../node_modules/react-native/react.gradle"
	apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
	
查看codepush 部署秘钥，拷贝出对应环境的 key （这点很重要）

	code-push deployment ls <appName> --displayKeys
	

	
修改`android/app/src/java/com/工程名/MainApplication.java`文件

React Native 版本 >= v0.29 添加代码如下样子（在`node_modules/react-native/package.json`中查看）
	
	import com.microsoft.codepush.react.CodePush;
	
	public class MainApplication extends Application implements ReactApplication {

	    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
	       
	        @Override
	        protected String getJSBundleFile() {
	            return CodePush.getJSBundleFile();
	        }
	
	        @Override
	        protected List<ReactPackage> getPackages() {
	           it, you can run "code-push deployment ls <appName> -k" to retrieve your key.
	            return Arrays.<ReactPackage>asList(
	                new MainReactPackage(),
	                new CodePush("这里填上文的 key ", MainApplication.this, BuildConfig.DEBUG)
	            );
	        }
	    };
	}	
	
修改 `android/app/build.gradle` 文件 中的 `versionName ` 改为 1.0.0（随便改成啥，但格式需要这样的，自行根据当前版本修改），因为 **codepush**需要三位

	android{
	    defaultConfig{
	        versionName "1.0.0"
	    }
	}

编辑 `android/index.android.js`文件，添加codepush代码

	import codePush from "react-native-code-push";
	
	
在 父组件的**componentDidMount** 方法中调用热更新方法

	componentDidMount(){
	    codePush.sync();
	  }

### react native 打包 bundle 文件

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

	react-native bundle --entry-file index.android.js --platform android --dev false --bundle-output ./android/bundle/index.android.jsbundle --assets-dest ./android/assets

此时成功会在***ios***目录下看到一个***bundle***目录，里面是我们的打包文件

	
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

	code-push release TripAnd ./android/bundle/index.android.jsbundle 1.0.0 

成功后重启app 则会看到更新（是重启app 需要结束程序后再打开）