#  进程如何阻塞？进程阻塞为什么不消耗CPU？

要想明白进程如何阻塞，阻塞为什么不消耗CPU，就要先明白

- 计算机是如何接受数据的
- 计算机如何知道什么时候要接收数据

## 1. 计算机是如何接收数据的

**从网卡接收数据说起：**

下边是一个典型的计算机结构图，计算机由 CPU、存储器(内存)与网络接口等部件组成，先从硬件的角度看计算机怎样接收网络数据。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413101434699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDg0NDA4OQ==,size_16,color_FFFFFF,t_70)
下图展示了网卡接收数据的过程：

- 在 1 阶段，网卡收到网线传来的数据。
- 经过 2 阶段的硬件电路的传输。
- 最终 3 阶段将数据写入到内存中的某个地址上。

这个过程涉及到 DMA 传输、IO 通路选择等硬件有关的知识，但我们只需知道：网卡会把接收到的数据写入内存。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413101530539.png)
网卡接收数据的过程：
通过硬件传输，网卡接收的数据存放到内存中，操作系统就可以去读取它们。

## 2. 计算机如何知道要接受数据？

**答：中断**

- 计算机执行程序时，会有优先级的需求。比如，当计算机收到断电信号时，它应立即去保存数据，保存数据的程序具有较高的优先级(电容可以保存少许电量，供 CPU 运行很短的一小段时间)。
- 一般而言，由硬件产生的信号需要 CPU 立马做出回应，不然数据可能就丢失了，所以它的优先级很高。
- CPU 理应中断掉正在执行的程序，去做出响应;当 CPU 完成对硬件的响应后，再重新执行用户程序。

中断的过程如下图，它和函数调用差不多，只不过函数调用是事先定好位置，而中断的位置由“信号”决定。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413102039724.png)

中断程序调用

以键盘为例，当用户按下键盘某个按键时，键盘会给 CPU 的中断引脚发出一个高电平，CPU 能够捕获这个信号，然后执行键盘中断程序。

下图展示了各种硬件通过中断与 CPU 交互的过程：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413102128826.png)
现在可以回答“如何知道接收了数据?”这个问题了：当网卡把数据写入到内存后，网卡向 CPU 发出一个中断信号，操作系统便能得知有新数据到来，再通过网卡中断程序去处理数据。

## 3. 进程阻塞为什么不占用 CPU 资源?

从操作系统进程调度的角度来看数据接收。阻塞是进程调度的关键一环，指的是进程在等待某事件(如接收到网络数据)发生之前的等待状态，Recv、Select 和 Epoll 都是阻塞方法。

下边分析一下进程阻塞为什么不占用 CPU 资源?为简单起见，我们从普通的 Recv 接收开始分析，先看看下面代码：

```java
//创建socket 
int s = socket(AF_INET, SOCK_STREAM, 0);    
//绑定 
bind(s, ...) 
//监听 
listen(s, ...) 
//接受客户端连接 
int c = accept(s, ...) 
//接收客户端数据 
recv(c, ...); 
//将数据打印出来 
printf(...) 
123456789101112
```

这是一段最基础的网络编程代码，先新建 Socket 对象，依次调用 Bind、Listen 与 Accept，最后调用 Recv 接收数据。

Recv 是个阻塞方法，当程序运行到 Recv 时，它会一直等待，直到接收到数据才往下执行。那么阻塞的原理是什么?

### 3.1 工作队列
操作系统为了支持多任务，实现了进程调度的功能，会把进程分为“运行”和“等待”等几种状态。

运行状态是进程获得 CPU 使用权，正在执行代码的状态;等待状态是阻塞状态，比如上述程序运行到 Recv 时，程序会从运行状态变为等待状态，接收到数据后又变回运行状态。

操作系统会分时执行各个运行状态的进程，由于速度很快，看上去就像是同时执行多个任务。

下图的计算机中运行着 A、B 与 C 三个进程，其中进程 A 执行着上述基础网络程序，一开始，这 3 个进程都被操作系统的工作队列所引用，处于运行状态，会分时执行。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413102456372.png)
工作队列中有 A、B 和 C 三个进程

### 3.2 等待队列

当进程 A 执行到创建 Socket 的语句时，操作系统会创建一个由文件系统管理的 Socket 对象(如下图)。

![image-20220525201344148](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20220525201344148.png)

创建 Socket

这个 Socket 对象包含了发送缓冲区、接收缓冲区与等待队列等成员。等待队列是个非常重要的结构，它指向所有需要等待该 Socket 事件的进程。

当程序执行到 Recv 时，操作系统会将进程 A 从工作队列移动到该 Socket 的等待队列中(如下图)。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413102636987.png)
Socket 的等待队列

由于工作队列只剩下了进程 B 和 C，依据进程调度，CPU 会轮流执行这两个进程的程序，不会执行进程 A 的程序。所以进程 A 被阻塞，不会往下执行代码，也不会占用 CPU 资源。

注：操作系统添加等待队列只是添加了对这个“等待中”进程的引用，以便在接收到数据时获取进程对象、将其唤醒，而非直接将进程管理纳入自己之下。上图为了方便说明，直接将进程挂到等待队列之下。

### 3.3 唤醒进程

当 Socket 接收到数据后，操作系统将该 Socket 等待队列上的进程重新放回到工作队列，该进程变成运行状态，继续执行代码。

同时由于 Socket 的接收缓冲区已经有了数据，Recv 可以返回接收到的数据。

### 3.4 内核接收网络数据全过程

这一步，贯穿网卡、中断与进程调度的知识，叙述阻塞 Recv 下，内核接收数据的全过程。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413102901283.png)
内核接收数据全过程

如上图所示，进程在 Recv 阻塞期间：

- 计算机收到了对端传送的数据(步骤 ①)
- 数据经由网卡传送到内存(步骤 ②)
- 然后网卡通过中断信号通知 CPU 有数据到达，CPU 执行中断程序(步骤 ③)

此处的中断程序主要有两项功能，先将网络数据写入到对应 Socket 的接收缓冲区里面(步骤 ④)，再唤醒进程 A(步骤 ⑤)，重新将进程 A 放入工作队列中。

唤醒进程的过程如下图所示：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210413103025205.png)

### 3.5 进程阻塞为什么不消耗CPU？

答：进程执行的过程的确是在内核CPU中执行的，会消耗CPU。但是当进程阻塞变成等待态的时候，会被加入该socket的等待队列中，并不会出现在内核中了，而是在socket的等待队列中“等待”。当进程被操作系统唤醒后，又会被加入到工作队列中。

## 总结

![image-20220704121255823](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img//img/20220704121255.png)



接收数据的步骤

- 网卡将数据帧DMA到内存的**RingBuffer的接收队列**中，然后向CPU发起中断通知
- CPU响应中断请求，调用网卡启动时注册的中断处理函数
- 中断处理函数几乎没干啥，就发起了软中断请求
- 内核线程ksoftirqd线程发现有软中断请求到来，先关闭硬中断
- ksoftirqd线程开始调用驱动的poll函数收包
- poll函数将收到的包送到协议栈注册的ip_rcv函数中
- ip_rcv函数再讲包送到udp_rcv函数中（对于tcp包就送到tcp_rcv）
- 再将数据送到socket的缓冲区
- 唤醒socket等待队列中的进程，将数据拷贝到用户空间



发送数据步骤

- **应用层数据拷贝到 Socket 发送缓冲区中**：应用程序会调用 Socket 发送数据包的接口，由于这个是**系统调用**（send），所以会从用户态陷入到内核态中的 Socket 层  找到对应的socket ，把数据拷贝到socket的发送队列
- **网络协议栈**从 Socket 发送缓冲区中取出数据包 一层一层 处理
- **进入到RingBuffer的发送队列**
- 会触发软中断告诉网卡驱动程序，这里有新的网络包需要发送，最后驱动程序通过 DMA，从发包队列中读取网络包，将其放入到硬件网卡的队列中，随后物理网卡再将它发送出去。
- **当数据发送完成**，工作还没有结束
- 触发中断，清理RingBuffer

·