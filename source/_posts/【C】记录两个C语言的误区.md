title: 【C】记录两个C语言的误区
date: 2015-11-15 20:42:55
tags: C
categories: C
---
## 前言
之前在windows上使用vc++6.0，编写过c的代码，主要是为了完成一些作业，并没有十分深入的学习C语言. 因此当时留下了两个对于c语的言的误区，现在记录一下。
### 关于函数的调用
一直以我都认为在调用一个方法之前，必须要在前面声明原型或者直接定义该方法， 大概如下面的形式， 否则程序就会出现编译错误。

```C
int test();

int main() {
    test();
    return 0;
}
int test() {
    //some code
    return 0;
}
```
<!-- more -->
直到我偶然使用gcc编译了一下下面的代码:

```C
int main() {
    aa();
    return 0;
}

int aa() {
    printf("this is aa\n");
    return 0;
}
```
编译器竟然没有报任何错误和警告， 并且程序可以正常运行。 瞬间有三观被刷新的感觉. 一开始我以为是使用的编译器的标准不同，因此尝试着使用c89，c90，c99，c11编译程序，使用c89和c90时， 编译器还是没有报任何错误，而使用c99和c11时，会报下面的警告:
```html
test.c: In function ‘main’:
test.c:5:2: warning: implicit declaration of function ‘aa’ [-Wimplicit-function-declaration]
  aa();
  ^
```
然而仅仅是警告，程序还是可以正常执行。 随后我又看了一下gcc的版本，发现是4.8.4， 然后查看了一下它的手册， 发现其默认使用的c编译标准是c90
```html
 The default， if no C language dialect options are given， is -std=gnu90;
```
不过有意思的如果将代码写成下面的形式:
```C
int main() {
    aa();
}

void aa(int n) {
    printf("this is aa\n");
}
```
那么编译时就会报下面的警告:
```html
test.c:8:7: warning: conflicting types for ‘aa’ [enabled by default]
 void  aa() {
       ^
test.c:4:2: note: previous implicit declaration of ‘aa’ was here
  aa();
  ^
```
如果将aa的void改为double，就会直接报错了:
```html
test.c:8:9: error: conflicting types for ‘aa’
 double  aa() {
         ^
test.c:4:2: note: previous implicit declaration of ‘aa’ was here
  aa();
  ^
```
查了一下， 大该就是如果不事先定义函数原型并且在函数定义前调用该函数， 那么编译器就会认为该函数 return int 类型， 并且接受的参数个数不确定， 因此当在下面的函数定义时不返回int类型， 就会重现冲突的警告或者错误。
总结一下就是在函数未被定义之前(并且没有声明函数原型)， 我们并不是绝对的不能调用它， 但是这种方式是十分不优雅的， 并且可能出现各种问题.。 所以还是采取函数原型的方式比较好。
### 静态数组
另一个误区就是静态数组的定义， 如下面的形式在vc++6.0中编译时会出现错误
```C
int n = 5;
int arr[n];
```
因此我一直以为在c中定义静态数组必须要制定一个确定的值，而不能是变量。 当然当我无意中使用gcc编译一下上面的代码，发现是可以编译通过的， 并且没有任务的警告和错误， 于是感觉三观又被刷新了。。。
