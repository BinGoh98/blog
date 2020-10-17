---
title: "「csapp」Unix系统级IO"
date: 2020-10-17T13:19:33+08:00
draft: false
toc: true
author: "彬Goh"
keywords: ["csapp","read", "write", "io"]
description: "「csapp」Unix系统级IO"
tags: ["csapp"]
categories: ["操作系统"]
---


输入/输出（I/O）是指主存与外部设备的数据复制过程。**输入**是指从I/O设备复制数据到主存，例如从磁盘读数据或者获得键盘的输入；**输出**则是从主存将数据复制到I/O设备。

<!--more-->


## 文件概述

一个 linux 文件就是一个m个字节的序列。在linux中，一切皆文件。<u>所有的I/O都被当作是对应文件的读和写。</u>

每个linux文件都有一个类型。Linux 内核将所有文件组织称一个目录层次结构，放在根目录「/」下面。

| 文件类型 | 英文名称         | 含义                                                     | 符号 |
| -------- | ---------------- | -------------------------------------------------------- | ---- |
| 普通文件 | regular file     | 一般的文件。通常要区分<u>二进制文件</u>和<u>文本文件</u> | -    |
| 目录     | directory        | 包含一组链接，每个链接都将一个文件名映射到另一个文件     | d    |
| 套接字   | socket           | 与另一个进程进行跨网络通信的文件                         | s    |
| 命名通道 | named pipe       |   一种先进先出的通信机制                  | p    |
| 符号链接 | symbolic link    |    一种快捷方式                                         | l    |
| 字符设备 | character device |      串行端口的接口设备，例如键盘、鼠标等等                                   | c    |
| 块设备   | block device     |     存储数据以供系统存取的接口设备，简单而言就是硬盘              | b    |

查看文件类型，`drwxr-xr-x`的第一个字符`d`表示这是一个 directory。

 ```bash
> ls -l
drwxr-xr-x   3 bingoh  staff    96 10  5 12:14 BlogContent
> file BlogContent
BlogContent: directory

> ls -lhi # human readable / inode
1523864 drwxr-xr-x   3 bingoh  staff    96B 10  5 12:14 BlogContent
392514 drwxr-xr-x   4 bingoh  staff   128B  9 28 15:01 go
 ```

## 打开和关闭文件

#### 文件描述符

应用程序通过要求内核打开相应的文件。之后内核返回一个小的非负整数，叫做<u>文件描述符</u>。对于应用程序来说，只需要记住这个描述符就行了，内核负责记录打开文件的所有信息。

**每个进程都有自己独立的描述符表。**这就意味着，当两个进程都尝试打开文件时，它们的文件描述符都是从`3`开始增长的。当文件读写结束时候，应用通知内核关闭这个文件，内核就会释放文件打开时创建的数据结构，并将这个文件描述符恢复到可用的描述符池中，下次可以再使用。

Linux shell创建的每个进程，开始时都有三个打开的文件：标准输入（fd = 0），标准输出（fd = 1），标准错误（fd = 2）。

在`<unistd.h>`中定义了常量来代替显式的描述符值。

```c
#define	 STDIN_FILENO	0	/* standard input file descriptor */
#define	STDOUT_FILENO	1	/* standard output file descriptor */
#define	STDERR_FILENO	2	/* standard error file descriptor */
```

进程各自维护的描述符表中的每个文件描述符，指向<u>打开文件表</u>，这个是所有进程共享的。文件表的每一项包括打开文件的文件位置、引用计数和指向`v-node表`的指针。引用计数指的是指向当前文件的饮用的数目。内核不会删除这个文件表表项，直到`refcnt = 0`。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gjr3ufay6bj314g0owwyz.jpg" style="zoom: 33%;" />

既然我们说每次打开一个新的文件，都是一个新的`fd`，并且不同进程之间是不共享描述符表的，那么在什么情况下`refcnt > 1`呢？

举个🌰，就是在`fork()`的时候。

假设`a.txt`文件的内容是

```c
123456789
abcdefghijk
```

执行下面这段代码。fork 一个子进程，在子进程里改变文件当前位置。

```c
// io.c
#include <stdio.h>
#include <sys/fcntl.h>
#include <unistd.h>
#include <sys/stat.h>
#include <sys/wait.h>

int main()
{
	int fd = open("a.txt", O_RDONLY);
    int pid;
    if((pid = fork()) == 0) {
        // 子进程
        printf("child process pid = %d\n", getpid());
        // 移动文件当前位置
        lseek(fd, 10L, SEEK_CUR);
        close(fd);
        return 0;
    }
    // 父进程
    wait(NULL);
    printf("father process pid = %d\n", getpid());
    char str[5];
    str[4] = '\0';
    read(fd, str, 4);
    printf("father read: %s\n", str);
    close(fd);
    return 0;
}
```

输出结果如下：

```c
child process pid = 14292
father process pid = 14290
father read: abcd
```

说明在子进程中移动文件当前位置的时候，父进程中文件位置也被影响了，所以最后读出的是`abcd`而不是`1234`。这是因为当我们`fork`了一个子进程之后，连着文件描述符表也拷贝了过来，而表中实际放着的是指向`打开文件表`的指针。既然拷贝的是指针，那么这两个进程实际指向的`文件表`其实是同一个。并且只有当父进程和子进程都关闭了文件之后，内核才会删除这个文件表表项。

#### 命令

进程通过`open`函数来打开一个已存在的文件或者创建一个新文件：

```c
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>

int open(char *filename, int flags, mode_t mode);
```

`open`将`filename`转换为一个文件描述符。文件描述符总是在进程中当前没有打开的最小描述符。

`flag`参数指明了进程打算如何访问这个文件。

| flag     | 含义 | flag     | 含义                                     |
| -------- | ---- | -------- | ---------------------------------------- |
| O_RDONLY | 只读 | O_CREAT  | 如果文件不存在，就创建一个空文件         |
| O_WRONLY | 只写 | O_TRUNC  | 如果文件存在，则使其成为空文件           |
| O_RDWR   | 读写 | O_APPEND | 在每次写操作前，设置文件位置到文件结尾处 |

> `fd = open("a.txt", O_WRONLY | O_APPEND);`
>
> 这个代码说明打开一个已存在的文件，在文件结尾处写东西。

`open`函数还有一个`mode`参数。指定了新文件的访问权限（由于需要是新文件，所以只对`O_CREAT`有效）。

作为上下文的一部分，每一个进程都有一个`umask`，可以通过`umask`来设置。文件的访问权限就是`mode & ~umask`。也就是`mode`所指定的权限去除掉`umask`中的权限。

`umask` 和 `mode`的各位的值可以如下：

| 掩码    | 描述                               |
| ------- | ---------------------------------- |
| S_IRUSR | 使用者（拥有者）能够读这个文件     |
| S_IWUSR | 使用者（拥有者）能够写这个文件     |
| S_IXUSR | 使用者（拥有者）能够执行这个文件   |
| S_IRGRP | 拥有者所在组的成员能够读这个文件   |
| S_IWGRP | 拥有者所在组的成员能够写这个文件   |
| S_IXGRP | 拥有者所在组的成员能够执行这个文件 |
| S_IROTH | 其他人（任何人）能够读这个文件     |
| S_IWOTH | 其他人（任何人）能够写这个文件     |
| S_IXOTH | 其他人（任何人）能够执行这个文件   |

系统一般会有默认的`umask`。比如我的电脑默认就是`022`。

```bash
> umask
022 # 这个是个`ls -l`输出的文件权限`-rwxr-xr-x`含义是一样的。022 == -----w--w-
```

```c
// 含义：文件拥有者有读写权限，所有其他人有读权限
#define DEF_MODE   S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH
#define DEF_UMASK  S_IWGRP|S_IWOTH
umask(DEF_UMASK); // 如果没有设置，则默认是 022
int fd = open("foo.txt", O_CREAT|O_TRUNC|O_WRONLY, DEF_MODE);

// 直接用数字表示也可以，并且更直观。但是必须要最前面加上0，否则会按照十进制去解析它
umask(0100);
int fd = open("foo2.txt", O_CREAT, 0764); // -rwxrw-r-- 减去 ---x------ 等于 -rw-rw-r--

// 结果符合预期
-rw-rw-r--  1 bingoh  staff  0 10 16 14:56 temp.txt
```

## 读写文件

应用程序使用`read` 和`write`来执行输入和输出。

```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t n); // 返回：若成功则为读的字节数，若 EOF 则为0，若出错为 -1。
ssize_t write(int fd, const void *buf, size_t n); // 返回：若成功则为写的字节数，若出错则为 -1。
```

**用法**

```c
int fd = open("temp.txt", O_CREAT | O_RDWR);

char* buf = "hello world!";
write(fd, buf, 12); // 写入文件

char str[11]; // 必须大一些，最后放上 \0，否则会有意想不到的输出。
lseek(fd, 0L, SEEK_SET); // 文件位置回到开头
read(fd, str, 10);
printf("read: %s\n", str); // read: hello worl

close(fd);
```

## 带缓冲的读写

众所周知，I/O是非常耗时的。当我们需要一个个读取文本文件中的字符的时候，频繁I/O会导致开销很大。一种经典的解决思路就是使用buffer，预先读数据填满缓冲区。用户从缓冲区读字符，当缓冲区为空时，再次调用I/O填满缓冲区。

Robust I/O 是csapp中引入的一个包，健壮地实现了I/O。

#### RIO无缓冲的输入输出函数

```c
#include "csapp.h"

ssize_t rio_readn(int fd, void *usrbuf, size_t n);
ssize_t rio_writen(int fd, void *usrbuf, size_t n); 
// 返回：若成功则为传送的字节数，若 EOF 则为 0(只对 rio_readn 而言)，若出错则为 -1。
```

这两个函数的实现如下：

```c
ssize_t rio_readn(int fd, void *usrbuf, size_t n)
{
    size_t nleft = n;
    ssize_t nread;
    char *bufp = usrbuf;

    while (nleft > 0) {
        if ((nread = read(fd, bufp, nleft)) < 0) {
            if (errno == EINTR) /* Interrupted by sig handler return */
                nread = 0;      /* and call read() again */
            else
                return -1;      /* errno set by read() */
        }
        else if (nread == 0)
            break;              /* EOF */
        nleft -= nread;
        bufp += nread;
    }
    return (n - nleft);         /* Return >= 0 */
}

ssize_t rio_writen(int fd, void *usrbuf, size_t n)
{
    size_t nleft = n;
    ssize_t nwritten;
    char *bufp = usrbuf;

    while (nleft > 0) {
        if ((nwritten = write(fd, bufp, nleft)) <= 0) {
            if (errno == EINTR)  /* Interrupted by sig handler return */
                nwritten = 0;    /* and call write() again */
            else
                return -1;       /* errno set by write() */
        }
        nleft -= nwritten;
        bufp += nwritten;
    }
    return n;
}
```

感觉除了在被中断之后，会重启`read`和`wirte`，其他和`read`、`wirte`并没有太大的区别。

#### RIO 带缓冲的输入函数

定义了一个结构，将缓冲区和文件描述符关联。

```c
#define RIO_BUFSIZE 8192
typedef struct {
    int rio_fd;                /* 关联的文件描述符 */
    int rio_cnt;               /* 缓冲中未读的字节 */
    char *rio_bufptr;          /* 指向下一个未读的指针 */
    char rio_buf[RIO_BUFSIZE]; /* 内置的缓冲区 */
} rio_t;
```

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr8ozp0wfj31jb0u01d2.jpg)

执行`rio_readinitb`将`fd`与`rio_t`相关联

```c
void rio_readinitb(rio_t *rp, int fd)
{
    rp->rio_fd = fd;
    rp->rio_cnt = 0;
    rp->rio_bufptr = rp->rio_buf;
}
```

之后调用`rio_read`就是按照上图进行读了。如果缓冲区是空的，则使用系统函数`read`去读。若非空则直接在缓冲区读。

```c
static ssize_t rio_read(rio_t *rp, char *usrbuf, size_t n)
{
    int cnt;

    while (rp->rio_cnt <= 0) {  /* Refill if buf is empty */
        rp->rio_cnt = read(rp->rio_fd, rp->rio_buf,
                           sizeof(rp->rio_buf));
        if (rp->rio_cnt < 0) {
            if (errno != EINTR) /* Interrupted by sig handler return */
                return -1;
        }
        else if (rp->rio_cnt == 0)  /* EOF */
            return 0;
        else
            rp->rio_bufptr = rp->rio_buf; /* Reset buffer ptr */
    }

    /* Copy min(n, rp->rio_cnt) bytes from internal buf to user buf */
    cnt = n;
    if (rp->rio_cnt < n)
        cnt = rp->rio_cnt;
    memcpy(usrbuf, rp->rio_bufptr, cnt);
    rp->rio_bufptr += cnt;
    rp->rio_cnt -= cnt;
    return cnt;
}
```

- [ ] 

可以看出来，如果文件大小比缓冲区大的话，即便你要读的字节数小于文件大小，也不能读满想要的字节数。



## 文件元数据

调用 `stat` 和 `fstat` 函数获得文件的基本信息。

```c
#include <unistd.h>
#include <sys/stat.h>

int stat(const char *filename, struct stat *buf);
int fstat(int fd, struct stat *buf); // 返回：若成功则为 0，若出错则为 -1。
```

元信息被存储到 `buf`中，结构如下：

```c
/* Metadata returned by the stat and fstat functions */
struct stat {
    dev_t         st_dev;      /* Device */
    ino_t         st_ino;      /* inode */
    mode_t        st_mode;     /* Protection and file type */
    nlink_t       st_nlink;    /* Number of hard links */
    uid_t         st_uid;      /* User ID of owner */
    gid_t         st_gid;      /* Group ID of owner */
    dev_t         st_rdev;     /* Device type (if inode device) */
    off_t         st_size;     /* Total size, in bytes */
    unsigned long st_blksize;  /* Block size for filesystem I/O */
    unsigned long st_blocks;   /* Number of blocks allocated */
    time_t        st_atime;    /* Time of last access */
    time_t        st_mtime;    /* Time of last modification */
    time_t        st_ctime;    /* Time of last change */
};
```

## 目录

调用`opendir`打开目录，返回指向**目录流**（directory stream）的指针。

```c
#include <sys/types.h>
#include <dirent.h>

DIR *opendir(const char *name); // 返回：若成功，则为处理的指针；若出错，则为 NULL。
```

用`readdir`来读目录。

```c
#include <dirent.h>

struct dirent *readdir(DIR *dirp);
// 返回：若成功，则为指向下一个目录项的指针；若没有更多的目录项或出错，则为 NULL。
```

`dirent` 的结构如下。读并不是递归的，可以结合`stat`知道更详细的情况。

```c
struct dirent {
    ino_t d_ino;      /* inode number */
    char d_name[256]; /* Filename */
};
```

## I/O重定向

`dup2` 函数复制描述符表表项 `oldfd` 到描述符表表项 `newfd`，覆盖描述符表表项 `newfd` 以前的内容。如果 `newfd` 已经打开了，`dup2` 会在复制 `oldfd` 之前关闭 `newfd`。

```c
#include <unistd.h>

int dup2(int oldfd, int newfd); // 返回：若成功则为非负的描述符，若出错则为 -1。
```

说的更直白一点就是，现在有一个已经打开的文件描述符 `oldfd`，把它在描述符表中的指针，复制给`newfd`，使得`newfd`也指向同一个文件表。倘若之前`newfd`已经存在的话，则先关闭再新建。




