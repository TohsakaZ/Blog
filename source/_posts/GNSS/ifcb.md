---
title: IFCB相关原理整理
p: GNSS/ifcb
date: 2021-03-02 15:51:12
tags:
---

#### 卫星钟差
目前IGS提供的卫星钟差主要是有12频估计得到的
载波相位观测值中存在一个和时间、信号（频率）、卫星相关的误差（称为时变载波相位误差）
目前主流机构的PCE估计的卫星钟差（无电离层组合)吸收了一些误差，主要包含：
* 卫星端的伪距偏差(即卫星端伪距信号的硬件延迟导致的偏差,随时间变化稳定,频率为12IF频段上的)
* 卫星端的来自载波相位的相关偏差（主要是随时间变化的部分，频率同样为12IF上的）

<!--more-->

#### 多频PPP
上述的12频无电离层组合钟差无法应用于多频PPP中
当用不同组合的频率估计出来的卫星钟差值不同，因此将差值定义为 ***IFCB（Inter-frequency Clock Bias)*** 频间钟偏差
显然，根据上述分析的卫星钟偏差的组成成分，可将IFCB分成 ***CIFCB(与伪距相关的部分稳定）和CIFCB(与载波相位相关的部分 时变)***

CIFCB显然是伪距偏差的一个线性组合
PIFCB是时变相位偏差的一个线性组合

PIFCB通过GFIF载波相位的时间序列进行估计
CIFCB通过组合DCB进行估计(这里 DCB本质上就是各卫星不同频率的伪距端偏差的相对差值)

##### 多频多系统PPP中涉及的主要偏差

* DCB （差分码偏差)
    本质就是伪距信号硬件延迟导致的，主要与信号通道相关（包括频率、伪距码)，卫星端，接收机端都存在
    * 卫星端的会吸收到卫星钟差，因此在使用非12频的观测值定位时需要改正使用的卫星钟差
    * 接收端的同样会吸收到接收机钟差，在多频定位时因此需要多估计相应的参数(如13频接收机钟差)

* DPB (差分相位偏差)
    本质是相位信号硬件延迟导致的，与DCB类似
    不过在处理过程中，卫星段与接收机端的都难以和模糊度参数分离，因此被整合进浮点模糊度进行估计了
    其值估计与UPD相关（具体关系？？）

* 信号群延迟（TGD/BGD)
    本质就是***卫星端***的伪距偏差的线性组合，可与DCB之间进行相互转换，播发在广播星历中

* IFB（频间偏差）
    本质上就是伪距信号延迟导致的，卫星端可以通过DCB改正处理，而估计IFB误差参数本质在于解决接收机端的偏差
    * 估计方式 
    一个基准频（如L1/L2频）接收机钟差+IFB参数 （等价于） 多个接收机钟差参数
    * 这里频间偏差是特别指估计系统内的

* ISB (系统间偏差)
    本质上也是由伪距信号延迟导致的，不过是专门针对系统间的（因为不同系统所采用的信号频率差异较大）

* UPD(相位小说偏差)
    为了模糊度固定而估计的参数，本质上包含了伪距(因为卫星钟差而导致引入伪距硬件延迟)和相位的硬件延迟

* SICB(卫星引起的伪距偏差)
    BDS伪距观测值中存在,根源是飞行器内部的多路径误差

* IFCB(频率间卫星钟偏差)
    这里是特别考虑了卫星端相位硬件延迟钟稳定和时变的部分，其中稳定部分如之前所述与模糊参数一起估计，而时变部分则是会被吸收到卫星钟差钟去
    因此卫星钟差吸收了卫星端的伪距硬件延迟部分和相位端的时变部分


#### IFCB的估计方式

* 网解的GFIF组合的方式
  本质上就是将15频IF组合的相位观测值和12频IF组合的相位观测值进行相减，从而得到包含PIFCB值的观测值
  即GFIF=PFIFCB + N+br+bc
  其中N为GFIF组合的模糊度，br，bc为GFIF组合的接收机端和卫星端的***稳定的***的硬件延迟偏差,实际估计中会被吸收到模糊度中
  因此利用模糊度即稳定偏差随时间内不变的特性，利用历元间差分进行了消除
* 改进策略：
  * 潘博士论文中估计方式是通过估计历元间差分的IFCB值，相当于假定第一历元的PIFCB值为0
  * 李盼论文中 使用最小二乘的方式估计绝对值，最后通过加平均值为0的方式进行估计
  * 使用迭代最小二乘去除粗差（great代码中没使用？）
  * 矩阵库使用和分段线性模型的使用

#### Great中的估计方式

* 输入是 o文件 和  log12、log13文件
* 输出是ifcb结果文件
---
* 入口在t_gupdlsq的processbatch
* 计算各历元各卫星的GFIF组合，频率通过设置文件给定
  * 将各测站各卫星的历元间差分的值存储在all_site_GFIF
* 历元循环
  * 测站卫星循环中初始化了_cal_upd值
  * 求各测站各卫星弧段的GFIF历元间差值均值？
  * 修复不同弧段之间的历元间差值（重点是把该历元的GFIF历元间差值存储到了 _site_sat_amb)结构上
  ---
  * 然后计算各卫星的所有测站的IFCB值的加权平均值，结果存储到了_cal_upd中
* 添加零和约束
* 写到文件中

总结：Great中使用的方式就是求各历元间差分的平均和，同时固定第一个历元为0，同时加了相应的约束
* 零和约束的加法：给定初始时刻一个非0的初始值,该值会使得当天所有历元的IFCB（绝对值）之和为0,不过有个nepo没看懂。。

BUG:
计算GFIF时的历元数与历元循环求IFCB加权和的历元数不统一，差1
然受计算当天IFCB绝对值之和 均值时，所除的数貌似也不对 差1