---
layout: post
title:  "x-up-bear-type 你是什么鬼"
author: "Zongquan"
comments: true
description: "部分iOS客户反应：app出现错误提示：网络错误1005。"
---

### 起因

在一个客户，使用我司的APP保护后，部分iOS客户反应：app出现错误提示：网络错误1005。

```
1005在iOS上的定义：1005 : "The network connection was lost." 即网络连接丢失
```

### 分析

整个保护系统的网络TOP如下：

```
APP  ==>  F5 ==> 我司防护服务器 ==> APP Server
```

理论上来说直接抓包就可以分析出具体问题，现在我司防护服务器抓包试试，分析数据包发现，所以数据包都是正常的，那基本可以把问题向前推，即问题是APP与F5之间发生了什么是导致的。

继续在F5上抓包分析，果不其然，的确有异常。

![](/assets/images/1005_APP_TO_F5.png)

**127**为Client，**172.16.2.83**为F5设备。

F5中所有异常的TCP流，都如上图所示，Client在与F5 TCP握手完成之后，Client立即向F5发了一个RST包，并且Seq number是从1289开始。一看这个1289就是一个奇怪的东西，Seq number是握手完后发的数据的值不应该是1289，显然在这个RST包之前，Client肯定是发了一个1288的数据包才对。

带着这个怀疑，看上尝试在手机APP端进行抓包来分析。

其实如果手机端能直接重现问题抓包就比较好分析，问题的关键是，这个异常情况不是必现的，几番折腾后确定在使用4G流量的时候才有概率出现，并且终于抓到了异常数据包。

iPhone抓包方法：

```
1. 将手机通过USB连接MAC，打开iTunes，然后找到该手机的UUID拷贝出来。
2. 在命令行终端使用命令:
 rvictl -s 12321908743243242ABDCBABD1213198(这是UUID)
 ifconfig -l找到关联到MAC的网卡，一般名字叫rvi0
3. 使用tcpdump对rvi0 进行抓包:
 sudo tcpdump -i rvi0 host IP -w ~/xxxx.pcap
4. 完事后删除网卡：
 rvictl -x 12321908743243242ABDCBABD1213198
```

异常数据包如下：

![](/assets/images/1005_iPhone_TO_F5.png)

**203**为iPhone，**83**为目标服务器。

对单个数据流进行分析，iPhone在与Server TCP握手完成之后，发送了一个Http请求，header部分因为MTU的限制(MSS=1300)，header被拆成了两个包发送，第一个len为1288，第二个len为533，接着是发送的body的内容。

而在iPhone刚发送出这几个数据包后，立刻收到了Server的RST，与在F5上的抓来看，F5也是收到了RST包，那么可以说明整个链路中连接的断开，既不是iPhone主动发出的，也不是F5发出的，很显然是一个中间设备因为某一种策略将iPhone和F5的连接都重置断开了。由于第一个数据包长度都为1288，这也解释了之前F5上收到的RST时Seq Number为1289的情况。

客户反复咨询了网络中心，说F5就是直接暴露到公网的，F5前面没有防火墙和其它设备了。怎么办，那这个问题怎么解决呢，解决不了可能会导致我司的设备直接被下线，只有继续在跟踪问题。

### 进一步分析

是否这个问题只会出现在iPhone上？这个问题是否必现？什么原因导致的？带着这样的疑问我们又进一步测试了多种场景。

**与MTU的值，MSS值是否有关？**

结论是无关

**与Header的Key：x-up-bear-type 有关：**

最终一个Header一个Header的屏蔽，发现与x-up-bear-type有关，当http请求的header长度(body以上的部分包括url)超过1272的时候，并且header中有Key：x-up-bear-type，必现该问题；但当header小于1272时，增加Body的长度，并不会出现该问题；去掉x-up-bear-type，增加header的长度也不会出现该问题。

iOS、Android、Burp、curl都能重现该问题。

iOS：

![](/assets/images/1005_iOS.png)

Android：

![](/assets/images/1005_Android.png)

那我想问，x-up-bear-type你是什么鬼。最终让客户在header中干掉这个key后，问题得到解决。