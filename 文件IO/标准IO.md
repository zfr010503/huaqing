# 文件的概念和类型

> Linux系统一切皆文件

### 概念

一组相关数据的有序集合

### 文件类型

1. 常规文件  r 
2. 目录文件  d
3. 字符设备文件  c
4. 块设备文件  b
5. 管道文件  p
6. 套接字文件  s
7. 符号链接文件  l

# 如何理解标准IO

 标准I/O由ANSI C标准定义

 主流操作系统上都实现了C库

 标准I/O通过缓冲机制减少系统调用，实现更高的效率

### IO的概念

-   I   input  输入设备 比如键盘鼠标都是Input设备
-   O output  输出设备 比如显示器
-   优盘，网口，既是输入也是输出

### 系统调用和库函数

- 系统调用就是操作系统提供的接口函数
- 如果我们把系统调用封装成库函数就可以起到隔离的作用，提供程序的可移植性
- printf就是库函数然后调用了系统调用才在显示器上显示字符

# 流（FILE）的含义

> 就是数据的流，在程序中就是一个结构体

### FILE

> 标准IO用一个结构体类型来存放打开的文件的相关信息
>
> 标准I/O的所有操作都是围绕FILE来进行

### 流（stream）

> FILE又被称为流(stream)
>
> 文本流/二进制流

### 标准I/O预定义3个流

> 标准I/O预定义3个流，程序运行时自动打开

|  标准输入流（键盘）  |  0   | STDIN_FILENO  | stdin  |
| :------------------: | :--: | :-----------: | :----: |
| 标准输出流（显示器） |  1   | STDOUT_FILENO | stdout |
|      标准错误流      |  2   | STDERR_FILENO | stderr |

# 流的缓冲类型（重点）

### **全缓冲**

当流的缓冲区无数据或无空间时才执行实际I/O操作

> 全缓冲的空间为$1k$
>
> C程序结束或者缓冲区满，缓冲区的内容才会被打印，如下：

```c
#include <stdio.h>
#include <unistd.h>
int main(int argc,char*argv[]){

    int i=0;
    for(i=0;i<1025;i++){   //i大于1024才会被打印
       printf("a");

    }
    //    printf("hello world\n");
    while(1){
    sleep(1);
    }
}
```



### **行缓冲**

当在输入和输出中遇到换行符(‘\n’)时，进行I/O操作

当流和一个终端关联时，典型的行缓冲

> Windows 和linux的换行符区别
>
> 1.   Windows是\r\n 
> 2.   Linux 是\n

### **无缓冲**

数据直接写入文件，流不进行缓冲

# 文件的打开和关闭

> 打开就是占用资源
>
> 关闭就是释放资源

### 标准I/O – 打开文件

下列函数可用于打开一个标准I/O流：

```c
FILE *fopen (const char *path, const char *mode);
```

- 成功时返回流指针；出错时返回NULL;  FILE是文件类型

**mode**参数：

| “r”  或 “rb”   | 以只读方式打开文件，文件必须存在。                           |
| -------------- | ------------------------------------------------------------ |
| “r+”  或 ”r+b” | 以读写方式打开文件，文件必须存在。                           |
| “w”  或 “wb”   | 以只写方式打开文件，若文件存在则文件长度清为0。若文件不存在则创建。 |
| “w+”  或 “w+b” | 以读写方式打开文件，其他同”w”。                              |
| “a”  或 “ab”   | 以只写方式打开文件，若文件不存在则创建；向文件写入的数  据被追加到文件末尾。 |
| “a+”  或 “a+b” | 以读写方式打开文件。其他同”a”                                |

### fopen-示例

```c
#include <stdio.h>

int main(int argc,char *argv[]){
	FILE *fp;
	fp = fopen("1.txt","r");
	if(fp == NULL){
        printf("Open filr failed\n");
		//perror("(perror)fopen");
		//printf("(strerror)fopen:%s\n",strerror(errno));
	}
	else{
		printf("Open filr success\n");
	}
	return 0;
}
```

### 处理错误信息

errno 存放错误号，由系统生成

perror先输出字符串s，再输出错误号对应的错误信息

strerror根据错误号返回对应的错误信息

```c
extern int errno;

void perror(const char *s);

char *strerror(int errno);
```

### 错误信息处理-示例

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(int argc,char *argv[]){
	FILE *fp;
	fp = fopen("1.txt","r");
	if(fp == NULL){
		perror("(perror)fopen");
		printf("(strerror)fopen:%s\n",strerror(errno));
	}
	else{
		printf("Open filr success\n");
	}
	return 0;
}
```

### 标准I/O – 关闭文件

```c
int fclose(FILE *stream)；
```

- fclose()调用成功返回0，失败返回EOF，并设置errno
- 流关闭时自动刷新缓冲中的数据并释放缓冲区
- 当一个程序正常终止时，所有打开的流都会被关闭
- 流一旦关闭后就不能执行任何操作

# 标准I/O的读写

 流支持不同的读写方式:

-  读写一个字符：fgetc()/fputc()一次读/写一个字符
-  读写一行：fgets()和fputs()一次读/写一行
-  读写若干个对象：fread()/fwrite() 每次读/写若干个对象，而每个对象具有相同的长度

==打开文件后读取，是从文件开头开始读。读完一个后读写指针会后移。读写注意文件位置！==

### 按字符输入

下列函数用来输入一个字符:

> 函数返回值设置成int型既能处理有符号数又能处理无符号数
>
> 函数返回值是int类型不是char类型，主要是为了扩展返回值的范围。

```c
 #include <stdio.h>
int fgetc(FILE \*stream); 
int getc(FILE \*stream); //宏
int getchar(void);
```

- 成功时返回读取的字符；若到文件末尾或出错时返回EOF（-1），
- getchar()等同于fgetc(stdin)
- getc()和fgetc()区别是一个是宏一个是函数
- 调用getchar会阻塞，等待你的键盘输入

示例：

```c
 FILE *fp;
 int ch, count = 0;

 if ((fp = fopen(argv[1], “r”)) == NULL) { 
  perror(“fopen”); return -1;
 }

 while ((ch = fgetc(fp)) != EOF) { 
  count++; 
 }
 
 printf(“total %d bytes\n”, count);
```

### 按字符输出

下列函数用来输出一个字符:

```c
#include <stdio.h>
int fputc(int c, FILE *stream);
int putc(int c, FILE *stream);
int putchar(int c);
```

- 成功时返回写入的字符；出错时返回EOF
- putchar(c)等同于fputc(c, stdout)

示例：

```c
 FILE *fp;
 int ch;

 if ((fp = fopen(argv[1], “w”)) == NULL) { 
   perror(“fopen”); return -1;
 }

 for(ch = ‘a’; ch <=‘z’; ch++) { 
   fputc(ch, fp); 
 }
```

### 按行输入

下列函数用来输入一行:

```c
#include <stdio.h>
char *gets(char *s);
char *fgets(char *s, int size, FILE *stream);
```

- 成功时返回s，到文件末尾或出错时返回NULL
- gets不推荐使用，没有设置长度**容易造成缓冲区溢出**
- **遇到’\n’或已输入size-1个字符时返回，总是包含’\0’**
- fgets 函数第二个参数，输入的数据超出size，size-1个字符会保存到缓冲区，最后添加’\0’，如果输入数据少于size-1 后面会添加换行符。

示例：

```c
 #define N 6
 
 char buf[N];
 fgets(buf, N, stdin);
 printf(“%s”, buf);
```

> 假设键盘输入分别是： 
>
> - abcd<回车>       buf中的内容是abcd
> - abcdef<回车>    buf中的内容是abcde

### 按行输出

下列函数用来输出字符串:

```c
#include <stdio.h>
int puts(const char *s);
int fputs(const char *s, FILE *stream);
```

- 成功时返回非负整数；出错时返回EOF
- puts将缓冲区s中的字符串输出到stdout，  **并追加’\n’**
- fputs将缓冲区s中的字符串输出到stream，**不追加 ‘\n’**

示例：

```c
 puts(“hello world”);

 FILE *fp;
 char buf[] = “hello world”;

 if ((fp = fopen(argv[1], “a”)) == NULL) { 
   perror(“fopen”); 
   return -1;
 }

 fputs(buf, fp);
```

> 注意：输出的字符串中可以包含’\n’，也可以不包含

### 按对象读写

下列函数用来从流中读写若干个对象:

```c
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t n, FILE *fp); (size_t = unsigned int)
size_t fwrite(const void *ptr, size_t size, size_t n, FILE *fp);
```

> void *ptr 读写内容放的位置指针
>
> size_t size 读写的块大小
>
> size_t n 读写的个数
>
> FILE *fp 读写的文件指针

- 成功返回读写的对象个数；出错时返回EOF
- 既可以读写文本文件，也可以读写数据文件
- 效率高

==注意事项：==

**文件写完后，文件指针指向文件末尾，如果这时候读，读不出来内容。**

解决办法：

- 移动指针（后面讲解）到文件头
- 关闭文件，重新打开