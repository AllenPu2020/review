# 网络通信

### POST和GET区别

- 1、数据的位置
    - GET请求，请求的数据会附加在URL之后，以 `?` 分割URL和传输数据，多个参数用 `&` 连接。
        - URL的编码格式采用的是ASCII编码，而不是unicode，即是说所有的非ASCII字符都要编码之后再传输。
    - POST请求：POST请求会把请求的数据放置在HTTP请求包的包体中。上面的item=bandsaw就是实际的传输数据。
    - 因此，GET请求的数据会暴露在地址栏中，而POST请求则不会。

- 2、传输数据的大小
    - 在HTTP规范中，没有对URL的长度和传输的数据大小进行限制。
    - 但是在实际开发过程中，对于GET，特定的浏览器和服务器对URL的长度有限制。
    - 因此，在使用GET请求时，传输数据会受到URL长度的限制。
    - 对于POST，由于不是URL传值，理论上是不会受限制的，但是实际上各个服务器会规定对POST提交数据大小进行限制，Apache、IIS都有各自的配置。
- 3、安全性
    - POST的安全性比GET的高。

        > 这里的安全是指真正的安全，而不同于安全方法中的安全，那种安全仅仅是不修改服务器的数据。
    
    - 比如，在进行登录操作，通过GET请求，用户名和密码都会暴露再URL上，因为登录页面有可能被浏览器缓存以及其他人查看浏览器的历史记录的原因，此时的用户名和密码就很容易被他人拿到了。
    - 除此之外，GET请求提交的数据还可能会造成`Cross-site request frogery`攻击。

### 三次握手和四次挥手

- **三次握手**
建立连接的过程是利用客户端服务器模式，假设主机A为客户端，主机B为服务器端。
    - （1）TCP的三次握手过程：主机A向B发送连接请求；主机B对收到的主机A的报文段进行确认；主机A再次对主机B的确认进行确认。
    - （2）采用三次握手是为了防止失效的连接请求报文段突然又传送到主机B，因而产生错误。
失效的连接请求报文段是指：主机A发出的连接请求没有收到主机B的确认，于是经过一段时间后，主机A又重新向主机B发送连接请求，且建立成功，顺序完成数据传输。
考虑这样一种特殊情况，主机A第一次发送的连接请求并没有丢失，而是因为网络节点导致延迟达到主机B，主机B以为是主机A又发起的新连接，于是主机B同意连接，并向主机A发回确认，但是此时主机A根本不会理会，主机B就一直在等待主机A发送数据，导致主机B的资源浪费。
    - （3）采用两次握手不行，原因就是上面说的失效的连接请求的特殊情况，因此采用三次握手刚刚好，两次可能出现失效，四次甚至更多次则没必要，反而复杂了

- **四次挥手**
    - 先由客户端向服务器端发送一个FIN，请求关闭数据传输。
    - 当服务器接收到客户端的FIN时，向客户端发送一个ACK，其中ack的值等于FIN+SEQ
    - 然后服务器向客户端发送一个FIN，告诉客户端应用程序关闭。
    - 当客户端收到服务器端的FIN是，回复一个ACK给服务器端。其中ack的值等于FIN+SEQ

- 名词解释
    - SYN: Synchronize Sequence Numbers 同步序列编号
    - ACK: Acknowledgement Number 确认编号
    - FIN: Finish 结束标志

![三次握手&四次挥手图解](http://p4emt3ysm.bkt.clouddn.com/tcp.png)

### 多线程有哪些模块
- 在Python中可使用的多线程模块主要有两个，`thread` 和 `threading` 模块。


### 什么是URL
- URL，即统一资源定位符，也就是我们说的网址
- 统一资源定位符是对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址
- 互联网上的每个文件都有一个唯一的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。


### 网络传输层

层|协议
-|-
应用层 | http ftp dns nfs
传输层 | tcp udp
网络层 | ip icmp igmp
链路层 | data link
物理层 | media

### 设置ip和掩码

```bash
ifconfig eth0 192.168.6.8 netmask 255.255.255.0
```

### 设置网关

```bash
route add default gw 192.168.6.1
```

### `2MSL`

- `2MSL`即两倍的`MSL`，TCP的`TIME_WAIT`状态也称为`2MSL`等待状态，
- 当TCP的一端发起主动关闭，在发出最后一个ACK包后，即第3次握手完成后发送了第四次握手的ACK包后就进入了`TIME_WAIT`状态
- 必须在此状态上停留两倍的MSL时间，等待2MSL时间主要目的是怕最后一个ACK包对方没收到
- 那么对方在超时后将重发第三次握手的`FIN`包，主动关闭端接到重发的FIN包后可以再发一个ACK应答包
- 在`TIME_WAIT`状态时两端的端口不能使用，要等到`2MSL`时间结束才可继续使用。
- 当连接处于2MSL等待阶段时任何迟到的报文段都将被丢弃。
- 在实际应用中可以通过设置`SO_REUSEADDR`选项达到不必等待2MSL时间结束再使用此端口。

### `TTL`，`MSL`，`RTT`

- `TTL`：**生存时间** (`timetolive`)
    - 这个生存时间是由源主机设置初始值但不是生存的具体时间，而是存储了一个ip数据报可以经过的最大路由数
    - 每经过一个处理他的路由器此值就减1，当此值为0则数据报将被丢弃
    - 同时发送`ICMP`报文通知源主机。`RFC793`中规定`MSL`为2分钟，实际应用中常用的是30秒，1分钟和2分钟等
    - `TTL`与`MSL`是有关系的但不是简单的相等的关系，`MSL`要大于等于`TTL`

- `MSL`：**报文最大生存时间** (`Maximum Segment Lifetime`)
    - 它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。

- `RTT`：**客户到服务器往返所花时间** (`round-trip time`)
    - TCP含有动态估算RTT的算法。
    - TCP还持续估算一个给定连接的RTT，这是因为RTT受网络传输拥塞程序的变化而变化。

### 创建一个简单tcp服务器需要的流程

No.|步骤|方法
-|-|-
1 | 创建套接字 | tcp_server_socket = socket(AF_INET, SOCK_STREAM)
2 | 设置服务器地址 | server_addr = ('127.0.0.1', 6666)
3 | 绑定地址 | tcp_server_socket.bind(server_addr)
4 | 监听 | tcp_server_socket.listen(128)
5 | 生成客户端套接字 | client_socket, client_addr = tcp_server_socket.accept()
6 | 接受内容 | recv_msg = client_socket.recv(1024).decode('gbk')
7 | 发送内容 | client_socket.send(send_msg.encode('gbk'))
8 | 关闭套接字 | client_socket.close() tcp_server_socket.close()


### 网络协议概述

- 应用层

Abbr.   | Protocol                              | Description
-|-|-
 FTP    | File Transfer Protocol                | 文件传输协议
 HTTP   | Hyper Text Transfer Protocol          | 超文本传输协议
 SMTP   | Simple Mail Transfer Protocol         | 简单邮件传输协议
 POP3   | Post Office Protocol                  | 邮局协议
 DNS    | Domain Name System                    | 域名系统

- 传输层

Abbr.   | Protocol                              | Description
-|-|-
 TCP    | Transmission Control Potocol          | 传输控制协议
 UDP    | User Datagram Potocol                 | 用户数据报协议

- 网络层

Abbr.   | Protocol                              | Description
-|-|-
 IP     | Internet Protocol                     | 网络协议
 ARP    | Adderss Resolution Protocol           | 地址解析协议
 ICMP   | Internet Control Message Protocol     | 控制报文协议
 HDLC   | High Data Link Control                | 高级数据链路控制

- 数据链路层

Abbr.   | Protocol                              | Description
-|-|-
 SLIP   | Serial Line Internet Protocol         | 串行线路网际协议
 PPP    |                                       | 点对点协议

- 物理层
    - 放大/再生若的信号，在两个电缆段之间复制每一个比特

### HTTP/HTTPS的区别，应用场合
- HTTP和HTTPS的区别
    - 端口不一样,前者是80,后者是443
    - http是超文本传输协议，信息是明文传输，连接很简单，是无状态的
    - https协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比http协议安全
    - https协议需要到ca申请证书，需要交费

- 应用场合
    - http:适合于对传输速度要求高，安全性要求不是很高，且需要快速开发的应用。如web应用，小的手机游戏等.
    - https:用于任何场景！

### HTTPS优点和缺点
- 优点：
    - （1）使用HTTPS协议可认证用户和服务器，确保数据发送到正确的客户机和服务器
    - （2）HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比http协议安全，可防止数据在传输过程中被窃取、改变，确保数据的完整性
    - （3）HTTPS是现行架构下最安全的解决方案，虽然不是绝对安全，但它大幅增加了中间人攻击的成本
    - （4）谷歌曾在2014年8月份调整搜索引擎算法，并称“比起同等HTTP网站，采用HTTPS加密的网站在搜索结果中的排名将会更高”

- 缺点：
    - （1）HTTPS协议握手阶段比较费时，会使页面的加载时间延长近50%，增加10%到20%的耗电
    - （2）HTTPS连接缓存不如HTTP高效，会增加数据开销和功耗，甚至已有的安全措施也会因此而受到影响
    - （3）SSL证书需要钱，功能越强大的证书费用越高，个人网站、小网站没有必要一般不会用
    - （4）SSL证书通常需要绑定IP，不能在同一IP上绑定多个域名，IPv4资源不可能支撑这个消耗
    - （5）HTTPS协议的加密范围也比较有限，在黑客攻击、拒绝服务攻击、服务器劫持等方面几乎起不到什么作用。最关键的，SSL证书的信用链体系并不安全，特别是在某些国家可以控制CA根证书的情况下，中间人攻击一样可行


### HTTPS通信步骤

![https](http://p4emt3ysm.bkt.clouddn.com/https%E9%80%9A%E4%BF%A1%E8%BF%87%E7%A8%8B.png)

- （1）客户使用https的URL访问Web服务器，要求与Web服务器建立SSL连接。
- （2）Web服务器收到客户端请求后，会将网站的证书信息（证书中包含公钥）传送一份给客户端。
- （3）客户端的浏览器与Web服务器开始协商SSL连接的安全等级，也就是信息加密的等级。
- （4）客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。
- （5）Web服务器利用自己的私钥解密出会话密钥。
- （6）Web服务器利用会话密钥加密与客户端之间的通信。


### 国内外有哪些机构可以申请HTTPS证书

- 国内：
    - 沃通(WoSign) 中国人民银行联合12家银行建立的金融CFCA
    - 中国电信认证中心（CTCA） 海关认证中心（SCCA）
    - 国家外贸部EDI中心建立的国富安CA安全认证中心
    - SHECA（上海CA）为首的UCA协卡认证体系

- 国外：
    - StartSSL
    - GlobalSign
    - GoDaddy
    - Symantec

### get和post请求有什么区别，分别应该在什么场合下

- **get**: 从指定的服务器中获取数据
    - GET请求能够被缓存
    - GET请求会保存在浏览器的浏览记录中
    - 以GET请求的URL能够保存为浏览器书签
    - GET请求有长度限制
    - GET请求主要用以获取数据

- **post**: 向指定的服务器中发送数据
    - POST请求不能被缓存下来
    - POST请求不会保存在浏览器浏览记录中
    - 以POST请求的URL无法保存为浏览器书签
    - POST请求没有长度限制
    - POST请求会把请求的数据放置在HTTP请求包的包体中
    - POST的安全性比GET的高.可能修改变服务器上的资源的请求.

- **应用场合**：
    - post：
        - 要传送的数据不是采用7位的ASCII编码
        - 请求的结果有持续性的副作用（数据库内添加新的数据行）
        - 如果使用GET方法，表单上收集的数据可能让URL过长
    - get：
        - 请求是为了查找资源，HTML表单数据仅用来帮助搜索
        - 请求结果无持续性的副作用
        - 收集的数据及HTML表单内的输入字段名称的总长不超过1024个字符

### HTTP请求会有哪些信息发送到后台服务器
- 请求行
    - POST /demo/login HTTP/1.1
- 请求头
- 请求体
    - username=xxxx&password=1234
