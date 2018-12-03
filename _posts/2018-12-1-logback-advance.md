---
layout: post
title: 'Logback 进阶'
tags: [code]
---

- 异步写入文件

在生产环境中，经常会使用使用异步写入文件来提升系统的响应时间，所以为了提高性能，logback也支持异步写入文件的配置,只需要新增一个`appender`标签
同时将class的属性配置为`ch.qos.logback.classic.AsyncAppender`,这个表示是一个异步输出的appender，刚创建的这个appender里面没有申明输出格式，
也没有声明文件名称，需要我们配置一下格式和文件名称，logback提供来类似spring里面bean的引用的配置，可以直接使用`<appender-ref ref="appenderName"/>`这个标签来引用
已经声明过的appender,这样新创建的appender就包含了引用的appender的配置,具体配置内容可以参考下面

```xml
<configuration>
    <property name="FILE_PATH" value="/Users/g5niusx/Workspace/logback-demo/log"/>

    <!--申明一个appender，用来定义是控制台输出还是文件输出，以及输出日志的格式-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- %msg表示需要输出的内容 -->
            <!-- %n 表示换行-->
            <!-- %logger表示输出日志的名称-->
            <pattern>%d{YYYY-MM-DD HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE_APPENDER" class="ch.qos.logback.core.FileAppender">
        <!--文件输出路径-->
        <file>${FILE_PATH}/file.log</file>
        <!--是否追加内容，默认为true-->
        <append>false</append>
        <encoder>
            <pattern>%d{YYYY-MM-DD HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 异步输出的日志appender-->
    <appender name="SYNC_FILE_APPENDER" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="FILE_APPENDER"/>
    </appender>
    
    
    <!-- level表示需要输出的日志等级,常用的是INFO,DEBUG,ERROR-->
    <root level="INFO">
        <appender-ref ref="SYNC_FILE_APPENDER"/>
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```
- 文件自动分割

在日志文件过大的情况下，我们需要logback将这个日志文件自动打包成压缩包，以节约硬盘空间。
只需要将`ch.qos.logback.core.FileAppender`修改为`ch.qos.logback.core.rolling.RollingFileAppender`，同时增加`rollingPolicy`标签，这个标签是用来配置我们需要保留的日志的策略
一般都是配置为`TimeBasedRollingPolicy`表示根据时间来保留日志,同时可以使用 `fileNamePattern`,`maxHistory`等标签来配置文件名称，保留的记录数等


```xml
<configuration>
    <property name="FILE_PATH" value="/Users/g5niusx/Workspace/logback-demo/log"/>

    <!--申明一个appender，用来定义是控制台输出还是文件输出，以及输出日志的格式-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- %msg表示需要输出的内容 -->
            <!-- %n 表示换行-->
            <!-- %logger表示输出日志的名称-->
            <pattern>%d{YYYY-MM-DD HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--文件输出路径-->
        <file>${FILE_PATH}/file.log</file>
        <!--是否追加内容，默认为true-->
        <append>true</append>
        <encoder>
            <pattern>%d{YYYY-MM-DD HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <!--回滚策略,时间和文件大小回滚-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--设置备份的格式-->
            <fileNamePattern>${FILE_PATH}/file.%d{yyyy-MM-dd-HH-mm-ss}.gz</fileNamePattern>
            <!--保留的时间，以文件的格式来决定，如果文件格式最后一位为秒，则单位是秒-->
            <maxHistory>5</maxHistory>
        </rollingPolicy>
    </appender>

    <!-- 异步输出的日志appender-->
    <appender name="SYNC_FILE_APPENDER" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="FILE_APPENDER"/>
    </appender>

    <!-- level表示需要输出的日志等级,常用的是INFO,DEBUG,ERROR-->
    <root level="INFO">
        <appender-ref ref="SYNC_FILE_APPENDER"/>
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

常用的文件策略有 `TimeBasedRollingPolicy`按照时间保留,`FixedWindowRollingPolicy`按照文件大小保留,`SizeAndTimeBasedRollingPolicy`按照时间和文件大小保留等。

- 按照包名输出

logback支持不同不包名或者类名下生成不同的日志格式和不同的策略,只需要在logback.xml中增加`logger`标签，将name属性写成包名，引用需要输出的appender就可以完成不同包名的输出
下面的配置设置`com.g5niusx.log.sync`中输出的日志级别为`debug`,设置了`additivity="false"`表示日志不想上传递只会打印一次。`appender-ref`决定了这个包下面的日志输出的格式以及策略

```xml
<logger name="com.g5niusx.log.sync" level="DEBUG" additivity="false">
        <appender-ref ref="SYNC_FILE_APPENDER"/>
</logger>
```

最终的运行结果如下，生成了压缩文件，没有配置文件输出的包将日志输出到了控制台，并没有写入到文件中。

![最终结果]({{"/public/images/logback/2018-12-2-logback-result.png"}} "最终结果")


- 总结
logback的核心主要是`appender`和`logger`,appender相当于是一个可以重复使用的组件，同时这个组件是可以个性化配置的，我们可以在
appender里面决定内容输出到控制台还是文件，输出内容的格式以及文件的保留策略等。logger具体指定了使用哪一种appender。

[源码地址](https://github.com/g5niusx/logback-demo)




