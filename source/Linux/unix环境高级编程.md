# unid环境高级编程

## 第一章 UNIX基础知识

### 1.2 UNIX体系结构

* 内核：控制计算机硬件资源，提供程序运行环境

* 系统调用：内核的接口
* 公共库：构建在系统调用接口知识
* 应用程序：可使用公共库、系统调用
* shell：特殊的应用程序，为其他应用程序提供了一个接口

![1591273071323](assets/1591273071323.png)

### 1.3 登录

#### 1.  登录名

**登录口令文件:**/etc/password

**登录项**:7个字段

* 登录名、加密口令、数据用户ID、数字组ID、注释字段、起始目录、shell程序

  `sar:x:205:105:Stephen Rago:/home/sar:/bin/ksh`

  加密口令已经移到另一个文件中

#### 2. shell

unix有不同的shell，如Boune shell; Bourne-again shell; C shell等

系统根据登录口令文件的最后一个字段判断为登录用户执行哪个shell

#### 1.4 目录

**1. 文件系统**

* 所有东西的起点为**根目录**

* **目录:** 包含目录项的文件
  * 逻辑上，每个目录项都包含一个文件名，还包含说明该文件属性的信息

  * 文件属性

    * 文件类型
    * 文件大小
    * 文件所有者
    * 文件权限
    * 文件的最后修改时间
    * 等
* **stat和fstat函数：** 返回包含所有文件属性的一个信息结构
* 目录项的逻辑视图与实际存**放在磁盘上的方式**是不同的
* 工作目录:
    * 每个进程都有一个工作目录
    * chdir函数:更改工作目录

#### 1.5 输入和输出
**1. 文件描述符**
* 小的非负整数
* 标识一个特定进程正在访问的文件

**2. 标准输入、标准输出和标准错误**
* 运行一个新程序 --> shell为其打开3个文件描述符
* 3个描述符默认链接到终端，可重定向

**3. 不带缓冲的I/O**
* open read write lseek close提供了不带缓冲的I/O, 这些函数都使用文件描述符

* 实例：
将标准输入复制到标准输出
    <details>
      <summary>代码</summary>

    ```c
    #include "apue.h" 

    #define BUFFSIZE 4096 
     
    int main(void) 
    { 
        int n; 
        char buf[BUFFSIZE]; 
     
        while ((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0) 
            if (write(STDOUT_FILENO, buf, n) != n) 
                err_sys("write error"); 
     
        if (n <0 ) 
            err_sys("read error"); 
     
        exit(0); 
        return 1; 
    } 

    ```

    </details>

* 说明:
    * 标准输入和标准输出通常定义为常量：STDING_FILENO STDOUT_FILENO
    * 文件结束符通常是:Ctrl+D 


**4. 标准I/O**
标准I/O：
* 为不带缓冲的I/O函数提供一个带缓冲的接口
* 无需担心如何选取最佳的缓冲区大小
* 实例：
使用标准I/O接口将标准输入复制到标准输出

    <details>
      <summary>代码</summary>

    ```c
    #include "apue.h"
    
    int main()
    {
        int c;
        while((c = getc(stdin)) != EOF)
            if (putc(c, stdout) == EOF)
                err_sys("out error");
        
        if (ferror(stdin))
            err_sys("input error");
        exit(0);
    }

    ```

    </details>


## 第4章 文件和目录
### 4.2 函数stat、fstat、sftatat和lstat
1. 函数定义：
```c
#include <sys/stat.h>

int stat(const char *restrict pathname, struct stat * restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const chat *restrict pathname, struct stat *restrict buf);
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);
```
* stat函数：返回与pathname文件有关的信息结构
* fstat函数：获得已在描述符fd上打开文件的有关信息
* lstat函数: 当文件是一个符号链接时，返回该符号链接有关的信息，而不是链接引用的文件的信息
* fstatat函数：
    * flag：控制是否跟随一个符号链接，当AT_SYMLINK_NOFOOLLOW标志被设置，不会跟随符号链接；默认跟随
    * fd: 值为AT_FDCWD,并且pathname参数是一个相对路径名，fstatat会计算相对于当前目录的pathname参数，如果pathname是一个绝对路径，fd参数就会被忽略

2. stat结构体:
```c
struct stat{
    mode_t              st_mode;        /* file type & mode (permissions) */
    ino_t               st_ino;         /* i-node number (serial number) */
    dev_t               st_dev;         /* device number (file system) */
    nlink_t             st_nlink;       /* number of links */
    uid_t               st_uid;         /* user ID of owner */
    gid_t               st_gid;         /* group ID of owner */
    struct timespec     st_atime;       /* time of last access */
    struct timespec     st_mtime;       /* time of last modification */
    struct timespec     st_ctime;       /* time of last file status change */
    blksize_t           st_blksize;     /* best I/O block size */
    blkcnt_t            st_blocks;      /* number of disk blocks allocated */
};
```

### 4.3 文件类型
**1. 文件类型**
* 普通文件(regular file)
* 目录文件(directory file)
* 块特殊文件(block special file)
    * 提供对设备（如磁盘）带缓冲的访问，每次访问以固定长度为单位进行
* 字符特殊文件(character special file)
    * 提供对设备不带缓冲的访问，每次访问长度可变 
* FIFO
    * 用于进程间通信，也称为命名管道（named pipe）
* 套接字(socket)
    * 进程间的网络通信
    * 一台宿主机上进程之间的非网络通信
* 符号链接(symbolic link)

**2. 文件类型宏** 

用来判断文件是什么类型的宏

**3. IPC类型宏**

用来判断ipc是什么类型的宏


### 4.4 设置用户ID和设置组ID
与一个进程相关联的ID有6个或更多
* **实际用户组ID和实际组ID：** 标识我们究竟是谁，这两个字段在登陆时取自口令文件中的登陆项。
* **有效用户ID、有效组ID以及附属组ID：** 决定文件的访问权限
* **保存的设置用户ID和保存的设置组ID：** 在执行一个程序时，包含了有效用户ID和有效值ID的副本

每个文件有一个所有者和组所有者，分别由stat结构中的st_uid和st_gid指定

当执行一个程序文件时，进程的有效用户ID通常就是实际用户ID，有效组ID通常是实际组ID。但可以在`文件模式字（st_mode）`中设置一个特殊标志：使得“当执行此文件时，将进程的有效用户ID设置为文件所有者的用户ID（st_uid）。同样可以设置组ID”

例如：若文件的所有者是超级用户，而且设置了该文件的设置用户ID位，那么当该程序文件由一个进程执行时，该进程具有超级用户权限。不过执行此文件的进程的实际用户ID是什么


### 4.5 文件访问权限

### 4.6 新文件和目录的所有权
新文件的权限：
* 用户ID：
    * 新文件的用户ID设置为进程的有效用户ID
* 组ID：POSIX.1允许实现选择下列之一作为新文件的组ID
    * 新文件的组ID可以是进程的有效组ID
    * 新文件的组ID可以是它所在目录的组ID
    * linux：新文件的组ID取决于它所在的目录的设置组ID位是否被设置。如果设置，则新文件的组ID设置为目录ID

### 4.7 函数access和faccessat
access函数和faccessat函数按实际用户ID和实际组ID进行访问权限测试，faccessat函数也可以测试有效用户ID和有效组ID的权限
```c
#include <unist.h>
int access(const char *pathname, int mode);
int faccessat(int fd, const char *pathname, int mode, int flag);
```

mode 参数:
* R_OK: 测试读权限
* W_OK: 测试写权限
* X_OK: 测试执行权限


## 第5章 标准I/O库

### 5.2 流和FILE对象
**1. 不带缓冲的IO和标准IO库**
* 在不带缓冲的IO中，所有I/O函数都是围绕文件描述符的
* 对于标准I/O库，都是围绕流（stream）进行的

当用标准I/O库打开或创建一个文件时，我们已使一个流和文件相关联

**2. 流的定向：**
* 流的定向决定了所读、写的字符时单字节还是多字节的
* 当一个流最初创建时，它并没有定向
    * 在未定向的流上使用一个多字节I/O函数，则将该流的定向设置为宽定向的
    * 在未定向的流上使用一个单字节I/O函数，则将该流的定向设置为单字节定向的

* freopen函数清除一个流的定向
* fwide函数可用于设置流的定向

### 5.3 标准输入、标准输出和标准错误
* 在头文件<stdio.h>中定义了文件指针stdin、stdout和stderr代表标准输入、标准输出和标准出错
* 这些流引用的文件与文件描述符STDIN_FILENO、STDOUT_FILENO和STDERR_FILENO所引用的相同



### 5.4 缓冲

标准I/O库提供缓冲的目的是尽可能减少使用read和write调用的次数，对每个I/O流自动地进行缓冲管理。

**标准I/O提供3种类型的缓冲:**

* **全缓冲**

  在填满标准I/O缓冲区后才进行实际I/O操作，对于驻留在磁盘上的文件通常是由标准I/O库实施全缓冲的。在一个流上执行第一次I/O操作，相关标准I/O函数通常调用malloc获得需使用的缓冲区

* **行缓冲**

    当在输入和输出遇到换行符时，标准I/O执行I/O操作。这允许我们一次输出一个字符（用标准I/O函数fputc），但只有在写了一行之后才进行实际I/O操作。
    行缓冲的两个限制：

    * 行缓冲区的长度是固定的，只要填满了缓冲区，没有换行符也进行I/O操作。
    * 任何时候只要通过标准I/O库要求从（a）一个不带缓冲的流或者（b）一个行缓冲的流（它从内核请求需要数据）得到输入数据，那么就会冲洗所有行缓冲输出流。在（b）中带了一个在括号中的说明，其理由是，所需的数据可能已在该缓冲区中，它并不要求一定从内核读取数据。很明显，从一个不带缓冲的流中输入（即（a）项）需要从内核获得数据。

* **不带缓冲**
  
    标准I/O库不对字符进行缓冲存储。例如，若用标准I/O函数fputs写15个字符到不带缓冲的流中，我们就期望这15个字符能立即输出。

    标准错误流stderr通常是不带缓冲的。

**冲洗：**

* 在标准I/O库方面，flush（冲洗）意味着将缓冲区中的内容写到磁盘上
* 在终端驱动程序方面，flush（刷清）表示丢弃已存储在缓冲区中的数据

### 5.5 打开流

下列3个函数打开一个标准I/O流

```c
#include <stdio.h>
FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
FILE *fdopen(int fd, const char *type);
```

关闭流

```c
#include <stdio.h>
int fclose(FILE *fp);
```



ISO C要求下列缓冲特征：
* 当且仅当标准输入和标准输出并不指向交互式设备时，它们才是全缓冲的
* 标准错误绝不会是全缓冲的

很多系统默认使用下类类型的缓冲
* 标准错误是不带缓冲的
* 若是指向终端设备的流，则是行缓冲的；否则是全缓冲的

两个更改缓冲类型的函数
```c
#include <stdio.h>
void setbuf(FILE *restrict fp, char *restrict buf);
int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
```

冲洗流函数
```c
#include <stdio.h>
int fflush(FILE *fp);
```
该函数使流所有未写的数据都被传送至内核。若fp是NULL，则此函数将导致所有输出流被冲洗至内核。若fp是NULL，则此函数将导致所有输出流被冲洗。

### 5.6 读和写流
一旦打开了流，则可在3中不同类型的非格式化I/O中进行选择，对其进行读、写操作：
* 每次一个字符的I/O

    一次读或写一个字符，如果流是带缓冲的，则标准I/O函数处理所有缓冲
* 每次一行的I/O
  
    如果想要一次读或写一行，则使用fgets和fputs。每行都以一个换行符终止。
    
* 直接I/O
  
    fread和fwrite函数支持这种类型的I/O。每次I/O操作读或写某种数量的对象，而每个对象具有指定的长度。

**1. 输入函数**

以下3个函数可用于一次读一个字符
```c
#include <stdio.h>
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);
```

大部分实现中，为每个流在FILE对象中维护了两个标志：
* 出错标志
* 文件结束标志

```c
#include <stdio.h>

// 用来判断是出错还是到达文件位端的两个函数
int ferror(FILE *fp);
int feof(FILE *fp);

//清除FILE对象中的两个标志：出错标志和文件结束标志
void clearerr(FILE *fp);
```

```c
#include <stdio.h>

//将字符压送回流中
int ungetc(int c, FILE *fp);
```

**2. 输出函数**

每次输出一个字符的函数:
```c
#include <stdio.h>

int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
```

### 5.7 每次一行I/O
下面两个函数提供每次输入一行的功能
```c
#include <stdio.h>

char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf);
```

提供每次输出一行的函数
```c
#include <stdio.h>

int fputs(const char *restrict str, FILE *restrict fp);
int puts(char  *str);
```

### 5.8 标准I/O的效率
用getc和putc将标准输入复制到标准输出
<details>
  <summary>代码</summary>

```c
#include "apue.h" 

#define BUFFSIZE 4096 
 
int main(void) 
{ 
    int c;

    while ((c = getc(stdin)) != EOF)
        if (putc(c, stdout) == EOF)
            err_sys("output error");

    if (ferror(stdin))
        err_sys("input error");
 
    exit(0); 
    return 1; 
} 

```

</details>
用fgets和fputs将标准输入复制到标准输出
<details>
  <summary>代码</summary>

```c
#include "apue.h" 

#define BUFFSIZE 4096 
 
int main(void) 
{ 
    char buf[MAXLINE];

    while (fgets(buf, MAXLINE, stdin) != NULL)
        if (fputs(buf, stdout) == EOF)
            err_sys("output error");

    if (ferror(stdin))
        err_sys("input error");
 
    exit(0); 
    return 1; 
} 

```

</details>

效率的对比（详细看书P124）

**系统调用与普通的函数调用相比需要花费更多的时间**

### 5.9 二进制I/O

如果进行二进制I/O操作，那么我们更愿意一次读/写一个完整的结构体，使用getc、putc一次只能读写一个字节，fputs和fgets遇到null字节就会停止。

因此，提供了两个函数执行二进制I/O操作

```c
#include <stdio.h>

size_t fread(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
size_t fwrite(const void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
```

两个常见的用法

* 读写一个二进制数组
* 读或写一个结构体

问题：在两个不同的系统交换数据时，这两个函数可能不能正常工作，原因是：

* 在一个结构体中，同一成员的偏移量可能随编译程序和系统的不同而不同；在同一个系统中，一个结构的二进制存放方式也可能因编译程序选项的不同而不同
* 用来存储多字节整数和浮点值的二进制格式在不同的系统结构间也可能不同

### 5.10 定位流

有3种方法定位标准I/O流：

* `ftell`和`fseek`函数

  假定文件的位置可以存放在一个长整型中

  ```c
  #include <stdio.h>
  long ftell(FILE *fp);
  int fseek(FILE *fp, long offset, int whence);
  void rewind(FILE *fp);
  ```

  * 对于一个二进制文件，其文件位置指示器是从文件起始位置开始度量，并以字节为度量单位的。
  * 为了定位一个文本文件，whence一定要是SEEK_SET，而且offset只能有两种值：0（后退到文件的起始位置），或是对该文件的ftell所返回的值。使用rewind函数也可将一个流设置到文件的起始位置。

* `ftello`和`fseeko`函数

  Single UNIX Specification引入，使用off_t数据类型代替了长整型

  处理偏移量类型以外，ftello函数和ftell相同，fseeko函数与fseek相同

  ```c
  #include <stdio.h>
  off_t ftello(FILE *fp);
  
  int fseeko(FILE *fp, off_t offset, int whence);
  ```

* `fgetpos`和`fsetpos`函数

  ISO C引入，使用一个抽象数据类型fpos_t记录文件的位置。这种数据类型可以根据需要定义一个足够大的数，用以记录文件位置。需要移植到非unix系统应使用fgetpos和fsetpos

  ```c
  #include <stdio.h>
  
  int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
  int fsetpos(FILE *fp, const fpos_t *pos);
  ```



### 5.11 格式化I/O

**1. 格式化输出**

* 5个printf函数和printf函数族的变体

**2. 格式化输入**

* 3个scanf函数和scanf函数族的变体

### 5.12 实现细节

每个标准I/O流都有一个与其相关联的文件描述符，可对一个流调用fileno函数以获得其描述符

```c
#include <stdio.h>
int fileno(FILE *fp);
```

如果要调用dup或fcntl等函数，则需要此函数。

实例： 

图5-11程序：为3个标准流以及一个与普通文件相关联的流 打印有关缓冲的转态信息，并对比标准输入、输出连接到终端或者普通文件时，有何不同

```c

```





## 备注
编译
gcc xxx -lapue


