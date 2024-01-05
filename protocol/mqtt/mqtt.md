### 声明：图片来自https://github.com/mcxiaoke/mqtt

# 控制报文概要

----

## 固定报头(Fixed Header)


### 剩余长度

从固定报头的第二个字节开始,表示[可变报头(Variable Header)+有效载荷(Payload)]部分的长度，是个可变长度，最大4子节，最小1子节

* 小于128字节 直接用单子节编码
* 大于128字节 低字节7位编码数字，第8位指示是否有更多子节。

### 大于128子节

* 假设是200，二进制是1100 1000，第八位是1，所以要再用一个字节表示，第八位的1代表进位，原来的1转移到下一个子节，变成 0000 0001 0100 1000
* 假设是1000，二进制是0011 1110 1000，超过7位了，所以要再用一个字节表示，第八位的1代表进位，原来的1转移到下一个子节，变成 0000 0111 1110 1000
* 假设是321，二进制是0001 0100 0001，超过7位了，所以要再用一个字节表示，第八位的1代表进位，原来的0转移到下一个子节，变成0000 0010 1100 0001


## 可变报头(Variable Header)

在固定报头和有效载荷之间

### 报文标识符

2个字节

* PUBLISH(QOS>0)
* SUBSCRIBE
* UNSUBSCRIBE

以上报文每发送一个，要分配一个没使用的，重发报文时，要用相同的标识。


## 有效载荷(Payload)

应用消息

# 控制报文详解

----

## CONNECT 连接请求
客户端向服务器建立连接后发送的第一个报文，只能发一次，重发会断开现在的链接。

### 固定报头

![image](./img/connect-header.png)

剩余长度=可变报头10字节+Payload长度

### 可变报头

10个字节

### 字节1-10 协议名

* 字节1 协议名长度高字节，数字0 
* 字节2 协议名长度低字节，数字4
* 字节3 M
* 字节4 Q
* 字节5 T
* 字节6 T
* 字节7 协议级别，数字4
* 字节8 链接标志，第一位是保留标志位，如果不是0则断开客户链接,2-8位分别代表不同的信息
* 字节9 KEEPALIVE时间 高字节
* 字节10 KEEPALIVE时间 低字节

keepalive表示两个控制报文之间允许最大的空闲间隔


![image](./img/connect-variable.png)

### Payload
内容由可变报头中的标识位决定,按照下面顺序出现：
* 客户端标识符 服务端使用clientId识别客户端,必须存在，并且是Payload第一个字段  
  1到23字节，只能包含大小写数字
* 遗嘱主题 
* 遗嘱消息
* 用户名 
* 密码 

## CONNACK 确认连接请求
服务端发送CONNACK报文响应从客户端收到的CONNECT，服务端发给客户端的第一个报文必须是CONNACK。

### 固定报头
![image](./img/connack-header.png)

### 可变报头

![image](./img/connack-variable.png)

第1个字节是 连接确认标志，位7-1是保留位且必须设置为0。第0 (SP)位 是当前会话（Session Present）标志。
第2个字节是 返回码，如果返回非0，SP=0
* clearSession=1：返回码=0，SP=0
* clearSession=0：
  - 已保存会话状态：返回码=0，SP=1
  - 未保存会话状态：返回码=0，SP=0
 
 
### Payload
无

## PUBLISH 发布

### 固定报头

![image](./img/publish-header.png)

* DUP 重发标志
  - 重发消息=1
  - QoS=0,DUP=1
* Qos-H Qos高位
* Qos-L Qos低位
* RETAIN 保留标志

#### Qos

* 0 最多分发一次
* 1 至少分发一次
* 2 正好分发一次


#### RETAIN

1. 客户端->服务端

如果客户端发给服务端的PUBLISH报文的保留（RETAIN）标志被设置为1，服务端必须存储这个应用消息和它的服务质量等级（QoS）。一个新的订阅建立时，对每个匹配的主题名，如果存在最近保留的消息，它必须被发送给这个订阅者。

如果服务端收到一条保留（RETAIN）标志为1的QoS 0消息，它必须丢弃之前为那个主题保留的任何消息。它应该将这个新的QoS 0消息当作那个主题的新保留消息。


2. 服务端->客户端


### 可变报头

* 主题名
* 报文标识符 只有当QoS等级是1或2时，报文标识符（Packet Identifier）字段才能出现在PUBLISH报文中

![image](./img/publish-variable.png)

### Payload

应用消息。


### 响应

PUBLISH报文的接收者必须按照根据PUBLISH报文中的QoS等级发送响应

* Qos=0 无响应
* Qos=1 PUBACK报文
* Qos=2 PUBREC报文

## PUBACK

PUBACK报文是对QoS 1等级的PUBLISH报文的响应。

### 固定报头

![image](./img/puback-header.png)


### 可变报头

包含等待确认的PUBLISH报文的报文标识符。

![image](./img/puback-variable.png)

### Payload

无


## PUBREC 发布收到

PUBREC报文是对QoS等级2的PUBLISH报文的响应。它是QoS 2等级协议交换的第二个报文。

### 固定报头

![image](./img/puback-header.png)


### 可变报头

![image](./img/puback-variable.png)

### Payload

无


## PUBREL 发布释放

PUBREL报文是对PUBREC报文的响应。它是QoS 2等级协议交换的第三个报文。






