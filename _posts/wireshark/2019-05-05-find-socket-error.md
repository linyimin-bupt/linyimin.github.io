---
layout: post
title: 使用wireshark查找socket连接失败
categories: [wireshark, socket]
---

> 管他什么真理无穷，进一寸有一寸的欢喜

<!-- TOC -->

- [使用wireshark查找socket连失败](#%E4%BD%BF%E7%94%A8wireshark%E6%9F%A5%E6%89%BEsocket%E8%BF%9E%E5%A4%B1%E8%B4%A5)
  - [字节序与网络字节序](#%E5%AD%97%E8%8A%82%E5%BA%8F%E4%B8%8E%E7%BD%91%E7%BB%9C%E5%AD%97%E8%8A%82%E5%BA%8F)
    - [字节序转换](#%E5%AD%97%E8%8A%82%E5%BA%8F%E8%BD%AC%E6%8D%A2)
  - [相关代码](#%E7%9B%B8%E5%85%B3%E4%BB%A3%E7%A0%81)

<!-- /TOC -->

## 使用wireshark查找socket连失败

在实现《TCP/IP网络编程》第5章的`计算器服务器端/客户端示例`时,服务器端和客户端代码都已经实现好了,相关代码如下.根据代码,当客户端连上服务器时,服务器端会打印`Connected...`字符串, 客户端会打印`Operand count: `提示输入相关信息.但是当运行的时候发现服务器端和客户端都没有打印相关信息, 服务器端和客户端一直在阻塞,也没有提示相关错误.大概两分钟后,客户端提示`connect() error.`

找了好久的原因也没找到(缺少错误信息提示), 于是就想着使用`wireshark`抓包分析一下, 没想到一抓包就找到了问题.

![超时重传](images/retransmission.png)

根据抓包信息,可以发现客户端发出的`SYN`数据段一直在超时重传.也就是服务器端一直收不到客户端的连接请求.再看`Destination`列发现, 客户端指定的服务IP居然是`1.0.0.127`, 很明显是客户端在网络地址初始化时出现了错误.查看客户端网络地址初始化的代码(如下):

```C
// init network address
memset(&server_addr, 0, sizeof(server_addr));
server_addr.sin_family = AF_INET;
server_addr.sin_addr.s_addr = htonl(inet_addr(argv[1]));
server_addr.sin_port = htons(atoi(argv[2]));
client_sock = socket(PF_INET, SOCK_STREAM, 0);
```

与IP地址相关的是第4行代码, 运行时输入的是`127.0.0.1`,为什么会变成`1.0.0.127`了呢?这个与CPU在内存中保存数据的方式有关.

### 字节序与网络字节序

CPU向内存中保存数据有两种方式:

- 大端序: 高位字节存放在低位地址(最高有效位对应实际地址)

- 小端序: 低位字节存放在高位地址(最低有效位对应实际地址)

目前主流的Intel系列CPU以小端序保存数据.由于存在不同的数据保存方式,所以两台字节序不同的计算机之间交换数据会出现问题.为了解决这个问题, 通过网络传输数据时需要约定统一方式,这种约定方式称为`网络字节序`, 目前统一为<font color="#dd0000">大端序</font>: 即先把数据数组转换成大端序格式在进行网络传输.

#### 字节序转换

在头文件`arpa/inet.h`中定义了如下常用的字节序转换函数:

- unsigned short htons(unsigned short);
- unsigned short ntohs(unsigned short);
- unsigned long htonl(unsigned long);
- unsigned long ntohl(unsigned long);

<pre>
htons中:
  h代表主机(host)字节序
  n代表网络(network)字节序
  s代表short类型
  l代表long类型
</pre>

在<arpa/inet.h>还提供了一个将字符串形式的IP地址转换成32位整型数据(<font color="#dd0000">大端序</font>)的函数`in_addr_t inet_addr(const char *string)`.

在对网络地址的初始化代码中, 先调用`inet_addr()`将字符串IP地址转成了大端序的整型数据, 然后有调用`htonl()`函数将大端序的IP地址转成小端序的整型数据.也就是使用小端序的数据在网络上传输, 所以小端序主机解析出来的就是`1.0.0.127`.至此, 终于能明白为什么传入的是`127.0.0.1`最后会变成了`1.0.0.127`.

在抓包中,我们还可以发现, 超时重传时间的变化规律:

**第一次重传**

![](images/RTO1.png)

**第二次重传**

![](images/RTO2.png)

**第三次重传**

![](images/RTO3.png)

**第四次重传**

![](images/RTO4.png)

**第五次重传**

![](images/RTO5.png)

**第六次重传**

![](images/RTO6.png)

<font color="#dd0000">1.01 --> 3.02 --> 7.06 --> 15.25 --> 31.38 --> 64.91</font>超时重传时间基本上以2倍的速度在增长,一共重传6次之后,如果还没有收到确认,则直接断开,不再重传. 

### 相关代码

**op_server.c**

```C

#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>

#define OPSZ 4
#define BUF_SIZE 1024

void error_handling(char *message) {
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}

int calculate(int opnum, int opinfo[], char op) {
    int result = opinfo[0];
    switch (op) {
        case '+': {
            for (int i = 1; i < opnum; i++) {
                result += opinfo[i];
            }
            break;
        }
        case '-': {
            for (int i = 1; i < opnum; i++) {
                result -= opinfo[i];
            }
            break;
        }
        case '*': {
            for (int i = 1; i < opnum; i++) {
                result *= opinfo[i];
            }
            break;
        }
        case '/': {
            for (int i = 1; i < opnum; i++) {
                result /= opinfo[i];
            }
            break;
        }
    }

    return result;
}

int main(int argc, char *argv[]) {
    int server_sock;
    int client_sock;

    char opinfo[BUF_SIZE];

    struct sockaddr_in server_addr;

    if (argc != 2) {
        printf("Usage: %s <port>", argv[0]);
        exit(1);
    }

    server_sock = socket(PF_INET, SOCK_STREAM, 0);
    if (server_sock == -1) {
        error_handling("socket() error");
    }

    // init network address
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(atoi(argv[1]));
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);

    if (bind(server_sock, (const struct sockaddr *) &server_addr, sizeof(server_addr)) == -1) {
        error_handling("bind() error");
    }

    if (listen(server_sock, 5) == -1) {
        error_handling("listen() error");
    }

    struct sockaddr_in client_addr;
    socklen_t client_addr_size = sizeof(client_addr);

    for (int i = 0; i , 5; i++) {
        client_sock = accept(server_sock, (struct sockaddr *) &client_addr, &client_addr_size);
        if (client_sock == -1) {
            error_handling("accept() error");
        } else {
            printf("Connected ...\n");
            printf("client info: \n");
            printf("server: %s, port: %u", inet_ntoa(client_addr.sin_addr), client_addr.sin_port);
        }

        int opnd_cnt = 0;
        read(client_sock, &opnd_cnt, 1);

        int rev_len = 0;
        while ((opnd_cnt * OPSZ + 1) > rev_len) {
            int rev_cnt = read(client_sock, &opinfo[rev_len], BUF_SIZE - 1);
            rev_len += rev_cnt;
        }
        int result = calculate(opnd_cnt, (int *)opinfo, opinfo[rev_len - 1]);
        write(client_sock, &result, sizeof(result));
        close(client_sock);
    }
    close(server_sock);
    return 0;
}
```


**op_client.c**

```C
#include <stdlib.h>
#include <stdio.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>

#define BUF_SIZE 1024
#define RLT_SIZE 4
#define OPSZ 4

void error_handling(char *message) {
    fputs(message, stderr);
    exit(0);
}

int main(int argc, char *argv[]) {
    int client_sock;
    struct sockaddr_in server_addr;

    char opmsg[BUF_SIZE];   // 存储操作数
    int result, opnd_cnt;

    char message[BUF_SIZE];
    if (argc != 3) {
        printf("Usage: %s <ip> <port>\n", argv[0]);
        exit(1);
    }

    // init network address
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(inet_addr(argv[1]));
    server_addr.sin_port = htons(atoi(argv[2]));

    client_sock = socket(PF_INET, SOCK_STREAM, 0);

    if (client_sock == -1) {
        error_handling("socket() error");
    }

    if (connect(client_sock, (const struct sockaddr *) &server_addr, sizeof(server_addr)) == -1) {
        error_handling("connect() error");
    }

    fputs("Operand count: ", stdout);
    scanf("%d", &opnd_cnt);
    opmsg[0] = (char)opnd_cnt;

    for (int i = 0; i < opnd_cnt; i++) {
        printf("Operand: %d", i+1);
        scanf("%d", (int *)&opmsg[OPSZ * i + 1]);
    }
    fgetc(stdin);
    fputs("Operator: ", stdout);
    scanf("%c", &opmsg[OPSZ * opnd_cnt + 1]);
    // send data to server
    write(client_sock, opmsg, opnd_cnt * OPSZ + 2);
    // received data from server
    read(client_sock, &result, RLT_SIZE);
    printf("Operation result: %d\n", result);
    close(client_sock);
    return 0;
}
```

