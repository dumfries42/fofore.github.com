---
layout: post
title: 别让我思考
categories: [technology]
tags: [python, excel, escape, github, think] 
---

有一本书就叫[“别让我思考”](http://book.douban.com/subject/1827702/)。简单看了下，是网页设计的。就是说，不要让用户猜测你的设计意图。或者说是耗费用户的脑袋，因为用户还是都比较懒的。

昨天写完sortx（现在已经被改名为[escex](https://github.com/fofore/escex)）了，这个程序有6个必填参数，还必须进入到命令行模式才能够使用。

睡觉时候觉得非常不对劲：因为这个程序本来就是写给自己用的，一个月以后，可能我自己都忘记了参数的顺序了。显然，这个程序违背了“不要让我思考”的准则。

这个问题将简化到下面的情况：

1.  不再需要任何参数，只要将需要比较的两个csv文件拷贝到与'mergex.py'同一个目录下
2.  不要进入命令行，双击‘mergex.py'就可以

新的项目在这里，名字是[escex](https://github.com/fofore/escex)，就是escape excel的简单缩写。


