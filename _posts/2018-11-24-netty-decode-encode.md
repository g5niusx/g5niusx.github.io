---
layout: post
title: 'Netty 编码和解码'
tags: [code]
---

### Netty管道的处理流程

![示意图]({{ "/public/images/netty/2018-11-24-netty-piple-1.png"}} "示意图")

当客户端向服务端发起请求时,会先经过**客户端的编码器**,然后经过**服务端的解码器**,在服务端处理完请求以后，再经过**服务端的编码器**,客户端接受到
服务端的返回以后,先经过**客户端的解码器**然后再进入到客户端的流程里面。

netty里面内置的编码器和解码器都是可以被添加到管道里面的组件，因为他们都实现了`ChannelHandler`的接口


### StringEncoder和StringDecoder
`StringDecoder`是将ByteBuf接转换为字符串的编码器,在netty里面，编码器和解码器一般都是成对出现的，也就是说一个编码器和一个解码器是同时作用于客户端
和服务端的这样才能对消息编码和解码成功

- 客户端

```java
ch.pipeline().addLast(new StringDecoder(UTF_8));
ch.pipeline().addLast(new SimpleClientHandler());
ch.pipeline().addLast(new StringEncoder(UTF_8));
```

- 服务端

```java
ch.pipeline().addLast(new StringDecoder(UTF_8));
ch.pipeline().addLast(new SimpleServerHandler());
ch.pipeline().addLast(new StringEncoder(UTF_8));
```

### DelimiterBasedFrameDecoder
`DelimiterBasedFrameDecoder`是一个用于解决拆包和粘包的解码器，主要用于消息通过特殊字符来分割，比如换行符或者我们自己自定义的符号。

- 不使用定界符解码器
在不使用解码器的情况下，向服务端写入50条消息，会发现服务器当做一条消息处理
![不使用解码器]({{"/public/images/netty/2018-11-24-netty-piple-2.png"}} "不使用解码器")
- 使用定界符解码器
下面的代码是通过制表符`\t`来分割消息,然后解决来拆包来粘包,通过解码器的处理以后，我们的消息变成来50条消息，通过这样可以解决拆包和粘包的场景

```java
// 设置来制表符为特殊的分隔符
ByteBuf byteBuf = Unpooled.copiedBuffer("\t".getBytes(UTF_8));
ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, byteBuf));
ch.pipeline().addLast(new SimpleServerHandler());
```
![使用解码器]({{"/public/images/netty/2018-11-24-netty-piple-3.png"}} "使用解码器")

### FixedLengthFrameDecoder
`FixedLengthFrameDecoder`定长解码器，也是可以解决拆包和沾包的。主要是通过消息定长来解决网络传输的拆包和粘包问题。比如和服务端约定，消息定长为2048
这种解码器就适用于这种场景
源码地址: [netty-demo](https://github.com/g5niusx/netty-demo/tree/netty-demo-code)


