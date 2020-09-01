# Linux TCP编程

## TCP/IP协议栈

根据传输方式不同，基于网络协议的套接字一般分为TCP套接字和UDP套接字。因为TCP套接字是**<u>面向连接</u>**的，因此又称**基于流**（**stream**）的套接字。TCP是**Transmission Control Protocol**（**<u>传输控制协议</u>**）的简写，意为“**对数据传输过程的控制**”。

![TCP/IP协议栈](F:\我的\博客记录\linux\image\tcp四层架构.png)

​																							**TCP/IP协议栈**

### 第一层 次 ：链路层

链路层是物理链接领域的标准化结果，也是最基本的领域，专门定义了**LAN，WAN，MAN**等网络标准。若两台主机通过网络交换数据，则需下图所示的物理链接，链路层就负责这些。

<img src="F:\我的\博客记录\linux\image\链路层.png" alt="链路层" style="zoom:80%;" />

### 第二层次：IP层

准备好物理链接之后就要传输数据。为了在复杂的网络环境中传输数据，首先需要考虑路径的选择。<u>**向目标传输数据需要经过哪条路径**</u>？解决该问题的就是**IP层**。该层用的协议就是**IP**。

**IP**是**面向消息的**、**不可靠的**协议，每次传输数据是会帮我们选择路径，但并不一致。如果传输中发生路径错误，则选择其他路径；但如果发生数据丢失或错误，则无法解决。换言之，IP协议是无法应对数据错误的。因此，又要放下一层。

### 第三层次：TCP/UDP层

IP层解决数据传输中的**路径选择问题**，只需照此路径传输数据即可。**TCP**和**UDP**以**IP**提供的路径信息为基础<u>**完成实际的数据传输**</u>，故该层又称**传输层**（**Transport**）。

IP层只关注**1个数据包（数据传输的基本单位）**的传输过程。因此即使传输多个数据包，每个数据包也是有IP层实际传输的，**也就是说传输顺序和传输本身是不可靠的**。若只是利用IP层传输数据，则有可能后传输的数据包B比先传输的数据包A提前到达。另外，传输的数据包A、B、C中可能只收到A和C，甚至收到的C可能已经损毁。

若添加TCP协议则按照下图的对话方式进行数据传输

<img src="F:\我的\博客记录\linux\image\TT.png" alt="T1" style="zoom:80%;" />







### 第四层次 应用层

前三个层次，套接字处理过程中都是**自动处理**的。为了使“程序员从这些细节中解放出来”。选择数据传输路径、数据确认过程都被隐藏到套接字内部。前三个层次是为了给应用层提供服务的。

## TCP 服务端

<img src="F:\我的\博客记录\linux\image\TS.png" alt="TCP服务端" style="zoom:80%;" />

### TCP 服务端代码实现

```c++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <cstring>
#include <unistd.h>

void tcp_server() {
    int server_sock, client_sock;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_addr_len;
    const char *message = "Hello Word!\n";
    server_sock = socket(PF_INET, SOCK_STREAM, 0);//TCP 协议
    if (server_sock < 0) {
        std::cout << "create socket failed!" << std::endl;
        return;
    }
    memset(&server_sock, 0, sizeof(server_sock));//清零
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_addr.sin_port = htons(9020);
    //(struct sockaddr *)&server_addr 为了兼容C语言 不能做成重载
    int ret = bind(server_sock, (struct sockaddr *) &server_addr, sizeof(server_addr));
    if (ret == -1) {
        std::cout << "bind failed!" << std::endl;
        close(server_sock);
        return;
    }
    ret = listen(server_sock, 3);//进入等待连接请求状态 成功时返回0，失败时返回-1
    if (ret == -1) {
        std::cout << "listen failed!" << std::endl;
        close(server_sock);
        return;
    }
    //受理客户端连接请求 成功时返回创建的套接字文件描述符，失败时返回-1
    client_sock = accept(server_sock, (struct sockaddr *) &client_addr, &client_addr_len);
    if(client_sock==-1){
        std::cout << "accept failed!" << std::endl;
        close(server_sock);
        return;
    }
    ssize_t write_len_message = write(client_sock, message, strlen(message));
    if(write_len_message!= strlen(message)){
        std::cout << "write failed!" << std::endl;
        close(server_sock);
        return;
    }
    close(client_sock);//可以省略，因为服务端关闭的时候客户端会自动关闭
    close(server_sock);
}

```

## TCP 客户端

### connect()函数

```c++
#include <sys/socket.h>
/**
*成功时返回0，失败是返回-1
* __fd：客户端套接字连接文件描述符
* __CONST_SOCKADDR_ARG=const struct sockaddr* ： 保存目标服务器端地址信息的地址变量
* __len ：以字节为单位传递已传递给第二个结构体参数__addr的地址变量长度
*/
int connect (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)
```

客户端套接字地址信息在哪儿？

实现服务端毕竟过程之一就是给套接字分配IP和端口号。而客户端实现过程套接字地址分配是在调用 **connect**函数时、在操作系统中（更准确的说是在内核中）、IP用计算机主机IP，端口号随机分配。

<u>客户端的IP地址和端口在调用**connect**函数时自动分配，无需调用标记**bind**函数进行分配。</u>

**这就是客户端与服务端的不同。**

### 客户端代码

#### 基于TCP服务端与客户端的函数调用关系

<img src="F:\我的\博客记录\linux\image\基于TS_TC.png" alt="基于TS_TC" style="zoom: 67%;" />

#### 客户端与服务端联调代码实现

```c++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <cstring>
#include <unistd.h>
#include <sys/wait.h>

void tcp_client_01() {
    //创建一个子进程
    __pid_t pid = fork();
    if (pid == 0) {
        sleep(2);//子进程休眠2秒钟，
        //开启客户端
        int client = socket(PF_INET, SOCK_STREAM, 0);
        struct sockaddr_in server_addr{};
        memset(&server_addr, 0, sizeof(server_addr));
        server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
        server_addr.sin_family = AF_INET;
        server_addr.sin_port = htons(9527);
        int ret = connect(client, (struct sockaddr *) &server_addr, sizeof(server_addr));
        if (ret == 0) {
            printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
            char buff[256] = "";
            read(client, buff, sizeof(buff));
            std::cout << buff << std::endl;
        } else{
            printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
        }
        close(client);
        std::cout << "client done" << std::endl;
    } else if (pid > 0) {
        tcp_server_01();
        int status = 0;
        wait(&status);
    } else {
        std::cout << "fork failed!" << std::endl;
    }
}
```

### 迭代服务器实现

```c++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <cstring>
#include <unistd.h>
#include <sys/wait.h>


void tcp_server_01() {
    int server_sock, client_sock;
    struct sockaddr_in server_addr{}, client_addr{};
    socklen_t client_addr_len;
//    const char *message = "Hello Word!\n";
    server_sock = socket(PF_INET, SOCK_STREAM, 0);//TCP 协议
    if (server_sock < 0) {
        std::cout << "create socket failed!" << std::endl;
        return;
    }
    memset(&server_addr, 0, sizeof(server_addr));//清零
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("0.0.0.0");
    server_addr.sin_port = htons(9527);
    int ret = bind(server_sock, (struct sockaddr *) &server_addr, sizeof(server_addr));
    if (ret == -1) {
        std::cout << "bind failed!" << std::endl;
        close(server_sock);
        return;
    }
    ret = listen(server_sock, 3);
    if (ret == -1) {
        std::cout << "listen failed!" << std::endl;
        close(server_sock);
        return;
    }
    printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
    char buff[1024] = "";

    while (true) {
        memset(&buff, 0, sizeof(buff));
        client_sock = accept(server_sock, (struct sockaddr *) &client_addr, &client_addr_len);
        if (client_sock == -1) {
            std::cout << "accept failed!" << std::endl;
            close(server_sock);
            return;
        }
        read(client_sock, buff, sizeof(buff));
        ssize_t write_len_message = write(client_sock, buff, strlen(buff));
        if (write_len_message != strlen(buff)) {
            std::cout << "write failed!" << std::endl;
            close(server_sock);
            return;
        }
        close(client_sock);//可以省略，因为服务端关闭的时候客户端会自动关闭
    }
    close(server_sock);
}

void tcp_client_01() {
    //创建一个子进程
    __pid_t pid = fork();
    if (pid == 0) {
        sleep(2);//子进程休眠2秒钟，
        //开启客户端
        int client = socket(PF_INET, SOCK_STREAM, 0);
        struct sockaddr_in server_addr{};
        memset(&server_addr, 0, sizeof(server_addr));
        server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
        server_addr.sin_family = AF_INET;
        server_addr.sin_port = htons(9527);
        int ret = connect(client, (struct sockaddr *) &server_addr, sizeof(server_addr));
        if (ret == 0) {
            printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
            char buff[256] = "hello ,here is client!";
            write(client, buff, strlen(buff));
            memset(&buff, 0, sizeof(buff));
            read(client, buff, sizeof(buff));
            std::cout << buff << std::endl;
        } else {
            printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
        }
        close(client);
        std::cout << "client done" << std::endl;
    } else if (pid > 0) {
        tcp_server_01();
        int status = 0;
        wait(&status);
    } else {
        std::cout << "fork failed!" << std::endl;
    }
}

int main() {
//    std::cout << "Hello, World!" << std::endl;
    tcp_client_01();
    return 0;
}

```

### 回声服务器实现

```c++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <cstring>
#include <unistd.h>
#include <sys/wait.h>

void tcp_server_02() {
    int server_sock, client_sock;
    struct sockaddr_in server_addr{}, client_addr{};
    socklen_t client_addr_len;
//    const char *message = "Hello Word!\n";
    server_sock = socket(PF_INET, SOCK_STREAM, 0);//TCP 协议
    if (server_sock < 0) {
        std::cout << "create socket failed!" << std::endl;
        return;
    }
    memset(&server_addr, 0, sizeof(server_addr));//清零
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr("0.0.0.0");
    server_addr.sin_port = htons(9527);
    int ret = bind(server_sock, (struct sockaddr *) &server_addr, sizeof(server_addr));
    if (ret == -1) {
        std::cout << "bind failed!" << std::endl;
        close(server_sock);
        return;
    }
    ret = listen(server_sock, 3);
    if (ret == -1) {
        std::cout << "listen failed!" << std::endl;
        close(server_sock);
        return;
    }
    printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
    char buff[1024] = "";
    for (int i = 0; i < 2; ++i) {
        memset(&buff, 0, sizeof(buff));
        client_sock = accept(server_sock, (struct sockaddr *) &client_addr, &client_addr_len);
        if (client_sock == -1) {
            std::cout << "accept failed!" << std::endl;
            close(server_sock);
            return;
        }
        ssize_t read_len = 0;
        while ((read_len = read(client_sock, buff, sizeof(buff))) > 0) {
            ssize_t write_len_message = write(client_sock, buff, strlen(buff));
            if (write_len_message != strlen(buff)) {
                std::cout << "write failed!" << std::endl;
                close(server_sock);
                return;
            }
            memset(buff, 0, read_len);
        }
        close(client_sock);//可以省略，因为服务端关闭的时候客户端会自动关闭
    }
    close(server_sock);
}

void run_client_02() {
    //创建一个子进程
    __pid_t pid = fork();
    if (pid == 0) {
        sleep(2);//子进程休眠2秒钟，
        //开启客户端
        int client = socket(PF_INET, SOCK_STREAM, 0);
        struct sockaddr_in server_addr{};
        memset(&server_addr, 0, sizeof(server_addr));
        server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
        server_addr.sin_family = AF_INET;
        server_addr.sin_port = htons(9527);
        int ret_conn = connect(client, (struct sockaddr *) &server_addr, sizeof(server_addr));
        while (0 == ret_conn) {
            printf("%s(%d):%s\n", __FILE__, __LINE__, __PRETTY_FUNCTION__);
            char buff[256] = "";
            fputs("Input messages (Q to quit):", stdout);
            fgets(buff, sizeof(buff), stdin);
            if ((strcmp(buff, "q\n") == 0) || (strcmp(buff, "Q\n") == 0)) {
                break;
            }
            size_t len = strlen(buff);
            size_t send_len = 0;
            while (send_len < len) {
                ssize_t ret = write(client, buff + send_len, len - send_len);
                if (ret <= 0) {
                    fputs("write failed:", stdout);
                    close(client);
                    std::cout << "client done" << std::endl;
                }
                send_len += ret;
            }

            memset(&buff, 0, sizeof(buff));
            size_t read_len = 0;
            while (read_len < len) {
                ssize_t ret = read(client, buff + read_len, len - read_len);
                if (ret <= 0) {
                    fputs("read failed:", stdout);
                    close(client);
                    std::cout << "client done" << std::endl;
                }
                read_len += ret;
            }

            std::cout << "form server:" << buff << std::endl;
        }
        close(client);
        std::cout << "client done" << std::endl;
    } else if (pid > 0) {
        tcp_server_01();
        int status = 0;
        wait(&status);
    } else {
        std::cout << "fork failed!" << std::endl;
    }
}

void tcp_02() {
    //创建一个子进程
    __pid_t pid = fork();
    if (pid == 0) {
        tcp_server_02();
    } else if (pid > 0) {
        for (int i = 0; i < 2; ++i) {
            run_client_02();
        }
        int status = 0;
        wait(&status);
    } else {
        std::cout << "fork failed!" << std::endl;
    }
}


int main() {
    tcp_02();
    return 0;
}
```

#### 效果图

<img src="F:\我的\博客记录\linux\image\回声服务.png" alt="回声服务" style="zoom:50%;" />

## **TCP** **套接字的** **I/O** **缓冲**

我们知道，TCP 套接字的数据收发无边界。服务器端即使调用 1 次 write 函数传输 40 字节的数据，客户端也有可能通过 4 次 read 函数调用每次读取 10 字节。但此处也有一些疑问，服务器端一次性传输了 40 字节，而客户端居然可以缓慢地分批接收。客户端接收 10 字节后，剩下的 30 字节在何处等候呢?是不是像飞机为等待着陆而在空中盘旋一样，剩下 30 字节也 在网络中徘徊并等待接收呢?

实际上，**<u>write 函数调用后并非立即传输数据，read 函数调用后也并非马上接收数据。</u>**更准确地说，如下图所示，write 函数调用瞬间，数据将移至输出缓冲;read 函数调用瞬间， 从输人缓冲读取数据。

![image-20200901205243727](C:\Users\GodHao\AppData\Roaming\Typora\typora-user-images\image-20200901205243727.png)

调用 write 函数时，数据将移到输出缓冲，在适当的时候（不管是分别传送还是一次性传送）传向对方的输入缓冲。这时对方将调用 read 函数从输入缓冲读取数据。这些 I/O 缓冲特性可整理如下。

1. I/O 缓冲在每个 TCP 套接字中单独存在。
2. I/O 缓冲在创建套接字时自动生成。
3. 即使关闭套接字也会继续传递输出缓冲中遗留的数据。
4. 关闭套接字将丢失输入缓冲中的数据。

那么，下面这种情况会引发什么事情?理解了 I/O 缓冲后，其流程：

"客户端输入缓冲为 50 字节，而服务器端传输了 100 字节。"

这的确是个问题。输入缓冲只有 50 字节，却收到了 100 字节的数据。可以提出如下解决方案∶

**填满输入缓冲前迅速调用 read 函数读取数据**，这样会腾出一部分空间，问题就解决了。

其实根本不会发生这类问题，因为 **TCP 会控制数据流。**

TCP 中有滑动窗口（Sliding Window）协议，用对话方式呈现如下

套接字 A∶"你好，最多可以向我传递 50 字节。"

套接字 B∶"OK!" 

套接字 A∶"我腾出了 20 字节的空间，最多可以收 70 字节。 

套接字 B∶"OK!

数据收发也是如此，因此 **<u>TCP 中不会因为缓冲溢出而丢失数据。</u>**

## **TCP** **的内部原理** 

### TCP 通信三大步骤

1. 三次握手建立连接
2. 开始通信，交换数据
3. 四次挥手断开连接

#### 三次握手

定义套接字 A为客户端，定义套接字 B 为服务端

【第一次握手】套接字 A∶"你好，套接字 B。我这儿有数据要传给你，建立连接吧。"

【第二次握手】套接字 B∶"好的，我这边已就绪。"

【第三次握手】套接字 A∶"谢谢你受理我的请求。"

<img src="F:\我的\博客记录\linux\image\图片1.png" alt="图片1" style="zoom:80%;" />

首先，请求连接的主机 A 向主机 B 传递如下信息∶ 

**[SYN] SEQ:1000, ACK: -** 

该消息中 SEQ 为 1000，ACK 为空，而 SEQ 为 1000 的含义如下∶ 

"现传递的数据包序号为 1000，如果接收无误，请通知我向您传递 1001 号数据包。"这是首 次请求连接时使用的消息，又称 SYN。SYN 是 **Synchronization** 的简写，表示收发数据前传输 的同步消息。 

接下来主机 B 向 A 传递如下消息∶ 

**[SYN+ACK]SEQ:2000, ACK:1001**

 此时 SEQ 为 2000，ACK 为 1001，而 SEQ 为 2000 的含义如下∶ "现传递的数据包序号为 2000 如果接收无误，请通知我向您传递 2001 号数据包。" 而 ACK1001 的含义如下∶ **"刚才传输的 SEQ 为 1000 的数据包接收无误，现在请传递 SEQ 为 1001 的数据包。"** 

对主机 A 首次传输的数据包的确认消息（ACK1001）和为主机 B 传输数据做准备的同步消息 （SEQ2000）拥绑发送，因此，此种类型的消息又称 SYN+ACK。 

**收发数据前向数据包分配序号，并向对方通报此序号，这都是为防止数据丢失所做的准备。** 

通过向数据包分配序号并确认，可以在数据丢失时马上查看并重传丢失的数据包。因此，TCP 可以保证可靠的数据传输。最后观察主机 A 向主机 B 传输的消息∶ 

**[ACK]SEQ:1001, ACK:2001** 

TCP 连接过程中发送数据包时需分配序号。 

在之前的序号 1000 的基础上加 1，也就是分配 1001。此时该数据包传递如下消息∶ 

"已正确收到传输的 SEQ 为 2000 的数据包，现在可以传输 SEQ 为 2001 的数据包。" 

这样就传输了添加 ACK 2001 的 ACK 消息。至此，主机 A 和主机 B 确认了彼此均就绪。

### 四次挥手

![图片2](F:\我的\博客记录\linux\image\图片2.png)

挥手过程： 

套接字 A∶"我希望断开连接。" 

套接字 B∶"哦，是吗?请稍候。" 

套接字 B∶"我也准备就绪，可以断开连接。" 

套接字 A∶"好的，谢谢合作。"