title: 【ASP】ASP基础
date: 2015-07-29 20:17:29
tags: ASP
categories: ASP
---
## 前言
asp是个奇葩的语言，语句结尾不加“;”号，我也是醉了

## 声明
在文件开头加上  
`<%@ Language=VBScript Codepage="65001"%>`  
Codepage=65001表示文件采用UTF-8编码

## 包含
`<!-- #INCLUDE FILE="DBUtil.asp" -->`
## 注释
使用英文单引号（'）注释  
`'这是一条注释`  
不支持/**/格式的多行注释

<!-- more -->
## 变量
使用Dim声明变量，其实也可以不声明，直接使用变量  
`Dim a, b`  
Dim 仅作声明使用，不可在声明时赋值，如下面的写法是错误的 
`Dim a = 1 `
Set 为一个变量指定对象，也可以理解为为对象变量赋值，操作数据库时会经常使用  
`Set con = server.createobject("adodb.connection") `  
在为数字、字符串、数组赋值时不需要Set

## 运算符
加 +   
减 -  
乘 *  
除 /(结果有小数)  
整除 \
取余 mod  
字符串连接 &


## 数组
### 定义定长数组
初始化定义  
`arr = Array(1,2,3)`  
不初始化  
```asp
Dim arr(3)
arr(0) = 1
arr(1) = 2
```
调用方式arr(1)，不使用[]，数组长度必须为数字，不能是变量，如arr(n)是错误的
### 定义动态长度
使用redim 可以改变数组长度

```asp
Dim arr()
redim arr(5)
redim preserve arr(10)
```
arr在声明完，必须要redim一次才可以使用，加上preserve关键字保留数组中原有的值，否则会清空数组
### 数组长度
使用ubound(arr)函数获得arr数组的最大索引，即length(arr)=ubound(arr)+1，没有length属性或者length方法

## 分支语句
### if
if语句
    
```asp
if a = 1 then
    do something
End if
```
if else语句

```asp
if a = 1 then
    do something
else
    do something
End if
```
    
if elseif else语句
```asp
if a = 1 then
    do something
elseif a = 2 then
     do something
else 
    do something
End if
```
没错，判读等于使用=，不等于使用<>

### switch(select case)
其实没有switch语句，使用select case一样效果
```asp
select case i
    case 1
        do something
    case 2
        do something
    case else
        do the rest
End select
```
## 循环语句
### while
```asp
    i = 0
    while i < 7
        do something
        i = i + 1
    wend
```
### do while
```asp
    i = 0
    do while i < 7
        do something
        i = i + 1
    loop
```
### for
```asp
    for i = 0 to 7
        do something
    next
```
## 字符串
### 长度
`Len(str)`
### 判断b是否为a的子串
`Instr(a, b)`
### 截取字符串
`left(str, length) 'str[0-(length-1)]`  
`mid(str, start, length) 'str[start-(start+length-1)]`  
`right(str,length) 'str[end-length, end]`  
### 分割字符串  
`Split("aa;aa", ";" )`  
### 去掉前后空格  
`Trim(str)`  
### 转为数字  
`cint(str) '转为整数`  
`CDbl(str) '转为小数`  
### 处理字符串中的"  
使用""即可，如下面就表示字符串为：哈哈"  
`a = "哈哈"""`
### 正则表达式
```asp
    Set regEx = New RegExp
    regEx.Pattern = pattern
    match = regEx.test(str)
```
## 定义函数
### 不带返回值
```asp
    function fun(p1,p2)
        ...
    End function
```
如果参数大于1个，调用时应加上Call，如Call fun(1, 1)，或者fun 1, 1这样调用
### 带返回值
```asp
    function fun(p1, p2)
        ...
        fun = returnValue
    End function
```
该函数正常调用即可

## 操作Access数据库

```asp
Dim oConn
Dim rs
Dim sql
'If not already defined, create object

If Not IsObject(oConn) Then
    Dim strConnQuery
    Set oConn = Server.CreateObject("ADODB.Connection")
    oConn.Mode = 3
    'Create the path to database
    strConnQuery = "DBQ=" & Server.mappath("data/tpd.mdb")
    'Connect
    oConn.Open("DRIVER={Microsoft Access Driver (*.mdb)}; " & strConnQuery)
End If

sql = "select * from table"
Set rs = oConn.execute(sql)

Do While Not rs.EOF
    rs('字段名称')
    rs.Movenext
Loop

rs.Close()
Set rs = Nothing
sql = "insert into ..."
oConn.execute(sql)
oConn.Close()
Set oConn = Nothing
```
### access分页
```asp
    if page = 1 then
        sql = "select top " & pageSize & " * from person as p  where " & con & " order by personId"
    else
        sql = "select top " & pageSize & " * from person as p where " & con & " and personId > (select MAX(personId) from (select top " & (pageSize * (page - 1)) & " personId from person as p where " & con & " order by personId) ) order by personId"
    End if
```
## 操作Excel
```asp
     Dim excelConn, excelStrConn, excelRs, excelSql
     Set excelConn = Server.CreateObject("ADODB.Connection")
     excelStrConn ="Provider=Microsoft.Jet.OLEDB.4.0;Data Source=" & Server.MapPath("upload/uploadFiles/" & Request.QueryString("filename")) & ";Extended Properties=""Excel 8.0;HDR=Yes;IMEX=1"""
     excelConn.Open excelStrConn
     Set excelRs = Server.CreateObject("ADODB.Recordset")
    
     excelSql = "select * from [Sheet1$]"
     excelRs.Open excelSql, excelConn, 1, 1
     
     do while Not excelRs.EOF 
        'excelRs.Fields.Count：列数
        for i = 0 to excelRs.Fields.Count-1
            '列名
            excelRs(i).name
            '列值
            excelRs(i)
        next
       
        excelRs.MoveNext 
     Loop
    
     excelRs.close 
     set excelRs=nothing 
     excelConn.close 
     set excelStrConn=nothing 
```
## 接受页面参数值
get方式提交的参数  
`Request.queryString(参数名)`  
post方式提交的参数  
`Request.form(参数名)`

## Session
`Session("参数名") = 值` 








