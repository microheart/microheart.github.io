---
layout: post
title: Netty简介及Demo
category: Netty
tags: Netty
keywords: Netty
description:
---

## 简介
[Netty](http://netty.io/)是一个基于异步和事件驱动的开源网络框架，帮助开发者快速的实现高效能和可扩展的客户端/服务器端应用程序。
它广泛应用于各大互联网企业。github地址：https://github.com/netty/netty

## Netty主要特征

1. 简单而强大的线程模型
2. 多种传输模型统一接口
3. 比原始Java网络API简单易用
4. 健壮，安全
5. 减少内存拷贝，吞吐量高，延迟低
6. 强大的社区支持

Netty组件图(来源于)[Netty官网](http://netty.io/)

![](/public/upload/java/netty_components.png)

## Netty Echo Demo
### Demo简介
Echo（回声）案例，将客户端发送的字符串原样返回，此应用很简单，为了理解请求-响应交互本身。

### 服务器端

基于Netty的应用需要以下两个部分：

1. 服务器`Handler`，`Handler`组件用来实现服务器端业务逻辑，对消息进行处理
2. 基于`ServerBootstrap`引导服务器端启动代码。

例子来源于官方Demo

    @Sharable
    public class EchoServerHandler extends ChannelInboundHandlerAdapter {

        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) {
            ctx.write(msg);
        }

        @Override
        public void channelReadComplete(ChannelHandlerContext ctx) {
            ctx.flush();
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
            // Close the connection when an exception is raised.
            cause.printStackTrace();
            ctx.close();
        }
    }

`EchoServerHandler#channelRead()`将接收到的信息直接写入。
`EchoServerHandler#channelReadComplete()`刷新缓冲区。
`EchoServerHandler#exceptionCaught()`当异常发生后，关闭上下文。

**引导服务器**

    public final class EchoServer {

        static final boolean SSL = System.getProperty("ssl") != null;
        static final int PORT = Integer.parseInt(System.getProperty("port", "8007"));

        public static void main(String[] args) throws Exception {
            // Configure SSL.
            final SslContext sslCtx;
            if (SSL) {
                SelfSignedCertificate ssc = new SelfSignedCertificate();
                sslCtx = SslContextBuilder.forServer(ssc.certificate(), ssc.privateKey()).build();
            } else {
                sslCtx = null;
            }

            // Configure the server.
            EventLoopGroup bossGroup = new NioEventLoopGroup(1);
            EventLoopGroup workerGroup = new NioEventLoopGroup();
            try {
                ServerBootstrap b = new ServerBootstrap();
                b.group(bossGroup, workerGroup)
                        .channel(NioServerSocketChannel.class)
                        .option(ChannelOption.SO_BACKLOG, 100)
                        .handler(new LoggingHandler(LogLevel.INFO))
                        .childHandler(new ChannelInitializer<SocketChannel>() {
                            @Override
                            public void initChannel(SocketChannel ch) throws Exception {
                                ChannelPipeline p = ch.pipeline();
                                if (sslCtx != null) {
                                    p.addLast(sslCtx.newHandler(ch.alloc()));
                                }
                                //p.addLast(new LoggingHandler(LogLevel.INFO));
                                p.addLast(new EchoServerHandler());
                            }
                        });

                // Start the server.
                ChannelFuture f = b.bind(PORT).sync();

                // Wait until the server socket is closed.
                f.channel().closeFuture().sync();
            } finally {
                // Shut down all event loops to terminate all threads.
                bossGroup.shutdownGracefully();
                workerGroup.shutdownGracefully();
            }
        }
    }

`EchoServer`基于异步事件监听8007端口，基于`EchoServerHandler`处理客户端请求，类和方法的作用将在后续文章介绍。

为了更有效的观察和分析Netty的处理流程，在应用中配置log4j日志。

    log4j.rootLogger=DEBUG,Console,File
    log4j.appender.Console=org.apache.log4j.ConsoleAppender
    log4j.appender.Console.Target=System.out
    log4j.appender.Console.layout=org.apache.log4j.PatternLayout
    log4j.appender.Console.layout.ConversionPattern=[%d{yyyy-MM-dd HH\:mm\:ss}] [%t] [%p] [%c{2}.%M:%L] %m%n

    log4j.appender.File=org.apache.log4j.DailyRollingFileAppender
    log4j.appender.File.DatePattern='.'yyyy-MM-dd
    log4j.appender.File.Append=true
    log4j.appender.File.File=logs/netty.log
    log4j.appender.File.Threshold=INFO
    log4j.appender.File.layout=org.apache.log4j.PatternLayout
    log4j.appender.File.layout.ConversionPattern=[%d{yyyy-MM-dd HH\:mm\:ss}] [%t] [%p] [%c{2}.%M:%L] %m%n

运行`EchoServer#main()`

    [2017-03-06 16:32:29] [main] [DEBUG] [logging.InternalLoggerFactory.debug:71] Using SLF4J as the default logging framework
    [2017-03-06 16:32:29] [main] [DEBUG] [channel.MultithreadEventLoopGroup.debug:76] -Dio.netty.eventLoopThreads: 8
    ...
    [2017-03-06 16:32:30] [nioEventLoopGroup-2-1] [INFO] [logging.LoggingHandler.info:101] [id: 0x353e3e66] BIND: 0.0.0.0/0.0.0.0:8007
    [2017-03-06 16:32:30] [nioEventLoopGroup-2-1] [INFO] [logging.LoggingHandler.info:101] [id: 0x353e3e66, L:/0:0:0:0:0:0:0:0:8007] ACTIVE

处理客户端请求的NioEventLoopGroup的线程数为8，即当前可用CPU数量的2倍，绑定8007端口。


**测试**

telnet 8007端口

    > telnet localhost 8087


## 基于JMeter的性能测试

[JMeter](http://jmeter.apache.org)是一个开源的Java性能测试软件，它支持对多种协议的测试，如HTTP, TCP,FTP等。

根据需求，建立测试计划，如：

### 添加 'Thread Group'
![](/public/upload/java/jmeter_add_thread_group.png)

![](/public/upload/java/jmeter_group_detail.png)

线程数代表模拟的用户数。

### 添加 'Logic Controller'

![](/public/upload/java/jmeter_add_controller.png)

![](/public/upload/java/jmeter_loop_detail.png)

每个模拟用户发送20次请求

### 添加 'TCP Sampler'
![](/public/upload/java/jmeter_tcp_sample.png)

填写正确的地址，端口和发送的内容，10代表`\n`

### 添加 Timer
![](/public/upload/java/jmeter_timer_detail.png)

每间隔300ms发送一个请求。

### 添加 listener

添加Aggregate Report Listener监听结果。

![](/public/upload/java/jmeter_add_listener.png)

### 运行并查看结果
![](/public/upload/java/jmeter_result.png)

总共发送了2000次请求，错误率为0，90% Line在1ms以下响应请求，注1ms并不是平均数。

此外，Jmeter还支持分布式测试。