## iOS多线程编程

You can use the [editor on GitHub](https://github.com/choogfunli/LCFMarkDown/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### NSThread

NSThread使用

# 1、实例初始化、

```markdown
 //创建线程
 NSThread *newThread = [[NSThread alloc] initWithTarget:self selector:@selector(demo:) object:@"Thread"];
 //或者
 NSThread *newThread = [[NSThread alloc] init];
 NSThread *newThread = [[NSThread alloc] initWithBlock:^{
     NSLog(@"initWithBlock");
 }];
 ```

# 2、常用实例方法

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

# 3、类方法

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
BOOL isSetting=[NSThread setThreadPriority:(0.0~1.0)];
+(NSArray *)callStackReturnAddresses;线程的调用都会有函数的调用函数的调用就会有栈返回地址的记录，在这里返回的是函 数调用返回的虚拟地址，说白了就是在该线程中函数调用的虚拟地址的数组
NSArray *addressArray=[NSThread callStackReturnAddresses];
+(NSArray *)callStackSymbols 同上面的方法一样，只不过返回的是该线程调用函数的名字数字
NSArray* nameNumArray=[NSThread callStackSymbols];
注意：callStackReturnAddress和callStackSymbols这两个函数可以同NSLog联合使用来跟踪线程的函数调用情况，是编程调试的重要手段

3、隐式创建&线程间通讯
以下方法位于NSObject (NSThreadPerformAdditions)分类中，所有继承NSObject 实例化对象都可调用以下方法

/**
  指定方法在主线程中执行
参数1. SEL 方法
    2.方法参数
    3.是否等待当前执行完毕
    4.指定的Runloop model
*/
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
    // equivalent to the first method with kCFRunLoopCommonModes
/**
  指定方法在某个线程中执行
参数1. SEL 方法
    2.方法参数
    3.是否等待当前执行完毕
    4.指定的Runloop model
*/
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
    // equivalent to the first method with kCFRunLoopCommonModes
/**
  指定方法在开启的子线程中执行
参数1. SEL 方法
    2.方法参数
*/
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
注意：我们经常提到的“线程间通讯”其实就是上面几个方法，并不是多高大上，也没有多复杂！！！

再注意：苹果声明UI更新一定要在UI线程（主线程）中执行，虽然不是所有后台线程更新UI都会出错。
4、线程间资源共享&线程加锁
在程序运行过程中，如果存在多线程，那么各个线程读写资源就会存在先后、同时读写资源的操作，因为是在不同线程，CPU调度过程中我们无法保证哪个线程会先读写资源，哪个线程后读写资源。因此为了防止数据读写混乱和错误的发生，我们要将线程在读写数据时加锁，这样就能保证操作同一个数据对象的线程只有一个，当这个线程执行完成之后解锁，其他的线程才能操作此数据对象。NSLock / NSConditionLock / NSRecursiveLock / @synchronized都可以实现线程上锁的操作。

@synchronized
直接上例子：相信12306卖火车票的例子大家了解
首先：开启两个线程同时售票
    self.tickets = 20;
    NSThread *t1 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTickets) object:nil];
    t1.name = @"售票员A";
    [t1 start];
    
    NSThread *t2 = [[NSThread alloc]initWithTarget:self selector:@selector(saleTickets) object:nil];
    t2.name = @"售票员B";
    [t2 start];
然后：将售票的方法加锁

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
NSLock
    -(BOOL)tryLock;//尝试加锁，成功返回YES ；失败返回NO ，但不会阻塞线程的运行
    -(BOOL)lockBeforeDate:(NSDate *)limit;//在指定的时间以前得到锁。YES:在指定时间之前获得了锁；NO：在指定时间之前没有获得锁。
     该线程将被阻塞，直到获得了锁，或者指定时间过期。
  - (void)setName:(NSString*)newName//为锁指定一个Name
  - (NSString*)name//**返回锁指定的**name
    @property (nullable, copy) NSString *name;线程锁名称 
举个例子：

 NSLock* myLock=[[NSLock alloc]init];
NSString *str=@"hello";
[NSThread detachNewThreadWithBlock:^{
            [myLock lock];
NSLog(@"%@",str);
str=@"world";
            [myLock unlock];
    }];
[NSThread detachNewThreadWithBlock:^{
            [myLock lock];
NSLog(@"%@",str);
str=@"变化了";
            [myLock unlock];
    }];
输出结果不加锁之前，两个线程输出一样 hello；加锁之后，输出分辨为hello 与world。

NSConditionLock
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
举个例子：

  NSConditionLock* myCondition=[[NSConditionLock alloc]init];
    [NSThread detachNewThreadWithBlock:^{
        for(int i=0;i<5;i++)
        {
            [myCondition lock];
            NSLog(@"当前解锁条件：%d",i);
            sleep(2);
            [myCondition unlockWithCondition:i];
            BOOL isLocked=[myCondition tryLockWhenCondition:2];
            if(isLocked)
            {
                NSLog(@"加锁成功！！！！！");
                [myCondition unlock];
            }
        }
    }];
输出结果，在条件2 解锁之后，等待条件2 的锁加锁成功。

NSRecursiveLock
此锁可以在同一线程中多次被使用，但要保证加锁与解锁使用平衡，多用于递归函数，防止死锁。
- (BOOL)tryLock;//尝试加锁，成功返回TRUE，失败返回FALSE
- (BOOL)lockBeforeDate:(NSDate *)limit;//在指定时间前尝试加锁，成功返回TRUE，失败返回FALSE
@property (nullable, copy) NSString *name;//线程锁名称
使用示例：

-(void)initRecycle:(int)value
{
   [myRecursive lock];
   if(value>0)
   {
       NSLog(@"当前的value值：%d",value);
       sleep(2);
       [self initRecycle:value-1];
   }
   [myRecursive unlock];
}
输出结果： 从你传入的数值一直到1，不会出现死锁

5、线程安全之原子属性 atomic
原子属性（线程安全）与非原子属性，平时我们@property声明对象属性时会用到nonatomic，是什么意思呢？
苹果系统在我们声明对象属性时默认是atomic，也就是说在读写这个属性的时候，保证同一时间内只有一个线程能够执行。当声明时用的是atomic，通常会生成 _成员变量 如果同时重写了getter&setter _成员变量 就不自动生成。实际上原子属性内部有一个锁，叫做“自旋锁”。
首先我们比较一下“自旋锁” & “互斥锁”的异同，然后回答上面的问题

共同点
都能够保证线程安全
不同点
互斥锁：如果其他线程正在执行锁定的代码，此线程就会进入休眠状态，等待锁打开；然后被唤醒
自旋锁：如果线程被锁在外面，哥么就会用死循环的方式一直等待锁打开！
无论什么锁，都很消耗性能，效率不高，所以在我们平时开发过程中，会使用nonatomic

@property (strong, nonatomic) NSObject *myNonatomic;
@property (strong,    atomic) NSObject *myAtomic;
根据上面描述，我们得出结论，当我们重写了myAtomic的setter和getter方法

- (void)setMyAtomic:(NSObject *)myAtomic{
      _myAtomic = myAtomic;
}
- (NSObject *)myAtomic{
    return _myAtomic;
}
那么我们就必须声明一个_myAtomic静态变量

@synthesize myAtomic = _myAtomic;
否则系统在编译的时候找不到 _myAtomic

6、子线程上的Runloop
在介绍子线程上的Runloop之前先来一个有意思的小插曲，我们来介绍一下Runloop，甚至模拟一个Runloop
Runloop 运行循环
-在目前iOS开发中，几乎用不到，在以前iOS黑暗时代，程序员会用到
目的：
保证程序不退出
监听事件
没有事件让程序进入休眠
区分模式：
NSDefaultRunLoopMode - 时钟、网络事件
NSRunLoopCommonModes - 用户交互
模拟Runloop

void click(int type){
    printf("正在运行第%d",type);
}
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        while (YES) {
            printf("请输入选项 0 表示退出");
            int result = -1;
            scanf("%d",&result);
            if (result == 0) {
                printf("程序结束\n");
                break;
            }else{
                click(result);
            }
        }
    }
    return 0;
}
在iOS中，开辟的子线程上的Runloop是默认不开启的，并且子线程中的Runloop开启之后是手动无法关闭的。那么当我们给子线程中重复添加不同任务时并且Runloop没有开启的情况下，子线程无法监听事件（确切说是子线程的Runloop），我们后来添加的任务就无法执行。
但是我们如果让子线程Runloop一直工作又浪费资源，下面介绍一个OC中常用到的可以控制子线程Runloop的例子：
首先，Runloop就是一个死循环，那么我们就创建一个死循环，然后声明一个可以判断是否应该退出Runloop循环的属性
@property (assign, nonatomic, getter=isFinished) BOOL finished;
创建子线程并添加任务

    NSThread *t = [[NSThread alloc]initWithTarget:self selector:@selector(demo) object:nil];
    [t start];
    self.finished = NO;
    [self performSelector:@selector(otherMethod) onThread:t withObject:nil waitUntilDone:NO];
在第一个任务中加入死循环

- (void)demo{
    NSLog(@"%@",[NSThread currentThread]);
    //在OC中使用比较多的，退出循环的方式
    while (!self.isFinished) {
        [[NSRunLoop currentRunLoop]runMode:NSDefaultRunLoopMode beforeDate:[NSDate dateWithTimeIntervalSinceNow:.1]];
    }
    NSLog(@"能来吗？");
}

在最后添加的任务结束后结束死循环


- (void)otherMethod{
    for (int i = 0; i < 10; i ++) {
        NSLog(@"%s   %@",__FUNCTION__,[NSThread currentThread]);

    }
  //让上面方法中的死循环结束
   self.finished = YES;
}

### GCD

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/choogfunli/LCFMarkDown/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### NSOperation

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
