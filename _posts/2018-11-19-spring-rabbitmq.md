---
layout: post
title: '使用rabbitmq来刷新多个应用的ehcache'
tags: [code]
---

### 业务背景

因为业务需要，对每台服务器上的ehcache都需要进行刷新，而且应用服务器使用来docker来部署，每次应用启动的时候，端口都会变动，通过System取到的ip是虚拟ip。
由于我们的项目已经集成来rabbitmq，所以选择使用rabbitmq来对每台服务器做广播，然后来触发消费者,所以设计了以下的流程图

![设计初步方案]({{ "/public/images/rabbitmq/2018-11-19-rabbitmq-1.png"}} "设计初步方案")

`可以看出，如果这个方案在服务器数量较小的情况下，可以接受，但是如果服务器数量扩增到几十台，会有几十个不同的queue`

- spring amqp的AnonymousQueue

通过对生成queue的时候，配置一个NameStrategy来生成一个随机的匿名队列，同时这个队列会在应用重启以后被销毁掉。
所以上面的方案被修改成，使用NameStrategy来生成一个随机的队列名称，然后注册到mq上，再由后台网页端发起刷新缓存的动作，在消费者里面做刷新缓存逻辑
只要应用不停止，这些queue一直都是可用。同时在停止应用以后，queue也会被主动删除。

#### spring boot配置

```java
@Bean
    public Queue test() {
        return new AnonymousQueue(new Base64UrlNamingStrategy("test"));
    }
```

### 总结
匿名队列在spring的集成中并不多见，但是在某些业务场景中可以达到很好的效果,在spring amqp的源码中`BrokerEventListener`就使用来匿名队列来传播事件；
如果你的项目需要这种临时性的队列，同时又需要申明很多个队列，可以考虑使用匿名队列(`AnonymousQueue`)来解决这种问题。