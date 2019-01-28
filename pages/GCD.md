### **GCD(Grand Central Dispatch)**
### 简介
> 是Apple开发的一个多核编程的解决方案；主要优化应用程序以及支持多核处理器以及其他对称多处理系统；是一个在线程池模式的基础上执行的并发任务。
### GCD作用
| 序号  | 作用 |
| :-- | :------------- |
| 1   |  GCD可用于多核的并行运算。      |
| 2   |  GCD会自动利用更多的CPU内核      |
| 3   |  GCD会自动管理线程的生命周期（创建线程、调度任务、销毁线程）|
### 任务执行的两种方式
- 同步执行（sync）
  - 同步添加到指定的队列，按照一定的顺序排队执行。
  - 只能在当前线程中执行任务，不具备开启新线程的能力。
- 异步执行（async）
  - 异步添加任务到指定的队列中，任务之间的各自执行。
  - 可以在新线程中执行任务，具备开启新线程的能力。
- 两者的区别
  - 是否等待队列的任务执行结束
  - 是否具备卡其新线程的能力

### 6种组合
| 区别 | 并发队列 | 串行队列 | 主队列 |
| :-- | :-- |:-- |:-- |
| 同步（sync） | 没有开启新线程，串行执行任务| 没有开启新线程，串行执行任务  | 主线程调用：死锁卡住不执行；其他线程调用：没有开启新线程，串行执行任务. |
| 异步（async） | 有开启新线程，并发执行任务| 有开启新线程（1条），串行执行任务  | 没有开启新线程，串行执行任务 |

##### 1、同步执行 + 并发队列
> 特点：在当前线程中执行任务，不会开启新线程，执行完一个任务，再执行下一个任务

```
- (void)syncConcurrent{
    // 打印当前线程
    NSLog(@"%@",[NSThread currentThread]);
    NSLog(@"syncConcurrent---start");
    dispatch_queue_t queue = dispatch_queue_create("fuzongjian.deltalpha.cn", DISPATCH_QUEUE_CONCURRENT);
    // 任务追加a
    dispatch_sync(queue, ^{
        // 耗时模拟
        [NSThread sleepForTimeInterval:2];
        NSLog(@"1----%@",[NSThread currentThread]);
    });
    // 任务追加b
    dispatch_sync(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2----%@",[NSThread currentThread]);
    });
    // 任务追加c
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i ++) {
            NSLog(@"3---%@",[NSThread currentThread]);
            [NSThread sleepForTimeInterval:2];
        }
    });
    NSLog(@"syncConcurrent---end");
}
```
`输出`

```
2019-01-28 14:27:05.634343+0800 Advance_Demo[12640:1278540] <NSThread: 0x60400006b980>{number = 1, name = main}
2019-01-28 14:27:05.634448+0800 Advance_Demo[12640:1278540] syncConcurrent---start
2019-01-28 14:27:07.635829+0800 Advance_Demo[12640:1278540] 1----<NSThread: 0x60400006b980>{number = 1, name = main}
2019-01-28 14:27:09.637230+0800 Advance_Demo[12640:1278540] 2----<NSThread: 0x60400006b980>{number = 1, name = main}
2019-01-28 14:27:09.637552+0800 Advance_Demo[12640:1278540] 3---<NSThread: 0x60400006b980>{number = 1, name = main}
2019-01-28 14:27:11.638312+0800 Advance_Demo[12640:1278540] 3---<NSThread: 0x60400006b980>{number = 1, name = main}
2019-01-28 14:27:13.639198+0800 Advance_Demo[12640:1278540] 3---<NSThread: 0x60400006b980>{number = 1, name = main}
2019-01-28 14:27:15.639801+0800 Advance_Demo[12640:1278540] syncConcurrent---end
```
##### 2、异步执行 + 并发队列
> 特点：可以开启多个线程，任务交替（同时）进行

```
- (void)asyncConcurrent{
    // 打印当前线程
    NSLog(@"%@",[NSThread currentThread]);
    NSLog(@"asyncConcurrent---start");
    dispatch_queue_t queue = dispatch_queue_create("fuzongjian.deltalpha.cn", DISPATCH_QUEUE_CONCURRENT);
    // 任务追加a
    dispatch_async(queue, ^{
        // 耗时模拟
        [NSThread sleepForTimeInterval:2];
        NSLog(@"1----%@",[NSThread currentThread]);
    });
    // 任务追加b
    dispatch_async(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2----%@",[NSThread currentThread]);
    });
    // 任务追加c
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            NSLog(@"3---%@",[NSThread currentThread]);
            [NSThread sleepForTimeInterval:2];
        }
    });
    NSLog(@"asyncConcurrent---end");
}
```
`输出`

```
2019-01-28 14:34:21.812533+0800 Advance_Demo[12719:1284017] <NSThread: 0x608000069240>{number = 1, name = main}
2019-01-28 14:34:21.812653+0800 Advance_Demo[12719:1284017] asyncConcurrent---start
2019-01-28 14:34:21.812764+0800 Advance_Demo[12719:1284017] asyncConcurrent---end
2019-01-28 14:34:21.812853+0800 Advance_Demo[12719:1284117] 3---<NSThread: 0x60c00006c240>{number = 3, name = (null)}
2019-01-28 14:34:23.818117+0800 Advance_Demo[12719:1284117] 3---<NSThread: 0x60c00006c240>{number = 3, name = (null)}
2019-01-28 14:34:23.818206+0800 Advance_Demo[12719:1284122] 2----<NSThread: 0x60800006dd80>{number = 4, name = (null)}
2019-01-28 14:34:23.818229+0800 Advance_Demo[12719:1284115] 1----<NSThread: 0x60c00006c6c0>{number = 5, name = (null)}
2019-01-28 14:34:25.820931+0800 Advance_Demo[12719:1284117] 3---<NSThread: 0x60c00006c240>{number = 3, name = (null)}
```
> 1. 可以从输出看到系统重新开了3个线程，并且任务是交替/同时执行的。
> 2. 异步执行具备开启线程的能力，并发队列可同时执行多个任务。

##### 3、同步执行 + 串行队列
> 特点：不会开启新的线程，只会在当前线程执行任务。

```
- (void)syncSerial{
    // 打印当前线程
    NSLog(@"%@",[NSThread currentThread]);
    NSLog(@"asyncConcurrent---start");
    dispatch_queue_t queue = dispatch_queue_create("fuzongjian.deltalpha.cn", DISPATCH_QUEUE_SERIAL);
    // 任务追加a
    dispatch_sync(queue, ^{
        // 耗时模拟
        [NSThread sleepForTimeInterval:2];
        NSLog(@"1----%@",[NSThread currentThread]);
    });
    // 任务追加b
    dispatch_sync(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2----%@",[NSThread currentThread]);
    });
    // 任务追加c
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i ++) {
            NSLog(@"3---%@",[NSThread currentThread]);
            [NSThread sleepForTimeInterval:2];
        }
    });
    NSLog(@"syncSerial---end");
}
```
`输出`
```
2019-01-28 15:28:54.338107+0800 Advance_Demo[12919:1315554] <NSThread: 0x604000074740>{number = 1, name = main}
2019-01-28 15:28:54.338245+0800 Advance_Demo[12919:1315554] asyncConcurrent---start
2019-01-28 15:28:56.339655+0800 Advance_Demo[12919:1315554] 1----<NSThread: 0x604000074740>{number = 1, name = main}
2019-01-28 15:28:58.341034+0800 Advance_Demo[12919:1315554] 2----<NSThread: 0x604000074740>{number = 1, name = main}
2019-01-28 15:28:58.341343+0800 Advance_Demo[12919:1315554] 3---<NSThread: 0x604000074740>{number = 1, name = main}
2019-01-28 15:29:00.342769+0800 Advance_Demo[12919:1315554] 3---<NSThread: 0x604000074740>{number = 1, name = main}
2019-01-28 15:29:02.344359+0800 Advance_Demo[12919:1315554] 3---<NSThread: 0x604000074740>{number = 1, name = main}
2019-01-28 15:29:04.344807+0800 Advance_Demo[12919:1315554] asyncConcurrent---end
```
##### 4、异步执行 + 串行队列
> 特点：会开启新线程，任务串行执行。

```
- (void)asyncSerial{
    // 打印当前线程
    NSLog(@"%@",[NSThread currentThread]);
    NSLog(@"asyncConcurrent---start");
    dispatch_queue_t queue = dispatch_queue_create("fuzongjian.deltalpha.cn", DISPATCH_QUEUE_SERIAL);
    // 任务追加a
    dispatch_async(queue, ^{
        // 耗时模拟
        [NSThread sleepForTimeInterval:2];
        NSLog(@"1----%@",[NSThread currentThread]);
    });
    // 任务追加b
    dispatch_async(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2----%@",[NSThread currentThread]);
    });
    // 任务追加c
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            NSLog(@"3---%@",[NSThread currentThread]);
            [NSThread sleepForTimeInterval:2];
        }
    });
    NSLog(@"asyncSerial---end");
}
```
`输出`
```
2019-01-28 15:49:57.942434+0800 Advance_Demo[13042:1329592] <NSThread: 0x604000074340>{number = 1, name = main}
2019-01-28 15:49:57.942547+0800 Advance_Demo[13042:1329592] asyncConcurrent---start
2019-01-28 15:49:57.942675+0800 Advance_Demo[13042:1329592] asyncSerial---end
2019-01-28 15:49:59.947089+0800 Advance_Demo[13042:1329696] 1----<NSThread: 0x604000463d40>{number = 3, name = (null)}
2019-01-28 15:50:01.952295+0800 Advance_Demo[13042:1329696] 2----<NSThread: 0x604000463d40>{number = 3, name = (null)}
2019-01-28 15:50:01.952618+0800 Advance_Demo[13042:1329696] 3---<NSThread: 0x604000463d40>{number = 3, name = (null)}
2019-01-28 15:50:03.954598+0800 Advance_Demo[13042:1329696] 3---<NSThread: 0x604000463d40>{number = 3, name = (null)}
2019-01-28 15:50:05.955595+0800 Advance_Demo[13042:1329696] 3---<NSThread: 0x604000463d40>{number = 3, name = (null)}
```
##### 5、同步执行 + 主队列
> 特点：会开启新线程，任务串行执行。

```
- (void)syncMain{
    // 打印当前线程
    NSLog(@"%@",[NSThread currentThread]);
    NSLog(@"asyncConcurrent---start");
    dispatch_queue_t queue = dispatch_get_main_queue();
    // 任务追加a
    dispatch_sync(queue, ^{
        // 耗时模拟
        [NSThread sleepForTimeInterval:2];
        NSLog(@"1----%@",[NSThread currentThread]);
    });
    // 任务追加b
    dispatch_sync(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2----%@",[NSThread currentThread]);
    });
    // 任务追加c
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i ++) {
            NSLog(@"3---%@",[NSThread currentThread]);
            [NSThread sleepForTimeInterval:2];
        }
    });
    NSLog(@"syncMain---end");
}
```
`输出`（直接奔溃）
```
2019-01-28 16:01:43.888486+0800 Advance_Demo[13138:1337247] <NSThread: 0x60000007b9c0>{number = 1, name = main}
2019-01-28 16:01:43.888594+0800 Advance_Demo[13138:1337247] asyncConcurrent---start
(lldb)
```
`原因：`
> 1. 主线程在执行syncMain方法，即相当于把syncMain任务放到了主线程的队列中。
> 2. 同步执行的会等待当前队列中的任务执行完毕，才会接着执行。当把任务a放在主队列中，任务a就在等待主线程处理完syncMain任务，而syncMain任务需要等待任务a执行完毕，才能接着执行。
> 3. 大家相互等待，所以就卡住，所以我们的任务执行不了，而且syncMain---end也没有打印。

`用这个方法调用：[NSThread detachNewThreadSelector:<#(nonnull SEL)#> toTarget:<#(nonnull id)#> withObject:<#(nullable id)#>]`

`输出`
```
2019-01-28 16:18:55.395620+0800 Advance_Demo[13213:1347088] <NSThread: 0x608000070e40>{number = 3, name = (null)}
2019-01-28 16:18:55.395733+0800 Advance_Demo[13213:1347088] asyncConcurrent---start
2019-01-28 16:18:57.399600+0800 Advance_Demo[13213:1346965] 1----<NSThread: 0x60400006e040>{number = 1, name = main}
2019-01-28 16:18:59.402140+0800 Advance_Demo[13213:1346965] 2----<NSThread: 0x60400006e040>{number = 1, name = main}
2019-01-28 16:18:59.403563+0800 Advance_Demo[13213:1346965] 3---<NSThread: 0x60400006e040>{number = 1, name = main}
2019-01-28 16:19:01.404801+0800 Advance_Demo[13213:1346965] 3---<NSThread: 0x60400006e040>{number = 1, name = main}
2019-01-28 16:19:03.408552+0800 Advance_Demo[13213:1346965] 3---<NSThread: 0x60400006e040>{number = 1, name = main}
2019-01-28 16:19:05.409044+0800 Advance_Demo[13213:1347088] syncMain---end
```
`从输出可以看出：`
> 1. 所有放在主队列中的任务，都会在主线程中执行。

`为什么现在不会卡住了呢？`
> 1. 主队列当前没有正在执行的任务。因为syncMain任务放在了其他线程里，而任务a、任务b、任务c都追加到主队列中。
##### 6、异步执行 + 主队列
> 特点：只在主线程中执行任务，执行完一个任务，再执行下一个任务。

```
- (void)asyncMain{
    // 打印当前线程
    NSLog(@"%@",[NSThread currentThread]);
    NSLog(@"asyncConcurrent---start");
    dispatch_queue_t queue = dispatch_get_main_queue();
    // 任务追加a
    dispatch_async(queue, ^{
        // 耗时模拟
        [NSThread sleepForTimeInterval:2];
        NSLog(@"1----%@",[NSThread currentThread]);
    });
    // 任务追加b
    dispatch_async(queue, ^{
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2----%@",[NSThread currentThread]);
    });
    // 任务追加c
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            NSLog(@"3---%@",[NSThread currentThread]);
            [NSThread sleepForTimeInterval:2];
        }
    });
    NSLog(@"asyncMain---end");
}
```

`输出`
```
2019-01-28 16:33:22.169881+0800 Advance_Demo[13316:1355800] <NSThread: 0x60400007aa40>{number = 1, name = main}
2019-01-28 16:33:22.169981+0800 Advance_Demo[13316:1355800] asyncConcurrent---start
2019-01-28 16:33:22.170076+0800 Advance_Demo[13316:1355800] asyncMain---end
2019-01-28 16:33:24.173958+0800 Advance_Demo[13316:1355800] 1----<NSThread: 0x60400007aa40>{number = 1, name = main}
2019-01-28 16:33:26.174348+0800 Advance_Demo[13316:1355800] 2----<NSThread: 0x60400007aa40>{number = 1, name = main}
2019-01-28 16:33:26.174647+0800 Advance_Demo[13316:1355800] 3---<NSThread: 0x60400007aa40>{number = 1, name = main}
2019-01-28 16:33:28.174915+0800 Advance_Demo[13316:1355800] 3---<NSThread: 0x60400007aa40>{number = 1, name = main}
2019-01-28 16:33:30.175247+0800 Advance_Demo[13316:1355800] 3---<NSThread: 0x60400007aa40>{number = 1, name = main}
```
`从输出可以看出`
>1. 虽然异步执行具备开启线程的能力，但是在主队列，所以所有任务都在主线程中执行。
### GCD线程间的通信
> 通常把一些耗时的操作放在其他线程，耗时操作完成会回到主线程，那么就会用到线程之间的通信。

```
- (void)communicationA{
    dispatch_queue_t global_queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_queue_t main_queue = dispatch_get_main_queue();
    dispatch_async(global_queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"1---%@",[NSThread currentThread]);
        }
        dispatch_async(main_queue, ^{
            [NSThread sleepForTimeInterval:2];
            NSLog(@"2---%@",[NSThread currentThread]);
        });
    });
}
```
`输出`
```
2019-01-28 16:56:00.338485+0800 Advance_Demo[13458:1370709] 1---<NSThread: 0x600000278680>{number = 3, name = (null)}
2019-01-28 16:56:02.341119+0800 Advance_Demo[13458:1370709] 1---<NSThread: 0x600000278680>{number = 3, name = (null)}
2019-01-28 16:56:04.341912+0800 Advance_Demo[13458:1370709] 1---<NSThread: 0x600000278680>{number = 3, name = (null)}
2019-01-28 16:56:06.343488+0800 Advance_Demo[13458:1370625] 2---<NSThread: 0x604000079f80>{number = 1, name = main}
```
### GCD其他方法
- **GCD栅栏方法**
> 有两组异步任务，第一组任务执行完成之后才能执行第二组任务。

![](/images/gcd_01.jpeg)
```
- (void)barrier{
    dispatch_queue_t queue = dispatch_queue_create("fuzongjian.deltalpha.cn", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"1---%@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"2---%@",[NSThread currentThread]);
        }
    });
    dispatch_barrier_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"barrier---1---%@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"barrier---2---%@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"barrier---3---%@",[NSThread currentThread]);
        }
    });
}
```
`输出`
```
2019-01-28 17:24:36.180135+0800 Advance_Demo[13631:1390987] 1---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:36.180148+0800 Advance_Demo[13631:1390989] 2---<NSThread: 0x60400007e000>{number = 3, name = (null)}
2019-01-28 17:24:38.184671+0800 Advance_Demo[13631:1390987] 1---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:38.184688+0800 Advance_Demo[13631:1390989] 2---<NSThread: 0x60400007e000>{number = 3, name = (null)}
2019-01-28 17:24:40.186860+0800 Advance_Demo[13631:1390987] 1---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:40.186860+0800 Advance_Demo[13631:1390989] 2---<NSThread: 0x60400007e000>{number = 3, name = (null)}
2019-01-28 17:24:42.190953+0800 Advance_Demo[13631:1390987] barrier---1---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:44.196517+0800 Advance_Demo[13631:1390987] barrier---1---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:46.200494+0800 Advance_Demo[13631:1390987] barrier---1---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:48.205759+0800 Advance_Demo[13631:1390987] barrier---2---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:48.205821+0800 Advance_Demo[13631:1390988] barrier---3---<NSThread: 0x60000007f180>{number = 5, name = (null)}
2019-01-28 17:24:50.210953+0800 Advance_Demo[13631:1390988] barrier---3---<NSThread: 0x60000007f180>{number = 5, name = (null)}
2019-01-28 17:24:50.210976+0800 Advance_Demo[13631:1390987] barrier---2---<NSThread: 0x60000007c980>{number = 4, name = (null)}
2019-01-28 17:24:52.214774+0800 Advance_Demo[13631:1390988] barrier---3---<NSThread: 0x60000007f180>{number = 5, name = (null)}
2019-01-28 17:24:52.214774+0800 Advance_Demo[13631:1390987] barrier---2---<NSThread: 0x60000007c980>{number = 4, name = (null)}
```
`由输出可以看出`
>dispatch_barrier_async函数追加的任务执行完毕后，异步队列才会恢复执行。

- **快速迭代方法： dispatch_apply**
> GCD提供给了快速迭代的函数dispatch_apply，类似for循环；dispatch_apply会按照指定的次数将指定的任务追加到指定的队列中，定等待全部任务执行结束。

```
- (void)apply{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i ++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"1---%@",[NSThread currentThread]);
        }
    });
    NSLog(@"appley---begin");
    dispatch_apply(6, queue, ^(size_t index) {
        NSLog(@"%zd---%@",index,[NSThread currentThread]);
    });
    NSLog(@"appley---end");
}
```
`输出`
```
2019-01-28 17:43:05.373426+0800 Advance_Demo[13741:1404042] appley---begin
2019-01-28 17:43:05.373671+0800 Advance_Demo[13741:1404162] 5---<NSThread: 0x60800006fb80>{number = 6, name = (null)}
2019-01-28 17:43:05.373672+0800 Advance_Demo[13741:1404089] 1---<NSThread: 0x600000074a40>{number = 4, name = (null)}
2019-01-28 17:43:05.373673+0800 Advance_Demo[13741:1404161] 4---<NSThread: 0x60800006f900>{number = 7, name = (null)}
2019-01-28 17:43:05.373679+0800 Advance_Demo[13741:1404090] 3---<NSThread: 0x600000074b00>{number = 5, name = (null)}
2019-01-28 17:43:05.373690+0800 Advance_Demo[13741:1404042] 2---<NSThread: 0x608000063780>{number = 1, name = main}
2019-01-28 17:43:05.373704+0800 Advance_Demo[13741:1404088] 0---<NSThread: 0x60c00026cb00>{number = 3, name = (null)}
2019-01-28 17:43:05.373935+0800 Advance_Demo[13741:1404042] appley---end
2019-01-28 17:43:07.373852+0800 Advance_Demo[13741:1404091] 1---<NSThread: 0x600000074fc0>{number = 8, name = (null)}
2019-01-28 17:43:09.376160+0800 Advance_Demo[13741:1404091] 1---<NSThread: 0x600000074fc0>{number = 8, name = (null)}
2019-01-28 17:43:11.380480+0800 Advance_Demo[13741:1404091] 1---<NSThread: 0x600000074fc0>{number = 8, name = (null)}
```
