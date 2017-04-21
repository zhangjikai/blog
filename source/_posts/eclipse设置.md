title: 【Eclipse】Eclipse设置
date: 2014-12-13 10:25:22
categories: 开发工具
tags: Eclipse
---
## 自定义自动生成的模版 #
Window->Preferences->Java->Code Style->Code Template   
其中 Comments 是自动生成的注释模版，code是自动生成的代码模版

## 自定义用户名称
一般用于@author选项，如果不定义，那么在windows系统中可能是 Administrator  
方法：在eclipse的主目录，找到eclipse.ini文件，在-vmargs后面添加-Duser.name=名字（要换行）

## 设置空格不上屏
参考[http://blog.csdn.net/zhangjk1993/article/details/24838489](http://blog.csdn.net/zhangjk1993/article/details/24838489)

## 自定义导入格式
默认eclipse导入类一般不会出现通配符，即如import org.eclipse.swt.\*;的形式，我们可以通过配置来实现上面的格式  
Window->Preferences->Java->Code Style->Organized Imports  
找到：Number of imports needed for .\*(xxxx) 和 Number of static imports needed for .\*(xxxx)  
这两个选项所配置的数字就是我们导入同一个包下类的最大数量，如果超过这个值，就会变成 .\*的格式

