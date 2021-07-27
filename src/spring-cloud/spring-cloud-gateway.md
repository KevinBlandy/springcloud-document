# Spring Cloud Gateway

- 当前版本：3.0.3
- 修改时间：2021年7月23日
- 官方文档：[https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-gateway](https://github.com/spring-cloud/spring-cloud-gateway)

这个项目提供了一个建立在Spring生态系统之上的API网关，包括。Spring 5、Spring Boot 2和Project Reactor。Spring Cloud Gateway旨在提供一种简单而有效的方式来路由到API，并提供跨领域的关注，如：安全、监控/指标和弹性。

## 1. 添加Spring Cloud Gateway

要在你的项目中包含Spring Cloud Gateway，请使用grup ID为`org.springframework.cloud`和artifact ID为`spring-cloud-starter-gateway`的starter。请参阅[Spring Cloud项目页面](https://projects.spring.io/spring-cloud/)，了解关于使用当前Spring Cloud发布列车设置构建系统的详细信息。

如果你包含了starter，但你不希望启用网关，请设置`spring.cloud.gateway.enabled=false`。

> Spring Cloud Gateway建立在[Spring Boot 2.x](https://spring.io/projects/spring-boot#learn)、[Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html)和[Project Reactor](https://projectreactor.io/docs)之上。因此，你所熟悉的许多同步库（例如Spring Data和Spring Security）和模式在你使用Spring Cloud Gateway时可能不适用。如果你不熟悉这些项目，我们建议你在使用Spring Cloud Gateway之前先阅读它们的文档，熟悉一些新的概念。

> Spring Cloud Gateway需要Spring Boot和Spring Webflux提供的Netty运行时间。它不能在传统的Servlet容器中工作，也不能以WAR的形式构建。

## 2. 术语表

- **Route**。网关的基本构造块。它由一个ID、一个目的地URI、一个谓词集合和一个过滤器集合定义。如果集合谓词为真，则路由被匹配。
- **Predicate**。这是一个[Java 8 Function Predicate](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)。输入类型是一个[Spring Framework `ServerWebExchange`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/server/ServerWebExchange.html)。这让你可以匹配HTTP请求中的任何内容，如头文件或参数。
- **Filter**。这些是[`GatewayFilter`](https://github.com/spring-cloud/spring-cloud-gateway/tree/main/spring-cloud-gateway-server/src/main/java/org/springframework/cloud/gateway/filter/GatewayFilter.java)的实例，已经用特定的工厂构建。在这里，你可以在发送下游请求之前或之后修改请求和响应。

## 3. 它是如何工作的

下图提供了一个关于Spring Cloud Gateway如何工作的高层次概述。

![springcloud](https://cdn.jsdelivr.net/gh/KevinBlandy/springcloud-images/2021/07/23/6026b5a0b1ad41a68dee61d1af1ff5d0.png)

客户端向Spring Cloud Gateway发出请求。如果Gateway处理程序映射确定一个请求与路由相匹配，它将被发送到Gateway Web处理程序。这个处理程序通过一个特定于该请求的过滤器链来运行该请求。过滤器被虚线划分的原因是，过滤器可以在代理请求发送之前和之后运行逻辑。所有的 "pre"过滤器逻辑都被执行。然后发出代理请求。在代理请求发出后，"post"过滤器逻辑被运行。

> 在路由中定义的没有端口的URI，其HTTP和HTTPS URI的默认端口值分别为80和443。

## 4. 配置 Route Predicate 和 Gateway Filter

有两种方式可以配置谓词和过滤器：快捷方式和完全展开参数。下面的大多数例子使用的是快捷方式。

名称和参数名称将在每个部分的第一或第二句中以`code`的形式列出。参数通常按照快捷方式配置所需的顺序列出。

### 4.1. 快捷配置

捷径配置是由过滤器名称识别的，后面是等号（`=`），后面是由逗号（`,`）分隔的参数值。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

前面的例子定义了`Cookie`路由谓词工厂，有两个参数，cookie名称`mycookie`和要匹配的值`mycookievalue`。

### 4.2. 完整展开的参数配置

完全展开的参数看起来更像标准的yaml配置，有name/vlue键值对。一般来说，会有一个`name`键和一个`args`键。`args`键是一个键值对的映射，用于配置谓词或过滤器。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

这是上面显示的`Cookie`谓词的快捷配置的完整配置。

## 5. 路由 Predicate 工厂

Spring Cloud Gateway将路由作为Spring WebFlux `HandlerMapping`基础设施的一部分进行匹配。Spring Cloud Gateway包括许多内置的路由谓词工厂。所有这些谓词都与HTTP请求的不同属性相匹配。你可以用逻辑上的 "and "语句组合多个路由谓词工厂。

### 5.1. The After Route Predicate Factory

"After" 路由谓词工厂需要一个参数，即 "datetime"（它是java的 "ZonedDateTime"）。这个谓词匹配发生在指定日期时间之后的请求。下面的例子配置了一个after路线谓词。

Example 1. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

请求的时间只有在 `2017-01-20T17:42:47.789-07:00[America/Denver]` 之后才会路由请求。

### 5.2. The Before Route Predicate Factory

"Before"路由谓词工厂需要一个参数，即 "datetime"（它是一个java的 "ZonedDateTime"）。这个谓词匹配发生在指定的`datetime`之前的请求。下面的例子配置了一个before路由谓词。

Example 2. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

请求的时间只有在 `2017-01-20T17:42:47.789-07:00[America/Denver]` 之前才会路由请求。

### 5.3. The Between Route Predicate Factory

`Between`路由谓词工厂需要两个参数，`datetime1`和`datetime2`，它们是java`ZonedDateTime`对象。这个谓词匹配发生在`datetime1`之后和`datetime2`之前的请求。参数`datetime2`必须在`datetime1`之后。下面的例子配置了一个路由谓词之间。

Example 3. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

这条路线匹配任何在`2017-01-20T17:42:47.789-07:00[America/Denver]`之后和`2017-01-21T17:42:47.789-07:00[America/Denver]`之前发起的请求。这对维护窗口可能是有用的。

### 5.4. The Cookie Route Predicate Factory

`Cookie'路由谓词工厂接受两个参数，即cookie`名称'和`regexp`（这是一个Java正则表达式）。这个谓词匹配具有给定名称且其值符合正则表达式的cookie。下面的例子配置了一个cookie路由谓词工厂。

Example 4. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

这个路由匹配有一个名为 "chocolate"的cookie的请求，其值与 "ch.p"正则表达式匹配。

### 5.5. The Header Route Predicate Factory

路由谓词工厂 "Header" 需要两个参数，header的 "name" 和 "regexp"（这是一个Java正则表达式）。这个谓词与具有给定名称且其值符合正则表达式的头匹配。下面的例子配置了一个hear的路由谓词。

Example 5. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

如果请求有一个名为`X-Request-Id`的头，其值与`d+`正则表达式相匹配（即，它有一个或多个数字的值），则该路由匹配。

### 5.6. The Host Route Predicate Factory

`Host`路由谓词工厂接受一个参数：一个主机名称`patterns`的列表。模式是一个Ant风格的模式，以`.`作为分隔符。这个谓词匹配符合该模式的`Host`头。下面的例子配置了一个主机路由谓词。

Example 6. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

也支持URI模板变量（如`{sub}.myhost.org`）。

如果请求有一个`Host`头，其值为`www.somehost.org`或`beta.somehost.org`或`www.anotherhost.org`，则该路由匹配。

这个谓词提取URI模板变量（如`sub`，在前面的例子中定义）作为名称和值的映射，并将其放在`ServerWebExchange.getAttributes()`中，其键定义在`ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE`。然后，这些值可供[`GatewayFilter` factories](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-route-filters) 使用。

### 5.7. The Method Route Predicate Factory

 "Method"路由谓词工厂接受一个 "methods"参数，这是一个或多个参数：要匹配的HTTP方法。下面的例子配置了一个方法路径谓词。

 Example 7. application.yml

 ```yaml
 spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
 ```

 如果请求方法是 "GET "或 "POST"，则该路由匹配。

### 5.8. The Path Route Predicate Factory

Path Route Predicate Factory需要两个参数：一个Spring `PathMatcher`模式的列表和一个可选的标志`matchTrailingSlash`（默认为`true`）。下面的例子配置了一个路径路由谓词。

Example 8. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

如果请求路径是，例如：`/red/1`或`/red/1/`或`/red/blue`或`/blue/green`，则该路径匹配。

如果`matchTrailingSlash`被设置为`false`，那么请求路径`/red/1/`将不被匹配。

这个谓词提取URI模板变量（如`segment`，在前面的例子中定义）作为名称和值的映射，并将其放在`ServerWebExchange.getAttributes()`中，其键定义在`ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE`。这些值然后可供[`GatewayFilter` factories](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-route-filters)使用。

有一个实用的方法（称为 "get"），以使访问这些变量更加容易。下面的例子显示了如何使用`get`方法。

```java
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");
```

### 5.9. The Query Route Predicate Factory

查询路由谓词工厂需要两个参数：一个必需的param和一个可选的regexp（这是一个Java正则表达式）。下面的例子配置了一个查询路由谓词。

Example 9. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=green
```

如果请求包含一个"green"的查询参数，前面的路由就会匹配。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```

如果请求中包含一个`red`的查询参数，其值与`gree.`的重合表达式相匹配，那么前面的路由就会匹配，所以`green`和`greet`会匹配。

### 5.10. The RemoteAddr Route Predicate Factory

RemoteAddr路由谓词工厂接受一个来源列表（最小尺寸为1），它是CIDR-注解（IPv4或IPv6）字符串，如192.168.0.1/16（其中192.168.0.1是一个IP地址，16是子网掩码）。下面的例子配置了一个RemoteAddr路由谓词。

Example 10. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

如果请求的远程地址是，例如，192.168.1.10，则该路由匹配。

### 5.11. The Weight Route Predicate Factory

Weight路线谓语工厂需要两个参数：group 和weight （一个int）。weight 是按group计算的。下面的例子配置了一个权重路由谓词。

Example 11. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

此路由将转发~80%的流量到 `weighthigh.org`，~20%的流量到`weightlow.org`。

#### 5.11.1. Modifying the Way Remote Addresses Are Resolved

默认情况下，RemoteAddr路由谓语工厂使用传入请求中的远程地址。如果Spring Cloud Gateway位于代理层后面，这可能与实际的客户端IP地址不一致。

你可以通过设置自定义的`RemoteAddressResolver`来定制远程地址的解析方式。Spring Cloud Gateway有一个非默认的远程地址解析器，基于[X-Forwarded-For header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)，`XForwardedRemoteAddressResolver`。

`XForwardedRemoteAddressResolver`有两个静态构造方法，它们采取不同的安全方法。

- `XForwardedRemoteAddressResolver::trustAll`返回一个`RemoteAddressResolver`，它总是采取在`X-Forwarded-For`头中发现的第一个IP地址。这种方法容易受到欺骗，因为恶意的客户可以为`X-Forwarded-For`设置一个初始值，这将被解析器所接受。
- `XForwardedRemoteAddressResolver::maxTrustedIndex`采取一个与Spring Cloud Gateway前面运行的受信任基础设施数量相关的索引。例如，如果Spring Cloud Gateway只能通过HAProxy访问，那么应该使用1的值。如果在Spring Cloud Gateway被访问之前需要两跳受信任的基础设施，那么应该使用2的值。

考虑一下下面的标头值。

```text
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3
```

以下`maxTrustedIndex`值产生以下远程地址。

|`maxTrustedIndex`|result|
| --- | --- |
|[ `Integer.MIN_VALUE` ,0]|(invalid, `IllegalArgumentException` during initialization)|
|1|0.0.0.3|
|2|0.0.0.2|
|3|0.0.0.1|
|[4, `Integer.MAX_VALUE` ]|0.0.0.1|

下面的例子显示了如何用Java实现同样的配置。

Example 12. GatewayConfig.java

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

## 6. GatewayFilter 工厂

路由过滤器允许以某种方式修改传入的 HTTP 请求或传出的 HTTP 响应。路由过滤器的范围是一个特定的路由。Spring Cloud Gateway包括许多内置的GatewayFilter Factories。

> 关于如何使用以下任何过滤器的更详细的例子，请看[单元测试](https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-server/src/test/java/org/springframework/cloud/gateway/filter/factory)。

### 6.1. The AddRequestHeader GatewayFilter Factory

`AddRequestHeader` `GatewayFilter`工厂需要一个`name`和`value`参数。下面的例子配置了一个`AddRequestHeader`的`GatewayFilter`。

Example 13. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```

这个列表将`X-Request-red:blue`头添加到所有匹配请求的下游请求的头文件中。

`AddRequestHeader`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时被扩展。下面的例子配置了一个`AddRequestHeader``GatewayFilter`，使用了一个变量。

Example 14. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

### 6.2. The AddRequestParameter GatewayFilter Factory

AddRequestParameter GatewayFilter Factory需要一个名称和值参数。下面的例子配置了一个AddRequestParameter GatewayFilter。

Example 15. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        filters:
        - AddRequestParameter=red, blue
```

这将为所有匹配的请求在下游请求的查询字符串中添加`red=blue`。

`AddRequestParameter`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时被扩展。下面的例子配置了一个`AddRequestParameter``GatewayFilter`，使用了一个变量。

Example 16. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```

### 6.3. The AddResponseHeader GatewayFilter Factory

`AddResponseHeader` `GatewayFilter`工厂需要一个`name`和`value`参数。下面的例子配置了一个`AddResponseHeader` `GatewayFilter`。

Example 17. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        filters:
        - AddResponseHeader=X-Response-Red, Blue
```

这将把`X-Response-Foo:Bar`头添加到所有匹配请求的下游响应的头文件中。

`AddResponseHeader`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时被扩展。下面的例子配置了一个`AddResponseHeader``GatewayFilter`，使用了一个变量。

Example 18. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddResponseHeader=foo, bar-{segment}
```

### 6.4. The DedupeResponseHeader GatewayFilter Factory

DedupeResponseHeader GatewayFilter工厂接收一个`name`参数和一个可选的`strategy`参数。`name`可以包含一个以空格分隔的头名称列表。下面的例子配置了一个`DedupeResponseHeader` `GatewayFilter`。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

在网关CORS逻辑和下游逻辑都添加了`Access-Control-Allow-Credentials`和`Access-Control-Allow-Origin`响应头的情况下，这将删除重复的值。

DedupeResponseHeader 过滤器还接受一个可选的 `strategy` 参数。接受的值是`RETAIN_FIRST`（默认），`RETAIN_LAST`，和`RETAIN_UNIQUE`。

### 6.5. Spring Cloud CircuitBreaker GatewayFilter Factory

Spring Cloud CircuitBreaker GatewayFilter工厂使用Spring Cloud CircuitBreaker APIs将Gateway路由包裹在一个断路器中。Spring Cloud CircuitBreaker支持多个可与Spring Cloud Gateway一起使用的库。Spring Cloud支持Resilience4J开箱即用。

要启用Spring Cloud CircuitBreaker过滤器，你需要将`spring-cloud-starter-circuitbreaker-reactor-resilience4j`放在classpath上。下面的例子配置了一个Spring Cloud CircuitBreaker `GatewayFilter`。

Example 20. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: https://example.org
        filters:
        - CircuitBreaker=myCircuitBreaker
```

要配置断路器，请参阅你所使用的底层断路器实现的配置。

- [Resilience4J Documentation](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html)

Spring Cloud CircuitBreaker过滤器也可以接受一个可选的`fallbackUri`参数。目前，只支持`forward:`模式化的URI。如果回退被调用，请求将被转发到URI所匹配的控制器。下面的例子配置了这样一个fallback。

Example 21. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
        - RewritePath=/consumingServiceEndpoint, /backingServiceEndpoint
```

下面的列表在Java中做同样的事情。

Example 22. Application.java

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

本例在调用断路器回退时转发到`/inCaseofFailureUseThis` URI。请注意，这个例子还演示了（可选）Spring Cloud LoadBalancer的负载均衡（由目标URI上的`lb`前缀定义）。

主要情况是使用`fallbackUri`来定义网关应用中的内部控制器或处理器。然而，你也可以将请求重新路由到外部应用程序的控制器或处理程序，如下所示。

Example 23. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

在这个例子中，网关应用程序中没有 `fallback` 端点或处理程序。然而，在另一个应用程序中有一个，在`localhost:9994`下注册。

在请求被转发到回退的情况下，Spring Cloud CircuitBreaker Gateway过滤器也提供了引起该请求的`Throwable`。它作为 `ServerWebExchangeUtils.CIRCUITBREAKER_EXECUTION_EXCEPTION_ATTR` 属性被添加到 `ServerWebExchange` 中，在网关应用程序中处理fallback时可以使用。

对于外部控制器/处理程序的情况，可以添加带有异常细节的头文件。你可以在[FallbackHeaders GatewayFilter Factory section](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#fallback-headers) 中找到更多关于这样做的信息。

#### 6.5.1. 状态码中的断路器熔断

在某些情况下，你可能想根据它所包裹的路由返回的状态代码来熔断。断路器配置对象需要一个状态代码列表，如果返回这些代码将导致断路器熔断。当设置你想让断路器熔断的状态代码时，你可以使用一个带有状态代码值的整数或HttpStatus枚举的字符串表示。

Example 24. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
            statusCodes:
              - 500
              - "NOT_FOUND"
```

Example 25. Application.java

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis").addStatusCode("INTERNAL_SERVER_ERROR"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

### 6.6. The FallbackHeaders GatewayFilter Factory

通过`FallbackHeaders`工厂，你可以在转发到外部应用程序中的`fallbackUri`的请求的标题中添加`Spring Cloud CircuitBreaker`的执行异常细节，如以下场景。

Example 26. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

在这个例子中，在运行断路器时发生执行异常后，请求被转发到运行在`localhost:9994`的应用程序中的`fallback`端点或处理器。带有异常类型、消息和（如果有）根本原因的异常类型和消息的头文件被`FallbackHeaders`过滤器添加到该请求中。

你可以通过设置以下参数的值来覆盖配置中的头文件名称（显示为默认值）。

- `executionExceptionTypeHeaderName` ( `"Execution-Exception-Type"` )
- `executionExceptionMessageHeaderName` ( `"Execution-Exception-Message"` )
- `rootCauseExceptionTypeHeaderName` ( `"Root-Cause-Exception-Type"` )
- `rootCauseExceptionMessageHeaderName` ( `"Root-Cause-Exception-Message"` )

关于断路器和网关的更多信息，请参阅[Spring Cloud CircuitBreaker Factory部分](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#spring-cloud-circuitbreaker-filter-factory)。

### 6.7. The MapRequestHeader GatewayFilter Factory

MapRequestHeader GatewayFilter工厂接受fromHeader和toHeader参数。它创建一个新的命名头（toHeader），并从传入的http请求的现有命名头（fromHeader）中提取值。如果输入的头不存在，过滤器没有任何影响。如果新的命名头信息已经存在，它的值就会被增加新的值。下面的例子配置了一个MapRequestHeader。

Example 27. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```

这将在下游请求中添加`X-Request-Red:<values>`头，并从传入的HTTP请求的`Blue`头中更新数值。

### 6.8. The PrefixPath GatewayFilter Factory

PrefixPath GatewayFilter工厂需要一个前缀参数。下面的例子配置了一个PrefixPath GatewayFilter。

Example 28. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

这将把`/mypath`作为所有匹配请求的路径的前缀。因此，一个到`/hello`的请求将被发送到`/mypath/hello`。

### 6.9. The PreserveHostHeader GatewayFilter Factory

PreserveHostHeader GatewayFilter工厂没有参数。这个过滤器设置一个请求属性，路由过滤器会检查该属性，以确定是否应该发送原始的主机头，而不是由HTTP客户端确定的主机头。下面的例子配置了一个PreserveHostHeader GatewayFilter。

Example 29. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: https://example.org
        filters:
        - PreserveHostHeader
```

### 6.10. The RequestRateLimiter GatewayFilter Factory

`RequestRateLimiter``GatewayFilter`工厂使用`RateLimiter`实现来确定是否允许当前请求继续进行。如果不允许，将返回 "HTTP 429 - Too Many Requests"（默认）的状态。

这个过滤器需要一个可选的`keyResolver`参数和特定于速率限制器的参数（在本节后面描述）。

`keyResolver`是一个实现`KeyResolver`接口的bean。在配置中，使用SpEL来引用Bean的名字。`#{@myKeyResolver}`是一个SpEL表达式，它引用了一个名为`myKeyResolver`的bean。下面的列表显示了`KeyResolver`的接口。

Example 30. KeyResolver.java

```java
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

`KeyResolver`接口让可插拔的策略得出限制请求的密钥。在未来的里程碑版本中，会有一些`KeyResolver`的实现。

`KeyResolver`的默认实现是`PrincipalNameKeyResolver`，它从`ServerWebExchange`中检索`Principal`并调用`Principal.getName()`。

默认情况下，如果`KeyResolver`没有找到一个密钥，请求将被拒绝。你可以通过设置`spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key`（`true`或`false`）和`spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code`属性调整这种行为。

`RequestRateLimiter` 不能用 "快捷方式" 符号来配置。下面的例子是无效的

Example 31. application.properties

```properties
# INVALID SHORTCUT CONFIGURATION
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
```

#### 6.10.1. Redis RateLimiter

Redis的实现是基于[Stripe](https://stripe.com/blog/rate-limiters)的工作。它需要使用`spring-boot-starter-data-redis-reactive` Spring Boot启动器。

使用的算法是[Token Bucket Algorithm](https://en.wikipedia.org/wiki/Token_bucket)。

`redis-rate-limiter.replenishRate`属性是你希望用户每秒可以做多少个请求，而不允许有任何放弃的请求。这就是代币桶被填充的速度。

`redis-rate-limiter.burstCapacity`属性是允许一个用户在一秒钟内完成的最大请求数。这是代币桶可以容纳的代币数量。将此值设置为零，可以阻止所有请求。

`redis-rate-limiter.requestedTokens`属性是一个请求需要花费多少代币。这是每次请求时从桶中取出的代币数量，默认为`1`。

稳定的速率是通过在`replenishRate`和`burstCapacity`中设置相同的值来完成的。通过设置`burstCapacity'高于`replenishRate'，可以允许临时的突发。在这种情况下，速率限制器需要在突发之间允许一些时间（根据`replenishRate`），因为连续两次突发将导致请求被放弃（`HTTP 429 - Too Many Requests`）。下面的列表配置了一个`redis-rate-limiter`。

通过设置 `replenishRate` 为想要的请求数，`requestedTokens` 为秒数，`burstCapacity` 为 `replenishRate` 和 `requestedTokens` 的乘积，来实现低于 `1 request/s` 的速率限制，例如，设置 `replenishRate=1`，`requestedTokens=60` 和 `burstCapacity=60` 将导致限制为 `1 request/min`。

Example 32. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

下面的例子在Java中配置了一个KeyResolver。

Example 33. Config.java

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户的请求率限制为10。爆发20次是允许的，但是，在下一秒，只有10个请求可以使用。`KeyResolver` 是一个简单的，获得 `user` 请求参数（注意，不建议在生产中这样做）。

你也可以把速率限制器定义为一个实现`RateLimiter`接口的bean。在配置中，你可以用SpEL来引用bean的名字。`#{@myRateLimiter}`是一个SpEL表达式，引用一个名为`myRateLimiter`的bean。下面的列表定义了一个速率限制器，它使用了前面列表中定义的`KeyResolver`。

Example 34. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```

### 6.11. The RedirectTo GatewayFilter Factory

`RedirectTo` `GatewayFilter`工厂需要两个参数，`status`和`url`。`status`参数应该是一个300系列的重定向HTTP代码，如301。`url`参数应该是一个有效的URL。这是`Location`头的值。对于相对重定向，你应该使用`uri: no://op`作为路由定义的URI。下面的列表配置了一个`RedirectTo` `GatewayFilter`。

Example 35. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```

这将发送一个带有`Location:https://acme.org`头的状态302来执行重定向。

### 6.12. The RemoveRequestHeader GatewayFilter Factory

RemoveRequestHeader GatewayFilter工厂需要一个name参数。它是要删除的头的名称。下面的列表配置了一个RemoveRequestHeader GatewayFilter。

Example 36. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```

这在向下游发送之前删除了`X-Request-Foo`标头。

### 6.13. RemoveResponseHeader GatewayFilter Factory

RemoveResponseHeader GatewayFilter工厂需要一个name参数。它是要删除的头的名称。下面的列表配置了一个RemoveResponseHeader GatewayFilter。

Example 37. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removeresponseheader_route
        uri: https://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```

他将在响应返回到网关客户端之前从响应中删除`X-Response-Foo`头。

要删除任何种类的敏感头，你应该为任何你可能想这样做的路由配置这个过滤器。此外，你可以通过使用`spring.cloud.gateway.default-filters`来配置一次这个过滤器，并让它应用于所有路由。

### 6.14. The RemoveRequestParameter GatewayFilter Factory

RemoveRequestParameter GatewayFilter工厂需要一个名称参数。它是要删除的查询参数的名称。下面的例子配置了一个RemoveRequestParameter GatewayFilter。

Example 38. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```

这将在向下游发送之前删除`red`参数。

### 6.15. The RewritePath GatewayFilter Factory

`RewritePath` `GatewayFilter`工厂接收一个路径`regexp`参数和一个`replacement`参数。这是用Java正则表达式来重写请求路径的一种灵活方式。下面的列表配置了一个`RewritePath` `GatewayFilter`。

Example 39. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red/?(?<segment>.*), /$\{segment}
```

对于一个`/red/blue`的请求路径，在进行下游请求之前将路径设置为`/blue`。注意，由于YAML规范，`$`应该被替换成`$\`。

### 6.16. RewriteLocationResponseHeader GatewayFilter Factory

RewriteLocationResponseHeader GatewayFilter工厂修改Location响应头的值，通常是为了去掉后台的特定细节。它需要 `stripVersionMode`、`locationHeaderName`、`hostValue` 和 `protocolsRegex` 参数。下面的列表配置了一个RewriteLocationResponseHeader GatewayFilter。

Example 40. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritelocationresponseheader_route
        uri: http://example.org
        filters:
        - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
```

例如，对于一个`POST api.example.com/some/object/name`的请求，`Location`的响应头值`object-service.prod.example.net/v2/some/object/id`被改写为`api.example.com/some/object/id`。

`stripVersionMode`参数有以下可能的值。`NEVER_STRIP`, `AS_IN_REQUEST` (默认), 和 `ALWAYS_STRIP` 。

- `NEVER_STRIP` : 即使原始请求路径不包含版本，版本也不会被剥离。
- `AS_IN_REQUEST` 只有在原始请求路径不包含版本的情况下，版本才会被剥离。
- `ALWAYS_STRIP` 即使原始请求路径包含版本，版本也会被剥离。

`hostValue`参数，如果提供的话，用于替换响应`Location`头的`host:port`部分。如果没有提供，则使用`Host`请求头的值。

参数 `protocolsRegex` 必须是一个有效的正则 `String`，与协议名称相匹配。如果没有匹配，过滤器不做任何事情。默认是`http|https|ftp|ftps`。

### 6.17. The RewriteResponseHeader GatewayFilter Factory

RewriteResponseHeader GatewayFilter工厂接受名称、regexp和替换参数。它使用Java正则表达式，以一种灵活的方式重写响应头的值。下面的例子配置了一个RewriteResponseHeader GatewayFilter。

Example 41. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

对于一个头值为`/42?user=ford&password=omg!what&flag=true`，它在发出下游请求后被设置为`/42?user=ford&password=***&flag=true`。由于YAML的规范，你必须用`$\`来表示`$`。

### 6.18. The SaveSession GatewayFilter Factory

SaveSession GatewayFilter工厂在转发下游调用之前强制进行`WebSession::save`操作。这在使用类似`Spring Session`的懒惰数据存储时特别有用，因为你需要确保在进行转发调用之前已经保存了会话状态。下面的例子配置了一个SaveSession GatewayFilter。

Example 42. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

如果你将[Spring Security](https://projects.spring.io/spring-security/)与Spring Session集成，并希望确保安全细节已被转发给远程进程，这一点至关重要。

### 6.19. The SecureHeaders GatewayFilter Factory

`SecureHeaders` `GatewayFilter`工厂根据[本博文](https://blog.appcanary.com/2017/http-security-headers.html)的建议，在响应中添加了一些头信息。

添加了以下标题（显示的是其默认值）。

- `X-Xss-Protection:1 (mode=block` )
- `Strict-Transport-Security (max-age=631138519` )
- `X-Frame-Options (DENY)`
- `X-Content-Type-Options (nosniff)`
- `Referrer-Policy (no-referrer)`
- `Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'`
- `X-Download-Options (noopen)`
- `X-Permitted-Cross-Domain-Policies (none)`

要改变默认值，请在`spring.cloud.gateway.filter.secure-headers`命名空间中设置相应的属性。以下属性是可用的。

- `xss-protection-header`
- `strict-transport-security`
- `x-frame-options`
- `x-content-type-options`
- `referrer-policy`
- `content-security-policy`
- `x-download-options`
- `x-permitted-cross-domain-policies`

要禁用默认值，请用逗号分隔的值设置`spring.cloud.gateway.filter.secure-headers.disable`属性。下面的例子显示了如何做到这一点。

```properties
spring.cloud.gateway.filter.secure-headers.disable=x-frame-options,strict-transport-security
```

> 需要使用安全头的小写全名来禁用它。

### 6.20. The SetPath GatewayFilter Factory

SetPath GatewayFilter工厂接受一个路径模板参数。它提供了一种简单的方法，通过允许模板化的路径段来操作请求路径。这使用了Spring Framework的URI模板。允许多个匹配段。下面的例子配置了一个SetPath GatewayFilter。

Example 43. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

对于请求路径为`/red/blue`的情况，在进行下游请求前将路径设置为`/blue`。

### 6.21. The SetRequestHeader GatewayFilter Factory

SetRequestHeader GatewayFilter工厂接受name和value参数。下面的列表配置了一个SetRequestHeader GatewayFilter。

Example 44. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

这个`GatewayFilter`替换（而不是添加）所有给定名称的头信息。因此，如果下游服务器响应的是`X-Request-Red:1234`，这将被替换为`X-Request-Red:Blue`，这就是下游服务将收到的内容。

`SetRequestHeader`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时被扩展。下面的例子配置了一个使用变量的`SetRequestHeader``GatewayFilter`。

Example 45. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetRequestHeader=foo, bar-{segment}
```

### 6.22. The SetResponseHeader GatewayFilter Factory

`SetResponseHeader` `GatewayFilter`工厂接受`name`和`value`参数。下面的列表配置了一个`SetResponseHeader` `GatewayFilter`。

Example 46. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        filters:
        - SetResponseHeader=X-Response-Red, Blue
```

这个GatewayFilter会替换（而不是添加）所有给定名称的头信息。因此，如果下游服务器响应的是`X-Response-Red:1234`，这将被替换为`X-Response-Red:Blue`，这就是网关客户端将收到的内容。

`SetResponseHeader`知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并将在运行时被扩展。下面的例子配置了一个`SetResponseHeader``GatewayFilter`，使用了一个变量。

Example 47. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetResponseHeader=foo, bar-{segment}
```

### 6.23. The SetStatus GatewayFilter Factory

SetStatus GatewayFilter工厂只接受一个参数，即status。它必须是一个有效的Spring HttpStatus。它可以是`404`的整数值或枚举的字符串表示。`NOT_FOUND`。下面的列表配置了一个SetStatus GatewayFilter。

Example 48. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: https://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

在这两种情况下，响应的HTTP状态被设置为401。

你可以配置`SetStatus` `GatewayFilter`，在响应中的一个头中返回代理请求的原始HTTP状态代码。如果配置了以下属性，该头会被添加到响应中。

Example 49. application.yml

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```

### 6.24. The StripPrefix GatewayFilter Factory

StripPrefix GatewayFilter工厂需要一个参数，即parts。parts参数表示在向下游发送请求之前要从路径中剥离的部分的数量。下面的列表配置了一个StripPrefix GatewayFilter。

Example 50. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

当通过网关向`/name/blue/red`发出请求时，向`nameservice`发出的请求看起来像`nameservice/red`。

### 6.25. The Retry GatewayFilter Factory

`Retry` `GatewayFilter`工厂支持以下参数。

- `retries` : 应该尝试的重试次数。
- `statuses` : 应该重试的HTTP状态代码，用`org.springframework.http.HttpStatus`表示。
- `methods`：应该重试的HTTP方法，用`org.springframework.http.HttpMethod`表示。
- `series`：要重试的状态代码系列，用`org.springframework.http.HttpStatus.Series`表示。
- `exceptions` : 抛出的异常列表，应该被重新尝试。
- `backoff`：为重试配置的指数后验。重试的时间间隔为`firstBackoff * (factor ^ n)`，其中`n`是迭代次数。如果配置了 "maxBackoff"，应用的最大后退时间限制为 `maxBackoff`。如果`basedOnPreviousValue`为真，后退是通过`prevBackoff * factor`计算的。

如果启用了 `Retry` 过滤器，下列默认值被配置。

- `retries` : Three times
- `series` : 5XX series
- `methods` : GET method
- `exceptions` : `IOException` and `TimeoutException`
- `backoff` : disabled

下面的列表配置了一个重试的`GatewayFilter`。

Example 51. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: http://localhost:8080/flakey
        predicates:
        - Host=*.retry.com
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
            methods: GET,POST
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

> 当使用带有`forward:`前缀的URL的重试过滤器时，应仔细编写目标端点，以便在出现错误时，它不会做任何可能导致响应被发送到客户端并提交的事情。例如，如果目标端点是一个有注释的控制器，目标控制器方法不应该返回`ResponseEntity`，并带有错误状态代码。相反，它应该抛出一个`Exception`或发出一个错误信号（例如，通过`Mono.error(ex)`返回值），重试过滤器可以被配置为通过重试来处理。

> 当对任何带有请求体的HTTP方法使用重试过滤器时，请求体将被缓存，网关将变得内存有限。请求体被缓存在一个由`ServerWebExchangeUtils.CACHED_REQUEST_BODY_ATTR`定义的请求属性中。该对象的类型是`org.springframework.core.io.buffer.DataBuffer`。

### 6.26. The RequestSize GatewayFilter Factory

当请求的大小超过允许的限制时，`RequestSize` `GatewayFilter`工厂可以限制请求到达下游服务。该过滤器需要一个`maxSize`参数。`maxSize`是一个`DataSize`类型，所以值可以定义为一个数字，后面有一个可选的`DataUnit`后缀，如`KB`或`MB`。默认值是`B`，表示字节数。它是以字节为单位定义的请求的可允许的大小限制。下面的列表配置了一个`RequestSize` `GatewayFilter`。

Example 52. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

RequestSize GatewayFilter工厂将响应状态设置为413 Payload Too Large，当请求由于大小而被拒绝时，会有一个额外的头 errorMessage。下面的例子显示了这样一个errorMessage。

```text
errorMessage` : `Request size is larger than permissible limit. Request size is 6.0 MB where permissible limit is 5.0 MB
```

> 如果在路由定义中没有提供过滤参数，默认请求大小被设置为5MB。

### 6.27. The SetRequestHostHeader GatewayFilter Factory

在某些情况下，host header可能需要被重写。在这种情况下，SetRequestHostHeader GatewayFilter工厂可以将现有的host header替换成指定的vaue。该过滤器需要一个`host`参数。下面的列表配置了一个SetRequestHostHeader GatewayFilter。

Example 53. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: set_request_host_header_route
        uri: http://localhost:8080/headers
        predicates:
        - Path=/headers
        filters:
        - name: SetRequestHostHeader
          args:
            host: example.org
```

`SetRequestHostHeader` `GatewayFilter`工厂将主机头的值替换为`example.org`。

### 6.28. Modify a Request Body GatewayFilter Factory

你可以使用ModifyRequestBody过滤器，在网关向下游发送请求体之前对其进行修改。

> 这个过滤器只能通过使用Java DSL来配置。

下面的列表显示了如何修改一个请求体GatewayFilter。

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

> 如果请求没有正文，`RewriteFilter`将被传递为空。应该返回`Mono.empty()`来指定请求中缺少的主体。

### 6.29. Modify a Response Body GatewayFilter Factory

你可以使用ModifyResponseBody过滤器来修改响应体，然后再把它送回给客户端。

> 这个过滤器只能通过使用Java DSL来配置。

下面的列表显示了如何修改一个响应体GatewayFilter。

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri))
        .build();
}
```

> 如果响应没有正文，`RewriteFilter`将被传递为空。应该返回`Mono.empty()`来指定响应中缺少的主体。

### 6.30. Token Relay GatewayFilter Factory

Token Relay是指OAuth2消费者作为客户端，将传入的令牌转发给传出的资源请求。消费者可以是一个纯粹的客户端（如SSO应用程序）或一个资源服务器。

Spring Cloud Gateway可以将OAuth2访问令牌转发到它所代理的服务的下游。要在网关中添加这个功能，你需要像这样添加TokenRelayGatewayFilterFactory。

App.java

```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
            .route("resource", r -> r.path("/resource")
                    .filters(f -> f.tokenRelay())
                    .uri("http://localhost:9000"))
            .build();
}
```

或者这样

application.yaml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: resource
        uri: http://localhost:9000
        predicates:
        - Path=/resource
        filters:
        - TokenRelay=
```

它将（除了登录用户和抓取令牌外）把认证令牌传递给下游的服务（在这里是`/resource`）。

要为Spring Cloud Gateway启用这个功能，需要添加以下依赖项

- `org.springframework.boot:spring-boot-starter-oauth2-client`。

它是如何工作的？{githubmaster}/src/main/java/org/springframework/cloud/gateway/security/TokenRelayGatewayFilterFactory.java[filter]从当前认证的用户中提取访问令牌，并将其放入下游请求的请求头。

完整的工作样本见[该项目](https://github.com/spring-cloud-samples/sample-gateway-oauth2login)

> 只有当适当的`spring.security.oauth2.client.*`属性被设置时，TokenRelayGatewayFilterFactory Bean才会被创建，这将触发`ReactiveClientRegistrationRepository` Bean的创建。

> `TokenRelayGatewayFilterFactory`使用的`ReactiveOAuth2AuthorizedClientService`的默认实现使用了一个内存数据存储。如果你需要一个更强大的解决方案，你将需要提供你自己的实现`ReactiveOAuth2AuthorizedClientService`。

### 6.31. Default Filters

要添加一个过滤器并将其应用于所有路由，可以使用 `spring.cloud.gateway.default-filters`。这个属性需要一个过滤器的列表。下面的列表定义了一组默认过滤器。

Example 54. application.yml

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

## 7. Global Filters

GlobalFilter接口的签名与GatewayFilter相同。这些是特殊的过滤器，有条件地应用于所有路由。

> 这个接口和它的用法在未来的里程碑版本中可能会有变化。

### 7.1. Combined Global Filter and GatewayFilter Ordering

当一个请求与路由匹配时，过滤网络处理器将`GlobalFilter`的所有实例和`GatewayFilter`的所有路由特定实例添加到一个过滤链中。这个组合的过滤器链由`org.springframework.core.Ordered`接口进行排序，你可以通过实现`getOrder()`方法来设置这个接口。

由于Spring Cloud Gateway区分了过滤器逻辑执行的 "pre"和 "post"阶段（见如何工作），优先级最高的过滤器在 "pre"阶段是第一个，在 "post"阶段是最后一个。

下面的列表配置了一个过滤器链。

Example 55. ExampleConfiguration.java

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### 7.2. Forward Routing Filter

`ForwardRoutingFilter`在exchange attribute `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`中寻找一个URI。如果URL有一个`forward`方案（如`forward:///localendpoint`），它将使用Spring的`DispatcherHandler`来处理请求。请求URL的路径部分被转发URL中的路径所覆盖。未修改的原始URL被附加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性的列表中。

### 7.3. The ReactiveLoadBalancerClientFilter

`ReactiveLoadBalancerClientFilter`在名为`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`的交换属性中寻找一个URI。如果URL有一个`lb`方案（如`lb://myservice`），它使用Spring Cloud的`ReactorLoadBalancer`将名称（本例中的`myservice`）解析为实际的主机和端口，并替换同一属性中的URI。未修改的原始URL被追加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性的列表中。过滤器也会查看`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`属性，看它是否等于`lb`。如果是的话，同样的规则也适用。下面的列表配置了一个`ReactiveLoadBalancerClientFilter`。

Example 56. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

> 默认情况下，当服务实例不能被`ReactorLoadBalancer`找到时，会返回`503`。你可以通过设置`spring.cloud.gateway.loadbalancer.use404=true`将网关配置为返回`404`。

> 从`ReactiveLoadBalancerClientFilter`返回的`ServiceInstance`的`isSecure`值覆盖了向网关发出的请求中指定的方案。例如，如果请求通过HTTPS进入Gateway，但`ServiceInstance`表明它不安全，那么下游请求将通过HTTP进行。相反的情况也可以适用。然而，如果在网关配置中为路由指定了 `GATEWAY_SCHEME_PREFIX_ATTR`，那么前缀将被剥离，来自路由 URL 的结果方案将覆盖 `ServiceInstance` 的配置。

> Gateway支持所有的LoadBalancer功能。你可以在[Spring Cloud Commons documentation](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)中阅读更多关于它们的信息。

### 7.4. The Netty Routing Filter

如果位于`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute 中的URL有一个http或https方案，Netty路由过滤器就会运行。它使用Netty HttpClient来进行下游代理请求。响应被放在`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange attribute中，供以后的过滤器使用。(还有一个实验性的WebClientHttpRoutingFilter，执行同样的功能，但不需要Netty。)

### 7.5. The Netty Write Response Filter

如果在`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange attribute中有一个Netty HttpClientResponse，NettyWriteResponseFilter就会运行。它在所有其他过滤器完成后运行，并将代理响应写回网关客户端响应。(还有一个实验性的WebClientWriteResponseFilter，执行同样的功能，但不需要Netty。)

### 7.6. The RouteToRequestUrl Filter

如果在`ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR` exchange attribute中有一个Route对象，RouteToRequestUrlFilter会运行。它创建了一个新的URI，基于请求URI，但用路由对象的URI属性进行更新。新的URI被放置在`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute`中。

如果URI有一个方案前缀，如`lb:ws://serviceid`，lb方案将从URI中剥离，并放在`ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`中，以便以后在过滤器链中使用。

### 7.7. The Websocket Routing Filter

如果位于`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute中的URL有ws或wss方案，则运行websocket路由过滤器。它使用Spring WebSocket基础设施来转发下游的websocket请求。

你可以通过在URI前加上lb，如`lb:ws://serviceid`，来平衡websocket的负载。

> 如果你使用[SockJS](https://github.com/sockjs)作为普通HTTP的后备方案，你应该配置一个普通的HTTP路由以及websocket路由。

下面的列表配置了一个websocket路由过滤器。

Example 57. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      # SockJS route
      - id: websocket_sockjs_route
        uri: http://localhost:3001
        predicates:
        - Path=/websocket/info/**
      # Normal Websocket route
      - id: websocket_route
        uri: ws://localhost:3001
        predicates:
        - Path=/websocket/**
```

### 7.8. The Gateway Metrics Filter

要启用网关指标，请添加`spring-boot-starter-actuator`作为项目依赖。然后，默认情况下，只要属性`spring.cloud.gateway.metrics.enabled`没有设置为`false`，网关指标过滤器就会运行。这个过滤器添加了一个名为`gateway.requests`的定时器指标，标签如下。

- `routeId` : 路径ID。
- `routeUri` : API被路由到的URI。
- `outcome` : 结果，由[HttpStatus.Series](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.Series.html)分类。
- `status` : 返回给客户端的请求的HTTP状态。
- `httpStatusCode` : 返回给客户端的请求的HTTP状态。
- `httpMethod` : 请求使用的HTTP方法。

这些指标可以从`/actuator/metrics/gateway.requests`中获取，并可以很容易地与Prometheus集成，创建一个[Grafana](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/images/gateway-grafana-dashboard.jpeg) [仪表盘](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/gateway-grafana-dashboard.json)。

> 要启用Prometheus端点，请添加`micrometer-registry-prometheus`作为项目依赖。

### 7.9. Marking An Exchange As Routed

在网关路由了一个`ServerWeb Exchange`后，它通过在exchange attributes中添加`gatewayAlreadyRouted`来标记该交换为 "routed"。一旦一个请求被标记为已路由，其他的路由过滤器将不再对该请求进行路由，基本上是跳过该过滤器。有一些方便的方法，你可以用来标记一个交换为路由，或者检查一个交换是否已经被路由。

- `ServerWebExchangeUtils.isAlreadyRouted`接收一个`ServerWebExchange`对象并检查它是否已经被 "routed"。
- `ServerWebExchangeUtils.setAlreadyRouted`接收一个`ServerWebExchange`对象并将其标记为 "routed"。

## 8. HttpHeadersFilters

HttpHeadersFilters在向下游发送请求之前被应用于请求，例如在NettyRoutingFilter。

### 8.1. Forwarded Headers Filter

Forwarded Headers Filter创建一个Forwarded header来发送给下游的服务。它将当前请求的主机头、方案和端口添加到任何现有的转发头中。

### 8.2. RemoveHopByHop Headers Filter

`RemoveHopByHop Headers Filter` 从转发的请求中删除头信息。被移除的默认头信息列表来自[IETF](https://tools.ietf.org/html/draft-ietf-httpbis-p1-messaging-14#section-7.1.3)。

默认删除的header是:

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- TE
- Trailer
- Transfer-Encoding
- Upgrade

要改变这一点，请将`spring.cloud.gateway.filter.remove-hop-by-hop.headers`属性设置为要删除的头名称列表。

### 8.3. XForwarded Headers Filter

XForwarded Headers Filter创建各种`X-Forwarded-*`头，以发送到下游服务。它使用当前请求的Host头、方案、端口和路径来创建各种头。

创建各个头信息可以由以下布尔属性控制（默认为true）。

- `spring.cloud.gateway.x-forwarded.for-enabled`
- `spring.cloud.gateway.x-forwarded.host-enabled`
- `spring.cloud.gateway.x-forwarded.port-enabled`
- `spring.cloud.gateway.x-forwarded.proto-enabled`
- `spring.cloud.gateway.x-forwarded.prefix-enabled`

附加多个header可以由以下布尔属性控制（默认为真）。

- `spring.cloud.gateway.x-forwarded.for-append`
- `spring.cloud.gateway.x-forwarded.host-append`
- `spring.cloud.gateway.x-forwarded.port-append`
- `spring.cloud.gateway.x-forwarded.proto-append`
- `spring.cloud.gateway.x-forwarded.prefix-append`

## 9. TLS and SSL

网关可以通过遵循通常的Spring服务器配置来监听HTTPS请求。下面的例子显示了如何做到这一点。

Example 58. application.yml

```yaml
server:
  ssl:
    enabled: true
    key-alias: scg
    key-store-password: scg1234
    key-store: classpath:scg-keystore.p12
    key-store-type: PKCS12
```

你可以将网关路由到HTTP和HTTPS后端。如果你要路由到HTTPS后端，你可以通过以下配置将网关配置为信任所有下游的证书。

Example 59. application.yml

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          useInsecureTrustManager: true
```

使用不安全的信任管理器不适合于生产。对于生产部署，你可以用一组已知的证书来配置网关，它可以通过以下配置来信任。

Example 60. application.yml

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          trustedX509Certificates:
          - cert1.pem
          - cert2.pem
```

如果Spring Cloud Gateway没有配置受信任的证书，就会使用默认的信任存储（你可以通过设置`javax.net.ssl.trustStore`系统属性来覆盖它）。

### 9.1. TLS Handshake

网关维护着一个客户端池，它用来路由到后端。当通过HTTPS进行通信时，客户端发起了一个TLS握手。一些超时与这个握手相关。你可以对这些超时进行配置（默认值显示），如下。

Example 61. application.yml

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          handshake-timeout-millis: 10000
          close-notify-flush-timeout-millis: 3000
          close-notify-read-timeout-millis: 0
```

## 10. Configuration

Spring Cloud Gateway的配置是由`RouteDefinitionLocator`实例的集合驱动的。下面的列表显示了`RouteDefinitionLocator`接口的定义。

Example 62. RouteDefinitionLocator.java

```java
public interface RouteDefinitionLocator {
    Flux<RouteDefinition> getRouteDefinitions();
}
```

默认情况下，`PropertiesRouteDefinitionLocator`通过使用Spring Boot的`@ConfigurationProperties`机制加载属性。

前面的配置例子都使用了一种快捷的符号，即使用位置参数而不是命名参数。下面的两个例子是等价的。

Example 63. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatus_route
        uri: https://example.org
        filters:
        - name: SetStatus
          args:
            status: 401
      - id: setstatusshortcut_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

对于网关的某些用途来说，属性已经足够了，但一些生产用例会从外部来源（如数据库）加载配置中受益。未来的里程碑版本将有基于Spring数据存储库的`RouteDefinitionLocator`实现，如`Redis`、`MongoDB`和`Cassandra`。

## 11. Route Metadata Configuration

你可以通过使用元数据为每个途径配置额外的参数，如下所示。

Example 64. application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: route_with_metadata
        uri: https://example.org
        metadata:
          optionName: "OptionValue"
          compositeObject:
            name: "value"
          iAmNumber: 1
```

你可以从一个exchange所获取所有的元数据属性，如下所示。

```java
Route route = exchange.getAttribute(GATEWAY_ROUTE_ATTR);
// get all metadata properties
route.getMetadata();
// get a single metadata property
route.getMetadata(someKey);
```

## 12. Http timeouts configuration

可以为所有路由配置Http超时（响应和连接），也可以为每个特定的路由重写。

### 12.1. 全局超时

要配置全局http超时。
`connect-timeout` 必须以毫秒为单位指定。
`response-timeout` 必须以`java.time.Duration`的形式指定。

全局超时

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

### 12.2. 每个路由的超时

要配置每个路由的超时。
`connect-timeout`必须以毫秒为单位指定。
`response-timeout` 必须以毫秒为单位指定。

通过yaml配置每个路由的超时

```YAML
      - id: per_route_timeouts
        uri: https://example.org
        predicates:
          - name: Path
            args:
              pattern: /delay/{timeout}
        metadata:
          response-timeout: 200
          connect-timeout: 200
```

使用Java DSL配置每个路由的超时

```java
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.CONNECT_TIMEOUT_ATTR;
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.RESPONSE_TIMEOUT_ATTR;

      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder routeBuilder){
         return routeBuilder.routes()
               .route("test1", r -> {
                  return r.host("*.somehost.org").and().path("/somepath")
                        .filters(f -> f.addRequestHeader("header1", "header-value-1"))
                        .uri("http://someuri")
                        .metadata(RESPONSE_TIMEOUT_ATTR, 200)
                        .metadata(CONNECT_TIMEOUT_ATTR, 200);
               })
               .build();
      }
```

### 12.3. Fluent Java Routes API

为了允许在Java中进行简单的配置，RouteLocatorBuilder Bean包括一个fluent的API。下面的列表显示了它是如何工作的。

Example 65. GatewaySampleApplication.java

```java
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
    return builder.routes()
            .route(r -> r.host("**.abc.org").and().path("/image/png")
                .filters(f ->
                        f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.path("/image/webp")
                .filters(f ->
                        f.addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
                .metadata("key", "value")
            )
            .route(r -> r.order(-1)
                .host("**.throttle.org").and().path("/get")
                .filters(f -> f.filter(throttle.apply(1,
                        1,
                        10,
                        TimeUnit.SECONDS)))
                .uri("http://httpbin.org:80")
                .metadata("key", "value")
            )
            .build();
}
```

这种风格也允许更多的自定义谓词断言。由`RouteDefinitionLocator` Bean定义的谓词使用逻辑上的和来组合。通过使用流畅的Java API，你可以在`Predicate`类上使用`and()`、`or()`和`negate()`操作符。

### 12.4. The DiscoveryClient Route Definition Locator

你可以将网关配置为基于在`DiscoveryClient`兼容的服务注册中心注册的服务来创建路由。

要启用这一点，请设置`spring.cloud.gateway.discovery.locator.enabled=true`，并确保`DiscoveryClien`t`实现（如Netflix Eureka、Consul或Zookeeper）位于classpath上并已启用。

#### 12.4.1. 为DiscoveryClient路由配置谓词和过滤器

默认情况下，网关为用`DiscoveryClient`创建的路由定义了一个谓词和过滤器。

默认谓词是用`/serviceId/**`模式定义的路径谓词，其中`serviceId`是来自`DiscoveryClient`的服务的ID。

默认过滤器是一个重写路径过滤器，采用重写词`/serviceId/?(?<remaining>.*)`和替换词`/${remaining}`。在请求被发送到下游之前，这将从路径中剥离服务ID。

如果你想自定义`DiscoveryClient`路由使用的谓词或过滤器，请设置`spring.cloud.gateway.discovery.locator.predicates[x]` 和 `spring.cloud.gateway.discovery.locator.filters[y]`。这样做时，如果你想保留该功能，你需要确保包括前面显示的默认谓词和过滤器。下面的例子显示了这是什么样子。

Example 66. application.properties

```properties
spring.cloud.gateway.discovery.locator.predicates[0].name: Path
spring.cloud.gateway.discovery.locator.predicates[0].args[pattern]: "'/'+serviceId+'/**'"
spring.cloud.gateway.discovery.locator.predicates[1].name: Host
spring.cloud.gateway.discovery.locator.predicates[1].args[pattern]: "'**.foo.com'"
spring.cloud.gateway.discovery.locator.filters[0].name: CircuitBreaker
spring.cloud.gateway.discovery.locator.filters[0].args[name]: serviceId
spring.cloud.gateway.discovery.locator.filters[1].name: RewritePath
spring.cloud.gateway.discovery.locator.filters[1].args[regexp]: "'/' + serviceId + '/?(?<remaining>.*)'"
spring.cloud.gateway.discovery.locator.filters[1].args[replacement]: "'/${remaining}'"
```

## 13. Reactor Netty 访问日志

要启用 Reactor Netty 访问日志，设置 `-Dreactor.netty.http.server.accessLogEnabled=true`。

> 它必须是一个Java系统属性，而不是一个Spring Boot属性。

你可以配置日志系统，使其有一个单独的访问日志文件。下面的例子创建了一个Logback配置。

Example 67. logback.xml

```xml
    <appender name="accessLog" class="ch.qos.logback.core.FileAppender">
        <file>access_log.log</file>
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>
    <appender name="async" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="accessLog" />
    </appender>

    <logger name="reactor.netty.http.server.AccessLog" level="INFO" additivity="false">
        <appender-ref ref="async"/>
    </logger>
```

## 14. CORS 跨域配置

你可以配置网关来控制CORS行为。全局CORS配置是URL模式与[Spring Framework `CorsConfiguration`](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)的映射。下面的例子配置了CORS。

Example 68. application.yml

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

在前面的例子中，对于所有 GET 请求路径，允许来自 `docs.spring.io` 的请求的 CORS 请求。

要为未被某些网关路由谓词处理的请求提供相同的 CORS 配置，请将 s`pring.cloud.gateway.globalcors.add-to-simple-url-handler-mapping` 属性设置为 `true`。当你试图支持 CORS 预检请求，而你的路由谓词因为 HTTP 方法是选项而不能评估为 true 时，这很有用。

## 15. Actuator API

`/gateway`执行器端点可以让你监控并与Spring Cloud Gateway应用互动。要进行远程访问，端点必须在应用程序属性中[启用](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-endpoints-enabling-endpoints)和[通过HTTP或JMX暴露](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints)。下面的列表显示了如何做到这一点。

Example 69. application.properties

```properties
management.endpoint.gateway.enabled=true # default value
management.endpoints.web.exposure.include=gateway
```

### 15.1. Verbose Actuator Format

在Spring Cloud Gateway中增加了一种新的、更粗略的格式。它为每个路由添加了更多细节，让你查看与每个路由相关的谓词和过滤器，以及任何可用的配置。下面的例子配置了`/actuator/gateway/routes`。

```json
[
  {
    "predicate": "(Hosts: [**.addrequestheader.org] && Paths: [/headers], match trailing slash: true)",
    "route_id": "add_request_header_test",
    "filters": [
      "[[AddResponseHeader X-Response-Default-Foo = 'Default-Bar'], order = 1]",
      "[[AddRequestHeader X-Request-Foo = 'Bar'], order = 1]",
      "[[PrefixPath prefix = '/httpbin'], order = 2]"
    ],
    "uri": "lb://testservice",
    "order": 0
  }
]
```

这个功能默认是启用的。要禁用它，请设置以下属性。

Example 70. application.properties

```properties
spring.cloud.gateway.actuator.verbose.enabled=false
```

在未来的版本中，这将默认为`true`

### 15.2. Retrieving Route Filters

本节详细介绍了如何检索路由过滤器，包括。

- [Global Filters](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-global-filters)
- [[gateway-route-filters]](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-route-filters)

#### 15.2.1. Global Filters

要检索应用于所有路由的[全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#global-filters)，向`/actuator/gateway/globalfilters`发出`GET`请求。得到的响应类似于以下内容。

```json
{
  "org.springframework.cloud.gateway.filter.ReactiveLoadBalancerClientFilter@77856cc5": 10100,
  "org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@4f6fd101": 10000,
  "org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@32d22650": -1,
  "org.springframework.cloud.gateway.filter.ForwardRoutingFilter@106459d9": 2147483647,
  "org.springframework.cloud.gateway.filter.NettyRoutingFilter@1fbd5e0": 2147483647,
  "org.springframework.cloud.gateway.filter.ForwardPathFilter@33a71d23": 0,
  "org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@135064ea": 2147483637,
  "org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@23c05889": 2147483646
}
```

响应包含全局过滤器的细节，这些过滤器已经到位。对于每个全局过滤器，有一个过滤器对象的字符串表示（例如，`org.springframework.cloud.gateway.filter.ReactiveLoadBalancerClientFilter@77856cc5`）和过滤器链中相应的[order](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-combined-global-filter-and-gatewayfilter-ordering)。 }。

#### 15.2.2. Route Filters

要检索应用于路由的[`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories)，向`/actuator/gateway/routefilters`发出`GET`请求。得到的响应类似于以下内容。

```json
{
  "[AddRequestHeaderGatewayFilterFactory@570ed9c configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
  "[SecureHeadersGatewayFilterFactory@fceab5d configClass = Object]": null,
  "[SaveSessionGatewayFilterFactory@4449b273 configClass = Object]": null
}
```

响应包含应用于任何特定路由的`GatewayFilter`工厂的细节。对于每个工厂，有一个相应对象的字符串表示（例如，`[SecureHeadersGatewayFilterFactory@fceab5d configClass = Object]`）。请注意，`null` 是由于端点控制器的不完整实现，因为它试图设置过滤器链中的对象的顺序，这不适用于`GatewayFilter`工厂对象。

### 15.3. 刷新路由缓存

要清除路由缓存，请向`/actuator/gateway/refresh`发出一个`POST`请求。该请求返回一个200，没有响应体。

### 15.4. 检索网关中定义的路由

要检索网关中定义的路由，请向`/actuator/gateway/routes`发出`GET`请求。得到的响应类似于以下内容。

```json
[{
  "route_id": "first_route",
  "route_object": {
    "predicate": "org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory$$Lambda$432/1736826640@1e9d7e7d",
    "filters": [
      "OrderedGatewayFilter{delegate=org.springframework.cloud.gateway.filter.factory.PreserveHostHeaderGatewayFilterFactory$$Lambda$436/674480275@6631ef72, order=0}"
    ]
  },
  "order": 0
},
{
  "route_id": "second_route",
  "route_object": {
    "predicate": "org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory$$Lambda$432/1736826640@cd8d298",
    "filters": []
  },
  "order": 0
}]
```

响应包含网关中定义的所有路由的详细信息。下表描述了响应中每个元素的结构（每个都是一个路由）。

|Path|Type|Description|
| --- | --- | --- |
|`route_id`|String|路由ID|
|`route_object.predicate`|Object|路由 predicate|
|`route_object.filters`|Array|应用于路由的[`GatewayFilter`工厂](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories)|
|`order`|Number|路由 order|

### 15.5. 检索指定路由的信息

要检索一条路由的信息，请向`/actuator/gateway/routes/{id}`发出一个GET请求。(例如，`/actuator/gateway/routes/first_route)`。得到的响应类似于下面的内容。

```json
{
  "id": "first_route",
  "predicates": [{
    "name": "Path",
    "args": {"_genkey_0":"/first"}
  }],
  "filters": [],
  "uri": "https://www.uri-destination.org",
  "order": 0
}]
```

下表描述了响应的结构。

|Path|Type|Description|
| --- | --- | --- |
|`id`|String|路由ID|
|`predicates`|Array|路由谓词的集合。每一项都定义了一个给定谓词的名称和参数。|
|`filters`|Array|应用于路由的过滤器集合。|
|`uri`|String|路由的目的地URI。|
|`order`|Number|路由的顺序|

### 15.6. 创建和删除一个指定的路由

要创建一个路由，请向`/gateway/routes/{id_route_to_create}`发出`POST`请求，并以JSON为主体指定路由的字段（见[检索特定路由的信息](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-retrieving-information-about-a-particular-route)）。

要删除一个路由，请向`/gateway/routes/{id_route_to_delete}`发出`DELETE`请求。

### 15.7. 总结：所有端点的清单

下面的表格总结了Spring Cloud Gateway的执行器端点（注意，每个端点都有`/actuator/gateway`作为基本路径）。

|ID|HTTP Method|Description|
| --- | --- | --- |
|`globalfilters`|GET|显示应用于路由的全局过滤器的列表。|
|`routefilters`|GET|显示应用于特定路由的 `GatewayFilter`工厂的列表。|
|`refresh`|POST|清楚路由缓存|
|`routes`|GET|D显示网关中定义的路由列表|
|`routes/{id}`|GET|显示指定路由的详细信息|
|`routes/{id}`|POST|添加一个新的路由到网关|
|`routes/{id}`|DELETE|从网关删除一个路由|

## 16. 故障排除

本节涵盖了你在使用Spring Cloud Gateway时可能出现的常见问题。

### 16.1. 日志级别

在`DEBUG`和`TRACE`级别，以下记录器可能包含有价值的故障排除信息。

- `org.springframework.cloud.gateway`
- `org.springframework.http.server.reactive`
- `org.springframework.web.reactive`
- `org.springframework.boot.autoconfigure.web`
- `reactor.netty`
- `redisratelimiter`

### 16.2. Wiretap

Reactor Netty的HttpClient和HttpServer可以启用wiretap。当与 reactor.netty 日志级别设置为 DEBUG 或 TRACE 相结合时，它能够记录信息，例如通过线路发送和接收的头信息和正文。要启用 wiretap，请分别为 HttpServer 和 HttpClient 设置 `spring.cloud.gateway.httpserver.wiretap=true` 或 `spring.cloud.gateway.httpclient.wiretap=true`。

## 17. 开发者指南

这些是编写网关的一些自定义组件的基本指南。

### 17.1. 编写自定义路由谓语工厂

为了编写一个Route Predicate，你将需要实现`RoutePredicateFactory`。有一个抽象的类叫做`AbstractRoutePredicateFactory`，你可以扩展它。

MyRoutePredicateFactory.java

```java
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<HeaderRoutePredicateFactory.Config> {

    public MyRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        // grab configuration from Config object
        return exchange -> {
            //grab the request
            ServerHttpRequest request = exchange.getRequest();
            //take information from the request to see if it
            //matches configuration.
            return matches(config, request);
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

### 17.2. 编写自定义GatewayFilter工厂

要写一个`GatewayFilter`，你必须实现`GatewayFilterFactory`。你可以扩展一个名为`AbstractGatewayFilterFactory`的抽象类。下面的例子展示了如何做到这一点。

Example 71. PreGatewayFilterFactory.java

```java
public class PreGatewayFilterFactory extends AbstractGatewayFilterFactory<PreGatewayFilterFactory.Config> {

    public PreGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            //If you want to build a "pre" filter you need to manipulate the
            //request before calling chain.filter
            ServerHttpRequest.Builder builder = exchange.getRequest().mutate();
            //use builder to manipulate the request
            return chain.filter(exchange.mutate().request(builder.build()).build());
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

PostGatewayFilterFactory.java

```java
public class PostGatewayFilterFactory extends AbstractGatewayFilterFactory<PostGatewayFilterFactory.Config> {

    public PostGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                ServerHttpResponse response = exchange.getResponse();
                //Manipulate the response in some way
            }));
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

#### 17.2.1. 在配置中命名自定义过滤器和引用

自定义过滤器的类名应该以`GatewayFilterFactory`结尾。

例如，要在配置文件中引用一个名为`Something`的过滤器，该过滤器必须在一个名为`SomethingGatewayFilterFactory`的类中。

> 可以创建一个没有`GatewayFilterFactory`后缀的网关过滤器，如`AnotherThing`类。这个过滤器可以在配置文件中被引用为`AnotherThing`。这不是一个被支持的命名惯例，这种语法可能在未来的版本中被删除。请更新过滤器的名称，使其符合要求。

### 17.3. Writing Custom Global Filters

要编写一个自定义的全局过滤器，你必须实现`GlobalFilter`接口。这将过滤器应用于所有的请求。

下面的例子分别展示了如何设置全局前置和后置过滤器。

```java
@Bean
public GlobalFilter customGlobalFilter() {
    return (exchange, chain) -> exchange.getPrincipal()
        .map(Principal::getName)
        .defaultIfEmpty("Default User")
        .map(userName -> {
          //adds header to proxied request
          exchange.getRequest().mutate().header("CUSTOM-REQUEST-HEADER", userName).build();
          return exchange;
        })
        .flatMap(chain::filter);
}

@Bean
public GlobalFilter customGlobalPostFilter() {
    return (exchange, chain) -> chain.filter(exchange)
        .then(Mono.just(exchange))
        .map(serverWebExchange -> {
          //adds header to response
          serverWebExchange.getResponse().getHeaders().set("CUSTOM-RESPONSE-HEADER",
              HttpStatus.OK.equals(serverWebExchange.getResponse().getStatusCode()) ? "It worked": "It did not work");
          return serverWebExchange;
        })
        .then();
}
```

## 18. 通过使用Spring MVC或Webflux构建一个简单的网关

> 下面描述的是另一种风格的网关。之前的文档都不适用于下面的内容。

Spring Cloud Gateway提供了一个名为`ProxyExchange`的实用对象。你可以在常规的Spring网络处理程序中作为一个方法参数使用它。它通过反映HTTP动词的方法支持基本的下游HTTP交换。在MVC中，它还支持通过`forward()`方法转发到本地处理程序。要使用`ProxyExchange`，在你的classpath中包含正确的模块（`spring-cloud-gateway-mvc或spring-cloud-gateway-webflux`）。

下面的MVC例子将一个到/test的请求代理到一个远程服务器。

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

    @Value("${remote.home}")
    private URI home;

    @GetMapping("/test")
    public ResponseEntity<?> proxy(ProxyExchange<byte[]> proxy) throws Exception {
        return proxy.uri(home.toString() + "/image/png").get();
    }

}
```

下面的例子用Webflux做同样的事情。

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

    @Value("${remote.home}")
    private URI home;

    @GetMapping("/test")
    public Mono<ResponseEntity<?>> proxy(ProxyExchange<byte[]> proxy) throws Exception {
        return proxy.uri(home.toString() + "/image/png").get();
    }

}
```

`ProxyExchange`上的便利方法使处理方法能够发现并增强传入请求的URI路径。例如，你可能想提取路径的尾部元素，将它们传递到下游。

```java
@GetMapping("/proxy/path/**")
public ResponseEntity<?> proxyPath(ProxyExchange<byte[]> proxy) throws Exception {
  String path = proxy.path("/proxy/path/");
  return proxy.uri(home.toString() + "/foos/" + path).get();
}
```

Spring MVC和Webflux的所有功能都可用于网关处理方法。因此，你可以注入请求头和查询参数，例如，你可以通过映射注解中的声明来限制传入的请求。关于这些功能的更多细节，请参见Spring MVC中`@RequestMapping`的文档。

你可以通过使用`ProxyExchange`上的`header()`方法向下游响应添加头信息。

你也可以通过给`get()`方法（和其他方法）添加一个映射器来操作响应头（以及响应中你喜欢的其他东西）。映射器是一个 "函数"，它接收传入的 "ResponseEntity "并将其转换为传出的。

对 "敏感 "头信息（默认情况下，`cookie`和`authorization`）提供一流的支持，这些头信息不会向下游传递，对 "代理"（`x-forwarded-*`）头信息也是如此。

## 19. 配置属性

要查看所有与Spring Cloud Gateway相关的配置属性列表，请参见[附录](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/appendix.html)。

{{#include ../license.md}}
