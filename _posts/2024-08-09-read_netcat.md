---
title: 'gnu-netcat执行部分原理分析(1)'
date: 2024-08-09
permalink: /posts/2024/08/read_netcat/
excerpt: '以gnu-netcat为案例分析VY-netcat将进程重定向到socket...'
tags:
  - netcat
  - gnu
  - c/cpp
---

## 前言

最近**VY-netcat**遭遇开发瓶颈, 还是打算回去看看其他的程序是怎么实现这些功能的, 其中指令执行需要参考的是**gnu-netcat**.

有一说一好久没啃大项目了, 最近看着感觉还挺麻烦的, 还好**gnu-netcat**编写非常规范, 从0开始也能学到很多知识, 还是挺推荐开发者读一读源码, 附上源码地址: [The GNU netcat - Browse /netcat/0.7.1 at SourceForge.net](https://sourceforge.net/projects/netcat/files/netcat/0.7.1/)

## 分析

### 文件结构

项目结构就不详细介绍了, 很明显代码主要来自于`./src`文件夹下, 这里可以简单介绍一下**VY-netcat**的文件结构:

- VY-netcat
  
  - bin: 存放编译文件路径
  
  - LICENSE: VY开源许可证
  
  - makefile: 用于linux/mac系统进行`make`的脚本, 想要理解编译原理可以从这个文件入手
  
  - README.md: markdown格式说明书
  
  - test: 用于存放测试文件的文件夹
    
    - execl: 用于测试c语言中的execl指令.
    
    - ...
  
  - image: 图片文件夹, 用于存放相关图片
  
  - make.bat: 用于在windows上编译文件的执行脚本
  
  - src: 用于存放主要源码
    
    - client: 用于存放socket连接的函数模组
    
    - cmd: 用于存放终端交互的参数模组
    
    - log: 用于编写日志
    
    - netcat.v: 主要编译文件
    
    - netcat.c: 使用**netcat.v**进行编译的中间文件, 可使用gcc直接编译为二进制文件

**VY-netcat**在一些编写上参考了**gnu-netcat**, 可以此作为基础进行学习, 本次要分析`-e, --exec=PROGRAM         program to exec after connect`参数的原理, 所以直接查看`./src/netcat.c`文件.

### 设置参数

直接定位到`-e`部分, 源码写得很清晰, 找到`./src/netcat.c`下`main()`函数:

```c
/* 如果根本没有给出 args，则从 stdin 中获取它们并生成 argv */
/* Cmd line: */
if (argc == 1)
    netcat_commandline_read(&argc, &argv);

  /* 检查命令行开关 */
while (TRUE) {
    int option_index = 0;
    static const struct option long_options[] = {
    { "close",    no_argument,        NULL, 'c' },
    { "debug",    no_argument,        NULL, 'd' },
    { "exec",    required_argument,    NULL, 'e' },
    { "gateway",    required_argument,    NULL, 'g' },
    { "pointer",    required_argument,    NULL, 'G' },
    { "help",    no_argument,        NULL, 'h' },
    { "interval",    required_argument,    NULL, 'i' },
    { "listen",    no_argument,        NULL, 'l' },
    { "tunnel",    required_argument,    NULL, 'L' },
    { "dont-resolve", no_argument,        NULL, 'n' },
    { "output",    required_argument,    NULL, 'o' },
    { "local-port",    required_argument,    NULL, 'p' },
    { "tunnel-port", required_argument,    NULL, 'P' },
    { "randomize",    no_argument,        NULL, 'r' },
    { "source",    required_argument,    NULL, 's' },
    { "tunnel-source", required_argument,    NULL, 'S' },
#ifndef USE_OLD_COMPAT
    { "tcp",    no_argument,        NULL, 't' },
    { "telnet",    no_argument,        NULL, 'T' },
#else
    { "tcp",    no_argument,        NULL, 1 },
    { "telnet",    no_argument,        NULL, 't' },
#endif
    { "udp",    no_argument,        NULL, 'u' },
    { "verbose",    no_argument,        NULL, 'v' },
    { "version",    no_argument,        NULL, 'V' },
    { "hexdump",    no_argument,        NULL, 'x' },
    { "wait",    required_argument,    NULL, 'w' },
    { "zero",    no_argument,        NULL, 'z' },
    { 0, 0, 0, 0 }
    };
```

尝试直接运行nc, 出现`Cmd line:`以补充参数, 这里在[VY-netcat v0.0.3版本](https://gitee.com/cryingn/vy-netcat/releases/tag/v0.0.3)更新情况中有解释过原因:

> 当windows用户直接打开程序时会因为无法传入参数导致闪退.

然后是下面定义**gnu-netcat**的参数, **VY-netcat**在设计时也有进行参考:

```v
if args.len == 1 {
            mut data := args[0] + ' ' 
            data += read_line('Cmd line:') or { '' }
            args = data.split(' ')
    }

    long_options := [
        CmdOption{
            abbr: '-h'
            full: '--help'
            vari: ''
            defa: ''
            desc: 'display this help and exit.'
        }
        CmdOption{
            abbr: '-e'
            full: '--exec'
            vari: '[shell]'
            defa: 'false'
            desc: 'program to exec after connect.'
        }
        CmdOption{
            abbr: '-lp'
            full: '--listen_port'
            vari: '[int]'
            defa: 'false'
            desc: 'listen the local port number.'
        }
        CmdOption{
            abbr: '-klp'
            full: '--keep_listen_port'
            vari: '[int]'
            defa: 'false'
            desc: 'keep to listen the local port number.'
        }
    ]
```

### 控制

继续往下, **gnu-netcat**使用switch对参数进行控制:

```c
c = getopt_long(argc, argv, "cde:g:G:hi:lL:no:p:P:rs:S:tTuvVxw:z",
            long_options, &option_index);
    if (c == -1)
      break;
    switch (c) {
        ...
        case 'e':            /* exec执行参数 */
        if (opt_exec)
        // 错误流 | 退出 | 报错日志(当e出现多次)
        ncprint(NCPRINT_ERROR | NCPRINT_EXIT,
            _("Cannot specify `-e' option double"));
        // 执行并返还.
        opt_exec = strdup(optarg);
        break;
        ...
    }
```

其中明显可以猜测`ncprint()`主要对nc过程中的日志进行打印, `strdup()`则是返回指向空终止字节字符串的指针, 故接下来可以选择调试或追踪`opt_exec`.

```bash
./netcat.c:61:char *opt_exec = NULL;            /*连接后执行程序*/
./netcat.c:122:  if ((p = strrchr(opt_exec, '/')))
./netcat.c:125:    p = opt_exec;
./netcat.c:129:  execl("/bin/sh", p, "-c", opt_exec, NULL);
./netcat.c:131:  execl(opt_exec, p, NULL);
./netcat.c:135:   opt_exec, strerror(errno));
./netcat.c:237:      if (opt_exec)
./netcat.c:242:      opt_exec = strdup(optarg);
./netcat.c:366:  if (opt_zero && opt_exec)
./netcat.c:504:      if (opt_exec) {
./netcat.c:604:      if (opt_exec) {
```

### dup2

开始之前最好顺便简单了解一下`execl`上面的代码:

```c
/* 将子进程进行重定向到socket */
dup2(ncsock->fd, STDIN_FILENO);    /* fiddlage 的精确顺序 */
close(ncsock->fd);            /* 显然是至关重要的;这是*/
dup2(STDIN_FILENO, STDOUT_FILENO);    /* swiped directly out of "inetd". */
dup2(STDIN_FILENO, STDERR_FILENO);    /* also duplicate the stderr channel */
```

通过解释, 将子进程重新定向到了socket, 也就是说在`nc -e [执行代码]`时我们实际新起了一个进程, 再将输入与输出通过socket发送到远程.

举个比较基本的重定向例子:

```bash
[root_cn@archlinux ~]$ neofetch
                   -`
                  .o+`
                 `ooo/
                `+oooo:
               `+oooooo:
               -+oooooo+:                root_cn@archlinux
             `/:-:++oooo+:               -----------------
            `/++++/+++++++:              OS: Arch Linux on Windows 10 x86_6 
           `/++++++++++++++:             Kernel: 4.4.0-19041-Microsoft
          `/+++ooooooooooooo/`           Uptime: 1 day, 6 hours
         ./ooosssso++osssssso+`          Packages: 602 (pacman)
        .oossssso-````/ossssss+`         Shell: bash 5.2.26
       -osssssso.      :ssssssso.        Terminal: Windows Terminal
      :osssssss/        osssso+++.       CPU: 11th Gen Intel i5-11300H (8)
     /ossssssss/        +ssssooo/-       Memory: 6593MiB / 16167MiB
   `/ossssso+/:-        -:/+osssso+-
  `+sso+:-`                 `.-/+oso:
 `++:.                           `-/+/
 .`                                 `/






[root_cn@archlinux ~]$ neofetch > a
[root_cn@archlinux ~]$ cat a
                   -`
                  .o+`
                 `ooo/
                `+oooo:
               `+oooooo:
               -+oooooo+:                root_cn@archlinux
             `/:-:++oooo+:               -----------------
            `/++++/+++++++:              OS: Arch Linux on Windows 10 x86_6 
           `/++++++++++++++:             Kernel: 4.4.0-19041-Microsoft
          `/+++ooooooooooooo/`           Uptime: 1 day, 6 hours
         ./ooosssso++osssssso+`          Packages: 602 (pacman)
        .oossssso-````/ossssss+`         Shell: bash 5.2.26
       -osssssso.      :ssssssso.        Terminal: Windows Terminal
      :osssssss/        osssso+++.       CPU: 11th Gen Intel i5-11300H (8)
     /ossssssss/        +ssssooo/-       Memory: 6593MiB / 16167MiB
   `/ossssso+/:-        -:/+osssso+-
  `+sso+:-`                 `.-/+oso:
 `++:.                           `-/+/
 .`                                 `/
```

### execl执行

通过追踪`opt_exec`得到以上信息:

* 第61行定义opt_exec

```c
/* 执行一个外部文件，使其 stdin/stdout/stderr 成为实际的socket */

static void ncexec(nc_sock_t *ncsock)
{
  int saved_stderr;
  char *p;
  assert(ncsock && (ncsock->fd >= 0));

  /* 保存 stderr fd，因为我们以后可能需要它 */
  saved_stderr = dup(STDERR_FILENO);

  /* 将子进程进行重定向到socket */
  dup2(ncsock->fd, STDIN_FILENO);    /* fiddlage 的精确顺序 */
  close(ncsock->fd);            /* 显然是至关重要的;这是*/
  dup2(STDIN_FILENO, STDOUT_FILENO);    /* swiped directly out of "inetd". */
  dup2(STDIN_FILENO, STDERR_FILENO);    /* also duplicate the stderr channel */

  /*更改已执行程序的标签*/
  if ((p = strrchr(opt_exec, '/')))
    p++;            /*较短的argv[0]*/
  else
    p = opt_exec;

  /* 用新的过程替换此过程 */
#ifndef USE_OLD_COMPAT
  execl("/bin/sh", p, "-c", opt_exec, NULL);
#else
  execl(opt_exec, p, NULL);
#endif
  dup2(saved_stderr, STDERR_FILENO);
  ncprint(NCPRINT_ERROR | NCPRINT_EXIT, _("Couldn't execute %s: %s"),
      opt_exec, strerror(errno));
}                /* 结束ncexec() */
```

 `static`表示函数为静态函数.

> 静态函数的作用是实现与类相关的功能，而不需要创建实例。静态全局变量仅对当前文件可见，其他文件不可访问，其他文件可以定义与其同名的变量，两者互不影响.

execl语法为:

```c
int execl(const char *path, const char *arg, ...);
```

可以看到p作为调用函数的`argv[0]`参数, 有两种执行方式: 

* 在`/bin/sh`下传参为`p -c opt_exec NULL`

* 在`opt_exec`下传参为`p NULL`

#### 示例

我们可以自己构造一些简单代码以了解execl函数:

编译**hello.c**文件

```c
// clang hello.c -o hello
#include <stdio.h>
int main(int argc,char *argv[],char *envp[]){
        printf("Filename: %s\n",argv[0]);
        printf("%s %s\n",argv[1],argv[2]);
        return 0;
}
```

然后编译**return.c**文件:

```c
// clang return.c -o return
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
        char *temp,*temp1,*temp2;
        temp="test";  //Filename
        temp1="Funny";
        temp2="world";

        execl("hello",temp,temp1,temp2,NULL);
        printf("Error");
        return 0;
}a
/* return:
 * Filename: test
 * Funny world
**/
```

可以发现本质上`execl`添加了一个新的进程进行工作(我觉得这么说很容易理解, 有问题欢迎指出).

#### 分析结论

仅从一个静态函数`ncexec()`能分析到的信息有限, 不过我们从`execl()`方法大致可以思考**gnu-netcat**的执行原理: 从外部调用脚本进行执行, 然后重定向到当前连接的远程系统中.

之后还需要考虑重定向在vlang上的使用问题, 对于**gnu-netcat**上执行方式与linux原理也需要更进一步学习, 有时间的话继续写写.