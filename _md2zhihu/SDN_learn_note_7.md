# SDN学习笔记

[TOC]

> 参考链接：[【SDN】理论概念 - 【SDN】教程资料 - 《通信系统》 - 极客文档 (geekdaxue.co)](https://geekdaxue.co/read/wangxi_chn@iku3gm/fv9ans)
> 
> [Mininet+Ryu安装及GUI可视化（ubuntu20.04）](https://blog.csdn.net/qq_39768922/article/details/123833664)
> 
> （1）http://osrg.github.io/ryu/resources.html
> 我比较喜欢里里面的 http://ryu.readthedocs.org/en/latest/
> 当然里面的电子书也是相当好的：http://osrg.github.io/ryu-book/en/html/
> （2）推荐一个小伙伴的博客：linton.tw
> 他在RYU上有更多的学习和研究。欢迎访问！
> （3）课程《RYU应用开发入门》，以视频直播的形式剖析、讲解、实验、答疑。这种形式能够比较快的完成在RYU的应用方面从0到1的突破。课程详情页：https://edu.sdnlab.com/training/189.html
> 
> [【SDN】Ryu网络拓扑可视化app使用简介_如何运行ryu app-CSDN博客](https://blog.csdn.net/qq_41854763/article/details/84639001)
> 
> [提高SDN控制器拓扑发现性能 - cotyb - 博客园 (cnblogs.com)](https://www.cnblogs.com/cotyb/p/5067844.html)
> 
> [流表中 idle_timeout、hard_timeout、idle_age、duration 的区别](https://www.jianshu.com/p/22e6feb2c662)


## 0. SDN

### 0.1 SDN概述

SDN（Software Defined Network）是一种将网络控制功能与转发功能分离、实现控制可编程的新兴网络架构,  这种架构将从控制层从网络设备转移到外部计算设备, 使得底层的基础设施对于应用和网络服务而言是透明的、抽象的，网络可被视为一个逻辑的或虚拟的实体。

-   ONRC（开放网络研究中心）的定义：“SDN是一种逻辑集 中控制的新网络架构，关键属性包括：数据平面与控制平面分离；控制平面与数据平面有统一的开放接口 OpenFlow”
-   ONF（开放网络基金会）认为：“SDN是一种支持动态、 弹性管理的新型网络体系结构，是实现高带宽、动态网络的理想架构。SDN将网络的控制平面与数据平面相分离，抽象了数据平面的网络资源，并支持通过统一的接口对网络直接进行编程控制。”

ONF 定义的架构共由四个平面组成即数据平面、控制平面、应用平面以及右侧的控制管理平面, 各平面之间使用不同的接口协议进行交互

![image-20240811222552557](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/8ea2b39ae080e21d-image-20240811222552557.png)

-   【数据平面】
    -   由若干网元组成，每个网元可以包含一个或多个SDN Datapath。
    -   每个SDN Datapath是一个逻辑上的网络设备，它没有控制能力，只是单纯用来转发和处理数据
    -   它在逻辑上代表全部或部分的物理资源
    -   一个SDN Datapath包含控制数据平面接口代理、转发引擎表和处理功能三部分。
    -   基础设施层由网络中的物理交换机组成，这些交换机将网络流量转发到它们的目的地。
    -   代表之一是开源社区的 Open vSwitch。值得一提的是，与 DPDK（由英特尔联合第三方软件开发公司推出的一款数据面开发套件）结合之后，Open vSwitch 在某些应用场合展示了优异的线性交换能力，逼近硬件的性能，而软件在可重配可扩展方面的潜力却是硬件无法比肩的，具有非常广阔的应用前景。

-   【控制平面】
    -   即所谓的SDN控制器，充当软件定义网络的大脑
    -   该控制器驻留在服务器上并管理整个网络的策略和流量
    -   SDN控制器是一个逻辑上集中的实体，它主要负责两个任务
        -   一是将SDN应用层请求转换到SDN Datapath
        -   二是为SDN应用提供底层网络的抽象模型（可以是状态、事件）

    -   一个SDN控制器包含北向接口代理、SDN控制逻辑以及控制数据平面接口驱动三部分。
    -   SDN控制器只是要求逻辑上完整，因此它可以由多个控制器实例组成，也可以是层级式的控制器集群
    -   从地理位置上讲，既可以是所有控制器实例在同一位置，也可以是多个实例分散在不同的位置。
    -   比较著名的控制器有 OpenDayLight（目前成熟度最高的开源平台）和 ONOS（面向运营商领域，在集群、可靠性和性能方面做了增强）。

-   【应用平面】
    -   由若干SDN应用组成，SDN应用时用户关注的应用程序。
    -   它可以通过北向接口与SDN控制器进行交互，即这些应用能够通过可编程方式把需要请求的网络行为提交给控制器。
    -   一个SDN应用可以包含多个北向接口驱动（使用多种不同的北向API），同时SDN应用也可以对本身的功能进行抽象、封装来对外提供北向代理接口，封装后的接口就形成了更为高级的北向接口。
    -   应用层包含组织使用的典型网络应用或功能包括OSS（Operation support system 运营支撑系统）、OpenStack架构的云平台等，还可能包括入侵检测系统、负载均衡或防火墙。
    -   传统网络将使用专用设备，例如防火墙或负载均衡器，而软件定义的网络则用使用控制器来管理数据平面行为的应用程序替换设备。
    -   被 5G 网络架构采用是 SDN 发展史上的又一重要里程碑（虽然 3GPP 将会定义新的南向接口）。在 4G/LTE 设计之初，由于业务不丰富以及用户的数据需求不高，分组核心网被设计成以下架构。

### 0.2 SDN LLDP协议

[SDN拓扑发现原理_sdn控制器的lldp协议-CSDN博客](https://blog.csdn.net/songfeihu0810232/article/details/116480876)

[LLDP帧格式](https://support.huawei.com/enterprise/zh/doc/EDOC1100174722/9f322d1)

## 1. OpenFlow 协议概述

### 1.1 开源控制器

![image-20240808132257724](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/28f2b29acf863c73-image-20240808132257724.png)

### 1.2 核心组件

#### 1.2.1 流表  FlowTable

OpenFlow v1.0 流表组成

![image-20240808132731763](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/5814bf3d75aea323-image-20240808132731763.png)

-   流表是 OpenFlow 交换机的核心部分，负责数据包的高速查询和转发，相当于传统网络中的转发表
-   在传统网络设备中，数据链路层和网络层其各自的转发依据分别为 MAC 地址表和 IP 地址路由表, 而在 OpenFlow 中，流表将传统网络中的所有转发表进行了整合，因此，其支持的协议字段更加丰富
-   流表由流表项构成
-   匹配域（Match Fields）指定了数据包具体的匹配规则如图所示，相当于传统网络中的数据包头

![image-20240808132838089](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/6576817e7adc7cb2-image-20240808132838089.png)

-   优先级（Priority）决定了流表项的优先级，优先级高的流表项会优先用来匹配数据包
-   计数器（Counters）用来统计流表项被匹配查询的次数，用于统计匹配数据包个数，方便流量流量监管
-   指令（Instruction）用来指示数据包匹配成功后要执行的具体动作，分为必备动作和可选动作
    -   必备动作是指所有 OpenFlow 交换机默认支持的，可选动作是指部分交换机支持的动作
    -   必备动作有两种: 转发和丢弃
    -   可选动作有三种: 转发、排队和修改域

-   超时（Timeouts）表示流表项最大的超时时间值或者流表项空闲的时间值
-   缓存（Cookie）用于配合匹配域完成数据包的匹配。
-   在接收到数据分组时，交换机从标号为 0 的流表开始对数据包头进行匹配
-   按照流表项的优先级与匹配域中的属性逐条进行比对，直至在流表中匹配到一个流表项
-   具体的流表匹配流程如图

![image-20240808133047452](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/31b46ececf4a5568-image-20240808133047452.png)

-   交换机从数据包的头中提取用于匹配的字段，匹配字段通常包括各种报头字段，如 MAC 地址、IP 地址、端口号等。
-   若匹配成功，则将该流表项中计数器、指令集一并进行更新，同时根据指令来决定处理流程是否转移到下一个流表。
-   若需要转移到下一个流表中继续进行匹配，则下一个流表号 n 要大于当前流表号i 。
-   当匹配完成后，则执行动作集合，通常是转发或者修改数据包。
-   Table-miss 用于处理与该流表中流表项未匹配成功的情况，由该流表项决定将数据包丢弃、上传给控制器或是转交给下一个流表。
-   若在一个流表中不存在 Table-miss 项，则默认将报文丢弃。

#### 1.2.2 安全通道 DataPath

OpenFlow交换机示意图

![image-20240808132523510](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/585a13d2c3100057-image-20240808132523510.png)

-   OpenFlow 交换机作为基本的数据转发单元是数据层的主要网络设备
-   OpenFlow 交换机和 OpenFlow 控制器通过安全通道进行通信，通信一般采用安全传输层协议进行加密保护
-   OpenFlow 交换机的核心功能就是把到达某个端口的分组从另一个端口转发出去，图中对应为从端口 2 进入到交换机的分组最终从端口 N 转发出去。
-   分组进入到交换机的本地入口后首先会进行分组匹配，即与流表中的流表项进行匹配，当分组找到了匹配的表项会进入图中流表右侧的动作盒中.
-   对于收到的分组，OpenFlow 交换机的动作盒在处理时有三种基本选择，动作 1 表示从从本地端口把分组转发出去，动作 2 表示丢弃该分组，动作 3 表示把分组转发到控制器，询问控制器如何处理该分组。

#### 1.2.3 OpenFlow 协议

-   OpenFlow 协议定义了控制器与交换机之间传输的特定消息及消息格式。

-   OpenFlow 协议是控制器与 OpenFlow 交换机进行通信的标准，它在安全通道进行传输

-   其定义的通信消息类型有 3 种：Controller-to-Switch 消息，Asynchronous(异步的) 消息和 Symmetric (对称的)消息

    -   1）Controller-to-Switch 消息由控制器发起，用来获取管理OpenFlow 交换机的运行状态
    -   Features：用于在控制器和交换机之间建立传输层安全会话。
    -   Configuration：用于控制器查询或设置交换机上的配置信息。
    -   Modify-state：用于控制器管理交换机中的流表项和端口状态等。
    -   Read-state：用于控制器向交换机请求一些统计信息例如流、数据包等。
    -   Packet-out：用于控制器通过交换机的指定端口发送数据包。
    -   2）Asynchronous 消息由交换机发起，用来将网络事件或交换机的状态变化传递到控制器
    -   Packet-in：当交换机收到一个数据包，在流表中没有匹配项时则发送该消息询问控制器如何处理。
    -   Flow-removed：当交换机中的流表项因超时等原因被删除时，会触发。
    -   Port-status：交换机端口状态发生变化时，会触发。
    -   Error：交换机发生问题时，会触发。
    -   3）Symmetric 消息可由控制器或交换机发起。
    -   Hello：用于控制器和交换机之间建立连接。
    -   Echo：用于测量延迟、双方是否保持连接等。

-   **Openflow协议定义的端口全面解读:https://www.jianshu.com/p/7eb86d164d26** 十分重要

## 2. 安装RYU Python SDN控制器

### 2.1 pip安装

```
pip install ryu
sudo apt install python3-ryu
```

### 2.2 源码安装

```
git clone https://github.com/faucetsdn/ryu.git
cd ryu; pip install .
```

-   https://www.cnblogs.com/wangxiaozhang/p/12274186.html)

### 2.3 快速测试

```
# 开启控制器，使用二层交换
python3 ~/ryu/ryu/app/simple_switch.py
ryu-manager simple_switch.py
# 启动Mininet
sudo mn --controller=remote
# 运行成功并且 pingall测试
pingall
```

![image-20240721175151452](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/61de1ab004cbc33d-image-20240721175151452.png)

![image-20240721175216193](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/05f29004703d4b70-image-20240721175216193.png)

## 3. Ryu控制器教程

### 3.1 简介

Ryu是一种基于Python语言的软件定义网络(SDN)控制器，它提供了一个开放的应用程序接口(API)，使 网络管理员Q和开发人员能够轻松地编写新的网络控制应用程序来进行网络流量的控制和管理。Ryu通过OpenFlow协议与网络交换机通信，可以对网络设备进行配置、监控和管理。Ryu是一个开源项目，由日本NTT实验室开发和维护，被广泛应用于SDN应用程序的开发和部署。

![image-20240721173505430](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/5e506d0f30d58fe2-image-20240721173505430.png)

**特点：**

-   1.简单易用:Ryu采用Python语言编写，易于学习和使用，提供了丰富的AP!和文档，使开发人员能够快速地构建自己的SDN应用程学
-   2.高度可扩展:Ryu提供了一个基于插件的架构，使开发人员可以方便地扩展和定制SDN应用程序的功能。
-   3.支持多种OpenFlow协议版本:Ryu支持多种OpenFlow协议版本，包括1.0、1.3和1.4版本，使其可以兼容不同厂家的网络交换机。
-   4.高性能:Ryu采用异步I/O模型和事件驱动的设计，提供了高性能的网络流量控制和管理能力。
-   5.开放源代码:Ryu是一个开源项目，可以自由获取代码并进行修改和定制，有助于推动SDN技术的发展和普及

**功能：**

-   网络拓扑发现和管理：RYU可以发现和管理SDN网络中的拓扑结构，包括交换机、主机、链路和路径等信息。
-   网络流量控制和管理：RYU可以控制网络流量的转发和策略，支持流表、QoS、ACL等功能，可以实现网络流量的精确控制和管理。
-   网络安全和监控：RYU可以对网络流量进行监控和分析，能够识别和处理恶意流量和攻击行为，提高网络安全性。
-   网络服务质量（QoS）保障：RYU可以实现基于流量的服务质量保障，包括带宽限制、拥塞控制、流量分类等功能。
-   网络编程和应用开发：RYU提供了丰富的API和SDK，支持Python编程语言，可以方便地开发和部署SDN应用程序，如基于SDN的网络监控、负载均衡、流量控制和优化等。

### 3.2 RYU源码简介

![image-20240721173816928](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/dde35827e748a61e-image-20240721173816928.png)

-   base

base中有一个非常重要的文件：**app_manager.py**，其作用是RYU应用的管理中心。用于加载RYU应用程序，接受从APP发送过来的信息，同时也完成消息的路由。
**app_manager.py**中主要的函数有app注册、注销、查找、并定义了RYUAPP基类，定义了RYUAPP的基本属性：包含name, threads, events, event_handlers和observers等成员，以及对应的许多基本函数。如：start(), stop()等。

-   controller

controller文件夹中许多非常重要的文件，如events.py,  ofp handler.py,  controller.py等。
其中controller.py中定义了OpenFlowControler基类。用于定义OpenFlow的控制器，用于处理交换机和控制器的连接等事件，同时还可以产生事件和路由事件。其事件系统的定义，可以查看events.py和ofp_events.py。
在ofp_handler.py中定义了基本的handler句柄，完成了基本的如:握手，错误信息处理和keep alive 等功能。更多的如packet in handler应该在app中定义。
在dpset.py文件中，定义了交换机端的一些消息，如端口状态信息等，用于描述和操作交换机。如添加端口，删除端口等操作。

-   Lib

lib中定义了我们需要使用到的基本的数据结构，如doid.mac和ip等数据结构，在lib/packet月录下，还定义了许多网络协议，如ICMPDHCP.MPLS和IGMP等协议内容，而每一个数据包的类中都有parser和serialize两个函数。用于解析和序列化数据包。ib目录下，还有ovs,netconf目录，对应的目录下有一些定义好的数据类型，不再述。

-   Ofproto

在这个目录下，基本分为两类文件，一类是协议的数据结构定义，另一类是协议解析，也即数据包处理函数文件。 如ofproto_v1_0.py是1.0版本的OpenFlow协议数据结构的定义，而ofproto v1 0 parser,py则定义了1.0版本的协议编码和解码。具体内容不赘述，实现功能与协议相同。

-   Topology

包含了switches.py等文件，基本定义了一套交换机的数据结构。
event.py定义了交换上的事件。
dumper.py定义了获取网络拓扑的内容。
最后api.py向上提供了一套调用topology目录中定义函数的接口。

-   contrib

这个文件夹主要存放的是开源社区贡献者的代码。

-   cmd

定义了RYU的命令系统，具体不赘述，

-   services

完成了BGP和vrrp的实现

-   tests

tests目录下存放了单元测试以及整合测试的代码

-   app

定义了各种网络北向应用

### 3.3 RYU应用基本编写流程

#### 3.3.1 Hub 应用代码

```python
from ryu.base import app_manager
from ryu.ofproto import ofproto_v1_3
from ryu.controller import ofp_event
from ryu.controller.handler import MAIN_DISPATCHER,CONFIG_DISPATCHER
from ryu.controller.handler import set_ev_cls


class Hub(app_manager.RyuApp):
    '''明确控制器所用OpenFlow版本'''
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    def __init__(self,*args,**kwargs):
        super(Hub,self).__init__(*args,**kwargs)

    @set_ev_cls(ofp_event.EventOFPSwitchFeatures,CONFIG_DISPATCHER)
    def switch_features_handler(self,ev):
        '''
        在Ryu控制器上，我们需要写一个函数去处理openvswitch的连接
        CONFIG_DISPATCHER : Version negotiated and sent features-request message
        '''
        #对事件进行解析
        datapath = ev.msg.datapath    #从连接中获取数据平面的datapath数据结构
        ofproto = datapath.ofproto    #获取OpenFlow协议信息
        ofp_parser = datapath.ofproto_parser    #获取协议解析
        #解析完成

    	'''在连接建立成功以后，需要控制器下发一个默认流表
           来指挥所有匹配不到交换机的数据，把他上传到控制器上
        '''

        #install the table-miss flow entry

        match = ofp_parser.OFPMatch()        #匹配域

        #OFPActionOutput将数据包发送出去,
        #第一个参数OFPP_CONTROLLER是接收端口，
        #第二个是数据包在交换机上缓存buffer_id,由于我们将数据包全部传送到控制器，所以不在交换机上缓存
        actions = [ofp_parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,ofproto.OFPCML_NO_BUFFER)]

        self.add_flow(datapath,0,match,actions,"default flow entry")   			#默认缺省流表项，设置优先级最低即可
    '''
    数据平面是由若干网元（Network Element）组成，每个网元包含一个或多个SDN数据路径（SDN Datapath）。
    SDN Datapath是逻辑上的网络设备，负责转发和处理数据无控制能力，一个SDN DataPath包含控制数据平面接口（Control Data Plane Interface,CDPI）、代理、转发引擎（Forwarding Engine）表和处理功能（Processing Function） 
    SDN数据面（转发面）的关键技术:对数据面进行抽象建模。
    '''
    def add_flow(self,datapath,priority,match,actions,remind_content):
        '''构建流表项 : add a flow entry, install it into datapath
        datapath:表示给哪一个逻辑设备下发流表
        priority：表示优先级
        match,actions:匹配域和动作
        '''

        #datapath属性        
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        #在OpenFlow1.3版本中定义了instruct指令集（交换机内部的一些操作）
        #construct a flow msg and send it
        inst = [ofp_parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,
                                             actions)]

        mod = ofp_parser.OFPFlowMod(datapath=datapath,priority=priority,
                                    match=match,instructions=inst);
        print("install to datapath,"+remind_content)
        #发送出去
        datapath.send_msg(mod);


    '''接收数据
    Ryu控制器通过装饰器去注册监听某些事件，去处理这些事件。
    从而实现从数据平面的消息上传到控制器，再从控制器平面到应用平面，应用程序去处理事件，再逐跳返回到openvswitch
    '''

    '''要处理这个事件，需要先去注册监听他
    EventOFPPacketIn: 是我们要监听的事件
    MAIN_DISPATCHER : 是什么状态下，去监听该事件---Switch-features message received and sent set-config message
    '''
    @set_ev_cls(ofp_event.EventOFPPacketIn,MAIN_DISPATCHER)
    def packet_in_handler(self,ev):
        '''Hub集线器类，所实现的功能：
        1.接收从OpenVSwitch发送过来的数据包
        2.将数据包泛洪到Hub中的其他端口中
        '''    

        #解析数据结构
        msg = ev.msg
        datapath = msg.datapath
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser
    	#获取源端口
        in_port = msg.match['in_port']        

        print("get packet in, install flow entry,and lookback parket to datapath")
         #因为我们是将所有转发，所以不用管匹配，填空表示全部匹配
        match = ofp_parser.OFPMatch();       
        #注意：FLOOD 是OpenFlow协议保留端口－－－泛洪使用
        actions = [ofp_parser.OFPActionOutput(ofproto.OFPP_FLOOD)]　　

        #调用add_flow,将流表项发送,指导后续数据包转发 install flwo entry to avoid packet in next time
        self.add_flow(datapath,1,match,actions,"hub flow entry")    #等级稍微比默认流表项高级

        # 注意：我们将流表项下发了，但是数据包我们这次接收的，并没有处理
        # 就是再将控制器上的数据包，重新发送给datapath,让他按照流表项处理
        # buffer_id是这个数据包，存放在控制器中的缓冲区位置,是在事件中的buffer_id获取
        req = ofp_parser.OFPPacketOut(datapath=datapath,buffer_id=msg.buffer_id,                                            in_port=in_port,actions=actions,data=msg.data)    
        datapath.send_msg(req);
```

#### 3.3.2 Hub 应用分析

-   1.当开始一个Hub集线器时，会先与控制器进行连接，我们需要在Ryu中设置函数去处理连接，设置并下发默认流表---------函数switch_features_handler实现

-   2.当主机之间通信时，主机上传信息到OpenVSwitch交换机，而交换机无法匹配到流表项时，我们设置将数据全部上传给Ryu控制器，我们在控制器端实现Hub集线器的泛洪功能，即设置流表项（match-actions为所有匹配数据包的动作为ofproto.OFPP_FLOOD，并且将该流变下发给原来datapath,同时我们要将之前交换机发送过来的数据包重新发送给交换机（让其按照新的流表项进行处理）--------函数packet_in_handler实现

-   3.构建公共函数add_flow，构建流表项并且下发流表提出

#### 3.3.3 应用编写教程

##### (1) 定义RYU 应用

-   `pp_manager.RyuApp`是所有`Ryu Applications`的基类，我们要实现一个控制器应用，必须继承该基类: `class Hub(app_manager.RyuApp)`

-   我们自定义的子类（继承于RyuAPP的子类），将在ryu-manager命令加载中被实例化(它是在ryu管理器加载所有请求的ryu应用程序模块后实例化的):

-   子类中的__init__方法需要调用父类的__init__方法，并且保持参数一致:

```python
def __init__(self,*args,**kwargs):
        super(Hub,self).__init__(*args,**kwargs)
```

##### (2) 设置OpenFlow协议版本

```python
from ryu.ofproto import ofproto_v1_3

OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]
```

##### (3) 事件监听处理函数定义与内容编写流程

-   定义事件监听处理函数:

```python
@set_ev_cls(ofp_event.EventOFPSwitchFeatures,CONFIG_DISPATCHER)
def switch_features_handler(self,ev):
    pass
```

-   获取逻辑设备datapath类实例: `datapath = ev.msg.datapath`
-   获取逻辑设备datapath中OpenFlow协议信息: 	`ofproto = datapath.ofproto`
-   获取逻辑设备datapath中OpenFlow协议解析器: `ofp_parser = datapath.ofproto_parser`. `ofp_parser`类定义在`ryu.ofproto.ofproto_v1_3_parser`模块中.
-   通过ofp_parser中定义match匹配类实例:`match = ofp_parser.OFPMatch()`
-   通过ofp_parser中定义actions动作类实例列表: `actions = [ofp_parser.OFPActionOutput(ofproto.OFPP_FLOOD)]`
-   通过`add_flow`函数下发流表到交换机:`self.add_flow(datapath,0,match,actions,"default flow entry")`   默认缺省流表项，设置优先级最低(即为0)即可
-   对于控制器从交换机处收集上来的数据包可以通过`ofp_parser.OFPPacketOut`处理:`req=ofp_parser.OFPPacketOut(datapath=datapath,buffer_id=msg.buffer_id,in_port=in_port,actions=actions,data=msg.data)`.控制器使用此消息通过交换机发送数据包（存在控制器的消息缓存，我们需要发送给交换机，让他进行发送）---注意我们不仅要传递buffer_id还要传递data才可以实现将数据返回
-   datapath执行从控制器处接收到的指定端口转发请求消息(req) ,在OpenFlow协议中 Packet-out消息：用于控制器通过交换机的指定端口发送数据包。`datapath.send_msg(req);`

##### (4) 关键函数解读

###### A. 事件监听与处理装饰器
`from ryu.controller.handler import set_ev_cls`

```python
def set_ev_cls(ev_cls, dispatchers=None):
    """
    RYU应用装饰器,用于声明一个事件处理函数.被装饰的函数将称为一个事件处理器event handler.
    ev_cls:事件类别,Ryu 应用想要监听并处理的事件类别, 一般事件定义在ofp_event类中,例如ofp_event.EventOFPPacketIn 事件
    dispatchers:声明处理的事件类别所属的OpenFlow 协议定义的协商流程阶段Phase
    """
    def _set_ev_cls_dec(handler):
        if 'callers' not in dir(handler):
            handler.callers = {}
        for e in _listify(ev_cls):
            handler.callers[e] = _Caller(_listify(dispatchers), e.__module__)
        return handler
    return _set_ev_cls_dec
```

###### B. OpenFlow协商阶段解读

<table>
<tr class="header">
<th style="text-align: center;">协商阶段</th>
<th style="text-align: center;">描述</th>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>ryu.controller.handler.HANDSHAKE_DISPATCHER</code></td>
<td style="text-align: center;">发送并等待Hello消息</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>ryu.controller.handler.CONFIG_DISPATCHER</code></td>
<td style="text-align: center;">版本协商以及发送features-request 消息</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>ryu.controller.handler.MAIN_DISPATCHER</code></td>
<td style="text-align: center;">Switch-features 消息接收以及发送set-config消息</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>ryu.controller.handler.DEAD_DISPATCHER</code></td>
<td style="text-align: center;">与对端断开连接</td>
</tr>
</table>

![image-20240808140638914](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/f65941d6cd5bc6b3-image-20240808140638914.png)

-   双方通过握手消息Hello建立安全连接
-   双方建立TLS隧道后，方法发送hello消息进行版本协商,如果协议版本协商成功，则连接建立。否则发送Error消息描述协商失败原因，并终止连接
-   协商完成后，控制器和交换机之间发送Features消息,获取交换机参数(参数包括支持的buffer数目、流表数、Actions等)
-   控制器发送SET_CONFIG消息向交换机发送配置参数
-   通过GET_CONFIG消息得到交换机修改后的配置信息
-   控制器与OpenFlow交换机之间，发送PACKET_OUT和PACKET_IN消息。通过PACKET_OUT中内置的LLDP包进行网络拓扑的探测
-   控制器通过FLOW_MOD向控制器下发流表操作

###### C. 事件处理器完整函数定义

```python
@set_ev_cls(ofp_event.EventOFPSwitchFeatures,CONFIG_DISPATCHER)
def switch_features_handler(self,ev):
    """
    ev: ryu.controller.ofp_event.EventOFPSwitchFeatures object. ev是OpenFlow交换机特征类实例(即事件event 实例)
    """
    pass
```

###### D. 事件event 实例属性及成员函数解读

-   `ev.msg`ev中包含消息类解读

    -   **msg消息打印举例**

    ```python
    EVENT ofp_event->Hub EventOFPPacketIn
    version=0x4,
    msg_type=0xa,
    msg_len=0x80,
    xid=0x0,
    OFPPacketIn(
      buffer_id=4294967295,
      cookie=0,
      data=b'33\xff\x9a\xd7\xb1\x1aj\xff\x9a\xd7\xb1\x86\xdd`\x00\x00\x00\x00 :\xff\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xff\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\xff\x9a\xd7\xb1\x87\x00~f\x00\x00\x00\x00\xfe\x80\x00\x00\x00\x00\x00\x00\x18j\xff\xff\xfe\x9a\xd7\xb1\x0e\x01\x1c\xddV\xff\xb4\xd8',
      match=OFPMatch(oxm_fields={'in_port': 2}),
      reason=0,
      table_id=0,
      total_len=86)
    ```

    -   **msg消息可访问属性或方法列表**:

    ```python
    'auxiliary_id', 'buf', 'capabilities', 'cls_from_jsondict_key', 'cls_msg_type', 
      
    'datapath', 'datapath_id', 
      
    'from_jsondict', 'msg_len', 'msg_type', 'n_buffers', 'n_tables', 
      
    'obj_from_jsondict', 'parser', 'serialize', 
      
    'set_buf', 'set_classes', 'set_headers', 'set_xid', 
      
    'stringify_attrs', 'to_jsondict', 'version', 'xid']
    ```

-   event实例属性

```
 ============ ==============================================================
    Attribute    Description
 ============ ==============================================================
    msg          OpenFlow消息对象
    msg.datapath ryu.controller.controller.Datapath 实例,表明该消息是从哪个交换机收到的
    timestamp    Datapath 生成此事件的时间戳
 ============ ==============================================================
```

###### E. 
`match = ofp_parser.OFPMatch()`
中OFPMatch类协议支持的匹配方式

```python
================ =============== ==================================
    Argument         Value           Description
    ================ =============== ==================================
    in_port          Integer 32bit   Switch input port
    in_phy_port      Integer 32bit   Switch physical input port
    metadata         Integer 64bit   Metadata passed between tables
    eth_dst          MAC address     Ethernet destination address
    eth_src          MAC address     Ethernet source address
    eth_type         Integer 16bit   Ethernet frame type
    vlan_vid         Integer 16bit   VLAN id
    vlan_pcp         Integer 8bit    VLAN priority
    ip_dscp          Integer 8bit    IP DSCP (6 bits in ToS field)
    ip_ecn           Integer 8bit    IP ECN (2 bits in ToS field)
    ip_proto         Integer 8bit    IP protocol
    ipv4_src         IPv4 address    IPv4 source address
    ipv4_dst         IPv4 address    IPv4 destination address
    tcp_src          Integer 16bit   TCP source port
    tcp_dst          Integer 16bit   TCP destination port
    udp_src          Integer 16bit   UDP source port
    udp_dst          Integer 16bit   UDP destination port
    sctp_src         Integer 16bit   SCTP source port
    sctp_dst         Integer 16bit   SCTP destination port
    icmpv4_type      Integer 8bit    ICMP type
    icmpv4_code      Integer 8bit    ICMP code
    arp_op           Integer 16bit   ARP opcode
    arp_spa          IPv4 address    ARP source IPv4 address
    arp_tpa          IPv4 address    ARP target IPv4 address
    arp_sha          MAC address     ARP source hardware address
    arp_tha          MAC address     ARP target hardware address
    ipv6_src         IPv6 address    IPv6 source address
    ipv6_dst         IPv6 address    IPv6 destination address
    ipv6_flabel      Integer 32bit   IPv6 Flow Label
    icmpv6_type      Integer 8bit    ICMPv6 type
    icmpv6_code      Integer 8bit    ICMPv6 code
    ipv6_nd_target   IPv6 address    Target address for ND
    ipv6_nd_sll      MAC address     Source link-layer for ND
    ipv6_nd_tll      MAC address     Target link-layer for ND
    mpls_label       Integer 32bit   MPLS label
    mpls_tc          Integer 8bit    MPLS TC
    mpls_bos         Integer 8bit    MPLS BoS bit
    pbb_isid         Integer 24bit   PBB I-SID
    tunnel_id        Integer 64bit   Logical Port Metadata
    ipv6_exthdr      Integer 16bit   IPv6 Extension Header pseudo-field
    pbb_uca          Integer 8bit    PBB UCA header field
                                     (EXT-256 Old version of ONF Extension)
    tcp_flags        Integer 16bit   TCP flags
                                     (EXT-109 ONF Extension)
    actset_output    Integer 32bit   Output port from action set metadata
                                     (EXT-233 ONF Extension)
    ================ =============== ==================================
```

**举例**

-   常规匹配

```python
match = parser.OFPMatch(
        in_port=1,
        eth_type=0x86dd,
        ipv6_src=('2001:db8:bd05:1d2:288a:1fc0:1:10ee',
        	       'ffff:ffff:ffff:ffff::'),
        ipv6_dst='2001:db8:bd05:1d2:288a:1fc0:1:10ee')
if 'ipv6_src' in match:
   print match['ipv6_src']
   # ('2001:db8:bd05:1d2:288a:1fc0:1:10ee', 'ffff:ffff:ffff:ffff::')
```

-   VLAN ID 匹配

```python
1) Packets with and without a VLAN tag

- Example::

match = parser.OFPMatch()

- Packet Matching

====================== =====
non-VLAN-tagged        MATCH
VLAN-tagged(vlan_id=3) MATCH
VLAN-tagged(vlan_id=5) MATCH
====================== =====

2) Only packets without a VLAN tag

- Example::

match = parser.OFPMatch(vlan_vid=0x0000)

- Packet Matching

====================== =====
non-VLAN-tagged        MATCH
VLAN-tagged(vlan_id=3)   x
VLAN-tagged(vlan_id=5)   x
====================== =====

3) Only packets with a VLAN tag regardless of its value

- Example::

match = parser.OFPMatch(vlan_vid=(0x1000, 0x1000))

- Packet Matching

====================== =====
non-VLAN-tagged          x
VLAN-tagged(vlan_id=3) MATCH
VLAN-tagged(vlan_id=5) MATCH
====================== =====

4) Only packets with VLAN tag and VID equal

- Example::

match = parser.OFPMatch(vlan_vid=(0x1000 | 3))

- Packet Matching

====================== =====
non-VLAN-tagged          x
VLAN-tagged(vlan_id=3) MATCH
VLAN-tagged(vlan_id=5)   x
====================== =====

```

###### F.  
`add_flow`
函数解读:
`ofp_parser.OFPFlowMod`
消息应用

**`ofp_parser.OFPFlowMod`类参数:**

```python
 Modify Flow entry message

    The controller sends this message to modify the flow table.

    ================ ======================================================
    Attribute        Description
    ================ ======================================================
    cookie           Opaque controller-issued identifier
    cookie_mask      Mask used to restrict the cookie bits that must match
                     when the command is ``OPFFC_MODIFY*`` or
                     ``OFPFC_DELETE*``
    table_id         ID of the table to put the flow in
    command          One of the following values.

                     | OFPFC_ADD
                     | OFPFC_MODIFY
                     | OFPFC_MODIFY_STRICT
                     | OFPFC_DELETE
                     | OFPFC_DELETE_STRICT
    idle_timeout     Idle time before discarding (seconds)
    hard_timeout     Max time before discarding (seconds)
    priority         Priority level of flow entry
    buffer_id        Buffered packet to apply to (or OFP_NO_BUFFER)
    out_port         For ``OFPFC_DELETE*`` commands, require matching
                     entries to include this as an output port
    out_group        For ``OFPFC_DELETE*`` commands, require matching
                     entries to include this as an output group
    flags            Bitmap of the following flags.

                     | OFPFF_SEND_FLOW_REM
                     | OFPFF_CHECK_OVERLAP
                     | OFPFF_RESET_COUNTS
                     | OFPFF_NO_PKT_COUNTS
                     | OFPFF_NO_BYT_COUNTS
    match            Instance of ``OFPMatch``
    instructions     list of ``OFPInstruction*`` instance
    ================ ======================================================
```

**`add_flow`函数:**

```python
def add_flow(self,datapath,priority,match,actions,remind_content):
        '''构建流表项 : add a flow entry, install it into datapath
        datapath:表示给哪一个逻辑设备下发流表
        priority：表示优先级
        match,actions:匹配域和动作
        '''

        #datapath属性        
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        #在OpenFlow1.3版本中定义了instruct指令集（交换机内部的一些操作）
        #construct a flow msg and send it
        inst = [ofp_parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,
                                             actions)]

        mod = ofp_parser.OFPFlowMod(datapath=datapath,priority=priority,
                                    match=match,instructions=inst);
        print("install to datapath,"+remind_content)
        #发送出去
        datapath.send_msg(mod);
```

-   `ofp_parser.OFPFlowMod`消息用于控制器管理(写入,应用,删除)交换机中的流表项,其中`instructions`参数用于指示对新增流表项的操作

```python
    ================ ======================================================
    Attribute        Description
    ================ ======================================================
    type             One of following values.

                     | OFPIT_WRITE_ACTIONS
                     | OFPIT_APPLY_ACTIONS
                     | OFPIT_CLEAR_ACTIONS
    actions          list of OpenFlow action class
    ================ ======================================================
```

-   `inst = [ofp_parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,actions)]`表示应用动作`actions`

-   `datapath.send_msg(mod)`**控制器发送此消息以修改流表**

**标准函数`send_flow_mod`发送流表更改消息**

```python
def send_flow_mod(self, datapath):
            ofp = datapath.ofproto
            ofp_parser = datapath.ofproto_parser

            cookie = cookie_mask = 0
            table_id = 0
            idle_timeout = hard_timeout = 0
            priority = 32768
            buffer_id = ofp.OFP_NO_BUFFER
            match = ofp_parser.OFPMatch(in_port=1, eth_dst='ff:ff:ff:ff:ff:ff')
            actions = [ofp_parser.OFPActionOutput(ofp.OFPP_NORMAL, 0)]
            inst = [ofp_parser.OFPInstructionActions(ofp.OFPIT_APPLY_ACTIONS,
                                                     actions)]
            req = ofp_parser.OFPFlowMod(datapath, cookie, cookie_mask,
                                        table_id, ofp.OFPFC_ADD,
                                        idle_timeout, hard_timeout,
                                        priority, buffer_id,
                                        ofp.OFPP_ANY, ofp.OFPG_ANY,
                                        ofp.OFPFF_SEND_FLOW_REM,
                                        match, inst)
            datapath.send_msg(req)
```

###### G.  
`send_packer_out`
函数解读:
`ofp_parser.OFPPacketOut`
消息应用

```python
def send_packet_out(self, datapath, buffer_id, in_port):
            ofp = datapath.ofproto
            ofp_parser = datapath.ofproto_parser

            actions = [ofp_parser.OFPActionOutput(ofp.OFPP_FLOOD, 0)]
            req = ofp_parser.OFPPacketOut(datapath, buffer_id,
                                          in_port, actions)
            datapath.send_msg(req)
```

###### H.  
`ofp_parser.OFPActionOutput`
函数解读

-   函数定义:

    ```python
    @OFPAction.register_action_type(ofproto.OFPAT_OUTPUT,
                                    ofproto.OFP_ACTION_OUTPUT_SIZE)
    class OFPActionOutput(OFPAction):
        """
        Output action

        This action indicates output a packet to the switch port.

        ================ ======================================================
        Attribute        Description
        ================ ======================================================
        port             Output port   
        max_len          Max length to send to controller
        ================ ======================================================
        """
      pass
    ```

-   在`switch_features_handler`函数中`actions = [ofp_parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,ofproto.OFPCML_NO_BUFFER)]`操作表示让交换机将数据包发送给控制器,本地不保留缓存

-   `ofproto.OFPAT_OUTPUT`类型列表:

<table>
<tr class="header">
<th style="text-align: center;">输出端口或模式类型定义</th>
<th style="text-align: center;">解释</th>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>OFPP_MAX = 0xffffff00</code></td>
<td style="text-align: center;"></td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>OFPP_IN_PORT = 0xfffffff8</code></td>
<td style="text-align: center;">将数据包发送到输入端口 This virtual port must be explicitly used in order to send back out of the input port.</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>OFPP_TABLE = 0xfffffff9</code></td>
<td style="text-align: center;">按照流表中的操作执行端口转发,This can only be the destination port for packet-out messages.</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>OFPP_NORMAL = 0xfffffffa</code></td>
<td style="text-align: center;">按照正常的L2/L3层交换处理</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>OFPP_FLOOD = 0xfffffffb</code></td>
<td style="text-align: center;">向除了输入端口外的所有物理端口转发,当使能STP协议时此功能被禁用</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>OFPP_ALL = 0xfffffffc</code></td>
<td style="text-align: center;">向除了输入端口外的所有物理端口转发,当使能STP协议时此功能仍有用</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>OFPP_CONTROLLER = 0xfffffffd</code></td>
<td style="text-align: center;">向控制器端口转发数据</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>OFPP_LOCAL = 0xfffffffe</code></td>
<td style="text-align: center;">本地openflow “port”</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>OFPP_ANY = 0xffffffff</code></td>
<td style="text-align: center;">Not associated with a physical port. 与物理端口无关的</td>
</tr>
</table>

-   `ofproto.OFP_ACTION_OUTPUT_SIZE`:

```python
# enum ofp_controller_max_len
OFPCML_MAX = 0xffe5         # maximum max_len value which can be used to　　　　可用于请求特定字节长度的最大Max值。（是向控制器请求的最大数据长度）
                            # request a specific byte length.
OFPCML_NO_BUFFER = 0xffff   # indicates that no buffering should be　　　　指示不应应用缓冲，并且将整个数据包发送到控制器
                            # applied and the whole packet is to be
                            # sent to the controller.
```

### 3.4 RYU应用进阶教程

#### 3.4.1RYU Switche.py源码分析-数据平面设备信息

##### (1) 拓扑成员类

<table>
<tr class="header">
<th>类名称</th>
<th>类作用</th>
<th>成员变量</th>
<th>备注</th>
</tr>
<tr class="odd">
<td><code>Port</code></td>
<td>存储端口有关信息</td>
<td><code>self.dpid</code>: Port所属交换机的<code>datapath id</code> <br/> <code>self.port_no</code>: Port 端口号 <code>No.</code>表示序号的意思,源自拉丁单词<code>numero</code>后演变为英语<code>number</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>PortState</code></td>
<td><code>port_no-&gt; OFPPort class</code>字典</td>
<td>字典类型<code>dict: int port_no -&gt; OFPPort port</code> 存储了从port_no（int型）到port（OFPPort类实例）的映射,主要用作self.port_state字典的值（键是dpid）</td>
<td></td>
</tr>
<tr class="odd">
<td><code>PortData</code></td>
<td>保存每个端口和与对应的LLDP报文数据</td>
<td><code>self.lldp_data = lldp_data</code>: LLDP报文的数据<br/><code>self.timestamp = None</code>: 每调用一次lldp_sent函数，便会把self.timestamp置为当前的时间（time.time()）<br/><code>self.sent = 0</code>: 每调用一次<code>lldp_sent</code>函数将<code>self.sent</code>加1;每调用一次<code>lldp_received</code>函数，便会把<code>self.sent</code>置为0</td>
<td></td>
</tr>
<tr class="even">
<td><code>PortDataState</code></td>
<td><code>Port class-&gt;PortData class</code>字典</td>
<td>继承自dict类，保存从Port类到PortData类的映射</td>
<td></td>
</tr>
<tr class="odd">
<td><code>Switch</code></td>
<td>交换机类</td>
<td><code>self.dp: Datapath</code>: dp是<code>ryu/controller/controller.py</code>中定义的Datapath类实例,<code>Datapath</code>类的属性包括<code>socket address is_active id ports</code> <br/><code>self.ports</code>:由Port类实例组成的列表，存储该交换机的端口</td>
<td></td>
</tr>
<tr class="even">
<td><code>Link</code></td>
<td>链路类</td>
<td><code>self.src</code>: 保存的是源端口（Port类实例） <br/> <code>self.dst</code>: 保存的是目的端口（Port类实例）</td>
<td></td>
</tr>
<tr class="odd">
<td><code>LinkState</code></td>
<td><code>Link class -&gt; timestamp</code>字典</td>
<td>继承自dict，保存从Link类到时间戳的映射。数据成员self._map字典用于存储Link两端互相映射的关系。</td>
<td></td>
</tr>
<tr class="even">
<td><code>LLDPPacket</code></td>
<td>LLDP数据包类</td>
<td>静态方法<code>lldp_packet(dpid,port_no,dl_addr,ttl)</code>用于构造LLDP报文<br/>静态方法<code>lldp_parse(data)</code>用于解析LLDP包，并返回源DPID和源端口号。</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>

##### (2) Switches类分析

> Switches类是app_manager.RyuApp类的子类，用于进行链路发现等工作, 当运行switches应用时会被实例化


-   常量设置:

```python
    LLDP_PACKET_LEN = len(LLDPPacket.lldp_packet(0, 0, DONTCARE_STR, 0))
    LLDP_SEND_GUARD = .05
    LLDP_SEND_PERIOD_PER_PORT = .9
    TIMEOUT_CHECK_PERIOD = 5.
    LINK_TIMEOUT = TIMEOUT_CHECK_PERIOD * 2
    LINK_LLDP_DROP = 5
```

-   成员变量

```python
self.name = 'switches'
self.dps = {}                 #self.dps字典用于保存dpid到Datapath类实例的映射，会在_register函数中添加新成员，_unregister函数中删除成员。遍历该字典可以得到连接的所有交换机。
self.port_state = {}          #字典中键为dpid，值为PortState类型。遍历该字典可以得到所有交换机对应的端口情况。当交换机连接时，会检查交换机的id是否在self.port_state中，不在则创建PortState类实例，把交换机的所有端口号和端口存储到该实例中；交换机断开时，会从self.port_state中删除。
self.ports = PortDataState()  #self.ports是PortDataState类的实例，保存每个端口（Port类型）对应的LLDP报文数据（保存在PortData类实例中），遍历self.ports用于发送LLDP报文。
self.links = LinkState()      #self.links是LinkState类的实例，保存所有连接（Link类型）到时间戳的映射。遍历self.links的键即可得到所有交换机之间的连接情况。
self.hosts = HostState()      # mac address -> Host class list
self.is_active = True
```

-   `__init__`函数关键判断

```python
self.link_discovery = self.CONF.observe_links
#如果ryu-manager启动时加了--observe-links参数，则下面的self.link_discovery将为真，从而执行if下面的语句：
if self.link_discovery: 
   self.install_flow = self.CONF.install_lldp_flow
   self.explicit_drop = self.CONF.explicit_drop
   self.lldp_event = hub.Event()
   self.link_event = hub.Event()
   # 添加两个协程
   self.threads.append(hub.spawn(self.lldp_loop))
   self.threads.append(hub.spawn(self.link_loop))
```

-   `lldp_loop`成员函数使能交换机端口定时发送LLDP报文

```python
def lldp_loop(self):
    while self.is_active:
        self.lldp_event.clear()

        now = time.time() #当前时刻
        timeout = None
        ports_now = []
        ports = []
        #遍历self.ports(PortDataState类的实例)，获得key（Port类实例）和data（PortData类实例）
        for (key, data) in self.ports.items(): 
            if data.timestamp is None: #如果data.timestamp为None（该端口还没发送过LLDP报文）
                ports_now.append(key) #则将key（端口）加入ports_now列表；
                continue #------------
                #否则，计算下次应该发送LLDP报文的时间expire
                expire = data.timestamp + self.LLDP_SEND_PERIOD_PER_PORT
                if expire <= now: #如果已经超时，则放到ports列表-----表示本来就应该发送下一次LLDP了
                    ports.append(key)
                    continue #------------
    				#获取timeout时间,表示还差多久到达期望的时间expire，进行LLDP数据包发送
                    timeout = expire - now 
                     #否则就是还没到发送时间，停止遍历
                    break
                    #（发送LLDP报文时是按序发的，找到第一个未超时的端口，后面的端口肯定更没有超时，
                    #因为后面端口上次发送LLDP是在前一端口之后，前一个都没超时后面的自然也没超时）
                    #注意：列表中都是按序的，先发ports_now中还没有发送过LLDP数据的Port实例（data.timestamp为None),
                    #然后查看已经发送过,但是已经超时（及本来应该进行下一次发送的）ports中的Port实例for port in ports_now: 
                    #遍历ports_now列表，对每个端口调用self.send_lldp_packet(port)，发送LLDP报文
                    self.send_lldp_packet(port) #--------发送LLDP报文
                    for port in ports: #遍历ports列表，对每个端口调用self.send_lldp_packet(port)，发送LLDP报文
                        self.send_lldp_packet(port) #--------发送LLDP报文
                        # don't burst 设置睡眠时间，防止回送数据太多，导致出现等待时延
                        hub.sleep(self.LLDP_SEND_GUARD)      
    					#timeout!=None表示：还有端口Port实例没有到达下一次发送LLDP报文的时机 Ports表示:本来应该发生                         LLDP报文的Port实例（超过本来周期）--上面发送过
                        if timeout is not None and ports: 
                            # We have already slept ---- ？？设置为0---主要为了避免None在下面wait中出错
                            timeout = 0     
                            # LOG.debug('lldp sleep %s', timeout)
                            self.lldp_event.wait(timeout=timeout) #事件等待timeout时刻
```

-   `send_lldp_packet`函数发送LLDP报文

    -   此函数由`lldp_loop`函数调用

    ```python
    def send_lldp_packet(self, port):
            try:
                #！！！--------！！！重点：在发送时我们会将当前时刻保存在self.timestamp = time.time()中，后面我们接             收到LLDP数据包时，
                #！！！--------！！！重点：到对应端口中获取timestamp,然后现在时刻-timestamp获得LLDP测量的C-S-S-C时             间！！！
      		#调用PortDataState类的lldp_sent函数，该函数会设置时间戳，
                port_data = self.ports.lldp_sent(port) 
                #移动相应端口在双向循环链表中的位置，最后返回PortData类实例port_data。
            except KeyError:
                # ports can be modified during our sleep in self.lldp_loop()
                # LOG.debug('send_lld error', exc_info=True)
                return
            if port_data.is_down: #如果该端口已经down掉，直接返回，否则执行下一步
                return

            dp = self.dps.get(port.dpid, None) #获取了datapath实例
            if dp is None: #根据port.dpid得到对应的Datapath类实例dp，如果不存在，则直接返回，否则执行下一步
                # datapath was already deleted
                return

            # LOG.debug('lldp sent dpid=%s, port_no=%d', dp.id, port.port_no)
            # TODO:XXX
            #发送LLDP报文。具体地：
            #(1)生成actions：从port.port_no端口发出消息；
            #(2)生成PacketOut消息：datapath指定为上一步得到的dp，actions为前面的，data为步骤a中返回的port_data的		lldp_data
            if dp.ofproto.OFP_VERSION == ofproto_v1_0.OFP_VERSION:
                actions = [dp.ofproto_parser.OFPActionOutput(port.port_no)]
                dp.send_packet_out(actions=actions, data=port_data.lldp_data)
            elif dp.ofproto.OFP_VERSION >= ofproto_v1_2.OFP_VERSION:
                actions = [dp.ofproto_parser.OFPActionOutput(port.port_no)] #设置actions
                out = dp.ofproto_parser.OFPPacketOut( #下发LLDP数据
                    datapath=dp, in_port=dp.ofproto.OFPP_CONTROLLER,
                    buffer_id=dp.ofproto.OFP_NO_BUFFER, actions=actions,
                    data=port_data.lldp_data) #PortData类实例的lldp_data数据
                dp.send_msg(out)
            else:
                LOG.error('cannot send lldp packet. unsupported version. %x',
                          dp.ofproto.OFP_VERSION)
    ```

-   `state_change_handler`基于交换机状态改变事件触发,下发流表函数

    -   ```python
        set_ev_cls(ofp_event.EventOFPStateChange,
                        [MAIN_DISPATCHER, DEAD_DISPATCHER]) #该函数用于处理EventOFPStateChange事件，当交换机连接或者断开时会触发该事件。
            def state_change_handler(self, ev):
                dp = ev.datapath
                assert dp is not None
                LOG.debug(dp)

                if ev.state == MAIN_DISPATCHER: #如果状态是MAIN_DISPATCHER：
                    dp_multiple_conns = False
                    if dp.id in self.dps: #(1)从ev.datapath获得Datapath类实例dp，如果该dp的dpid已经在self.dps里有，则报出重复链接的警告。
                        LOG.warning('Multiple connections from %s', dpid_to_str(dp.id))
                        dp_multiple_conns = True
                        (self.dps[dp.id]).close()

                    #(2)调用_register()，将dp.id和dp添加到self.dps中；
                    #如果该dp.id不在self.port_state中，则创建该dp.id对应的PortState实例，
                    #并遍历dp.ports.values，将所有port（OFPPort类型）添加到该PortState实例中。
                    self._register(dp)

                    #(3)调用_get_switch()，如果dp.id在self.dps中，则创建一个Switch类实例，
                    #并把self.port_state中对应的端口都添加到该实例中，最终返回该实例。
                    switch = self._get_switch(dp.id)
                    LOG.debug('register %s', switch)

                    if not dp_multiple_conns: #(4)如果交换机没有重复连接，触发EventSwitchEnter事件。
                        self.send_event_to_observers(event.EventSwitchEnter(switch))
                    else:
                        evt = event.EventSwitchReconnected(switch)
                        self.send_event_to_observers(evt)

                    if not self.link_discovery: #(5)如果没设置self.link_discovery，返回；否则执行下一步。
                        return

                    #(6)如果设置了self.install_flow，则根据OpenFlow版本生成相应流表项，
                    #使得收到的LLDP报文（根据目的MAC地址匹配）上报给控制器。
                    if self.install_flow:
                        ofproto = dp.ofproto
                        ofproto_parser = dp.ofproto_parser

                        # TODO:XXX need other versions
                        if ofproto.OFP_VERSION == ofproto_v1_0.OFP_VERSION:
                            rule = nx_match.ClsRule()
                            rule.set_dl_dst(addrconv.mac.text_to_bin(
                                            lldp.LLDP_MAC_NEAREST_BRIDGE))
                            rule.set_dl_type(ETH_TYPE_LLDP)
                            actions = [ofproto_parser.OFPActionOutput(
                                ofproto.OFPP_CONTROLLER, self.LLDP_PACKET_LEN)]
                            dp.send_flow_mod(
                                rule=rule, cookie=0, command=ofproto.OFPFC_ADD,
                                idle_timeout=0, hard_timeout=0, actions=actions,
                                priority=0xFFFF)
                        elif ofproto.OFP_VERSION >= ofproto_v1_2.OFP_VERSION:
                            match = ofproto_parser.OFPMatch( 
                                eth_type=ETH_TYPE_LLDP,
                                #重点：设置match---交换机获取得到以太网LLDP数据包（由另一个交换机转发而来）！！！！！
                                eth_dst=lldp.LLDP_MAC_NEAREST_BRIDGE) 
                            # OFPCML_NO_BUFFER is set so that the LLDP is not
                            # buffered on switch
                            parser = ofproto_parser
                            #设置actions,当第二个交换机获取到LLDP数据后，不需要缓存LLDP数据，全部上交给控制器
                            actions = [parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,
                                                              ofproto.OFPCML_NO_BUFFER
                                                              )] 
                            inst = [parser.OFPInstructionActions(
                                    ofproto.OFPIT_APPLY_ACTIONS, actions)]
                            mod = parser.OFPFlowMod(datapath=dp, match=match,
                                                    idle_timeout=0, hard_timeout=0,
                                                    instructions=inst,
                                                    priority=0xFFFF) # 标识流表的优先级，范围为0-65535，值越大，优先级越高
                            dp.send_msg(mod) #发送给datapath
                        else:
                            LOG.error('cannot install flow. unsupported version. %x',
                                      dp.ofproto.OFP_VERSION)

                    # Do not add ports while dp has multiple connections to controller.
                    if not dp_multiple_conns:
                        for port in switch.ports:
                            if not port.is_reserved():
                                self._port_added(port)

                    self.lldp_event.set()

                elif ev.state == DEAD_DISPATCHER:
                    # dp.id is None when datapath dies before handshake
                    if dp.id is None:
                        return

                    switch = self._get_switch(dp.id)
                    if switch:
                        if switch.dp is dp:
                            self._unregister(dp)
                            LOG.debug('unregister %s', switch)
                            evt = event.EventSwitchLeave(switch)
                            self.send_event_to_observers(evt)

                            if not self.link_discovery:
                                return

                            for port in switch.ports:
                                if not port.is_reserved():
                                    self.ports.del_port(port)
                                    self._link_down(port)
                            self.lldp_event.set()
        ```

-   `lldp_packet_in_handler` 基于`EventOFPacketIn`事件触发,LLDP数据接收处理函数

    -   ```python
         @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
            def lldp_packet_in_handler(self, ev):
                if not self.link_discovery:
                    return

                msg = ev.msg
                try:
                    src_dpid, src_port_no = LLDPPacket.lldp_parse(msg.data) #解析lldp数据包，获取src_dpid和src_port_no
                except LLDPPacket.LLDPUnknownFormat:
                    # This handler can receive all the packets which can be
                    # not-LLDP packet. Ignore it silently
                    return

                dst_dpid = msg.datapath.id #获取目的datapath的dst_dpid
                if msg.datapath.ofproto.OFP_VERSION == ofproto_v1_0.OFP_VERSION:
                    dst_port_no = msg.in_port #获取目的datapath的dst_port_no
                elif msg.datapath.ofproto.OFP_VERSION >= ofproto_v1_2.OFP_VERSION:
                    dst_port_no = msg.match['in_port']
                else:
                    LOG.error('cannot accept LLDP. unsupported version. %x',
                              msg.datapath.ofproto.OFP_VERSION)

                src = self._get_port(src_dpid, src_port_no)
                if not src or src.dpid == dst_dpid:
                    return
                try:
                    self.ports.lldp_received(src)
                except KeyError:
                    # There are races between EventOFPPacketIn and
                    # EventDPPortAdd. So packet-in event can happend before
                    # port add event. In that case key error can happend.
                    # LOG.debug('lldp_received error', exc_info=True)
                    pass

                dst = self._get_port(dst_dpid, dst_port_no)
                if not dst:
                    return

                old_peer = self.links.get_peer(src)
                # LOG.debug("Packet-In")
                # LOG.debug("  src=%s", src)
                # LOG.debug("  dst=%s", dst)
                # LOG.debug("  old_peer=%s", old_peer)
                if old_peer and old_peer != dst:
                    old_link = Link(src, old_peer)
                    del self.links[old_link]
                    self.send_event_to_observers(event.EventLinkDelete(old_link))

                link = Link(src, dst)
                if link not in self.links:
                    self.send_event_to_observers(event.EventLinkAdd(link))

                    # remove hosts if it's not attached to edge port
                    host_to_del = []
                    for host in self.hosts.values():
                        if not self._is_edge_port(host.port):
                            host_to_del.append(host.mac)

                    for host_mac in host_to_del:
                        del self.hosts[host_mac]

                if not self.links.update_link(src, dst):
                    # reverse link is not detected yet.
                    # So schedule the check early because it's very likely it's up
                    self.ports.move_front(dst)
                    self.lldp_event.set()
                if self.explicit_drop:
                    self._drop_packet(msg)
        ```

#### 3.4.2 RYU OpenFlow消息字段解读

> 查询消息字段解释和数据类型的路径: `ofproto_v1_2_parser.py` `ofproto_v1_3_parser.py`
> 
> 查询某种协议数据包的字段类型:`from ryu.lib.packet import ethernet`
> 
> 待添加


#### 3.4.2 交换机自学习应用

> 当涉及到交换机自学习功能时,就需要我们定义新的数据结构,比如mac_to_port表,分析协议处理流程,并解析新的数据包格式,例如ethernet以太网数据帧,并从中获取目的MAC地址,原MAC地址等信息


##### (1) 基本思路设计思路

SDN中交换机不存储MAC表，（datapath)只存在流表。其地址学习操作由控制器(控制器中包含MAC 地址表)实现，之后控制器下发流表项给交换机

-   主机A向主机B发送信息，流表中只存在默认流表，告诉交换机将数据包发送给控制器。

    ![image-20240808160344605](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/8163bc71a1586481-image-20240808160344605.png)

-   控制器先进行MAC地址学习，记录主机A的MAC地址和其对应交换机端口，然后查询MAC地址表，查找主机Ｂ信息。没有则下发流表项告诉交换机先泛洪试试

    ![image-20240808160356028](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/a3a78ac04d16c624-image-20240808160356028.png)

-   泛洪后，主机Ｃ接收后丢弃数据包，不处理。主机Ｂ发现是寻找自己的，则进行消息回送，由于交换机流表中没有处理主机Ｂ到主机Ａ的信息的流表项，所以只能向控制器发送数据包。控制器先学习主机Ｂ的ＭＡＣ地址和对应交换机端口，之后查询ＭＡＣ地址表，找到主机A的MAC信息，下发流表项，告诉交换机如何处理主机B->主机A的消息

    ![image-20240808160405174](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/d68a461987a23ccc-image-20240808160405174.png)

注意：这里交换机的流表项中只存在主机B->主机A的流表项处理方案，不存在主机Ａ－>主机Ｂ的处理流表项（但是控制器ＭＡＣ地址表中是存在主机Ｂ的信息），所以会在下一次数据传送中，控制器下发响应的流表项。但是其实可以实现（在３中一次下发两个流表项）

##### (2) 代码实现与解读

```python
from ryu.base import app_manager
from ryu.ofproto import ofproto_v1_3
from ryu.controller import ofp_event
from ryu.controller.handler import set_ev_cls
from ryu.controller.handler import CONFIG_DISPATCHER,MAIN_DISPATCHER
from ryu.lib.packet import packet
from ryu.lib.packet import ethernet

class SelfLearnSwitch(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]    #set openflow protocol version while we support

    def __init__(self,*args,**kwargs):
        super(SelfLearnSwitch,self).__init__(*args,**kwargs)
        #set a data construction to save MAC Address Table
        self.Mac_Port_Table={} # {dpid:{mac:port}}

    @set_ev_cls(ofp_event.EventOFPSwitchFeatures)
    def switch_features_handler(self,ev):
        # 与3.3.3相同不赘述
        pass

    def add_flow(self,datapath,priority,match,actions,extra_info):
        """
        add flow entry to switch
        """
        # 与3.3.3相同不赘述
        pass
    # 关键代码
    @set_ev_cls(ofp_event.EventOFPPacketIn,MAIN_DISPATCHER)
    def packet_in_handler(self,ev):
        '''
        manage infomation from switch
        '''

        #first parser openflow protocol　　　　先解析OpenFlow协议信息
        msg = ev.msg
        datapath = msg.datapath
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        #get datapath id from datapath, and save dpid into MAC table (default) 　　获取datapath(虚拟交换机的id),用dpid初始化一个键值
        dpid = datapath.id
        self.Mac_Port_Table.setdefault(dpid, {})

        #analysize packet, get ethernet data, get host MAC info　　分析packert数据包，因为转发的包，都是基于以太网协议的，所以我们需要用到以太网协议进行解析，获取源MAC和目的MAC
        pkt = packet.Packet(msg.data)
        eth_pkt = pkt.get_protocol(ethernet.ethernet)
        dst = eth_pkt.dst
        src = eth_pkt.src

        #get switch port where host packet send in　　获取datapath的数据输入端口
        in_port = msg.match['in_port']

        self.logger.info("Controller %s get packet, Mac address from: %s send to: %s , send from datapath: %s,in port is: %s"
                            ,dpid,src,dst,dpid,in_port)　　#打印调试信息

        #save src data into dictionary---MAC address table　　将源MAC地址保存，学习，放入MAC表中
        self.Mac_Port_Table[dpid][src] = in_port

        #query MAC address table to get destinction host`s port from current datapath　　查询MAC表，是否有目标MAC地址的键值
        #---first: find port to send packet　　如果找到，我们则按照该端口发送
        #---second: not find port,so send packet by flood　　如果没有找到，我们需要泛洪发送给下一个（或者下几个）交换机，依次查询
if dst in self.Mac_Port_Table[dpid]:
            Out_Port = self.Mac_Port_Table[dpid][dst]
        else:
            Out_Port = ofproto.OFPP_FLOOD

        #set match-action from above status　　开始设置match-actions匹配动作
        actions = [ofp_parser.OFPActionOutput(Out_Port)]

        #add a new flow entry to switch by add_flow　　进行对应的流表项下发　　《重点》
        if Out_Port != ofproto.OFPP_FLOOD:   
            match = ofp_parser.OFPMatch(in_port=in_port,eth_dst = dst)
            self.add_flow(datapath, 1, match, actions,"a new flow entry by specify port")
            self.logger.info("send packet to switch port: %s",Out_Port)

        #finally send the packet to datapath, to achive self_learn_switch　　最后我们将之前交换机发送上来的数据，重新发给交换机
        Out = ofp_parser.OFPPacketOut(datapath=datapath,buffer_id=msg.buffer_id,
                                in_port=in_port,actions=actions,data=msg.data)　　#我们必须加上这个data,才可以将packet数据包发送回去《重点》不然会出错××××××

        datapath.send_msg(Out)
```

##### (3) 补充内容

-   `pkt = packet.Packet(msg.data)`一个类，在Ryu/lib/packet/模块下，用于包的解码/编码

```python
class Packet(StringifyMixin):
    """A packet decoder/encoder class.

    An instance is used to either decode or encode a single packet.

    *data* is a bytearray to describe a raw datagram to decode.　　data是一个未加工的报文数据， 即msg.data直接从事件的msg中获取的数据
    When decoding, a Packet object is iteratable.
    Iterated values are protocol (ethernet, ipv4, ...) headers and the payload.
    Protocol headers are instances of subclass of packet_base.PacketBase.
    The payload is a bytearray.  They are iterated in on-wire order.

    *data* should be omitted when encoding a packet.
    """

    # Ignore data field when outputting json representation.
    _base_attributes = ['data']

    def __init__(self, data=None, protocols=None, parse_cls=ethernet.ethernet):　　协议解析，默认是按照以太网协议
        super(Packet, self).__init__()　　
        self.data = data
        if protocols is None:
            self.protocols = []
        else:
            self.protocols = protocols
        if self.data:
            self._parser(parse_cls)
```

-   `eth_pkt = pkt.get_protocol(ethernet.ethernet)`　　返回与指定协议匹配的协议列表。从packet包中获取协议信息

```python
class Packet(StringifyMixin):

    def add_protocol(self, proto):
        """Register a protocol *proto* for this packet.

        This method is legal only when encoding a packet.

        When encoding a packet, register a protocol (ethernet, ipv4, ...)
        header to add to this packet.
        Protocol headers should be registered in on-wire order before calling
        self.serialize.
        """

        self.protocols.append(proto)

    def get_protocols(self, protocol):
        """Returns a list of protocols that matches to the specified protocol.
        """
        if isinstance(protocol, packet_base.PacketBase):
            protocol = protocol.__class__
        assert issubclass(protocol, packet_base.PacketBase)
        return [p for p in self.protocols if isinstance(p, protocol)]
```

-   `eth_pkt = pkt.get_protocol(ethernet.ethernet)`　　一个类，也在Ryu/lib/packet/模块下，用于以太网报头编码器/解码器类

```python
class ethernet(packet_base.PacketBase):
    """Ethernet header encoder/decoder class.

    An instance has the following attributes at least.
    MAC addresses are represented as a string like '08:60:6e:7f:74:e7'.
    __init__ takes the corresponding args in this order.

    ============== ==================== =====================
    Attribute      Description          Example
    ============== ==================== =====================
    dst            destination address  'ff:ff:ff:ff:ff:ff'
    src            source address       '08:60:6e:7f:74:e7'
    ethertype      ether type           0x0800
    ============== ==================== =====================
    """
    pass
```

#### 3.4.3 流量监控应用

![image-20240809194343714](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/70cb4247b147449b-image-20240809194343714.png)

##### (1) 基本设计思路

-   控制器向交换机周期下发获取统计消息，请求交换机信息
    -   1.端口流量统计信息
    -   2.请求流表项统计信息

-   根据交换机统计信息计算流量信息
    -   流速公式(t1时刻的流量-t0时刻的流量)/(t1-t0):    <img src="https://www.zhihu.com/equation?tex=speed%3D%5Cfrac%7Bs%28t1%29-s%28t0%29%7D%7Bt1-t0%7D" alt="speed=\frac{s(t1)-s(t0)}{t1-t0}" class="ee_img tr_noresize" eeimg="1">
    -   剩余带宽公式(链路总带宽-流速):     <img src="https://www.zhihu.com/equation?tex=free%20bw%3Dcapability%20-speed" alt="free bw=capability -speed" class="ee_img tr_noresize" eeimg="1">
    -   路径有效带宽：这一整条路径中，按照最小的剩余带宽处理

##### (2) 代码实现与解读

```python
from operator import attrgetter

from ryu.app import simple_switch_13
from ryu.controller.handler import set_ev_cls
from ryu.controller import ofp_event
from ryu.controller.handler import MAIN_DISPATCHER,DEAD_DISPATCHER
from ryu.lib import hub

#simple_switch_13 is same as the last experiment which named self_learn_switch
class MyMonitor(simple_switch_13.SimpleSwitch13):    
    '''
    design a class to achvie managing the quantity of flow
    流量管理类
    '''

    def __init__(self,*args,**kwargs):
        super(MyMonitor,self).__init__(*args,**kwargs)
        # dpid---> Datapath class
        self.datapaths = {}
        #use gevent to start monitor
        # 使用协程使能监控功能
        self.monitor_thread = hub.spawn(self._monitor)

    @set_ev_cls(ofp_event.EventOFPStateChange,[MAIN_DISPATCHER,DEAD_DISPATCHER])
    def _state_change_handler(self,ev):
        '''
        design a handler to get switch state transition condition
        当交换机状态改变时响应处理函数: 在self.datapaths = {}注册该交换机或者移除交换机
        '''
        #first get ofprocotol info
        datapath = ev.datapath
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        #judge datapath`s status to decide how to operate
        if datapath.state == MAIN_DISPATCHER:    #should save info to dictation 
            if datapath.id not in self.datapaths:
                self.datapaths[datapath.id] = datapath
                self.logger.debug("Regist datapath: %16x",datapath.id)
        elif datapath.state == DEAD_DISPATCHER:    #should remove info from dictation
            if datapath.id in self.datapaths:
                del self.datapaths[datapath.id]
                self.logger.debug("Unregist datapath: %16x",datapath.id)


    def _monitor(self):
        '''
        design a monitor on timing system to request switch infomations about port and flow
        设计一个监控函数定期请求交换机端口和流信息
        '''
        while True:    #initiatie to request port and flow info all the time
            for dp in self.datapaths.values():
                self._request_stats(dp)
            hub.sleep(5)    #pause to sleep to wait reply, and gave time to other gevent to request

    def _request_stats(self,datapath):
        '''
        the function is to send requery to datapath
        发送信息查询请求到交换机datapath
        '''
        self.logger.debug("send stats reques to datapath: %16x for port and flow info",datapath.id)

        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser
    	# 生成OFPFlowStatsRequest请求信息
        req = parser.OFPFlowStatsRequest(datapath)
        datapath.send_msg(req)
    	# 生成OFPPortStatsRequest请求信息
        req = parser.OFPPortStatsRequest(datapath, 0, ofproto.OFPP_ANY)
        datapath.send_msg(req)


    @set_ev_cls(ofp_event.EventOFPPortStatsReply,MAIN_DISPATCHER)
    def _port_stats_reply_handler(self,ev):
        '''
        monitor to require the port state, then this function is to get infomation for port`s info
        关于端口状态统计查询请求响应信息处理函数
        '''
        # print("6666666666port info:")
        # print(ev.msg)
        # print(dir(ev.msg))
        body = ev.msg.body
        self.logger.info('datapath             port     '
                        'rx_packets            tx_packets'
                        'rx_bytes            tx_bytes'
                        'rx_errors            tx_errors'
                        )
        self.logger.info('---------------    --------'
                        '--------    --------'
                        '--------    --------'
                        '--------    --------'
                        )
        for port_stat in sorted(body,key=attrgetter('port_no')):
                self.logger.info('%016x %8x %8d %8d %8d %8d %8d %8d',
                    ev.msg.datapath.id,port_stat.port_no,port_stat.rx_packets,port_stat.tx_packets,
                    port_stat.rx_bytes,port_stat.tx_bytes,port_stat.rx_errors,port_stat.tx_errors
                        )


    @set_ev_cls(ofp_event.EventOFPFlowStatsReply,MAIN_DISPATCHER)
    def _flow_stats_reply_handler(self,ev):
        '''
        monitor to require the flow state, then this function is to get infomation for flow`s info
    	关于交换机流状态查询请求的响应消息处理函数
        '''
        # print("777777777flow info:")
        # print(ev.msg)
        # print(dir(ev.msg))
        body = ev.msg.body

        self.logger.info('datapath             '
                        'in_port            eth_src'
                        'out_port            eth_dst'
                        'packet_count        byte_count'
                        )
        self.logger.info('---------------    '
                        '----    -----------------'
                        '----    -----------------'
                        '---------    ---------'
                        )
        for flow_stat in sorted([flow for flow in body if flow.priority==1],
                        key=lambda flow:(flow.match['in_port'],flow.match['eth_src'])):
                self.logger.info('%016x    %8x    %17s    %8x    %17s    %8d    %8d',
                    ev.msg.datapath.id,flow_stat.match['in_port'],flow_stat.match['eth_src'],
                    flow_stat.instructions[0].actions[0].port,flow_stat.match['eth_dst'],
                    flow_stat.packet_count,flow_stat.byte_count
                        )
```

#### 3.4.4 网络拓扑发现

##### (1) 基本设计思路

-   网络拓扑信息
    -   ![image-20240809220137230](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/946df8705198b218-image-20240809220137230.png)
    -   其中1,2,3表示该交换机对应的端口号

-   网络邻接矩阵表示
    -   ![image-20240809220351612](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/7ce48a4539bcd40c-image-20240809220351612.png)
    -   其中左侧列S1,S2,S3,S4表示入节点，上面S1,S2,S3,S4表示出节点。（m,0），m表示入节点的端口，0暂时表示两个节点之间的时延信息

-   主机信息表示
    -   ![image-20240809220527836](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/0952ef310d54763d-image-20240809220527836.png)

##### (2) 代码实现与解读

```python
from ryu.base import app_manager

from ryu.ofproto import ofproto_v1_3

from ryu.controller import ofp_event
from ryu.controller.handler import MAIN_DISPATCHER,CONFIG_DISPATCHER,DEAD_DISPATCHER #只是表示datapath数据路径的状态
from ryu.controller.handler import set_ev_cls

from ryu.lib import hub
from ryu.lib.packet import packet,ethernet

from ryu.topology import event,switches
from ryu.topology.api import get_switch,get_link,get_host

import threading #需要设置线程锁

DELAY_MONITOR_PERIOD = 5
# 重入锁（Reentrant Lock） LOCK.acquire()和LOCK.release() 数量需要对应
LOCK = threading.RLock() #实现线程锁

# =========================================定义拓扑检测类============================================
class TopoDetect(app_manager.RyuApp):
    """
    拓扑检测类
    """
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    def __init__(self,*args,**kwargs):
        super(TopoDetect,self).__init__(*args,**kwargs)
        self.topology_api_app = self  #用于保持对象本身，后面get_switch等方法需要（我们也可以直接传入self）
        self.link_list = None         #保存所有的link信息，由get_link获得
        self.switch_list = None       #保存所有的switch信息，由get_switch获得
        self.host_list = None         #保存所有的host信息，由get_host获得

        self.dpid2id = {}             #获取交换机dpid,以及自定义id--->{dpid:id}
        self.id2dpid = {}             #对应上面的self.dpid2id,翻转即可，因为我们使用id进行建立邻接矩阵，这两个结构方便查找
        self.dpid2switch = {}         #保存dpid和对应的交换机全部信息---->通过矩阵获得id,然后获得dpid,最后获得交换机对象信息

        self.ip2host = {}             #根据ip，保存主机对象信息--->{ip:host}
        self.ip2switch = {}           #根据ip,获取当前主机是连接到哪个交换机--->{ip:dpid}

        self.net_size = 0             #记录交换机个数（网络拓扑大小）
        self.net_topo = []            #用于保存邻接矩阵

        self.net_flag = False         #标识：用于表示拓扑网络拓扑self.net_topo是否已经更新完成
        self.net_arrived = 0          #标识：用于表示网络中交换机消息到达，每当一个交换机到达以后，我们设置+1

        self.monitor_thread = hub.spawn(self._monitor)  #协程实现定时检测网络拓扑
    # =========================================OpenFlow消息处理========================================
    @set_ev_cls(ofp_event.EventOFPSwitchFeatures,CONFIG_DISPATCHER)
    def switch_feature_handle(self,ev):
        """
        datapath中有配置消息到达
        """
        LOCK.acquire()
        self.net_arrived += 1 #表示有1个交换机到达
        LOCK.release()
        # print("------XXXXXXXXXXX------%d------XXXXXXXXXXX------------switch_feature_handle"%  
        # (self.net_arrived))
        msg = ev.msg
        datapath = msg.datapath
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        match = ofp_parser.OFPMatch()

        actions = [ofp_parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,ofproto.OFPCML_NO_BUFFER)]

        self.add_flow(datapath=datapath,priority=0,match=match,actions=actions,extra_info="config\
        infomation arrived!!")


    def add_flow(self,datapath,priority,match,actions,idle_timeout=0,hard_timeout=0,extra_info=None):
        #print("------------------add_flow:")
        if extra_info != None:
            print(extra_info)
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        inst = [ofp_parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,actions)]

        mod = ofp_parser.OFPFlowMod(datapath=datapath,priority=priority,
                                    idle_timeout=idle_timeout,
                                    hard_timeout=hard_timeout,
                                    match=match,instructions=inst)
        datapath.send_msg(mod);

    @set_ev_cls(ofp_event.EventOFPPacketIn,MAIN_DISPATCHER)
    def packet_in_handler(self,ev):
        msg = ev.msg
        datapath = msg.datapath
        ofproto = datapath.ofproto
        ofp_parser = datapath.ofproto_parser

        dpid = datapath.id
        in_port = msg.match['in_port']

        pkt = packet.Packet(msg.data)
        eth_pkt = pkt.get_protocol(ethernet.ethernet)
        dst = eth_pkt.dst
        src = eth_pkt.src

        #self.logger.info("------------------Controller %s get packet, Mac address from: %s send to: %s , send from datapath: %s,in port is: %s", (dpid,src,dst,dpid,in_port))
        # get_topology 会更新主机连接信息, 因此通过EventOFPPacketIn触发后执行该函数可以更新主机连接信息
        self.get_topology(None)
    # =========================================拓扑发现功能实现========================================
    def _monitor(self):
        """
        协程实现伪并发，探测拓扑状态
        """
        while True:
            #print("------------------_monitor")
            self._host_add_handler(None) #主机信息单独处理
            self.get_topology(None)
            if self.net_flag:
                try:
                    self.show_topology()
                except Exception as err:
                    print("Please use cmd: pingall to detect topology and wait a moment")
            hub.sleep(DELAY_MONITOR_PERIOD) #5秒一次

    @set_ev_cls([event.EventHostAdd])
    def _host_add_handler(self,ev):    #主机信息单独处理，不属于网络拓扑
        #需要使用pingall,主机通过与边缘交换机连接，才能告诉控制器
        self.host_list = get_host(self.topology_api_app)  
        #获取主机信息字典ip2host{ipv4:host object}  ip2switch{ipv4:dpid}
        for i,host in enumerate(self.host_list):
            self.ip2switch["%s"%host.ipv4] = host.port.dpid
            self.ip2host["%s"%host.ipv4] = host


    events = [event.EventSwitchEnter, event.EventSwitchLeave,
               event.EventSwitchReconnected,
               event.EventPortAdd, event.EventPortDelete,
               event.EventPortModify,
               event.EventLinkAdd, event.EventLinkDelete]

    @set_ev_cls(events)
    def get_topology(self,ev):
        if not self.net_arrived:
            return

        LOCK.acquire()
        self.net_arrived -= 1
        if self.net_arrived < 0:
            self.net_arrived = 0
        LOCK.release()

        self.net_flag = False
        self.net_topo = []

        print("-----------------get_topology")
        #获取所有的交换机、链路
        self.switch_list = get_switch(self.topology_api_app)  #1.只要交换机与控制器联通，就可以获取
        self.link_list = get_link(self.topology_api_app)      #2.在ryu启动时，加上--observe-links即可用于拓 
        # 扑发现

        #获取交换机字典id2dpid{id:dpid} dpid2switch{dpid:switch object}
        for i,switch in enumerate(self.switch_list):
            self.id2dpid[i] = switch.dp.id
            self.dpid2id[switch.dp.id] = i
            self.dpid2switch[switch.dp.id] = switch


        #根据链路信息，开始获取拓扑信息
        self.net_size = len(self.id2dpid) #表示网络中交换机个数
        for i in range(self.net_size):
            self.net_topo.append([0]*self.net_size)

        for link in self.link_list:
            src_dpid = link.src.dpid
            src_port = link.src.port_no

            dst_dpid = link.dst.dpid
            dst_port = link.dst.port_no

            try:
                sid = self.dpid2id[src_dpid]
                did = self.dpid2id[dst_dpid]
            except KeyError as e:
                print("--------------Error:get KeyError with link infomation(%s)"%e)
                return
            self.net_topo[sid][did] = [src_port,0] #注意：这里0表示存在链路，后面可以修改为时延
            self.net_topo[did][sid] = [dst_port,0] #注意：修改为列表，不要用元组，元组无法修改，我们后面要修改时延

        self.net_flag = True #表示网络拓扑创建成功

    def show_topology(self):
        print("-----------------show_topology")
        print("----------switch network----------")
        line_info = "         "
        for i in range(self.net_size):
            line_info+="        s%-5d        "%self.id2dpid[i]
        print(line_info)
        for i in range(self.net_size):
            line_info = "s%d      "%self.id2dpid[i]
            for j in range(self.net_size):
                if self.net_topo[i][j] == 0:
                    line_info+="%-22d"%0
                else:
                    line_info+="(%d,%.12f)    "%tuple(self.net_topo[i][j])
            print(line_info)

        print("----------host 2 switch----------")
        for key,val in self.ip2switch.items():
            print("%s---s%d"%(key,val))
```

启动代码

`ryu-manager TopoDetect.py --verbose --observe-links`

`--verbose` 打印Debug信息.

`--observe-links` 使能LLDP探测报文周期性发送

#### 3.4.5 网络链路时延探测

##### (1) 基本设计思路

网络时延探测应用利用了Ryu自带的Switches模块的数据，获取到了LLDP数据发送时的时间戳，然后和收到的时间戳进行相减，得到了LLDP数据包从控制器下发到交换机A，然后从交换机A到交换机B，再上报给控制器的时延T1，示例见图1的蓝色箭头。

同理反向的时延T2由绿色的箭头组成。

此外，控制器到交换机的往返时延由一个蓝色箭头和一个绿色箭头组成，此部分时延由echo报文测试，分别为Ta，Tb。最后链路的前向后向平均时延<img src="https://www.zhihu.com/equation?tex=T%3D%5Cfrac%7BT1%2BT2-Ta-Tb%7D%7B2%7D" alt="T=\frac{T1+T2-Ta-Tb}{2}" class="ee_img tr_noresize" eeimg="1">

![image-20240809215824139](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/4016ab178aa90e90-image-20240809215824139.png)

记录LLDP数据报文总时间 即

-   1.  controller 向所有交换机下发流表： 当接收到目的MAC地址为最近相邻交换机的LLDP报文时，不缓存该LLDP报文，全部上传给控制器(switches.py中实现)

-   1.  controller 针对每个交换机端口，生成一个对应的LLDP报文打上controller此时的时间戳(switches.py中实现)

-   1.  controller 下发 PacketOut 消息给该交换机，要求该交换机转发生成的LLDP报文(switches.py中实现)

-   1.  交换机根据自身的流表把LLDP报文上传给控制器(switches.py中实现)

-   1.  控制器接收到该报文，报文的帧头部存储了该报文是从哪个交换机收上来的，报文的body部分存储了报文是控制器下发给哪个交换机的(本文件实现)

-   1.  确定由controller------switchA-------switchB------controller的时间(本文件实现)

##### (2) 代码实现与解读

```python
from ryu.base import app_manager
from ryu.base.app_manager import lookup_service_brick

from ryu.ofproto import ofproto_v1_3

from ryu.controller import ofp_event
from ryu.controller.handler import MAIN_DISPATCHER,CONFIG_DISPATCHER,DEAD_DISPATCHER,HANDSHAKE_DISPATCHER #只是表示datapath数据路径的状态
from ryu.controller.handler import set_ev_cls

from ryu.lib import hub
from ryu.lib.packet import packet,ethernet

from ryu.topology.switches import Switches
from ryu.topology.switches import LLDPPacket

import time

ECHO_REQUEST_INTERVAL = 0.05
DELAY_DETECTING_PERIOD = 5


# =========================================定义时延检测类============================================
class DelayDetect(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    def __init__(self,*args,**kwargs):
        super(DelayDetect,self).__init__(*args,**kwargs)
        self.name = "delay"
    	#注意：我们使用lookup_service_brick加载模块实例时，对于我们自己定义的app,我们需要在类中定义self.name。
        self.topology = lookup_service_brick("topology")
    	#此外，最重要的是：我们启动本模块DelayDetect时，必须同时启动自定义的模块！！！ 比如：
        #ryu-manager ./TopoDetect.py ./DelayDetect.py --verbose --observe-links
        self.switches = lookup_service_brick("switches") 

        self.dpid2switch = {} # 或者直接为{}，也可以。下面_state_change_handler也会添加进去
        self.dpid2echoDelay = {} # 记录echo时延
    	
        #记录LLDP报文测量的时延。实际上可以直接更新，这里单独记录，为了单独展示 
        # {”src_dpid-srt_port-dst_dpid“：delay}
        self.src_sport_dst2Delay = {} 

        self.detector_thread = hub.spawn(self._detector)
    # =========================================协程获取链路时延======================================
    def _detector(self):
        """
        协程实现伪并发，探测链路时延
        """
        while True:
            if self.topology == None:
                self.topology = lookup_service_brick("topology")
            if self.topology.net_flag:
                #print("------------------_detector------------------")
                self._send_echo_request()
                self.get_link_delay()
                if self.topology.net_flag:
                    try:
                        self.show_delay()
                        self.topology.show_topology()　　#拓扑显示
                    except Exception as err:
                        print("------------------Detect delay failure!!!------------------")
            hub.sleep(DELAY_DETECTING_PERIOD) #5秒一次
     # =========================================协程获取Echo时延======================================
    def _send_echo_request(self):
        """
        发生echo报文到datapath
        """
        for datapath in self.dpid2switch.values():
            parser = datapath.ofproto_parser
            #获取当前时间
            echo_req = parser.OFPEchoRequest(datapath,data=bytes("%.12f"%time.time(),encoding="utf8")) 
            datapath.send_msg(echo_req)

            #重要！不要同时发送echo请求，因为它几乎同时会生成大量echo回复。
            #在echo_reply_处理程序中处理echo reply时，会产生大量队列等待延迟。
            hub.sleep(ECHO_REQUEST_INTERVAL)

    @set_ev_cls(ofp_event.EventOFPEchoReply,[MAIN_DISPATCHER,CONFIG_DISPATCHER,HANDSHAKE_DISPATCHER])
    def echo_reply_handler(self,ev):
        """
        处理echo响应报文，获取控制器到交换机的链路往返时延
                  Controller
                      |    
         echo latency |  
                      |
                    Switch        
        """
        now_timestamp = time.time()
        try:
            echo_delay = now_timestamp - eval(ev.msg.data)
            self.dpid2echoDelay[ev.msg.datapath.id] = echo_delay
        except:
            return

    # =========================================获取LLDP时延=====================================
    """
    通过lookup_service_brick("switches")，实例化了switches模块。详细见：3.4.1,该模块中通过协程实现了周期0.05s发送
    LLDP数据包, 所以我们下面可以直接获取LLDP数据报。
    """
    @set_ev_cls(ofp_event.EventOFPPacketIn,MAIN_DISPATCHER)
    def packet_in_handler(self,ev): 
        """
        处理到达的LLDP报文，从而获得LLDP时延
                     Controller
                    |        /|\    
                   \|/         |
                Switch----->Switch
        """
        msg = ev.msg
        try:
            #获取两个相邻交换机的源交换机dpid和port_no(与目的交换机相连的端口)
            src_dpid,src_outport = LLDPPacket.lldp_parse(msg.data) 
            #获取目的交换机（第二个），因为来到控制器的消息是由第二个（目的）交换机上传过来的
            dst_dpid = msg.datapath.id 
            dst_inport = msg.match['in_port']
            if self.switches is None:
                #获取交换机模块实例
                self.switches = lookup_service_brick("switches") 

            #获得key（Port类实例）和data（PortData类实例）
            for port in self.switches.ports.keys(): #开始获取对应交换机端口的发送时间戳
                if src_dpid == port.dpid and src_outport == port.port_no: #匹配key
                    port_data = self.switches.ports[port] 
                    timestamp = port_data.timestamp
                    if timestamp:
                        delay = time.time() - timestamp
                        self._save_delay_data(src=src_dpid,dst=dst_dpid,
                                  src_port=src_outport,lldpdealy=delay)
        except:
            return

    def _save_delay_data(self,src,dst,src_port,lldpdealy):
        key = "%s-%s-%s"%(src,src_port,dst)
        self.src_sport_dst2Delay[key] = lldpdealy
    # =====================根据LLDP和Echo时延，更新网络拓扑图中的权值信息==================  
    def get_link_delay(self):
        """
        更新图中的权值信息
        """
        print("--------------get_link_delay-----------")
        for src_sport_dst in self.src_sport_dst2Delay.keys():
                src,sport,dst = tuple(map(eval,src_sport_dst.split("-")))
                if src in self.dpid2echoDelay.keys() and dst in self.dpid2echoDelay.keys():
                    sid,did = self.topology.dpid2id[src],self.topology.dpid2id[dst]
                    if self.topology.net_topo[sid][did] != 0:
                        if self.topology.net_topo[sid][did][0] == sport:
                            s_d_delay = self.src_sport_dst2Delay[src_sport_dst]-
                            (self.dpid2echoDelay[src]+self.dpid2echoDelay[dst])/2;
                            #注意：可能出现单向计算时延导致最后小于0，这是不允许的。则不进行更新，使用上一次原始值
                            if s_d_delay < 0: 
                                continue
                            self.topology.net_topo[sid][did][1] =\                                                                              self.src_sport_dst2Delay[src_sport_dst]-\
                                        (self.dpid2echoDelay[src]+\
                                         self.dpid2echoDelay[dst])/2
    # ===========================显示网络拓扑图和Echo、LLDP时延信息===================
    @set_ev_cls(ofp_event.EventOFPStateChange,[MAIN_DISPATCHER, DEAD_DISPATCHER])
    def _state_change_handler(self, ev):
        datapath = ev.datapath
        if ev.state == MAIN_DISPATCHER:
            if not datapath.id in self.dpid2switch:
                self.logger.debug('Register datapath: %016x', datapath.id)
                self.dpid2switch[datapath.id] = datapath
        elif ev.state == DEAD_DISPATCHER:
            if datapath.id in self.dpid2switch:
                self.logger.debug('Unregister datapath: %016x', datapath.id)
                del self.dpid2switch[datapath.id]

        if self.topology == None:
            self.topology = lookup_service_brick("topology")
        print("-----------------------_state_change_handler-----------------------")
        print(self.topology.show_topology())
        print(self.switches)

    def show_delay(self):
        print("-----------------------show echo delay-----------------------")
        for key,val in self.dpid2echoDelay.items():
            print("s%d----%.12f"%(key,val))
        print("-----------------------show LLDP delay-----------------------")
        for key,val in self.src_sport_dst2Delay.items():
            print("%s----%.12f"%(key,val))
```

#### 3.4.6 ARP代理应用

在传统网络中，存在着一定的广播流量，占据了一部分的网络带宽。同时，在有环的拓扑中，如果不运行某些协议，广播数据还会引起网络风暴，使网络瘫痪。传统的解决方案是运行STP（生成树协议），来解决环路带来的风暴隐患。本节将介绍如何在控制器RYU上开发ARP代理模块，用于代理回复ARP请求，以及解决环状拓扑风暴的问题。 然而本文这个简单的APP，并没有很好地解决ARP的问题，因为ARP也有生存时间。而过时的数据会影响网络的正常运行，所以，进一步的优化将是设置ARP记录的刷新时间。以及`sw{dpid, eth_src,arp_dst_ip}`的刷新时间。以保证数据的有效性。推而广之，我们可以按照这样的模式去处理其他的广播数据，如DHCP。更多的功能数据包或者信令数据包的代理，都可以模仿本篇的流程实现。

[18张图详解ARP协议所有细节](https://cloud.tencent.com/developer/article/1948193)

##### (1) 基本设计思路

![image-20240811162248461](https://gitee.com/wangwzzb/openpictures/raw/master-md2zhihu-asset/SDN_learn_note_7/dd3f816c8ab1b49e-image-20240811162248461.png)

```python
packet_in
    |
    |
  ARP learning
  MAC_to_Port learning
    |
    |
    |               No  
Multicast? -------------------------------------------->|
    |                                                   |
    | Yes                                               |
    |                                                   |
    |                                                   |
    |      No                                           |
   loop? ----->(dpid,eth_src,dst_ip)learning            |
    |                   |                               |
    |                   |                               |
    |                   |               No              |         No
    |Yes        dst_ip in arp_table? ------->dst in mac_to_port? ---->Flood
    |                   |                               |               |
    |                   |Yes                            |Yes            |
    |                   |                               |               |
   drop             ARP_REPLY                       flow_mod            |
    |                   |                               |               |
    |                   |                               |               |
    |<------------------|<------------------------------|<--------------|               
    |
    |
    end
```

``

##### (2) 代码解决与实现

-   模块代码: 防止成环代码

    -   在回复ARP请求之前，必须要解决的是网络环路问题。我们的解决方案是：以（dpid, eth_src,arp_dst_ip）为key，记录第一个数据包的in_port，并将从网络中返回的数据包丢弃，保证同一个交换机中的某一个广播数据包只能有一个入口，从而防止成环。在此应用中默认网络中发起通信的第一个数据包都是ARP数据包。

    -   `sw[(datapath.id, eth_src, arp_dst_ip)] = in_port`

    -   ```python
        if eth_dst == ETHERNET_MULTICAST and ARP in header_list:
            arp_dst_ip = header_list[ARP].dst_ip
            if (datapath.id, eth_src, arp_dst_ip) in self.sw:  # Break the loop
                if self.sw[(datapath.id, eth_src, arp_dst_ip)] != in_port:
                    out = datapath.ofproto_parser.OFPPacketOut(
                        datapath=datapath,
                        buffer_id=datapath.ofproto.OFP_NO_BUFFER,
                        in_port=in_port,
                        actions=[], data=None)
                    datapath.send_msg(out)
                    return True
            else:
                self.sw[(datapath.id, eth_src, arp_dst_ip)] = in_port
        ```

-   模块代码: ARP回复代码

    -   解决完环路拓扑中存在的广播风暴问题之后，我们还需要利用SDN控制器可获取网络全局的信息的能力，去代理回复ARP请求，从而减少网络中泛洪的ARP请求数据。这个逻辑非常简单，和二层学习原理基本一样，也是通过自学习主机ARP记录，再通过查询记录并回复。具体代码实现如下：

    -   ```python
        if ARP in header_list:
            hwtype = header_list[ARP].hwtype
            proto = header_list[ARP].proto
            hlen = header_list[ARP].hlen
            plen = header_list[ARP].plen
            opcode = header_list[ARP].opcode

            arp_src_ip = header_list[ARP].src_ip
            arp_dst_ip = header_list[ARP].dst_ip

            actions = []

            if opcode == arp.ARP_REQUEST:
                if arp_dst_ip in self.arp_table:  # arp reply
                    actions.append(datapath.ofproto_parser.OFPActionOutput(
                        in_port)
                    )

                    ARP_Reply = packet.Packet()
                    ARP_Reply.add_protocol(ethernet.ethernet(
                        ethertype=header_list[ETHERNET].ethertype,
                        dst=eth_src,
                        src=self.arp_table[arp_dst_ip]))
                    ARP_Reply.add_protocol(arp.arp(
                        opcode=arp.ARP_REPLY,
                        src_mac=self.arp_table[arp_dst_ip],
                        src_ip=arp_dst_ip,
                        dst_mac=eth_src,
                        dst_ip=arp_src_ip))

                    ARP_Reply.serialize()

                    out = datapath.ofproto_parser.OFPPacketOut(
                        datapath=datapath,
                        buffer_id=datapath.ofproto.OFP_NO_BUFFER,
                        in_port=datapath.ofproto.OFPP_CONTROLLER,
                        actions=actions, data=ARP_Reply.data)
                    datapath.send_msg(out)
                    return True
        return False
        ```

-   完成实现代码

    -   ```python
        from ryu.base import app_manager
        from ryu.controller import ofp_event
        from ryu.controller.handler import CONFIG_DISPATCHER, MAIN_DISPATCHER
        from ryu.controller.handler import set_ev_cls
        from ryu.ofproto import ofproto_v1_3
        from ryu.lib.packet import packet
        from ryu.lib.packet import ethernet
        from ryu.lib.packet import arp

        ETHERNET = ethernet.ethernet.__name__
        ETHERNET_MULTICAST = "ff:ff:ff:ff:ff:ff"
        ARP = arp.arp.__name__


        class ARP_PROXY_13(app_manager.RyuApp):
            OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

            def __init__(self, *args, **kwargs):
                super(ARP_PROXY_13, self).__init__(*args, **kwargs)
                self.mac_to_port = {}
                self.arp_table = {}
                self.sw = {}

            @set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)
            def switch_features_handler(self, ev):
                datapath = ev.msg.datapath
                ofproto = datapath.ofproto
                parser = datapath.ofproto_parser

                # install table-miss flow entry
                #
                # We specify NO BUFFER to max_len of the output action due to
                # OVS bug. At this moment, if we specify a lesser number, e.g.,
                # 128, OVS will send Packet-In with invalid buffer_id and
                # truncated packet data. In that case, we cannot output packets
                # correctly.

                match = parser.OFPMatch()
                actions = [parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,
                                                  ofproto.OFPCML_NO_BUFFER)]
                self.add_flow(datapath, 0, match, actions)

            def add_flow(self, datapath, priority, match, actions):
                ofproto = datapath.ofproto
                parser = datapath.ofproto_parser

                inst = [parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,
                                                     actions)]

                mod = parser.OFPFlowMod(datapath=datapath, priority=priority,
                                        idle_timeout=5, hard_timeout=15,
                                        match=match, instructions=inst)
                datapath.send_msg(mod)

            @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
            def _packet_in_handler(self, ev):
                msg = ev.msg
                datapath = msg.datapath
                ofproto = datapath.ofproto
                parser = datapath.ofproto_parser
                in_port = msg.match['in_port']

                pkt = packet.Packet(msg.data)

                eth = pkt.get_protocols(ethernet.ethernet)[0]
                dst = eth.dst
                src = eth.src
                dpid = datapath.id

                header_list = dict(
                    (p.protocol_name, p)for p in pkt.protocols if type(p) != str)
                if ARP in header_list:
                    self.arp_table[header_list[ARP].src_ip] = src  # ARP learning

                self.mac_to_port.setdefault(dpid, {})
                self.logger.info("packet in %s %s %s %s", dpid, src, dst, in_port)

                # learn a mac address to avoid FLOOD next time.
                self.mac_to_port[dpid][src] = in_port

                if dst in self.mac_to_port[dpid]:
                    out_port = self.mac_to_port[dpid][dst]
                else:
                    if self.arp_handler(header_list, datapath, in_port, msg.buffer_id):
                        # 1:reply or drop;  0: flood
                        print "ARP_PROXY_13"
                        return None
                    else:
                        out_port = ofproto.OFPP_FLOOD
                        print 'OFPP_FLOOD'

                actions = [parser.OFPActionOutput(out_port)]

                # install a flow to avoid packet_in next time
                if out_port != ofproto.OFPP_FLOOD:
                    match = parser.OFPMatch(in_port=in_port, eth_dst=dst)
                    self.add_flow(datapath, 1, match, actions)

                data = None
                if msg.buffer_id == ofproto.OFP_NO_BUFFER:
                    data = msg.data
                out = parser.OFPPacketOut(datapath=datapath, buffer_id=msg.buffer_id,
                                          in_port=in_port, actions=actions, data=data)
                datapath.send_msg(out)

            def arp_handler(self, header_list, datapath, in_port, msg_buffer_id):
                header_list = header_list
                datapath = datapath
                in_port = in_port

                if ETHERNET in header_list:
                    eth_dst = header_list[ETHERNET].dst
                    eth_src = header_list[ETHERNET].src

                if eth_dst == ETHERNET_MULTICAST and ARP in header_list:
                    arp_dst_ip = header_list[ARP].dst_ip
                    if (datapath.id, eth_src, arp_dst_ip) in self.sw:  # Break the loop
                        if self.sw[(datapath.id, eth_src, arp_dst_ip)] != in_port:
                            out = datapath.ofproto_parser.OFPPacketOut(
                                datapath=datapath,
                                buffer_id=datapath.ofproto.OFP_NO_BUFFER,
                                in_port=in_port,
                                actions=[], data=None)
                            datapath.send_msg(out)
                            return True
                    else:
                        self.sw[(datapath.id, eth_src, arp_dst_ip)] = in_port

                if ARP in header_list:
                    hwtype = header_list[ARP].hwtype
                    proto = header_list[ARP].proto
                    hlen = header_list[ARP].hlen
                    plen = header_list[ARP].plen
                    opcode = header_list[ARP].opcode

                    arp_src_ip = header_list[ARP].src_ip
                    arp_dst_ip = header_list[ARP].dst_ip

                    actions = []

                    if opcode == arp.ARP_REQUEST:
                        if arp_dst_ip in self.arp_table:  # arp reply
                            actions.append(datapath.ofproto_parser.OFPActionOutput(
                                in_port)
                            )

                            ARP_Reply = packet.Packet()
                            ARP_Reply.add_protocol(ethernet.ethernet(
                                ethertype=header_list[ETHERNET].ethertype,
                                dst=eth_src,
                                src=self.arp_table[arp_dst_ip]))
                            ARP_Reply.add_protocol(arp.arp(
                                opcode=arp.ARP_REPLY,
                                src_mac=self.arp_table[arp_dst_ip],
                                src_ip=arp_dst_ip,
                                dst_mac=eth_src,
                                dst_ip=arp_src_ip))

                            ARP_Reply.serialize()

                            out = datapath.ofproto_parser.OFPPacketOut(
                                datapath=datapath,
                                buffer_id=datapath.ofproto.OFP_NO_BUFFER,
                                in_port=datapath.ofproto.OFPP_CONTROLLER,
                                actions=actions, data=ARP_Reply.data)
                            datapath.send_msg(out)
                            return True
                return False
        ```



Reference:

