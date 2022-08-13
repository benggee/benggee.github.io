# 捕获及停止条件
- -D列举所有网卡设备
- -i选择网卡设备
- -c抓取多少条报文
- --time-stamp-precision 指定捕获时的时间精度，默认毫秒micro，可选纳秒nano
- -s指定每条报文的最大字节数，默认262144字节


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
![image.png](https://note.youdao.com/yws/res/18208/WEBRESOURCEfaaebe9f208a7cecbd900d2f049f6967)


# 文件操作
- -w 输出结果至文件
- -C 限制输入文件的大小，超出后以后缀加1等数字的形式递增。注意：单位是1000000字节
- -W 指定输出文件的最大数量，到达后会重新覆写第1个文件
- -G 指定每隔N秒就重新输出至新文件，注意-w参数应基于strftime参数指定文件名
- -r 读取一个抓包文件
- -V 将待读取的多个文件名写入一个文件中，通过读取该文件同时读取多个文件


# 输出时间戳格式
- -t 不显示时间戳
- -tt 自1970-01-01 00:00:00至今的秒数
- -ttt 显示邻近两行报文间经过的秒数
- -tttt 带日期的完整时间
- -ttttt 自第一个抓取的报文起经历的秒数


# 分析信息详情
- -e 显示数据链路层头部
- -q 不显示传输层信息
- -v 显示网络层头部更多的信息，如TTL, id等
- -n 显示IP地址、数字端口代替hostname等
- -S TCP信息以绝对序列号替代相对序列号
- -A 以ASCII方式显示报文内容，适用HTTP分析
- -x 以16进制方式显示报文内容，显示数据链路层
- -xx 以16进制方式显示报文内容，显示数据链路层
- -X 同时以16进制及ASCII方式显示报文内容，不显示数据链路层
- -XX 同时以16进制及ASCII方式显示报文内容，显示数据链路层