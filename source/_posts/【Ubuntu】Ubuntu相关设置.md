title: 【Ubuntu】Ubuntu相关设置
date: 2015-03-14 21:29:05
tags: Ubuntu
---
## 添加PPA
`sudo add-apt-repository ppa:user/ppa-name`  
`sudo apt-get update`  
`sudo apt-get upgrade`

## 安装apt-file （查找文件所属的包）
`sudo apt-get install apt-file`  
`apt-file update`  
`apt-file search libwpg`  
使用方法：`apt-file search xxx`
  
## 64位系统无法运行wps
在终端运行wps，即直接输入wps，会提示缺少lib，使用apt-file查找该lib对应的包，下载对应包  
注意要在包名后面添加":i386", 比如要安装libfontconfig1,那么我们所输入的命令就是 
`sudo apt-get install libfontconfig1:i386`  
具体参见：[关于WPS libgthread-2.0.so.0 启动报错的解决方法](http://www.cnblogs.com/jamesarch/p/3362794.html)
<!-- more -->
## wps提示缺少字体
下载对应字体，复制到/usr/share/fonts/wps-office下面

## 安装rabbitCvs(git客户端)
`sudo add-apt-repository ppa:rabbitvcs/ppa`  
`sudo apt-get update`  
`sudo apt-get install rabbitvcs-nautilus3`  
`nautilus -q` 

## 修改文件默认打开方式
右键->属性->打开方式

## 快捷键
锁屏：Ctrl+Alt+L
显示隐藏文件：Ctrl+H
查看系统信息：more /etc/issue



