## select，poll，epoll，iocp
在Linux下并发网络程序,Apache模型（Process Per Connection,简称PPC),TPC（Thread PerConnection）模型,select模型和poll模型及epoll模型

###  2. 常用模型的缺点
####  2.1  PPC/TPC模型
          PPC:每个连接建立一个进程
          TPC:每个连接建立一个线程
          缺点:进程之间切换和线程之间切换比较耗费资源,最大连接数不太高,一般在几百个左右。

####  2.2 select模型
          1. 最大并发数限制:进程打开的FD文件描述符，由FD_SETSIZE设置，默认值是1024/2048
          2. 是通过推打开的FD进行循环扫描,效率不高
          3. 内核空间需要将消息通知给用户空间,存在内存拷贝的问题
####  2.3 poll模型
          与select类同,不过没有select的1的问题
####  3.  epoll模型
          3.1. 最大连接上限是进程最大可以打开文件的数目,系统内存关系很大，可以cat /proc/sys/fs/file-max察看。
          3.2. 时间驱动,只管“活跃”的连接,与连接总数无关,效率高。
          3.3. 使用“共享内存”，不存在内存拷贝的问题。
### 3.  epoll使用详解
        epoll - I/O 时间驱动能力
        3.1 创建epoll句柄:int epoll_create(int size);
              size用来告诉内核这个监听的数目一共有多大。
              注意:创建epoll句柄后，它就是会占用一个fd值,使用完epoll后,必须调用close()关闭，否则可能导致fd被耗尽。
        3.2 epoll事件注册:int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event); 
              第一个参数是epoll_create()的返回值
              第二个参数表示动作，用三个宏来表示：
                    EPOLL_CTL_ADD：注册新的fd到epfd中；
                    EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
                    EPOLL_CTL_DEL：从epfd中删除一个fd；
              第三个参数是需要监听的fd，第四个参数是告诉内核需要监听什么事，struct epoll_event结构如下：

              typedef union epoll_data {
                  void *ptr;
                  int fd;
                  __uint32_t u32;
                  __uint64_t u64;
              } epoll_data_t;

              struct epoll_event {
                  __uint32_t events; /* Epoll events */
                  epoll_data_t data; /* User data variable */
              };

              events可以是以下几个宏的集合：
              EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
              EPOLLOUT：表示对应的文件描述符可以写；
              EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
              EPOLLERR：表示对应的文件描述符发生错误；
              EPOLLHUP：表示对应的文件描述符被挂断；
              EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
              EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
       3.3 等待事件:int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
              参数events用来从内核得到事件的集合
              maxevents告之内核这个events有多大，这个 maxevents的值不能大于创建epoll_create()时的size
              参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有 说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。


### 4、关于ET、LT两种工作模式：
    ET模式仅当状态发生变化的时候才获得通知,状态的变化并不包括缓冲区中还有未处理的数据,需要一直read/write直到出错为止,有可能只接收了一部分数据就再也得不到通知;
    LT模式是只要有数据没有处理就会一直通知下去的.


### 5.  使用epoll
        1 包含头文件#include <sys/epoll.h>
        2 通过create_epoll(int maxfds)来创建一个epoll的句柄,在用完之后,记得用close()来关闭。
        3 主循环里面nfds = epoll_wait(kdpfd, events, maxevents, -1);

        for( ; ; )
        {
            nfds = epoll_wait(epfd,events,20,500);
            for(i=0;i<nfds;++i)
            {
                if(events[i].data.fd==listenfd) //有新的连接
                {
                    connfd = accept(listenfd,(sockaddr *)&clientaddr, &clilen); //accept这个连接
                    ev.data.fd=connfd;
                    ev.events=EPOLLIN|EPOLLET;
                    epoll_ctl(epfd,EPOLL_CTL_ADD,connfd,&ev); //将新的fd添加到epoll的监听队列中
                }
                else if( events[i].events&EPOLLIN ) //接收到数据，读socket
                {
                    n = read(sockfd, line, MAXLINE)) < 0    //读
                    ev.data.ptr = md;     //md为自定义类型，添加数据
                    ev.events=EPOLLOUT|EPOLLET;
                    epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);//修改标识符，等待下一个循环时发送数据，异步处理的精髓
                }
                else if(events[i].events&EPOLLOUT) //有数据待发送，写socket
                {
                    struct myepoll_data* md = (myepoll_data*)events[i].data.ptr;    //取数据
                    sockfd = md->fd;
                    send( sockfd, md->ptr, strlen((char*)md->ptr), 0 );        //发送数据
                    ev.data.fd=sockfd;
                    ev.events=EPOLLIN|EPOLLET;
                    epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev); //修改标识符，等待下一个循环时接收数据
                }
                else
                {
                    //其他的处理
                }
            }
        }

服务器端例子：

#include <iostream>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

using namespace std;

#define MAXLINE 5
#define OPEN_MAX 100
#define LISTENQ 20
#define SERV_PORT 5000
#define INFTIM 1000

void setnonblocking(int sock)
{
    int opts;
    opts=fcntl(sock,F_GETFL);
    if(opts<0)
    {
        perror("fcntl(sock,GETFL)");
        exit(1);
    }
    opts = opts|O_NONBLOCK;
    if(fcntl(sock,F_SETFL,opts)<0)
    {
        perror("fcntl(sock,SETFL,opts)");
        exit(1);
    }
}

int main(int argc, char* argv[])
{
    int i, maxi, listenfd, connfd, sockfd,epfd,nfds, portnumber;
    ssize_t n;
    char line[MAXLINE];
    socklen_t clilen;


    if ( 2 == argc )
    {
        if( (portnumber = atoi(argv[1])) < 0 )
        {
            fprintf(stderr,"Usage:%s portnumber/a/n",argv[0]);
            return 1;
        }
    }
    else
    {
        fprintf(stderr,"Usage:%s portnumber/a/n",argv[0]);
        return 1;
    }



    //声明epoll_event结构体的变量,ev用于注册事件,数组用于回传要处理的事件

    struct epoll_event ev,events[20];
    //生成用于处理accept的epoll专用的文件描述符

    epfd=epoll_create(256);
    struct sockaddr_in clientaddr;
    struct sockaddr_in serveraddr;
    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    //把socket设置为非阻塞方式

    //setnonblocking(listenfd);

    //设置与要处理的事件相关的文件描述符

    ev.data.fd=listenfd;
    //设置要处理的事件类型

    ev.events=EPOLLIN|EPOLLET;
    //ev.events=EPOLLIN;

    //注册epoll事件

    epoll_ctl(epfd,EPOLL_CTL_ADD,listenfd,&ev);
    bzero(&serveraddr, sizeof(serveraddr));
    serveraddr.sin_family = AF_INET;
    char *local_addr="127.0.0.1";
    inet_aton(local_addr,&(serveraddr.sin_addr));//htons(portnumber);

    serveraddr.sin_port=htons(portnumber);
    bind(listenfd,(sockaddr *)&serveraddr, sizeof(serveraddr));
    listen(listenfd, LISTENQ);
    maxi = 0;
    for ( ; ; ) {
        //等待epoll事件的发生

        nfds=epoll_wait(epfd,events,20,500);
        //处理所发生的所有事件

        for(i=0;i<nfds;++i)
        {
            if(events[i].data.fd==listenfd)//如果新监测到一个SOCKET用户连接到了绑定的SOCKET端口，建立新的连接。

            {
                connfd = accept(listenfd,(sockaddr *)&clientaddr, &clilen);
                if(connfd<0){
                    perror("connfd<0");
                    exit(1);
                }
                //setnonblocking(connfd);

                char *str = inet_ntoa(clientaddr.sin_addr);
                cout << "accapt a connection from " << str << endl;
                //设置用于读操作的文件描述符

                ev.data.fd=connfd;
                //设置用于注测的读操作事件

                ev.events=EPOLLIN|EPOLLET;
                //ev.events=EPOLLIN;

                //注册ev

                epoll_ctl(epfd,EPOLL_CTL_ADD,connfd,&ev);
            }
            else if(events[i].events&EPOLLIN)//如果是已经连接的用户，并且收到数据，那么进行读入。

            {
                cout << "EPOLLIN" << endl;
                if ( (sockfd = events[i].data.fd) < 0)
                    continue;
                if ( (n = read(sockfd, line, MAXLINE)) < 0) {
                    if (errno == ECONNRESET) {
                        close(sockfd);
                        events[i].data.fd = -1;
                    } else
                        std::cout<<"readline error"<<std::endl;
                } else if (n == 0) {
                    close(sockfd);
                    events[i].data.fd = -1;
                }
                line[n] = '/0';
                cout << "read " << line << endl;
                //设置用于写操作的文件描述符

                ev.data.fd=sockfd;
                //设置用于注测的写操作事件

                ev.events=EPOLLOUT|EPOLLET;
                //修改sockfd上要处理的事件为EPOLLOUT

                //epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);

            }
            else if(events[i].events&EPOLLOUT) // 如果有数据发送

            {
                sockfd = events[i].data.fd;
                write(sockfd, line, n);
                //设置用于读操作的文件描述符

                ev.data.fd=sockfd;
                //设置用于注测的读操作事件

                ev.events=EPOLLIN|EPOLLET;
                //修改sockfd上要处理的事件为EPOLIN

                epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);
            }
        }
    }
    return 0;
}

Epoll在LT和ET模式下的读写方式
在一个非阻塞的socket上调用read/write函数, 返回EAGAIN或者EWOULDBLOCK(注: EAGAIN就是EWOULDBLOCK)
从字面上看, 意思是:EAGAIN: 再试一次，EWOULDBLOCK: 如果这是一个阻塞socket, 操作将被block，perror输出: Resource temporarily unavailable

总结:
这个错误表示资源暂时不够，能read时，读缓冲区没有数据，或者write时，写缓冲区满了。遇到这种情况，如果是阻塞socket，read/write就要阻塞掉。而如果是非阻塞socket，read/write立即返回-1， 同时errno设置为EAGAIN。
所以，对于阻塞socket，read/write返回-1代表网络出错了。但对于非阻塞socket，read/write返回-1不一定网络真的出错了。可能是Resource temporarily unavailable。这时你应该再试，直到Resource available。

综上，对于non-blocking的socket，正确的读写操作为:
读：忽略掉errno = EAGAIN的错误，下次继续读
写：忽略掉errno = EAGAIN的错误，下次继续写

对于select和epoll的LT模式，这种读写方式是没有问题的。但对于epoll的ET模式，这种方式还有漏洞。

epoll的两种模式LT和ET
二者的差异在于level-trigger模式下只要某个socket处于readable/writable状态，无论什么时候进行epoll_wait都会返回该socket；而edge-trigger模式下只有某个socket从unreadable变为readable或从unwritable变为writable时，epoll_wait才会返回该socket。

所以，在epoll的ET模式下，正确的读写方式为:
读：只要可读，就一直读，直到返回0，或者 errno = EAGAIN
写:只要可写，就一直写，直到数据发送完，或者 errno = EAGAIN

正确的读

n = 0;
while ((nread = read(fd, buf + n, BUFSIZ-1)) > 0) {
n += nread;
}
if (nread == -1 && errno != EAGAIN) {
perror("read error");
}
正确的写

int nwrite, data_size = strlen(buf);
n = data_size;
while (n > 0) {
nwrite = write(fd, buf + data_size - n, n);
if (nwrite < n) {
if (nwrite == -1 && errno != EAGAIN) {
perror("write error");
}
break;
}
n -= nwrite;
}
正确的accept，accept 要考虑 2 个问题
(1) 阻塞模式 accept 存在的问题
考虑这种情况：TCP连接被客户端夭折，即在服务器调用accept之前，客户端主动发送RST终止连接，导致刚刚建立的连接从就绪队列中移出，如果套接口被设置成阻塞模式，服务器就会一直阻塞在accept调用上，直到其他某个客户建立一个新的连接为止。但是在此期间，服务器单纯地阻塞在accept调用上，就绪队列中的其他描述符都得不到处理。

解决办法是把监听套接口设置为非阻塞，当客户在服务器调用accept之前中止某个连接时，accept调用可以立即返回-1，这时源自Berkeley的实现会在内核中处理该事件，并不会将该事件通知给epool，而其他实现把errno设置为ECONNABORTED或者EPROTO错误，我们应该忽略这两个错误。

(2)ET模式下accept存在的问题
考虑这种情况：多个连接同时到达，服务器的TCP就绪队列瞬间积累多个就绪连接，由于是边缘触发模式，epoll只会通知一次，accept只处理一个连接，导致TCP就绪队列中剩下的连接都得不到处理。

解决办法是用while循环抱住accept调用，处理完TCP就绪队列中的所有连接后再退出循环。如何知道是否处理完就绪队列中的所有连接呢？accept返回-1并且errno设置为EAGAIN就表示所有连接都处理完。

综合以上两种情况，服务器应该使用非阻塞地accept，accept在ET模式下的正确使用方式为：

while ((conn_sock = accept(listenfd,(struct sockaddr *) &remote, (size_t *)&addrlen)) > 0) {
handle_client(conn_sock);
}
if (conn_sock == -1) {
if (errno != EAGAIN && errno != ECONNABORTED && errno != EPROTO && errno != EINTR)
perror("accept");
}
一道腾讯后台开发的面试题
使用Linuxepoll模型，水平触发模式；当socket可写时，会不停的触发socket可写的事件，如何处理？

第一种最普遍的方式：
需要向socket写数据的时候才把socket加入epoll，等待可写事件。
接受到可写事件后，调用write或者send发送数据。
当所有数据都写完后，把socket移出epoll。

这种方式的缺点是，即使发送很少的数据，也要把socket加入epoll，写完后在移出epoll，有一定操作代价。

一种改进的方式：
开始不把socket加入epoll，需要向socket写数据的时候，直接调用write或者send发送数据。如果返回EAGAIN，把socket加入epoll，在epoll的驱动下写数据，全部数据发送完毕后，再移出epoll。

这种方式的优点是：数据不多的时候可以避免epoll的事件处理，提高效率。

最后贴一个使用epoll,ET模式的简单HTTP服务器代码:

#include <sys/socket.h>
#include <sys/wait.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <sys/epoll.h>
#include <sys/sendfile.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <strings.h>
#include <fcntl.h>
#include <errno.h>
 
#define MAX_EVENTS 10
#define PORT 8080
 
//设置socket连接为非阻塞模式
void setnonblocking(int sockfd) {
int opts;
 
opts = fcntl(sockfd, F_GETFL);
if(opts < 0) {
perror("fcntl(F_GETFL)n");
exit(1);
}
opts = (opts | O_NONBLOCK);
if(fcntl(sockfd, F_SETFL, opts) < 0) {
perror("fcntl(F_SETFL)n");
exit(1);
}
}
 
int main(){
struct epoll_event ev, events[MAX_EVENTS];
int addrlen, listenfd, conn_sock, nfds, epfd, fd, i, nread, n;
struct sockaddr_in local, remote;
char buf[BUFSIZ];
 
//创建listen socket
if( (listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
perror("sockfdn");
exit(1);
}
setnonblocking(listenfd);
bzero(&local, sizeof(local));
local.sin_family = AF_INET;
local.sin_addr.s_addr = htonl(INADDR_ANY);;
local.sin_port = htons(PORT);
if( bind(listenfd, (struct sockaddr *) &local, sizeof(local)) < 0) {
perror("bindn");
exit(1);
}
listen(listenfd, 20);
 
epfd = epoll_create(MAX_EVENTS);
if (epfd == -1) {
perror("epoll_create");
exit(EXIT_FAILURE);
}
 
ev.events = EPOLLIN;
ev.data.fd = listenfd;
if (epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &ev) == -1) {
perror("epoll_ctl: listen_sock");
exit(EXIT_FAILURE);
}
 
for (;;) {
nfds = epoll_wait(epfd, events, MAX_EVENTS, -1);
if (nfds == -1) {
perror("epoll_pwait");
exit(EXIT_FAILURE);
}
 
for (i = 0; i < nfds; ++i) {
fd = events[i].data.fd;
if (fd == listenfd) {
while ((conn_sock = accept(listenfd,(struct sockaddr *) &remote,
(size_t *)&addrlen)) > 0) {
setnonblocking(conn_sock);
ev.events = EPOLLIN | EPOLLET;
ev.data.fd = conn_sock;
if (epoll_ctl(epfd, EPOLL_CTL_ADD, conn_sock,
&ev) == -1) {
perror("epoll_ctl: add");
exit(EXIT_FAILURE);
}
}
if (conn_sock == -1) {
if (errno != EAGAIN && errno != ECONNABORTED
&& errno != EPROTO && errno != EINTR)
perror("accept");
}
continue;
}
if (events[i].events & EPOLLIN) {
n = 0;
while ((nread = read(fd, buf + n, BUFSIZ-1)) > 0) {
n += nread;
}
if (nread == -1 && errno != EAGAIN) {
perror("read error");
}
ev.data.fd = fd;
ev.events = events[i].events | EPOLLOUT;
if (epoll_ctl(epfd, EPOLL_CTL_MOD, fd, &ev) == -1) {
perror("epoll_ctl: mod");
}
}
if (events[i].events & EPOLLOUT) {
sprintf(buf, "HTTP/1.1 200 OKrnContent-Length: %drnrnHello World", 11);
int nwrite, data_size = strlen(buf);
n = data_size;
while (n > 0) {
nwrite = write(fd, buf + data_size - n, n);
if (nwrite < n) {
if (nwrite == -1 && errno != EAGAIN) {
perror("write error");
}
break;
}
n -= nwrite;
}
close(fd);
}
}
}
 
return 0;
}
