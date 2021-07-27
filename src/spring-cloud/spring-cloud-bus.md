# Spring Cloud Bus

- 当前版本：3.0.2
- 修改时间：2021年7月22日
- 官方文档：[https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/](https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-bus](https://github.com/spring-cloud/spring-cloud-bus)

Spring Cloud Bus将分布式系统的节点与一个轻量级的消息代理联系起来。然后，这可以用来广播状态变化（如配置变化）或其他管理指令。该项目包括AMQP和Kafka代理的实现。另外，在classpath上找到的任何Spring Cloud Stream绑定器都可以作为传输工具开箱即用。

## 1. 快速开始

如果Spring Cloud Bus在classpath上检测到自己，它就会通过添加Spring Boot自动配置来工作。要启用总线，请将`spring-cloud-starter-bus-amqp`或`spring-cloud-starter-bus-kafka`添加到您的依赖性管理中。Spring Cloud会处理剩下的事情。确保代理（RabbitMQ或Kafka）是可用的并已配置。当在本地主机上运行时，你不需要做任何事情。如果你远程运行，请使用Spring Cloud Connectors或Spring Boot约定来定义代理凭证，如下面Rabbit的例子所示。

application.yml

```yaml
spring:
  rabbitmq:
    host: mybroker.com
    port: 5672
    username: user
    password: secret
```

总线目前支持向所有监听的节点或某一特定服务的所有节点发送消息（由Eureka定义）。`/bus/*`执行器命名空间有一些HTTP端点。目前，有两个已经实现。第一个，`/bus/env`，发送键/值对以更新每个节点的Spring环境。第二个，`/bus/refresh`，重新加载每个应用程序的配置，就像它们都被ping到了`/refresh`端点一样。

> Spring Cloud Bus的启动程序涵盖了Rabbit和Kafka，因为这是最常见的两种实现方式。然而，Spring Cloud Stream是相当灵活的，而且该绑定器与`spring-cloud-bus`一起使用。

## 2. Actuator 端点

Spring Cloud Bus提供了两个端点，`/actuator/busrefresh`和`/actuator/busenv`，分别对应于Spring Cloud Commons中的单个执行器端点`/actuator/refresh`和`/actuator/env`。

### 2.1. Refresh Endpoint

`/actuator/busrefresh`端点清除`RefreshScope`缓存并重新绑定`@ConfigurationProperties`。更多信息请参见 [Refresh Scope](https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/#refresh-scope) 文档。

为了暴露`/actuator/busrefresh`端点，你需要向你的应用程序添加以下配置。

```properties
management.endpoints.web.exposure.include=busrefresh
```

### 2.2.  Env Endpoint

`/actuator/busenv`端点用指定的键/值对在多个实例中更新每个实例环境。

为了暴露`/actuator/busenv`端点，你需要在你的应用程序中添加以下配置。

```properties
management.endpoints.web.exposure.include=busenv
```

`/actuator/busenv`端点接受`POST`请求，请求体格式如下。

```json
{
    "name": "key1",
    "value": "value1"
}
```

## 3. 对一个实例进行寻址

应用程序的每个实例都有一个服务ID，其值可以用`spring.cloud.bus.id`来设置，其值预计是一个用冒号分隔的标识符列表，从最不具体到最具体的顺序。默认值从环境中构建，作为`spring.application.name`和`server.port`的组合（或`spring.application.index`，如果设置）。ID的默认值是以`app:index:id`的形式构建的，其中。

- `app`是`vcap.application.name`，如果它存在的话，或者是`spring.application.name`。
- `index`是`vcap.application.instance_index`，如果它存在的话，`spring.application.index`，`local.server.port`，`server.port`或`0`（按顺序排列）。
- `id`是`vcap.application.instance_id`，如果它存在，或者是一个随机值。

HTTP 端点接受一个 "destination" 路径参数，如`/busrefresh/customers:9000`，其中`目的地`是一个服务 ID。如果该ID为总线上的一个实例所拥有，它就会处理该消息，而所有其他实例都会忽略它。

## 4. 对一个服务的所有实例进行寻址

"destination"参数在Spring的`PathMatcher`中使用（路径分隔符为冒号 - `:` ），以确定一个实例是否处理该消息。使用前面的例子，`/busenv/customers:**`针对 "customers "服务的所有实例，而不管服务ID的其他部分。

## 5. Service ID 必须唯一

总线尝试两次来消除对一个事件的处理--一次来自原始的`ApplicationEvent`，一次来自队列。为此，它检查发送的服务ID和当前的服务ID。如果一个服务的多个实例有相同的ID，事件就不会被处理。在本地机器上运行时，每个服务都在一个不同的端口上，该端口是 ID 的一部分。Cloud Foundry 提供了一个索引来进行区分。为确保 ID 在 Cloud Foundry 之外是唯一的，请将 `spring.application.index` 设置为服务的每个实例的唯一内容。

## 6. 定制消息代理

Spring Cloud Bus使用[Spring Cloud Stream](https://cloud.spring.io/spring-cloud-stream)来广播消息。因此，为了让消息流动起来，你只需要在classpath中包含你选择的绑定器实现。对于使用AMQP（RabbitMQ）和Kafka（`spring-cloud-starter-bus-[amqp|kafka]`）的总线，有一些方便的启动器。一般来说，Spring Cloud Stream依靠Spring Boot的自动配置约定来配置中间件。例如，AMQP代理地址可以通过`spring.rabbitmq.*`配置属性来改变。Spring Cloud Bus在`spring.cloud.bus.*`中拥有少量的本地配置属性（例如，`spring.cloud.bus.destination`是用作外部中间件的主题名称）。通常情况下，默认值就足够了。

要了解更多关于如何定制消息代理的设置，请查阅Spring Cloud Stream文档。

## 7. 追踪总线事件

总线事件（`RemoteApplicationEvent`的子类）可以通过设置`spring.cloud.bus.trace.enabled=true`来追踪。如果你这样做，Spring Boot `TraceRepository`（如果它存在的话）会显示每个事件的发送情况和每个服务实例的所有acks。下面的例子来自`/trace`端点。

```json
{
  "timestamp": "2015-11-26T10:24:44.411+0000",
  "info": {
    "signal": "spring.cloud.bus.ack",
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "stores:8081",
    "destination": "*:**"
  }
  },
  {
  "timestamp": "2015-11-26T10:24:41.864+0000",
  "info": {
    "signal": "spring.cloud.bus.sent",
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "customers:9000",
    "destination": "*:**"
  }
  },
  {
  "timestamp": "2015-11-26T10:24:41.862+0000",
  "info": {
    "signal": "spring.cloud.bus.ack",
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "customers:9000",
    "destination": "*:**"
  }
}
```

前面的跟踪显示，一个 "RefreshRemoteApplicationEvent"从`customers:9000`发出，广播到所有服务，并由`customers:9000`和`stores:8081`接收（acked）。

要自己处理ack信号，你可以为`AckRemoteApplicationEvent`和`SentApplicationEvent`类型添加一个`@EventListener`到你的应用程序（并启用跟踪）。另外，你也可以进入`TraceRepository`，从那里挖掘数据。

> 任何总线应用都可以跟踪acks。然而，有时候，在一个中央服务中进行这项工作是很有用的，它可以对数据进行更复杂的查询，或者将其转发给专门的追踪服务。

## 8. 广播自定义事件

总线可以携带任何类型的`RemoteApplicationEvent`的事件。默认的传输方式是JSON，反序列化器需要提前知道哪些类型将被使用。要注册一个新的类型，你必须把它放在`org.springframework.cloud.bus.event`的一个子包中。

要自定义事件名称，你可以在你的自定义类上使用`@JsonTypeName`，或者依靠默认策略，即使用类的简单名称。

> 生产者和消费者都需要访问该类的定义。

### 8.1. 在自定义包中注册事件

如果你不能或不想为你的自定义事件使用`org.springframework.cloud.bus.event`的子包，你必须通过使用`@RemoteApplicationEventScan`注解指定哪些包来扫描`RemoteApplicationEvent`类型的事件。用`@RemoteApplicationEventScan`指定的包包括子包。

例如，考虑下面的自定义事件，称为`MyEvent`:

```java
package com.acme;

public class MyEvent extends RemoteApplicationEvent {
    ...
}
```

你可以通过以下方式向deserializer注册该事件:

```java
package com.acme;

@Configuration
@RemoteApplicationEventScan
public class BusConfiguration {
    ...
}
```

如果没有指定值，则使用`@RemoteApplicationEventScan`的类的包被注册。在这个例子中，`com.acme`是通过使用`BusConfiguration`的包注册的。

你也可以通过使用`@RemoteApplicationEventScan`上的`value`、`basePackages`或`basePackageClasses`属性明确指定要扫描的包，如下例所示。

```java
package com.acme;

@Configuration
//@RemoteApplicationEventScan({"com.acme", "foo.bar"})
//@RemoteApplicationEventScan(basePackages = {"com.acme", "foo.bar", "fizz.buzz"})
@RemoteApplicationEventScan(basePackageClasses = BusConfiguration.class)
public class BusConfiguration {
    ...
}
```

所有前面的`@RemoteApplicationEventScan`的例子都是等同的，即通过明确指定`@RemoteApplicationEventScan`上的包来注册`com.acme`包。

> 你可以指定多个base packages进行扫描

## 9. 配置属性

要查看所有与总线相关的配置属性列表，请查看[附录页面](https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/appendix.html)。

{{#include ../license.md}}
