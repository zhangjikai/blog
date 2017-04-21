title: 【C】文件操作
date: 2015-12-15 19:52:01
tags: C
categories: C
---
## 打开文件
### fopen
我们可以使用fopen()创建一个新的或者打开一个文件, 文件信息会保存在一个`FILE`类型的指针中, 该函数的原型为:
```C
FILE *fopen( const char * filename, const char * mode );
```
`filename`是文件名, `mode`是打开模式, 可选值如下:
* `r` - 以只读方式打开一个文件, 该文件必须存在
* `w` - 以只写方式打开一个文件, 文件不存在会创建新的文件, 文件存在会首先清空原有内容
* `a` - 以追加的方式写文件, 文件不存在会创建新的文件, 文件存在从文件尾开始写文件
* `r+` - 以读写方式打开文件, 文件不存在不会创建新的文件
* `w+` - 以读写方式打开文件, 文件不存在会创建新的文件, 文件存在会首先清空原有内容
* `a+` - 以追加方式读写文件, 文件不存在会创建新的文件, 文件存在从文件尾开始写文件

如果是操作二进制文件, 那么需要在`mode`里加上`b`, 如下所示:
```html
"rb", "wb", "ab", "rb+", "r+b", "wb+", "w+b", "ab+", "a+b"
```
文件成功打开会返回一个'FILE'类型的指针, 如果打开失败, 会返回一个空指针, 并把错误代码存在`errno`中. 
<!-- more -->
### __r+__ 和 __w+__的区别
看下面的代码
```C
#include <stdio.h>
void  test() {
    FILE *fp;
    fp = fopen("test.txt", "w");
    fprintf(fp, "this is a test...\n");
    fprintf(fp, "this is a test...\n");
    fprintf(fp, "this is a test...\n");
    fclose(fp);
}
void test_w() {
    FILE *fp;
    fp = fopen("test.txt", "w+");
    fprintf(fp, "hello");
    fclose(fp);
}
void test_r() {
    FILE *fp;
    fp = fopen("test.txt", "r+");
    fprintf(fp, "hello");
    fclose(fp);
}
int main() {
    test();
    test_w();
    //test();
    //test_r();
}
```
首先运行`test`, 然后运行`test_w`, test.txt中的结果如下:
```html
hello
```
然后运行`test`和`test_r`, test.txt中的结果如下:
```html
hellois a test...
this is a test...
this is a test...
```
由上面我们可以看到`r+`在写时并不清空已有的内容, 但是会从文件开头开始写, 写入的内容会覆盖已有内容.

###  __r, w, a, b, +__ 的解释
`mode`一般由上面5个字符组成, 有些可能还会使用`t`, 下面是该它们的含义
* `r` - read, 读
* `w` - write, 写
* `a` - append, 追加
* `t` - text, 文本文件, 可省略不写
* `b` - binary, 二进制文件
* `+` - 读和写

### 新的修饰符 __x__
在C2011中, 添加一个新的修饰符`x`, 和`w` 一起使用, 如下 
```html
"wx", "wbx", "w+x" or "w+bx"/"wb+x"
```
当文件存在时, `x`会强制使文件访问出错, 而不是清空文件内容.

## 关闭文件
我们可以使用fclose来关闭文件, 函数原型为:
```C
int fclose( FILE *fp );
```
如果fclose执行成功, 会返回`0`, 如果执行出错则会返回`EOF`(在`stdio.h`中定义). 当fclose关闭文件时, 会首先将输出流(output) buffer 中的内容写入到文件, 将输入流(input) buffer 中的内容丢弃, 然后关闭文件, 释放其对应的内存.

## 写文件
在C中有多种方式可以读写文件, 下面将具体介绍它们
### fputc
将一个字符写入到fp所指向的输出流中(不只是文件输出流), 写入成功会返回写入的字符, 写入失败会返回`EOF`, 函数原型为
```C
int fputc( int c, FILE *fp );
```
下面是一个示例:
```C
void test_fputc() {
    FILE *fp;
    fp = fopen("test.txt", "w+");
    char c;
    int num;
    for(c = 'A'; c <= 'Z'; c++) {
       num =  fputc(c, fp);
       printf("num is %d\n", num);
    }
    fclose(fp);
}
```
test.txt中的内容为
```html
ABCDEFGHIJKLMNOPQRSTUVWXYZ
```
另外`putchar`就是以`fputc`为基础实现的(额, 这个和实现有关, 这里暂且这么认为)
```C
#define putchar(__c) fputc(__c, stdout)
```
`putc`和`fputc`的功能一样, 都是将一个字符写入到对应的输出流中, 并且两者的原型是相同的:
```C
extern int fputc(int __c, FILE *__stream);
extern int putc(int __c, FILE *__stream);
```
但是在实现的时候, `putc`是以宏定义的方式实现的, 而`fputc`则是以函数的方式实现的(f表示function)
```C
#define putc(__c, __stream) fputc(__c, __stream)
```
关于两者的区别可以参考下面两篇文章
[fputc vs putc in C](http://stackoverflow.com/questions/14008907/fputc-vs-putc-in-c)  
[C语言中fgetc、fputc和getc、putc的区别是什么](http://www.cnblogs.com/bwangel23/p/4159414.html) 
大概的意思就是使用`putc`在一般情况下会快些, 但是可能会出现一些问题, 所以建议使用`fputc`.

### fputs
fputs从指定的地址(str)开始复制, 直到遇到标志结束的null字符`\0`, 同时`\0`不会被复制到输出流中. 如果函数执行成功会返回一个非负整数, 否则返回`EOF`, 该函数的原型为:
```C
int fputs ( const char * str, FILE * stream );
```
下面是一个使用示例
```C
void test_fputs() {
    FILE *fp;
    fp = fopen("test.txt", "w+");
    char *c = "this is a test...\n";
    char c1[6] = {'A', 'B', 'C', '\0', 'E', 'F'};
    int num;
    num = fputs(c, fp);
    printf("num is %d\n", num);
    num = fputs(c1, fp);
    printf("num is %d\n", num);
    fclose(fp); 
}
```
执行完`test_fputs`, test.txt中的内容如下所示, 注意c1只写入了`ABC`
```html
this is a test...
ABC
```
和`fputs`相关的一个函数为`puts`, 其函数原型为:
```C
extern int puts(const char *__str);
```
`puts`不可指定输出流, 默认会输出到标准输出流(stdout), 除此之外`puts`在输出完内容之后会在内容后面追加上换行符(newline character). 

### fprintf
`fprintf`用来将格式化数据输出到输出流, 和`printf`用法相同, 下面是函数原型
```C
int fprintf ( FILE * stream, const char * format, ... );
```
下面是一个使用示例
```C
void test_fprintf() {
    FILE *fp;
    fp = fopen("test.txt", "w+");
    int num;
    int i = 100;
    num = fprintf(fp, "this is a test...\n");
    printf("num is %d\n", num);
    num = fprintf(fp, "i is %d and address of i is %p", i, &i);
    printf("num is %d\n", num);
    fclose(fp); 
}
```
执行`test_fprintf`后, test.txt中的内容为:
```html
this is a test...
i is 100 and address of i is 0x7ffd32721f90
```
关于格式化形式可以参考`printf`, 或者下面的链接
[http://www.cplusplus.com/reference/cstdio/fprintf/](http://www.cplusplus.com/reference/cstdio/fprintf/)

## 读文件
### fgetc
`fgetc`一次读取一个字符, 同时将文件指针往后移一个字符, 如果读取成功会返回读取的字符, 出现错误会返回`EOF`. 当读到文件末尾时, 也会返回`EOF`, 并且在输出流中设置文件结束标志(end-of-file indicator). 下面是该函数的原型:
```C
int fgetc ( FILE * stream );
```
下面是一个使用示例, 其中test.txt中的内容为`this is a test...`
```C
void test_fgetc() {
    FILE *fp;
    fp = fopen("test.txt", "r");
    int i, n = 20;
    char c;
    for(i = 0; i < 20; i++) {
        c = fgetc(fp);
        printf("%c",c);
    }
    printf("\n");
    fclose(fp); 
}
```
输出结果为:
```html
this is a test...���
```
对应的ascii码值为:
```html
116 104 105 115 32 105 115 32 97 32 116 101 115 116 46 46 46 -1 -1 -1 
```
当读到文件末尾时返回`EOF`(即-1), 而ascii码中没有-1的对应值, 所以会显示乱码.
和`fgetc`相关的函数有`getchar`和`getc`, 它们的关系和`fputc`与`putchar`, `putc`的关系一样, 下面是`getchar`和`getc`的实现
```C
 #define getchar() fgetc(stdin)
 #define getc(__stream) fgetc(__stream)
```
### fgets
该函数的原型为:
```C
char * fgets ( char * str, int num, FILE * stream );
```
`fgets`从`stream`中读取内容到`str`, 当满足下面任意一个条件时完成读取操作:
* 读取了__num-1__个字符
* 读到了换行符(newline character)
* 读到了文件结尾(end-of-file)


注意第二条, 换行符也会被读到str中. 读取完成后会在str后面追加上 *终止null字符* (即`\0`), 这也是第一条为什么只读 __num-1__ 个字符的原因. 函数返回值是一个指向str的指针.  
下面是一个使用示例, 
```C
void test_fgets() {
    FILE *fp;
    fp = fopen("test.txt", "r");
    char c[50];
    fgets(c, 5, fp);
    printf("c is '%s'\n", c);
    printf("c length is %ld\n", strlen(c));
    // 重置文件指针到文件开头
    rewind(fp);
    fgets(c, 20, fp); 
    printf("c is '%s'\n", c);
    printf("c length is %ld\n", strlen(c));
    fclose(fp); 
}
```
其中test.txt中内容为:
```html
this is a test...
this is a test...
this is a test...
```
执行结果为:
```html
c is 'this'
c length is 4
c is 'this is a test...
'
c length is 18
```
`gets`从标准输入流(stdin)里读取数据, 以换行符(回车)作为读取完成的标志, 下面是该函数的原型
```C
char * gets ( char * str );
```
下面是一个使用示例:
```C
void test_gets() {
    char string [256];
    printf ("Insert your full address: ");
    gets (string);     // warning: unsafe (see fgets instead)
    printf ("Your address is: %s\n",string);
}
```
在编译时会有警告
```html
file.c:102:5: warning: ‘gets’ is deprecated (declared at /usr/include/stdio.h:638) [-Wdeprecated-declarations]
    gets (string);     // warning: unsafe (see fgets instead)
```
即`gets`函数已过时`deprecated`, 不建议使用, 而应使用`fgets`代替

### fscanf
fscanf以格式化的形式读入数据, 函数原型如下:
```C
int fscanf ( FILE * stream, const char * format, ... );
```
`fscanf`以空格和换行符作为读入的结束字符, 同时在`fscanf`读入时会忽略第一个非空字符前面的空白符(空格,换行,tab), 下面是一个测试示例
```C
void test_scanf() {
    FILE *fp;
    fp = fopen("test.txt", "r");
    int num[3], i;
    for(i = 0; i < 3; i++) {
        num[i] = 0;
    }
    fscanf(fp, "%d%d", num, num+1);
    for(i = 0; i < 3; i++) {
        printf("num[%d] is %d\n", i, num[i]);
    }
    fclose(fp); 
}
```
其中test.txt中的内容为:
```html

      10     55dddd   
```
程序执行结果为:
```html
num[0] is 10
num[1] is 55
num[2] is 0
```
在test.txt中 `10` 前面有个空行, `55` 前面有多个空格, 并且后面有 `dddd`等非数字字符, `fscanf`在读入时会忽略掉前面的空白符, 并且执行到不匹配的地方就不再往后读入.
<br />

__参考文章__
***
[http://www.tutorialspoint.com/cprogramming/c_file_io.htm](http://www.tutorialspoint.com/cprogramming/c_file_io.htm)
[http://www.cplusplus.com/reference/cstdio/](http://www.cplusplus.com/reference/cstdio/)
[http://www.2cto.com/kf/201207/143344.html](http://www.2cto.com/kf/201207/143344.html)
