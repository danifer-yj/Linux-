![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016161240893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNzYwMjkx,size_16,color_FFFFFF,t_70#pic_center)

# 套接字描述符

1.创建一个套接字

```cpp
#include <sys/socket.h>

int socket (int domain, int type, int protocol);
//返回值:成功返回套接字的文件描述符,失败返回-1,并设置errno
```

参数: 

- domain: (域)确定通信的特性,包括地址格式.各个域都有自己表示地址的格式.而表示各个域的常数都以AF_开头(地址簇)

| 域        | 描述         |
| --------- | ------------ |
| AF_INET   | IPv4英特网域 |
| AF_INET6  | IPv6因特网域 |
| AF_UNIX   | UNIX域       |
| AF_UPSPEC | 未指定       |

- type: 确定套接字的类型,进一步确定通信特征.

| 类型           | 描述                                        |
| -------------- | ------------------------------------------- |
| SOCK_DGRAM     | 固定长度的,无连接的,不可靠的报文传递        |
| SOCK_RAW       | IP协议的数据报接口                          |
| SOCK_SEQPACKET | 固定长度的,有序的,可靠的,面向连接的报文传递 |
| SOCK_STREAM    | 有序的,可靠的,双向的,面向连接的字节流       |

- protocol: protocol通常为0.表示为给定的域和套接字选择默认的协议.

| 协议         | 描述               |
| ------------ | ------------------ |
| IPPROTO_IP   | IPv4网际协议       |
| IPPROTO_IPV6 | IPv6网际协议       |
| IPPROTO_ICMP | 因特网控制报文协议 |
| IPPROTO_RAW  | 原始IP数据包协议   |
| IPPROTO_TCP  | 传输控制协议       |
| IPPROTO_UDP  | 用户数据报协议     |

虽然套接字的本质是一个文件描述符,但并不是所有以文件描述符为参数的函数都接受套接字描述符.下表总结了大多数以文件描述符为参数的函数的使用套接字描述符是的行为.

![image-20201101152645351](C:\Users\杨金金\AppData\Roaming\Typora\typora-user-images\image-20201101152645351.png)

# 字节序

在同一台机器上的两个进程进行通信时,是不用考虑字节序问题.字节序是处理器的特性.用来指示一个数据类型是如何排序的.因为网络通信是涉及到两台不同的机器上的进程通信,就要考虑到两个处理器的字节序(大小端)问题.字节序就是保证接收端接收到的数据可以正确的解析.如果不考虑字节序问题,两个网络设备就没法传输正确数据了.

在发送数据前需要将数据从主机序转换为网络序,在从网络中接收到数据后,需要将网络序转换成主机序.

网络字节序一般是大端序,主机序则根据硬件的不同而不同,在调用字节转换函数,函数内部会根据当前的机器的结构来自动为我们选择是否要转换数据的字节序.不管怎样.只要我们从主机发送数据到网络上就要调用hton函数,从网络上接数据到主机是就要调用ntoh函数.h----->host,n------>network,l-------->long,s-------->short

- 大端: 低址存数据高位, 高址存数据低位

- 小端: 低址存数据低位, 高址存数据高位

![img](https://images0.cnblogs.com/blog2015/234353/201507/041752279239537.png)

下面是一组字节序转换函数.

```cpp
htonl, htons, ntohl, ntohs (字节序转换)

#include <arpa/inet.h>

uint32_t hotnl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

数据发送到网络: 就要调用hton

从网络中收数据到主机: 就要调用ntoh

# C/S两端通信流程

C端:

1. 取得Socket
2. 给Socket取得地址
3. 发/收消息
4. 关闭Socket

S端:(先运行)

1. 取得Socket
2. 给Socket取得地址
3. 收/发消息
4. 关闭Socket

# 地址格式

struct sockaddr 和 struct sockaddr_in这两个结构体来处理网络通信地址

sockaddr

```cpp
//在#include <sys/socket.h>中定义

struct sockaddr{
	sa_family_t sin_family;	//地址族
	char sa_data[14];		//14字节,包含套接子中的目标地址和端口信息
}
```

sockaddr_in

```cpp
//在#include <netinet/in.h> or #include <arpa/inet.h>中定义
struct sockaddr{
	sa_family_t		sin_family;		//地址族(address family)
	uint16_t		sin_port;		//16位TCP/UDP端口号
	struct in_addr	sin_addr;		//32位IP地址
	char			sin_zero[8];	//不使用
}

struct in_addr{
	In_addr_t		s_addr;		//32位IPv4地址
}
```





文本地址==>二进制地址:inet_pton;                |                  二进制地址==>文本地址inet_ntop

```cpp
#include <arpa/inet.h>

const char *inet_ntop(int domain,const void *restrict addr,
				char *restrict str, socklen_t size);
//返回值:成功返回地址字符串指针,错误返回NULL

int inet_pton(int domain, char *restrict str, void *restrict addr);
//返回值:成功返回1,若格式无效返回0,出错返回-1
```

inet_ntop将网络字节序的二进制地址转换成文本字符串格式.

inet_pton将文本字符串格式转换成网络字节序的二进制地址.

参数:

- domain: 仅支持量个值:AF_INET和AF_INET6
- 对于inet_ntop参数size指定了保存文本字符串的缓冲区(str)的大小.有两个常数用来简化工作:INET_ADDRSTRLEN定义了足够大的空间来存放一个表示IPv4地址的文本字符串;INET6_ADDRSTRLEN定义了足够大的空间来存放一个表示IPv6地址的文本字符串.对于inet_pton,若domain是AF_INET,则缓冲区addr需要足够大的空间来存放一个32位地址,若domain是AF_INET6,则需要足够大的空间来存放一个128位的地址.

# 将套接字与地址相关连



5.bind函数将地址和套接字关联

```cpp
#include < sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr,socklen_t len);
//返回值:成功返回0,失败返回-1
```

bind函数用于绑定本机端口号的,

参数: 

- sockfd: 就是socket创建的套接字所得到的文件描述符,
- addr: 要绑定到套接字上的地址.
- addrlen:addr传递的地址结构体的长度.

以AF_INET为例,

```cpp
struct sockaddr_in {
sa_family_t sin_family; /* 指定协议族，一定是 AF_INET，因为既然是 man ip(7)，那么一定是 AF_INET 协议族的 */
in_port_t sin_port; /* 端口，需要使用 htons(3) 转换为网络序 */
struct in_addr sin_addr; /* internet address */
};

/* Internet address. */
struct in_addr {
uint32_t s_addr; /* 无符号32位大整数，可以使用 inet_pton(3) 将便于记忆的点分式 IP 地址表示法转换为便于计算机使用的大整数，inet_ntop(3) 的作用则正好相反。本机地址转换的时候可以使用万能IP：0.0.0.0(称为any address)，函数会自动将 0.0.0.0 解析为真实的本机 IP 地址。 */
};
```

# 建立连接

在交换数据之前,需要在请求服务的套接字和提供服务的套接字之间建立一个连接,在这里客户端使用connet函数来建立连接.

```cpp
#include <sys/socket.h>

int connet(int sockfd, const struct sockaddr *addr, sockelen_t len);
//返回值:成功返回0,失败返回-1;
```

参数:

- sockfd: 客户端要建立连接的套接字,
- addr: 这个地址是我们想与之通信的服务器地址.若sockfd没有绑定到一个地址,connect会给调用者绑定一个默认的地址.

服务器调用listen函数来宣告他愿意接受连接请求

```cpp
#include <sys/socket.h>

int listen(int sockfd, int backlog);
//返回值: 成功返回0,失败返回-1
```

一旦服务器调用了listen,所用的套接字就可以接收连接请求.服务端调用accept函数获得连接请求并建立连接.

```cpp
#include <sys/socket.h>

int accept (int sockfd, struct sockaddr *restrict addr, socklen_t *restrict len);
//返回值:成功返回套接字描述符,出错返回-1;
```

函数accept所返回的文件描述符是套接字描述符,描述符连接到调用connect的客户端.这新的套接字和原始套接字(sockfd)具有相同的套接字类型和地址族.传给accept的原始套接字没有关联到这个连接,而是继续保持可用的状态并接受其他请求.如果不关心客户端标识,可以将参数addr和len置为NULL.

如果没有连接请求,accept会阻塞知道一请求到来.如果sockfd处于非阻塞模式,accept会返回-1,并将erron设置为ENGAIN或EWOULDBLOCK.

# 数据传输

1.发送数据:send/sendto函数

```cpp
#include <sys/socket.h>
#include <sys/types.h>

ssize_t send(int sockfd, const void *buf, size_t len, int flags);//TCP
ssize_t sendto(int sockfd,const void *buf,size_t len,int flags,
			const struct sockaddr *dest_addr, socklen_t addrlen);//UDP,因而为dest_addr指明了地址类型
//成功返回发送的字节数,出错返回-1;
```

参数列表:

- sockfd: 通过哪个Socket往外发送数据,
- buf: 要发送的数据
- len: 要发送的数据的长度
- flags: 特殊要求,没有填0
- dest_addr: 目标地址;
- addrlen: 目标地址的长度.

2.接收数据:recv/recvfrom

```cpp
#include <sys/types.h>
#include <sys/socket.h>

ssize_t rec (int sockfd, void *buf, size_t len, int flags);//流式
ssize_t recvfrom(int sockfd, void *buf,size_t len,int flags,
				struct sockaddr *src_addr, socklen_t *addrlen);//报式
//成功返回数据的长度;若无可用数据或对等方已经按需结束,返回0;出错返回-1
```

这两个函数的作用是:从网络中接收内容并写入len字节长度的数据到buf中,将发送端的地址填写到src_addr中.

close(fd) 关闭套接字.