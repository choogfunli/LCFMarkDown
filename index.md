# iOS多线程编程

You can use the [editor on GitHub](https://github.com/choogfunli/LCFMarkDown/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

# NSThread

NSThread使用

## 1、实例初始化、

```markdown
 //创建线程
 NSThread *newThread = [[NSThread alloc] initWithTarget:self selector:@selector(demo:) object:@"Thread"];
 //或者
 NSThread *newThread = [[NSThread alloc] init];
 NSThread *newThread = [[NSThread alloc] initWithBlock:^{
     NSLog(@"initWithBlock");
 }];
 ```

## 2、常用实例方法

-(void)start; 实例化线程需要手动启动

```markdown
[thread start];
```

-(BOOL)isMainThread; 是否为主线程

```markdown
BOOL isMain = [thread isMainThread];
```

-(void)setName:(NSString *)str; 设置线程名称

```markdown
[thread setName = @"my Second Thread"];
```

-(void)cancel ; 取消线程

```markdown
[thread cancel];
```

-(void)main ; 线程的入口函数

```markdown
[thread main];
```

-(void)isExecuting; 判断线程是否正在执行

```markdown
BOOL isRunning = [thread isExecuting];
```

-(void)isFinished;判断线程是否已经结束

```markdown
BOOL isEnd = [thread isFinished];
```

-(void)isCancelled; 判断线程是否撤销

```markdown
BOOL isCancel = [thread isCancelled];
```

## 3、类方法

+(void)currentThread;获取当前线程
```markdown
[NSThread currentThread]
```

+(BOOL)isMultiThreaded; 当前代码运行所在线程是否是子线程
```markdown
BOOL isMulti = [NSThread isMultiThreaded];
```

+(void)sleepUntilDate:(NSDate *)date; 当前代码所在线程睡到指定时间
```markdown
[NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:1.0]];
```

+(void)sleepForTimeInterval:(NSTimeInterval)ti; 当前线程睡多长时间
```markdown
[NSThread sleepForTimeInterval:1.0];
```

+(void)exit; 退出当前线程
```markdown
[NSThread exit];
```

+(double)threadPriority; 设置当前线程优先级
```markdown
double dPriority=[NSThread threadPriority];
```

+(BOOL)setThreadPriority:(double)p; 给当前线程设定优先级，调度优先级的取值范围是0.0 ~ 1.0，默认0.5，值越大，优先级越高。
```markdown
BOOL isSetting=[NSThread setThreadPriority:(0.0~1.0)];
```

## 4、隐式创建&线程间通讯
```markdown
/**
  指定方法在主线程中执行
参数1. SEL 方法
    2.方法参数
    3.是否等待当前执行完毕
    4.指定的Runloop model
*/
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
- 
/**
  指定方法在某个线程中执行
参数1. SEL 方法
    2.方法参数
    3.是否等待当前执行完毕
    4.指定的Runloop model
*/
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
/**
  指定方法在开启的子线程中执行
参数1. SEL 方法
    2.方法参数
*/
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

再注意：苹果声明UI更新一定要在UI线程（主线程）中执行，虽然不是所有后台线程更新UI都会出错。
```

## 5、线程间资源共享&线程加锁
在程序运行过程中，如果存在多线程，那么各个线程读写资源就会存在先后、同时读写资源的操作，因为是在不同线程，CPU调度过程中我们无法保证哪个线程会先读写资源，哪个线程后读写资源。因此为了防止数据读写混乱和错误的发生，我们要将线程在读写数据时加锁，这样就能保证操作同一个数据对象的线程只有一个，当这个线程执行完成之后解锁，其他的线程才能操作此数据对象。

NSLock / NSConditionLock / NSRecursiveLock / @synchronized都可以实现线程上锁的操作。

### 1、@synchronized
首先：开启两个线程同时售票
```markdown
    self.tickets = 20;
    NSThread *t1 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTickets) object:nil];
    t1.name = @"售票员A";
    [t1 start];
    
    NSThread *t2 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTickets) object:nil];
    t2.name = @"售票员B";
    [t2 start];
```
然后：将售票的方法加锁
```markdown
- (void)saleTickets{
    while (YES) {
        [NSThread sleepForTimeInterval:1.0];
        //互斥锁 -- 保证锁内的代码在同一时间内只有一个线程在执行
        @synchronized (self){
            //1.判断是否有票
            if (self.tickets > 0) {
                //2.如果有就卖一张
                self.tickets --;
                NSLog(@"还剩%d张票  %@",self.tickets,[NSThread currentThread]);
            }else{
                //3.没有票了提示
                NSLog(@"卖完了 %@",[NSThread currentThread]);
                break;
            }
        }
    }
}
```
### 2、NSLock
```markdown
  -(BOOL)tryLock;//尝试加锁，成功返回YES ；失败返回NO ，但不会阻塞线程的运行
  -(BOOL)lockBeforeDate:(NSDate *)limit;//在指定的时间以前得到锁。YES:在指定时间之前获得了锁；NO：在指定时间之前没有获得锁。该线程将被阻塞，直到获得了锁，或者指定时间过期。
  - (void)setName:(NSString*)newName//为锁指定一个Name
  - (NSString*)name//**返回锁指定的**name
    @property (nullable, copy) NSString *name;线程锁名称 
```

### 3、NSConditionLock
```markdown
使用此锁，在线程没有获得锁的情况下，阻塞，即暂停运行，典型用于生产者／消费者模型。
- (instancetype)initWithCondition:(NSInteger)condition;//初始化条件锁
- (void)lockWhenCondition:(NSInteger)condition;//加锁 （条件是：锁空闲，即没被占用；条件成立）
- (BOOL)tryLock; //尝试加锁，成功返回TRUE，失败返回FALSE
- (BOOL)tryLockWhenCondition:(NSInteger)condition;//在指定条件成立的情况下尝试加锁，成功返回TRUE，失败返回FALSE
- (void)unlockWithCondition:(NSInteger)condition;//在指定的条件成立时，解锁
- (BOOL)lockBeforeDate:(NSDate *)limit;//在指定时间前加锁，成功返回TRUE，失败返回FALSE，
- (BOOL)lockWhenCondition:(NSInteger)condition beforeDate:(NSDate *)limit;//条件成立的情况下，在指定时间前加锁，成功返回TRUE，失败返回FALSE，
@property (readonly) NSInteger condition;//条件锁的条件
@property (nullable, copy) NSString *name;//条件锁的名称
```

### 4、NSRecursiveLock
```markdown
此锁可以在同一线程中多次被使用，但要保证加锁与解锁使用平衡，多用于递归函数，防止死锁。
- (BOOL)tryLock;//尝试加锁，成功返回TRUE，失败返回FALSE
- (BOOL)lockBeforeDate:(NSDate *)limit;//在指定时间前尝试加锁，成功返回TRUE，失败返回FALSE
```

### 5、线程安全之原子属性 atomic
苹果系统在我们声明对象属性时默认是atomic，也就是说在读写这个属性的时候，保证同一时间内只有一个线程能够执行。当声明时用的是atomic，通常会生成 _成员变量 如果同时重写了getter&setter _成员变量 就不自动生成。实际上原子属性内部有一个锁，叫做“自旋锁”。
首先我们比较一下“自旋锁” & “互斥锁”的异同，然后回答上面的问题

### 共同点
都能够保证线程安全
### 不同点
互斥锁：如果其他线程正在执行锁定的代码，此线程就会进入休眠状态，等待锁打开；然后被唤醒
自旋锁：如果线程被锁在外面，哥么就会用死循环的方式一直等待锁打开！
无论什么锁，都很消耗性能，效率不高，所以在我们平时开发过程中，会使用nonatomic

```markdown
@property (strong, nonatomic) NSObject *myNonatomic;
@property (strong,    atomic) NSObject *myAtomic;
```

# GCD

```markdown
1）[dispatch_async(queue, ^{想执行的任务} );]
这里的queue表示执行处理的等待队列，可以是串行队列也可以是并行队列，串行队列即等待处理的任务先进先出，并行队列即等待处理的任务不需要按顺序执行，可以分出多个线程同时处理
```

```markdown
2）创建队列的方法
dispatch_queue_create   可以生成串行队列和并行队列，dispatch_queue_create可以生成任意数量的serial dispatch queue串行队列，但实际上受到系统资源的限制，容易发生线程竞争的问题
而concurrent dispatch queue不用创建多个，也不会出现线程竞争的问题，系统只会管理有效执行的线程。

GCD线程需要程序员自己维护，不会像ARC一样，能够自动释放
dispatch_retain                
dispatch_release
```

```markdown
3) dispatch_get_main_queue 主队列，本质是一个串行队列
    dispatch_get_global_queue 全局队列，本质是一个并行队列，有优先级（DISPATCH_QUEUE_PRIORITY_HIGH, DISPATCH_QUEUE_PRIORITY_DEFAULT, DISPATCH_QUEUE_PRIORITY_LOW, DISPATCH_QUEUE_PRIORITY_BACKGRAOUND）

dispatch_get_main_queue和dispatch_get_global_queue执行dispatch_retain或者dispatch_release不产生任何影响
```

```markdown
4）dispatch_set_target_queue
dispatch_queue_create 创建的队列，无论是串行还是并行队列，其优先级都等同于dispatch_get_global_queue的DISPATCH_QUEUE_PRIORITY_DEFAULT优先级。
dispatch_set_target_queue可以改变队列的优先级
```

```markdown
5）dispatch_after
dispatch_after(DISPATCH_TIME_NOW,   3 * NSEC_PER_SEC, dispatch_get_main_queue())；  表示3秒后将要执行的操作添加到队列中
```

```markdown
6）dispatch_group
可以监听group中的队列执行完毕并做结束处理
```

```markdown
7) dispatch_barrier_async
等待追加到Concurrent Dispatch Queue上的并行执行的处理全部结束后，dispatch_barrier_async再将指定的处理追加到队列中，优先执行完了，队列才恢复并发执行。
```

```markdown
8）dispatch_sync
```

```markdown
9）dispatch_apply ： 按照指定的次数将指定的Block追加到指定的Dispatch queue中
```

```markdown
10) dispatch_suspend/dispatch_resume
suspend: 挂起队列，剩下没执行的任务暂时挂起
resume: 恢复队列
```

```markdown
11) [dispatch_semaphore_wait]  比serial dispatch queue和dispatch_barrier_async更细粒度的排他控制
dispatch_semaphore_t t = dispatch_semaphore_create(0);   //0表示等待，计数为1或大于1时，减去1而不等待
dispatch_semaphore_signal(t);   //信号量+1  可以轮到下一个最先等待的线程执行
dispatch_semaphore_wait(t, DISPATCH_TIME_FOREVER)；  //等待信号
```

```markdown
12) [dispatch_once]
保证在应用程序中只执行一次指定处理的API，常用来单例初始化
```

[网络编程不要用多线程，要用NSURLConnection异步API]


# NSOperation

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
