---
title: WebFlux响应式编程
date: 2022-02-07 10:04:02
categories: 分布式技术
tags: Java
---

# 概要

在SpringMvc框架下，http的请求是同步的，在某些场景下为了提供性能，可以采用异步的方式来进行优化。WebFlux便是提供了异步的技术栈
<!--more-->  
响应式编程（reactive programming）是一种基于数据流（data stream）和变化传递（propagation of change）的声明式（declarative）的编程范式

WebFlux经常会跟响应式编程挂钩，其实其只不过是响应式编程中的一部分技术，代表的是web控制端实现异步从而实现状态变化流转。

# 入门WebFlux
WebFlux使用的响应式流是一个叫做`Reactor响应式流库`。所以，入门WebFlux其实更多是了解怎么使用Reactor的API，下面我们来看看~

Reactor是一个响应式流，它也有对应的发布者(Publisher )，Reactor的发布者用两个类来表示：

* Mono(返回0或1个元素)
* Flux(返回0-n个元素)

而订阅者则是Spring框架去完成

下面我们来看一个简单的例子(基于WebFlux环境构建)：
```java
@EnableDiscoveryClient
@SpringBootApplication
public class WebFluxClient {
    public static void main( String[] args )
    {
        SpringApplication.run(WebFluxClient.class, args);
    }
}

@RestController
@RequestMapping("/WebFluxClient")
public class WebFluxClientController {

    @Autowired
    WebFluxClientService service;

    @GetMapping("/{id}")
    public Mono<String> getById(@PathVariable long id) {
        return service.getOrder(id);
    }
}

public class WebFluxClientService {
    WebClient webClient = WebClient.create();

    private <T> Mono<T> getMono(String url, Class<T> resType) {
        return webClient.get().uri(url).retrieve().bodyToMono(resType);
    }

    public Mono<String> getOrder(long id) {

            Mono<String> userMono =  getMono("http://user-service/user/mock/",String.class).onErrorReturn(new String());

            return userMono;
    }
}

```
pom引入webflux依赖
```java
 <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

构建WebFlux环境启动时，应用服务器默认是Netty的,有别于SpringMvc采用的Servlet容器运行在tomcat中。

相比于SpringMvc同步阻塞，WebFlux返回的是Mono，这也正是WebFlux的优势可以固定线程，而不需要每个请求都新开线程造成线程开销。

* 如果适用WebFlux那么即不能引入spring-boot-starter-web


## WebClient
生产中，我们使用WebClient来发送异步请求，并支持响应式编程。

WebClient通过调用uri将返回的Mono数据转成我们的接收类型，而其uri的访问可以请求Eureka上注册的服务


以下为WebClient的一些配置

底层框架
WebClient底层使用的Netty实现异步Http请求，我们可以切换底层库，如Jetty
```java
@Bean
public JettyResourceFactory resourceFactory() {
    return new JettyResourceFactory();
}

@Bean
public WebClient webClient() {
    HttpClient httpClient = HttpClient.create();
    ClientHttpConnector connector =
            new JettyClientHttpConnector(httpClient, resourceFactory());
    return WebClient.builder().clientConnector(connector).build();
}
```
连接池
WebClient默认是每个请求创建一个连接。
我们可以配置连接池复用连接，以提高性能。
```java
ConnectionProvider provider = ConnectionProvider.builder("order")
    .maxConnections(100)
    .maxIdleTime(Duration.ofSeconds(30))
    .pendingAcquireTimeout(Duration.ofMillis(100))  
    .build();
return WebClient
    .builder().clientConnector(new ReactorClientHttpConnector(HttpClient.create(provider)));
```
maxConnections：允许的最大连接数
pendingAcquireTimeout：没有连接可用时，请求等待的最长时间
maxIdleTime：连接最大闲置时间

