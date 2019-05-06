# Fick's Law 菲克定律研究

葛鑫， 2019-4-3

[TOC]

## 介绍

菲克（扩散）定律，在不依靠宏观的混合作用发生的传质现象时，描述分子扩散过程中传质通量和浓度梯度之间关系的定律，可以被用来求解扩散系数$D$, 建立扩散方程。

主要包括两个内容，菲克第一定律和菲克第二定律，第二定律由第一定律推导而来。



## 菲克第一定律

菲克第一定律建立了**稳态环境下**，扩散通量(diffusive flux)与浓度的关系。稳态环境定义如下：

>In systems theory, a system or a process is in a **steady state** if the variables which define the behavior of the system or the process are unchanging in time. In continuous time, this means that for those properties *p* of the system, the partial derivative with respect to time is zero and remains so:

第一定律假设了物质通量(flux)从浓度高的地方运动到浓度低的地方，并且通量与浓度梯度大小成正比。在一维环境下，菲克第一定律通常被写成以下的摩尔形式。

$$J = -D\frac{d\varphi}{dx}$$

物理含义：扩散通量 = 物质的浓度**随距离的变化率***扩散系数

* $J$是扩散通量，单位时间单位面积通过物质的物质的量。单位是$m^{-2}s^{-1}$
* $D$是扩散系数，单位$m^2/s$
* $\varphi$，对于理想混合物，表示浓度，单位体积里的物质的量，单位$mol/m^3$
* $x$，表示位置，单位是$m$

$D$正比于扩散粒子速度的平方，并且也取决于温度。室温下稀释液体中离子的扩散系数通常是$(0.6-2)*10^{-9}m^2/s$，生物分子的扩散系数通常是$10^{-11}$到$10^{-10}m^2/s$之间。

在二维或以上维度的情况下，需要使用$\bigtriangledown$，那么第一定律写成

$J=-D\nabla \varphi$

* 注意，第一定律只适用于稳态扩散的场合，即浓度和扩散通量不随时间变化。大多数扩散过程都是在非稳态条件下进行的。
* 非稳态扩散的特点是：扩散过程中，扩散通量随时间和距离变化。此时要应用菲克第二定律。



## 菲克第二定律

菲克第二定律预测了**扩散浓度如何随时间变化**，是一个偏微分方程，在一维下有如下形式

$$\frac{\partial \varphi}{\partial t} = D\frac{\partial^2\varphi}{\partial x^2}$$

物理含义：浓度随时间的变化率 = 浓度随距离的二阶梯度*扩散系数

* $\varphi$是浓度，单位是$mol/m^3$，$\varphi=\varphi(x,t)$，$x$是位置，$t$是时间，其他符号含义与上述相同。

在二维或更高维上使用拉普拉斯算子$\Delta = \nabla^2$，得到下列方程

$\frac{\partial \varphi}{\partial t}= D\Delta \varphi $



## 菲克定律的推导

菲克第二定律由菲克第一定律和质量守恒定律得到

* $$\frac{\partial \varphi}{\partial t} + \frac{\partial}{\partial t}J = 0 $$

上述物理含义：浓度随时间变化的速率和通量随时间变化的速率，大小相等方向相反。浓度的变化，衡量的是在单位空间内，经单位时间，粒子的流出/流入情况。而这本来就是扩散通量的定义：单位时间单位面积通过的物质的量。

然后将$J$由菲克第一定律带入得到

* $$\frac{\partial \varphi}{\partial t} - \frac{\partial}{\partial x}(D\frac{\partial}{\partial x} \varphi)=0 $$

假设扩散系数是一个常数，改变微分顺序可以得到

* $$\frac{\partial}{\partial x}(D\frac{\partial}{\partial x} \varphi) = D\frac{\partial^2\varphi}{\partial x^2}$$

这样就得到了菲克第二定律，$$\frac{\partial \varphi}{\partial t} = D\frac{\partial^2\varphi}{\partial x^2}$$，或 $\frac{\partial \varphi}{\partial t}= D\Delta\varphi $

如果出现了$\varphi$处于稳定状态，也就是浓度不随时间改变，方程左边为0。在一维情况下，若扩散系数$D$为常数，那么浓度随着$x$轴是线性变化的，在二维或高维情况下，可以得到$\bigtriangledown ^2 \varphi =0$ 。（一阶导数为常数，二阶导数为0）

如果扩散系数$D$不是常数，而取决于物理位置和浓度，那么菲克第二定律写成

* $\frac{\partial \varphi}{\partial t} = \nabla \cdot (D\nabla \varphi) $





菲克第二定律是**对流扩散方程( Convection-diffusion Equation)**在没有**对流(advective flux)**和没有**净体积源(net volumetric source)**的特殊情况。可以从对流扩散方程得到

![{\displaystyle {\frac {\partial \varphi }{\partial t}}+\nabla \cdot \mathbf {j} =R,}](https://wikimedia.org/api/rest_v1/media/math/render/svg/f5d5772f9dd5f0f74e7dbac7752ce8bf404af3c7)

$j$是总流量， $R$是$\varphi$的净体积源，在这样的情况下，$j$的唯一来源是**扩散流量(flux flux)**

![{\displaystyle \mathbf {j} _{\text{diffusion}}=-D\nabla \varphi }](https://wikimedia.org/api/rest_v1/media/math/render/svg/133001e30858f5a14db613818fa9898045e1fdd6)

把上式带入到上上式，并假设没有源($R=0$)，得到菲克第二定律

![{\displaystyle {\frac {\partial \varphi }{\partial t}}=D{\frac {\partial ^{2}\varphi }{\partial x^{2}}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/be2b6de15ad902fab517740ab7d484e984a1cdf4)



## 菲克定律的解

