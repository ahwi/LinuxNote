# c++高性能服务器网络编程



## 4. 回顾基础的Sockets API

### 1. Test TCP(ttcp)

ttcp是传统的测试ttcp性能的工具，主要测试两个机器之间tcp的吞吐量

**关心的指标：**

* Bandwidth(带宽),MB/s
* Throughput(吞吐量)、messages/s、queries/s(QPS)、transactions/s(TPS)
* Latency(延迟), milliseconds, percentiles(百分位延时)
* Utilization, percent, payload vs.carrier, goodpus vs.theory BW
* Overhead, eg.CPU usage, for compression and/or encryption

**为什么要选用TTCP作为例子：**

* 使用基本的socket APIs: socket, listen, bind, accept, connect, read/recv, write/send, shutdown, colose, etc
* 协议本身是有格式的
* 本身使用tcp实现的程序，有典型的行为
* 该协议可以用多种语言实现，可以根据吞吐量的不同，对比各个语言的开销
* 没有并发连接，比较简单



**The Protocol 协议：**

![1554908669053](assets/1554908669053.png)



**代码：**

* Straight Forward with blocking IO(使用阻塞IO)
  * muduo/examples/ace/ttcp/ttcp_blocking.cc(C with sockets API)
  * recipes/tcp/ttcp.cc(c++ with a thin wrapper)
  * muduo-examples-in-go/examples/ace/ttcp/ttcp.go(Go)
* Non-blocking IO with muduo library(非阻塞的IO)
  * muduo/examples/ace/ttcp/ttcp.cc
* None of above support concurrent connections（上面的都不支持并发连接）
  * Pretty easy to enable, thread-per-connection for first three



## 10. 时钟概述

网络编程实战的第二个例子：Roundtrip（基于UDP的例子）

作用：测量两个主机之间的时间差

内容：

* 计时
* 测量时间差
* udp和TCP的对比（从网络编程角度）



### 计时：

时钟 = 振荡器 + 计算器