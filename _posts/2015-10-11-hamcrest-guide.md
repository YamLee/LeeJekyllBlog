---
layout:     post
title:      "单元测试断言框架Hamcrest使用指南"
subtitle:   "Hamcrest中断言方法的使用与自定义匹配模式"
date:       2015-10-11
author:     "YamLee"
header-img: "img/post-bg-universe.jpg"
tags:
    - Android,Junit,Hamcrest
    - assertThat
    - 翻译
---

## 单元测试中断言框架的使用-hamcrest 


> Robolectric基于Junit,Junit本身支持段元框架，但是因为语义不清，测试意图表达不明显而广为人诟病，所以在使用Robolectric做单元测试的时候，使用hamcrest作为断言框架使用语义清晰，现在介绍下hamcrest的使用

### 依赖

 Grdle依赖: `testCompile 'org.hamcrest:hamcrest-all:1.3'`


### 静态导入
```java
import static org.hamcrest.Matchers.*;
import static org.hamcrest.MatcherAssert.*;
```

> Matchers类中包含各类验证方法，如is(),startWith()...一类的验证方法，MatcherAssert只包含了一个assertThat方法作为断言关键字,快速开始中的一个代码片段`assertThat(true, is(false));`便是使用的这个assertThat方法
	
### Matchers详解

#### Core

* **anything**	 : 没有条件通过测试
* **describedAs** : Matcher的封装如Mathcer中的equaTo()方法,可以如下使用
	```java
	assertThat(result, describedAs("A integer should equals %0 ", equalTo(2), 2));```
	
	`java.lang.AssertionError: 
	 Expected: A integer should equals <2> 
     but: was <3>`	
	如上为测试代码与结果，从结果中可以看到Expected加入了describeAs中的解释
	
* **is** :此matcher方法主要用来来提升代码可读性
 	
#### Logical

* **allof**: 如Java中的&&操作,通常用来操作集合型的数据
* **anyof**: 如Java中\|\|操作
* **not**： 如Java中!操作

#### Object

* **equalTo**: 通过调用对象的equals方法来判断对象是否相等
* **hasToString**: 通过调用对象的toString方法，判断toString是否返回的指定字符
* **instanceOf**: 判断实例类型
* **notNullValue**,**nullValue**:是否空类型
* **sameInstance**

#### Beans
* **hasProperty**: 测试JavaBean中是否含有指定属性

#### Collections

* **array**
	 	
#### Number
* **closeTo**: 测试浮点型数接近expected value
* **greaterThan,greterThanOrEualTo,lessThan,lessThanOrEqualTo**: 比较大小或相等

#### Text
* **eualToIgnoringCase**:测试字符串相等，忽略大小写
* **equalToIgnoringWhiteSpace**: 测试字符串相等，忽略空格
* **containsString,endsWith,startsWith**

### 自定义Matchers 
> hamcrest本身自带的Matchers类中就包含了许多好用的匹配方法，但是如果我们想要自己定义新的匹配模式，我们也可以自行定义

```java
public class NotTextEmpty extends TypeSafeMatcher<String> {
    @Override
    protected boolean matchesSafely(String item) {
        if(null == item || "".equals(item)||item.trim().equals(item)){
            return false;
        }
        return true;
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("a text is not null and content is not a white space");

    }

    @Factory
    public static <T> Matcher<String> notTextEmpty(){
        return new NotTextEmpty();
    }
}
```

如上所示，如果我们想要自己定义一个匹配方法，用来验证一个String字符是否为Null获取为空内容，我们便可以如上代码行中定义，我们在使用时便可以直接调用NotTextEmpty.notTextEmpty()方法如下:
`assertThat(null,notTextEmpty());`
	
> 运行测试可以看到

```
java.lang.AssertionError: 
Expected: a text is not null and content is not a white space
     but: was null
```
