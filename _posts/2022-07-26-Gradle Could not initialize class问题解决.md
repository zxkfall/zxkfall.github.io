---
layout: post
title: "Gradle Could not initialize class问题解决"
subtitle: Gradle Could not initialize class problem solve
categories: gradle
tags: [ gradle, mac, java]
banner: "/assets/images/cover/2022-07-26-wallhaven-6k3oox.jpg"

---

使用项目路径下的gradlew命令构建项目时，出现报错提示Could not initialize class org.codehaus.groovy.runtime.InvokerHelper

<!--more-->

## 1. 问题描述

使用项目路径下的gradlew命令构建项目时，出现报错提示

Could not initialize class org.codehaus.groovy.runtime.InvokerHelper

## 2. 解决方法

> For Mac

1. 找到项目路径下的`gradle/wrapper/gradle-wrapper.properties`路径，查看构建项目使用的gradle版本。

   `distributionUrl=https\://services.gradle.org/distributions/gradle-5.3-bin.zip`里面的5.3即gradle的版本。

2. 下载与Gradle版本对应的JDK版本，对应Gradle需要的JDK版本可以从官网查看

   https://docs.gradle.org/current/userguide/compatibility.html

3. 以openJDK为例，在openJDK官网，下载满足要求的JDK版本(Mac版本)

   https://jdk.java.net/archive/

4. 解压后，将其移动到`/Library/Java/JavaVirtualMachines/`路径下(以17.0.2为例)

   ```bash
   $ sudo mv ~/Downloads/jdk-17.0.2.jdk /Library/Java/JavaVirtualMachines/
   ```

5. 编辑.zshrc文件，配置JAVA_HOME

   ```bash
   $ sudo vim ~/.zshhrc
   ```

   添加

   ```bash
   export JAVA_HOME=$(/usr/libexec/java_home -v 17.0.2)
   export PATH=$PATH:${JAVA_HOME}/bin
   ```

6. 运行`echo $JAVA_HOME`，此时`JAVA_HOME`的路径以及java版本就和我们预期的一致了，这样gradlew就可以找到对应的JDK版本

## 3. 解析

1. 为什么要复制到`/Library/Java/JavaVirtualMachines/`路径下，而不是自己指定一个位置，直接设置`JAVA_HOME`路径？

   在Mac OS X 10.5之后， `/usr/libexec/` 路径下多了一个叫java_home的文件，这是Mac上专门用来管理`JAVA_HOME`的文件，我们可以靠它得到不同版本的`JAVA_HOME`，而`/usr/libexec/java_home`会自动映射到`/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home`这个路径下，其中`jdk-17.0.2.jdk`为JDK的版本。

   ```bash
   $ /usr/libexec/java_home -V
   Matching Java Virtual Machines (2):
       17.0.2 (x86_64) "Oracle Corporation" - "OpenJDK 17.0.2" /Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
       1.8.0_201 (x86_64) "Oracle Corporation" - "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
   ```

   运行`/usr/libexec/java_home -V`命令，会得到所有的JDK版本，以

   ```
   17.0.2 (x86_64) "Oracle Corporation" - "OpenJDK 17.0.2" /Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
   ```

   为例，其中`17.0.2`为版本，`Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home`为JDK所在的位置。

   在`.zshrc`文件中配置的`/usr/libexec/java_home -v 17.0.2`实际上就是`Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home`，如果指定版本，即去掉`-v 17.0.2`，那么默认配置的`JAVA_HOME`位置就是运行`/usr/libexec/java_home -V`命令后，最后一行显示的那个版本

   所以如果想要安装多个版本的JDK，就可以通过这种方法来很方便的切换JDK版本。

   同时，如果没有在`.zshrc`中设置`JAVA_HOME`变量，那么默认的`JAVA_HOME`实际上就是` /usr/libexec/java_home`里面指定的版本。

2. `/Library/Java/JavaVirtualMachines/`并不是`/usr/libexec/java_home`的唯一映射路径，它实际上还会映射到`/Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin `，以及用户目录下的几个路径，从这几个路径中获取JDK

## 4. 使用asdf快速切换JDK

安装asdf

```bash
$ brew install asdf
```

安装java插件

```bash
$ asdf plugin add java
```

修改`.zshrc`文件，添加

```bash
 . /usr/local/opt/asdf/libexec/asdf.sh
 . ~/.asdf/plugins/java/set-java-home.zsh
```

使配置文件 立即生效

```bash
$ source ~/.zshrc
```

安装JDK

```bash
$ asdf install java adoptopenjdk-17.0.3+7
```

设置Java版本为17.0.3

```bash
$ asdf global java adoptopenjdk-17.0.3+7
```

之后便可以使用java了。

## 参考资料

1. [Gradle 与 JDK版本对应关系](https://docs.gradle.org/current/userguide/compatibility.html)

2. [Gradle 与JDK版本不对应导致报错](https://stackoverflow.com/questions/35000729/android-studio-could-not-initialize-class-org-codehaus-groovy-runtime-invokerhel)

3. Mac 扫描到的JDK位置

   [Article1](https://blog.csdn.net/supercooly/article/details/52252126)

   [Article2](https://juejin.cn/post/6871959224314757134)

   [Article3](https://blog.csdn.net/caoxiaohong1005/article/details/73611424)

4. asdf与java插件配置

   [asdf](https://asdf-vm.com/)

   [asdf-java](https://github.com/halcyon/asdf-java)

5. [homebrew安装](https://brew.sh/index_zh-cn)

