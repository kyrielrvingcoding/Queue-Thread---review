#复习回顾一下队列和线程
## iOS中队列和线程的概念
&nbsp;队列分成主队列和全局队列，其他队列(自己手动创建(有串行和并行之分))。  
执行方法有两种:同步和异步
###执行方法的区分: 
    1.同步:当前队列的当前线程中，一个一个任务顺序执行，不具备开线程的能力  
    2.异步:开启一条或者多条线程，在开启的线程中并发的执行任务，具备开启线程的能力
###队列区分:
    1.串行队列:只具备开一条线程的能力。   
    2.并行队列:具备开多条线程的能力。    
####双方量量组个会产生多种情况:  
    1.串行 同步:  
        NSLog(@"开始");
        dispatch_queue_t queue = dispatch_queue_create("ddd", nil);
        for (int i = 0; i < 10; i ++ ) {
            dispatch_sync(queue, ^{
                NSLog(@"%@  打印：%d",[NSThread currentThread],i);
            });
        }
        NSLog(@"结束");
        结果:顺序打印，在主线程中。         
    2. 串行 异步: 
        NSLog(@"开始");
        dispatch_queue_t queue = dispatch_queue_create("ddd", nil);
        for (int i = 0; i < 10; i ++ ) {
            dispatch_async(queue, ^{
             NSLog(@"%@  打印：%d",[NSThread currentThread],i);
            });
     }
     NSLog(@"结束");

        结果:先输出开始，输出结束有时在其他打印的中间，有时直接跟在开始的后面。输出打印的线程的number是2，说明
        开启了一条线程，并且所以的打印都是在这条线程中顺序执行的。   
    3. 并行 同步:
        NSLog(@" %@开始",[NSThread currentThread]);
        dispatch_queue_t queue = dispatch_queue_create("ddd", DISPATCH_QUEUE_CONCURRENT);
        for (int i = 0; i < 10; i ++ ) {
           dispatch_sync(queue, ^{
     	      NSLog(@"%@  打印：%d",[NSThread currentThread],i);
          });
     }
        NSLog(@" %@结束",[NSThread currentThread]);
        结果:在主线程中，都按顺序执行下来，说明同步并不能开启线程   

    4.并发 异步
     NSLog(@" %@开始",[NSThread currentThread]);
        dispatch_queue_t queue = dispatch_queue_create("ddd", DISPATCH_QUEUE_CONCURRENT);
        for (int i = 0; i < 10; i ++ ) {
         dispatch_async(queue, ^{
             NSLog(@"%@  打印：%d",[NSThread currentThread],i);
            });
        }
        NSLog(@" %@结束",[NSThread currentThread]);

        结果:开启多条线程，在不同线程中执行任务。
###特殊情况
        1.主队列是一个串行队列，所以如果在主队列中，开启一个同步任务，就会造成线程死锁:
     - (void)ceshi {
        NSLog(@" %@开始",[NSThread currentThread]);
        //dispatch_queue_t queue = dispatch_queue_create("ddd", DISPATCH_QUEUE_CONCURRENT);
     __weak type(self)wkSelf = self; 
     dispatch_queue_t queue = dispatch_get_main_queue();
        [wkSelf  ceshi2];
        }
     NSLog(@" %@结束",[NSThread currentThread]);
        }
   	此时主队列中的主线程正在执行ceshi这个任务，但是在测试任务中，把一个同步任务ceshi2放到了主线程中。此时ceshi2
    任务正在等待ceshi这个任务完成，但是ceshi这个任务必须要等ceshi2完成，这样就会造成死锁。

###总结
    1.同步开线程，异步不开线程:(同步只能在队列的主线程中执行，异步不会在队列的主线程中执行)
    2.串行开一条，并行开多条。(串行顺序执行)

网址:http://www.cnblogs.com/dsxniubility/p/4296937.html