## minihttpd
快速提供web服务(pure c)，采用epoll。

可以将自己的脚本放到该目录下，每次push，需要时直接clone或pull，就立即可以提供web服务。

## Use it ?

### start httpd
```
gcc -o httpd main.c
./httpd
```

## Update Log
### 2015.11.14
用c语言的完成的web server,采用select实现非阻塞,正在实现epoll.

可以根据get的路径返回html或者not found.

暂时没有处理get参数和get请求.

### 2015.11.16
支持epoll

### 2015.11.21
#### fix了一个`send_buffer`的bug:
默认发送缓冲区的buffer是8k字节，send函数仅仅将数据copy到发送缓冲区，如果采用while或者for会导致，发送缓冲区快速充满，因为采用非阻塞的socket会导致直接返回返回EAGAIN(-1)，处理方法就是记录ftell()或者sum(每次read志),重新丢入epoll，等待下次可写，然后根据上次读的位置定位文件指针，继续read.

#### 添加模块jmp2string
提供比较全面的编码解码。(目前在`encode img =>base64`有问题)

### 2015.11.23
#### 修复了EPOLL的ET模式问题。

ET模式，在于减少事件的频繁响应，只有当socket的状态change的时候，才会发出响应，所以`listen_fd`接受到accept请求，并不知道具体有几个并发socket请求，所以需要`while(accept())`直到`EAGAIN`的`errno`(读取也是类似).

PS：手动很难模拟并发，一个很类似并发的情况就是在一个html文件下面，引入多个本地js，这些js在加载的时候是并发去请求的。

###2015.12.04
#### 规格化返回状态

#### 按照非阻塞的方式处理request请求
记录header的处理状态，遇到EAGAIN保存状态，对于包含大量数据的请求头做了预处理。

#### 宏定义的合理使用
宏这真是个好东西。

用宏打印log`#define LOG(t) {printf("log[%d]..........\n",t);fflush(stdout);}`

用宏处理`#define free_buf(buf) if(buf){free(buf);buf=NULL;}`

### 2015.12.09

添加了一个简单的url路由模块

`read_line`:尽量不要在函数引用过多第三方元素，减少耦合

### 2015.12.13
重构，更加规范，并且引入**通用性List**

发现设计模式真的重要，之前一直在纠结整个处理流程应该怎么处理，然后琢磨出应该用状态码来表明步骤。过了一段时间，再一想，这特么不就是状态机么？！

在处理流程的过程中,case并没有接`continue;`(PS:如果`while+switch+continue`会不会更好)
```
int handle_request(int client_fd){
    int r;
    SocketNode *client_sock;
    client_sock=find_socket_node(SocketHead,client_fd);
    switch (client_sock->IO_STATUS) {
        case R_HEADER_INIT:
            printf("\0");
        case R_HEADER_START:
        {
            r=request_header_start(client_fd);
            if (r==IO_EAGAIN) {
                if (client_sock->request.method==M_ERROR)
                return IO_ERROR;
                return IO_EAGAIN;
            }
            else if(r==IO_ERROR)
            return IO_ERROR;
        }
        case R_HEADER_BODY:
        {
            r=request_header_body(client_fd);
            if (r==IO_EAGAIN) {
                if (client_sock->request.method==M_ERROR)
                return IO_ERROR;
                return IO_EAGAIN;
            }
            else if(r==IO_ERROR)
            return IO_ERROR;
        }
        case R_BODY:
        {
            r=request_body(client_fd);
            if (r==IO_EAGAIN||(client_sock->request.method==M_ERROR)) {
                return IO_EAGAIN;
            }
            else if(r==IO_ERROR)
            return IO_ERROR;
        }
        default:
            break;
    }
    return IO_DONE;
}
```

### 2015.12.18
fix bug:select的timeout每次会进行重置，所以需要在每次select之前进行重新设置


## 附录

#### Index
```
Index:http://xxx.xx/index.html
```

#### jsonp 一些xss脚本(from http://root.cool)
```
json:http://xxx.xx/jsonp.js
```

#### jmp2string 提供全面的encode/decode(modify http://faststring.lgvirtual.com/)
```
jmp2string http://xxx.xx/scripts/jmp2string/index.htm
```

#### csrf 收集的关于csrf的知识
```
csrf http://xxx.xx/scripts/csrf/csrf.form.html
```
#### tool.site.txt
```
tools http://xxx.xx/tool.site.txt
`
