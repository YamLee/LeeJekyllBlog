---
layout:     post
title:      "Android开发中Gradle的使用系列三：gradle依赖管理"
subtitle:   "Gradle依赖管理"
date:       2016-04-13
author:     "YamLee"
header-img: "img/post-bg-universe.jpg"
tags:
    - Android,Gradle,Dependency Manage
    - Gradle For Android
    - 翻译
---

# Android开发中Gradle的使用系列三：gradle依赖管理



>gradle中的依赖可以说是gradle引以为傲的一个特性，你只要需要添加一行代码，gradle就可以自动通过配置的依赖仓库去下载你所需要的第三方包，如果你依赖的某个项目还会依赖其他的其他的项目**（传递性依赖：transitive dependencies）**,gradle会自动解决其依赖
	
## 依赖仓库

传统的第三方包引用需要下载相应的Jar包，然后加入到项目中，这种方式一是找Jar包比较繁琐，再就是Jar包升级比较麻烦，但是通过在gradle中配置一个依赖仓库，那么在项目中只需要添加一行依赖的代码就能让gadle从指定的仓库中下载相应的依赖。

> gradle默认不会添加依赖仓库，但是Android Studio会在project的根build.gradle文件中添加如下代码

```
repositories{
		jcenter()
}
```
gradle支持三种依赖库：Maven，Ivy和静态文件或者目录（aar,jar等文件或者libs目录），添加依赖后，当要执行gradle task就会同步，gradle会将对应版本下载到本地（~/.gradle目录下）所以一个依赖版本只用下载一次。

添加依赖只需要在相应模块的build.gradle文件下添加如下代码块

```
dependencies {
	compile group: 'com.google.code.gson', name: 'gson',version:'2.3'
    compile group: 'com.squareup.retrofit', name: 'retrofit',version: '1.9.0'
￼}
```

从上可知，一行依赖的代码有三个要素：group，name,version,其中name是必须得，group和version是可选的，但是默认都是添加group和version保证其准确性和gradle的运行正确，gradle还提供如下简写方式，一般也是用的如下简写方式

```
dependencies{
	compile 'com.google.code.gson:gson:2.3'
	compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```

### 内置版本仓库
为了方便，gradle已经内置了三个Maven版本仓库，如果要使用，只需要在repositories代码块中进行声明，其内置的Maven版本仓库为：JCenter,Maven Center和local Maven repository。如下代码演示了如何添加版本仓库

```
repositories {
       mavenCentral()
	   jcenter()
       mavenLocal()
   }
```
mavenCentral和jCenter都是有名的版本仓库，不必同时使用，当前AndroidStudio默认使用jcenter，同时jcenter还支持Https，所以还是建议使用Jcenter

关于mavenLocal如果原先你使用Maven管理过依赖，那么在gradle中添加mavenLocal（）即可使用原先的依赖包，但是确保在你的用户主目录有.m2目录，在OS X或Linux系统式**~/.m2**,在Windowns环境下是**\%UserProfile%\.m2**目录下

### 远程仓库

有一些大的公司或者组织他们更喜欢将自己的开源出来的二进制包放在自己的Maven或者Ivy服务器下而不是发布到MavenCetral或者Jcenter上，所以如果要添加这些版本仓库里的依赖，可以参考如下代码

```
repositories {
       maven {
           url "http://repo.acmecorp.com/maven2"
       }
}
```
同理Ivy库如下

```
repositories {
       ivy {
           url "http://repo.acmecorp.com/repo"
       }
}
```

如果你的公司有自己的Maven或者Ivy库，同时需要通过用户名和密码验证，可以参考如下代码

```
 repositories {
       maven {
           url "http://repo.acmecorp.com/maven2"
           credentials {
               username 'user'
               password 'secretpassword'
           }
	} 
}
```

## 本地依赖

> 在某些情况下，你可能任然需要下载Jar包，或者要引用一些So包，因为公共的版本仓库上没有相应的包，这一节主要介绍这种情况下该如何配置gradle

### 文件依赖
添加一个Jar包作为依赖，你可以使用gradle提供的**files**方法，如下代码演示了如何使用

```
dependencies {
       compile files('libs/domoarigato.jar')
}
```

如果你有很多的Jar包，你可以直接添加jar包所在的目录

```
dependencies {
       compile fileTree('libs')
￼}
```

默认情况下，在AndroidStudio中新创建一个项目会默认创建libs目录，在build.gradle文件中默认添加了如下代码
```
 dependencies {
       compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
所以在AndroidStudio创建的项目中你可以直接将jar包放入libs目录下

### So包的引用

从C或者C++编译过来的针对不同平台So包的引用，你只需要在模块的根目录下创建**jniLibs**目录，然后针对不同的平台放入不同的平台编译过的so包，具体架构可看如下图

```
  app
   ├── AndroidManifest.xml
   └── jniLibs
       ├── armeabi
       │   └── nativelib.so
       ├── armeabi-v7a
       │   └── nativelib.so
       ├── mips
       │   └── nativelib.so
       └── x86
           └── nativelib.so
```

如果添加上述结构不管用，可能是AndroidStudio 版本或者build tools版本过低不支持，你可通过在build.gradle文件加入如下代码解决此问题

```
 android {
       sourceSets.main {
           jniLibs.srcDir 'src/main/libs'
       }
￼}
```

### 库项目的依赖（Library Projects）

> 如果你创作了一些很酷的UI控件或者实用的工具包，你可以通过创建Library Project来解决这个问题，Library Project能生成一个.aar的文件，这个aar文件即可以被app项目引用，或者你可以直接引用library模块，详细见下文

#### 创建和使用库模块
跟Android application plugin不一样，库项目在build.gradle文件的开头使用的是

```
apply plugin: 'com.android.library‘
```
有两种方式可以引用库模块，方式一是在你的项目（project）中直接引用，方式二是通过库模块生成aar文件，然后再到你的项目中引用aar文件

如果你通过方式一引用项目你需要在**settings.gradle**文件中添加如下样式的代码

```
include ':app', ':library'
```
然后在build.gradle文件的dependencies块中加入如下代码

```
  compile project(':library')
```

#### 使用.aar文件
如果你想在不同的项目中重用你的库组件而又不想导入库项目，那么你可以使用.aar文件。你可以在你的库项目的/output/aar/目录中找到生成aar文件，将aar文件添加成依赖，你可以在你的项目下创建**aars**目录，然后添加repository，如下代码所示

```
repositories {
       flatDir {
	   		dirs 'aars'
	   	 }
}
```
然后再添加依赖代码，如下所示

```
dependencies {
       compile(name:'libraryname', ext:'aar')
}
```


## Gradle依赖需要理解的几个概念

> gradle在配置依赖的时候有一些概念需要理解，即便现在我们可能还没有用到，但是理解这些如“compile”、“testCompile”这些scope属性的意义可以更好的使用gradle

### scope的配置
在配置依赖的时候有时候你可能只在某个特定的设备上使用，如你在某个设备上使用蓝牙的功能，你需要加入蓝牙sdk的引用，但是设备上本身含有蓝牙功能的二进制文件，所以你的sdk就不用加入的apk文件中，所以这个依赖范围的配置就应运而生

gradle中scope的标准配置类型为如下几种类型：

1. **compile：**此类型为默认的类型，所有配置为此类型的依赖包不仅会加入到classpath中同时也会加入到apk文件中

2. **apk：** 此类型只会加入到apk文件中，不会加入到编译的classpath中

3. **provided：** 此类型同apk类型正好相反，此类型只会加入到编译的classpath中，不会加入到apk中，apk和provide都只支持jar包，如果依赖library project会产生错误

4. **testCompile：** 当gradle运行用于单元测试的task时会将依赖包加入classpath

5. **androidTestCompile：** 当gradle运行function测试时会将此类依赖包加入classpath，并且加入到test apk中

以上五类为标准的配置类型，如果你还配置了build variant，就会有根据不同的build varialt配置，如**debugCompile**、**releaseProvided**等等

### 版本号的规定习惯

> 依赖包得版本号是依赖管理的一个重要标准，一个依赖包版本号的格式通常是按照 ***major.minor.patch***的格式来规定的，并且升级版本号通常按照如下标准来升级

* 当提供的api不兼容老版本了，即升级major
* 如果功能有大的改进，并且是兼容以前版本的就升级minor
* 当你修改了一下bug，打了一些补丁就升级patch

### 动态的版本号

> 在某些情况下，你可能需要所依赖项目最新版本，但是你有不能每次自己去查询版本然后再修改，这个时候你可以使用动态版本号，如下代码示例所示

```
dependencies {
       compile 'com.android.support:support-v4:22.2.+'
       compile 'com.android.support:appcompat-v7:22.2+'
       compile 'com.android.support:recyclerview-v7:+'
}
```

v4包是设置主版本号22，小版本好是2，补丁号使用最新的；v7包是在主版本号为22的版本下，小版本是最新的；而V7得recycleview包是当前能在版本仓库下获取的最新的，没有主版本号的限制

> 需要注意的是，当使用动态版本号时，有可能使得gradle获取依赖包时不稳定而导致构建失败，更严重可能导致获取到不同版本到本地缓存中从而引起程序调用api紊乱，Androidstudio也会在使用动态版本号时做出提示


## 总结

这一部分主要说明了如何在gradle中添加包的依赖，以及添加版本仓库的依赖，和libray project的依赖，同时对依赖中代码关键字的一些概念作了阐述。






	

	



