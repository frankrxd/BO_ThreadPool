# Linux多线程

## 线程相关操作

#### pthread_create

```c++
int pthread_create(pthread_t*thread,pthread_attr_t*attr,void *(*start_routine)(void*), void*arg);

//常用形式
pthread_t pthid;
pthread_create(&pthid,NULL,pthfunc,NULL);
//pthread_create(&pthid,NULL,pthfunc,(void*)3)
pthread_exit(NULL);
//pthread_exit((void*)3); //3作为返回值被pthread_join()函数捕获
```

#### pthread_join

```c++
int pthread_join(pthread_t th, void**thread_return); 
//如果线程通过调用 pthread_exit()终止，则 pthread_exit()中的参数相当于自然返回值，照样可以被其它线程用 pthread_join 获取到。

//example
pthread_join(pthid,NULL);//等待线程pthid结束，阻塞
```

#### pthread_detach

> 调用pthread_join后，如果该线程没有运行结束，调用者会被阻塞，在有些情况下我们并不希望如此。比如在Web服务器中当主线程为每个新来的链接创建一个子线程进行处理的时候，主线程并不希望因为调用pthread_join而阻塞（因为还要继续处理之后到来的链接），这时可以在子线程中加入代码`pthread_datach`脱离阻塞，这时候子线程状态为`detached`，运行结束后会自动释放资源。

```c++
int pthread_detach(pthread_t thread) //成功返回0
```

#### pthread_self

> 获取当前线程id  数据类型为 %u

```c++
pthread_t pthreadself(void)
```

#### pthread_exit

```c++
void pthread_exit(void *retval)
```

#### pthread_main_np

```c++
if(pthread_main_np()){
    //main thread
}else{
    //others thread
}
```

#### pthread_once

> 在多线程环境中只执行一次，将多线程环境中只需要执行一次的操作交给 `pthread_once`执行，则会线程安全的只执行一次。

```c++
  1 #include<iostream>                                                 
  2 #include<pthread.h>
  3 #include <unistd.h>
  4 using namespace std;
  5 
  6 pthread_once_t once = PTHREAD_ONCE_INIT;
  7 
  8 void once_run()
  9 {
 10     cout<<"once_run in thread "<<(unsigned int )pthread_self()<<end
 11 }
 12 
 13 void * child1(void * arg)
 14 {
 15     pthread_t tid =pthread_self();
 16     cout<<"thread "<<(unsigned int )tid<<" enter"<<endl;
 17     pthread_once(&once,once_run);
 18     cout<<"thread "<<tid<<" return"<<endl;
 19     return nullptr;
 20 }
 21 
 22 
 23 void * child2(void * arg)
 24 {
 25     pthread_t tid =pthread_self();
 26     cout<<"thread "<<(unsigned int )tid<<" enter"<<endl;
 27     pthread_once(&once,once_run);
 28     cout<<"thread "<<tid<<" return"<<endl;
 29     return nullptr;
 30 }
 31 
 32 int main()
 33 {
 34     pthread_t tid1,tid2;
 35     pthread_create(&tid1,NULL,child1,NULL);
 36     pthread_create(&tid2,NULL,child2,NULL);
 37     sleep(10);
 38     cout<<"main thread exit."<<endl;
 39     return 0;
 40 }

```

#### pthread_cancel

> 发送终止信号给thread线程，如果成功则返回0.发送成功并不意味着thread终止。若是在整个程序退出时，要终止各个线程，应该在成功发送 cancel指令后，使用 pthread_join 函数，等待指定的线程已经完全退出以后，再继续执行；否则，很容易产生 “段错误”。

```c++
int pthread_cancel(pthread_t thread)
```



## 线程互斥

#### pthread_mutex_init

```c++
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);//动态创建
```

#### pthread_mutex_t

```c++
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;//静态创建
```

#### pthread_mutex_destroy

```c++
int pthread_mutex_destroy(pthread_mutex_t *mutex);//注销互斥锁
```

#### pthread_mutex_lock

```c++
int pthread_mutex_lock(pthread_mutex_t *mutex);
```

#### pthread_mutex_unlock

```c++
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

#### pthread_mutex_trylock

```c++
//判断是否可以加锁，如果可以加锁并返回0，否则返回非0
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```



## 线程同步

#### pthread_cond_t

```c++
//静态创建 
pthread_cond_t cond=PTHREAD_COND_INITIALIZER; 
```

#### pthread_cond_init

```c++
//动态创建 
int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);
```

#### pthread_cond_destroy

```c++
//注销条件变量 
int pthread_cond_destroy(pthread_cond_t *cond);
```

#### pthread_cond_wait

```c++
//条件等待
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex); 
```

#### pthread_cond_timedwait

```c++
//超时等待
int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex,                            const struct timespec *abstime); 
```

#### pthread_cond_broadcast

```c++
//开启条件，启动所有等待线程 
int pthread_cond_broadcast(pthread_cond_t *cond); 
```

#### pthread_cond_signal

```c++
//开启一个等待信号量,激活一个等待该条件的线程
int pthread_cond_signal(pthread_cond_t *cond);
```

