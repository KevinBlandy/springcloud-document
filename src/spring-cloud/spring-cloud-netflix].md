# Spring Cloud Netflix

- 当前版本：3.0.3
- 修改时间：2021年7月22日
- 官方文档：[https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/](https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-netflix](https://github.com/spring-cloud/spring-cloud-netflix)

Spring Cloud Netflix通过自动配置和绑定到Spring环境和其他Spring编程模型习语，为Spring Boot应用提供了Netflix OSS集成。通过一些简单的注释，您可以在应用程序中快速启用和配置通用模式，并使用经过实战考验的Netflix组件构建大型分布式系统。提供的模式包括服务发现(Eureka)、断路器(Hystrix)、智能路由(Zuul)和客户端负载均衡(Ribbon)。

## 1. 服务发现：Eureka Clinet

服务发现是基于微服务的体系结构的关键原则之一。尝试手动配置每个客户端或某种形式的约定可能很困难，而且可能易挂。Netflix的服务发现器和客户端Eureka，可以将服务器配置和部署为高可用性，每个服务器都可以将已注册服务的状态复制给其他服务器。

### 1.1. 如何引入Eureka Client

要在项目中包含Eureaka Client，请使用start中gruop ID为`org.springframework.cloud`和artifact ID为`spring-cloud-starter-netflix-eureka-client`的包，有关使用Spring cloud设置构建系统的详细信息，请参阅[Spring Cloud Project](https://spring.io/projects/spring-cloud)
页面。

### 1.2. 向Eureka注册服务

当客户端向Eureka注册时，它会提供关于自身的元数据，例如主机、端口、健康状态URl、主页和其他详细信息。Eureka接收来自属于某个服务的每个实例的心跳消息。如果在可配置的时间内检测心跳失败，实例客户端通常会从注册中心删除。

下面的例子展示了一个最小的Eureka客户端应用程序:
```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

注意，上面的示例只是一个普通的Spring Boot应用程序。通过包管理文件引入`spring-cloud-starter-netflix-Eureka-client`依赖，你的应用程序将自动注册到Eureka Server。定位 Eureka服务器需要配置，如下面的例子所示:

**application.yml**
```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

在前面的示例中，`defaultZone` 是一个缺省字符串回调值，它为任何不填写注册中心地址的客户端提供一个默认值。(换句话说：不填此项，eureka使用http://localhost:8761/eureka/作为默认地址)

> 📍 这个`defaultZone`属性是区分大小写的，并且需要驼峰命令法，因为`serviceUrl`属性是`Map<String, String>`。因此，`defaultZone`属性不遵循常规Spring Boot蛇形命名法约定。

默认的应用程序名称(即服务 ID)、虚拟主机和非安全端口(取自环境)分别为`${ spring.application.name }`、`${ spring.application.name }`和`${ server.port }`。

在包管理工具中引入`spring-cloud-starter-netflix-Eureka-client`,使得应用程序既成为Eureka示例，也成为客户端（它可以查询注册中心定位其他服务）。实例行为是由`euraka.instance.*`配置驱动的，但是你要确保你的应用程序配置`spring.application.name`值。

> 有关可配置选项的更多详细信息，请参阅[EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)和[EurekaClientConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java)。

要禁用Eureka发现客户端，你可以将`Eureka.Client.enabled`设置为`false`。当 `spring.cloud.Discovery.enabled`设置为`false`时，Eureka发现客户端也将被禁用。

### 1.3 在Eureka server上验证

如果`eureka.client.serviceur.defaultzone`的Url中嵌入了凭据(curl样式的，示例：user: password@localhost: 8761/eureka)，HTTP的基本身份验证将自动添加到Eureka客户机中。对于更负载的需求，你可以创建类型为：`DiscoveryClientOptionalArgs` 的`@bean`，并将`ClientFilter`实例插入其中，所有这些都应用于从客户机到服务器的调用。

当Eureka服务器需要客户端证书进行身份验证时，客户端证书和信任存储可以通过属性进行配置，如下例所示:

**application.yml**
```yml
eureka:
  client:
    tls:
      enabled: true
      key-store: <path-of-key-store>
      key-store-type: PKCS12
      key-store-password: <key-store-password>
      key-password: <key-password>
      trust-store: <path-of-trust-store>
      trust-store-type: PKCS12
      trust-store-password: <trust-store-password>
```

启用`Eureka.client.TLS.enabled `要为true才能启用Eureka客户端的TLS。当省略`eurea.client.tls.trust-store`时，将使用JVM默认信任存储。`Eureka.client.tls.key-store-type`和`eureka.client.tls.trust-store-type`的默认值是 PKCS12。如果省略密码属性，则假定为空密码。

> 由于 Eureka 中的限制，不可能支持每个服务器的基本授权凭证，因此只使用找到的第一组凭证。

如果你想定制Eureka HTTP客户端使用的`RestTemplate`，你可以创建一个 `EurekaClientHttpRequestFactorySupplier`，并编写自己的逻辑来生成`ClientHttpRequestFactory`实例。

### 1.4 状态页和健康指示器

Eureka 实例的状态页面和健康状态指示器分别默认为`/info` 和`/health`，这是Spring Boot Actuator应用程序中有用的端点默认位置。如果使用非默认的上下文路径或servlet路径(比如 server.servletPath =/custom) ，那么即使对于实现的应用程序也需要修改这些路径。下面的示例显示了这两个设置的默认值:

**application.yml**
```yml
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

这些链接显示在客户端使用的元数据中，并且在某些场景中用于决定是否向应用程序发送请求，因此如果这些请求是准确的，就很有帮助。

> 在Dalston(版本名字)中，当更改管理上下文路径时，还需要设置状态和运行状况检查url。这个要求从Edgware开始就被删除了

### 1.5 注册安全的应用程序

如果你的应用程序希望通过 HTTPS 进行联系，可以在EurekaInstanceConfigBean 中设置两个属性:

* `Eureka.instance.[nonsecurtenabled]=[false]`
* `Eureka.instance.[securePortEnabled]=[true]`

这样做使Eureka发布的实例信息显示了对安全通信的明确偏好。Spring Cloud `DiscoveryClient` 总是返回一个以 https开头的 URI，用于以这种方式配置的服务。类似地，当以这种方式配置服务时，Eureka (本机)实例信息具有一个安全的健康检查 URL。

由于Eureka内部的工作方式，它仍然为状态和主页发布一个不安全的URL，除非你也显式地覆盖这些内容。你可以使用占位符来配置eureka实例 url，如下面的例子所示:

**application.yml**
```yml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
    healthCheckUrl: https://${eureka.hostname}/health
    homePageUrl: https://${eureka.hostname}/
```

(注意 `${Eureka.hostname}`是一个本机占位符，只能在 Eureka 的后续版本中使用。对于Spring占位符也可以实现同样的功能ーー例如，使用 `${eureka.instance.hostname}`.)

> 如果你的应用程序运行在代理之后，并且SSL终止在代理中(例如，如果你作为服务运行在Cloud Foundry或其他平台上) ，然后，你需要确保“转发”的代理头被应用程序拦截和处理。如果在Spring Boot应用程序中嵌入的Tomcat容器对'X-Forwarded-\* '头有显式配置，这将自动生效。应用程序呈现给自身的链接是错误的(错误的主机、端口或协议)，这是配置错误的标志。


### 1.6 Eureka的健康检查

默认情况下，Eureka使用客户端心跳来确定客户端是否启动。除非另行指定，否则发现客户端不会根据Spring Boot执行器传播应用程序的当前运行状况检查状态。因此，在成功注册后，Eureka总是宣布应用程序处于`UP`状态。可以通过启用Eureka运行状况检查来更改此行为，这将导致将应用程序状态传播到Eureka。因此，每个其他应用程序不会向处于`UP`状态以外的应用程序发送流量。下面的示例显示如何为客户端启用运行状况检查。

**application.yml**
```yml
eureka:
  client:
    healthcheck:
      enabled: true
```

> `eureka.client.healthcheck.enabled=true`应该只在`application.yml`中设置。在`bootstrap.yml`中设置这个值会引起不好的副作用。例如在Eureka以`UNKNOWN`状态注册。

如果你需要对运行状况检查进行更多的控制，可以考虑实现自己的检查`com.netflix.appinfo.HealthCheckHandler`。


### 1.7 Eureka client和实例的元数据

花点时间了解Eureka元数据是如何工作的是值得的，这样您就可以在您的平台上以一种有意义的方式使用它。对于主机名、IP地址、端口号、状态页和健康检查等信息，有标准的元数据。这些信息发布在服务注册中心中，客户端使用它们以一种简单的方式联系服务。附加的元数据可以添加到实例注册的`eureka.instance.metadataMap`中，该元数据可以在远程客户机中访问。一般来说，附加的元数据不会改变客户机的行为，除非客户机知道元数据的含义。在一些特殊情况下，Spring Cloud已经为元数据映射赋予了意义，本文稍后将对此进行描述。

#### 1.7.1 在Cloud Foundry上使用Eureka

Cloud Foundry有一个全局路由器，这样同一个应用的所有实例都有相同的主机名(其他具有类似架构的PaaS解决方案也有相同的安排)。这并不一定是使用Eureka的障碍。但是，如果你使用路由器(建议或强制使用，这取决于平台的设置方式)，则需要显式设置主机名和端口号(安全或不安全)，以便它们使用路由器。你可能还希望使用实例元数据，以便能够区分客户机上的实例(例如，在自定义负载平衡器中)。默认情况下，`eureka.instance.instanceId`是`vcap.application.Instance_id`，示例如下:

**application.yml**
```yml
eureka:
  instance:
    hostname: ${vcap.application.uris[0]}
    nonSecurePort: 80
```

根据在Cloud Foundry实例中设置安全规则的方式，您能够注册并使用主机VM的IP地址来进行直接的服务到服务调用。这个特性在关键的Web服务(PWS)上还不可用。

#### 1.7.2 在AWS上使用Eureka

如果计划将应用部署到AWS云，则必须将Eureka实例配置为AWS感知。你可以通过如下方式定制`EurekaInstanceConfigBean`:

```java
@Bean
@Profile("!default")
public EurekaInstanceConfigBean eurekaInstanceConfig(InetUtils inetUtils) {
  EurekaInstanceConfigBean bean = new EurekaInstanceConfigBean(inetUtils);
  AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
  bean.setDataCenterInfo(info);
  return bean;
}
```

#### 1.7.3 修改Eureka的实例ID
一个普通的Netflix Eureka实例是用一个与它的主机名相等的ID注册的(也就是说，每个主机只有一个服务)。Spring Cloud Eureka提供了一个合理的默认值，定义如下:
`${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}`

一个例子是：`myhost:myappname:8080`。
通过使用Spring Cloud，您可以通过在`eureka.instance.instanceId`中提供唯一的标识符来覆盖这个值。，如下例所示:

**application.yml**
```yml
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

使用前面示例中显示的元数据和部署在localhost上的多个服务实例，将插入随机值以使实例唯一。在Cloud Foundry中，`vcap.application.instance_id`是在Spring Boot应用程序中自动填充的，因此不需要这个随机值。


### 1.8 使用EurekaClient

一旦你拥有了一个作为发现客户端的应用程序，您就可以使用它从`Eureka Server`发现服务实例。一种方法是使用本机`com.netflix.discovery.EurekaClient`(与Spring Cloud DiscoveryClient相反)，如下面的示例所示。

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```

> 不要在`@PostConstruct`方法或`@Scheduled`方法中使用`eurekclient`(或在`ApplicationContext`可能还没有启动的任何地方)。它是在`SmartLifecycle`中初始化的(`phase=0`)，所以你最早可以依赖它是在另一个具有更高阶段的`SmartLifecycle`中。

#### 1.8.1 EurekaCliet没有Jersey

默认情况下，eurekclient使用Spring的`RestTemplate`进行HTTP通信。如果您希望使用Jersey，则需要将Jersey依赖项添加到类路径中。下面的例子显示了你需要添加的包管理文件中:

```pom
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-client</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-core</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.jersey.contribs</groupId>
    <artifactId>jersey-apache-client4</artifactId>
</dependency>
```


### 1.9 替代原生Netflix EurakaClient

你不需要使用原始的`Netflix eurekclient`。在某种封装后面使用它通常更方便。Spring Cloud通过逻辑的Eureka服务标识符(VIPs)而不是物理url支持`Feign`(一个REST客户端构建器)和`Spring RestTemplate`的远程调用。你还可以使用`org.springframework.cloud.client.discovery.DiscoveryClient`它为发现客户端提供了一个简单的API(不特定于Netflix)，如下面的示例所示:

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```

### 1.10 为什么注册服务如此缓慢

作为一个实例还涉及到到注册中心的周期性心跳(通过客户机的serviceUrl)，默认持续时间为30秒。在实例、服务器和客户端本地缓存中都有相同的元数据之前，客户端无法发现服务(因此可能需要3次心跳)。可以通过设置`eureka.instance.leaseRenewalIntervalInSeconds`来修改周期。将其设置为小于30的值将加快使客户端连接到其他服务的过程。在生产中，使用默认值可能更好，因为服务器中的内部计算会对租期续期做出假设。


### 1.11 Zones

如果您已经将Eureka客户端部署到多个区域，您可能希望这些客户端在尝试其他区域中的服务之前先使用相同区域中的服务。要进行设置，需要正确配置Eureka客户端。
首先，您需要确保将Eureka服务器部署到每个区域，并且它们彼此是对等的。有关更多信息，请参阅区域和地区一节。

接下来，你需要告诉Eureka你的服务在哪个区域。可以通过使用`metadatmap`属性来实现。例如，service 1同时部署在zone 1和zone 2，则需要在service 1中设置如下Eureka属性:

**Service 1 in Zone 1**
```yml
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

**Service 1 in Zone 2**
```yml
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

### 1.12 刷新Eureka client

默认情况下，`EurekaClient bean`是可刷新的，这意味着可以更改和刷新Eureka客户机属性。当刷新发生时，客户端将从Eureka服务器注销，并且可能有一段短暂的时间内给定服务的所有实例都不可用。消除这种情况的一种方法是禁用刷新Eureka客户机的功能。设置`eureka.client.refresh.enable=false`。


### 1.13 使用SpringCloud负载均衡的Eureka

我们提供对`Spring CLoud LoadBalancer ZonePreferenceServiceInstanceListSupplier`的支持。Eureka实例元数据(Eureka.instance. metadatmap zone)中的zone值用于设置spring-cloud-loadbalancer-zone属性的值，该属性用于按zone过滤服务实例。

如果没有这个属性，并且`spring.cloud.loadbalancer.eureka.approximateZoneFromHostname`标志被设置为true，那么它可以使用服务器主机名中的域名作为区域的代理。

如果没有其他区域数据来源，则基于客户端配置（而不是实例配置）进行猜测。 我们采取`eureka.client.availabilityZones`，它是从区域名称到区域列表的地图，并释放了实例自己的区域的第一个区域（即`eureka.client.region`，默认为`us-east-1`，用于与本机Netflix的兼容性）。



## 2. 服务发现：Eureka Server

介绍搭建Eureka服务器的操作步骤。

### 2.1 如何包含Eureka Server

要将`Eureka Server`包含到您的项目中，使用gourp ID为`org.springframework.cloud`和 artifact ID为`spring-cloud-start-netflix- Eureka-Server`的启动器。有关使用当前`Spring Cloud Release Train`设置构建系统的详细信息，请参阅Spring Cloud Project页面。

> 如果您的项目已经使用`Thymeleaf`作为模板引擎，Eureka服务器的`Freemarker`模板可能无法正确加载。在这种情况下，有必要手动配置模板加载器:

**application.yml**
```yml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    prefer-file-system-access: false
```

### 2.2 如何运行Eureka Server

下面的例子展示了一个最小的Eureka服务器:

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

服务器具有主页，其中包含UI界面和HTTP API端点，作用与/eureka/*。

以下链接有一些Eureka背景知识，[flux capacitor](https://github.com/cfregly/fluxcapacitor/wiki/NetflixOSS-FAQ#eureka-service-discovery-load-balancer) and [google group discussion](https://groups.google.com/forum/?fromgroups#!topic/eureka_netflix/g3p2r7gHnN0).

由于Gradle的依赖解析规则和缺少父bom特性，依赖`spring-cloud-start-netflix-eureka-server`可能会导致应用启动失败。为了解决这个问题，添加`Spring Boot Gradle`插件，并导入Spring云启动器的父bom如下:

**build.gradle**

```gradle
buildscript {
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:{spring-boot-docs-version}")
  }
}

apply plugin: "spring-boot"

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:{spring-cloud-version}"
  }
}
```

### 2.3 高可用，Zones 和 Regions

Eureka服务器没有后端存储，但是注册中心中的服务实例都必须发送心跳以保持其状态的更新(所以这可以在内存中完成)。客户端也有一个Eureka注册的内存缓存(所以他们不必为每个服务请求都去注册中心)。
默认情况下，每个Eureka服务器也是一个Eureka客户端，并且需要(至少一个)服务URL来定位对等点。如果你不提供它，服务也会运行并工作，但它会用许多关于无法注册进对等注册中心的信息记录填充你的日志。

### 2.4 独立模式

两个缓存(客户端和服务器)和心跳的结合使独立的Eureka服务器在故障时相当有弹性，只要有某种监控或弹性运行时(如Cloud Foundry)保持它的活力。在独立模式下，你可能更希望关闭客户端行为，这样它就不会一直尝试访问其他节点而失败。下面的例子展示了如何关闭客户端行为:

**application.yml (独立的Eureka服务器)**

```yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

注意`serviceUrl`指向与本地实例相同的主机。

### 2.5 对等模式
通过运行多个实例并要求它们相互注册，Eureka可以变得更加灵活和可用。事实上，这是默认的行为，所以你只需要向对等体添加一个有效的serviceUrl，如下面的例子所示:

**application.yml (两个对等的Eureka服务器)**
```yml
---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: https://peer2/eureka/

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/
```

在上面的示例中，我们有一个YAML文件，可以通过在不同的Spring文件中配置运行它来在两台主机(peer1和peer2)上运行相同的服务器。通过操作`/etc/hosts`解析主机名，您可以使用此配置在单个主机上测试对等感知(在生产中这样做没有多大价值)。事实上，如果运行在知道自己主机名的机器上(默认情况下，使用`java.net.InetAddress`查找)，就不需要`eureka.instance.hostname`。

你可以在一个系统中添加多个对等体，只要它们彼此至少有一个连接，它们就会同步自己的注册。如果对等点在物理上是分离的(在一个数据中心内或多个数据中心之间)，那么系统原则上可以承受`split-brain`类型的故障。你可以在一个系统中添加多个对等体，只要它们都是直接连接的，它们就会在自己之间同步注册。

**application.yml (三个对等的eureka服务器)**
```yml
eureka:
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2

---
spring:
  profiles: peer3
eureka:
  instance:
    hostname: peer3
```

### 2.6 何时选择IP地址

在某些情况下，Eureka最好发布服务的IP地址而不是主机名。设置`eureka.instance.preferipaddress=true`，当应用程序向eureka注册时，它使用它的IP地址而不是它的主机名。

> 如果主机名不能由Java确定，则将IP地址发送给Eureka。设置主机名的唯一方法是设置eureka.instance.hostname属性。你可以在运行时使用环境变量设置主机名—例如：`eureka.instance.hostname=${HOST_NAME}`。

### 2.7 保护Eureka Server

只需通过`Spring-boot-starter-Security`将`Spring Security`添加到服务器的包管理文件中，就可以保护Eureka服务器。默认情况下，当Spring Security在包管理文件中，它将要求一个有效的CSRF令牌被发送到应用程序。Eureka客户端通常不会拥有一个有效的跨站请求伪造(CSRF)令牌，你需要为`/Eureka/**`端点禁用这个要求。例如:

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

有关CSRF的更多信息，请参阅[Spring Security](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf)文档。
在Spring Cloud Samples [repo](https://github.com/spring-cloud-samples/eureka/tree/Eureka-With-Security)中可以找到一个演示Eureka Server。


### 2.8 JDK11支持

JDK 11删除了Eureka服务器所依赖的JAXB模块。如果你想在运行Eureka服务器时使用JDK 11，你必须在你的POM或Gradle文件中包含这些依赖项。
```pom
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
```

## 3. 配置属性

要查看所有Spring Cloud Netflix相关配置属性的列表，请检查[附录页面](https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/appendix.html)。
