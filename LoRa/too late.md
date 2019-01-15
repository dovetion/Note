###  下行线程的代码逻辑

```c
预填写beacon包
while( !exit_sig && !quit_sig ) {
	构造PULL request包，并发送到服务器;	
	while( 收到ACK的时间 - 发送REQ的时间 < keepalive time) {
		尝试接收服务器数据(包括ack, 数据帧);	
		while(队列中的beacon数量 < 队列中应有的beacon数量) {
			计算beacon相应字段并入队;
		}	
		若没有收到服务器数据则 continue;		
		if( 收到的是ACK ){
			处理完成后 continue;
		}	
		// 否则一定是一个PULL_RESP
		解析数据，封装成txpkt，尝试入队。
		发送tx_ack;
	}
}
```



### 下行通行协议

```
5. Downstream protocol
-----------------------

### 5.1. Sequence diagram ###

	+---------+                                                    +---------+
	| Gateway |                                                    | Server  |
	+---------+                                                    +---------+
	     | -----------------------------------\                         |
	     |-| Every N seconds (keepalive time) |                         |
	     | ------------------------------------                         |
	     |                                                              |
	     | PULL_DATA (token Y, MAC@)                                    |
	     |------------------------------------------------------------->|
	     |                                                              |
	     |                                           PULL_ACK (token Y) |
	     |<-------------------------------------------------------------|
	     |                                                              |

	+---------+                                                    +---------+
	| Gateway |                                                    | Server  |
	+---------+                                                    +---------+
	     |      ------------------------------------------------------\ |
	     |      | Anytime after first PULL_DATA for each packet to TX |-|
	     |      ------------------------------------------------------- |
	     |                                                              |
	     |                            PULL_RESP (token Z, JSON payload) |
	     |<-------------------------------------------------------------|
	     |                                                              |
	     | TX_ACK (token Z, JSON payload)                               |
	     |------------------------------------------------------------->|
	     
```

服务器发送PULL_RESP的条件是“Anytime after first PULL_DATA for each packet to TX ”，我的理解就是“对每个下行包，在收到PULL DATA之后返回PULL RESP”

这么做的原因协议解释为

> This data exchange is initialized by the gateway because it might be 
> impossible for the server to send packets to the gateway if the gateway is 
> behind a NAT.

>When the gateway initialize the exchange, the network route towards the 
>server will open and will allow for packets to flow both directions.
>The gateway must periodically send PULL_DATA packets to be sure the network 
>route stays open for the server to be used at any time.

![image-20181018113250619](../../../var/folders/10/w42hb35s38q7tg1ynvgvm_180000gn/T/abnerworks.Typora/image-20181018113250619.png)

keepalive time为5秒的测试结果图

代码里默认keepalive time是5秒，在平时测试的过程是keepalive time是1秒，对比代码结构就会发现如下问题。

![image-20181018111042510](/var/folders/10/w42hb35s38q7tg1ynvgvm_180000gn/T/abnerworks.Typora/image-20181018111042510.png)

虽然网关所有部分的处理耗时基本都在1ms内完成，但是由于keepalive time设置，即使发送成功，程序一直会在第2个while循环里。下行线程在剩余的时间里被卡死。

解决方案

* 改小keepalive time  (已完成) ：测试表明改到800ms后，下行TOO LATE拒绝完全消失。但网关和服务器之间通信会更加频繁。
* 更改下行通信协议并更改实现：
  * keepalive机制可以由单独的线程去完成，更改网关代码
  * 服务器不等待PULL req即发送PULL resp，在不同局域网下可能会产生问题，服务器更改代码。