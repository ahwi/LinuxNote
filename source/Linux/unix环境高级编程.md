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















## 备注
编译
gcc xxx -lapue


