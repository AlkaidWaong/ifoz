---
layout: post
title: "OMS:结合生活服务行业谈订单状态、订单合并和订单拦截"
subtitle: ''
date:       2022-12-21 23:54:00
author: "Alkaid"
header-img: "/img/post-bg-unix-linux.jpg"
tags:
  - OMS订单系统
---

OMS是业务中台中很重要的部分，承接了来自上游系统的数据，进行处理并向下游的系统传递（如WMS）。中间还要与财务系统这样的相关业务系统进行数据传递。因此，订单是承载了大量的数据且需要有极快的处理效率。

![oms_001](https://p.ipic.vip/3nuoha.png)

上面是我对OMS系统的相关服务和功能做了一些简化的理解。关于订单拆分服务前面已经写了一篇文章[《OMS订单系统中的拆单过程》](https://www.ifoz.net/2022/12/18/How-to-split-orders-in-OMS/)直接点链接即可。



### 订单状态的流转

我之前是做教育相关的产品设计的，在进到生活服务相关的领域的时候发现订单的状态比课程类的要复杂不少。原因在于两点：一个是业务流转链路更长，一个是业务场景复杂对订单状态的变更较多。

我自己在设计时的感受是前端展示只展示客户有感知的状态不宜过细，在我们的业务中师傅在上门前一天都是会和客户约定具体的时间，待预约其实就是告知客户师傅会提前安排联系。虽然名字叫待预约，界面设计上会有额外的一些说明和对于时效的预估情况。这样的做法能缓解客户对时间安排的焦虑。



下面的图是我在设计订单状态时做的一张表，可以看看和你们的订单状态之间有多少差别。

![](https://p.ipic.vip/lzhgi8.png)



另外今天想结合一些我在生活服务家政服务行业中的经验来说一下：

1. 订单合并
2. 订单拦截



### 订单合并
订单合并是将同一个客户的多张订单合并在一起打包配送或进行上门服务，好处在于节省客户需要多次在家等候的时间，降低师傅多次上门的路程成本，提升了单次服务的客单价。所以合并订单在业务规模大后是非常有必要的精细化运营手段。

**订单合并的条件**在业务上会存在一些约束条件，不同业务有差异，设计时需要充分考虑：

 	1. 相同的下单账号*
 	2. 相同的服务地址/收货地址*
 	3. 相同的开票抬头
 	4. 相同的订单类型
 	5. 相同的订单来源（不同平台下单的不合并）
 	6. 相同的配送方式（实物商品中存在）



**订单合并的时点**

不同业务订单合并的时点有很多的差异，系统设计上最好能做配置来确保对不同业务做精细化的处理。合并时点的设计要遵循尽快送达原则，即不因合并订单而对订单时效有太大影响。比如在家政服务中，一般都是要提前一晚为师傅排好次日的订单。从客户下单到预约上门时间的间隔可能是2天左右的时间，时点设计就相对宽松，可以是8小时，12小时甚至更久。外卖配送服务就不能是这么久的间隔，可能就几分钟的时间。

如果在系统完成合并订单后，还存在情况要合并订单的话，需要为客服提供手动合并的功能来完成。



订单合并以后需要将原先的订单作废，生成一张新的订单来完成后续的履约流程。这里要注意的是合并后的新订单是用来走履约流程的，即订单下发到商家和师傅是合并的订单（实物中仓库拣货到配送是按照合并订单来的），客户端看到的订单是不合并的，仍是多张订单。

合并订单是需要做的一些处理：

	订单商品信息：对需要合并的子订单的SKU、数量、商品金额、优惠金额、实际售价等信息
	订单收货人信息：从任意的订单中获取复制到新订单即可
	下单人信息：从任意的订单中获取复制到新订单即可
	发票信息：发票信息的合并主要是将开票金额、开票内容、抬头、税号等信息合并
	运费信息：在O20类虚拟服务中运费为上门费，上门费则一般合并后只收取一次，在实物配送中可以直接累加，也可以按特殊规则计算。
	促销优惠信息：促销信息明细集中到合并的父订单



### 订单拦截

![订单整理学习-第 3 页.drawio](https://p.ipic.vip/42tmas.png)



订单拦截是在拉单服务和拆单服务之间的一个服务，目的是拦截恶意订单。订单被设定的拦截规则后，可能被强制取消订单或进行人工的核验。这么做的目的是避免恶意刷单和释放锁定的库存。

常出现的拦截：

1. 单个账号下单数量异常：特别是在预订金模式下，锁库存严重。例如空调清洗服务柜机和挂机的数量不太可能大于6；
2. 单个账号下单过于频繁且地址分散：可能薅羊毛二次转卖了订单，和活动之间密切。或者出现了服务定价异常的低于市场价

我的经验来看，对于订单的拦截需要对基础的灰产做法有一定了解，还要结合对营销活动、商品特点来进行规则的设计。开始之前多和在这个市场上做经营的人、运营同事聊一聊，会有不少超出认知的收获。



另外，虚拟物品服务中订单取消也有一些特殊的地方，以后有时间再分享。
