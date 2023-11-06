## 应用编程

### 信号

~~~sh
root@lubancat:~# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
root@lubancat:~# 
~~~

signal

### 管道

#### 匿名管道

pipe

#### 命名管道

fifo

### system-V IPC通讯

~~~
# 查询系统当前的IPC对象
ipcs

# 以下是示例输出，没有使用的情况下可能为空
--------- 消息队列 -----------
键        msqid      拥有者  权限     已用字节数 消息
0x000004d2 98345      flyleaf    666        0            0

------------ 共享内存段 --------------
键        shmid      拥有者  权限     字节     连接数  状态

--------- 信号量数组 -----------
键        semid      拥有者  权限     nsems
~~~

#### 消息队列

msgget、msgsnd、msgrcv、msgctl

#### 共享内存

shmget、shmat、shmdt、shmctl

#### 信号量

semget、semop、semctl

### 进程

fork()

### 套接字

socket

### POSIX 可移植操作系统接口

POSIX标准中各种并发和通信机制的初始化函数。这些机制在多线程和进程间通信编程中非常重要。

POSIX threads (pthreads) 是一个用于 UNIX-like 系统的线程库。

#### 互斥锁 (Mutex)

- 初始化: `pthread_mutex_init()`
- 销毁: `pthread_mutex_destroy()`
- 加锁: `pthread_mutex_lock()`
- 尝试加锁: `pthread_mutex_trylock()`
- 解锁: `pthread_mutex_unlock()`

#### 线程 (Threads)

- 创建线程: `pthread_create()`
- 退出线程: `pthread_exit()`
- 等待线程: `pthread_join()`
- 获取线程ID: `pthread_self()`

POSIX消息队列、信号量、共享内存并不是pthreads库的一部分，但是是POSIX标准的一部分。

#### 消息队列 (Message Queues)

- 创建或打开: `mq_open()`
- 关闭: `mq_close()`
- 发送消息: `mq_send()`
- 接收消息: `mq_receive()`
- 删除消息队列: `mq_unlink()`

#### 信号量 (Semaphores)

- 初始化: `sem_init()`
- 销毁: `sem_destroy()`
- 等待: `sem_wait()` 或 `sem_trywait()`
- 释放: `sem_post()`

#### 共享内存 (Shared Memory)

- 打开: `shm_open()`
- 关闭: `shm_unlink()`
- 映射: `mmap()`
- 取消映射: `munmap()





### gdb调试



### c标准库/系统调用
