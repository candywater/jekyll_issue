---
layout: post
title: Load Balance
date: 2020-02-09 +0900
tags: nginx load-balancing
# category: tech
---

{% include toc.html %}

### LOAD BALANCE的概念

大致就是说同时利用多个CPU，达到更高效的目的。

这里涉及一个基本概念就是现代的NODE JS，或者是GO等高效率语言，都是倾向于单线程。比如NGINX也是一个单线程的（简单粗暴的讲，实质上更为复杂）。与之相对的是上一个时代的产品比如APACHE，就是一个多线程的服务器。

多线程的概念就是，每次处理一个事件，需要开启一个单独的线程(thread)。
而单线程是属于排队机制，基本上可以理解为JS里面的一个基础概念，就是异步编程(Asynchronous Programming Model)。

本质上看起来好像是没有优劣的样子，但是基于一方面是linux系统的线程有上限。一方面是即便不考虑上限，很明显看起来单线程的NGINX也要比APACHE速度快。（当然这里面有fast cgi等多方面因素，就不详细说明了）孰优孰劣市场会证明，编程者会证明，就不再赘述。

（这里补充说明一下，在物理层面，事实上所谓的多线程/进程在CPU层面也是单线程在跑，只不过是硬件在来回的不停切换。比如单CORE的CPU的话，如果有100个线程，那么在物理层面，是这个单CORE的CPU在不同的线程之间不同的切换，看起来好像是在同时运行。出处是自己过去看到的资料的记忆...）

那LOAD BALANCE就是基于这个看起来单线程的模式，进而产生的一个模型。

简单而言就是，一个CPU CORE配一个单独的子程序，让它可以跑满整个CORE的这种。
那么这个CPU有32个CORE，那就运行32个子程序。

这里只介绍基于NGINX的LOAD BALANCE。

### Load Balancing Methods

根据官方文档，有下面三种款式可以选择。

- round-robin 
> requests to the application servers are distributed in a round-robin fashion

- least-connected 
> next request is assigned to the server with the least number of active connections

- ip-hash 
> a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).

- round-robin 
  看起来是用round robin algorithm的样子，中文名称叫做轮训调度。基本可以简单理解为所有CORE轮一遍。
- least-connected 
  就是连接到目前接收req最少的core
- ip-hash 
  就是根据IP地址来hash一下，然后根据hash值来选择core

### Load Balancing Configuration

官网上有例子

```
http {
    upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

其中，在**upstream**区块中，不写method代表使用round-robin，
least_conn代表least-connected ，ip_hash代表ip-hash

### Weighted Load Balancing

可以在NGINX中配置权重。

官网中给出的code是这样的
```
    upstream myapp1 {
        server srv1.example.com weight=3;
        server srv2.example.com;
        server srv3.example.com;
    }
```

其中，默认权重为1，那么也就是说，来5个req的话，其中3个给**weight=3```的，剩下2个分配给没有配置```weight**的两个，一个server配一个。


### 除了NGINX的解决方案

比如nodejs有PM2。

但是可惜不知为什么我用PM2环境变量没办法在第一次开机的时候写进去。（在配置文件中写也没有用），所以我不得不放弃PM2这种简洁的方式，选择手写。

### 参考资料

- [Asynchronous Programming Model (APM)](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm)
- [异步编程](https://www.jianshu.com/p/c4dc7866eb81)
- [异步网络模型](https://tech.youzan.com/yi-bu-wang-luo-mo-xing/)
- [github: luocheng/twisted-intro-cn](https://github.com/luocheng/twisted-intro-cn)
- [How to configure load balancing using Nginx](https://upcloud.com/community/tutorials/configure-load-balancing-nginx/)
- [HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
- [Using nginx as HTTP load balancer](https://nginx.org/en/docs/http/load_balancing.html)

- [Round Robin](https://www.techopedia.com/definition/13205/round-robin)
- [wikipedia: Round-robin scheduling](https://en.wikipedia.org/wiki/Round-robin_scheduling)