五一复习OS，第二章信号量机制这块有些难，感觉以后也可能会忘，做笔记留作期末复习备用

1. 利用记录型信号量解决生产者——消费者问题

   可利用互斥信号量mutex实现诸进程对缓冲池的互斥使用；利用信号量empty和full分别表示缓冲池中空缓冲池和满缓冲池的数量。 假定这些生产者和消费者相互等效

```
Int i=0,out=0;
Item buffer[n];
Semaphore mutex=1,empty=n,full=0;
```

**mutex:生产者间，消费者间互斥使用缓冲区**

**empty:** **缓冲区的空闲容量**

**full:** **缓冲区的已占容量**

```
Void producer(){ 
 	do{
                    生产一个产品放入nextp;
                    wait(empty);
                    wait(mutex);
                    buffer[in]=nextp;
                    in=(in+1) % n;
                    signal(mutex);
                    signal(full);
	}while(TRUE)
}
```

```
Void consumer(){ 
    do{
           wait(full);
           wait(mutex);
           nextc =buffer[out];
           out =(out+1) mod n;
           signal(mutex);
           signal(empty);
           消费 nextc中的产品;                        
        }while(TRUE)
}
```

```
Void main(){
	cobegin
	    proceducer();
 	    consumer();
	coend
}
```

producer进程的wait(mutex)和signal(mutex)保证只有一个生产者在生产，同一个时刻只能有1个生产者进程。

同样，consumer进程的wait(mutex)和signal(mutex)保证只有一个消费者消费，同一个时刻只能有1个消费者进程。

empty用来标记库存剩余容量，每次执行生产者进程，容量-1；执行消费者进程，容量+1。

full用来标记产品数量，每次执行生产者进程，容量+1；执行消费者进程，容量-1。

producer进程中，如果empty减到了0，表示产品已经满库了，不能再生产，只能等待consumer进程。

consumer进程中，如果empty为n，表示没有产品，进程无法执行，只能等待producer进程。

当任何一个进程等待的时候，另一个进程都可以执行，不会产生死锁。

在每个程序中的多个wait操作顺序不能颠倒。应先执行对资源信号量的wait操作，再执行对互斥信号量的wait操作，否则可能引起进程死锁。

2. 利用AND信号量解决生产者——消费者问题

```
Void producer(){ 
 	do{
                    生产一个产品放入nextp;
                    …
                   Swait(empty, mutex);
                   buffer[in]=nextp;
                   in=(in+1) mod n;
                   Ssignal(mutex, full);
	}while(TRUE)
}
```

```
Void consumer(){ 
    do{
	Swait(full, mutex);
           nextc:=buffer[out];
           out=(out+1) mod n;
           Ssignal(mutex, empty);
           消费产品nextc;                        
      }while(TRUE)
   }
```

------

**2.5.3** **读者——写者问题**

1.利用记录型信号量解决读者——写者问题

```
semaphore rmutex=1, wmutex =1;
int readcount =0;
Wmutex: 读、写互斥；写、写互斥
Rmutex: 读间访问Readcount互斥
Readcount: 记录读者进程数
```

```
Void Reader(){
             do{
                wait(rmutex);
                if (Readcount==0) 
		wait(wmutex);
                Readcount ++;
                signal(rmutex);
                   …
                   读;
                   …
                wait(rmutex);
                Readcount - -;
                if (Readcount==0) 
		signal(wmutex);
                signal(rmutex);
            }while(TRUE);
}
```

```
Void writer(){
     do{
         wait(wmutex);
         写;
         signal(wmutex);
     }while(TRUE);
}
```

```
Void main(){
   cobegin
       reader();  writer();
Coend
}
```

仔细想了想，Reader进程的两对wait(rmutex)和signal(rmutex)，为了实现semaphore wmutex 和 int readcount的互斥访问，多个Reader进程，也只能有一个在访问临界区。

还有这题的Readcount很奇