title: 【序列比对】seed-and-extend-based mapper执行过程
date: 2015-08-06 22:01:53
tags: seed-and-extend
categories: 序列比对
---
read mapper大体上分为两类，一类是以seed-and-extend（或者说是hash-table）算法为基础，一类是以suffix-array算法为基础，这里要讲的是第一类算法的执行流程。

<!-- more -->
## 预处理
在进行mapping之前，这类算法要对reference进行预处理。首先会选取固定长度的一些DNA片段作为种子（seeds也可称为k-mers），
该种子的长度一般在10-13bp。根据这些种子创建一个hash table，其中key就是这些seeds的值（比如AATTGGGCCC），
value是对应的seed在reference中出现的所有位置（seed locations）。这个hash table会被保存起来，在后续的mapping过程中使用。
## Mapping
mapping的过程大致可以分为6步
1. 将read分为多个k-mers，每个k-mers的长度和hash table中key值的长度相同  
2. 在上述的k-mers中选取部分k-mers作为key值查询hash table，hash table会返回这些key值在reference中的位置列表  
3. 依次搜索在第2步中获得k-mers 位置 
4. 对于每个位置，提取出该位置附近的参考基因片段  
5. 将read和获得参考基因片段进行比对  
6. 重复4和5直到遍历完所有位置  
  
如下图所示

![seed-and-extend based mapper](/images/seed-and-extend-mapping-flow.png)  

参考文章: [Accelerating read mapping with FastHASH](http://www.biomedcentral.com/1471-2164/14/S1/S13/)