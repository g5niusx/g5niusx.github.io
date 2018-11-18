---
layout: post
title: 'Netty 服务端初步使用'
tags: [code]
---

### 什么是Netty
Netty是一个开源的网络编程框架，netty封装了java的nio，简化了java网络编程。[netty官网](https://netty.io/)。可以让开发者快速的上手网络编程

### 最简单的一个例子
`本例子中会使用netty实现可以接收请求并且响应消息的服务端`

#### 创建服务端
- 启动类

```java
package com.java.netty.simple.demo;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.extern.slf4j.Slf4j;

/**
 * 服务端
 */
@Slf4j
public class SimpleServer {

    private static final int PORT = 9999;

    public static void main(String[] args) throws InterruptedException {
        // 创建监听线程，这个线程池会用来监听连接到客户端的连接，
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        // 创建工作线程，用来处理具体的请求
        NioEventLoopGroup workEventLoopGroup = new NioEventLoopGroup();
        // 创建服务端的引导
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        ChannelFuture sync = serverBootstrap.group(nioEventLoopGroup, workEventLoopGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new SimpleServerHandler());
                    }
                })
                .bind(PORT).addListener(future -> {
                    if (future.isSuccess()) {
                        log.info("端口{}绑定成功", PORT);
                    }
                }).sync();
        sync.channel().closeFuture().sync();
    }
}
```
- 服务端处理类

```java
package com.java.netty.simple.demo;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import lombok.extern.slf4j.Slf4j;

import static com.java.netty.Constants.UTF_8;

/**
 * 服务端处理
 */
@Slf4j
public class SimpleServerHandler extends SimpleChannelInboundHandler {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 在不加管道的情况下，默认的对象都是ByteBuf
        ByteBuf byteBuf = (ByteBuf) msg;
        log.info("服务器接收到:{}", byteBuf.toString(UTF_8));
    }
}
```
- 测试服务是否正常
使用`telnet`命令可以测试一个端口是否有应用活动，运行`main`方法，结果如下

![应用启动]({{ "/public/images/netty/2018-11-14-netty-server-1.png"}} "应用启动")
![telnet测试]({{ "/public/images/netty/2018-11-14-netty-server-2.png"}} "telnet测试")
![服务器结果]({{ "/public/images/netty/2018-11-14-netty-server-3.png"}} "服务器结果")

可以看到，服务端已经接收到了我们输入的内容。这就是使用netty创建的一个最简单的服务端,只需要十几行代码就可以实现网络编程的基础功能。

源码地址: [netty-demo](https://github.com/g5niusx/netty-demo/tree/master/src/main/java/com/java/netty/simple/demo)


