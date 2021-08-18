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