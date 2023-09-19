# 线程池 thread_pool 开发手册

适用于线程并发任务的线程池。可以动态规划线程池内线程数目（  **注意：仅在于初始化线程池阶段** ），基于POSIX 开发。

## 数据说明

- task_t：任务类型

  - ```c
    typedef void (thread_t)(void *);
    typedef struct tasks
    {
        // 任务指针
        void (task_point)(void *);
    
        // 任务参数
        void *args;
        
        // 下一个任务
        struct tasks *next;
    
    }task_t;
    
    ```

  - task_point：任务函数指针，即用户自定义任务函数（线程池中线程需要执行的任务函数）。

  - args：线程在执行任务函数时，需要传递给任务函数的参数

  - next：下一个任务结点

- thread_pool_t : 线程池类型

  - ```c
    typedef struct thread_pool
    {
    // 线程个数
    int             thread_count;
      // 标明线程池是否启动
    int             pool_status;
      // 线程id的集合
    pthread_t      *thread_id;
      // 线程共享的互斥锁
    pthread_mutex_t shared_mutex;
      // 线程共享的条件变量
    pthread_cond_t  shared_cond;
      // 任务链表
    task_t         *task_list;
      /*
        最大线程数目：表示可以支持线程并发的最大线程数
        当前服役线程数目：表示当前可以用执行任务的线程
        当前休眠线程数目：表示当前可支持同时并发的线程数
        ...
    */
    
    }thread_pool_t;
    ```

  - thread_count：线程池中最大线程数目，即能够支持的最高并发数目

  - pool_status ：线程池当前状态，即启动或停止状态

  - thread_id ：线程池的线程集合，即线程池中正在等待任务的线程集合。

  - shared_mutex：线程池中线程所共享的线程互斥锁

  - shared_cond : 线程池中线程所共享的条件变量

  - task_list ：线程池中线程所需要执行的任务链表指针

## 线程池thread_poolAPI介绍

### 初始化线程池

```c
thread_pool_t *thread_pool_init(int count);
/*
	作用：
		初始化/创建一个线程池
		初始化thread_pool_t 线程池中的成员变量
			- thread_count：线程池中最大线程数目，即能够支持的最高并发数目
            - pool_status ：初始化为1，表示线程池为开启状态
            - thread_id ： 即每开启一个线程就将该线程的id好保存至thread_id中
            - task_list ：初始化为NULL即不存在任何任务。

	@count：
		即线程池中最大并发线程数目
	@return：
		成功返回初始化/创建好的线程池指针
		失败返回NULL
*/
```

### 为线程池增加任务

```c
void add_new_task(thread_pool_t *pool,void *args,thread_t task_rountine);
/*
	作用：
		往一个已存在的线程池中，添加线程任务
		即向task_list 中增加结点
	实现过程：
		..
	@pool:
		需要增加任务结点的线程池指针
	@args：
		任务结点的参数指针
	@task_routine:
		任务指针
*/
```

### 销毁线程池

```c
void thread_destroy(thread_pool_t **pool);
/*
	作用：
		销毁一个已存在的线程池
		即将pthread_pool_t 线程池中的成员逐一销毁
	@pool：
		需要销毁的线程池指针
*/
```



