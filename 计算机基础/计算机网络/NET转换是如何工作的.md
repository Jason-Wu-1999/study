作者：网工Fox
链接：https://www.zhihu.com/question/31332694/answer/1917791148
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



IP 地址中预留了 3 个**私有地址**网段，在私有网络内，可以任意使用。
![img](https://pic3.zhimg.com/50/v2-77e65904bd9b38531d6d023164eb8fdf_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-77e65904bd9b38531d6d023164eb8fdf_720w.jpg?source=1940ef5c)
其余的 IP 地址可以在互联网上使用，由 IANA 统一管理，称为**公网地址**。
![img](https://pic2.zhimg.com/50/v2-e0c227df1503e89785eeb8a03be36562_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-e0c227df1503e89785eeb8a03be36562_720w.jpg?source=1940ef5c)
NAT 解决了 IPv4 地址不够用的问题，另外 NAT 屏蔽了私网用户真实地址，提高了私网用户的安全性。典型的 **NAT 组网**模型，网络通常是被划分为私网和公网两部分，各自使用独立的地址空间。私网使用私有地址 10.0.0.0/24 ，而公网使用公网地址。为了让主机 A 和 B 访问互联网上的服务器 Server ，需要在网络边界部署一台 NAT 设备用于执行地址转换。NAT 设备通常是**路由器**或**[防火墙](https://www.zhihu.com/search?q=防火墙&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1917791148})**。
![img](https://pic3.zhimg.com/50/v2-ffed54cbf18a54ecbc04c95cb04cced3_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-ffed54cbf18a54ecbc04c95cb04cced3_720w.jpg?source=1940ef5c)
基本 NAT**基本 NAT** 是最简单的一种地址转换方式，它只对数据包的 IP 层参数进行转换，它可分为**静态 NAT** 和**动态 NAT** 。**静态 NAT** 是公网 IP 地址和私有 IP 地址有一对一的关系，一个公网 IP 地址对应一个私有 IP 地址，建立和维护一张静态地址映射表。**动态 NAT** 是公网 IP 地址和私有 IP 地址有一对多的关系，同一个[公网](https://www.zhihu.com/search?q=公网&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1917791148}) IP 地址分配给不同的私网用户使用，使用时间必须错开。它包含一个公有 IP 地址池和一张动态地址映射表。**举个动态 NAT 栗子**私网主机 **A**（ 10.0.0.1 ）需要访问公网的服务器 **Server**（ 61.144.249.229 ），在路由器 RT 上配置 **NAT** ，地址池为 219.134.180.11 ~ 219.134.180.20 ，地址转换过程如下：
![img](https://pic3.zhimg.com/50/v2-91f5deefa68b224931098dfe28e3791d_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-91f5deefa68b224931098dfe28e3791d_720w.jpg?source=1940ef5c)
**A** 向 Server 发送报文，网关是 10.0.0.254 ，源地址是 10.0.0.1 ，目的地址是 61.144.249.229 。
![img](https://pic2.zhimg.com/50/v2-bbcaa5a03b2eb868889cb74d4516460c_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-bbcaa5a03b2eb868889cb74d4516460c_720w.jpg?source=1940ef5c)
**RT** 收到 IP 报文后，查找[路由表](https://www.zhihu.com/search?q=路由表&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1917791148})，将 IP 报文转发至出接口，由于出接口上配置了 NAT ，因此 RT 需要将源地址 10.0.0.1 转换为公网地址。
![img](https://pic2.zhimg.com/50/v2-f3736378c423302b81ad889642ceb587_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-f3736378c423302b81ad889642ceb587_720w.jpg?source=1940ef5c)
**RT** 从地址池中查找第一个可用的公网地址 219.134.180.11 ，用这个地址替换数据包的源地址，转换后的数据包源地址为 219.134.180.11 ，目的地址不变。同时 RT 在自己的 NAT 表中添加一个表项，记录私有地址 10.0.0.1 到 公网地址 219.134.180.11 的映射。RT 再将报文转发给目的地址 61.144.249.229 。
![img](https://pic2.zhimg.com/50/v2-dc2a84984269febe7eb626b5f4a69861_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-dc2a84984269febe7eb626b5f4a69861_720w.jpg?source=1940ef5c)
**Server** 收到报文后做相应处理。
**Server** 发送回应报文，报文的源地址是 61.144.249.229 ，目的地址是 219.134.180.11 。

![img](https://pic3.zhimg.com/50/v2-df85bbcd19b278f90cc31a8da2b2193f_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-df85bbcd19b278f90cc31a8da2b2193f_720w.jpg?source=1940ef5c)
**RT** 收到报文，发现报文的目的地址 219.134.180.11 在 NAT 地址池内，于是检查 NAT 表，找到对应表项后，使用私有地址 10.0.0.1 替换公网地址 219.134.180.11，转换后的报文源地址不变，目的地址为 10.0.0.1 。RT 在将报文转发给 A 。
![img](https://pic1.zhimg.com/50/v2-bbe3420b9e486a27fc9fd746217d2d31_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-bbe3420b9e486a27fc9fd746217d2d31_720w.jpg?source=1940ef5c)
**A** 收到报文，地址转换过程结束。
![img](https://pic3.zhimg.com/50/v2-a95551a727491fcd9811c359e095b3e7_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-a95551a727491fcd9811c359e095b3e7_720w.jpg?source=1940ef5c)
如果 **B** 也要访问 Server ，则 RT 会从地址池中分配另一个可用公网地址 219.134.180.12 ，并在 NAT 表中添加一个相应的表项，记录 B 的私有地址 10.0.0.2 到公网地址 219.134.180.12 的映射关系。
![img](https://pic3.zhimg.com/50/v2-47e97d84330daa09116ec8ddba259c43_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-47e97d84330daa09116ec8ddba259c43_720w.jpg?source=1940ef5c)
NAPT在基础 NAT 中，私有地址和公网地址存在一对一地址转换的对应关系，即一个公网地址同时只能分配给一个私有地址。它只解决了公网和私网的通信问题，并没有解决公网地址不足的问题。
![img](https://pic3.zhimg.com/50/v2-75c232c35ed94e9c7b85eb26c9f24270_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-75c232c35ed94e9c7b85eb26c9f24270_720w.jpg?source=1940ef5c)
**NAPT**（ Network Address Port Translation ）对数据包的 IP 地址、协议类型、传输层端口号同时进行转换，可以明显提高公网 IP 地址的利用率。
![img](https://pic1.zhimg.com/50/v2-34da92151023c595b9bbad1e180fde15_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-34da92151023c595b9bbad1e180fde15_720w.jpg?source=1940ef5c)
**举个栗子**私网主机 **A**（ 10.0.0.1 ）需要访问公网的服务器 **Server** 的 WWW 服务（ 61.144.249.229 ），在路由器 RT 上配置 NAPT ，地址池为 219.134.180.11 ~ 219.134.180.20 ，地址转换过程如下：**A** 向 Server 发送报文，网关是 RT（ 10.0.0.254 ），源地址和端口是 10.0.0.1:1024 ，目的地址和端口是 61.144.249.229:80 。
![img](https://pic3.zhimg.com/50/v2-2d7294a46a905979c211b35b5d2f8f70_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-2d7294a46a905979c211b35b5d2f8f70_720w.jpg?source=1940ef5c)
**RT** 收到 IP 报文后，查找路由表，将 IP 报文转发至出接口，由于出接口上配置了 NAPT ，因此 RT 需要将源地址 10.0.0.1:1024 转换为公网地址和端口。
**RT** 从地址池中查找第一个可用的公网地址 219.134.180.11 ，用这个地址替换数据包的源地址，并查找这个公网地址的一个可用端口，例如 2001 ，用这个端口替换源端口。转换后的数据包源地址为 219.134.180.11:2001 ，目的地址和端口不变。同时 RT 在自己的 NAT 表中添加一个表项，记录私有地址 10.0.0.1:1024 到 公网地址 219.134.180.11:2001 的映射。RT 再将报文转发给目的地址 61.144.249.229 。

![img](https://pic2.zhimg.com/50/v2-51c40f6df15710213bedcb6d7e21976e_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-51c40f6df15710213bedcb6d7e21976e_720w.jpg?source=1940ef5c)
**Server** 收到报文后做相应处理。
**Server** 发送回应报文，报文的源地址是 61.144.249.229:80 ，目的地址是 219.134.180.11:2001 。

![img](https://pic2.zhimg.com/50/v2-a85d3f82a7fbcb1a251a8e3e77502175_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-a85d3f82a7fbcb1a251a8e3e77502175_720w.jpg?source=1940ef5c)
**RT** 收到报文，发现报文的目的地址在 NAT 地址池内，于是检查 NAT 表，找到对应表项后，使用私有地址和端口 10.0.0.1:1024 替换公网地址 219.134.180.11:2001，转换后的报文源地址和端口不变，目的地址和端口为 10.0.0.1:1024 。RT 再将报文转发给 A 。
![img](https://pic2.zhimg.com/50/v2-020cdbe23f009791bd7542ea20b05e1a_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-020cdbe23f009791bd7542ea20b05e1a_720w.jpg?source=1940ef5c)
**A** 收到报文，地址转换过程结束。如果 **B** 也要访问 Server ，则 RT 会从地址池中分配同一个公网地址 219.134.180.11 ，但分配另一个端口 3001 ，并在 NAT 表中添加一个相应的表项，记录 B 的私有地址 10.0.0.2:1024 到公网地址 219.134.180.12:3001 的映射关系。
![img](https://pica.zhimg.com/50/v2-a42be2a6f28e2e8c45b7b7f26d82cbb3_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-a42be2a6f28e2e8c45b7b7f26d82cbb3_720w.jpg?source=1940ef5c)
Easy IP在标准的 NAPT 配置中需要创建公网地址池，也就是必须先知道公网 IP 地址的范围。而在拨号接入的上网方式中，公网 IP 地址是由运营商动态分配的，无法事先确定，标准的 NAPT 无法做地址转换。要解决这个问题，就要使用 **Easy IP** 。**Easy IP** 又称为基于接口的地址转换。在地址转换时，Easy IP 的工作原理与 NAPT 相同，对数据包的 IP 地址、协议类型、传输层端口号同时进行转换。但 Easy IP 直接使用公网接口的 IP 地址作为转换后的源地址。Easy IP 适用于**拨号接入**互联网，动态获取公网 IP 地址的场合。
![img](https://pic1.zhimg.com/50/v2-0835f8e0f92bd103b3b7a77d18dfd7a8_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-0835f8e0f92bd103b3b7a77d18dfd7a8_720w.jpg?source=1940ef5c)
Easy IP 无需配置地址池，只需要配置一个 **ACL**（[访问控制列表](https://www.zhihu.com/search?q=访问控制列表&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1917791148})），用来指定需要进行 NAT 转换的私有 IP 地址范围。NAT Server从基本 NAT 和 NAPT 的工作原理可知，NAT 表项由私网主机主动向公网主机发起访问而生成，公网主机无法主动向私网主机发起连接。因此 NAT 隐藏了内部网络结构，具有屏蔽主机的作用。但是在实际应用中，内网网络可能需要对外提供服务，例如 Web 服务，常规的 NAT 就无法满足需求了。为了满足公网用户访问私网内部服务器的需求，需要使用 **NAT Server** 功能，将私网地址和端口静态映射成公网地址和端口，供公网用户访问。
![img](https://pic3.zhimg.com/50/v2-ee948ed1149e4ae092cd2eb83fb47079_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-ee948ed1149e4ae092cd2eb83fb47079_720w.jpg?source=1940ef5c)
**举个栗子****A** 的私网地址为 10.0.0.1 ，端口 8080 提供 Web 服务，在对公网提供 Web 服务时，要求端口号为 80 。在 NAT 设备上启动 NAT Server 功能，将私网 IP 地址和端口 10.0.0.1:8080 映射成公网 IP 地址和端口 219.134.180.11:80 ，这样**公网主机 C** 就可以通过 219.134.180.11:80 访问 A 的 Web 服务。NAT ALG基本 NAT 和 NAPT 只能识别并修改 IP 报文中的 IP 地址和端口号信息，无法修改报文内携带的信息，因此对于一些 IP 报文内携带网络信息的协议，例如 FTP 、DNS 、SIP 、H.323 等，是无法正确转换的。**ALG** 能够识别应用层协议内的网络信息，在转换 IP 地址和端口号时，也会对应用层数据中的网络信息进行正确的转换。**举个栗子：ALG 处理 FTP 的 Active 模式****FTP** 是一种基于 TCP 的协议，用于在客户端和服务器间传输文件。FTP 协议工作时建立 2 个通道：**Control 通道**和 **Data 通道**。Control 用于传输 FTP 控制信息，Data 通道用于传输文件数据。私网 **A**（ 10.0.0.1 ）访问公网 **Server**（ 61.144.249.229 ）的 FTP 服务，在 RT 上配置 NAPT，地址池为 219.134.180.11 ~ 219.134.180.20 ，地址转换过程如下：
![img](https://pic3.zhimg.com/50/v2-625236e44506487bf716ce0d8c8c3db8_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-625236e44506487bf716ce0d8c8c3db8_720w.jpg?source=1940ef5c)
**A** 发送到 Server 的 FTP Control 通道建立请求，报文源地址和端口为 10.0.0.1:1024 ，目的地址和端口为 61.144.249.229:21 ，携带数据是 “ IP = 10.0.0.1 port=5001 ”，即告诉 Server 自己使用 TCP 端口 5001 传输 Data。
![img](https://pic2.zhimg.com/50/v2-402bdc508b0ce7e218f00024bb3ca886_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-402bdc508b0ce7e218f00024bb3ca886_720w.jpg?source=1940ef5c)
**RT** 收到报文，建立 10.0.0.1:1024 到 219.134.180.11:2001 的映射关系，转换源 IP 地址和 TCP 端口。根据目的端口 21 ，RT 识别出这是一个 FTP 报文，因此还要检查应用层数据，发现原始数据为 “ IP = 10.0.0.1 port=5001 ”，于是为 Data 通道 10.0.0.1:5001 建立第二个映射关系：10.0.0.1:5001 到 219.134.180.11:2002 ，转换后的报文源地址和端口为 219.134.180.11:2001 ，目的地址和端口不变，携带数据为 “ IP = 219.134.180.11 port=2002 ”。
![img](https://pica.zhimg.com/50/v2-fbddff3f2a5728e33919b49b9e05e67b_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-fbddff3f2a5728e33919b49b9e05e67b_720w.jpg?source=1940ef5c)
**Server** 收到报文，向 A 回应 command okay 报文，FTP Control 通道建立成功。同时 Server 根据应用层数据确定 A 的 Data 通道网络参数为 219.134.180.11:2002 。
**A** 需要从 FTP 服务器下载文件，于是发起文件请求报文。Server 收到请求后，发起 Data 通道建立请求，IP 报文的源地址和端口为 61.144.249.229:20 ，目的地址和端口为 219.134.180.11:2002，并携带 FTP 数据。

![img](https://pic3.zhimg.com/50/v2-9b4e1d736f66e0d557fbfc17bfb7c552_720w.jpg?source=1940ef5c)![img](https://pic3.zhimg.com/80/v2-9b4e1d736f66e0d557fbfc17bfb7c552_720w.jpg?source=1940ef5c)

[发布于 2021-06-01 22:50](http://www.zhihu.com/question/31332694/answer/1917791148)