title: 【软件】Intellij IDEA 导入maven web项目并部署到Tomcat
date: 2015-12-26 16:55:10
tags: ["IDEA", "WEB"]
categories: "软件"
---
## 前言
本文中所使用的IDEA版本为14.1.1, 所使用测试项目为sping mvc的一个最简单的示例, 可以在[这里](http://download.csdn.net/detail/zhangjk1993/9378641)下载示例代码
<!-- more -->

## 导入项目
* __File__ -> __New__ -> __Project from Existing Sources...__
![](/images/idea_maven_web/001.png)

* __选择maven项目所在的文件夹__
![](/images/idea_maven_web/002.png)

* __Import project from external model__ -> __Maven__
![](/images/idea_maven_web/003.png)  

* 使用默认设置, 然后一直__Next__, 直到项目创建成功.

## 配置项目
### 添加Spring支持
我们打开`applicationContext.xml` 会提示 `Create Spring facet`, 我们点击它, 增加对Spring 的支持
![](/images/idea_maven_web/004.png)

点击右侧的`+` 号选择Spring 的配置文件
![](/images/idea_maven_web/005.png)

### 添加Web支持
* __File__ -> __Project Structure...__ -> __Modules__ -> __选中项目(不是Spring)__ , 然后点击上方的`+`号
![](/images/idea_maven_web/006.png)

* 选择  __`Web`__
![](/images/idea_maven_web/007.png)

* 然后我们会看到在下方会提示`'Web' Facet resources are not included in an artifact`, 我们点击`Create Artifact`新建一个
![](/images/idea_maven_web/008.png)

* 然后我们会跳转到`Artifacts`选项中, 注意右侧`Available Elements` , 这些是Spring的依赖包, 我们在这些依赖包上双击, 就可以添加到`WEB-INF`的`lib`文件夹中, 这样部署到Tomcat上程序才可以正常运行.
![](/images/idea_maven_web/009.png)


* 下图是点击之后的效果, 然后点击OK即可
![](/images/idea_maven_web/010.png)

### 配置Source文件夹
在本项目中, src是默认的source文件夹, 造成的结果就是包名要以 `main.java`开头(不知道是没配置好, 还是默认这样), 为了解决这个问题, 我们可以手动更改一下默认的source文件夹
![](/images/idea_maven_web/011.png) 

* __File__ ->  __Project Structure...__ -> __Modules__ -> 点击项目名称,  在`Sources`选项卡中我们可以看到项目的结构, 在右侧有当前的Sources Folders是哪些文件夹, 我们把点击src右侧的`X`号, 将其从Sources Folders中删除, 然后在左侧面板中, 展开src, 右击java文件夹, 选择 Sources 选项
![](/images/idea_maven_web/012.png)

* 修改完成后如下图所示, 然后点击 OK 即可
![](/images/idea_maven_web/013.png)

### 配置Tomcat
* __Run__ -> __Edit Configurations...__
![](/images/idea_maven_web/014.png)

* 点击`+`号, 添加Tomcat
![](/images/idea_maven_web/015.png)

* 如果之前没有配置过Tomcat, 那么需要配置一下Tomcat的路径, 点击 Application server 右边的 `Configure... `
![](/images/idea_maven_web/016.png)

* 切换到 `Deployment` 选项卡, 点击 `+` 号, 选择 `Artifact...`
![](/images/idea_maven_web/017.png)

* 选择完后, 应该为下图的样子, 在 Application context 里可以更改项目的url路径
![](/images/idea_maven_web/018.png)

## 运行程序
到了这里我们终于配置完了, 下面就可以运行程序了, 在面板下方有个`Application Server` 选项卡, 如果没有默认打开, 我们点击它即可, 然后点击 run 按钮, 就可以运行我们的程序了.
![](/images/idea_maven_web/019.png)

下图是运行结果
![](/images/idea_maven_web/020.png)
