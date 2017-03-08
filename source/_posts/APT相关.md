---
title: APT相关
date: 2017-03-07 14:32:53
tags: [Android,Java]
---
APT注解编译工具`Annotation Compile Tool`,用来在编译时根据注解自动生成Java代码.
像`ButterKnife`,`EventBus`等库都为了避免因为反射带来的性能损失,都使用了APT注解方式.
<!-- more -->

## 使用APT需要的组件
通常一个库有三个以下部分组成.
* 核心包 用来对外提供使用,及库的逻辑部分
* Anntation包 用来存放自定义的注解
* Compiler包 这是APT运行所必需的,用来解释注解,告诉APT注解的意思及使用

对与一个使用APT的库来说,Compiler包是必须的.Anntation包有时会包含在核心包里面.

## APT Pulgin
虽然Java提供了APT,但在使用Gradle2.2之前的版本在编译时并不会调用APT,所以通过插件来调用.
这个插件也只能用Javac的方式进行编译.
从Gradle2.2开始,内置了APT的插件,不需要再进行声明,而且还支持以Jack的方式编译.

## APT的使用
### 2.2之前的版本
1. 在工程的Gradle中声明APT插件的依赖
```java
dependencies {
      classpath 'com.android.tools.build:gradle:2.1.0'
      classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
  }
```

2. 在Module的Gradle中声明使用APT插件
```java
apply plugin : `android-apt`
```

3. 在Module的依赖中添加库的依赖.核心包和Compiler包.
```java
dependencies {
  compile fileTree(include: ['*.jar'], dir: 'libs')       
  compile 'com.jakewharton:butterknife:8.5.1'
  apt 'com.jakewharton:butterknife-compiler:8.5.1'
}
```

任何Module,只要需要使用APT,就必须在Gradle中声明使用APT插件及添加Compiler包的依赖

### 2.2之后的版本
1. 工程的Gradle中不需要声明APT插件,除非是库指定.
```java
dependencies {
      classpath 'com.android.tools.build:gradle:2.3.0'
      classpath 'com.jakewharton:butterknife-gradle-plugin:8.5.1'
  }
```

2. 在项目的Gradle中不需要声明使用的插件,除非是库指定的.如果是个库,则依赖这个库的其它Module都不需要再声明
```java
apply plugin: 'com.jakewharton.butterknife'
```

3. 在Module的Gradle中添加核心包及Compiler包的依赖.Compiler包不再使用APT命令,而是`annnotationProcessor`
```java
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.jakewharton:butterknife:8.5.1'
    annotationProcessor 'com.jakewharton:butterknife-compiler:8.5.1'
}
```
任何Module,只需要使用APT,就必须在Gradle中添加Compiler包的依赖.
