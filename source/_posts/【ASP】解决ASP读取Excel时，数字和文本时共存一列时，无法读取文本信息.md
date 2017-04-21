title: 【ASP】解决ASP读取Excel时，数字和文本时共存一列时，无法读取文本信息
date: 2015-07-29 21:50:50
tags: ASP
categories: ASP
---
ASP读取Excel时，默认情况下会根据excel的前8行的值，来判断某列的值类型，如果判断值类型为数字，
那么如果该列剩余行有文字信息，则无法读取或者直接报错，下面是解决方法：  
### 1. 修改连接字符串

		excelStrConn ="Provider=Microsoft.Jet.OLEDB.4.0;Data Source=" & Server.MapPath("upload/uploadFiles/" & 
		Request.QueryString("filename")) & ";Extended Properties=""Excel 8.0;HDR=Yes;IMEX=1""" 
<!-- more -->
### 2. 配置注册表
开始->运行->regedit.exe  
找到下面注册项：  
32位：Hkey_Local_Machine/Software/Microsoft/Jet/4.0/Engines/Excel/  
64位：Hkey_Local_machine/software/wow6432Node/microsoft/jet/4.0/Engines/Excel  
修改下面两项的值  
`TypeGuessRows = 0（根据前16384判断）`  
`ImportMixedTypes = Text`  