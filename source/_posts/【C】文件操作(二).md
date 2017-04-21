title: 【C】文件操作(二)
date: 2016-03-04 18:45:21
tags: C
categories: C
---
## 前言
这里主要记录一下C对二进制的读写操作, 包括随机读取文件和写入文件
## fseek 和 ftell
__fseek__
fseek主要用来移动文件指针, 它允许用户像对待数组那样对待一个文件, 可以直接将文件指针移动到任意字节处, 下面是它的函数原型:
```C
int fseek ( FILE * stream, long int offset, int origin );
```
下面是个参数的含义
* stream - 打开的文件指针
* offset - 偏移量, 表示从起始点开始要移动的距离(起始点的选择由origin指定), 可以为正(向前移)、负(往回移), 也可以为零(保持不动).
* origin - 用来指定起始点的模式, 可以使用下面定义的几个模式常量:
    1. SEEK_SET : 以文件开始位置作为起始点
    2. SEEK_CUR : 以文件指针当前所在的位置作为起始点
    3. SEEK_END : 以文件结尾作为起始点

<!-- more -->
下面是一些使用示例, 其中fp是一个文件指针
```C
fseek(fp, 0L, SEEK_SET)  // 移动到文件开头
fseek(fp, 10L, SEEK_SET)  // 移动到文件的第10个字节
fseek(fp, 2L, SEEK_CUR)  // 从文件的当前位置向前移动两个字节
fssek(fp, 0L, SEEK_END)  // 移动到文件的结尾处
fseek(fp, -10L, SEEK_END)  // 从文件结尾处退回10个字节
```
如果函数执行正常, 那么返回值为0, 如果有错误, 则返回值为-1.
__ftell__
ftell函数用来获得当前文件指针的位置, 它返回当前文件指针距离文件开始处的字节数目, 函数原型如下
```C
long int ftell ( FILE * stream );
```
如果函数执行失败会返回-1,

下面是一个使用示例, 接合fseek和ftell用来获得文件的大小
```C
long file_size(char *fileName) {
    long size;
    FILE *fp;
    if((fp = fopen(fileName, "rb")) == NULL) {
        printf("can't open file %s\n", fileName);
        exit(EXIT_FAILURE);
    }
    fseek (fp, 0 , SEEK_END);
    size = ftell(fp);

    double mb;
    mb = size * 1.0 / 1024 / 1024;
    printf("file size is %ldB and %.2fMB \n", size, mb); 
    fclose(fp);
    return size;
}
```
## fwrite
> Writes an array of __count__ elements, each one with a size of __size__ bytes, from the block of memory pointed by __ptr__ to the current position in the __stream__.

以二进制的形式将数据块写入文件, 函数原型为:
```C
size_t fwrite ( const void * ptr, size_t size, size_t count, FILE * stream );
```
下面是参数意义:
* ptr - 要写入的内存数据地址
* size - 单个数据块的大小
* count - 写入的数据块的数量
* stream - 写入的目标文件

如果写入成功, 会返回写入的数据块的数量, 即count, 如果返回值不等于count, 说明程序运行出现了错误.
下面是一个使用示例:
```C

void file_fwrite(char *fileName) {
    FILE *fp;
    if((fp = fopen(fileName, "w+b")) == NULL) {
        printf("can't open file %s\n", fileName);
        exit(EXIT_FAILURE);
    }
    int count = 20, i;
    int *buffer;
    buffer = malloc(sizeof(int) * count);
    for(i = 0; i < count; i++) {
        buffer[i] = i;
    }
    
    long success_num = 0;
    success_num = fwrite(buffer, sizeof(int) , count, fp);
    printf("success_num is %ld\n", success_num);

    fclose(fp);
    free(buffer);
}
```
下面是写入的内容
```html
0000 0000 0100 0000 0200 0000 0300 0000
0400 0000 0500 0000 0600 0000 0700 0000
0800 0000 0900 0000 0a00 0000 0b00 0000
0c00 0000 0d00 0000 0e00 0000 0f00 0000
1000 0000 1100 0000 1200 0000 1300 0000
```
上面是以16进制的形式进行显示, 即一个数字为4位, 一个int值占32位(4个字节),  在上面的内容中, 8个数字为1个int, 如 `0000 0000`为第一个int值, 即0, `0100 0000`为第二个int值, 即1. 这里需要说明的是在写入时是字节作为一个基本单位的, 并且低位字节是先写入的, 如`0100 0000`, 其中`01`就是int的最低位的字节. 我们来看这个例子, 如果写入文件之后的值为`1234 5678`, 那么其原先的值就是`0x78563412`

## fread
> Reads an array of __count__ elements, each one with a size of __size__ bytes, from the __stream__ and stores them in the block of memory specified by __ptr__.

以二进制的形式将数据块读入内存, 下面是函数原型:
```C
size_t fread ( void * ptr, size_t size, size_t count, FILE * stream );
```
下面是参数含义
* ptr - 读入文件数据的内存地址
* size - 单个数据块的大小
* count - 数据块的数量
* stream - 读取的文件

如果读入成功, 会返回读入的数据块的数量. 下面是一个使用示例:
```C
void file_fread(char *fileName) {
    long count;
    int *buffer;
    FILE *fp;
    if((fp = fopen(fileName, "rb")) == NULL) {
        printf("can't open file %s\n", fileName);
        exit(EXIT_FAILURE);
    }
    fseek (fp, 0 , SEEK_END);
    count = ftell(fp);
    rewind(fp);
    buffer =(int *) malloc(sizeof(int) * count);

    fread(buffer, sizeof(int), count / sizeof(int), fp);
    
    int i;
    for(i = 0; i < count/ sizeof(int); i++) {
        printf("%d\n", buffer[i]);
    }
    fclose(fp);
    free(buffer);
}
```
## rewind
> Sets the position indicator associated with stream to the beginning of the file.

重置文件指针到文件开头位置, 下面是函数原型:
```C
void rewind ( FILE * stream );
```
## setbuf 和 setvbuf
当打开一个文件后, 系统会自动为该文件流分配一个缓冲区, 其大小为`BUFSIZ`, 我们可以通过打印`BUFSIZ`来获得默认的缓冲区大小. 如果想自定义缓冲区, 可以使用setbuf和setvbuf函数
```C
printf("%d", BUFSIZ);
```
__setbuf__

> Specifies the buffer to be used by the stream for I/O operations, which becomes a fully buffered stream. Or, alternatively, if buffer is a null pointer, buffering is disabled for the stream, which becomes an unbuffered stream.

为文件流指定一个缓冲区, 函数原型为
```C
void setbuf ( FILE * stream, char * buffer );
```
buffer表示指定的缓冲区, 需要注意的一点是, 这里缓冲区的大小仍然为`BUFSIZ`, 只不过是缓冲区的位置发生了改变, 因此buffer的大小应该大于或者等于`BUFSIZ`.  该函数应该在文件刚被打开时调用, 不能在进行了读写操作之后再调用. 如果buffer的为NULL, 就表示禁用缓冲区. 下面是一个使用示例:
```C
#include <stdio.h>

int main ()
{
  char buffer[BUFSIZ];
  FILE *pFile1, *pFile2;

  pFile1=fopen ("myfile1.txt","w");
  pFile2=fopen ("myfile2.txt","a");

  setbuf ( pFile1 , buffer );
  fputs ("This is sent to a buffered stream",pFile1);
  fflush (pFile1);

  setbuf ( pFile2 , NULL );
  fputs ("This is sent to an unbuffered stream",pFile2);

  fclose (pFile1);
  fclose (pFile2);

  return 0;
}
```

__setvbuf__

> Specifies a buffer for stream. The function allows to specify the mode and size of the buffer (in bytes).

为文件指定一个缓冲区, 同时可以指定缓冲区的类型和大小, 下面是函数原型:
```C
int setvbuf ( FILE * stream, char * buffer, int mode, size_t size );
```
其中 stream表示操作的文件, buffer为指定的缓冲区首地址, 如果缓冲区为NULL, 系统会自动创建一个大小为size的缓冲区, mode为缓冲区的类别, size为缓冲区的大小, 其中mode的值可以为下面几个:
* **_IOFBF** - 全缓冲(Full buffering), 当缓冲区满时才执行真正的I/O操作, 例如对磁盘文件的读写. 
>  On output, data is written once the buffer is full (or flushed). On Input, the buffer is filled when an input operation is requested and the buffer is empty.

* **_IOLBF** - 行缓冲(Line buffering), 在输入和输出时遇到换行符时才进行真正的I/O操作, 例如标准输入(stdin)和标准输出(stdout). 
> On output, data is written when a newline character is inserted into the stream or when the buffer is full (or flushed), whatever happens first. On Input, the buffer is filled up to the next newline character when an input operation is requested and the buffer is empty. 

* **_IONBF** - 无缓冲(No buffering), 在这种情况下buffer和size参数会被忽略.

其实setbuf将相当于调用了setvbuf
```C
setvbuf(stream, buf, buf ? _IOFBF : _IONBF, BUFSIZE)
```
下面是一个使用示例:
```C
#include <stdio.h>

int main (){
  FILE *pFile;
  pFile=fopen ("myfile.txt","w");
  setvbuf ( pFile , NULL , _IOFBF , 1024 );

  // File operations here

  fclose (pFile);
  return 0;
}
```

## fflush
对于输入输出流, 下列情况会自动刷新缓冲区
* 当进行输出(output)操作时, 输出缓冲区满了
* 当流(stream)被关闭
* 当程序调用exit方法终止
* 当缓冲区为行缓冲区时, 一个换行符(newline)被写入
* Whenever an input operation on any stream actually reads data from its file.

对于一个输出流, 可以调用fflush进行显示的刷新缓冲区, 即将缓冲区的内容写入到文件中, 但是对于一个输入流使用fflush函数的效果没有定义. 下面是函数原型:
```C
int fflush ( FILE * stream );
```
如果stream为NULL, 那么所有的缓冲区都将被刷新.

## stat
stat函数主要用于获取文件状态, 函数原型为
```C
int stat (const char *filename, struct stat *buf)
```
下面是struct stat的定义:
```C
struct stat {
    //device 文件的设备编号
    dev_t st_dev;   
    //inode 文件的i-node
    ino_t st_ino;   
    //protection 文件的类型和存取的权限
    mode_t st_mode;  
    //number of hard links 连到该文件的硬连接数目, 刚建立的文件值为1.
    nlink_t st_nlink;  
    //user ID of owner 文件所有者的用户识别码
    uid_t st_uid;  
    //group ID of owner 文件所有者的组识别码
    gid_t st_gid;  
    //device type 若此文件为装置设备文件, 则为其设备编号
    dev_t st_rdev;  
    //total size, in bytes 文件大小, 以字节计算
    off_t st_size;  
    //blocksize for filesystem I/O 文件系统的I/O 缓冲区大小.
    unsigned long st_blksize;  
    //number of blocks allocated 占用文件区块的个数, 每一区块大小为512 个字节.
    unsigned long st_blocks; 
    //time of lastaccess 文件最近一次被存取或被执行的时间, 一般只有在用mknod、utime、read、write 与tructate 时改变.
    time_t st_atime;  
    //time of last modification 文件最后一次被修改的时间, 一般只有在用mknod、utime 和write 时才会改变
    time_t st_mtime;  
    //time of last change i-node 最近一次被更改的时间, 此参数会在文件所有者、组、权限被更改时更新
    time_t st_ctime;  
};
```
其中st_mode定义了以下数种情况
1. **S_IFMT** - 0170000 文件类型的位遮罩
2. **S_IFSOCK** - 0140000 scoket
3. **S_IFLNK** - 0120000 符号连接
4. **S_IFREG** - 0100000 一般文件
5. **S_IFBLK** - 0060000 区块装置
6. **S_IFDIR** - 0040000 目录
7. **S_IFCHR** - 0020000 字符装置
8. **S_IFIFO** - 0010000 先进先出
9. **S_ISUID** - 04000 文件的 (set user-id on execution)位
10. **S_ISGID** - 02000 文件的 (set group-id on execution)位
11. **S_ISVTX** - 01000 文件的sticky 位
12. **S_IRUSR (S_IREAD)** - 00400 文件所有者具可读取权限
13. **S_IWUSR (S_IWRITE)** - 00200 文件所有者具可写入权限
14. **S_IXUSR (S_IEXEC)** - 00100 文件所有者具可执行权限
15. **S_IRGRP** - 00040 用户组具可读取权限
16. **S_IWGRP** - 00020 用户组具可写入权限
17. **S_IXGRP** - 00010 用户组具可执行权限
18. **S_IROTH** - 00004 其他用户具可读取权限
19. **S_IWOTH** - 00002 其他用户具可写入权限
20. **S_IXOTH** - 00001 其他用户具可执行权限上述的文件类型在 POSIX 中定义了检查这些类型的宏定义
21. **S_ISLNK (st_mode)** - 判断是否为符号连接
22. **S_ISREG (st_mode)** - 是否为一般文件
23. **S_ISDIR (st_mode)** - 是否为目录
24. **S_ISCHR (st_mode)** - 是否为字符装置文件
25. **S_ISBLK (s3e)** - 是否为先进先出
26. **S_ISSOCK (st_mode)** - 是否为socket 若一目录具有sticky 位 (S_ISVTX), 则表示在此目录下的文件只能被该文件所有者、此目录所有者或root 来删除或改名.

下面是一个使用示例:
```C
void file_stat(char * fileName) {

    struct stat fileStat;
    stat(fileName, &fileStat);
 
    printf("Information for %s\n", fileName);
    printf("---------------------------\n");
    printf("File Size: \t\t%ld bytes\n",fileStat.st_size);
    printf("Number of Links: \t%ld\n",fileStat.st_nlink);
    printf("File inode: \t\t%ld\n",fileStat.st_ino);
 
    printf("File Permissions: \t");
    printf( (S_ISDIR(fileStat.st_mode)) ? "d" : "-");
    printf( (fileStat.st_mode & S_IRUSR) ? "r" : "-");
    printf( (fileStat.st_mode & S_IWUSR) ? "w" : "-");
    printf( (fileStat.st_mode & S_IXUSR) ? "x" : "-");
    printf( (fileStat.st_mode & S_IRGRP) ? "r" : "-");
    printf( (fileStat.st_mode & S_IWGRP) ? "w" : "-");
    printf( (fileStat.st_mode & S_IXGRP) ? "x" : "-");
    printf( (fileStat.st_mode & S_IROTH) ? "r" : "-");
    printf( (fileStat.st_mode & S_IWOTH) ? "w" : "-");
    printf( (fileStat.st_mode & S_IXOTH) ? "x" : "-");
    printf("\n\n");
 
    printf("The file %s a symbolic link\n", (S_ISLNK(fileStat.st_mode)) ? "is" : "is not");
}
```
输出结果如下:
```html
Information for data/test.txt
---------------------------
File Size:      24023896 bytes
Number of Links:    1
File inode:         8261278
File Permissions:   -rw-rw-r--

The file is not a symbolic link
```



