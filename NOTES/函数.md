# fcntl() 文件控制函数（file control）
[Reference](https://www.cnblogs.com/xuyh/p/3273082.html)

>`int fcntl(int fd, int cmd);`

>`int fcntl(int fd, int cmd, long arg);`

>`int fcntl(int fd, int cmd, struct flock *lock);`

  
fcntl()函数具有5种功能：
* 复制一个现有的描述符(cmd = F_DUPFD)
* 获得/设置文件描述符标记(cmd = F_GETFD / F_SETFD)
* 获得/设置文件状态标记(cmd = F_GETFL / F_SETFL)
* 获得/设置异步I/O所有权(cmd = F_GETOWN / F_SETOWN)
* 获得/设置记录锁(cmd = F_GETLK, F_SETLK / F_SETLKW)

失败返回-1，成功的返回值与各cmd有关


# ulimit -a  可用于查看各限制信息（命令行输入）

# fpathconf() 和 pathconf() 用于获取文件的各项配置信息（详见 man 手册）

# exec函数族 (一般用于执行某个程序)
* 让父子进程执行不相干的操作
* 能够替换进程地址空间中的源代码段
* 当前程序中调用另一个应用程序
* 一般在使用exec函数族之前先创建子进程，在子进程中使用exec

常用的函数：
1. execl  
> 函数原型 `int execl(const char *path, const char *arg, ...);`
* path表明要执行的程序的绝对路径
* 第一个arg用于占位，不可为空，一般将其写为待执行程序名
* 后面的变参arg用于给带执行程序提供参数，写完参数后最后添加 NULL 作为结束标志
* 函数执行成不返回，执行失败则返回 -1

eg  : `execl("/bin/ls", "ls", "-la", NULL);`

2. execlp
> 函数原型 `int execlp(const char *file, const char *arg, ...)`
* 从环境变量中查找 file 程序并执行，一般用于调用系统自带的应用程序，使用方法同上execl


# wait() 与 waitpid()  回收进程
* 孤儿进程 与 僵尸进程
  * 当父进程先于子进程结束，此时子进程变为孤儿进程，最终会被系统的特定进程收养并释放（取决于操作系统）
  
  * 在子进程运行结束之后，父进程不对其进行处理，则该子进程变为僵尸进程，其内存资源已被释放，但是PCB资源未被释放，极端情况下，若僵尸进程过多会导致系统PCB资源爆满，产生异常。

## wait()
>函数原型  `pid_t wait(int *status);`

* wait函数是阻塞的，直至有子进程结束，对其进行回收（先来先回收）
* 调用一次wait函数，只能回收一个进程，若需要回收多个进程，则需要使用多个wait函数
* 成功返回被回收的子进程pid，失败返回 -1
* status用于接收回收子进程的退出状态 `wait(&status);`，若用户不关心其退出状态则填NULL `wait(NULL);`

## waitpid()
>函数原型   `pid_t wait(pid_t pid, int *status, int options);`
* pid参数用于指定回收的目标子进程
  * pid > 0     等待其PID等于此pid的进程 
  * pid == 0    等待与调用进程同组ID的任一子进程  
  * pid == -1   等待任一子进程（此时与wait等效）
  * pid < 0     等待组ID与pid绝对值相同的任一子进程
* 通过options参数可以设置其是否阻塞（ 为WNOHANG 不阻塞，为 0 阻塞 ）


## 关于退出状态
* 正常退出：
  * `WIFEXITED(status)`     正常退出返回非0，否则为假
  * `WEXITSTATUS(status)`   返回退出状态的参数
* 非正常退出
  * `WIFSIGNALED(status)`   为真表示非正常退出
  * `WTERNSIG(status)`      返回使进程终止的信号的编号
  