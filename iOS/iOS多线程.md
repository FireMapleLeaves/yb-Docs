### 任务

任务是我们需要执行的操作，在GCD中，我们通常将需要执行的操作放在block中。执行任务有两种方式：同步和异步。

- 同步：同步表示任务调用一旦开始，那么调用者必须等到任务返回之后，才能进行后续操作。同步任务是在当前线程中执行，不会开辟新的线程。
- 异步：异步则表示任务一调用就会立即返回，不会阻碍调用者执行下一步操作。而任务实际是在新开辟的线程中执行。

> **因此，同步和异步最大的区别就是：是否具有开辟新线程的能力，以及是否会阻塞当前线程**


###队列

队列主要分为两种：串行队列和并发队列

- 串行队列：表示同一时间只会有一个任务执行，执行完毕后才会执行下一个任务。**串行队列只会开启一个线程执行任务。**
- 并发队列：表示同一时间有多个任务在执行。这也就意味着并发队列可以开启多个线程同时执行任务。

串行队列和并发队列任务的插入方式都遵循FIFO（先进先出）原则，也就是新的任务总会插入到队列的末尾，但是串行队列中先进入队列的任务会先执行，并且等到任务执行完之后才会执行后面的任务。而并发队列则会同时执行队列中的多个任务，并且任务之间不会相互等待，任务的执行顺序和执行过程也不可预测。

### 串行队列中同步异步添加任务

```objective-c
    dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue", DISPATCH_QUEUE_SERIAL);
    NSLog(@"任务1-----%@",[NSThread currentThread]);
    dispatch_async(serialQueue, ^{
        NSLog(@"任务2开始执行-----%@",[NSThread currentThread]);
        sleep(2);
        NSLog(@"任务2执行完毕-----%@",[NSThread currentThread]);
    });
    NSLog(@"任务3-----%@",[NSThread currentThread]);
    dispatch_sync(serialQueue, ^{
        NSLog(@"任务4开始执行-----%@",[NSThread currentThread]);
        sleep(2);
        NSLog(@"任务4执行完毕-----%@",[NSThread currentThread]);
    });
    NSLog(@"任务5-----%@",[NSThread currentThread]);
```

上述代码，打印情况如下，先打印下面3条

```
任务1-----<_NSMainThread: 0x600000aac440>{number = 1, name = main}
任务3-----<_NSMainThread: 0x600000aac440>{number = 1, name = main}
任务2开始执行-----<NSThread: 0x600000ae14c0>{number = 6, name = (null)}
```

2s后，打印

```
任务2执行完毕-----<NSThread: 0x600001e66900>{number = 6, name = (null)}
任务4开始执行-----<_NSMainThread: 0x600001e64140>{number = 1, name = main}
```

再2s后，打印

```
任务4执行完毕-----<_NSMainThread: 0x600001e64140>{number = 1, name = main}
任务5-----<_NSMainThread: 0x600001e64140>{number = 1, name = main}
```

分析下原因：

- 异步任务调用后立即返回，所以任务3先于任务2打印；
- **同步任务会阻塞当前线程，执行完同步任务才会执行后面的任务**
- 串行队列中的任务遵循FIFO（先进先出）原则，先添加进去的任务先执行，并且前面的任务执行完成之后才会执行后面的任务（先执行任务2，后执行任务4）。
- 只有异步操作（任务2）会开辟新线程。

如果把任务4改为异步执行，则打印结果为

```
2022-06-28 21:01:54.886935+0800  任务1-----<_NSMainThread: 0x6000038541c0>{number = 1, name = main}
2022-06-28 21:01:54.887118+0800  任务3-----<_NSMainThread: 0x6000038541c0>{number = 1, name = main}
2022-06-28 21:01:54.887147+0800  任务2开始执行-----<NSThread: 0x60000381a740>{number = 7, name = (null)}
2022-06-28 21:01:54.887213+0800  任务5-----<_NSMainThread: 0x6000038541c0>{number = 1, name = main}

-----2s之后-----

2022-06-28 21:01:56.888800+0800  任务2执行完毕-----<NSThread: 0x60000381a740>{number = 7, name = (null)}
2022-06-28 21:01:56.889259+0800  任务4开始执行-----<NSThread: 0x60000381a740>{number = 7, name = (null)}

-----2s之后-----

2022-06-28 21:01:58.894791+0800  任务4执行完毕-----<NSThread: 0x60000381a740>{number = 7, name = (null)}
```

###并发队列同步异步添加任务

```objc
    //创建并发队列
    dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"任务1-----%@",[NSThread currentThread]);
    dispatch_async(concurrentQueue, ^{
        NSLog(@"任务2开始执行-----%@",[NSThread currentThread]);
        sleep(2);
        NSLog(@"任务2执行完毕-----%@",[NSThread currentThread]);
    });
    NSLog(@"任务3-----%@",[NSThread currentThread]);
    dispatch_sync(concurrentQueue, ^{
        NSLog(@"任务4开始执行-----%@",[NSThread currentThread]);
        sleep(2);
        NSLog(@"任务4执行完毕-----%@",[NSThread currentThread]);
    });
    NSLog(@"任务5-----%@",[NSThread currentThread]);
```

输出为

```
2022-06-28 21:18:40.982251+0800  任务1-----<_NSMainThread: 0x6000004a8000>{number = 1, name = main}
2022-06-28 21:18:40.982414+0800  任务3-----<_NSMainThread: 0x6000004a8000>{number = 1, name = main}
2022-06-28 21:18:40.982439+0800  任务2开始执行-----<NSThread: 0x6000004b6ac0>{number = 7, name = (null)}
2022-06-28 21:18:40.982519+0800  任务4开始执行-----<_NSMainThread: 0x6000004a8000>{number = 1, name = main}

-----2s之后-----

2022-06-28 21:18:42.983807+0800  任务4执行完毕-----<_NSMainThread: 0x6000004a8000>{number = 1, name = main}
2022-06-28 21:18:42.984252+0800  任务5-----<_NSMainThread: 0x6000004a8000>{number = 1, name = main}
2022-06-28 21:18:42.985715+0800  任务2执行完毕-----<NSThread: 0x6000004b6ac0>{number = 7, name = (null)}

```

所以，可以得出结论

- 异步任务不会阻塞当前线程，所以任务1和任务3先执行，任务2后执行
- 并发队列中多个任务可以同时执行，因此任务2和任务4并发执行
- 异步任务会开辟新的线程，同步任务会在当前线程执行。因此任务2在子线程中执行，任务4在主线程中执行。
- 同步任务阻塞当前线程，因此任务4执行完成之后才会执行任务5

如果把任务4改为异步添加，那么打印结果为

```
2022-06-28 21:24:12.111769+0800  任务1-----<_NSMainThread: 0x600001680000>{number = 1, name = main}
2022-06-28 21:24:12.111907+0800  任务3-----<_NSMainThread: 0x600001680000>{number = 1, name = main}
2022-06-28 21:24:12.111927+0800  任务2开始执行-----<NSThread: 0x6000016c86c0>{number = 7, name = (null)}
2022-06-28 21:24:12.112003+0800  任务5-----<_NSMainThread: 0x600001680000>{number = 1, name = main}
2022-06-28 21:24:12.112019+0800  任务4开始执行-----<NSThread: 0x6000016da580>{number = 5, name = (null)}

------2s之后-------------

2022-06-28 21:24:14.112883+0800  任务2执行完毕-----<NSThread: 0x6000016c86c0>{number = 7, name = (null)}
2022-06-28 21:24:14.112883+0800  任务4执行完毕-----<NSThread: 0x6000016da580>{number = 5, name = (null)}
```

也就是说

|                 | 串行队列（手动创建）          | 主队列                        | 并发队列                      |
| --------------- | ----------------------------- | ----------------------------- | ----------------------------- |
| 同步任务(sync)  | ●不会开辟新线程 ●串行执行任务 | 产生死锁                      | ●不会开辟新线程 ●串行执行任务 |
| 异步任务(async) | ●开辟新线程 ●串行执行任务     | ●不会开辟新线程 ●串行执行任务 | ●开辟新线程 ●并发执行任务     |

### 死锁

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"任务1");
    dispatch_sync(dispatch_get_main_queue(), ^{ //block1
        NSLog(@"任务2");
    });
    NSLog(@"任务3");
}
```

上面这种情况下，`viewDidLoad`先被添加到主队列中执行，执行过程中，block1被添加到主队列中，因此，要执行block1，需要先执行完`viewDidLoad`，但是执行完`viewDidLoad`需要先执行完block1以及任务3，导致两者互相等待，形成死锁。

```objc
//创建串行队列
dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue", DISPATCH_QUEUE_SERIAL);
dispatch_async(serialQueue, ^{  //此处称为block1
    NSLog(@"任务1");
    dispatch_sync(serialQueue, ^{ //此处称为block2
        NSLog(@"任务2");
    });
    NSLog(@"任务3");
});
```

类似的，上面这种情况，通过dispatch_async添加异步任务时会开启新的线程，所以此时block1中的任务是在子线程中执行，同时因为是在串行队列中增加的异步任务，所以block1会被添加到串行队列中去，并且在队首。在子线程中执行block1中的方法，先执行任务1，然后执行dispatch_sync方法，此时会向串行队列中增加同步任务block2,并且需要等到block2执行完成之后才会执行任务3。此时在串行队列中存在两个任务，block1和block2，block2想要执行，就必须等待block1执行完，而block1想要执行完，必须要执行完block2以及任务3，但是任务3想要执行，又必须执行完block2，因此block1在等待block2执行完，block2又在等待block1执行完，从而造成死锁。

####栅栏方法：dispatch_barrier_async

栅栏方法主要是在多组操作之间增加栅栏，从而分割多组操作，使得各组操作之间顺序执行。例如：有两组操作，需要执行完第一组操作之后再执行第二组操作，此时就需要用到dispatch_barrier_async，**前提是必须要添加到同一个队列中**，代码如下：

```objc
    //创建并发队列
    dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
    //任务组一
    for (int i = 0; i < 5; i++) {
        dispatch_async(concurrentQueue, ^{
            [NSThread sleepForTimeInterval:1];
            NSLog(@"执行组一任务%d",i);
        });
    }
    //栅栏方法
    dispatch_barrier_sync(concurrentQueue, ^{
        NSLog(@"栅栏方法");
    });
    //任务组二
    for (int i = 0; i < 5; i++) {
        dispatch_async(concurrentQueue, ^{
            [NSThread sleepForTimeInterval:1];
            NSLog(@"执行组二任务%d",i);
        });
    }
```

打印结果

```
2022-06-28 21:51:00.711511+0800  执行组一任务4
2022-06-28 21:51:00.711511+0800  执行组一任务1
2022-06-28 21:51:00.711511+0800  执行组一任务0
2022-06-28 21:51:00.711516+0800  执行组一任务3
2022-06-28 21:51:00.711511+0800  执行组一任务2
2022-06-28 21:51:00.712209+0800  栅栏方法
2022-06-28 21:51:01.713051+0800  执行组二任务1
2022-06-28 21:51:01.713051+0800  执行组二任务0
2022-06-28 21:51:01.713051+0800  执行组二任务2
2022-06-28 21:51:01.713082+0800  执行组二任务3
2022-06-28 21:51:01.713174+0800  执行组二任务4
```

关于`dispatch_barrier_async`和`dispatch_barrier_sync`的区别

`dispatch_barrier_sync`会等待自己block内部操作执行完后，再执行后续操作；而`dispatch_barrier_async`不会等待自己的block执行完毕，就会将后面的操作加入到队列中。

###Dispatch_Group