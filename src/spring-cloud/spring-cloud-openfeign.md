# Spring Cloud OpenFeign

- 当前版本：3.0.3
- 修改时间：2021年7月24日
- 官方文档：[https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-openfeign](https://github.com/spring-cloud/spring-cloud-openfeign)

该项目通过自动配置和与Spring环境及其他Spring编程模型习语的绑定，为Spring Boot应用提供OpenFeign集成。

## 1.  声明式REST客户：Feign

[Feign](https://github.com/OpenFeign/feign)是一个声明式的网络服务客户端。它使编写Web服务客户端更容易。要使用Feign，需要创建一个接口并对其进行注释。它有可插拔的注解支持，包括Feign注解和JAX-RS注解。Feign还支持可插入的编码器和解码器。Spring Cloud增加了对Spring MVC注释的支持，并支持使用Spring Web中默认使用的`HttpMessageConverters`。Spring Cloud集成了Eureka、Spring Cloud CircuitBreaker以及Spring Cloud LoadBalancer，以便在使用Feign时提供一个负载均衡的http客户端。

### 1.1. 如何引入 Feign

要在你的项目中包含Feign，请使用group为`org.springframework.cloud`和artifact id为`spring-cloud-starter-openfeign`的`starter`。请参阅[Spring Cloud项目页面](https://projects.spring.io/spring-cloud/)，了解关于用当前Spring Cloud发布列车设置构建系统的详细信息。

Example spring boot app

```java
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

StoreClient.java

```java
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    Page<Store> getStores(Pageable pageable);

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```

在`@FeignClient`注解中，字符串值（上面的 "stores"）是一个任意的客户端名称，它被用来创建一个[Spring Cloud LoadBalancer客户端](https://github.com/spring-cloud/spring-cloud-commons/blob/main/spring-cloud-loadbalancer/src/main/java/org/springframework/cloud/loadbalancer/blocking/client/BlockingLoadBalancerClient.java)。你也可以使用`url`属性指定一个URL（绝对值或只是一个主机名）。应用程序上下文中的Bean名称是接口的完全限定名称。要指定你自己的别名值，你可以使用`@FeignClient`注解的`qualifiers`值。

上面的负载平衡器客户端将想发现 "stores "服务的物理地址。如果你的应用程序是一个Eureka客户端，那么它将在Eureka服务注册表中解析该服务。如果你不想使用Eureka，你可以使用[`SimpleDiscoveryClient`](https://cloud.spring.io/spring-cloud-static/spring-cloud-commons/current/reference/html/#simplediscoveryclient)在你的外部配置中配置一个服务器列表。

Spring Cloud OpenFeign支持Spring Cloud LoadBalancer的阻塞模式的所有可用功能。你可以在[项目文档](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)中阅读更多关于它们的信息。

### 1.2. 覆盖Feign默认值

Spring Cloud的Feign支持中的一个核心概念是命名的客户端。每个feign客户端都是一个组件集合的一部分，这些组件一起工作，根据需要联系远程服务器，集合有一个名字，你作为应用开发者使用`@FeignClient`注解给它。Spring Cloud使用`FeignClientsConfiguration`为每个命名的客户端按需创建一个新的组合作为`ApplicationContext`。这包括（除其他外）一个`feign.Decoder`，一个`feign.Encoder`，和一个`feign.Contract`。通过使用`@FeignClient`注解的`contextId`属性，可以重写该集合的名称。

Spring Cloud允许你通过使用`@FeignClient`声明额外的配置（在`FeignClientsConfiguration`之上）来完全控制feign客户端。

Example

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```

在这种情况下，客户端由 `FeignClientsConfiguration` 中的组件和 `FooConfiguration`中的任何组件组成（后者将覆盖前者）。

> `FooConfiguration`不需要用`@Configuration`来注释。然而，如果是这样的话，请注意将它从任何`@ComponentScan`中排除，否则将包括这个配置，因为它将成为`feign.Decoder`、`feign.Encoder`、`feign.Contract`等的默认来源。这可以通过把它放在一个单独的、与任何`@ComponentScan`或`@SpringBootApplication`不重叠的包中来避免，或者可以在`@ComponentScan`中明确地排除它。

> 使用`@FeignClient`注解的`contextId`属性，除了改变`ApplicationContext`集合的名称外，它将覆盖客户端名称的别名，它将作为为该客户端创建的配置Bean名称的一部分。

**以前，使用`url`属性，不需要`name`属性。现在使用`name`是必须的**。

在`name`和`url`属性中支持占位符。

```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
    //..
}
```

Spring Cloud OpenFeign默认为feign提供了以下bean（ `BeanType` beanName: `ClassName` ）。

- `Decoder` feignDecoder: `ResponseEntityDecoder` (它包装了一个`SpringDecoder` )
- `Encoder` feignEncoder: `SpringEncoder`。
- `Logger` feignLogger: "Slf4jLogger".
- `MicrometerCapability` micrometerCapability: 如果`feign-micrometer`在classpath上并且`MeterRegistry`是可用的
- `Contract` feignContract: `SpringMvcContract`。
- `Feign.Builder` feignBuilder: "FeignCircuitBreaker.Builder"。
- `Client` feignClient: 如果Spring Cloud LoadBalancer在classpath上，就会使用`FeignBlockingLoadBalancerClient`。如果它们都不在classpath上，则使用默认的feign客户端。

> `spring-cloud-starter-openfeign`支持`spring-cloud-starter-loadbalancer`。然而，由于它是一个可选的依赖关系，如果你想使用它，你需要确保它被添加到你的项目中。

OkHttpClient和ApacheHttpClient以及ApacheHC5 feign客户端可以通过设置`feign.okhttp.enabled`或`feign.httpclient.enabled`或`feign.httpclient.hc5.enabled`分别为`true`，并将它们放在classpath上。你可以自定义使用的HTTP客户端，当使用Apache时提供`org.apache.http.impl.client.CloseableHttpClient`，当使用OK HTTP时提供`okhttp3.OkHttpClient`，当使用Apache HC5时提供`org.apache.hc.client5.http.impl.classic.CloseableHttpClient`的Bean。

Spring Cloud OpenFeign *不*为feign提供以下Bean，但仍然从应用上下文中查找这些类型的Bean来创建feign客户端。

- `Logger.Level`
- `Retryer`
- `ErrorDecoder`
- `Request.Options`
- `Collection<RequestInterceptor>`
- `SetterFactory`
- `QueryMapEncoder`
- `Capability` (默认提供`MicrometerCapability`)

默认情况下，创建一个类型为`Retryer.NEVER_RETRY`的bean，它将禁止重试。注意这个重试行为与Feign的默认行为不同，它将自动重试IOExceptions，将其视为瞬时网络相关的异常，以及从ErrorDecoder抛出的任何RetryableException。

创建一个这些类型的Bean，并将其放置在`@FeignClient`配置中（如上面的`FooConfiguration`），允许你覆盖所述的每一个Bean。

Example

```java
@Configuration
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}
```

这就用`feign.Contract.Default`替换了`SpringMvcContract`，并在`RequestInterceptor`的集合中添加了一个`RequestInterceptor`。

`@FeignClient`也可以使用配置属性进行配置。

application.yml

```yaml
feign:
    client:
        config:
            feignName:
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: full
                errorDecoder: com.example.SimpleErrorDecoder
                retryer: com.example.SimpleRetryer
                defaultQueryParameters:
                    query: queryValue
                defaultRequestHeaders:
                    header: headerValue
                requestInterceptors:
                    - com.example.FooRequestInterceptor
                    - com.example.BarRequestInterceptor
                decode404: false
                encoder: com.example.SimpleEncoder
                decoder: com.example.SimpleDecoder
                contract: com.example.SimpleContract
                capabilities:
                    - com.example.FooCapability
                    - com.example.BarCapability
                metrics.enabled: false
```

默认配置可以在`@EnableFeignClients`属性`defaultConfiguration`中指定，其方式与上述类似。不同的是，这个配置将适用于*所有*feign客户。

如果你喜欢使用配置属性来配置所有的`@FeignClient`，你可以创建带有`default`feign名称的配置属性。

你可以使用`feign.client.config.feignName.defaultQueryParameters`和`feign.client.config.feignName.defaultRequestHeaders`来指定查询参数和头信息，这些参数将在每个名为`feignName`的客户端的请求中发送。

application.yml

```yaml
feign:
    client:
        config:
            default:
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: basic
```

如果我们同时创建了`@Configuration`bean和配置属性，配置属性将获胜。它将覆盖`@Configuration`的值。但如果你想改变优先级为`@Configuration`，你可以把`feign.client.default-to-properties`改为`false`。

如果我们想创建多个具有相同名称或网址的feign客户端，使它们指向同一个服务器，但每个客户端都有不同的自定义配置，那么我们必须使用`@FeignClient`的`contextId`属性，以避免这些配置Bean的名称冲突。

```java
@FeignClient(contextId = "fooClient", name = "stores", configuration = FooConfiguration.class)
public interface FooClient {
    //..
}
```

```java
@FeignClient(contextId = "barClient", name = "stores", configuration = BarConfiguration.class)
public interface BarClient {
    //..
}
```

也可以配置FeignClient不继承父环境的bean。你可以通过重写`inheritParentConfiguration()`在`FeignClientConfigurer`bean中返回`false`来实现。

```java
@Configuration
public class CustomConfiguration{

@Bean
public FeignClientConfigurer feignClientConfigurer() {
            return new FeignClientConfigurer() {

                @Override
                public boolean inheritParentConfiguration() {
                    return false;
                }
            };

        }
}
```

> 默认情况下，Feign客户端不对斜线`/`字符进行编码。你可以通过设置`feign.client.decodeSlash`的值为`false`来改变这一行为。

#### 1.2.1. SpringEncoder configuration

在我们提供的 `SpringEncoder` 中，我们为二进制内容类型设置了 `null` 字符集，为所有其他类型设置了 `UTF-8`。

你可以通过设置`feign.encoder.charset-from-content-type`的值为`true`来修改这一行为，从`Content-Type`头的字符集中获得字符集。

### 1.3. 超时处理

我们可以在默认客户端和命名客户端上配置超时。OpenFeign使用两个超时参数。

- `connectTimeout`防止因服务器处理时间过长而阻塞调用者。
- `readTimeout`从连接建立时开始应用，当返回响应的时间过长时触发。

> 如果服务器没有运行或可用，数据包的结果是*connection refused*。通信会以错误信息或回退的方式结束。如果 `connectTimeout` 设置得很低，这可能会在它之前发生。执行查找和接收这种数据包所需的时间导致了这一延迟的重要部分。它可以根据涉及DNS查询的远程主机而改变。

### 1.4. 手动创建Feign客户端

在某些情况下，可能需要对你的Feign客户进行定制，而使用上述方法是不可能的。在这种情况下，你可以使用[Feign Builder API](https://github.com/OpenFeign/feign/#basics)创建客户端。下面是一个例子，它创建了两个具有相同接口的Feign客户端，但每个客户端都配置了一个单独的请求拦截器。

```java
@Import(FeignClientsConfiguration.class)
class FooController {

    private FooClient fooClient;

    private FooClient adminClient;

    @Autowired
    public FooController(Client client, Encoder encoder, Decoder decoder, Contract contract, MicrometerCapability micrometerCapability) {
        this.fooClient = Feign.builder().client(client)
                .encoder(encoder)
                .decoder(decoder)
                .contract(contract)
                .addCapability(micrometerCapability)
                .requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
                .target(FooClient.class, "https://PROD-SVC");

        this.adminClient = Feign.builder().client(client)
                .encoder(encoder)
                .decoder(decoder)
                .contract(contract)
                .addCapability(micrometerCapability)
                .requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
                .target(FooClient.class, "https://PROD-SVC");
    }
}
```

> 在上面的例子中，`FeignClientsConfiguration.class`是由Spring Cloud OpenFeign提供的默认配置。

> `PROD-SVC`是客户将向其发出请求的服务名称。

> Feign `Contract`对象定义了哪些注解和值对接口有效。自动连接的`Contract`bean提供了对SpringMVC注释的支持，而不是默认的Feign本地注释。

> 你也可以使用`Builder`来配置FeignClient不从父级上下文中继承bean。你可以通过覆盖调用`Builder`上的`inheritParentContext(false)`来做到这一点。

### 1.5. Feign Spring Cloud 断路器

如果Spring Cloud CircuitBreaker在classpath上，并且`feign.circuitbreaker.enabled=true`，Feign将用断路器来包装所有方法。

要在每个客户的基础上禁用Spring Cloud CircuitBreaker的支持，需要创建一个vanilla `Feign.Builder`，其范围为 `prototype`，例如。

```java
@Configuration
public class FooConfiguration {
    @Bean
    @Scope("prototype")
    public Feign.Builder feignBuilder() {
        return Feign.builder();
    }
}
```

断路器的名称遵循这种模式`<feignClientName>#<calledMethod>`。当调用名称为 `foo` 的`@FeignClient` 时，被调用的接口方法为 `bar`，那么断路器的名称将是 `foo_bar`。

要启用Spring Cloud CircuitBreaker组，请设置`feign.circuitbreaker.group.enabled`属性为`true`（默认为`false`）。

### 1.6. Feign Spring Cloud 断路器熔断

Spring Cloud CircuitBreaker支持`fallback`的概念：一个默认的代码路径，在电路打开或出现错误时执行。要为一个给定的`@FeignClient`启用回退，将`fallback`属性设置为实现回退的类名。你还需要将你的实现声明为一个Spring Bean。

```java
@FeignClient(name = "test", url = "http://localhost:${server.port}/", fallback = Fallback.class)
    protected interface TestClient {

        @RequestMapping(method = RequestMethod.GET, value = "/hello")
        Hello getHello();

        @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
        String getException();

    }

    @Component
    static class Fallback implements TestClient {

        @Override
        public Hello getHello() {
            throw new NoFallbackAvailableException("Boom!", new RuntimeException());
        }

        @Override
        public String getException() {
            return "Fixed response";
        }

    }
```

如果需要访问触发`fallback`的原因，可以使用`@FeignClient`里面的`fallbackFactory`属性。

```java
@FeignClient(name = "testClientWithFactory", url = "http://localhost:${server.port}/",
            fallbackFactory = TestFallbackFactory.class)
    protected interface TestClientWithFactory {

        @RequestMapping(method = RequestMethod.GET, value = "/hello")
        Hello getHello();

        @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
        String getException();

    }

    @Component
    static class TestFallbackFactory implements FallbackFactory<FallbackWithFactory> {

        @Override
        public FallbackWithFactory create(Throwable cause) {
            return new FallbackWithFactory();
        }

    }

    static class FallbackWithFactory implements TestClientWithFactory {

        @Override
        public Hello getHello() {
            throw new NoFallbackAvailableException("Boom!", new RuntimeException());
        }

        @Override
        public String getException() {
            return "Fixed response";
        }

    }
```

### 1.7. Feign 和 @Primary

当使用Feign与Spring Cloud CircuitBreaker回退时，`ApplicationContext`中有多个相同类型的bean。这将导致`@Autowired`不工作，因为没有确切的一个bean，或一个被标记为主要的bean。为了解决这个问题，Spring Cloud OpenFeign将所有Feign实例标记为`@Primary`，这样Spring Framework就知道要注入哪个bean。在某些情况下，这可能是不可取的。要关闭这种行为，请将`@FeignClient`的`primary`属性设置为false。

```java
@FeignClient(name = "hello", primary = false)
public interface HelloClient {
    // methods here
}
```

### 1.8. Feign 支持继承

Feign通过单继承接口支持模板式的apis。这允许将常见的操作归入方便的基础接口。

UserService.java

```java
public interface UserService {

    @RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
    User getUser(@PathVariable("id") long id);
}
```

UserResource.java

```java
@RestController
public class UserResource implements UserService {

}
```

UserClient.java

```java
package project.user;

@FeignClient("users")
public interface UserClient extends UserService {

}
```

> 一般来说，在服务器和客户端之间共享一个接口是不可取的。它引入了紧耦合，而且实际上在目前的形式下也不能与Spring MVC一起工作（方法参数映射不被继承）。

### 1.9. Feign 请求/响应 压缩

你可以考虑为你的Feign请求启用请求或响应的GZIP压缩。你可以通过启用其中一个属性来做到这一点。

```properties
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

Feign请求压缩给你的设置与你可能为你的网络服务器设置的类似。

```properties
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

这些属性允许你对压缩的媒体类型和最小请求阈值长度进行选择。

对于除OkHttpClient以外的http客户端，可以启用默认的gzip解码器，以UTF-8编码解码gzip响应。

```properties
feign.compression.response.enabled=true
feign.compression.response.useGzipDecoder=true
```

### 1.10. Feign 日志

每个创建的Feign客户端都会创建一个记录器。默认情况下，日志记录器的名称是用于创建Feign客户端的接口的全类名称。Feign的日志只对`DEBUG`级别做出响应。

application.yml

```yaml
logging.level.project.user.UserClient: DEBUG
```

你可以为每个客户配置`Logger.Level`对象，告诉Feign要记录多少。选择是。

- `NONE` , 不记录(**DEFAULT**)。
 `BASIC` , 只记录请求方法和URL以及响应状态代码和执行时间。
- `HEADERS` , 记录基本信息以及请求和响应头信息。
- `FULL` , 记录请求和响应的头信息、正文和元数据。

例如，下面将设置`Logger.Level`为`FULL`。

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

### 1.11. Feign Capability 支持

Feign的能力暴露了Feign的核心组件，因此这些组件可以被修改。例如，这些功能可以使用 `Client`，对其进行*装饰*，并将装饰后的实例反馈给Feign。对`metrics libraries`的支持就是一个很好的现实例子。参见[Feign metrics](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#feign-metrics)。

创建一个或多个 `Capability` Bean，并把它们放在`@FeignClient` 配置中，可以让你注册它们并修改相关客户端的行为。

```java
@Configuration
public class FooConfiguration {
    @Bean
    Capability customCapability() {
        return new CustomCapability();
    }
}
```

### 1.12. Feign metrics

如果以下条件都是`true`，一个`MicrometerCapability`bean就会被创建和注册，这样你的Feign客户端就会向`Micrometer`发布`metrics`。

- `feign-micrometer`在classpath上
- `MeterRegistry` bean是可用的
- feign metrics properties设置为`true`（默认）
  - `feign.metrics.enabled=true` (对于所有客户端)
  - `feign.client.config.feignName.metrics.enabled=true` (对于单个客户端)

> 如果你的应用程序已经使用了Micrometer，启用`metrics`就像把`feign-micrometer`放到classpath上一样简单。

你也可以通过以下方式禁用该功能。

- 从你的classpath中排除`feign-micrometer`。
- 将feign的一个metrics properties设置为 "false"。
  - `feign.metrics.enabled=false`
  - `feign.client.config.feignName.metrics.enabled=false`

> `feign.metrics.enabled=false` 禁用对**所有**Feign客户端的`metrics`支持，而不考虑客户端级别的标志值。`feign.client.config.feignName.metrics.enabled` 。如果你想启用或禁用每个客户端的merics，不要设置`feign.metrics.enabled`，使用`feign.client.config.feignName.metrics.enabled`。

你也可以通过注册你自己的bean来定制`MicrometerCapability`。

```java
@Configuration
public class FooConfiguration {
    @Bean
    public MicrometerCapability micrometerCapability(MeterRegistry meterRegistry) {
        return new MicrometerCapability(meterRegistry);
    }
}
```

### 1.13.  Feign @QueryMap support

OpenFeign `@QueryMap`注解提供了对POJO的支持，可用作GET参数映射。不幸的是，默认的OpenFeign QueryMap注解与Spring不兼容，因为它缺少一个`value`属性。

Spring Cloud OpenFeign提供了一个等价的`@SpringQueryMap`注解，用于注解一个POJO或Map参数作为查询参数地图。

例如，`Params`类定义了参数`param1`和`param2`。

```java
// Params.java
public class Params {
    private String param1;
    private String param2;

    // [Getters and setters omitted for brevity]
}
```

下面的feign客户端通过使用`@SpringQueryMap`注解来使用`Params`类。

```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/demo")
    String demoEndpoint(@SpringQueryMap Params params);
}
```

如果你需要对生成的查询参数图有更多的控制，你可以实现一个自定义的`QueryMapEncoder`bean。

### 1.14. HATEOAS support

Spring提供了一些API来创建遵循[HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)原则的REST表示，[Spring Hateoas](https://spring.io/projects/spring-hateoas)和[Spring Data REST](https://spring.io/projects/spring-data-rest)。

如果你的项目使用`org.springframework.boot:spring-boot-starter-hateoas`启动器或`org.springframework.boot:spring-boot-starter-data-rest`启动器，Feign HATEOAS支持默认为启用。

当HATEOAS支持被启用时，Feign客户端被允许序列化和反序列化HATEOAS表示模型。[EntityModel](https://docs.spring.io/spring-hateoas/docs/1.0.0.M1/apidocs/org/springframework/hateoas/EntityModel.html), [CollectionModel](https://docs.spring.io/spring-hateoas/docs/1.0.0.M1/apidocs/org/springframework/hateoas/CollectionModel.html) 和 [PagedModel](https://docs.spring.io/spring-hateoas/docs/1.0.0.M1/apidocs/org/springframework/hateoas/PagedModel.html)。

```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/stores")
    CollectionModel<Store> getStores();
}
```

### 1.15. Spring @MatrixVariable Support

Spring Cloud OpenFeign提供对Spring `@MatrixVariable`注解的支持。

如果一个Map被作为方法参数传递，`@MatrixVariable`路径段是通过用`=`连接地图中的键值对而创建的。

如果传递的是不同的对象，那么`@MatrixVariable`注解中提供的`name`（如果定义了的话）或者注解的变量名将用`=·与提供的方法参数连接。

重要提示

即使在服务器端，Spring并不要求用户将路径段占位符的名称与矩阵变量的名称相同，因为这在客户端太模糊了，Spring Cloud OpenFeign要求你添加一个路径段占位符，其名称与`@MatrixVariable`注解（如果定义了）中提供的`name`或注释变量的名称一致。

For example:

```java
@GetMapping("/objects/links/{matrixVars}")
Map<String, List<String>> getObjects(@MatrixVariable Map<String, List<String>> matrixVars);
```

注意，变量名和路径段占位符都被称为`matrixVars`。

```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/stores")
    CollectionModel<Store> getStores();
}
```

### 1.16. Feign CollectionFormat support

我们通过提供`@CollectionFormat`注解来支持`feign.CollectionFormat`。你可以通过传递所需的`feign.CollectionFormat`作为注解值，用它来注解一个Feign客户端方法。

在下面的例子中，使用`CSV`格式而不是默认的`EXPLODED`来处理该方法。

```java
@FeignClient(name = "demo")
    protected interface PageableFeignClient {

        @CollectionFormat(feign.CollectionFormat.CSV)
        @GetMapping(path = "/page")
        ResponseEntity performRequest(Pageable page);

    }
```

> 在发送`Pageable`作为查询参数时设置`CSV`格式，以便正确编码。

### 1.17. Reactive Support

由于[OpenFeign项目](https://github.com/OpenFeign/feign)目前不支持Reactive客户端，如[Spring WebClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html)，Spring Cloud OpenFeign也不支持。一旦核心项目中出现，我们将在此添加对它的支持。

在这之前，我们建议使用[feign-reactive](https://github.com/Playtika/feign-reactive)来支持Spring WebClient。

#### 1.17.1. 早期初始化错误

根据你使用Feign客户端的方式，你可能在启动应用程序时看到初始化错误。为了解决这个问题，你可以在自动连接你的客户端时使用`ObjectProvider`。

```java
@Autowired
ObjectProvider<TestFeginClient> testFeginClient;
```

### 1.18. Spring Data Support

你可以考虑为支持`org.springframework.data.domain.Page`和`org.springframework.data.domain.Sort`解码而启用`Jackson`模块。

```properties
feign.autoconfiguration.jackson.enabled=true
```

### 1.19. Spring @RefreshScope Support

如果启用了Feign client refresh，每个feign客户端都是以`feign.Request.Options`作为刷新范围的bean来创建的。这意味着诸如`connectTimeout`和`readTimeout`等属性可以通过`POST /actuator/refresh`对任何Feign客户端实例进行刷新。

默认情况下，Feign客户端的刷新行为是禁用的。使用以下属性来启用刷新行为。

```properties
feign.client.refresh-enabled=true
```

> **不要用** `@RefreshScope` 注解来注释`@FeignClient`接口。

## 2. 配置属性

要查看所有Spring Cloud OpenFeign相关的配置属性列表，请查看[附录页面](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/appendix.html)。

{{#include ../license.md}}
