title: 【并行】Ubuntu下配置Eclipse并行程序开发环境
date: "2015-03-14 21:21:01"
tags: 并行程序
---
## 下载用于开发并行程序的Eclipse版本
找到[Eclipse下载页面](https://eclipse.org/downloads/)，找到Eclipse for Parallel Application Developers，点击下载

## 添加MPI库支持
window->preferences->Parallel Tools->Parallel Language->MPI->点击New->输入 /usr/include/mpich 
<!-- more -->

## 添加pthread库支持
右击项目，选择 Properties->C/C++ Build->Settings->GCC C++ Linker->Libraries  
在Libraries(-l)中添加pthread  
在Libraries search path(-L)中添加crypto  

## 开发openmp程序
直接新建openmp工程即可