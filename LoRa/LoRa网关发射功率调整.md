###LoRa网关发射功率调整

#### 葛鑫 2019-1-1



在网关-服务器下行通信协议中，有个字段叫`powe`，作用是

```
powe | number| TX output power in dBm (unsigned integer, dBm precision)
```

所以下行JSON down中应该对此参数进行设置来满足发射功率要求。



网关的发射功率配置具体原理

* 网关加载配置文件`gloabal_conf.json`
* 其中一系列字段叫做`tx_lut_0` 到`tx_lut_15`，例如

```
"tx_lut_0": {
            /* TX gain table, index 0 */
            "pa_gain": 0,
            "mix_gain": 8,
            "rf_power": -6,
            "dig_gain": 0,
            "dac_gain": 3
        },
"tx_lut_15": {
            /* TX gain table, index 15 */
            "pa_gain": 3,
            "mix_gain": 14,
            "rf_power": 27,
            "dig_gain": 0,
            "dac_gain": 3
        }
```

在semtech官网论文找到了对它的描述

地址：https://www.thethingsnetwork.org/forum/t/tx-power-lut-in-gateway-configuration/6343



在网关开启前，会对射频部分进行校准。但服务器要求网关发送14dBm增益的下行时，网关会去查LUT，对应于`rf_power: 14` ，然后网关就知道了如何去调整放大器增益， mixer增益， dig增益去获得+14dBm的输出。



* 如果在配置文件中设置了网关天线增益`antenna_gain` 为`x`， 那么网关会去查找LUT中`rf_power = powe - x`的对应项。



但是LUT只有16项， rf_power的范围最小是-6，最大是27。所以当服务器指定的`powe`减去`antenna_gain`在LUT找不到对应的`rf_power`的时候，会得到一个发送错误，网关不会发送下行。合法的`rf_power`从小到大是

 `rf_power = [-6, -3, 0, 3, 6, 10, 11, 12, 13, 14, 16, 20, 23, 25, 26, 27]`

因为目前网关配置文件设置的天线增益为0，所以服务器下行JSON down中直接设置`powe`字段为上述集合中的一个就可以。







