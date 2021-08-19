# sdn

## mininet tutorial

[Mininet Topologies and Mininet Python API | Coursera](https://www.coursera.org/learn/sdn/lecture/sYg1F/mininet-topologies-and-mininet-python-api)



![image-20210817202345627](coursera_sdn.assets/image-20210817202345627.png)

![image-20210817202356608](coursera_sdn.assets/image-20210817202356608.png)

如果此时退出mn程序，要马上进行一个新的实验

![image-20210817202603902](coursera_sdn.assets/image-20210817202603902.png)

则会提示删除原先的controller，这可能与源代码的控制器保存机制有关。但不影响体验，程序仍然包含了这个相关内容，会自动删除。我们再运行一遍即可。

![image-20210817202851912](coursera_sdn.assets/image-20210817202851912.png)

三个host连接到一个switch上面

---

![image-20210817203011959](coursera_sdn.assets/image-20210817203011959.png)



实际上，mn底层调用的是python函数。

----

在每次运行新的程序时，一定要清楚之前的controller

![image-20210818074907830](coursera_sdn.assets/image-20210818074907830.png)

就像在walkthrough文档中提到的：

[Mininet Walkthrough - Mininet](http://mininet.org/walkthrough/#cleanup)

> sudo mn -c 

这一句指令即可实现目的。

![image-20210818075120952](coursera_sdn.assets/image-20210818075120952.png)

![image-20210818080334765](coursera_sdn.assets/image-20210818080334765.png)

当然，课程中也进行了 较为复杂的内容展示。无非就是调用了更多的api，代码结构采用了对象等使得结构更加简洁的方式来处理。

```PYTHON
#!/usr/bin/python

from mininet.topo import Topo
from mininet.net import Mininet
from mininet.util import dumpNodeConnections
from mininet.log import setLogLevel

class SingleSwitchTopo(Topo):
    "Single Switch connected to n hosts."
    def __init__(self,n=2,**opts):

        # initializae topology and default options
        Topo.__init__(self,**opts)
        switch = self.addSwitch('s1')

        for h in range(n):
            host = self.addHost('h%s'%(h+1))
            self.addLink(host,switch)


def simpleTest():
    #create and test a simple network"
    topo = SingleSwitchTopo(n=4)
    net = Mininet(topo)

    net.start()

    print "Dumping host conections"
    dumpNodeConnections(net.hosts)
    print "Testing network connectivity"
    net.pingAll()
    net.stop()

if __name__=='__main__':
    setLogLevel('info')
    simpleTest()
```

![image-20210818090914631](coursera_sdn.assets/image-20210818090914631.png)

之后的课程会涉及一些本节课没讲到的

![image-20210818080414951](coursera_sdn.assets/image-20210818080414951.png)

---

第二课即开始讲解一些mininet的操作，mininet指令环境，linux环境，sudo权限环境

![image-20210818092204534](coursera_sdn.assets/image-20210818092204534.png)

![image-20210818092211335](coursera_sdn.assets/image-20210818092211335.png)

![image-20210818092221760](coursera_sdn.assets/image-20210818092221760.png)

dpctl指令对debug用处很大，增强一个控制器流表的能见度和控制属性

但是有的节点没法ping通，是要注意到 switch的flow table是空的，在没有建立映射关系前，OFS也不知道怎么传输数据。

---

dpctl: failed to send packet to switch: Connection refused

这个是在跟随课程实验时遇到的问题



192.168.31.176





跟着coursera的课有bug，试着跟着官方文档。 

如果还有错误，就重新下载镜像处理



xterm能用了，解决方式就是下载了一个开源的软件xming，起到一个位linux服务器终端开通GUI界面的途径

[ssh - Ubuntu Windows10 App -- X11 Forwarding -- $DISPLAY Error - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/427574/ubuntu-windows10-app-x11-forwarding-display-error#comment1213744_489406)

[Xming X Server 配置和使用_小宋的博客-CSDN博客_xming使用](https://blog.csdn.net/qq_39101111/article/details/78727355)



图形化显示问题解决了，但是

dpctl这几个需要访问端口的操作仍然是失败的

![image-20210818152511764](coursera_sdn.assets/image-20210818152511764.png)

wireshark无法启动也是一个问题

[Problems attaching over X11 and Xming (kinetic) · Issue #9 · ros-visualization/rqt_image_view (github.com)](https://github.com/ros-visualization/rqt_image_view/issues/9)

这里提到了，可能由于xming是一个老工具

现在用一个新的工具试试

[VcXsrv Windows X Server download | SourceForge.net](https://sourceforge.net/projects/vcxsrv/)

![image-20210818155920281](coursera_sdn.assets/image-20210818155920281.png)

确实，更新软件之后，这里就可行了。

而且也确实发现这个软件2021年还在维护

[Mininet Walkthrough - Mininet](http://mininet.org/walkthrough/#cleanup)

继续跟着workthrough中进行操作

注意到：Note that *only* the network is virtualized; each host process sees the same set of processes and directories. For example, print the process list from a host process:

用ps打印指令，发现host都能了解network已经执行了什么操作

![image-20210818160942809](coursera_sdn.assets/image-20210818160942809.png)

在启动mn程序时，加上--mac字段，能让设备初始化的地址尽可能的小和有序，方便debug

巧妙利用xterm的存在

![image-20210818162940033](coursera_sdn.assets/image-20210818162940033.png)

在交换机上配置指令，

然后在其他host上执行操作，如ping，以起到影响flow table的效果

![image-20210818163036440](coursera_sdn.assets/image-20210818163036440.png)

---

有一点注意，coursera课程中的端口127.0.0.1:6634是在他们的环境上

 ![image-20210818163319809](coursera_sdn.assets/image-20210818163319809.png)

本机提示的虽然在6653上创建端口，但我这边就要后备端口6654，原因可能可以借鉴下方

![image-20210818163401360](coursera_sdn.assets/image-20210818163401360.png)

当启用远程remote controller来运行程序时

 h2 ping -c3 h3

这种ping的操作可能无法执行，因为

> As you saw before, switch flow table is empty. Besides that, there is no controller connected to the switch and therefore the switch doesn’t know what to do with incoming traffic, leading to ping failure.

解决方案：![image-20210818163743910](coursera_sdn.assets/image-20210818163743910.png)

注意，端口号在本机上改为6654

![image-20210818163858824](coursera_sdn.assets/image-20210818163858824.png)

但直接这么设置，返回另一个cli是不成功的，原来是这种dump-flows是有时效性的

> Do you get replies now? Check the flow-table again and look the statistics for each flow entry. Is this what you expected to see based on the ping traffic? NOTE: if you didn’t see any ping replies coming through, it might be the case that the flow-entries expired before you start your ping test. When you do a ”dpctl dump-flows” you can see an ”idle timeout” option for each entry. This means that the flow will expire after this many secs if there is no incoming traffic. Run again respecting this limit, or install a flow-entry with longer timeout.

![image-20210818164303619](coursera_sdn.assets/image-20210818164303619.png)

之前两条装载指令加上idle时限即可

![image-20210818164329842](coursera_sdn.assets/image-20210818164329842.png)

---

**一些关键术语**：

OpenFlow Interface: a standard open interface between the OpenFlow controller and the OpenFlow programmable devices (i.e., switches etc)

OpenFlow Controller: sits above the OpenFlow interface. The OpenFlow reference distribution includes a controller that acts as an Ethernet learning switch in combination with an OpenFlow switch. You'll run it and look at messages being sent.

OpenFlow Switch: sits below the OpenFlow interface. The OpenFlow reference distribution includes a user-space software switch. Open vSwitch is another software but kernel-based switch, while there is a number of hardware switches available from Broadcom (Stanford Indigo release), HP, NEC, and others.

dpctl: command-line utility that sends quick OpenFlow messages, useful for viewing switch port and flow stats, plus manually inserting flow entries.

Wireshark: general (non-OF-specific) graphical utility for viewing packets. The OpenFlow reference distribution includes a Wireshark dissector, which parses OpenFlow messages sent to the OpenFlow default port (6633) in a conveniently readable way.

iperf: general command-line utility for testing the speed of a single TCP connection.

Mininet: network emulation platform. Mininet creates a virtual OpenFlow network - controller, switches, hosts, and links - on a single real or virtual machine. More Mininet details can be found at the Mininet web page .

cbench: utility for testing the flow setup rate of OpenFlow controllers.

---

OpenFlow 接口：OpenFlow 控制器和 OpenFlow 可编程设备（即交换机等）之间的标准开放接口 

OpenFlow 控制器：位于 OpenFlow 接口之上。 OpenFlow 参考分布包括一个控制器，该控制器与 OpenFlow 交换机结合用作以太网学习交换机。 您将运行它并查看正在发送的消息。

OpenFlow 交换机：位于 OpenFlow 接口下方。 OpenFlow 参考分布包括用户空间软件交换机。 

Open vSwitch 是另一种基于内核的软件交换机，而 Broadcom（Stanford Indigo 版本）、HP、NEC 和其他公司提供了许多硬件交换机。

dpctl：发送快速 OpenFlow 消息的命令行实用程序，可用于查看交换机端口和流统计信息，以及手动插入流条目。

Wireshark：用于查看数据包的通用（非 OF 特定）图形实用程序。 OpenFlow 参考分布包括一个 Wireshark 解析器，它以一种方便可读的方式解析发送到 OpenFlow 默认端口 (6633) 的 OpenFlow 消息。

iperf：用于测试单个 TCP 连接速度的通用命令行实用程序。

Mininet：网络仿真平台。 Mininet 在单个真实或虚拟机上创建虚拟 OpenFlow 网络——控制器、交换机、主机和链接。 更多 Mininet 详细信息可以在 Mininet 网页上找到。

cbench：用于测试 OpenFlow 控制器的流设置速率的实用程序。

---

继续学习新的课程

overview

- what is control/data seperation
- why it's a good idea
- 机会和挑战都是什么

在多种领域的机会

- 路由，数据中心等

挑战和方法

- scaling，reliability



### control and data planes

#### control plane

logic for controlling forwading behavior，控制转发行为的存在，类似于控制人类整体的大脑。实际例子包含：- 路由协议，- network middlebox configuration

#### data plane

forward traffic 即实际上执行转发流量的操作，其根据的规则是来自控制平面

其实际例子包括ip转发层，被创建的路由表。通常它的代表设备是硬件，但是现在越来越多的被以软件的方式实现



为什么要分离：

1. independent evolution and development

   软件对网络的控制与硬件的限制分离

2. control from high-level software program

   更容易推理网络行为，也更容易debug

---

### Opportunities in various Domains

维护，egress selection security 很有帮助，方便我们对架构进行升级，容错性也高

#### Interdomain routing: constrained policies

之前的背景：

![image-20210818172851910](coursera_sdn.assets/image-20210818172851910.png)

有了SDN：route controller can directly update state 不需要再走那些步骤和流程，利用控制器的存在直接更新整个网络的情况。

---

dry out

egress selection

​	不用再为每个出口单独部署流量出规则，直接由控制屏幕中央控制即可

better bgp security

上面的内容都或多或少借助了RCP的作用，但是由于都是指定prefix进行传播，导致网络对流量的理解十分有限，能提供的动作也十分有限了。

data centers

![image-20210819075335743](coursera_sdn.assets/image-20210819075335743.png)

![image-20210819075640885](coursera_sdn.assets/image-20210819075640885.png)

layer 2让问题变成了topology specific而不是topology dependent

![image-20210819075809699](coursera_sdn.assets/image-20210819075809699.png)

readdress the host，so we can use the mac 

![image-20210819080059191](coursera_sdn.assets/image-20210819080059191.png)

---

### challenges in seperating the data and control planes

![image-20210819080215393](coursera_sdn.assets/image-20210819080215393.png)

SOLVING approaches: RCP,ONIX

> 查询一下RCP的资料，之前忘记整理
>
> [BGP-based Routing Control Platform (RCP) (network-insight.net)](https://network-insight.net/2015/11/bgp-based-routing-control-platform-rcp/)
>
> The Routing Control Platform (RCP) is a centralized forwarding solution enabling the collection of a network topology map, running an algorithm and selecting preferred BGP route for each router in an Autonomous System (AS). RCP是一种中心化的转发方案，使得运行网络拓扑映射的集合通过运行算法来为每个在AS中的路由器选取适合的BGP路由
>
> It does this by peering both IGP and iBGP to neighboring routers and communicates the preferred routes using *unmodified iBGP*. 它（算法）通过与邻居路由器建立IGP以及iBGP对等体，并利用无修改的iBGP来交换各自prefer的路由
>
> It acts similar to that of an enhanced route reflector and does not sit in the data path.  它的行为像是一个路由反射器，但是并不出现在数据路径当中
>
> ![image-20210819080927055](coursera_sdn.assets/image-20210819080927055.png)

所以下面，以RCP为例子，探讨一下所出现的问题

![image-20210819081412947](coursera_sdn.assets/image-20210819081412947.png)

![image-20210819081531571](coursera_sdn.assets/image-20210819081531571.png)

把动态成本化为静态，存储下来即可。（但这样同样会有设备达到其限制）

![image-20210819081644908](coursera_sdn.assets/image-20210819081644908.png)

第二点，reliability for RCP

- replicate RCPs("Hot Spare热备")
  - run multiple identical servers
- run independent replicas
  - each replica has its own feed of routes
  - each replica receives the same inputs and runs the same routing algorithm
  - no need for a consistency protocol if both replicas always see the same information

![image-20210819081945940](coursera_sdn.assets/image-20210819081945940.png)

可能没法保证一致性，因为考虑的内容不太相同。

![image-20210819082158155](coursera_sdn.assets/image-20210819082158155.png)

![image-20210819083236950](coursera_sdn.assets/image-20210819083236950.png)

上述三大挑战是很关键的。

----

这个课后测试头皮发麻，

![image-20210819084721528](coursera_sdn.assets/image-20210819084721528.png)

![image-20210819084733215](coursera_sdn.assets/image-20210819084733215.png)

![image-20210819084814727](coursera_sdn.assets/image-20210819084814727.png)	![image-20210819084837392](coursera_sdn.assets/image-20210819084837392.png)

![image-20210819084908588](coursera_sdn.assets/image-20210819084908588.png)

![image-20210819084926542](coursera_sdn.assets/image-20210819084926542.png)

![image-20210819084933291](coursera_sdn.assets/image-20210819084933291.png)

![image-20210819084941540](coursera_sdn.assets/image-20210819084941540.png)

![image-20210819084955429](coursera_sdn.assets/image-20210819084955429.png)

![image-20210819085002164](coursera_sdn.assets/image-20210819085002164.png)

---

### routing Control Platform

![image-20210819092020648](coursera_sdn.assets/image-20210819092020648.png)

一些BGP面临的问题

![image-20210819092101029](coursera_sdn.assets/image-20210819092101029.png)

![image-20210819092125466](coursera_sdn.assets/image-20210819092125466.png)

路由器之间是某种情况下独立的，没有任何路由器具有整个网络完整的BGP状态

![image-20210819092404758](coursera_sdn.assets/image-20210819092404758.png)

![image-20210819092413758](coursera_sdn.assets/image-20210819092413758.png)

政策总是在每个网络局部被分袂，所以要找到一个代表统一网络范围政策是相当困难的

路由本身必须携带状态，但上图这种分解让过程relatively difficult。

所以，

#### 引入RCP是有好处的

![image-20210819092629560](coursera_sdn.assets/image-20210819092629560.png)

有了RCP的控制，路由器本身不必使用状态路由标记，在配置上也简单多了

![image-20210819092722300](coursera_sdn.assets/image-20210819092722300.png)

且原始状况下，BGP会以意想不到的方式与底层进行交互，例如考虑IGP权重，会让host之间流量转发产生环路

![image-20210819092841026](coursera_sdn.assets/image-20210819092841026.png)

但例如RCP，会学习所有外部路由，具有完整路由信息。避免的内部环路

![image-20210819093007438](coursera_sdn.assets/image-20210819093007438.png)

但面临的问题就是，在所有设备已经运行IGP 相互做出独立决策的情况下由RCP来指定转发行为

#### control protocal interactions

![image-20210819093127593](coursera_sdn.assets/image-20210819093127593.png)

![image-20210819093311970](coursera_sdn.assets/image-20210819093311970.png)

设置固定点，即使区域内部某点故障了，通过RCP的设定，仍能保持egress出口不出错

第二阶段

![image-20210819093324323](coursera_sdn.assets/image-20210819093324323.png)

不仅了解最佳路由，也要了解其他路由情况，所以知道的信息更加全面

![image-20210819094453047](coursera_sdn.assets/image-20210819094453047.png)

之前的BGP操作默认进行了路由汇总，但是路由器不知道哪些路由器需要更加具体的路由，以保证准确性

![image-20210819094550450](coursera_sdn.assets/image-20210819094550450.png)

第三

![image-20210819094634821](coursera_sdn.assets/image-20210819094634821.png)

![image-20210819094642786](coursera_sdn.assets/image-20210819094642786.png)

总结：

![image-20210819094709407](coursera_sdn.assets/image-20210819094709407.png)

### video the4d network architecture

![image-20210819094808374](coursera_sdn.assets/image-20210819094808374.png)

![image-20210819094825152](coursera_sdn.assets/image-20210819094825152.png)

这里面的control plane和前几节提到的也有不同，主要讲的还是路由协议，跟踪拓扑的变化，计算路由，转发表等，并不会做其他更复杂的事情（比如实现网络范围内的管理调用）。

所以4D network的出现是来应对这种情况

![image-20210819095022219](coursera_sdn.assets/image-20210819095022219.png)

大量功能转入软件实现，消除了对网络供应商的依赖，加速了网络管理的创新

![image-20210819095252621](coursera_sdn.assets/image-20210819095252621.png)

#### 4d网络的终极目标

![image-20210819095341682](coursera_sdn.assets/image-20210819095341682.png)

实现网络级别的配置，而不是仅停留在路由器设备商，**最大限度减少整个网络的链路利用率**? 

![image-20210819095526373](coursera_sdn.assets/image-20210819095526373.png)

![image-20210819095537947](coursera_sdn.assets/image-20210819095537947.png)

---

![image-20210819095553447](coursera_sdn.assets/image-20210819095553447.png)

![image-20210819095708926](coursera_sdn.assets/image-20210819095708926.png)

![image-20210819095723602](coursera_sdn.assets/image-20210819095723602.png)

4d模型的关键在于好的抽象，减少了问题的复杂性

![image-20210819095823003](coursera_sdn.assets/image-20210819095823003.png)

![image-20210819095855340](coursera_sdn.assets/image-20210819095855340.png)

在这里，决策层可以同时看到流量工程与访问控制，所以在这里也可以执行负载均衡

![image-20210819100022907](coursera_sdn.assets/image-20210819100022907.png)

该论文还是期待消除控制平面的存在，但是SDN并未这样

总结：

![image-20210819100159206](coursera_sdn.assets/image-20210819100159206.png)

![image-20210819100234294](coursera_sdn.assets/image-20210819100234294.png)

》quiz 我承认，做题让人头皮发麻了。 做了30%，先继续往下看本节课程

---

[David Clark Interview | Coursera](https://www.coursera.org/learn/sdn/lecture/KoZho/david-clark-interview)

专题 访谈1 

SDN 的role 以及之后的作用

1. 数据平面和控制平面分离。 而互联网原始架构中，考虑过分离吗？ 之前考虑过但是没用：基于一种哲学，我们需要可靠，所以我们向电脑妥协。不能基于第三方，让两个pc无法交互（例如dhcp服务器的崩坏让通信无法进行）在技术发展初期，很少有公司能把这个技术发展都发展好。

虽然我们没能限制BGP在全局上的收敛速度，它明显比我们预计的时间要长。但是这些动态协议确实维持了很好的功能性

当我们没有好的工具去控制网络延迟，我们需要新的网络

我们无法在网络上得到鲁棒性的方案，可能是我们将流量路径过度具象化了

sdn魔种程度上坚挺了边界的划分（ip layer 和 layer two）

---

另一个访谈就先跳过了。



## week 3 module3：the control plane

- key terms:
  - openflow specification
  - flow table caching: 流表结果缓存在交换机中的过程，用来防止所有的数据包都要经过控制器转发
  - control channel：SDN控制器用来与SDN-capable的交换机交流的信道
  - controller overhead: 控制器接管？指的是当交换机没有某个数据包的对应流表转发项而将该包发送到控制器处理的过程。
  - openstack：云操作系统，利用网络虚拟化技术和openflow去展示资源的logical pool的抽象

列举了一系列openflow的项目列表：

[List of OpenFlow Software Projects (stanford.edu)](http://yuba.stanford.edu/~casado/of-sw.html)

---

### the control plane

![image-20210819192247059](coursera_sdn.assets/image-20210819192247059.png)

基础知识about openflow：

![image-20210819192329643](coursera_sdn.assets/image-20210819192329643.png)

控制信道的目的主要是为了更新流表，

protocol specification 规定了两个交换机组件：

第一是flow table

![image-20210819192635260](coursera_sdn.assets/image-20210819192635260.png)

交换机行为都取决于flow table中的表项

第二是secure channel：

![image-20210819192825573](coursera_sdn.assets/image-20210819192825573.png)

它决定了交换机与外部控制器的通信方式

 openflow1.0

![image-20210819193414632](coursera_sdn.assets/image-20210819193414632.png)

避免了过多流量在控制器拥塞

![image-20210819193440044](coursera_sdn.assets/image-20210819193440044.png)

当然openfflow switch也可以表现普通

![image-20210819193705312](coursera_sdn.assets/image-20210819193705312.png)

![image-20210819193728857](coursera_sdn.assets/image-20210819193728857.png)

![image-20210819193805491](coursera_sdn.assets/image-20210819193805491.png)

![image-20210819193824460](coursera_sdn.assets/image-20210819193824460.png)

dpctl指令允许我们与switch进行交互，查看流表条目，修改流表条目等

![image-20210819194208617](coursera_sdn.assets/image-20210819194208617.png)

![image-20210819194217389](coursera_sdn.assets/image-20210819194217389.png)

例如展示信息如上。

我们可以用dump-flows指令来展示流表的装载情况

![image-20210819194408291](coursera_sdn.assets/image-20210819194408291.png)



![image-20210819194655935](coursera_sdn.assets/image-20210819194655935.png)

如果装载条目为空，我们可以利用add-flow指令来增加，但是记得要把反向接口也加入其中，

1.3版本引入了很多概念，action set和group



![image-20210819194739527](coursera_sdn.assets/image-20210819194739527.png)

![image-20210819194810172](coursera_sdn.assets/image-20210819194810172.png)

这边也有一些可选项

![image-20210819194820398](coursera_sdn.assets/image-20210819194820398.png)

![image-20210819195046604](coursera_sdn.assets/image-20210819195046604.png)

总结：

![image-20210819195139425](coursera_sdn.assets/image-20210819195139425.png)

poll 轮询

### overview of sdn controller

大纲：

![image-20210819195333159](coursera_sdn.assets/image-20210819195333159.png)

![image-20210819195346545](coursera_sdn.assets/image-20210819195346545.png)

对控制器的选择有很多考量

- 编程语言，这通常影响了性能
- 学习曲线，即学习成本
- 使用人员的基础以及相应的社区支持
- 重点：
  - 南向接口数量
  - 北向接口（policy layer，如何能与应用层对接）
  - 对openstack的支持
  - 用于的场景，教育研究还是生产

![image-20210819195545877](coursera_sdn.assets/image-20210819195545877.png)

![image-20210819195619280](coursera_sdn.assets/image-20210819195619280.png)

![image-20210819195638134](coursera_sdn.assets/image-20210819195638134.png)

pox：

![image-20210819195652499](coursera_sdn.assets/image-20210819195652499.png)

简单的学习曲线，我们选择这个为例子

![image-20210819195707200](coursera_sdn.assets/image-20210819195707200.png)

ryu：

![image-20210819195836299](coursera_sdn.assets/image-20210819195836299.png)

floodlight：

![image-20210819195856989](coursera_sdn.assets/image-20210819195856989.png)

但是学习曲线还是相对较高的

opendaylight：

![image-20210819195927439](coursera_sdn.assets/image-20210819195927439.png)

![image-20210819200031792](coursera_sdn.assets/image-20210819200031792.png)

summary：

![image-20210819200309454](coursera_sdn.assets/image-20210819200309454.png)

---

### customizing SDN control(part 1 : switching)

![image-20210819200417396](coursera_sdn.assets/image-20210819200417396.png)

自定义switch的行为

回顾集线器和交换器

利用pox交换机和mininet的拓扑结构

实现两种不同的控制

![image-20210819200459192](coursera_sdn.assets/image-20210819200459192.png)

根据转发条目来学习路由

利用dpctl来查看流表。

code walkthrough

实例1topo：

![image-20210819200657335](coursera_sdn.assets/image-20210819200657335.png)

![image-20210819200731297](coursera_sdn.assets/image-20210819200731297.png)

执行上述操作完毕后，但实际上还并未实例化控制器。实例先尝试pingall操作

![image-20210819215645780](coursera_sdn.assets/image-20210819215645780.png)

![image-20210819215654547](coursera_sdn.assets/image-20210819215654547.png)

先了解一下hub

![image-20210819215717823](coursera_sdn.assets/image-20210819215717823.png)

它不存储信息，它直接将接收的流量进行发送。

这一节所用的实例代码：

[pox/hub.py at carp · noxrepo/pox (github.com)](https://github.com/noxrepo/pox/blob/carp/pox/forwarding/hub.py)

```python
# Copyright 2012 James McCauley
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

"""
Turns your complex OpenFlow switches into stupid hubs.
"""

from pox.core import core
import pox.openflow.libopenflow_01 as of
from pox.lib.util import dpidToStr

log = core.getLogger()


def _handle_ConnectionUp (event):
  msg = of.ofp_flow_mod()
  msg.actions.append(of.ofp_action_output(port = of.OFPP_FLOOD))
  event.connection.send(msg)
#得到消息后创建消息，然后再反馈回switch
  log.info("Hubifying %s", dpidToStr(event.dpid))

def launch ():
  core.openflow.addListenerByName("ConnectionUp", _handle_ConnectionUp)

  log.info("Hub running.")
```

但是仅仅用pip install指令的pox 好像没法像这个实例一样导入core

[Installing POX — POX Manual Current documentation (noxrepo.github.io)](https://noxrepo.github.io/pox-doc/html/)

查看pox的安装文档

![image-20210819222719750](coursera_sdn.assets/image-20210819222719750.png)

本实验出的结果与展示的并不一致

![image-20210819222746315](coursera_sdn.assets/image-20210819222746315.png)

