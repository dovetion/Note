# 使用TCP通信的LoRa网关

葛鑫 2019-4-7

[TOC]

## 1. 使用TCP的理由与缺点

在 Semtech 网关源代码中，在网关-服务器通信中使用了UDP协议。这种无状态协议不需要有建立连接的过程，协议层面没有确认（**ACK**） ，交互次数更少。

与此同时带来的问题是，由于网关和服务器可能处于不同子网，网关侧可能有NAT，因此，必须由网关主动发起通信，所以网关需要周期性发起通信请求，然后服务器才能向相关发送下行。

如果使用TCP，建立双向连接之后，服务器不用等待，网关与服务器有需求就可以发，并且通信是可靠的。

带来的一个问题是，对于网关来说，如果服务器关闭，死机。那么网关需要特殊处理这种情况，比如重新建立连接。



## 2. 使用UDP的网关-服务器上行通信协议

### 2.1 网关-服务器上行通信协议

```sequence
Note left of Gateway: when 1-N RF pkt are received
Gateway->Server: PUSH_DATA(token X, GW MAC, JSON Payload)
Server-Gateway: PUSH_ACK(token x)
Note right of Server: process packets after ack
```

* UDP没有ACK机制，于是上层自己实现了ACK。使用TCP可以删除ACK，**但是第一个版本先不改这个地方。**

### 2.2 网关上行代码框架

```c
void thread_up( void ) {
	setsockopt(); //设置socket超时时间
    while(1) {
    	lgw_receive(); //收包
        //////////////////////////
        ///LoRa包->JSON Payload///
        //////////////////////////
        if (sendto(JSON Payload) < 0 ) //发送失败
            printf("(client)sending error.\n");
        else{
            记录发送时间；
      
        for(int i=0;i<2;i++) {
        	recvfrom();
            if( TIMEOUT )
                continue;
            else if(NOT ACK WANTED)
                continue;
            else{
                记录ACL;
                break;
            }
        }
    }
}
```



### 2.3 使用TCP的网关上行代码框架

```C
void thread_up( void ) {
    /*
    使用TCP连接，若服务器被杀，掉线，客户端再进行尝试发送数据时，系统会发出一个SIGPIPE信号给进程，告诉进程连接断开，不要再写。根据信号的默认处理规则，SIGPIPE信号的默认动作是terminate。所以为防止网关程序被终止，需要忽略SIGPIPE信号。
    */
    signal(SIGPIPE, SIG_IGN);
    while(1) {
    	lgw_receive(); //收包
        //////////////////////////
        ///LoRa包->JSON Payload///
        //////////////////////////
        if (sendto(JSON Payload) < 0 ) { //发送失败
            printf("(client)sending error.\n");
            /*
            断开了连接，需要重新尝试连接服务器.
            只尝试重连一次,此次上行放弃发送。
            */
            TCP_reconnect();   
        }
        else{
            记录发送时间；
      
        for(int i=0;i<2;i++) {
        	recvfrom();
            if( TIMEOUT )
                continue;
            else if(NOT ACK WANTED)
                continue;
            else{
                记录ACK;
                break;
            }
        }
    }
}
```

* 需要忽略`SIGPIP`信号，防止服务器断开后网关进程被杀
* 需要进行断线重连





##3. 使用UDP的网关-服务器下行通信协议

### 3.1 网关-服务器上行通信协议

```sequence
Note left of  Gateway: Every N seconds(keepalive)
Gateway->Server: PULL_REQ(token Y, MAC@)
Server->Gateway: PULL_ACK(token Y)
Note right of Server: Anytime after first PULL_DATA
Server->Gateway: PULL_RESP(token Z, JSON payload)
Gateway->Server:TX_ACK(token Z, JSON payload)
```



### 3.2 网关下行代码框架

```C
void thread_down(void){
	变量初始化
    设置socket
    填充buff_req
    while(1) {
    	填充buff_req里的token
        把buff_req发给服务器，记录send_time
        recv_time = send_time;
        while(recv_time - send_time < keepalive_time){
            更新recv_time
        	msg_len = recv(buff_down); //从服务器端接受buff_down
            
            while (beacon_loop && (beacon_period != 0)) {
        		处理beacon内容;
            }
            
            if(msg_len == -1)
            	continue;
            else if(无法辨别的下行内容)
            	continue;
            else if (下行内容是 PULL_ACK ){
                req_ack = true;
                continue;
            }
            else if(下行内容是PULL_RESP){
                解析JSON PAYLOAD,封装成LoRa格式;
                jit_result = enqueue();
            }
            send_tx_ack();
            // if(msg_len > 0) break;
        }
    }
}
```





### 3.3 使用TCP的网关-服务器下行协议

* 删除了网关主动发起通信的`PULL_REQ`，服务器也不需要再向网关发送`PULL_ACK`。
* 需要发送下行的情况，服务器直接发。
* 服务器需要处理`PULL_RESP`发送失败的情况。

```sequence
Note left of Server: Anytime for each packet to TX
Server->Gateway: PULL_RESP(token Z, JSON payload)
Gateway->Server:TX_ACK(token Z, JSON payload)
```



### 3.3 使用TCP的网关下行代码框架

```C
void thread_down(void){
	变量初始化
    设置socket
    while(1) {
        while (beacon_loop && (beacon_period != 0)) {
        	处理beacon内容;
        }
        msg_len = recv(buff_down) //从服务器端接受buff_down
        if(msg_len < 1)
            continue;
        else if(无法辨别的下行内容)
            continue;
        else if(下行内容是PULL_RESP){
            解析JSON PAYLOAD,封装成LoRa格式;
            jit_result = enqueue();
        }
        send_tx_ack();
    }
}
```



## 4. 特殊情况说明

### 4.1 服务器侧主动关闭

* 由于网关上行协议已经变成了**先收再发**，服务器断线之后，网关可以照常发出最后的`TX_ACK`，但会发送失败，然后网关被卡在收包阶段。如果此时服务器再开启，服务器端的下行线程在开启TCP连接时只会处于`listen`状态，等待网关作为客户端接入，但是网关此时被卡在收数据。解决办法目前是，在**在一段时间内收不到服务器的下行数据，直接关闭TCP连接，开始重连。**