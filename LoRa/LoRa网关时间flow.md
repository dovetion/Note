### LoRa网关时间flow

#### 葛鑫 2019-1-2

---

![image-20190102202436504](../Library/Application Support/typora-user-images/image-20190102202436504.png)



网关的所有时间都来自GPS，但是具体到不同使用场景的时，可能会使用内部计数器，可能使用到UTC时间

因此也具有如下关系

![image-20190102203232719](../Library/Application Support/typora-user-images/image-20190102203232719.png)

它们之间的转换关系通过一个时间同步线程来进行同步，也就是`thrid_gps`和`thrid_valid`

`thread_gps`不断从串口读取接收到的gps报文，用parser解析，更新底层全局gps时间变量

`thread_valid`不断判断当前time_reference是否有效，修正晶振偏差（calculate XTAL  correction）



---

网关的时间流很复杂，并且众多函数的算法一一都看也很难定位错误，且一些错误校准算法大致一看也看不懂。所以我觉得直接去检验这三个时间，查看它们的变化规律是否线性就可以判断网关的时间转换算法到底是不是有错。



每收到一个包，网关会打上时间戳

```
JSON up: {"rxpk":[{"tmst":322954380,"time":"2019-01-02T19:53:24.747868000Z","tmms":1230465223748,"chan":3,"rfch":0,"freq":433.175000,"stat":1,"modu":"LORA","datr":"SF12BW125","codr":"4/5","lsnr":6.2,"rssi":-51,"size":15,"data":"QKRQegGAHwADfRk7nj0T"}]}
```

`tmst`是内部计数器， `time`是上行收到的UTC时间，`tmms`是gps时间（从1980年1月6日到现在的毫秒数）

其中代码里`time`字段的变量是这样得到的

```C
j = lgw_cnt2utc(local_ref, p->count_us, &pkt_utc_time);
                if (j == LGW_GPS_SUCCESS) {
                    /* split the UNIX timestamp to its calendar components */
                    x = gmtime(&(pkt_utc_time.tv_sec));
                    ...
                }
```

网关的时间全部来自GPS，在某个阶段一定做了GPS与CNT的同步，此时又做了cnt2utc的同步，最终得到了`time`



* 测试方法，终端定时发包，网关记录打印信息，拿到每一个包的三个字段`tmst`，`time`和`tmms`，两两比较，查看线性程度，如果转换算法有错，一定会有点偏离直线，或所有点的趋势不在直线上。

* 测试结果





* `tmst`与`time`对比图

  ![image-20190103140446726](../Library/Application Support/typora-user-images/image-20190103140446726.png)

  ![image-20190103140724925](../Library/Application Support/typora-user-images/image-20190103140724925.png)


* `time`与`tmms` 对比图

   ![image-20190103140648261](../Library/Application Support/typora-user-images/image-20190103140648261.png)

  ![image-20190103140754887](../Library/Application Support/typora-user-images/image-20190103140754887.png)

*  `tmst`与`tmms` 对比图

  ![image-20190103140630381](../Library/Application Support/typora-user-images/image-20190103140630381.png)

  ![image-20190103140818119](../Library/Application Support/typora-user-images/image-20190103140818119.png)

但是由于时间精度需要很高，需要计算能产生的最大误差才能有说服力，因此使用下面的方法去检查最大的误差

![image-20190104164715415](../Library/Application Support/typora-user-images/image-20190104164715415.png)

经过确认，时间同步线程的更新间隔的确是1000ms

结果如下

  ```
tmst为X time为Y
avr slop = 1.0001139029094916
max slop = 1.0402998057566095
min slop = 0.914324528573485
最坏情况下，每秒同步一次造成的误差如下
对 time 的影响为40.185902847117866 ms/ -85.78937433600665 ms

tmst为X tmms为Y
avr slop = 0.0010000128310457149
max slop = 0.0010001063687701292
min slop = 0.0009998831785983193
最坏情况下，每秒同步一次造成的误差如下
对 tmms 的影响为0.0935377244143127 ms/ -0.12965244739556075 ms

time为X tmms为Y
avr slop = 0.0009999397804777307
max slop = 0.0010936292243793055
min slop = 0.0009613628233261116
最坏情况下，每秒同步一次造成的误差如下
对 tmms 的影响为93.68944390157489 ms/ -38.57695715161905 ms
  ```



结论：

似乎有`time`字段的地方的确有精度问题，下一步需要查看下行发送要使用到哪些时间转换函数，以及同步线程的同步周期大概是是多少。如果非常快的话那么基本是不会有问题。如果不是这样，那么有`utc`存在的时间转换函数的算法的确有问题，需要查找。

![image-20190114152521362](../Library/Application Support/typora-user-images/image-20190114152521362.png)