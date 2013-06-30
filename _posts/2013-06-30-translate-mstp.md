---
layout: post
title: 理解MSTP
category: story 
---

#1.动机
首先,这是一篇"奇葩"的翻译文.因为这篇文章采用的方法为"直译法",而且是非常直译,刻意而为之的直译.为何进行这样子的直译.其实本来是准备规规矩矩的将这篇文档翻译出来,因为网络上关于MSTP介绍的文章乏善可陈.所以使用goole英文搜索到这篇比较有诚意的文章:[Understanding MSTP](http://blog.ine.com/2010/02/22/understanding-mstp/).但是阅读过程遇到不少问题,关键还是英文思维的问题.因为这些句子都是比较长,如果整句读下来,再去用中文思维理解,非常之困难和费时间.而且如果遇到一些比较难懂的英文单词,整句话,就没有办法理解.于是我在想,是不是这样阅读英文文章的方法都是错误的.

最终,我发现了一个有趣的现象,当我改变了翻译方法,不再一整句一整句的想用中文表述出来,而是就跟着文章的英文语序,简单的进行转化到中文的变化,不但能够很快的理解文章,而且似乎还有了一点"英文思维"的感觉在里面,不知道这样的感觉是不是错觉呢?(大把流汗!)

不管是否是错觉,我举得还是要执行下去,最后看看效果,必须声明的是:这篇译文的结果看起来绝对是一窍不通的.但是为了督促自己工作,我有必要将它完成.所谓:善始者盖繁,克终者盖寡.(古文功底显露啦!)

那么,关于"英文思维"这个词呢,是我在用一整天看完了[外语学习的真实方法及误区](http://www.douban.com/group/topic/6767868),然后边看边总结,发掘出来,这是我当前来说,十万火急,刻不容缓的事情.

放置一个TODO在这边,如果将中英文对比着放置在这里,是不是会有更好的效果呢?(不如抽空就试试.)

对于不确定,还有我不熟悉的单词,我就用括号将它们暴露出来啦.(不要害羞.)这些内容对我来说是可理解+1的内容,都是重要的客户,接下来是要特别针对的,请颤抖.



--------------- **我是用来分割闲话和正文的分割线** -------------------

#理解MSTP

##介绍
..待完成

##表格-目录的
..待完成

##历史回顾
..待完成

##逻辑和物理拓扑(们)
..待完成

##增强MSTP
..待完成

##MISTP vs MSTP
..待完成

##M-Records (M-记录?)
..待完成

##MSTI树建造(建设)(construction)
..待完成

##STP争议(dispute)
..待完成

##注意事项(caveats)-关于MSTP域的

这里会有一些问题,由于这个事实-生成树实例没有被映射(一个一个的)到VLAN.使用PVST,剪裁一个VLAN在一个链路上,会同样禁止相应的STP在同样的链路上.既然MSTI脱离(decouple)VLAN,每个MSTI都跑在每个link上(域中的).这个在MSTI实例之间,差异在-他们做决定让哪条链路转发或者阻塞.通过剪裁(prune)VLAN,你可以终止于一个情形-VLAN没有使能在链路-(这些链路:相应的MSTI是转发或者使能,相应的MSTI阻塞). 考虑下面的例子用于解释这个想法.

![mstp-3-vlan-pruning-caveats.png](/images/mstp-3-vlan-pruning-caveats.png)

在这个拓扑中,VLAN被手动的剪裁了-在trunk中.TODO由于这个过滤是不符合相应(respective)的MSTI阻塞决定,VLAN2的流量是被阻塞了,在SW1和SW2之间.为了避免这种情况,不要用静态的"VLAN"剪裁方法-分配VLAN跨越trunk,当你有MSTP使能.一种情况相同于这个(描述的)就是当端口连接交换机是**接入(access)端口**.MSTP跑在这些端口上,并且有逻辑拓扑-可能是阻塞或者转发-在端口上.取决于VLAN到MSTI的映射,一个给定的VLAN可能被阻塞在接入端口-由于MSTP决定,尽管这个接入VLAN是不同的,他们使用相同的STP.为了避免这个问题,不要跑MSTP在接入端口,而且用他们于链接"末端"设备(只这样用)-例如,主机以及叶子(leaf)交换机. (同一个实例中的vlan,不要使用access端口, 同一个实例中不要将VLAN允许到不同的躯干(trunk)中)


