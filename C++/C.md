# c语言

[TOC]

GNU C库，又称glibc是c的编译程序

# 可执行文件的生成

(以下括号内均为windows系统)

编译一次只能编译一个.cpp文件，先编译生成.o文件（.obj）。lib,dll,exe都算是最终的目标文件，是最终产物。lib和dll的命名规划都是以lib开头

**文件：**

**.o编译文（.obj）**：编译后的文件，相当于一个中间文件,.a和.so都由他生成

**.a静态库（.lib）**:将若干.o文件打包,静态库在程序编译时会被连接到目标代码中，程序运行时将不再需要该静态库。

**.ko驱动模块文件(）**：该文件的意义就是把内核的一些功能移动到内核外边， 需要的时候插入内核，不需要时卸载，在2.6版本后出现，在原本.o文件的基础上增加了一些信息，可以当成一种.o文件

**.so动态链接库(.dll）**：用于动态连接的

**生成：**

## 调用静态库得到可执行文件

```c
程序1: hello.h
#ifndef HELLO_H
#define HELLO_H
void hello(const char *name);
#endif //HELLO_H

程序2: hello.c
#include <stdio.h>
void hello(const char *name){
printf("Hello %s!\\n", name);
}

程序3: main.c
#include "hello.h"
int main(){
hello("everyone");
return 0;
}
```

1.编译生成.o文件：`gcc -c hello.c`    #得到hello.o

2.由.o文件创建静态库：`ar -cr libmyhello.a hello.o`    #得到libmyhello.a

3.调用静态库得到可执行文件：`gcc main.c libmyhello.a -o hello` 或`gcc -o hello main.o libmyhello.a`  或`gcc -o hello main.c -L. -lmyhello`#得到可执行文件hello

## 调用动态库得到可执行文件

1.编译生成.o文件：`gcc -c hello.c`    #得到hello.o

2.由.o文件创建动态库：`gcc -shared -fPIC -o libmyhello.so hello.o`

3.生成可执行文件：`gcc -o hello main.c -L. -lmyhello` (如果不用-L将在/usr/lib下查找链接库)

```
         此时执行hello时会报错，需要将`libmyhello.so` 文件放入/usr/lib路径下
```

或者用`gcc -o hello main.c libmyhello.so` ,[可以动态链接当前路径的.so](http://xn--onqw6f1h48bw04apia53ab7sm19b26yc2hq.so)

4.此时如果删掉.so文件 再执行hello理所应当的会报错

当静态库和动态库重名时优先用动态库

# 不需要头文件

## int main()

**`int** main(void)`

**`int** main(**int** argc)`//合法，但不常用

**`int** main(**int** argc, **char** *argv)`

**`int** main(**int** argc, **char** **argv)`

**`int** main(**int** argc, **char** *argv[],**char** *env[])`

argc:整数，为传给main()的命令行参数个数。

argv:字符串数组。

env:字符串数组。env[]的每一个元素都包含ENVVAR=value形式的字符串。其中ENVVAR为环境变量如PATH或87。value为ENVVAR的对应值如C:\DOS，C:\TURBOC(对于PATH)或YES(对于87)。

//一旦想说明这些参数，则必须按argc，argv， env的顺序

# #include <stdio.h>

- **int** fprintf (**FILE*** stream, **const char*** format, [argument])

  - **stream**-- 这是指向 FILE 对象的指针，该 FILE 对象标识了流。

    <aside> 🌟 stdout – 标准输出设备 stdout。

    stderr – 标准错误输出设备

    两者默认向屏幕输出。

    stdout是行缓冲的，输出放在buffer里，只到换行时才输出到屏幕

    stderr是无缓冲的，会直接输出。

    - 满缓冲、行缓冲、无缓冲

      ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/57cf0a2f-b07c-4fb5-935a-97846b23f398/Untitled.png)

    </aside>

  - **format**- 这是 C 字符串，包含了要被写入到流 stream 中的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。format 标签属性是**%[flags][width][.precision][length]specifier**

  - [argument]：附加参数列表 [3]

- FILE * popen( const char * command,const char * type);

  popen()用创建管道的方式调用fork()产生子进程，然后从子进程中调用/bin/sh -c来执行参数command的指令。

  若成功返回文件指针，否则返回NULL，错误存放errno（EINVAL参数type不合法）中

  参数type可使用“r”代表读取，“w”代表写入。依照此type值，popen()会建立管道连到子进程的标准输出设备或标准输入设备，然后返回一个文件指针。

  //在编写具SUID/SGID权限的程序时请尽量避免使用popen()，popen()会继承环境变量，通过环境变量可能会造成系统安全的问题。

  popen打开的属于管道，需要用pclose关闭，而不是fclose关闭

  - 例子

    ![ytpe为W](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5a33add-4ae7-4a6f-9c03-640104b481fb/Untitled.png)

    ytpe为W

    ![type为r](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/234e1c5f-da6a-4f24-b20f-2740f23bb746/Untitled.png)

    type为r

- int pclose(FILE * stream)

  关闭管道指针

- int fclose( FILE *fp )

  fclose 返回 0，否则返回EOF（-1）

  可以把 **缓冲区**内最后剩余的数据输出到**内核缓冲区**，并**释放** [文件指针](http://baike.baidu.com/view/5019859.htm)和有关的缓冲区

  close 对应 open

  fclose 对应 fopen

- FILE *fopen(const char *filename, const char *mode);

  ```jsx
  mode 打开模式：
  1. r 只读方式打开一个文本文件
  2. rb 只读方式打开一个二进制文件
  3. w 只写方式打开一个文本文件
  4. wb 只写方式打开一个二进制文件
  5. a 追加方式打开一个文本文件
  6. ab 追加方式打开一个二进制文件
  7. r+ 可读可写方式打开一个文本文件
  8. rb+ 可读可写方式打开一个二进制文件
  9. w+ 可读可写方式创建一个文本文件
  10. wb+ 可读可写方式生成一个二进制文件
  11. a+ 可读可写追加方式打开一个文本文件
  12. ab+ 可读可写方式追加一个二进制文件
  ```

- 读函数

  ```c
     int fgetc(FILE *stream);
  	 //读写指针后移一位，返回读取结果转为int
  	 char *fgets(char *s, int size, FILE *stream);
  
     int getc(FILE *stream);
  
     int getchar(void);
  
     int ungetc(int c, FILE *stream);
  ```

- 写函数

  ```c
     int fputc(int c, FILE *stream);
  	 //覆盖写，还是追加写取决于前面打开文件的方式
  	 //写后指针后移，可以继续写
  	 int fputs(const char *s, FILE *stream);
  
     int putc(int c, FILE *stream);
  
     int putchar(int c);
  
     int puts(const char *s);
  ```

- scanf

  返回值：返回输入参数的个数，如果输入不匹配返回0，如果输入结束返回EOF（-1）

- printf

  返回输出的字符数字

  ***\*printf(“1234\n”)的返回值是5\****

- void perror(const char* s);

  **功能**：输出函数调用失败的错误信息

  **参数**：s：打印错误信息的提示消息

- int snprintf ( char * str, size_t size, const char * format, ... );

  `snprintf(buffer, 6, "%s**\\n**", s);`

  将字符串s的前6个字符以`"%s**\\n**"` 格式输入buffer中

  - **str** -- 目标字符串，用于存储格式化后的字符串的字符数组的指针。
  - **size** -- 字符数组的大小。
  - **format** -- 格式化字符串。
  - **...** -- 可变参数，可变数量的参数根据 format 中的格式化指令进行格式化。

  返回值是输出到str的字符数，不包括\0

  会添加一个空字符，但不会算入返回值的长度里

- int sprintf(char *str, const char *format, ...)

  将可变参数的内容以format的格式输出到str

- int vsprintf(char *str, const char *format, va_list arg)

  将文本写入str中

  第一个参数为要写入的空间

  第二个参数为格式化文本， 要写入的数据

  第三个参数是用于替换格式化文本内的信息

  \`````vsprintf(buffer, "``````%d %f %s", aptr);`

- int vprintf(const char *format, va_list arg)

  将arg里的内容以format格式输出到标准输出

# #include<unistd.h>

- int access(const char *pathname,int mode)

  测试路径pathname是否2有mode权限，成功返回0，失败返回-1

  mode：

  - *pathname:表示要测试的文件的路径*
  - *mode:表示测试的模式可能的值有:*
  - *R_OK:是否具有读权限*
  - *W_OK:是否具有可写权限*
  - *X_OK:是否具有可执行权限*
  - F_OK：文件是否存在

# #include <stdlib.h>

- int system(const char * string);

  执行参数里的命令行语句

- void exit(int status);

  参数：status传递进程结束时的状态，exit(0) 表示程序正常退出，exit(1) 或者 exit(-1) 表示程序异常退出。

  ```c
  //头文件 stdlib.h 
  *#define EXIT_FAILURE    1       /* Failing exit status.  */
  #define EXIT_SUCCESS    0       /* Successful exit status.  */
  ```

  正常退出：exit(EXIT_SUCCESS)

  异常退出：exit(EXIT_FAILURE)

  //在实际编程中，可以用 wait() 系统调用来接收 子进程 的返回值，从而根据不同的返回值进行不同的处理。

  //`return`：函数结束，是**函数级别**的结束，而 `exit()` 和 `_eixt()` 是**系统级别**的结束。

  //有些数据认为已被写入文件，实际上还在缓冲区，这时用`_exit()`函数直接将进程关闭掉，**缓冲区中的数据就会丢失**。用`exit()`函数会处理这个缓冲。

  //如果 main() 在一个递归程序中，**exit()** 仍然会**终止程序**,**return** 返回递归的**前一级**。另外在 main() 之外的函数中调用 exit()，也将终止整个进程。

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/feeb5bfb-a50a-4f11-ad17-c7c8344ec2a2/Untitled.png)

- int atexit(void (*func) (void))

  返回值：成功返回0，出错返回1

  参数：一个无返回值无参数的函数指针

  功能：用atexit函数来登记这些终止处理程序 (exithandler)，当这个程序执行exit时会倒序调用atexit函数登记的这些函数，如果登记了两次也会调用两次

  意义：用于记录进程的执行状态，统计系统运行时的一些参数便于调试

  //A N S I C 的规定，一个进程可以登记多至 3 2 个函数

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e39e0c0b-1cad-40e2-a82d-d378ee27e4fc/Untitled.png)

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07869188-8ee9-4168-aaae-4048ff2581df/Untitled.png)

- void *calloc(size_t nitems, size_t size)

  分配所需的内存空间，并返回一个指向它的指针

  比起malloc，它会设置内存为0

# #include<sys/resource.h>

- int  getrlimit ( int  resource,  struct  rlimit  *rlptr );

  头文件: #include<sys/time.h>与#include<sys/resource.h>

  返回值：成功0，失败-1

  功能：获取resource这种资源的个数信息

- int  setrlimit ( int resource,  const struct  rlimit  *rlptr );

  头文件#include<sys/time.h>与#include<sys/resource.h>

  功能：修改resource这种资源的个数信息（在rlptr指向的配置文件里）

- 这里的resource时一种宏，包括

  RLIMIT_AS //进程的最大虚内存空间，字节为单位。

  RLIMIT_CORE //内核转存文件的最大长度。

  RLIMIT_CPU //最大允许的CPU使用时间，秒为单位。当进程达到软限制，内核将给其发送SIGXCPU信号，这一信号的默认行为是终止进程的执行。然而，可以捕捉信号，处理句柄可将控制返回给主程序。如果进程继续耗费CPU时间，核心会以每秒一次的频率给其发送SIGXCPU信号，直到达到硬限制，那时将给进程发送 SIGKILL信号终止其执行。

  RLIMIT_DATA //进程数据段的最大值。

  RLIMIT_FSIZE //进程可建立的文件的最大长度。如果进程试图超出这一限制时，核心会给其发送SIGXFSZ信号，默认情况下将终止进程的执行。

  RLIMIT_LOCKS //进程可建立的锁和租赁的最大值。

  RLIMIT_MEMLOCK //进程可锁定在内存中的最大数据量，字节为单位。RLIMIT_MSGQUEUE //进程可为POSIX消息队列分配的最大字节数。

  RLIMIT_NICE //进程可通过setpriority() 或 nice()调用设置的最大完美值。

  RLIMIT_NOFILE //指定比进程可打开的最大文件描述词大一的值，超出此值，将会产生EMFILE错误。

  RLIMIT_NPROC //用户可拥有的最大进程数。

  RLIMIT_RTPRIO //进程可通过sched_setscheduler 和 sched_setparam设置的最大实时优先级。

  RLIMIT_SIGPENDING //用户可拥有的最大挂起信号数。

  RLIMIT_STACK //最大的进程堆栈，以字节为单位。

  rlim：描述资源软硬限制的结构体，原型如下

  `struct rlimit {   　　rlim_t rlim_cur;   　　rlim_t rlim_max;   };`

# #include<pthread.h>

valgvinel调试工具

- int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(start_rtn)(void *), void *arg);

  返回值：成功为0，出错为>0的EXXX值

  //当一个程序由exec启动时，系统会创建一个初始线程或主线程的单个线程。额外线程由上述函数创建；

  - 参数：

    thread，表示指向线程标识符的指针

    attr用来设置线程属性包括：优先级、初始栈大小、是否应该是守护线程等等。可为NULL

    start_rtn参数，表示线程运行函数的地址

    arg（必须是静态函数）表示线程运行函数的参数

  - 例子

    ```c
    #include <stdio.h>
    #include <pthread.h>
    #include <unistd.h>
    #include <string.h>
    #include <errno.h>
    
    int* abc(){
    	fprintf(stderr, "abc start\\n");
    	return NULL;
    }
    int main()
    {
     	pthread_t tid;
    	fprintf(stderr, "main start\\n");
    	pthread_create(&tid, NULL, abc, NULL);
    	sleep(1);//这里如果不睡眠，直接return会导致线程堆栈来不及打印，线程没开始进程结束了
    	fprintf(stderr, "main end\\n");
    	return 0;
    }
    //因为线程不是默认库，编译需要用gcc 123.c -o 123 -lpthread
    //输出：
    //main start
    //abc start
    //main end
    ```

//一个线程可以隐式的退出（线程的例程结束），也可以显式的调用pthread_exit函数来退出。

- void pthread_exit(void *retval);

  参数：retval表示线程退出状态,如果有pthread_join将传给它的retval参数

  //pthread_exit或者return返回的指针所指向的内存单元必须是全局的或者是用malloc分配的，不能在线程函数的**栈**上分配，因为当其它线程得到这个返回指针时线程函数已经退出了，栈空间就会被回收。

  //在线程中禁止调用exit函数，否则会导致整个进程退出

  //在主线程中使用pthread_exit函数，会导致子线程成为僵尸进程。需要用pthread_join函数

- int pthread_join(pthread_t thread, void **retval);

  当前线程阻塞等待thread线程退出，获取线程退出状态。其作用，对应进程中的waitpid() 函数。

  返回值：成功：0；失败：错误号

  参数：thread：线程ID

  retval：存储线程结束状态，整个指针和pthread_exit的参数是同一块内存地址。

  - 例子

    ```c
    #include <stdio.h>
    #include <pthread.h>
    #include <unistd.h>
    #include <string.h>
    #include <errno.h>
    struct stu{
    	int num;
    	char * str1;
    	char * str2;
    };
    
    void *mythread1(void *arg)
    {
    	fprintf(stderr, "mythread start\\n");
    	char str1[] = "dasi";
    	char *str2 = (char*)malloc(3);
    	str2[0] = 'm';
    	str2[1] = 'a';
    	str2[2] = '\\0';
    	struct stu *stu = (struct stu*)malloc(sizeof(struct stu));
    	stu->num = 2;
    	stu->str1 = str1;
    	stu->str2 = str2;
    	printf("child thread, pid==[%d], id==[%ld]\\n", getpid(), pthread_self());
    	pthread_exit(stu);
    }//用指针的方式
    struct stu stuall;
    void *mythread2(void *arg)
    {
    	fprintf(stderr, "mythread start\\n");
    	char str1[] = "dasi";
    	char *str2 = (char*)malloc(3);
    	str2[0] = 'm';
    	str2[1] = 'a';
    	str2[2] = '\\0';
    	stuall.num = 2;
    	stuall.str1 = str1;
    	stuall.str2 = str2;
    	printf("child thread, pid==[%d], id==[%ld]\\n", getpid(), pthread_self());
    	pthread_exit(&stuall);
    }//用全局变量的方式
    int main()
    {
     	pthread_t tid1, tid2;
    	fprintf(stderr, "main start\\n");
    	if(pthread_create(&tid1, NULL, mythread1, NULL) != 0){
    		 printf("pthread_create error, [%s]\\n");
                    	 return -1;
    	}
    	fprintf(stderr, "main thread, pid==[%d], id==[%ld]\\n", getpid(), pthread_self());
    	sleep(1);
    
    	if(pthread_create(&tid2, NULL, mythread2, NULL) != 0){
    		 printf("pthread_create error, [%s]\\n");
                    	 return -1;
    	}
    
    	void *p = NULL;
    	pthread_join(tid1, &p);
    	struct stu *stu1 = (struct stu*)p;
    	pthread_join(tid2, &p);
    	struct stu *stu2 = (struct stu*)p;
    	fprintf(stderr, "tid1 :%d   string have %d, str1 is %s, str2 is %s\\n", tid1, stu1->num, stu1->str1, stu1->str2);
    	fprintf(stderr, "tid2 :%d   string have %d, str1 is %s, str2 is %s\\n", tid2, stu2->num, stu2->str1, stu2->str2);
    	fprintf(stderr, "main end\\n");
    	return 0;
    }//可以看到在线程中str1用的栈区，所以在线程回收后，输出str1为NULL
    //输出
    //main start
    //main thread, pid==[7018], id==[140332969019200]
    //mythread start
    //child thread, pid==[7018], id==[140332969014848]
    //mythread start
    //child thread, pid==[7018], id==[140332889011776]
    //tid1 :-792414656   string have 2, str1 is , str2 is ma
    //tid2 :-872417728   string have 2, str1 is , str2 is ma
    //main end
    ```

- int pthread_cancel(pthread_t thread);

  一个线程可以借助 pthread_cancel() 函数向另一个线程发送“终止执行”的信号，从而令目标线程结束执行。

  成功返回0，失败返回非0数，未找到目标返回ESRCH（3）

  接收cancel信号后借宿执行的目标线程等同于执行 pthread_exit

- int pthread_detach(pthread_t thread);

- int pthread_equal(pthread_t t1, pthread_t t2);

  比较两个线程ID是否相等*。*

  不同系统线程类型不同，所以要用函数而不能==来判断

- int pthread_kill(pthread_t thread, int sig);

  成功返回0，线程不存在返回ESRCH，信号不合法返回EINVAL

  向指定ID的线程发送sig信号

  如果发送了信号，但该线程却没有signal处理函数则这个进程退出，即杀死整个进程

  当sig为0时，可以用来判断线程是否还活着

  - 例子

    ```c
    int kill_rc = pthread_kill(thread_id,0);
    if(kill_rc == ESRCH)
    printf("the specified thread did not exists or already quit\\n");
    else if(kill_rc == EINVAL)
    printf("signal is invalid\\n");
    else
    printf("the specified thread is alive\\n");
    ```

- pthread_t pthread_self(void);

  返回线程ID

  - pthread_self与getpid的区别

    ```c
    void *f(void *arg) {
     
        if(*(int *)arg){
            printf("tid 父进程所属线程 ID 为 %d\\n",gettid());
    				//tid 父进程所属线程 ID 为 22560
            printf("pid 父进程所属线程 ID 为 %ld\\n",pthread_self());
    				//pid 父进程所属线程 ID 为 140257971205888
        }
        else{
            printf("tid 子进程所属线程 ID 为 %d\\n",gettid());
    				//tid 子进程所属线程 ID 为 22561
            printf("pid 子进程所属线程 ID 为 %ld\\n",pthread_self());
    				//pid 子进程所属线程 ID 为 140257971205888
        }
        pthread_exit(NULL);
     
    }
    int main(){
        pid_t pid1;
        pthread_t tid1;
        pthread_t tid2;
        int *retval_1;
        int *retval_2;
        int a;
        pid1 = fork();
        
        if(pid1 == 0){
            a = 0;
            printf("pid 子进程主线程 ID 为 %ld\\n",pthread_self());
    				//pid 子进程主线程 ID 为 140257979672384
            printf("tid 子进程主线程 ID 为 %d\\n",gettid());
    				//tid 子进程主线程 ID 为 22559
            pthread_create(&tid1,NULL,f,(void *)&a);
        }
        else{
            a = 1;
            printf("pid 父进程主线程 ID 为 %ld\\n",pthread_self());
    				//pid 父进程主线程 ID 为 140257979672384
            printf("tid 父进程主线程 ID 为 %d\\n",gettid());
    				//tid 父进程主线程 ID 为 22558
            pthread_create(&tid2,NULL,f,(void *)&a);
        }
        pthread_join(tid1,(void*)retval_1);
        pthread_join(tid2,(void*)retval_2);
    //可以看到主进程和子进程用pthread_self()得到的值相等，gettid()得到的不同，
    //因为pthread_self()是posix描述的线程ID，并非内核真正的线程id，对于这个进程内是唯一的，不同进程可能有一样的线程ID
    //为什么需要两个不同的ID呢？因为线程库实际上由两部分组成：内核的线程支持+用户态的库支持(glibc)
    ```

互斥锁

- int pthread_mutex_init(pthread_mutex_t *mutex,const pthread_mutexattr_t *attr);

  pthread_mutex_t 锁变量名 =PTHREAD_MUTEX_INITIALIZER;

  mutex必须提前分配内存空间

  attr：设置互斥量的属性，可为NULL

  成功返回0，出错返回错误码

- int pthread_mutex_destroy( pthread_mutex_t *mutex);

  成功返回0，出错返回错误码

  互斥量在使用完毕后，必须要对互斥量进行销毁，以释放资源

- int pthread_mutex_lock(pthread_mutex_t *mutex);

  若已加锁，当前线程阻塞

- int pthread_mutex_trylock(pthread_mutex_t *mutex);

  未加锁则锁住返回0，若已加锁，则直接返回EBUSY

  意义：尝试一次加锁，没加成功则可以区做其他事，比如用一个双缓冲，如果一个缓冲上锁了，就往另一个缓冲里写。在大量数据高并发时有很大意义

- Int pthread_mutex_unlock(pthread_mutex_t *mutex);

条件变量

- int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *attr);

  pthread_cond_t 条件变量名 =PTHREAD_COND_INITIALIZER;//两种方式都可以初始化条件变量，这种是全局变量在全局区，函数初始化时创建的时局部变量为栈区

  条件变量初始化，cond为条件变量，需要提前分配 空间，attr为条件变量属性，NULL则用默认值

  成功返回0，出错返回错误编号

- int pthread_cond_signal(pthread_cond_t *cond);

  cond为条件变量，返回成功为0

  适用于单一生产者单一消费者

- int pthread_cond_broadcast(pthread_cond_t *);

  适用多生产者多消费者，向多个消费者发送信号

- int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

  使调用线程进入阻塞状态，直到条件为真

  在线程阻塞前，需要调用pthread_mutex_unlock 在线程唤醒后，需要调用pthread_mutex_lock

  - wait的内部执行

    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85388a21-b510-4d4b-8410-ec72793fc29b/Untitled.png)

- int pthread_cond_destroy(pthread_cond_t * cond);

  销毁条件变量

  cond为条件变量，需要提前分配 空间，attr为条件变量属性，NULL则用默认值

  成功返回0，出错返回错误编号

信号量

- 线程与fork的一些问题

  父进程中：

  互斥量加锁

  ……………

  fork();

  ……………

  互斥量减锁

  子进程中：

  互斥量加锁 …………… 互斥量减锁

  问题：父进程加锁后fork，子进程再加锁：此时子进程会被阻塞，因为锁已经上了。

  如果此时父进程解锁，子进程还处于阻塞状态，因为父子进程锁所在的堆区各不相关

- 

# #include<string.h>

## **void \*memmove(void \*str1, const void \*str2, size_t n)** 

 从 **str2** 复制 **n** 个字符到 **str1**

比起memcpy，能确保重叠的地址也能复制成功

##  **int memcmp(const void \*str1, const void \*str2, size_t n))**

把存储区 **str1** 和存储区 **str2** 的前 **n** 个字节进行比较

- 如果返回值 < 0，则表示 str1 小于 str2。
- 如果返回值 > 0，则表示 str1 大于 str2。
- 如果返回值 = 0，则表示 str1 等于 str2。

## char *strerror(int errnum)

\#include <errno.h>

参数为错误号

从内部数组中搜索错误号 **errnum**，并返回一个指向错误消息字符串的指针

## int strcmp(char *str1,char * str2);

功能是比较两个字符串的大小

若字符串1>字符串2，返回结果>0；

若字符串1<字符串2，返回结果<0；

若字符串1==字符串2， 返回结果==0;

## char *strstr(char *str1, const char *str2);

若str2是str1的子串，则返回str2在str1的首次出现的地址；如果str2不是str1的子串，则返回NULL。

查找字符串str2在字符串str1中的位置

## char *strtok(char *str, const char *delim)

返回str被delim分解的第一个子字符串或空指针

```cpp
/* 获取第一个子字符串 */
   token = strtok(str, s);
   
   /* 继续获取其他的子字符串 */
   while( token != NULL ) {
      printf( "%s\\n", token );
    
      token = strtok(NULL, s);
   }
```

原字符串str中的分隔符delim会被替换成’\n’

## int memcmp(const void *str1, const void *str2, size_t n)

把存储区 **str1** 和存储区 **str2** 的前 **n** 个字节进行比较

## char *strdup(const char *s);

字符串拷贝函数，一般和free成对出现

拷贝字符串s，内部调用malloc分配内存，返回指向复制字符串分配的空间，分配失败返回NULL

## **char *strcpy(char *dest, const char \*src)**

把 **src** 所指向的字符串复制到 **dest，**需要dest先分配足够的空间

## size_t strlen(const char *str)

计算字符长度，不包含空字符

# #include<stdlib.h>

malloc

# #include<time.h>

- `clock()`

  成功：返回从程序启动起累计的毫秒数。失败：返回-1。

以下是使用模板

```c
clock_t start = clock();
//程序
	clock_t finish = clock();
	printf("Computation timing = %10.4f sec\\n", (double)(finish - start) / ((clock_t)1000));
```

# #include<stdarg.h>

用于可变参数列表的变量读取

其中定义了`va_list` 相当于char*

以下均为宏定义

- `void va_start(va_list ap, last arg)`

  初始化ap变量，指向参数arg。它也会将指针后移一位，以便指向第一个可变参数的位置

- `type va_arg(va_list ap, type)`

  返回函数参数类别下一个为type的参数，（并不是往后查找直到找到type类型，而是把下一个当作type类型）会将ap指针后移

- `void va_end(va_list ap)`

  允许va_start宏的带有可变参数的函数返回，相当于清空va_list可变参数列

```cpp
#include <iostream>
#include <cstdarg>
using namespace std;

/****这里我们定义了一个可变参数列表的函数*******/
void Print (int n, ...)
{
  int i;
  //char ch;
  cout <<"输出： ";
  
  va_list vl;//创建一个参数vl，在我们看cstdarg函数原型的时候，我们可以这样理解，这里的vl
  //           是va_list型指针，至于它指向的位置则通过后面的va_start()来确定

  va_start(vl,n);//vl指向定参数n，会将指针后移一位，以便指向第一个可变参数
  for (i=0;i<n;i++)
  {
    cout <<(char)va_arg(vl,int) ;//这里使用了强制类型转换
    //也可以这样输出
    //cout << va_arg(vl,char);
   }
  va_end(vl);
  cout << "\\n" ;
}
int main ()
{
  Print(4,'z','a','q','r');
  return 0;
}
```

# 关键字

## ***\**\*attribute\*\**\***

1. 取消结构体对齐
2. **函数属性**

[**attribute** 用法_wiggens的博客-CSDN博客](https://blog.csdn.net/swj9099/article/details/96713277)

## #**define** 宏名（形参表）字符串

## extern

如果全局变量不在文件的开头定义，有效的作用范围将只限于其定义处到文件结束。

关键字 extern 对该变量作“外部变量声明”，表示该变量是一个已经定义的外部变量。

- 例子

  ```c
  #include <stdio.h>
  int max(int x,int y);
  int main(void)
  {
      int result;
      /*外部变量声明*/
      extern int g_X;
      extern int g_Y;
      result = max(g_X,g_Y);
      printf("the max value is %d\\n",result);
      return 0;
  }
  /*定义两个全局变量*/
  int g_X = 10;
  int g_Y = 20;
  int max(int x, int y)
  {
      return (x>y ? x : y);
  }
  ```

  在多项目的情况下可以避免重复定义

  ```c
  /****max.c****/
  #include <stdio.h>
  /*外部变量声明*/
  extern int g_X ;
  extern int g_Y ;
  int max()
  {
      return (g_X > g_Y ? g_X : g_Y);
  }
  
  /***main.c****/
  #include <stdio.h>
  /*定义两个全局变量*/
  int g_X=10;
  int g_Y=20;
  int max();
  int main(void)
  {
      int result;
      result = max();
      printf("the max value is %d\\n",result);
      return 0;
  }
  ```

## inline

定义内敛函数，必须在声明的函数定义前面加上inline

目的：代替宏定义，函数的代码被放入[符号表](https://baike.baidu.com/item/符号表?fromModule=lemma_inlink)中，在使用时直接进行替换（像宏一样展开），减少函数调用的开销，效率也很高。

注意这是对编译器的请求，最后由编译器自行决定是否真的进行扩展

1. 使用宏定义在实现一些函数功能，使用[预处理器](https://baike.baidu.com/item/预处理器?fromModule=lemma_inlink)实现，没有了参数压栈**，[代码生成](https://baike.baidu.com/item/代码生成?fromModule=lemma_inlink)**等一系列的操作，效率很高，
2. 但在使用它时，仅仅只是做预处理器[符号表](https://baike.baidu.com/item/符号表?fromModule=lemma_inlink)中的简单替换，因此它不**能进行参数有效性的检测**，也就**不能享受C++[编译器](https://baike.baidu.com/item/编译器?fromModule=lemma_inlink)严格类型检查**的好处，另外它的**返回值也不能被强制转换**为可转换的**合适的类型**。这样，它的使用就存在着一系列的隐患和局限性。
3. 如果一个操作涉及到类的保护成员或私有成员，你就不可能使用这种宏定义来实现

## volatile

类型修饰符,直接存取原始内存地址

volatile提醒编译器它后面所定义的变量随时都有可能改变，因此编译后的程序每次需要存储或读取这个变量的时候，告诉编译器对该变量不做优化，都会直接从变量内存地址中读取数据而不是寄存器中读取数据

- 例子，***\*并行设备的硬件寄存器\****

  ```cpp
  int *output = (unsigned int *)0xff800000;//定义一个IO端口；
  int  init(void)
  {
        int i;
        for(i=0;i< 10;i++){
          *output = i;
  }
  }
  ```

  经过编译器优化后，

  ```cpp
  int init(void)
  {
  *output =9;
  }
  ```

  如果你对此外部设备进行初始化的过程是必须是像上面代码一样顺序的对其赋值，显然优化过程并不能达到目的。

- 例子，***\*中断服务程序中修改的供其它程序检测的变量\****

  当变量在触发某中断程序中修改，而编译器判断主函数里面没有修改该变量，因此可能只执行一次从内存到某寄存器的读操作，而后每次只会从该寄存器中读取变量副本，使得中断程序的操作被短路。

***\*多任务环境下各任务间共享的标志，应该加volatile；\****

***\*存储器映射的硬件寄存器通常也要加volatile说明，因为每次对它的读写都可能由不同意义；\****

***\*一个参数既可以是const还可以是volatile吗？\****

可以的，例如只读的状态寄存器。它是volatile因为它可能被意想不到地改变。它是const因为程序不应该试图去修改它。

***\*一个指针可以是volatile 吗？\****

可以，当一个中服务子程序修改一个指向buffer的指针时。

- ***\*下面的函数有什么错误\****

  ```cpp
  int  square(volatile int*ptr)
  {
      return*ptr * *ptr;//返回ptr指向内容的平方
  }
  ```

  多线程下，ptr可能改变，ptr可能在两次取值时取到不同的数值，正确如下

  ```cpp
  long square(volatile int*ptr){
      int a;
      a = *ptr;
      return a * a;
  }
  ```

  频繁地使用volatile很可能会增加代码尺寸和降低性能

# 宏

- ***\*LINE****

  输出当前代码的行号

- ***\**\*func\*\**\***

  输出当前函数的名称

# 语法

- cr4 |= (1 << 21);

  这句话的意思是将`cr4`变量中第21位的值设为1。具体来说，`cr4`是一个32位的无符号整数，每一位代表了CR4寄存器中的一个属性。通过左移21位，可以将数字1移动到第21位，然后通过与操作符将其与`cr4`中的原值合并，即可将第21位的值设为1

- void *a = acb;//？

  ```c
  int acb *=* (int*)malloc（sizeof(int));
  void *a = acb;
  //这样的语法不是在所有编译器里都能通过的
  //linux内核开发中，实测不行
  ```

- 数组的初始化

  ```c
  int a[10]={
  	[1]=1,
  	[2]=2,
  	[9]=9
  }
  ```

- 结构体的初始化

  ```c
  typedef struct{
  	int bandrate;
  	int databits;
  	int stopbits;
  	int parity;
  	int dtr;
  }serial_hard_config_def;
  //顺序初始化
  serial_hard_config_def serial ={
  	115200,
  	8,
  	1,
  	0,
  	0
  };
  //指定初始化
  serial_hard_config_def seria2 = {
  	.bandrate = 115200,
  	.databits = 8,
  	.stopbits = 1,
  	.parity = 0,
  	.dtr = 0
  };//乱序在一些编译器里不支持
  ```

- int& b = a;

  int& b =  这里必须跟上一个int型的变量名字，且int&b必须初始化

  功能相当于给变量a起了个别名，他们都是共用同一个地址，存放同一个值

  示例：

  `int a = 10;`

  `int &b = a;`

  此时，a和b都是int型，并且修改其中一个，另一个值也会发生变化

  ```c
  int a = 1024;
  int& b = a;
  cout << &a<<' ' << &b;
  //输出同样的地址
  ```

- 逗号表达式

  两个及其以上的式子联接起来，从左往右逐个计算表达式，整个表达式的值为最后一个表达式的值。

  `a=(a=3**5,a**4)`//a==60

# 枚举

```
enum season {spring, summer=3, autumn, winter};
```

对应0，3，4，5

`enum season sea;` //定义枚举变量

```cpp
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;//定义枚举的同时定义枚举变量
enum 
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;//直接定义枚举变量
```

枚举是连续时可以支持遍历

```cpp
enum DAY{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
int main(){
    for (day = MON; day <= SUN; day++) {
        printf("枚举元素：%d \\n", day);
    }
}
```

# 理论

- 结构体对⻬规则

  - 为什么要对齐

    1. 平台原因(移植原因)： 不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能 在某些地址处取某些特定类型的数据，否则抛出硬件异常。

    2.性能原因

    32位cpu，32位/8位=4字节，所以cpu一次工作可以取到4个字节的数据，对于如下这个数据

    ![image-20231102142603781](C:\Users\松夸夸\AppData\Roaming\Typora\typora-user-images\image-20231102142603781.png)

    图左：如果不对齐，需要读取到完整的i需要读取两次（或者说两行）然后拼接再一起，I/O操作是很耗时的，这是很浪费时间的。

    图右：如果需要更快读到数据，那一个数据最好是存在一整行，像上面第三张图那样。

  从起始地址为0开始，之后每一个数组成员都从自己成员大小的整数倍的地方开始，

  整个结构体的大小又以内部最大成员的大小的倍数补齐

  如果A结构体嵌套B结构体，B的地址以B结构体最大成员的倍数的地方开始

  数组的话起始地址只按数组的一个值来算

  ```c
  struct X
  {
      char a;   //1
      double b; //8
      int c;    //4
  }//24
  struct X
  {
      char a;   //1
      int c;    //4
      double b; //8
  }//16
  ```

  ```c
  struct X
  {
      char a;   //1
      int b;    //4
      double c; //8
  };
  
  struct Y
  {
      char a; //8
      X b;// 16
  };//8+16=24,a的大小应为X中元素最宽元素double c的倍数而不是16的倍数
  ```

  ```c
  struct tag0
  {
      int a;               //4
      char b;              //1
  } _attribute_ ((packed)); //编译器选项，不填充
  ```

  ```c
  typedef union {
  	 long i;//4
  	 int k[5];
  	 char c; 
  } DATE; //20
  
  struct data {
  	int cat;//4
  	DATE cow;//20，这里里面的元素是4，所以计算补齐时这里不按20来算，而是按4来算
  	double dog;//8
  	//int a;//加上后变40（最大宽度为8，补齐8的倍数）
  };//32（最大宽度为8，所以8的倍数）
  ```

- 位域

  结构体位域大小计算原则

  1.如果相邻位域字段的类型相同，且其位宽之和小于类型的sizeof大小，则后面的字段将紧邻前一个字段存储，直到不能容纳为止；

  2.如果相邻位域字段的类型相同，但其位宽之和大于类型的sizeof大小，则后面的字段将从新的存储单元开始，其偏移量为其类型大小的整数倍；

  3.如果相邻的位域字段的类型不同，则各编译器的具体实现有差异，VC6采取不压缩方式，Dev-C++采取压缩方式；

  4.如果位域字段之间穿插着非位域字段，则不进行压缩；

  1. 整个结构体的总大小为最宽基本类型成员大小的整数倍。

  [//1.char占1个字节8位](http://1.xn--char18-tv7it1fd4owwump2h)，由于b1只占用前5位，但是后三位不够存放b2（则需要用新的存储单元来放b2）

  [//2.所以b2存放在新的存储空间](http://2.xn--b2-hf3cy0ey3n0pgba252ldrgv6a774jz6jw53g),以此类推，

  8+8+8+8+8=5个字节 typedef struct AA {undefined

  unsigned char b1 : 5; //位域，char占1个字节8位，5代表b1占用前5位，后3位不用

  unsigned char b2 : 5;

  unsigned char b3 : 5;

  unsigned char b4 : 5;

  unsigned char b5 : 5; } AA;

  [//1.int占4个字节32位](http://1.xn--int432-sv7iu1fc4owwump2h)，b1占用1位（0），空出两位（12），b3占用3位（35），b4占用2位（56），b5占用3位（79）

  [//2.b5遇到类型不同的short](http://2.xn--b5short-5w3kn12abyei2lu40hchqfl7d) b6,b6需要存在新的存储单元，补齐之后b1和b3b5占4个字节，b6占用4位（03），

  [//3.b6遇到类型不同的int](http://3.xn--b6int-pi1h60w31dd6jry7f2ynt69c) b7，b7需要存在新的存储单元,补齐之后b6占4个字节（short占4个字节32位）

  [//4.b7占用1位](http://4.xn--b71-u49d79ps62d)（0），补齐之后b7占用4个字节，所以结构体大小为4+4+4=12个字节

  typedef struct CC

  {

  int b1 : 1;

  int : 2;  //空出2位，啥也没存

  int b3 : 3; //b1~b5为同类型的变量，所以可以存储在连续的空间中，总共占4个字节

  int b4 : 2;

  int b5 : 3;

  short b6 : 4; //b6和上面的类型不同，所以需要存储在新的单元中，总共占4个字节

  int b7 : 1;  //b7和上面的b6类型不同，所以需要存储在新的单元中，总共占4个字节

  } CC;

  [//1.int占4个字节32位](http://1.xn--int432-sv7iu1fc4owwump2h),a占用29位，剩余的3位不够放b的29位，所以b存放在新的单元中，剩余的3位不够放c的4位

  [//2.所以c存放在新的单元中](http://2.xn--c-lq6awwz9eysd28hnoib0ocrgv6a774j)，所以结构体大小4+4+4=12

  struct test3

  {

  int a : 29;

  int b : 29;

  int c : 4;

  };

## itoa 函数

`itoa` 函数是将整数转化为字符串的函数。它的函数原型为：

```c
char* itoa(int value, char* str, int base);
```

其中，`value` 为要转化的整数，`str` 为转化后的字符串，`base` 为进制数（如十进制、八进制、十六进制等）。

`itoa` 函数的功能主要有以下两个：

- 将整数转化为指定进制的字符串；
- 将字符串反转。

atoi