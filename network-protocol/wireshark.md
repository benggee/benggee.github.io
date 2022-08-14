# 1. Wireshark面板

![image.png](/_media/wireshark-001.png)

### 菜单

![image.png](/_media/wireshark-002.png)

#### 时间显示格式
**绝对时间**
![image.png](/_media/wireshark-003.png)

**相对时间**
![image.png](/_media/wireshark-004.png)

**设定基于某一个数据包的相对时间**
![image.png](/_media/wireshark-006.png)

#### 数据包列表面板标记符号
![image.png](/_media/wireshark-007.png)


#### 四种流跟踪
- TCP
- UDP
- SSL
- HTTP


#### 标记报文

ctrl + m


#### 抓取移动设备包
1. 在操作系统打开wifi热点
2. 手机连接wifi热点
3. 用wireshark打开捕获->选项面板，选择wifi热点对应的接口设备抓包


# 2. Wireshark过滤器
## BPF过滤器：Wireshark捕获过滤器
- Berkeley Packet Filter, 在设备驱动级别提供抓包过滤接口，多数抓包工具都支持
- expression 表达式：由多个源语组成

## Expression表达式
#### Primitives原语：由名称或数字，以及描述它的多个限定词组成
- qualifiers限定词
    **Type: 设置数字或者名称所指定的类型，例如 host www.baidu.com**
        host, port
        net, 设定子网， net 192.168.0.0 mask 255.255.255.0 等价于 net 192.168.0.0/24
        portrange, 设置端口范围，例如portrange 6000-8000
        
    **Dir: 设置网络出入方向，例如 dst port 80**
        src、dst、src or dst、 src and dst
        ra、ta、addr1、addr2、addr3、addr4(仅对IEEE802.11 Wireless LAN有效)

    **Proto: 指定协议类型，例如 udp**
        ether、fddi、tr、 wlan、 ip、 ip6、 arp、 rarp、 decnet、 tcp、udp、icmp、igmp、icmp、 igrp、pim、ah、esp、vrrp

    **其他**
        • gateway：指明网关 IP 地址，等价于 ether host ehost and not host host 
        • broadcast：广播报文，例如 ether broadcast 或者 ip broadcast 
        • multicast：多播报文，例如 ip multicast 或者 ip6 multicast 
        • less, greater：小于或者大于
- 原语运算符
    与： && 或者  and
    或： || 或者 or
    非： ！或者 not

- 举例：src or dst portrange 6000-8000 && tcp or ip6


## 基于协议域过滤
#### 捕获所有TCP中的RST报文
    tcp[13]&4==4
    
#### 抓取HTTP GET报文
    port 80 and tcp[((tcp[12:1]&0xf0)>>2):4] = 0x47455420
       注意：47455420是ASCII码的16进制，表示 “GET”
       TCP报头可能不只20字节，data offset提示了承载数据的偏移，但它以4字节为单位
![image.png](/_media/wireshark-008.png)


## 显示过滤器的过滤属性
任何在报文细节面板中解析出的字段名，都可以作为过滤属性。菜单：视图>内部>支持的协议面板
例如，在报文细节面板中TCP协议头中的Source Port，对应着过滤属性为tcp.srcport
![image.png](/_media/wireshark-009.png)


## 过滤值比较符号

| 英文 | 符号 | 描述及示例 |
| --- | --- | --- |
| eq | == | 等于.ip.src=10.0.0.5 |
| ne | != | 不等于.ip.src!=10.0.0.5 |
| gt | > | 大于.frame.len>10 |
| lt | < | 小于.frame.len<128 |
| ge | >= | 大于等于.frame.len ge 0x100 |
| le | <= | 小于等于.fram.len <=0x20 |
| contains |  | 包含.sip.To contains "a1762" |
| matches | ~ | 正则匹配.host matches "acme\.(org|com|net)" |
| bitwise_and | & | 位与操作.tcp.flags & 0x02 |

## 过滤值类型
- Unsigned integer: 无符号整型，例如 ip.len le 1500
- Signed integer: 有符号整型
- Boolean: 布尔值，例如 tcp.flags.syn
- Ethernet address: 以：、-或者.分隔的6字节地址，例如 eth.dst==ff:ff:ff:ff:ff:ff
- IPv4 address: 例如 ip.addr==192.168.0.1
- IPv6 address: 例如 ipv6.addr==::1
- Text string: 例如 http.request.uri=="https://www.wireshark.org/"

## 多个表达式间的组合

| 英文 | 符号 | 意义及示例 |
| --- | --- | --- |
| and | && | AND逻辑与.ip.src==10.0.0.5 and tcp.flags.fin |
| or | || | OR逻辑或.ip.src==10.0.0.5 or ip.src==192.1.1.1 |
| xor | ^^ | XOR逻辑异或.tr.dst[0:3] ==0.6.29 xor tr.src[0:3]==0.6.29|
| not | ! | NOT逻辑非. not || c |
| [...] |  | 见slice切片操作符 |
| in |  | 见集合操作符 |

## 其它常用操作符
### 大括号{}集合操作符
- 例如tcp.port in {443 4430..4434}，实际操作等价于tcp.port==443||(tcp.port>=4430 &&tcp.port<=4434)
### 中括号[]slice切片操作符
- [n:m]表示n是起始偏移量，m是切片长度，eth.src[0:3]==00:00:83
- [n-m]表示n是起始偏移量，m是截止偏移量，eth.src[1-2]==00:83
- [:m]表示从开始处至m截止偏移量，eth.src[:4]==00:00:83:00
- [m:]表示m是起始偏移量，至字段结尾，eth.src[4:] == 20:20
- [m]表示取偏移量m处的字节,eth.src[2] == 83
- [,]使用逗号分隔时，允许以上方式同时出现, eth.src[0:3,1-2,:4,4:,2]==00:00:83:00:83:00:00:83:00:20:20:83


## 可用函数
- upper       Converts a string field to uppercase
- lower       Converts a string field to lowercase
- len         Returns the byte length of a string or bytes field
- count       Returns the number of field occurrences in a frame.
- string      Converts a non-string field to a string.


## 显示过滤器的可视化对话框
![image.png](/_media/wireshark-010.png)