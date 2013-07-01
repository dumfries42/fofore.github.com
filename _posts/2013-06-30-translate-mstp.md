---
layout: post
title: 理解MSTP
category: story 
---

#1.动机
首先,这是一篇"奇葩"的翻译文.因为这篇文章采用的方法为"直译法",而且是非常直译,刻意而为之的直译.为何进行这样子的直译.其实本来是准备规规矩矩的将这篇文档翻译出来,因为网络上关于MSTP介绍的文章乏善可陈.所以使用google英文搜索到这篇比较有诚意的文章:[Understanding MSTP](http://blog.ine.com/2010/02/22/understanding-mstp/).但是阅读过程遇到不少问题,关键还是英文思维的问题.因为这些句子都是比较长,如果整句读下来,再去用中文思维理解,非常之困难和费时间.而且如果遇到一些比较难懂的英文单词,整句话,就没有办法理解.于是我在想,是不是这样阅读英文文章的方法都是错误的.

最终,我发现了一个有趣的现象,当我改变了翻译方法,不再一整句一整句的想用中文表述出来,而是就跟着文章的英文语序,简单的进行转化到中文的变化,不但能够很快的理解文章,而且似乎还有了一点"英文思维"的感觉在里面,不知道这样的感觉是不是错觉呢?(大把流汗!)

不管是否是错觉,我举得还是要执行下去,最后看看效果,必须声明的是:这篇译文的结果看起来绝对是一窍不通的.但是为了督促自己工作,我有必要将它完成.所谓:善始者盖繁,克终者盖寡.(古文功底显露啦!)

另外,这个方法有个非常好的好处:它能够让你很快进入一个flow状态.学习的最佳状态,精神力高度集中.想想现在人一天能有几分钟是在这个状态的,进入这个状态还不怕被打断.如果你再加上一个比较好的翻译,发布,阅读环境VIM+github.那么你就可以随时随地的进入免打扰的flow模式啦.这个flow模式是我在这本[人件集](http://www.amazon.cn/%E4%BA%BA%E4%BB%B6%E9%9B%86-%E4%BA%BA%E6%80%A7%E5%8C%96%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91-%E5%BA%B7%E6%96%AF%E5%9D%A6%E4%B8%81/dp/B005YWYHLW/ref=sr_1_1?ie=UTF8&qid=1372575585&sr=8-1&keywords=%E4%BA%BA%E4%BB%B6)里面看到的,这是本老书了,不过浏览浏览还是值得的.

那么,关于"英文思维"这个词呢,是我在用一整天看完了[外语学习的真实方法及误区](http://www.douban.com/group/topic/6767868),然后边看边总结,发掘出来,这是我当前来说,十万火急,刻不容缓的事情.

放置一个TODO在这边,如果将中英文对比着放置在这里,是不是会有更好的效果呢?(不如抽空就试试.)

对于不确定,还有我不熟悉的单词,我就用括号将它们暴露出来啦.(不要害羞.)这些内容对我来说是可理解+1的内容,都是重要的客户,接下来是要特别针对的,请颤抖.



--------------- **我是用来分割闲话和正文的分割线** -------------------

#理解MSTP


##介绍

随着时间的推移(over time)我在思考-放一起-把两个博客文章-之前做的-关于MSTP和增加更多的澄清(clarification)-为MSTP多域那一段.这个新的博客文章概括了(recaps)新的信息-被发表过-在之前的,并且提供了更多细节-这次.额外的,则例讨论了一些MSTP设计相关的问题.单域和多域MSTP配置都被提到了在这个文章中.读者被假设有个很好的理解-对传统的STP和RSTP协议,还有Cisco的PVST/PVST+实现.

##内容表格

由于大体积-这个文档,一个表格-内容的被提供了-为了方便导航(不过我还没有搞定如何加上目录跳转,先暂时就是个罗列吧)

* 历史回顾
* 逻辑和物理拓扑
* 实现MSTP
* 注意事项在MSTP设计
* MSTP单域配置例子
* 公共和内部生成树(CIST)
* 公共生成树(CST)
* 映射MSTI到CIST
* MSTP多域设计注意事项
* 互操作和PVST+
* 脚本1: CIST总根和CIST域根
* 脚本2: MSTI和主口
* 脚本3: PVST+和MSTP互操作
* 结论
* 进一步阅读


##历史回顾

在最开始,有IEEE STP协议,它是优化于DEC和IBM的生成树变种.他们都使用一样的逻辑-初始的由Radia Perlman提出的(proposed)在80年代,当时她工作在DEC.这个IEEE版本是适配于使用多VLAN-使用802.1q框架标签的VLAN.一个共享生成树,有些时候被叫做Mono生成树MST-被思科这么叫,或者更多时候-公共生成树(CST)被用来创建一个单个的无环的拓扑.这个(途径)approach的缺点是不能实现VLAN流量工程-在冗余链路上的:如果一个链路被阻塞了,在它上面就阻塞了所有的VLAN.另外一个问题与STP建造有关-更多的流量被转发在链路-更靠近根桥的,这样提出很高的需求在根桥的资源-对CPU和链路capacity(容量)使用.

为了克服这些限制,思科引入了私有的(propietary)Per-VLAN生成树(PVST),使用独立的STP实例-每个VLAN.开始的时候,PVST被创造于使用-与思科的私有ISL封装-只与,但是,随后PVST+版本允许隧道PVST BPDU基于802.1q trunk以及IEEE STP域.PVST允许使用不同的逻辑拓扑-对于每个VLAN,增强基础的二层流量工程.每个VLAN使用它自己的根桥和转发拓扑-允许更多公平的资源的使用.这个方法有一些限制-因为它没有处理实际的网络链路容量和采用,只是(but rather)统计学地复用VLAN到不同的拓扑.然而,这里有个限制(inherent to)固有的-任何负载平衡方法-基于STP.主要的问题-PVST的-是VLAN增长的数量,PVST成为一个浪费-对交换资源和管理(burden).这是因为这个数目-不同的逻辑拓扑-是不同寻常的小-比活动的VLAN的数目.

随着时间的推移(with time), PVST接受了快速收敛的特性-介绍自IEEE RSTP协议,但是核心的特性-保持单独的STP副本-每个VLAN-没有改变.看到这个问题结合在PVST途径(approach)上.思科想出主意-分裂概念-STP实例和VLAN.初始的实现叫MISTP(多实例生成树)并且随后提交到(evolved)IEEE 802.1s标准叫做MSTP(多生成树协议).



##逻辑和物理拓扑(们)

MSTP的核心思想是使用这个事实-一个冗余的物理拓扑只有很少数量的不同的生成树(逻辑拓扑).这个图-在下面-显示了一个环拓扑-包括3个交换机和3个不同的生成树-那些可能来源于不同的根桥位置的.

![mstp-3-physical-logical-topos.png](/images/mstp-3-physical-logical-topos.png)

替换掉跑一个STP实例为每个VLAN,MSTP跑有限个VLAN独立的(VLAN-independent)STP实例(代表representing逻辑拓扑),然后管理员映射对应的VLAN到最接近的逻辑拓扑(STP实例).STP实例的数量保持最小(保存资源),但是网络容量被使用域一个最优的(optimal)趋势,通过使用所有可能的路径-为VLAN.

这个交换机逻辑-VLAN流量转发的-变化了一点点.为了每个帧被转发出一个端口,两个条件必须符合:1,VLAN必须是使能在这个端口的(例如:没有被过滤),2:STP实例-VLAN映射的那个-必须在非丢弃状态-对这个端口.第二个条件(property)通常是强制的(enforced)自动地,因为MAC地址不会学习在丢弃端口上.这是值得提醒的-对于多个逻辑拓扑-活动在一个端口,这个端口可能被阻塞-为了某个实例,并且又是转发的为了另外一个实例.(注意到在RPVST+,一个端口对于某个VLAN来说不是转发就是丢弃).这个图片-下面的-显示了6个VLAN使用两个实例,这样减少了STP树的数量-在PVST中需要6个的,这里减少到了2个.

![mstp-3-physical-logical-topos-2.png](/images/mstp-3-physical-logical-topos-2.png)



##实现(Implementing)MSTP

以下是一系列实施相关的问题-理论实现需要解决的:

* **逻辑拓扑计算**. 如何建立多个STP实例(逻辑实例)在一个物理拓扑中?我们应该跑多个STP发送他们的BPDU-单独地吗?如果是的,我们如何区分(distinguish)每个实例的BPDU?VLAN标签不能被使用在这个目的下,因为STP不再绑定到VLAN-任何的.

* **信息分发**. 什么协议应该被用来分发VLAN到实例的映射表格-在交换机间.VLAN ID应该被放在BPDU中与相应的实例号一起吗?

* **一致性(consistency)检查**. 如何保证VLAN到实例映射是一致的-在所有交换机之间?是说:一个交换机如何得知其他的交换机映射VLAN到了同样的实例.


##MISTP vs MSTP

原始的思科MSTP预标准实现是发送独立的BPDU-为单个实例.每个BPDU包括实例号和VLAN列表,被映射在发送交换机的这个特殊的实例-这允许一致性检查-在单个的交换机上独立的.没有自动化的机制来分发VLAN到实例-映射在交换机间的.

最终的实现-被IEEE 802.1s标准接受的-使得这个机制更优雅(elegant)和简单.在我们执行讨论IEEE实现之前.让我们定义**MSTP域**为一个交换机集合,共享同样的物理拓扑视图-(partition)划分为一系列逻辑拓扑.为了使两个交换机成为成员-同一个域的,下面的条件必须满足:

* 配置名字
* 配置revision号(16比特的值)
* 表格(有4096元素的)映射对应的VLAN到STP实例号.

IEEE 802.1s实现没有发送BPDU-为每个活动的STP实例-独立地,也没有封装VLAN号列表配置消息.取而代之的是,一个特殊的STP实例-号码为0的-叫做内部生成树IST(IST(aka)又名MSTI0,多生成树实例0)被指定来承载所有的STP相关的信息.这些BPDU-IST的-包括了所有的标准RSTP格式的信息-为IST本身,也携带了额外的信息区域.在这些区域里是配置名字,revision号和一个哈希值-计算自VLAN到MSTP映射表内容.使用就这个(压缩的)condensed信息,交换机可以检测到错误配置-VLAN映射的-通过比较哈希值-把从邻居接受的和自己本地的.

##M-Records (M-记录?)

缺省的,所有的VLAN是映射到IST的.这就代表了经典IEEE RSTP的案例-所有VLAN公用一个同样的生成树.其他MSTP实例可以被使能,并且他么被参考为(be refer to)多生成树实例(MSTI).么个MSTI指定他么的自己优先级-对交换机-并且使用自己的链路开销来产生一个私有的逻辑拓扑,独立于IST.既然MST不发送MSTI的消息-在独立的BPDU,这个信息是被piggybacked(搭载)到IST的BPDU使用特殊的M-记录区域(一个区域为所有的活动的MSTI).使用TLV(类型-长度-值)-这些区域携带根优先级-指定桥优先级-端口优先级和根路径开销-在其他的之间.

##MSTI树建造(建设)(construction)

类似于RSTP,每个交换机发出(emit)他自己的配置BPDU,一个每hello秒数.这些BPDU有全部的信息-关于IST和携带的M-记录-每个活动的MSTI的.使用RSTP的收敛机制(提议和同意比特),独立的STP实例被建造-为IST和每个MSTI.重要的去注意到-基本的(fundamental)STP时间如Hello,转发时间,Maxage**只能**被调整(be tuned)-由IST-这是必然的(natural)结果-所有的MSTI信息被搭载(piggyback)到IST BPDU.

MSTP有特殊的机制来老化(age out)旧的信息-区域(domain)外的.IST BPDU有特殊的一块-叫做**Remaining Hops**.IST根发送BPDU带有跳数技术等于**MaxHops**(配置值)以及每个下游交换机减少这个跳数计数块-当收到IST BPDU时.当跳数计数变成0时候,这个信息在BPDU中被忽略了,同时这个交换机将开始声明它自己为新的IST根.传统的MaxAge和ForwardDelay时间任然被在使用-当MSTP协作于RSTP,STP或PVST+桥时.

##STP争议(dispute)

思科交换机用了很长时间实现无环的特性-允许阻塞非指定端口-当它丢失了STP BPDU流.这是有帮助的-对于检测单向(unidirectionla)链路(通常在光纤链路)-并且阻止了2层环-有可能被STP检测不到的.思科的MSTP实现允许检测单向链路情况,通过比较下游端口状态-在BPDU中报告的.如果上游交换机发送更优根桥信息到下行桥-但是接收的BPDU设置了指定比特,上行的交换机断定-下行的交换机没有收到它的BPDU.上行的交换机将阻塞这个下行口,并且把它标记为STP dispute链路.

![mstp3-stp-dispute.png](/images/mstp3-stp-dispute.png)

##注意事项(caveats)-关于MSTP域的

这里会有一些问题,由于这个事实-生成树实例没有被映射(一个一个的)到VLAN.使用PVST,剪裁一个VLAN在一个链路上,会同样禁止相应的STP在同样的链路上.既然MSTI脱离(decouple)VLAN,每个MSTI都跑在每个link上(域中的).这个在MSTI实例之间,差异在-他们做决定让哪条链路转发或者阻塞.通过剪裁(prune)VLAN,你可以终止于一个情形-VLAN没有使能在链路-(这些链路:相应的MSTI是转发或者使能,相应的MSTI阻塞). 考虑下面的例子用于解释这个想法.

![mstp-3-vlan-pruning-caveats.png](/images/mstp-3-vlan-pruning-caveats.png)

在这个拓扑中,VLAN被手动的剪裁了-在trunk中.TODO由于这个过滤是不符合相应(respective)的MSTI阻塞决定,VLAN2的流量是被阻塞了,在SW1和SW2之间.为了避免这种情况,不要用静态的"VLAN"剪裁方法-分配VLAN跨越trunk,当你有MSTP使能.一种情况相同于这个(描述的)就是当端口连接交换机是**接入(access)端口**.MSTP跑在这些端口上,并且有逻辑拓扑-可能是阻塞或者转发-在端口上.取决于VLAN到MSTI的映射,一个给定的VLAN可能被阻塞在接入端口-由于MSTP决定,尽管这个接入VLAN是不同的,他们使用相同的STP.为了避免这个问题,不要跑MSTP在接入端口,而且用他们于链接"末端"设备(只这样用)-例如,主机以及叶子(leaf)交换机. (同一个实例中的vlan,不要使用access端口, 同一个实例中不要将VLAN允许到不同的躯干(trunk)中)

##MSTP单域配置实例

现在我们有了基础的认识-对于如何MSTP工作在一个域内.让我们创建一个样例配置. 考虑到如下的物理拓扑-这三个交换机的:

![3-switchs](/images/mstp-3-single-region-config-scenario.png)

这个拓扑有这几个VLAN:1,10,20,30,40,50,60. 我们的目标-为了这个脚本(scenario)是:

    让VLANs 10,20,30跟随链路-从SW3-Sw1
    让VLANs 40 50 60跟随链路-从SW3-SW2
    如果任何上面的链路失效了,受影响的VLAN应该调整(fall-back)到其他的链路.

为了做到(accomplish)这个,我们创建两个MSTI,编号1和2.SW1将是根-用于实例1,SW2将是根-用于实例2.至于IST(MSTI0),我们使SW3为根交换机-给IST(尽管这不是提倡的recommended-指定根角色给接入交换机).至于VLAN到MSTI的映射,VLAN1将仍然映射到IST.剩下的VLANs 10,20和30将映射到MSTI1,同事VLANs 40 50和60将映射到MSTI2. 下面是配置:

    SW1:
    spanning-tree mode mst
    !
    spanning-tree mst configuration
     name REGION1
     instance 1 vlan 10, 20, 30
     instance 2 vlan 40, 50, 60
    !
    ! Root for MSTI1
    !
    spanning-tree mst 1 priority 8192
    !
    interface FastEthernet0/13
     switchport trunk encapsulation dot1q
     switchport mode trunk
    !
    interface FastEthernet0/16
     switchport trunk encapsulation dot1q
     switchport mode trunk
    
    SW2:
    spanning-tree mode mst
    !
    spanning-tree mst configuration
     name REGION1
     instance 1 vlan 10, 20, 30
     instance 2 vlan 40, 50, 60
    !
    ! Root for MSTI 2
    !
    spanning-tree mst 2 priority 8192
    !
    interface FastEthernet0/13
     switchport trunk encapsulation dot1q
     switchport mode trunk
    !
    interface FastEthernet0/16
     switchport trunk encapsulation dot1q
     switchport mode trunk
    
    SW3:
    spanning-tree mode mst
    !
    spanning-tree mst configuration
     name REGION1
     instance 1 vlan 10, 20, 30
     instance 2 vlan 40, 50, 60
    !
    ! Root for the IST
    !
    spanning-tree mst 0 priority 8192
    !
    interface FastEthernet0/13
     switchport trunk encapsulation dot1q
     switchport mode trunk
    !
    interface FastEthernet0/16
     switchport trunk encapsulation dot1q
     switchport mode trunk

下面的显示命令将显示出效果-我们的配置对于流量转发:

    SW1#show spanning-tree mst configuration
    Name      [REGION1]
    Revision  0     Instances configured 3
    
    Instance  Vlans mapped
    --------  ---------------------------------------------------------------------
    0         1-9,11-19,21-29,31-39,41-49,51-59,61-4094
    1         10,20,30
    2         40,50,60
    -------------------------------------------------------------------------------
    
    SW1#show spanning-tree mst               
    
    ##### MST0    vlans mapped:   1-9,11-19,21-29,31-39,41-49,51-59,61-4094
    Bridge        address 0019.5684.3700  priority      32768 (32768 sysid 0)
    Root          address 0012.d939.3700  priority      8192  (8192 sysid 0)
                  port    Fa0/16          path cost     0
    Regional Root address 0012.d939.3700  priority      8192  (8192 sysid 0)
                                          internal cost 200000    rem hops 19
    Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6
    Configured    hello time 2 , forward delay 15, max age 20, max hops    20
    
    Interface        Role Sts Cost      Prio.Nbr Type
    ---------------- ---- --- --------- -------- --------------------------------
    Fa0/13 Desg FWD 200000    128.15   P2p
    Fa0/16 Root FWD 200000    128.18   P2p 
    
    ##### MST1 vlans mapped:   10,20,30
    Bridge        address 0019.5684.3700  priority      8193  (8192 sysid 1)
    Root          this switch for MST1
    
    Interface        Role Sts Cost      Prio.Nbr Type
    ---------------- ---- --- --------- -------- --------------------------------
    Fa0/13 Desg FWD 200000    128.15   P2p
    Fa0/16 Desg FWD 200000    128.18   P2p 
    
    ##### MST2 vlans mapped:   40,50,60
    Bridge        address 0019.5684.3700  priority      32770 (32768 sysid 2)
    Root          address 001e.bdaa.ba80  priority      8194  (8192 sysid 2)
                  port    Fa0/13          cost          200000    rem hops 19
    
    Interface        Role Sts Cost      Prio.Nbr Type
    ---------------- ---- --- --------- -------- --------------------------------
    Fa0/13 Root FWD 200000    128.15   P2p
    Fa0/16           Altn BLK 200000    128.18   P2p 
    
    SW1#show spanning-tree mst interface fastEthernet 0/13
    
    FastEthernet0/13 of MST0 is designated forwarding
    Edge port: no             (default)        port guard : none        (default)
    Link type: point-to-point (auto)           bpdu filter: disable     (default)
    Boundary : internal                        bpdu guard : disable     (default)
    Bpdus sent 561, received 544
    
    Instance Role Sts Cost      Prio.Nbr Vlans mapped
    -------- ---- --- --------- -------- -------------------------------
    0        Desg FWD 200000    128.15   1-9,11-19,21-29,31-39,41-49,51-59
                                         61-4094
    1        Desg FWD 200000    128.15   10,20,30
    2        Root FWD 200000    128.15   40,50,60
    
    SW1#show spanning-tree mst interface fastEthernet 0/16
    
    FastEthernet0/16 of MST0 is root forwarding
    Edge port: no             (default)        port guard : none        (default)
    Link type: point-to-point (auto)           bpdu filter: disable     (default)
    Boundary : internal                        bpdu guard : disable     (default)
    Bpdus sent 550, received 1099
    
    Instance Role Sts Cost      Prio.Nbr Vlans mapped
    -------- ---- --- --------- -------- -------------------------------
    0        Root FWD 200000    128.18   1-9,11-19,21-29,31-39,41-49,51-59
                                         61-4094
    1        Desg FWD 200000    128.18   10,20,30
    2        Altn BLK 200000    128.18   40,50,60

这个链路开销值是更高的-比默认的STP开销(IEEE标准值),并且MSTIx是叫做MSTx(例如IST是MST0).除此之外,注意到术语(term)"域根"-那将会被介绍-详细的-在下面.

##公共和内部生成树

正如在前面提到的,每个MSTP域跑特殊的实例-生成树的-被知道:IST或者是内部生成树(=MSTI0).这个实例主要的服务于这个目的:disseminating生成树信息-从MSTIs中.IST有一个根桥,被选举-基于最小的桥ID(桥优先级+MAC地址).这个情况跟着变化-当多个MSTP域在网络中.当一个交换机检测到BPDU信息:源于另外的域(或STP/PVST+BPDU),它标记这些对应的端口为MSTP边界.对于这个(for the)convenience,我们能称所有的其他口为"内部".一个交换机-有边界口,被认为是边界交换机.

在这个图片中-下面的-你能看到3个MSTP域-interconnected为"环"拓扑-用一对线-在每两个域.这个链路-连接这些域的-连接了边界端口.既然每个交换机有一个连接到其他域,所有的交换机都是边界.需要注意简化的表达(简化表达法notation)-用于(for)链路开销和桥优先级.我讲使用这些来演示:如何-这些CIST被组成的.为了简化,假设所有的链路开销-在域内的-是同样的值:1.

![mstp-3-multi-region-physical-topology.png](/images/mstp-3-multi-region-physical-topology.png)

当多个域链接到一起,每个域需要建立它自己的IST,并且所有域应该建议一个公共的CIST生成树贯穿所有的域.为了看到如何这个完成的,第一,看一下这个结构:MSTP BPDU的.在个图-下面的,注意到MSTP用协议版本3作为opposed(相对)RSTP版本2.版本4是保留个SPT的-最短路径树-新的环路抑制以及报文桥接标准-定义在emergingIEEE 802.1aq文档.o

![mstp-3-multi-region-cst-mstp-packet-format.png](/images/mstp-3-multi-region-cst-mstp-packet-format.png)

这个MSTP BPDU包含了两个重要的块-块中是信息.第一个,高亮在红色,是相关到CIST根和CIST域根选举.就你将看到的-不久以后,CIST总根是被选举-在所有域里,CIST的域根是选举的-在每个域里.这个绿色的块-高亮出了信息-关于CIST域根(谁-成为IST根in presence of(当存在)多个域).CIST的内部根路径开销是区域内开销-要达到CIST域根的.这是重要的-记在脑子里-**IST根==CIST域根**在这种情况下:当多个域协同工作.这个转换(transformation)是会被解释的-将来-在文章里.先要,要去定义CIST总根和CIST域根的角色了.

* CSIT总根是这样的桥-它有最低的桥ID-在所有的域中.它可以是一个桥-在一个域内部或者一个边界交换机-域的.

* CIST域根是一个**边界交换机**被选举-为每个域-基于最短的外部路径开销-到达CIST总根的.路径开销是被计算的-基于开销-链路连接到域的.不包括内部的域内路径.CIST域根成为根-IST的-对于这个指定的域-同样的.

##CIST总根桥选举过程

* 当一个交换机启动了,它宣称自己是CIST总根和CIST域根,并且宣布这个事实-在出去的BPDU中.这个交换机将修正(adjust)它的决定-基于回信-有更好的信息,并且继续宣告这个最好的CIST总根和CIST域根在所有的**内部端口**.在边界端口,这个交换宣告的**只有**CIST总根的桥ID和CIST外部路径开销,这样隐藏了细节-关于域的内部拓扑的.

* CIST外部根路径开销是一个开销-去到达CIST总根的-穿过链路-连接到边界端口的.例如.域间的链路. 当一个BPDU被收到了-在一个内部的端口,这个开销(CIST外部根路径开销)是**不会变化的**.当一个BPDU被收到了-再一个边界端口,这个开销是会被调整的(adjusted)-基于这个收到的边界端口的端口开销.结果是,CIST外部根路径开销被宣扬-没有变化的-在任何域内部.

* 只有一个边界路由器能够被选举为CIST域根,且这个交换-有 最小的开销到达CIST总根.如果一个边界交换机听到了更好的CIST外部路径开销-接收在它的内部链路,它会释放(relinquish)它的角色-作为CIST域根的,并且开始宣布新的度量-在它的边界端口出去时.

* 每个边界交换机需要恰当的(properly)阻塞它的边界端口.如果该交换机是一个CIST域根,它选举一个边界端口作为"CIST总根端口",并且阻塞所有其他边界端口.如果一个边界交换机不是CIST域根,它将标记边界端口为CIST指定或者备用口.边界端口-在一个非域根桥上成为指定口-只有当它有更优先(superior)的信息-对于CIST总根:更好的外部路径开销或者当开销相同的时候有更好的CIST域根桥ID.这个情况是根据实际规则-STP处理过程(process)的.

* 作为结果-CIST的建立,每个域将有**一个交换机**有单个端口没有阻塞在这个方向-往CIST去的.这个交换机是CIST域根.所有的边界交换机将宣告这个域的CIST域根桥ID-往外-到他们的非阻塞的边界端口.从外面看(outside perspective),这整个域将看起来是个单独的虚拟桥,拥有桥ID==CIST域根的ID,并且有单独的根端口选举在CIST域根交换机上.

* 这个域-包含CIST总根的-将拥有所有的边界端口非阻塞,并且被标记为CIST指定端口.有效的-域应该被看做虚拟根桥-拥有桥ID等于CIST总根ID,并且所有的端口为指定的.注意到-这个域-有CIST总根-它的CIST域根等于CIST总根-因为他们拥有相同的最低的优先级-贯穿整个域.

看一下这个图片-在下面.他显示了(*demonstrate*)CIST拓扑计算-从物理拓扑-我们强调过的-在上面.首先,SW-1是被选举了-作为CIST总根-因为他有最低的桥ID-在所有的桥-在所有的域.这就自动的让域1成为了一个虚拟桥-它的所有的边界端口非阻塞.下一步,SW2-1和SW3-1被选举了-成为CIST域根在他们对应的域,每个域-没有包括CIST总根的-必须改变IST根选举流程-让IST根等于CIST域根.

![mstp-3-multi-region-cist-constructions.png](/images/mstp-3-multi-region-cist-constructions.png)

##公共生成树

从上面的信息,我们可以总结出-CIST(实质上)(essentially)组织于 两等级层次.第一个等级对待所有的域为"虚拟桥",并且操作-用外部路径开销.第一层生成树根子CIST总根桥,并且encompass(包括,围绕)虚拟桥.这棵生成树被得知-作为CST或者公共生成树.CST连接所有边界端口,并且perceive(感知,觉察)每个域-作为单独的虚拟桥-它的桥ID等于域根桥ID.

![mstp-3-multi-region-cst-and-vbridges.png](/images/mstp-3-multi-region-cst-and-vbridges.png)

CST是这样建设-这里MSTP互操作-与IEEE STP/RSTP域-也可以.历史的交换机域(jion with)组合它们的STP实例和CST,并且perceive(当做,感受)MSTP域为"透明的"虚拟桥,持续的不感知它们的内部拓扑.这样,连接到IEEE STP/RSTP域就扩张了CST.MSTP发现合适的(appropriate)STP版本-在边界链路-通过侦听外部,和(并发现)交换机对应的模式-用来操作(RSTP/STP).这会发生,一个交换机拥有最低的桥ID-属于RSTP/STP域.这个情况导致这个结果-所有的MSTP域选举本地CIST域根,然后认为最新的CIST总根位于MSTP"域"的外面.

第二个等级-CIST层次-组成为-不同的MSTP域内IST.每个MSTP域建造IST实例-使用内部路径开销-并且遵循最优的(*optimal*)"内部"拓扑,使用CIST域根作为IST根.这个变化-CST的-可能影响IST(们)在所有的域,因为这些变化可能导致重选-新的CIST域根.变化-各个域的内部拓扑的-通常不会影响CST,除非这些变化分裂(*partition*)了域.

##映射MSTI到CIST

MSTI(们)被建立-独立的-在每个域,但是它们必须被映射到CIST在边界端口.这就意味着(*inability*)地来负载平衡VLAN流量-在边界端口--通过映射VLAN(们)到不同的实例.所有的VLAN(们)使用同样的非阻塞边界端口,他们是-或是上行或是下行-对于CIST总根.这个情形(*statement*)只在这时有效-就是(with respect to)CST路径连接域虚拟桥.在任何域内部,VLAN们遵循内部逻辑路径,基于相关的(respective)MSTI配置.

这些MSTI没有知识-关于CIST总根-完全的;他们只用内部路径和内部MSTI根来建立生成树.然而所有的MSTP实例看到根端口(去往CIST总根的)-属于CIST域根的-被当做一个特殊的**主口(master port)**连接他们到CIST根桥.这个端口服务于这样的目的-"网关"-连接MSTI到其他的域.回忆下(Recall)交换机没有发送M-记录(MSTI信息)出到边界端口外,只有CIST信息.这样,CIST和MSTI可以收敛-独立的并且并行的(*parallel*).主口只开始转发-唯有当所有的对应的MSTI端口都同步了而且转发了-来阻止临时桥环路.

##MSTP多域设计考虑
..待完成

##互操作-与PVST+
..待完成

![mstp-3-mstp-and-pvst-interaction.png](/images/mstp-3-mstp-and-pvst-interaction.png)


##配置脚本1:CIST总根和CIST域根
..待完成

![mstp-3-multi-region-config-scenario.png](/images/mstp-3-multi-region-config-scenario.png)

##配置脚本2:MSTI和主口
..待完成

##配置脚本3:PVST+和MSTP互操作
..待完成

##结论

MSTP被设计成克服一个主要的问题-传统的STP协议的--无力(*inability*)使用阻塞链路-为流量转发-针对于单个的STP实例-目前来说(*present*).这被实现(*accomplished*)-通过跑多个生成树在一个拓扑,并且映射VLANs到不同的树上-来进行流量转发.尽管如此,这个特性没有允许(*precise*)和优先的流量工程,它改善了冗余链路(*utilization*).通过使用域,MSTP允许分立不同的物理拓扑-他们互相各自的-当去管理2层连接-域之间的.然而,即使有了增强的错误孤立,MSTP任然遭受着问题-(*inherent to*)固有的-以太网拓扑的:单播和广播泛洪以及缓慢的生成树的收敛.这些限制了MSTP部署(*deployment*)到小的2层域中,比如单的接入-分发交换块.大一点的MSTP部署应该被计划-仔细的,并且需要严格的管理控制.作为一个建议:私有VLAN可以被用作于大一点的域来最小化流量泛洪.

##进一步阅读
[IEEE 802.1 系列标准](http://standards.ieee.org/about/get/802/802.1.html)

[MSTP配置指导](http://www.cisco.com/en/US/docs/switches/datacenter/nexus5000/sw/configuration/guide/cli_rel_4_0_1a/MST.html)

[RFC 5517: 思科系统的私有VLAN](http://tools.ietf.org/html/rfc5517)

