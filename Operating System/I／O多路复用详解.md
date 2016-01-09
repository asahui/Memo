##来源 
[I/O多路复用详解](http://www.cnblogs.com/gmth/p/3214168.html)  
[说说IO（二）- IO模型](http://pengjiaheng.iteye.com/blog/847615)

##一、I/O模型 

首先，输入操作一般包含两个步骤： 

1. 等待数据准备好（waiting for data to be ready）。对于一个套接口上的操作，这一步骤关系到数据从网络到达，并将其复制到内核的某个缓冲区。 

2. 将数据从内核缓冲区复制到进程缓冲区（copying the data from the kernel to the process）。 

##二、I/O模型类型 

其次了解一下五种I/O模型： 

* blocking I/O 
* nonblocking I/O 
* I/O multiplexing (select and poll) 
* signal driven I/O (SIGIO) 
* asynchronous I/O (the POSIX aio_functions) 

---

###1、阻塞I/O模型 

　　最广泛的模型是阻塞I/O模型，默认情况下，所有套接口都是阻塞的。 

　　进程调用recvfrom系统调用，整个过程是阻塞的，直到数据复制到进程缓冲区时才返回（当然，系统调用被中断也会返回）。 

###2、非阻塞I/O模型 

　　当我们把一个套接口设置为非阻塞时，就是在告诉内核，当请求的I/O操作无法完成时，不要将进程睡眠，而是返回一个错误。当数据没有准备好时，内核立即返回EWOULDBLOCK错误，第四次调用系统调用时，数据已经存在，这时将数据复制到进程缓冲区中。这其中有一个操作时轮询（polling）。 

###3、I/O复用模型 

　　此模型用到select和poll函数，这两个函数也会使进程阻塞，select先阻塞，有活动套接字才返回，但是和阻塞I/O不同的是，这两个函数可以同时阻塞多个I/O操作，而且可以同时对多个读操作，多个写操作的I/O函数进行检测，直到有数据可读或可写。select被调用后，进程会被阻塞，内核监视所有select负责的socket，当有任何一个socket的数据准备好了，select就会返回套接字可读，我们就可以调用recvfrom处理数据。 

###4、信号驱动I/O模型（signal driven I/O， SIGIO） 

　　首先我们允许套接口进行信号驱动I/O,并安装一个信号处理函数，进程继续运行并不阻塞。当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用I/O操作函数处理数据。当数据报准备好读取时，内核就为该进程产生一个SIGIO信号。我们随后既可以在信号处理函数中调用recvfrom读取数据报，并通知主循环数据已准备好待处理，也可以立即通知主循环，让它来读取数据报。无论如何处理SIGIO信号，这种模型的优势在于等待数据报到达(第一阶段)期间，进程可以继续执行，不被阻塞。免去了select的阻塞与轮询，当有活跃套接字时，由注册的handler处理。 

###5、异步I/O模型(AIO, asynchronous I/O)

　　进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。 

　　这个模型工作机制是：告诉内核启动某个操作，并让内核在整个操作(包括第二阶段，即将数据从内核拷贝到进程缓冲区中)完成后通知我们。 

这种模型和前一种模型区别在于：信号驱动I/O是由内核通知我们何时可以启动一个I/O操作，而异步I/O模型是由内核通知我们I/O操作何时完成。 


##三、下面是以上五种模型的比较
可以看出，越往后，阻塞越少，理论上效率也是最优。  
5种模型的比较比较清晰了，剩下的就是把select, epoll, iocp, kqueue按号入座那就OK了。 

select和iocp分别对应第3种与第5种模型，那么epoll与kqueue呢？其实也于select属于同一种模型，只是更高级一些，可以看作有了第4种模型的某些特性，如callback机制。 

那么，**为什么epoll,kqueue比select高级？**

答案是，他们无轮询。因为他们用callback取代了。想想看，当套接字比较多的时候，每次select()都要通过遍历FD_SETSIZE个Socket来完成调度,不管哪个Socket是活跃的,都遍历一遍。这会浪费很多CPU时间。如果能给套接字注册某个回调函数，当他们活跃时，自动完成相关操作，那就避免了轮询，这正是epoll与kqueue做的。 

**windows or *nix （IOCP or kqueue/epoll）？**

诚然，Windows的IOCP非常出色，目前很少有支持asynchronous I/O的系统，但是由于其系统本身的局限性，大型服务器还是在UNIX下。而且正如上面所述，kqueue/epoll 与 IOCP相比，就是多了一层从内核copy数据到应用层的阻塞，从而不能算作asynchronous I/O类。但是，这层小小的阻塞无足轻重，kqueue与epoll已经做得很优秀了。 

**提供一致的接口，IO Design Patterns**

实际上，不管是哪种模型，都可以抽象一层出来，提供一致的接口，广为人知的有ACE,Libevent这些，他们都是跨平台的，而且他们自动选择最优的I/O复用机制，用户只需调用接口即可。说到这里又得说说2个设计模式，Reactor and Proactor。有一篇[经典文章](http://www.artima.com/articles/io_design_patterns.html)值得阅读，Libevent是Reactor模型，ACE提供Proactor模型。实际都是对各种I/O复用机制的封装。 

**Java nio包是什么I/O机制？**

我曾天真的认为java nio封装的是IOCP。。现在可以确定，目前的java本质是select()模型，可以检查/jre/bin/nio.dll得知。至于java服务器为什么效率还不错。。我也不得而知，可能是设计得比较好吧。。-_-。 


##总结

总结一些重点： 

1. 只有IOCP是asynchronous I/O，其他机制或多或少都会有一点阻塞。 
2. select低效是因为每次它都需要轮询。但低效也是相对的，视情况而定，也可通过良好的设计改善 
3. epoll, kqueue是Reacor模式，IOCP是Proactor模式。 
4. java nio包是select模型。。
