---
layout: post
title: 'Netty 客户端初步使用'
tags: [code]
---

### Netty客户端实现
`使用客户端向服务端发送消息,并且服务端向客户端应答消息`

### 客户端编码

- 客户端启动类

```java
package com.java.netty.simple.demo;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * 客户端
 *
 * @author g5niusx
 */
public class SimpleClient {

    private static final int    PORT = 9999;
    private static final String IP   = "127.0.0.1";


    public static void main(String[] args) throws InterruptedException {
        // 客户端不需要监听端口，只需要一个线程池来发送消息
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        // 创建一个引导器
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(nioEventLoopGroup)
                // 指定使用哪种channel来处理消息,分别有nio，oio等
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new SimpleClientHandler());
                    }
                })
                .connect(IP, PORT).channel().closeFuture().sync();

    }
}

```

- 客户端处理类

由于netty中管道的设计，使用`ChannelHandlerContext`发送消息的时候，传输的对象要和管道接收的对象一致，否则会导致消息发送失败，但是也不报错

```java
package com.java.netty.simple.demo;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import lombok.extern.slf4j.Slf4j;

import static java.nio.charset.StandardCharsets.UTF_8;


@ChannelHandler.Sharable
@Slf4j
public class SimpleClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        String message = "测试";
        // 由于没有使用任何的管道，所以向channel中写入数据的时候，要使用ByteBuf,否则会造成消息无法发送
        ByteBuf byteBuf = Unpooled.copiedBuffer(message, UTF_8);
        ctx.writeAndFlush(byteBuf);
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf byteBuf) throws Exception {
        log.info("客户端接收到: {}", byteBuf.toString(UTF_8));
    }
}

```

### 测试结果
![应用启动]({{ "/public/images/netty/2018-11-19-netty-client-1.png"}} "应用启动")

### 总结
通过上面的代码可以看到，使用netty就可以轻松的完成socket的基础编程,而且netty对于socket的封装很完善，让我们可以非常简便的使用它的api


源码地址: [netty-demo](https://github.com/g5niusx/netty-demo/tree/master/src/main/java/com/java/netty/simple/demo)


