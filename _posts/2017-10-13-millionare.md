---
title: "理解姚氏百万富翁问题"
layout: post
excerpt: "两富人比谁钱多。如何能实现互相保密但可以比出谁钱多？"
categories: 解问题
tags:
  - Algorithm
  - Protocol
---

昨天在知乎上看到了一个问题：[两富人比谁钱多。如何能实现互相保密但可以比出谁钱多？](https://www.zhihu.com/question/66376147)

乍一看感觉不是什么特别复杂的问题，比如知乎用户[@辰舰长](https://www.zhihu.com/people/chen-f-32/activities)的答案：

> 用天平，每个富商领到一个质量固定的黑箱，让富商在其中放入质量与财富相对应的筹码（例如除以1000000）。放在天平两边看哪边重一点，就酱。

但更多的人饮用了文献：*Yao, Andrew C. "Protocols for secure computations." *Foundations of Computer Science, 1982. SFCS'08. 23rd Annual Symposium on*. IEEE, 1982.*。这个问题其实是著名的**姚氏百万富翁问题**，是计算机学家姚期智1982年的一篇论文。

> 姚期智(Andrew Chi-Chih Yao)，祖籍湖北省孝感市孝昌县，世界著名计算机学家，2000年图灵奖得主，中国科学院院士，美国科学院院士，美国科学与艺术学院院士，清华大学高等研究中心教授，香港中文大学博文讲座教授。1967年获得台湾大学物理学士学位，1972年获得美国哈佛大学物理博士学位，1975年获得美国伊利诺依大学计算机科学博士学位。1975年至1986年曾先后在美国麻省理工学院数学系、斯坦福大学计算机系、加利福尼亚大学伯克利分校计算机系任助理教授、教授。2004年起在清华大学任全职教授。2005年出任香港中文大学博文讲座教授。2017年2月，姚期智教授放弃外国国籍成为中国公民，正式转为中国科学院院士，加入中国科学院信息技术科学部。
>
> ——引自[百度百科](https://baike.baidu.com/item/%E5%A7%9A%E6%9C%9F%E6%99%BA/10170340?fr=aladdin)

在看论文之前还看了一些参考资料，比如知乎专栏[用密码学玩暗军棋 -- 闲聊多方计算](https://daily.zhihu.com/story/9304295)

，但对于其中的算法细节还是存疑的，以及公钥这样的概念个人比较含糊。干脆直接啃原文了（原文链接：[1982 749 AC Yao, Protocols for secure computations](https://wenku.baidu.com/view/fa0dfdf43186bceb19e8bbee.html)）。感谢[@能纵玄的孙权](https://www.zhihu.com/people/dai-chao-25/activities)对我的部分问题进行答疑解惑。

下面用尽量简单易懂的语言介绍一下这个算法的过程。

### 问题描述

假设富翁A和B分别有$i$，$j$亿资产，其中$0<i,j<=10$（上限可扩展到任意正整数），他们希望在彼此均不公布各自金钱数量的情况下，知道谁更富有。

（假设A和B都是君子，必须严格诚信执行下面的算法）

### 算法

设富翁A有4百万资产，富翁B有2百万资产，即$i=4,j=2$，且A和B互不知晓对方的资产。算法过程如下：

1.生成$N-bit$正整数集$M$，bit是二进制位，设$N=5$（实际操作中N应该根据$i，j$的上限取值，且尽量取大。为了书写方便这里的取值较小），生成正整数集$M$如下，共计16个数：

```
16，17，18，19，20，...，29，30，31
```

2.生成$M-M$的所有一一映射集合$Q_N$。则步骤1的例子中，得到的$Q_N$含$16!$个元素。

3.富翁A随机选取$Q_N$中的一个元素作为公钥（用于加密传输的加密方式），并告知B**（注意：这个映射关系的反向就是私钥，A不能将此私钥信息告诉B）**，不妨设A选择的公钥为：

```
16-24，17-16，18-29，19-31，20-17，21-22，22-27，23-28，24-18，25-30，26-25，27-19，28-23，29-26，30-21，31-30
```

4.富翁B随机选择一个$N-bit$正整数$x$，因为$x\in M$，所以B可以在公钥中找到$x$的对应元素$k$。设$x = 20$，则对应的$k=17$。

5.B将$k-j$的计算结果发给A，即这里B将发送15。

6.A通过第三步的加密／解密方法，将$k-j+1,k-j+2,…,k-j+10$逐个解密，计作序列$y_u$：

```
yu = [17,20,24,27,31,30,21,28,16,26]
```

​	并选取$N/2-bit$的素数$p$，将$y_u$中每个元素除以$p$得到的余数（即$y_i\ mod\ p$）计作序列$z_u$，若$z_u$中至少有两个元素不相同则$p$满足条件，否则更换$p$。这里取$p=3$，则$z_u$如下：

```
zu = [2,2,0,0,1,0,0,1,1,2]
```

7.A将上述$z_u$序列进行处理：保持前$i$个数字不变，从第$i+1$个数到最后一个数全部增加1。并将处理过的序列$z_u'$和素数$p$发给B，如下：

```
zu' = [2,2,0,0,2,1,1,2,2,3], p = 3
```

8.B接收$z_u'$和$p$，查看$z_u'$中的第$j$个元素$z_u'(j)$，若$x\ mod\ p = z_u'(j)$，则$i\geq j$，反之，$i<j$。这里由于$j=2$，因此查看$z_u'(2) = 2 = 20\ mod\ 3$，因此得到结论$i \ge j$。

9.B将结果告诉A，算法结束。

### 正确性

第一遍看这个算法的时候，虽然自己按照上面的过程带入数值模拟了一遍，但没有想通为什么这样做是正确的。思考了一阵后豁然开朗。

因为$0<j\le10$，因此在序列$k-j+1,k-j+2,…,k-j+10$中，一定有第$j$个数等于$k$，即将这10个数全部解码后，一定有$y_u(j) = x$，即：
$$
z_u(j)=y_u(j)\ mod\ p = x\ mod\ p
$$
只要$z_u'(j)$没有被增加1，就有$x\ mod\ p = z_u'(j)$，也就说明了$i\ge j$。

反之，若：$x\ mod\ p\neq z_u'(j)$，即第$j$个数被A处理过了，那么B就知道$i\le j$。

### 说明

> 在上面的百万富翁问题的解法，第6步是取十个明文除以素数p的余数。这一步可以省略吗？

回答是：不能省略！

假设省略了素数$p$的参与，直接进入第七步，A将得到：

```
yu' = [17,20,24,27,32,31,22,29,17,27]
```

并发给B。

B在查看了第2个元素后发现与$x$一致，B就知道A的资产比自己多了。

这个时候B不甘心：我倒要看看你A有多少钱！

B用A给的公钥将$y_u'$中的数字全部编码，依次得到：`16，17，18，19`这样的自然序列，在第5个元素的时候，发现找不到对应的加密方式了（数字32在本例中产生了溢出，如果序列范围更大理论上可以避免），第6个元素对应的密文为20，第7个元素对应密文为27...从第5个元素开始就已经被A处理过了，于是B知道了，A的资产是4百万。

从这个过程我们看出，一定需要合适的素数$p$来对A的信息加密。

### 小结

引用[Vichare Wang](https://www.zhihu.com/question/66376147/answer/242254093)的一段回答：

> 第一步，经过特定的操作，让A构造出 n 把锁，B有且仅有第 j 把锁的钥匙，但是A不知道 j 是多少；
>
> 第二步，A给B n 把锁锁着的标志位，其中前 i 个标志位置0，后 n-i 个置1；
>
> 第三步，B检查第 j 把锁锁着的标志位是否为0。如果为0则 i >= j，否则 i < j。

这差不多就是这个算法的精髓所在。

原论文还讨论了防作弊、多方比较等通用方案，本文不再赘述。

回到这篇文章：[用密码学玩暗军棋 -- 闲聊多方计算](https://daily.zhihu.com/story/9304295)，如果真的要用密码学玩暗军棋，则一定是需要程序解决的。上面算法中也举出了简单的例子，这个算法看似不复杂，但手工计算和处理还是挺麻烦的。



参考文章：

[公钥与私钥](http://www.blogjava.net/yxhxj2006/archive/2012/10/15/389547.html)