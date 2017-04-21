title: 【C】Storage Class
date: 2015-10-26 11:47:21
tags: C
categories: C
---
## 什么是Storage Class
Storage Class翻译成中文为*存储类*（总感觉翻译成汉语不太好），用来修饰C中变量和函数。如果没有显式的指定storage class，会使用默认值。它的作用主要以下几点：
- 决定变量存储的位置。每个变量都需要一定的空间来存储，经常用到的存储单元就是内存，除了内存之外，CPU中的寄存器也可以存储变量，而且一般来说寄存器的访问速度要远远大于内存的访问速度。
- 决定变量的生命周期。
- 决定变量的可见级别。
- 决定变量是否初始化。  
<!-- more -->

## Storage Class 说明符（Specifiers）
下面列出了5中Storage Class的说明符，但是只有前四种是真正意义上的说明符，typedef只是为了语义上的方便，才将其称为一个storage class的说明符。
- auto
- register
- static
- extern
- typedef  

需要注意的是我们在一个声明中，我们至多使用一个storage class的说明符。如果没有显示使用说明符，则会使用以下的默认规则：
1. 在__函数内部__声明的变量默认使用 auto 说明符
2. 在__函数内部__声明的函数默认使用 extern 说明符
3. 在__函数外部__声明的变量和函数默认使用static说明符，并且具有外部链接（external linkage）

具有外部链接的变量和函数可以作用于程序中的所有文件，单纯使用static的变量和函数具有文件作用域（File Scope），它们只有内部链接（internal linkage），只能作用于当前文件。局部变量没有链接，它们只作用于定义的代码块。

## Storage Class 类别（Type）
根据上面所说，在C中一共有四类storage class：
1. Automatic Storage Class
2. Register Storage Class
3. Static Storage Class
4. External Storage Class

下面是详细介绍

### Auto Storage Class
在代码块或者函数中，使用auto声明的变量属于automatic storage class，如果没有显示调用storage class说明符，那么auto就是其默认值。auto storage class的变量属于局部变量，只在其定义的代码块或者函数中起作用，当离开代码块或者函数执行完毕之后就会被销毁（destroyed）。下面是一个示例：
```C
#include <stdio.h>

int main( )
{
    auto int i = 1;
    {
        auto int i = 2;
        {
            auto int i = 3;
            printf ( "\n%d ", i);
        }
        printf ( "%d ", i);
    }
    printf( "%d\n", i);
}
```
输出结果为：3 2 1  
在上面的代码中我们定义3个相同名称的变量i，并且成功执行。这是因为这三个变量的作用域不同，并且8-11行代码块中的i覆盖了6-13行中对于i的定义（类似于局部变量覆盖全局变量），即程序执行时会先在当前代码块的作用域中查找相应变量，如果找不到再去其属于的更大范围的代码块查找，直至找到或者报错。需要注意的地方是__automatic storage class变量并不会被初始化__，在使用之前要手动为其赋初值，否则程序可能会出现意想不到的结果。

### Register Storage Class
使用register声明的变量属于Register Storage Class。Register Storage Class类型的变量可以看作是一种特殊形式的automatic变量，Automatic变量是在内存中分配存储空间的，但是对于大多数的电脑来说，数据的访问速度要小于CPU的计算速度，因此CPU会有一定的空间来缓存少量的数据，以加快访问数据的速度，CPU的这些存储单元就叫做寄存器（Register）。
&emsp;&emsp;一般来说编译器会决定何时将数据存储到CPU的寄存器中。不过C同时提供了一种方式__建议__编译器将变量放到寄存器中，这种方式就是register storage class，之所以说__建议__是因为编译器并不一定会将register变量放入到寄存器中，这个和具体的实现以及寄存器的空间大小有关系，但是大多数情况中只要显示调用了register说明符，编译器就会在寄存器上为其分配空间。同时，并不是所有类型的变量都可以放到编译器中，这个也和具体的编译器实现有关。
&emsp;&emsp;同时需要注意的是，register变量不能使用取地址符'&'，因为按照标准它是存储在寄存器中的，并没有内存的地址，所以下面的代码是编译不过的
```C
#include <stdio.h>

int main()
{
    register int i = 1;
    int *p =  &i; //error: address of register variable requested
    printf("Value of i: %d\n", *p);
}
```
register变量同样没有初始值。另外需要说明的是并不是使用了register变量就一定会比使用automatic变量快，比如你定义了很多的register的变量，导致寄存器的空间不够使用，那么为了使其他register变量可以正常使用，就不得不将一些寄存器的值交换到内存中，而这个交换过程往往会浪费大量的时间。同时，自定义register变量的使用可能会影响编译器对于寄存器使用的默认行为，例如存储表达式求值的临时值。所以应该慎重使用register变量。

### Static Storage Class
static用来声明static storage class的变量。static可以修饰局部变量或者全局变量，当修饰局部变量时，static并不改变该变量的作用域，但是会一直保存该局部变量的值，直至程序结束。当修饰全局变量或者函数时，static会限定该变量的作用域为当前文件（具有内部链接），其他文件并不能使用该变量或者函数。下面是一个示例：
```C
#include <stdio.h>
static int gInt = 1;
static void staticDemo()
{
    static int i;
    printf("%d ", i);
    i++;
    printf("%d\n", gInt);
    gInt++;
}

int main()
{
    staticDemo();
    staticDemo();
}
```
输出结果为：
0 1  
1 2  
当第一次调用staticDemo时，会首先初始化static变量，如果手动设置了初始值，那么就将其值设为指定的值，否则初始为0，程序打印出0和1，但是当执行完staticDemo时，变量i并没有别销毁。当第二次调用staticDemo时，即便执行到第5行代码，编译器也不会重新初始化i了，而是使用已经创建的i变量。因此可以总结两点：（1）static变量__初始值为0__（2）在其生命周期中只会被__初始化一次__。

### External Storage Class
extern用来声明external storage class的变量，它的作用是：__告诉编译器这个变量或者函数已经在其他地方定义过了，这里无需再定义了，直接使用即可__。下面简单说明一下声明（declaration）和定义（definition）的区别：__声明__是告诉编译器变量或者函数的一些基本信息，而__定义__是为这些变量或者函数分配存储空间（详细的的可以看上一篇文章），同一个名称的变量或者函数可以__声明__很多次，但是__只能__定义一次。 
&emsp;&emsp;对于全局变量或者函数来说，如果不加static说明符，那么变量或者函数就可以被程序中的所有文件使用，而extern就是实现这一功能的关键。如果想在文件A中使用文件B中的变量i，那么就需要在A中使用extern声明i，下面是一个例子：
```C
A.c
#include <stdio.h>
extern int i;
int main() {
    printf("the i is %d\n", i);
}

B.c
int i = 5;
```
注意使用extern只是声明一个变量，并没有为其分配存储空间，所以`extern int i = 5`的写法是错误的，如果不加extern，那么就是对于i的定义。external变量也是会初始化的，其初始值为0。
对于函数声明（不是定义）来说，其默认storage class说明符就是extern，即`void a();`和`extern void a();`是等同的。  
<br/>

__参考文章__
***
[C Storage Classes and Storage Class Specifiers](http://cs-fundamentals.com/c-programming/storage-classes-in-c-and-storage-class-specifiers.php)
[Storage Class and Scope](http://www-ee.eng.hawaii.edu/~tep/EE160/Book/chap14/chapter2.1.html)  




