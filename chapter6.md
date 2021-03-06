# 局域网

## ARP协议

### MAC地址

* 32位IP地址：
  * 接口的网络层地址
  * 用于标识网络层分组，支持分组转发
* MAC地址：
  * 作用：用于局域网内标识一个帧从哪个接口发出，到达哪个物理相连的其他接口
  * 48位MAC地址（用于大部分LANs），固化在网卡的ROM中，有时也可以软件设置
* 局域网中的每一块网卡都有一个唯一的MAC地址
* MAC地址由IEEE统一管理与分配
* 网卡生产商购买MAC地址空间（前24比特）
* 类比：
  * MAC地址：身份证号
  * IP地址：邮政地址
* MAC地址是“平面”地址：可“携带”
  * 可以从一个LAN移到另一个LAN
* IP地址是层次地址：不可“携带”
  * IP地址依赖于结点连接到哪个子网

### ARP：地址解析协议

* ARP表：LAN中的每个IP结点维护一个表
  * 存储某些LAN结点的IP/MAC地址映射关系：
    * <IP:MAC:TTL>
  * TTL：经过这个时间以后该映射关系会被遗弃

### ARP：同一个局域网内

* A想要给同一局域网内的B发送数据报
  * B的MAC地址不在A的ARP表中。
* A广播ARP查询分组，其中包含B的IP地址
  * 目的MAC地址 = FF-FF-FF-FF-FF-FF
  * LAN中所有结点都会接收ARP查询
* B接收ARP查询分组，IP地址匹配成功，向A应答B的MAC地址
  * 利用单播帧向A发送应答
* A在其ARP表中，缓存B的IP-MAC地址对，直至超时
  * 超时后，再次刷新
* ARP是“即插即用”协议：
  * 结点自主创建ARP表，无需干预

### 寻址：从一个LAN路由至另一个LAN

* 通信过程：A通过路由器R向B发送数据报
  * 关注寻址：IP地址（数据报中）和MAC地址（帧中）
  * 假设A知道B的IP地址
  * 假设A知道第一跳路由器R接口IP地址
  * 假设A知道第一跳路由器R接口MAC地址
* A构造IP数据报，其中IP地址是A的IP地址，目的IP地址是B的IP地址
* A构造链路层帧，其中源MAC地址是A的MAC地址，目的MAC地址是R接口的MAC地址，封装A到B的IP数据报
* 帧从A发送至R
* R接收帧，提取IP数据报，传递上层IP协议
* R转发IP数据报（源和目的IP地址不变）
* R创建链路层帧，其中源MAC地址是R接口的MAC地址，目的MAC地址是B的MAC地址，封装A到B的IP数据报

## 以太网

### 以太网（ETHERNET）

* “统治地位”的有线LAN技术
* 造价低廉
* 应用最广泛的LAN技术
* 比令牌局域网和ATM等，简单、便宜
* 满足网络速率需求：10Mbps-10Gbps

### 以太网：物理拓扑

* 总线：上世纪90年代中期前流行
  * 所有结点在同一冲突域（可能彼此冲突）
* 星型：目前主流网络拓扑
  * 中心交换机
  * 每个节点一个单独冲突域（结点间彼此不冲突）

### 以太网：不可靠、无连接服务

* 无连接：发送帧的网卡与接收帧网卡间没有“握手”过程
* 不可靠：接收网卡不向发送网卡进行确认
  * 差错帧直接丢弃，丢弃帧中的数据恢复依靠高层协议，否则，发生数据丢失
* 以太网的MAC协议：采用二进制指数退避算法的CSMA/CD

### 以太网CSMA/CD算法

* NIC从网络层接收数据报，创建数据帧
* 监听信道：
  * 如果NIC监听到信道空闲，则开始发送帧
  * 如果NIC监听到信道忙，则一直等待到信道空闲，然后发送帧
* NIC发送完整个帧，而没有检测到其他结点的数据发送，则NIC确认帧发送成功
* 如果NIC检测到其他结点传输数据，则中止发送并发送堵塞信号
* 中止发送后，NIC进入二进制指数退避：
  * 第m次连续冲突后：
    * 取n=Max(m, 10)
    * NIC从{0,1,2,...,2^n-1}中随机选择一个数K
    * NIC等待512*k比特的传输延迟时间，再返回第2步
  * 连续冲突次数越多，平均等待时间越长

### 以太网帧结构

* 发送端网卡将IP数据报（或其他网络层协议分组）封装到以太网帧中：
  * 前导码（8B）：
    * 7个字节的10101010，第8字节为10101011
    * 用于发送端与接收端的时钟同步
* 目的MAC地址、源MAC地址（各6B）
* 类型（2B）：指示帧中封装的是哪种高层协议的分组
* 数据（46-1500B）：指上层协议载荷
* CRC（4B）：循环冗余校验码
  * 丢弃差错帧

### 802.3以太网标准：链路与物理层

* 许多不同的以太网标准
  * 相同MAC协议和帧格式
  * 不同速率：2Mbps，10Mbps，100Mbps，1Gbps，10Gbps
  * 不同物理介质：光纤，线缆

## 交换机

### 以太网交换机

* 链路层设备
  * 存储-转发以太网帧
  * 检验到达帧的目的MAC地址，选择性向一个或多个输出链路转发帧
  * 利用CSMA/CD访问链路，发送帧
* 透明
  * 主机感知不到交换机存在
* 即插即用
* 自学习
  * 交换机无需配置

### 交换机：多端口间同时传输

* 主机利用独享链路直接连接交换机
* 交换机缓存帧
* 交换机在每段链路上利用CSMA/CD收发帧，但无冲突，且可以全双工
  * 每段链路一个独立的冲突域
* 交换：A-A'与B-B'的传输可以同时进行，没有冲突

### 交换机转发表：交换表

* 每个交换机有一个交换表，每个入口：
  * 主机的MAC地址，到达主机的接口，时间戳
  * 看起来像路由表

### 交换机：自学习

* 交换机通过自学习，获知到达主机的接口信息
  * 当收到帧时，交换机“学习”到发送帧的主机（通过帧的源MAC地址），位于收到该帧的接口所连接的LAN网段
  * 将发送主机MAC地址/接口信息记录到交换表中

### 交换机：帧过滤/转发

* 当交换机收到帧：
  * 记录帧的源MAC地址与输入链路接口
  * 利用目的MAC地址检索交换表
  * 若在交换表中检索到与目的MAC地址匹配的入口
    * 若目的主机位于收到帧的网段，则丢弃
    * 否则将帧转发到该入口指向的接口
  * 若没找到，则泛洪，向除收到该帧的接口之外的所有接口转发

### 交换机 vs 路由器

* 两者均为存储-转发设备：
  * 路由器：网络层设备（检测网络层分组首部）
  * 交换机：链路层设备（检测链路层的首部）
* 两者均使用转发表：
  * 路由器：利用路由算法（路由协议）计算（设置），依据IP地址
  * 交换机：利用自学习、泛洪构建转发表，依据MAC地址

## 虚拟局域网（VLAN）

### VLANs

* 支持VLAN划分的交换机，可以在一个物理LAN架构上配置、定义多个VLAN
* 基于端口的VLAN：分组交换机端口（通过交换机管理软件），于是，单一的物理交换机就像多个虚拟交换机一样运行

### 基于端口的VLAN

* 流量隔离：
  * 也可以基于MAC地址定义VLAN，而不是交换端口
* 动态成员：端口可以动态分配给不同VLAN
* 在VLAN间转发：通过路由（就像在独立的交换机之间）
  * 实践中，厂家会将交换机与路由器集成在一起

### 跨越多交换机的VLAN

* 多线缆连接
  * 每个线缆连接一个VLAN
* 中继端口：在跨越多个物理交换机定义的VLAN承载帧
  * 为多VLAN转发802.1帧容易产生歧义（必须携带VLAN ID信息）
  * 802.1q协议为经过中继端口转发的帧增加/去除额外的首部域

## PPP协议

### 点对点数据链路控制

* 一个发送端，一个接收端，一套链路：比广播链路容易
  * 无需介质访问控制
  * 无需明确的MAC寻址
  * eg：拨号链路，ISDN链路
* 常见的点对点数据链路控制协议：
  * HDLC：高级数据链路控制
  * PPP：点对点协议

### PPP设计需求

* 组帧：将网络层数据报封装到数据链路层帧中
  * 可以同时承载任何网络层协议分组（不仅IP数据报）
  * 可以向上层实现分用（多路分解）
* 比特透明传输：数据域必须支持承载任何比特模式
* 差错检测：（无纠正）
* 连接活性检测：检测、并向网络层通知链路失效
* 网络层地址协商：端结点可以配置/学习彼此网络地址

### PPP无需支持的功能

* 无需差错纠正/恢复
* 无需流量控制
* 不存在乱序交付
* 无需支持多点链路
* 差错恢复、流量控制等由高层协议处理

### PPP数据帧

* 标识：定界符（01111110），首尾各一个
* 地址：无效（仅仅是一个选项）（0x11）
* 控制：无效；未来可能的多种控制域
* 协议：上层协议（1B or 2B）
* 信息：上层协议分组
* 校验：CRC，用于差错检测（2B or 4B）

### 字节填充

* “数据透明传输”需求：数据域必须允许包含标识模式<01111110>
  * 发送端：在数据中的<01111110>和<01111101>字节前添加额外的字节<01111101>
  * 接收端：
    * 单个字节<01111101>表示一个填充字节
    * 连续两个字节<01111101>:丢弃第1个，第二个作为数据接收
    * 单个字节<01111110>则为定界符

### PPP数据控制协议

* 在交换网络层数据之前，PPP数据链路两个端必须：
  * 配置PPP链路
    * 最大帧长
    * 身份认证
  * 学习/配置网络层信息
    * 对于IP协议：通过交换IPCP协议报文，完成IP地址等相关信息配置

## 802.11无线局域网简介

### IEEE 802.11无线局域网

* 802.11b
  * 2.4-2.5GHz免费频段
  * 最高速率：11Mbps
  * 物理层采用直接序列扩频技术
    * 所有主机使用相同码片序列
* 802.11a
  * 5-6GHz
  * 最高速率：54Mbps
* 802.11g
  * 2.4-2.5GHz
  * 最高速率：54Mbps
* 802.11n：多天线（MIMO）
  * 2.4-2.5GHz
  * 最高速率：600Mbps
* 均使用CSMA/CA多路访问控制协议
* 均有基础设施（基站）网络模式和特定网（自组网）网络模式

### IEEE 802.11体系结构

* 无线主机与基站通信
  * 基站 = 访问点（AP）
* 基本服务集BSS，也称为单元
  * 基础设施网络模式
    * 无线主机
    * AP：基站
  * 自组网模式
    * 只有主机

### 802.11：信道与AP关联

* 802.11b：2.4GHz-2.485GHz频谱划分为11个不同频率的信道
  * 每个AP选择一个频率（信道）
  * 存在干扰可能：相邻的AP可能选择相同的信道
* 主机：必须与某个AP关联
  * 扫描信道，监听包含AP名称（服务集标识符-SSID）和MAC地址的信标帧
  * 选择一个AP进行关联
  * 可能需要进行身份认证
  * 典型情形：运行DHCP获取IP地址等信息

### 802.11AP关联：被动扫描与主动扫描

* 被动扫描：
  * 各AP发送信标帧
  * 主机向选择的AP发送关联请求帧
  * AP向主机发送关联响应帧
* 主动扫描：
  * 主机主动广播探测请求帧
  * AP发送探测响应帧
  * 主机向选择的AP发送关联请求帧
  * AP向主机发送关联响应帧

### 802.11：多路访问控制

* 避免冲突：2个结点同时传输
* 802.11：CSMA-发送数据前监听信道
  * 避免与正在进行传输的其他结点冲突
* 802.11：不能像CSMA/CD那样，边发送、边检测冲突
  * 无线信道很难实现
  * 无法侦听到所有可能的冲突：隐藏站、信号衰落
  * 目标：避免冲突-CSMA/CA

### IEEE 802.11 MAC协议：CSMA/CA

* 802.11 sender
  * 监听到信道空闲了DIFS时间则发送整个帧（无同时检测冲突，即CD）
  * 监听到信道忙则开始随机退避计时，当信道空闲时，计时器倒计时，当计时器超时时，发送帧如果没有收到ACK则增加随机退避间隔时间
* 802.11 receiver
  * 正确接收帧
    * 延迟SIFS时间后，向发送端发送ACK（由于存在隐藏站问题）
* 基本思想：允许发送端“预约”信道，而不是随机发送数据帧，从而避免长数据帧的冲突
* 发送端首先利用CSMA向BS发送一个很短的RTS帧
  * RTS帧仍然可能彼此冲突（但RTS帧很短）
* BS广播一个CTS帧作为对RTS的响应
* CTS帧可以被所有结点接收
  * 消除隐藏站影响
  * 发送端可以发送数据帧
  * 其他结点推迟发送
* 利用很小的预约帧彻底避免了数据帧冲突
