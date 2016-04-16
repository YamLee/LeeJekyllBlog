---
layout:     post
title:      "Android开发中Gradle的使用系列四：创建Build Variants"
subtitle:   "Gradle Build Variants"
date:       2016-04-16
author:     "YamLee"
header-img: "img/post-bg-universe.jpg"
tags:
    - Android,Gradle,BuildVariants, 
    - Gradle For Android
    - 翻译
---

# Android开发中Gradle的使用系列四：创建Build Variants

> 当你在开发一款应用时，通常会面临发布不同的版本需求。举两个常见的场景，场景一：你正在增加新功能，然后你需要发布版本提交给QA，测试通过后再发布线上版本，可能线下版本和测试版本的服务器接口域名不一样又或者有不同的api接口；场景二：你的app需要发布一个免费版本和付费版本，付费版本会有更高的使用权限，针对如上两种情况你就需要发布四个apk，免费QA版，免费线上版，付费QA版，付费线上版，如果在代买里面硬编码会使得项目异常复杂。gradle针对这种情况，提出了解决这种问题的方法，对于QA版或线上版可以配置**build type**，Androidstudio默认配置了Debug和Release两种type，对于付费版或者免费版，可以通过配置**Build flavors**。这两种类型组合起来就叫做**build variant**

## Build types

Gradle在Android中的build type是用来处理app或者library应该被构建成什么类型，在这个配置中，你可以定义应用的包名是什么，是否自动去除掉没有引用的资源，是否开启混淆等等。具体配置可参考如下代码

```
android {
       buildTypes {
           release {
               minifyEnabled false
               proguardFiles getDefaultProguardFile
                 ('proguard-android.txt'), 'proguard-rules.pro'
           }
		} 
}
```
当你新建一个项目或者模块时，build.gradle文件会默认配置上release的build type，默认配置屏蔽了混淆功能（即将 minifyEnabled设置为false）和定义混淆文件的位置。
除了release build type再就是debug build type也是默认创建好的，但是没有在代码里显示出来，如果你想修改默认Debug中的配置，你需要自己声明出来

>debug的build type默认设置debug属性为true，方便调试

### 创建build types
当你觉得默认的两种类型不够用是，你也能很容易的自定义build type，如下代码展示了创建一个新的build type名叫**qa_test**的类型

```
android {
    buildTypes {
		qa_test {
   			applicationIdSuffix ".qatest"
    		versionNameSuffix "-qatest"
    		buildConfigField "String", "API_URL","\"http://staging.example.com/api\""
    		} 
	}
}
```

上面的代码主要配置qa_test的类型功能为：给包名加上了.qatest的后缀，这样你能在手机上安装相同的程序，应为包名不一样；在version name上加上了-qatest的后缀，然后再BuildConfig类中加了一个API_URL的string属性。

在自定义build type是，你还可以重用已用的build type，如下代码所示

```
 android {
       buildTypes {
           qa_test.initWith(buildTypes.debug)
           qa_test {
               applicationIdSuffix ".staging"
               versionNameSuffix "-staging"
               debuggable = false
} }
}
```
**initWith()**方法为qa_test类型拷贝了debug类型的所有属性，但是你也可以修改其中的属性，通过显示的声明出来

### 不同build types的代码目录结构设置

当你创建一个新的builld type，gradle也会默认为你指定一个同名的目录在你的项目里面，但是不会把目录自动创建出来，所以你需要创建一个同名的目录，其结构如下所示

![](http://ww4.sinaimg.cn/large/6b051377gw1f2ylgvqp1pj20ry0yodjb.jpg)


这种配置可以使得你在任意build类型去自定义修改，比方说在release的类型登录界面带上正式版本的标示，在debug版本的登录界面就带上登录的标示

> **Tips**：当你创建了不同的build type，你在不同的type里面作了不同的修改，但是使用的源是基于main目录下的代码，比方说你要在不同的type里面有不同的login界面，那么main包下面就不能包括LoginActivity，而只在不同buildtype下加入各自自定义的LoginActivity，如果你也在main目录下加入，编译器会提示你dumpllicated file的错误

对于Resources资源，和处理Java代码有些许不同，对于layout、Drwable资源和图片，如果在不同的type里面定义了，那个定义的这些文件会直接替换掉main目录里面相同的文件；对于String，Color的资源则会合并，举个例子，如果你在main下的string.xml如下

```
 <resources>
       <string name="app_name">TypesAndFlavors</string>
       <string name="hello_world">Hello world!</string>
</resources>
```

如果你的qa_test type中的string.xml如下

```
 <resources>
       <string name="app_name">TypesAndFlavors QA_Test</string>
</resources>
```

合并后的string.xml如下所示

```
 <resources>
       <string name="app_name">TypesAndFlavors QA_Test</string>
       <string name="hello_world">Hello world!</string>
</resources>
```

如果你没有自定义string那gradle就会默认使用main目录下的string，对于AndroidManifest.xml文件也是一样的，你如果要修改manifest文件不全部拷贝，你只需要加入你需要的代码,gradle会自动合并，在后续部分会更深入的介绍gradle合并的原理

### 依赖

每一个build type有可能有自己独立的依赖，如果你配置了，gradle会自动识别配置，例如你要在在debug type中加入一个日志框架，你可以参考如下代码

```
 dependencies {
       compile fileTree(dir: 'libs', include: ['*.jar'])
       compile 'com.android.support:appcompat-v7:22.2.0'
       debugCompile 'de.mindpipe.android:android-logging-log4j:1.0.3'
}
```

**debugCompile**表示只在debug type中把log4j加入到编译路径中，关于常用的集中依赖scope请参考[系列三:Gradle依赖管理](http://yamlee.me/2016/04/13/Android%E5%BC%80%E5%8F%91Gradle%E7%9A%84%E4%BD%BF%E7%94%A8%E7%B3%BB%E5%88%97%E4%B8%89%E4%B9%8BGradle%E4%BE%9D%E8%B5%96%E7%AE%A1%E7%90%86/)


## Product fllavors

> 与build type相反，build type用来构建不同类型的app，debug版或release版，而produc flavors则是用来在同一个app上创建不同的版本，如付费版和免费版。一个非常常见的应用场景就是你创建了一个银行管理的app给不同银行提供服务，但是不同的银行App的logo不一样，有produc flavors就能在基于一套代码上创建不同版本的app或者library。

> 如果你不确定什么时候用build type，什么时候改用produc flavors，那么问自己几个问题，如果是构建不同类型供内部使用或者是在google palay上提交一个新的app，那么建议使用build type；如果app要分发在不同的渠道上，那么建议product flavors

### 创建 product flavors

创建product flavors与build types非常相似，你只需要添加**productFlavor**代码块，如下代码所示

```
 android {
       productFlavors {
           red {
               applicationId 'com.gradleforandroid.red'
               versionCode 3
￼				}
			blue {
            	applicationId 'com.gradleforandroid.blue'
            	minSdkVersion 14
            	versionCode 4
				} 
		}
}
```

Product flavors与build types属性不同，因为product flavors其实是ProductFlavors类，build.gradle文件中默认添加的***defaultConfig***也是ProductFlavors类的实例。


### 不同product flavors的代码目录结构设置

与build type的设置类似，product flavors也可以配置自己的代码目录

### Multiflavor variants

在某些情况下，你可能需要进行flavors的组合，比方说你的app有两套主题，绿色主题和红色主题，然后有两个版本，付费版和免费版，你可能需要进行组合，类似于红色免费，红色收费，绿色免费，绿色收费。通过使用**flavorDimensions**可以解决flavors组合的问题，配置可参考如下代码

```
android {
       flavorDimensions "color", "price"
       productFlavors {
           red {
               flavorDimension "color"
           }
           blue {
               flavorDimension "color"
			}
			free {
               flavorDimension "price"
           }
            paid {
               flavorDimension "price"
			} 
		}
}
```

当你添加了flavor dimension之后，gradle会判断你是否为flavor指定了flavorDimensions中设定的值，如果没有设置，gradle编译的时候就会报错,而且flavorDimensions中设置的顺序也是非常重要的，比方说如果红色主题的flavor和付费版的flavor中某处代码修改是一样的，那么在运行的时候就以flavorDimensions中配置的先后顺序为准，根据上面的配置，就是以红色主题中的代码为准。

假设你的build type的类型是配置的debug和release那么根据上面代码的配置生成的build variants为如下

*  **blueFreeDebug** 和 **blueFreeRelease**
*  **bluePaidDebug** 和 **bluePaidRelease**
*  **redFreeDebug** 和 **redFreeRelease**
*  **redPaidDebug** 和 **redPaidRelease**


## Build variants

build variants就是build type和product flavors组合的结果，当你创建一个新的build type或者product flavor的时候，新的variants也就相应的创建了。例如，如果你的build type是标准的debug和release，那么你创建一个product flavor为red theme和blue theme的时候，相对应的build variants为自动生成，类似于下图

![](http://ww1.sinaimg.cn/mw690/6b051377gw1f2ylxe43qdj20dk0c6gmm.jpg)


如上截图为Android Studio中Build Varants的工具框，默认在Android Studio界面的左下角边缘倒数第二个，亦或可以通过View | Tool Windows | Build Variants打开。


如果你没有配置product flavors，variats只会包括build types.如过你也没有配置过build type，Gradle的Android插件也会默认配置debug build type，所以build variants不会出现为空的情况


## Tasks


Gradle的Android插件会根据你配置的build variant自动生成task。一个新的Android app会默认有debug和release两种build type，所以你可以运行**assembleDebug**和**assembleRelease**去生成不同apk，又或者直接运行**assemble**来生成。当你新添加build type，就会相应的添加新的task。当你添加新的flavors product。task重新生成变化较大，因为每一个product flavor和build type是组合的。所以即便是最简单的一个build type和一个flavor的组合，你都会生成有三个对应的task：

* **assembleBlue** 使用Blue flavor，相当于运行**assembleBlueRelease**和**assembleBlueDebug**

* **assembleDebug** 


## 代码目录配置（Source sets）

Build variants就是build type和product flavor的组合，比如你源代码中有main，releas，debug，red四个目录，其中，release和debug是build type的类别，red是设置的productflavor，那个对应的build variant就是redReleas,redDebug,也就是说redReleas使用的源码是red和release目录的合并，同理redDebug

## 资源文件和manifest文件的合并

正如上面部分所示，不同的build type和product flavor有不同的源码目录，在生成不同的build variant包时，会需要合并资源，例如比方说你在debug build type中的manifest文件设置了要存储log日志的权限，但是在main目录中的manifest文件却不需要，这样在生成编译debug build variant的时候就需要合并main目录中的manifest文件和debug目录中的manifest文件。其中选取的资源的有限及如下图所示：

![](http://ww4.sinaimg.cn/mw690/6b051377gw1f2ym04dcu1j20uu04it96.jpg)

如上可知，如果flavor中有一个图片资源为logo.png，main目录中也有一个图片资源为logo.png。在打包的时候，因为flavor的优先级大于Main所以，会使用flavor中的logo.png

> 关于资源和manifest文件的合并，有很多具体的细节在这里没法阐述，官方文档给了更多更详细的解释，你可以访问这个地址 [Android Developer:Manifest-Merger](http://tools.android.com/tech-docs/new-build- system/user-guide/manifest-merger)

## 创建build variants

gradle使得处理复杂的build varinats非常容易，及时你创建了两种build type和两种product flavors。build文件中代码还是非常清晰明了

```
android {
       buildTypes {
           debug {
               buildConfigField "String", "API_URL",
               "\"http://test.example.com/api\""
			}
           staging.initWith(android.buildTypes.debug)
           staging {
               buildConfigField "String", "API_URL",
                 "\"http://staging.example.com/api\""
               applicationIdSuffix ".staging"
           }
		}
       productFlavors {
           red {
               applicationId "com.gradleforandroid.red"
               resValue "color", "flavor_color", "#ff0000"
           }
           blue {
               applicationId "com.gradleforandroid.blue"
               resValue "color", "flavor_color", "#0000ff"
￼￼￼￼￼			} 
	}
}
```

在上面的代码中，我们创建了四种build variats：blueDebug,blueStaging,redDebug,redStaging.每一个variant都自定义了API URL 和flavor corlor.运行程序bllueDebug可能为如下所示

![](http://ww4.sinaimg.cn/large/6b051377gw1f2ym6exl4xj20im06eq3c.jpg)

而redStaging可能如下图所示

![](http://ww4.sinaimg.cn/mw690/6b051377gw1f2ym73936uj20ic066q3e.jpg)


第一个截图是blueDebug，使用的是debug build type中的URL和blue product flavor中的颜色，同理可类推redStaging

## Variant filters

在某些情况下，可能你不想使用某种build variant，例如你有debug和release的build type，red和bule的flavors但是，blue还是测试环境根本现在用不到bule的release版本，那么你可以直接过滤掉，那么在android stuido中的buildVariants窗口就不会出现了，过滤可以在app模块或者library模块的build.gradle文件中加入如下代码：

```
android.variantFilter { variant ->
    if (variant.buildType.name.equals('release')) {
        variant.getFlavors().each() { flavor ->
            if (flavor.name.equals('blue')) {
                variant.setIgnore(true);
            }
        }
    }
}          
```

加入以上代码后，再同步项目，在AndroidStudio的BuildVariants窗口就可以看见没有了bule release的版本了

## Signing configuarations(apk包签名配置)

在将app发布到Google Play或者其他应用市场之前，你需要对apk进行秘钥签名。如果你针对用户有免费和付费两个版本，那么你需要给每一个flavor不同的签名，签名的配置代码参考如下

```
 android {
       signingConfigs {
           staging.initWith(signingConfigs.debug)
           release {
               storeFile file("release.keystore")
               storePassword"secretpassword"
               keyAlias "gradleforandroid"
               keyPassword "secretpassword"
} }
}
```

在上面的例子里，我们创建了两种不同签名配置

debug类型的签名配置是由gradle的android插件自动生成的，密码的alias name都是默认的。

> Tips：密码建议不要写在build.gradle文件中，可以写在local.properties文件中

加入了签名配置后，就可以使用签名信息了，代码可参考如下

在buildType中使用

```
 android {
       buildTypes {
           release {
               signingConfig signingConfigs.release
￼￼￼￼￼} }
}
```

在flavors中使用

```
 android {
       buildTypes {
           release {
               productFlavors.red.signingConfig signingConfigs.red
               productFlavors.blue.signingConfig signingConfigs.blue
} }
}
```
由上代码可知，你不能如下配置

```
 android {
       productFlavors {
           blue {
               signingConfig signingConfigs.release
} }
}
```

因为在合并type和flavor是，flavor会覆盖type中的签名

## 总结

在这一部分，主要讨论了build type，produc flavors，和两者的组合以及组合后源码，资源文件的一些细节处理；再就是介绍了签名的配置。