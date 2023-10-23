 
 # **openEuler系统环境实验**
 ## 1.1 进程
 ### 1.11 输出父、子进程的pid
 ```#include  <sys/types.h>
#include  <stdio.h>
#include  <unistd.h>
#include<sys/wait.h>
int  main()
{
pid_t  pid, pid1;
pid=fork();
if (pid<0)
{
fprintf(stderr,"fork fail");
return  1;
}
else  if (pid==0)
{
pid1=getpid();
printf("child:pid=%d\n",pid); //A
printf("child:pid1=%d\n",pid1); //B
}
else
{
pid1=getpid();
printf("parent:pid=%d\n",pid); //C
printf("parent:pid1=%d\n",pid1); //D
wait(NULL);
}
return  0;
}
```
结果：
>![1.11](https://github.com/duxiaozhod/os1/blob/main/1.11.png)
>
解释：在函数中fork（）生成父进程副本，返回父子进程，且若是父进程则fork（）返回子进程pid，若是子进程则返回0.若未创建子进程则返回负值。所以在打印的结果中，parent：pid是子进程的pid，parent：pid1才是父进程的pid。因为fork（）返回值的原因，所以子进程中child：pid的值为0，child：pid1为子进程的pid。
### 1.12（1.11删除wait（NULL）的影响）
```#include  <sys/types.h>
#include  <stdio.h>
#include  <unistd.h>
int  value=0;
int  main()
{
pid_t  pid, pid1;
pid=fork();
if (pid<0)
{
fprintf(stderr,"fork fail");
return  1;
}
else  if (pid==0)
{
pid1=getpid();
printf("child:pid=%d value=%d &value=%d\n",pid,value,&value); //A
printf("child:pid1=%d value=%d &value=%d\n",pid1,value,&value); //B
value--;
}
else
{
pid1=getpid();
printf("parent:pid=%d value=%d &value=%d\n",pid,value,&value); //C
printf("parent:pid1=%d value=%d &value=%d\n",pid1,value,&value); //D
value++;
}
printf("return_ vaule=%d,&value=%d\n",value,&value);
return  0;
}
```
结果：
>![1.12](https://github.com/duxiaozhod/os1/blob/main/1.12.png)
>
解释：wait(NULL)的作用是让父进程等待子进程结束防止僵尸进程的产生，由于1.11的程序父进程是否等待子进程结束的结果不明显，所以我在这里特地增添一个全局变量value显示，wait（NULL）的作用。显然两个程序中打印value的时间不同，在有wait（NULL）的程序中父进程是在子进程运行结束后才打印value的值，而没有wait的程序父进程在子进程还没结束就打印了value的值。
###  1.13&1.14 进程全局变量间的关系
```#include  <sys/types.h>
#include  <stdio.h>
#include  <unistd.h>
#include<sys/wait.h>
int  value=5;
int  main()
{
pid_t  pid, pid1;
pid=fork();
if (pid<0)
{
fprintf(stderr,"fork fail");
return  1;
}
else  if (pid==0)
{
pid1=getpid();
value--;
printf("child:pid=%d value=%d &value=%d\n",pid,value,&value); 
printf("child:pid1=%d value=%d &value=%d\n",pid1,value,&value); 
}
else
{
pid1=getpid();
value++;
printf("parent:pid=%d value=%d &value=%d\n",pid,value,&value); 
printf("parent:pid1=%d value=%d &value=%d\n",pid1,value,&value); 
}
wait(NULL);
printf("return_ vaule=%d,&value=%d\n",value,&value);
return  0;
}
```
结果：
>![1.13](https://github.com/duxiaozhod/os1/blob/main/1.13.png)
>
解释：我将1.13和1.14结合起来一次性解释， fork（）函数创建的子进程是父进程的副本，这意味着子进程中的所有变量和数据结构都会在内存中有与父进程相同的位置和值。但同时父进程和子进程在操作系统中分别获得了一个不同的虚拟地址空间，所以在父子空间中value的值不一定是一样的，且父子进程中的value不会对对方造成影响。所以虽然是全局变量value，初始值为5，在父进程中被加一变成了6，在子进程中减一变成了4，两个进程互不影响。但同时父子进程中的这些地址空间的内容是一致的。所以在程序中对value取地址，父子进程的value地址是一致的。同时我们还在程序结束前加了return_value返回当前value的值，因为函数中有wait（NULL）所以父进程会等子进程结束。所以最终结果是子进程先结束运行打印了在子进程虚拟空间中的value值4，在打印在父进程虚拟空间的value值6.
### 1.15 进程中调用system（） exec（）
首先是system（）和exec（）所调用程序的代码：
```#include  <sys/types.h>
#include  <stdio.h>
#include  <unistd.h>
int  main()
{
printf("system_call pid=%d\n",getpid());
return  0;
}
```
在进程中用system（）调用函数：
```
#include  <sys/types.h>
#include  <stdio.h>
#include  <unistd.h>
#include<stdlib.h>
int  main()
{
pid_t  pid, pid1;
pid=fork();
if (pid<0)
{
fprintf(stderr,"fork fail");
return  1;
}
else  if (pid==0)
{
printf("child:pid=%d\n",getpid());
system("./1.141");
}
else
{
printf("parent:pid=%d\n",getpid());
}
return  0;
}
```
在进程中用execvp（）调用函数：
```#include  <sys/types.h>
#include  <stdio.h>
#include  <unistd.h>
int  main()
{
pid_t  pid, pid1;
pid=fork();
if (pid<0)
{
fprintf(stderr,"fork fail");
return  1;
}
else  if (pid==0)
{
printf("child:pid=%d\n",getpid());
char *args[]={"./1.141",NULL};
execvp(args[0],args);
}
else
{
printf("parent:pid=%d\n",getpid());
}
return  0;
}
```
实验结果：
>![1.14](https://github.com/duxiaozhod/os1/blob/main/1.14.png)
>
解释：从结果可以简单看出调用execvp（）函数打印出的system_call的pid值与子进程的pid值一致，而调用system（）打印出的system_call的pid值与子进程的pid值不一致，为子进程pid值加1.所以使用execvp（）是在当前进程中调用exe文件并更换程序映像，而使用system（）是创建一个新进程，在新进程中运行调用的exe文件。
## 1.2 线程
### 1.21 创建线程
```
#include  <stdio.h>
#include  <pthread.h>
int  value=0;
void  *thraead1(void  *param); /* the thread1 */
void  *thraead2(void  *param); /* the thread2 */
int  main()
{
pthread_t tid1,tid2;
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_create(&tid1, &attr, thraead1, NULL);
pthread_create(&tid2, &attr, thraead2, NULL);
pthread_join(tid1,NULL);
pthread_join(tid2,NULL);
printf("value=%d\n", value);
return  0;
}
void  *thraead1(void  *param)
{
printf("thread1 create success\n");
for(int  i=0;i<1000000;i++)
value+=100;
pthread_exit(0);
}
void  *thraead2(void  *param)
{
printf("thread2 create success\n");
for(int  i=0;i<1000000;i++)
value-=100;
pthread_exit(0);
}
```
实验结果：
>![1.21](https://github.com/duxiaozhod/os1/blob/main/1.21.png)
>
解释：我们在主函数中创建了两个线程tid1和tid2.其中tid1做的是循环1000000次对1全局变量value加100，tid2做的是循环1000000次对value减100.但是由于我们没有对线程进行同步操作，所以线程tid1和tid2可能同时对全局变量value进行修改导致最终结果与我们想象中的不一致。我们预期经过tid1和tid2，全局变量的值仍为0，但是实际结果确实一些不确定的数，可以看出线程没进行同步操作的影响。
### 1.22  添加锁进行线程同步
```
#include  <stdio.h>
#include  <pthread.h>
int  value=0;
void  *thraead1(void  *param); /* the thread1 */
void  *thraead2(void  *param); /* the thread2 */
pthread_mutex_t lock=PTHREAD_MUTEX_INITIALIZER;
int  main()
{
pthread_t tid1,tid2;
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_create(&tid1, &attr, thraead1, NULL);
pthread_create(&tid2, &attr, thraead2, NULL);
pthread_join(tid1,NULL);
pthread_join(tid2,NULL);
printf("value=%d\n", value);
pthread_mutex_destroy(&lock);
return  0;
}
void  *thraead1(void  *param)
{
pthread_mutex_lock(&lock);
printf("thread1 create success\n");
for(int  i=0;i<10000;i++)
value+=100;
pthread_mutex_unlock(&lock);
pthread_exit(0);
}
void  *thraead2(void  *param)
{
pthread_mutex_lock(&lock);
printf("thread2 create success\n");
for(int  i=0;i<10000;i++)
value-=100;
pthread_mutex_unlock(&lock);
pthread_exit(0);
```
实验结果：
>![1.22](https://github.com/duxiaozhod/os1/blob/main/1.22.png)
>
解释：在这个程序里，我们通过对线程进行加锁实现进程同步，这样两个进程不会同时修改全局变量造成错误，从结果也可以看出全局变量最终输出结果为0与我们预期一致。
### 1.23 在线程中调用system（） exec（）
>程序中调用的1.141与前文使用一致，作用为打印当前进程pid
```#include  <stdio.h>

#include  <pthread.h>

#include  <sys/types.h>

#include  <unistd.h>

#include<stdlib.h>

void  *thraead1(void  *param); /* the thread1 */

void  *thraead2(void  *param); /* the thread2 */

pthread_mutex_t lock=PTHREAD_MUTEX_INITIALIZER;

int  main()

{

pthread_t tid1,tid2;

pthread_attr_t attr;

pthread_attr_init(&attr);

pthread_create(&tid1, &attr, thraead1, NULL);

pthread_create(&tid2, &attr, thraead2, NULL);

pthread_join(tid1,NULL);

pthread_join(tid2,NULL);

pthread_mutex_destroy(&lock);

return  0;

}

void  *thraead1(void  *param)

{

pthread_mutex_lock(&lock);

printf("thread1 create success\n");

printf("thread1 tid=%d,pid=%d\n",pthread_self(),getpid());

system("./1.141");

printf("thread1 system_callreturn\n");

pthread_mutex_unlock(&lock);

pthread_exit(0);

}

void  *thraead2(void  *param)

{

pthread_mutex_lock(&lock);

printf("thread2 create success\n");

printf("thread2 tid=%d,pid=%d\n",pthread_self(),getpid());

system("./1.141");

printf("thread2 system_callreturn\n");

pthread_mutex_unlock(&lock);

pthread_exit(0);

}
```
结果：
>![1.23](https://github.com/duxiaozhod/os1/blob/main/1.23.png)
>
解释：从结果可以看出，tid1和tid2的tid不一致但是pid是一致的，在两个线程中分别调用system调用其他程序本质上是创建一个新的进程在新的进程中运行该程序，所以在tid1和tid2调用system打印出的pid不一致。
```#include  <stdio.h>

#include  <pthread.h>

#include  <sys/types.h>

#include  <unistd.h>

#include<stdlib.h>

void  *thraead1(void  *param); /* the thread1 */

void  *thraead2(void  *param); /* the thread2 */

pthread_mutex_t lock=PTHREAD_MUTEX_INITIALIZER;

int  main()

{

pthread_t tid1,tid2;

pthread_attr_t attr;

pthread_attr_init(&attr);

pthread_create(&tid1, &attr, thraead1, NULL);

pthread_create(&tid2, &attr, thraead2, NULL);

pthread_join(tid1,NULL);

pthread_join(tid2,NULL);

pthread_mutex_destroy(&lock);

return  0;

}

void  *thraead1(void  *param)

{

pthread_mutex_lock(&lock);

printf("thread1 create success\n");

printf("thread1 tid=%d,pid=%d\n",pthread_self(),getpid());

char *args[]={"./1.141",NULL};

execvp(args[0],args);

printf("thread1 system_callreturn\n");

pthread_mutex_unlock(&lock);

pthread_exit(0);

}

void  *thraead2(void  *param)

{

pthread_mutex_lock(&lock);

printf("thread2 create success\n");

printf("thread2 tid=%d,pid=%d\n",pthread_self(),getpid());

char *args[]={"./1.141",NULL};

execvp(args[0],args);

printf("thread2 system_callreturn\n");

pthread_mutex_unlock(&lock);

pthread_exit(0);

}
```
结果：
>![1.24](https://github.com/duxiaozhod/os1/blob/main/1.24.png)
>
解释：可以看出来实验结果与system有很大的不一致，原因在于execvp会替换当前进程的映像为新的程序映像，因此这个线程将不再执行后续的代码。所以程序执行到打印system_call的pid值后便不在进行，且该pid值与线程的pid值是一致的。execvp（）是在当前进程进行调用更换程序映像。
## 1.3 自旋锁的设计
```#include  <stdio.h>

#include  <pthread.h>

// 定义自旋锁结构体

typedef  struct {

int  flag;

} spinlock_t;

// 初始化自旋锁

void  spinlock_init(spinlock_t  *lock) {

lock->flag = 0;

}

// 获取自旋锁

void  spinlock_lock(spinlock_t  *lock) {

while (__sync_lock_test_and_set(&lock->flag, 1)) {

}

}

// 释放自旋锁

void  spinlock_unlock(spinlock_t  *lock) {

__sync_lock_release(&lock->flag);

}

// 共享变量

int  shared_value = 0;

// 线程函数

void  *fthread1(void  *arg) {

printf("thread1 create success\n") ;

spinlock_t *lock = (spinlock_t *)arg;

for (int  i = 0; i < 5000; ++i) {

spinlock_lock(lock);

shared_value++;

spinlock_unlock(lock);

}

return  NULL;

}

void  *fthread2(void  *arg) {

printf("thread2 create success\n") ;

spinlock_t *lock = (spinlock_t *)arg;

for (int  i = 0; i < 5000; ++i) {

spinlock_lock(lock);

shared_value++;

spinlock_unlock(lock);

}

return  NULL;

}

int  main() {

pthread_t thread1, thread2;

spinlock_t  lock;

spinlock_init(&lock); // 初始化自旋锁

pthread_create(&thread1, NULL, fthread1, &lock); // 创建第一个线程

pthread_create(&thread2, NULL, fthread2, &lock); // 创建第二个线程

pthread_join(thread1, NULL); // 等待第一个线程结束

pthread_join(thread2, NULL); // 等待第二个线程结束

printf("Shared value: %d\n", shared_value);// 输出共享变量的值

return  0;

}
```
结果：
>![1.3](https://github.com/duxiaozhod/os1/blob/main/1.3.png)
>
解释：我们通过自己定义spinlock_t实现了自旋锁，在spinlock_t中有一个整形变量flag，当flag为1时表示资源被占用进行上锁。我们创建了两个线程，在两个线程中分别利用锁的机制来实现进程同步，当线程1需要对公共变量shared_value进行修改时就先进行加锁将lock的flag置为1，防止线程2同时进行以实现同步。当线程完成任务后便将lock的flag置为0进行释放锁。锁释放后线程2便可以进行。
>[github仓库](https://github.com/duxiaozhod/os1)

