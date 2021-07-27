# Spring Cloud Consul

- 当前版本：3.0.3
- 修改时间：2021年7月23日
- 官方文档：[https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-consul](https://github.com/spring-cloud/spring-cloud-consul)

该项目通过自动配置和与Spring环境及其他Spring编程模型习语的绑定，为Spring Boot应用程序提供[Consul](https://consul.io/)集成。通过一些简单的注解，你可以在你的应用程序中快速启用和配置常见的模式，并使用基于Consul的组件构建大型分布式系统。提供的模式包括服务发现、控制总线和配置。通过与其他Spring Cloud项目的集成，提供了智能路由和客户端负载平衡、断路器。

## 1. 快速开始

这篇快速入门指南介绍了如何使用Spring Cloud Consul进行服务发现和分布式配置。

首先，在你的机器上运行Consul Agent。然后你就可以访问它，并将其作为Spring Cloud Consul的服务注册和配置源。

### 1.1. 服务发现

要在应用程序中使用这些功能，你可以将其构建为一个依赖`spring-cloud-consul-core`的Spring Boot应用程序。添加依赖的最方便方式是使用Spring Boot启动器：`org.springframework.cloud:spring-cloud-starter-consul-discovery`。我们建议使用依赖性管理和`spring-boot-starter-parent`。

下面的例子显示了一个典型的Maven配置。

pom.xml

```xml
<project>
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{spring-boot-version}</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

下面的例子显示了一个典型的Gradle设置。

build.gradle

```groovy
plugins {
  id 'org.springframework.boot' version ${spring-boot-version}
  id 'io.spring.dependency-management' version ${spring-dependency-management-version}
  id 'java'
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-consul-discovery'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

现在你可以创建一个标准的Spring Boot应用程序，比如下面的HTTP服务器。

```java
@SpringBootApplication
@RestController
public class Application {

    @GetMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

当这个HTTP服务器运行时，它会连接到运行在本地默认8500端口的Consul Agent。要修改启动行为，你可以通过`application.properties`来改变Consul Agent的位置，如下例所示。

```yaml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
```

现在你可以使用`DiscoveryClient`、`@LoadBalanced` `RestTemplate`或`@LoadBalanced` `WebClient.Builder`来从`Consul`检索服务和实例数据，如以下例子所示。

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri().toString();
    }
    return null;
}
```

### 1.2. 分布式配置

要在应用程序中使用这些功能，您可以将其构建为依赖`spring-cloud-consul-core`和`spring-cloud-consul-config`的Spring Boot应用程序。添加依赖的最便捷方式是使用Spring Boot启动器：`org.springframework.cloud:spring-cloud-starter-consul-config`。我们建议使用依赖性管理和`spring-boot-starter-parent`。

下面的例子展示了一个典型的Maven配置。

pom.xml

```xml
<project>
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{spring-boot-version}</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

下面的例子显示了一个典型的Gradle设置。

build.gradle

```groovy
plugins {
  id 'org.springframework.boot' version ${spring-boot-version}
  id 'io.spring.dependency-management' version ${spring-dependency-management-version}
  id 'java'
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-consul-config'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

现在你可以创建一个标准的Spring Boot应用程序，比如下面的HTTP服务器。

```java
@SpringBootApplication
@RestController
public class Application {

    @GetMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

该应用程序从Consul检索配置数据。

> 如果你使用Spring Cloud Consul配置，你需要设置`spring.config.import`属性，以便与Consul绑定。你可以在[Spring Boot配置数据导入部分](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/#config-data-import)阅读更多信息。

## 2. 安装Consul

请参阅[安装文档](https://www.consul.io/intro/getting-started/install.html)，了解如何安装Consul。

## 3. Consul Agent

所有Spring Cloud Consul应用程序必须有一个Consul Agent客户端。默认情况下，代理客户端应该在`localhost:8500`。关于如何启动代理客户端以及如何连接到Consul代理服务器集群的具体细节，请参阅[代理文档](https://consul.io/docs/agent/basics.html)。对于开发来说，在你安装了consul之后，你可以使用以下命令启动一个Consul Agent。

```bash
./src/main/bash/local_run_consul.sh
```

这将在8500端口启动一个服务器模式的代理，用户界面在`localhost:8500`可用。

## 4. 使用Consul进行服务发现

服务发现是基于微服务的架构的关键原则之一。试图手工配置每个客户端或某种形式的约定是非常困难的，而且可能非常脆弱。Consul通过[HTTP API](https://www.consul.io/docs/agent/http.html)和[DNS](https://www.consul.io/docs/agent/dns.html)提供服务发现服务。Spring Cloud Consul利用HTTP API进行服务注册和发现。这并不妨碍非Spring Cloud应用程序利用DNS接口。Consul代理服务器在一个[集群](https://www.consul.io/docs/internals/architecture.html)中运行，该集群通过[流言协议](https://www.consul.io/docs/internals/gossip.html)进行通信并使用[Raft共识协议](https://www.consul.io/docs/internals/consensus.html)。

### 4.1. 如何激活

要激活Consul服务发现，请使用group为`org.springframework.cloud`和artifact id为`spring-cloud-starter-consul-discovery`的`starter`。请参阅[Spring Cloud项目页面](https://projects.spring.io/spring-cloud/)，了解关于用当前Spring Cloud发布列车设置构建系统的细节。

### 4.2. 注册到Consul

当客户端在Consul注册时，它提供了关于自己的元数据，如主机和端口、ID、名称和标签。默认情况下，会创建一个HTTP检查，Consul每隔10秒会点击`/actuator/health`端点。如果健康检查失败，服务实例将被标记为关键。

示例Consul客户端。

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

(即完全是正常的Spring Boot应用)。如果Consul客户端位于`localhost:8500`以外的地方，则需要配置来定位该客户端。例子。

application.yml

```yaml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
```

> 如果你使用[Spring Cloud Consul Config](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/#spring-cloud-consul-config)，并且设置了`spring.cloud.bootstrap.enabled=true`或`spring.config.use-legacy-processing=true`或使用`spring-cloud-starter-bootstrap`，那么上述值将需要放在`bootstrap.yml`而不是`application.yml`中。

默认的服务名称、实例ID和端口取自`Environment`，分别为`${spring.application.name}`、Spring Context ID和`${server.port}`。

要禁用Consul发现客户端，可以将`spring.cloud.consul.discovery.enabled`设为`false`。当`spring.cloud.discovery.enabled`被设置为`false`时，Consul发现客户端也将被禁用。

要禁用服务注册，你可以将`spring.cloud.consul.discovery.register`设为`false`。

#### 4.2.1. 将管Management为一项单独的服务进行注册

当管理服务器的端口被设置为与应用程序的端口不同时，通过设置`management.server.port`属性，管理服务将被注册为比应用程序服务单独的服务。比如说。

application.yml

```yaml
spring:
  application:
    name: myApp
management:
  server:
    port: 4452
```

上述配置将注册以下两个服务。

1. Application Service:

    ```text
    ID: myApp
    Name: myApp
    ```

2. Management Service:

    ```text
    ID: myApp-management
    Name: myApp-management
    ```

Management Service将从Application Service继承其`instanceId`和`serviceName`。比如说。

application.yml

```yaml
spring:
  application:
    name: myApp
management:
  server:
    port: 4452
spring:
  cloud:
    consul:
      discovery:
        instance-id: custom-service-id
        serviceName: myprefix-${spring.application.name}
```

上述配置将注册以下两个服务。

1. Application Service:

    ```text
    ID: custom-service-id
    Name: myprefix-myApp
    ```

2. Management Service:

    ```text
    ID: custom-service-id-management
    Name: myprefix-myApp-management
    ```

通过以下属性可以进行进一步的定制。

```properties
/** Port to register the management service under (defaults to management port) */
spring.cloud.consul.discovery.management-port

/** Suffix to use when registering management service (defaults to "management" */
spring.cloud.consul.discovery.management-suffix

/** Tags to use when registering management service (defaults to "management" */
spring.cloud.consul.discovery.management-tags
```

#### 4.2.2. HTTP健康检查

Consul实例的健康检查默认为"/actuator/health"，这是Spring Boot Actuator应用程序中健康端点的默认位置。如果你使用非默认的上下文路径或servlet路径（如`server.servletPath=/foo`）或管理端点路径（如`management.server.servlet.context-path=/admin`），你需要改变这一点，即使对于Actuator应用程序来说。

也可以配置Consul用于检查健康端点的时间间隔。"10s"和 "1m"分别代表10秒和1分钟。

这个例子说明了上述情况（更多选项请参见附录页中的 `spring.cloud.consul.discovery.health-check-*` 属性）。

application.yml

```yaml
spring:
  cloud:
    consul:
      discovery:
        healthCheckPath: ${management.server.servlet.context-path}/actuator/health
        healthCheckInterval: 15s
```

你可以通过设置 `spring.cloud.consul.discovery.register-health-check=false` 完全禁用 HTTP 健康检查。

**Applying Headers**

头信息可以应用于健康检查请求。例如，如果你试图注册一个使用[Vault Backend](https://github.com/spring-cloud/spring-cloud-config/blob/master/docs/src/main/asciidoc/spring-cloud-config.adoc#vault-backend)的[Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/)服务器。

application.yml

```yaml
spring:
  cloud:
    consul:
      discovery:
        health-check-headers:
          X-Config-Token: 6442e58b-d1ea-182e-cfa5-cf9cddef0722
```

根据HTTP标准，每个header可以有多个值，在这种情况下，可以提供一个数组。

application.yml

```yaml
spring:
  cloud:
    consul:
      discovery:
        health-check-headers:
          X-Config-Token:
            - "6442e58b-d1ea-182e-cfa5-cf9cddef0722"
            - "Some other value"
```

#### 4.2.3. Actuator Health Indicator(s)

如果服务实例是一个Spring Boot执行器应用程序，可以向它提供以下的Actuator health indicator.

**DiscoveryClientHealthIndicator**

当Consul服务发现被激活时，一个[DiscoverClientHealthIndicator](https://cloud.spring.io/spring-cloud-commons/2.2.x/reference/html/#health-indicator)被配置并提供给执行器健康端点使用。关于配置选项，请看[这里](https://cloud.spring.io/spring-cloud-commons/2.2.x/reference/html/#health-indicator)。

**ConsulHealthIndicator**

配置了一个指标来验证`ConsulClient`的健康状况。

默认情况下，它检索Consul领导节点状态和所有注册服务。在有许多注册服务的部署中，每次健康检查都要检索所有的服务，成本很高。要跳过服务检索，只检查领导节点状态，请设置 `spring.cloud.consul.health-indicator.include-services-query=false`。

要禁用该指标，请设置 `management.health.consul.enabled=false`

> 当应用程序运行在[bootstrap context mode](https://cloud.spring.io/spring-cloud-commons/2.2.x/reference/html/#the-bootstrap-application-context) (默认情况下)，该指标被加载到bootstrap上下文中，不提供给执行器健康端点。

#### 4.2.4. Metadata

Consul支持服务的元数据。Spring Cloud的`ServiceInstance`有一个`Map<String, String>`元数据字段，它是由服务元字段填充的。为了填充元字段，在`spring.cloud.consul.discovery.metadata`或`spring.cloud.consul.discovery.management-metadata`属性上设置值。

application.yml

```yaml
spring:
  cloud:
    consul:
      discovery:
        metadata:
          myfield: myvalue
          anotherfield: anothervalue
```

上述配置将产生一个服务，其元字段包含`myfield→myvalue`和`anotherfield→anothervalue`。

**Generated Metadata**

`Consul` 自动注册将自动生成一些条目。

Table 1. 自动生成的元数据

|Key|Value|
| --- | --- |
|`group`|属性 `spring.cloud.consul.discovery.instance-group` 。这个值只有在`instance-group`不是空的情况下才会生成。|
|`secure`|如果属性`spring.cloud.consul.discovery.scheme`等于`https`则为真，否则为: `false`。|
|属性 `spring.cloud.consul.discovery.default-zone-metadata-name` ，默认为 `zone`。|属性 `spring.cloud.consul.discovery.instance-zone` 。这个值只有在`instance-zone`不是空的情况下才会生成。|

> 旧版本的Spring Cloud Consul通过解析`spring.cloud.consul.discovery.tags`属性，从Spring Cloud Commons填充`ServiceInstance.getMetadata（）`方法。这已不再被支持，请迁移到使用`spring.cloud.consul.discovery.metadata` Map。

#### 4.2.5. 使Consul实例的ID唯一

默认情况下，consul实例的注册ID等于其Spring应用上下文ID。默认情况下，Spring应用上下文ID是`${spring.application.name}:comma,separated,profiles:${server.port}`。对于大多数情况，这将允许一个服务的多个实例在一台机器上运行。如果需要进一步的唯一性，使用Spring Cloud你可以通过在`spring.cloud.consul.discovery.instanceId`中提供一个唯一的标识符来覆盖这一点。比如说。

application.yml

```yaml
spring:
  cloud:
    consul:
      discovery:
        instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

有了这个元数据，并且在 localhost 上部署了多个服务实例，随机值就会在那里启动，使实例独一无二。在 Cloudfoundry 中，`vcap.application.instance_id` 将会在 Spring Boot 应用程序中自动填充，因此不需要随机值。

### 4.3. 查阅服务

#### 4.3.1. 使用负载均衡

Spring Cloud支持[Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign)（一个REST客户端构建器）和[Spring `RestTemplate`](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#rest-template-loadbalancer-client)，用于使用逻辑服务名称/ID而不是物理URL来查找服务。Feign和具有发现意识的RestTemplate都利用[Spring Cloud LoadBalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)进行客户端的负载均衡。

如果你想使用RestTemplate访问`STORES`服务，只需声明。

```java
@LoadBalanced
@Bean
public RestTemplate loadbalancedRestTemplate() {
     return new RestTemplate();
}
```

并像这样使用它（注意我们如何使用来自Consul的`STORES`服务名称/ID，而不是一个完全合格的域名）。

```java
@Autowired
RestTemplate restTemplate;

public String getFirstProduct() {
   return this.restTemplate.getForObject("https://STORES/products/1", String.class);
}
```

如果你在多个数据中心有Consul集群，并且你想访问另一个数据中心的服务，仅有服务名称/ID是不够的。在这种情况下，你可以使用属性`spring.cloud.consul.discovery.datacenters.STORES=dc-west`，其中STORES是服务名称/id，dc-west是STORES服务所在的数据中心，而不是一个完全合格的域名）。)

> Spring Cloud现在还提供对[Spring Cloud LoadBalancer](https://cloud.spring.io/spring-cloud-commons/reference/html/#_spring_resttemplate_as_a_load_balancer_client) 的支持。

#### 4.3.2. 使用DiscoveryClient

你也可以使用`org.springframework.cloud.client.discovery.DiscoveryClient`，它为发现客户端提供了一个简单的API，而不是专门针对Netflix的，比如说

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

### 4.4. Consul Catalog Watch

Consul Catalog Watch利用了consul[观察服务](https://www.consul.io/docs/agent/watches.html#services)的能力。目录监视进行阻塞的Consul HTTP API调用，以确定是否有任何服务改变。如果有新的服务数据，就会发布心跳事件。

要改变配置观察被调用的频率，请改变`spring.cloud.consul.config.discovery.catalog-services-watch-delay`。默认值是1000，单位是毫秒。延迟是指前一次调用结束后到下一次调用开始的时间量。

要禁用目录观察，请设置`spring.cloud.consul.discovery.catalogServicesWatch.enabled=false`。

该观察使用Spring的`TaskScheduler`来安排对consul的调用。默认情况下，它是一个`ThreadPoolTaskScheduler`，`poolSize`为1。 要改变`TaskScheduler`，创建一个`TaskScheduler`类型的bean，用`ConsulDiscoveryClientConfiguration.CATALOG_WATCH_TASK_SCHEDULER_NAME`常量命名。

## 5. 使用Consul的分布式配置

Consul提供了一个[Key/Value Store](https://consul.io/docs/agent/http/kv.html)来存储配置和其他元数据。Spring Cloud Consul配置是[配置服务器和客户端](https://github.com/spring-cloud/spring-cloud-config)的替代品。在特殊的 "bootstrap" 阶段，配置被加载到Spring环境中。配置默认存储在`/config`文件夹中。多个 `PropertySource` 实例根据应用程序的名称和活动配置文件创建，模仿Spring Cloud Config的属性解析顺序。例如，一个名称为 "testApp "并使用 "dev "配置文件的应用程序将创建以下属性源。

```text
config/testApp,dev/
config/testApp/
config/application,dev/
config/application/
```

最具体的属性来源在顶部，最不具体的在底部。`config/application`文件夹中的属性适用于所有使用consul进行配置的应用程序。`config/testApp`文件夹中的属性只适用于名为 "testApp "的服务实例。

目前，配置是在应用程序的启动时读取的。向`/refresh`发送HTTP POST将导致配置被重新加载。[Config Watch](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/#spring-cloud-consul-config-watch)也将自动检测变化并重新加载应用程序上下文。

### 5.1. 如何激活

要开始使用Consul配置，请使用 `org.springframework.cloud` group和 `spring-cloud-starter-consul-config` artifact id的starter。请参阅[Spring Cloud项目页面](https://projects.spring.io/spring-cloud/)，了解用当前的Spring Cloud Release Train设置构建系统的细节。

### 5.2. Spring Boot配置数据导入

Spring Boot 2.4引入了一种通过`spring.config.import`属性导入配置数据的新方法。现在这是从Consul获取配置的默认方式。

要选择性地连接到Consul，请在`application.properties`中设置以下内容。

application.properties

```properties
spring.config.import=optional:consul:
```

这将在 `http://localhost:8500` 的默认位置连接到Consul代理。移除`optional:`前缀将导致Consul配置在无法连接到Consul时失败。要改变Consul配置的连接属性，可以设置`spring.cloud.consul.host`和`spring.cloud.consul.port`或者将主机/端口对添加到`spring.config.import`语句中，例如，`spring.config.import=optional:consul:myhost:8500`。import属性中的位置优先于host和port属性。

Consul配置将根据`spring.cloud.consul.config.name`（默认为`spring.application.name`属性的值）和`spring.cloud.consul.config.default-context`（默认为`application`），尝试从四个自动上下文加载值。如果你想指定上下文而不是使用计算出来的上下文，你可以在`spring.config.import`语句中添加这些信息。

application.properties

```properties
spring.config.import=optional:consul:myhost:8500/contextone;/context/two
```

这将有选择地只从`/contextone`和`/context/two`加载配置。

> 通过`spring.config.import`导入的Spring Boot配置数据方法不需要一个`bootstrap`文件（properties或yaml）。

### 5.3. 定制化

Consul配置可以使用以下属性进行定制。

```yaml
spring:
  cloud:
    consul:
      config:
        enabled: true
        prefix: configuration
        defaultContext: apps
        profileSeparator: '::'
```

> 如果你设置了`spring.cloud.bootstrap.enabled=true`或`spring.config.use-legacy-processing=true`，或者包含了`spring-cloud-starter-bootstrap`，那么上述数值需要放在`bootstrap.yml`而不是`application.yml`中。

- `enabled`将此值设置为 "false"，可禁用Consul配置。
- `prefix`设置配置值的基础文件夹
- `defaultContext`设置所有应用程序使用的文件夹名称。
- `profileSeparator`设置分隔符的值，用于在有配置文件的属性源中分隔配置文件的名称。

### 5.4. Config Watch

Consul Config Watch利用了consul [watch a key prefix](https://www.consul.io/docs/agent/watches.html#keyprefix) 的能力。Config Watch进行阻塞式Consul HTTP API调用，以确定当前应用程序的任何相关配置数据是否发生了变化。如果有新的配置数据，就会发布一个刷新事件。这等同于调用`/refresh`执行器端点。

要改变配置观察被调用的频率，请改变`spring.cloud.consul.config.watch.delay`。默认值是1000，单位是毫秒。延迟是指上一次调用结束后到下一次调用开始的时间量。

要禁用配置观察，请设置`spring.cloud.consul.config.watch.enabled=false`。

观察使用Spring的`TaskScheduler`来安排对consul的调用。默认情况下，它是一个`ThreadPoolTaskScheduler`，`poolSize`为1。 要改变`TaskScheduler`，创建一个`TaskScheduler`类型的bean，用`ConsulConfigAutoConfiguration.CONFIG_WATCH_TASK_SCHEDULER_NAME`常量命名。

### 5.5. YAML或Properties的配置

相对于单独的键/值对，以YAML或Properties格式来存储属性的blob可能更方便。设置`spring.cloud.consul.config.format`属性为`YAML`或`properties`。例如，使用YAML。

```yaml
spring:
  cloud:
    consul:
      config:
        format: YAML
```

> 如果你设置了 `spring.cloud.bootstrap.enabled=true` 或 `spring.config.use-legacy-processing=true`，或者包含了 `spring-cloud-starter-bootstrap`，那么上述值就需要放在 `bootstrap.yml` 而不是 `application.yml` 中。

YAML 必须设置在`consul`中适当的data `key`中。使用上面的默认值，键会看起来像。

```text
config/testApp,dev/data
config/testApp/data
config/application,dev/data
config/application/data
```

你可以将YAML文档存储在上面列出的任何一个键中。

你可以使用`spring.cloud.consul.config.data-key`改变data key。

### 5.6. git2consul的配置

git2consul是一个Consul社区项目，它将文件从git仓库加载到Consul的单个键中。默认情况下，键的名字就是文件的名字。支持YAML和属性文件，文件扩展名分别为`.yml`和`.properties`。设置`spring.cloud.consul.config.format`属性为`FILES`。比如说。

bootstrap.yml

```yaml
spring:
  cloud:
    consul:
      config:
        format: FILES
```

给出`/config`中的以下键，`development` 配置文件和应用程序名称为`foo`。

```text
.gitignore
application.yml
bar.properties
foo-development.properties
foo-production.yml
foo.properties
master.ref
```

将创建以下配置资源。

```text
config/foo-development.properties
config/foo.properties
config/application.yml
```

每个key的value需要是一个格式正确的YAML或属性文件。

### 5.7. 快速失败

在某些情况下（如本地开发或某些测试场景），如果consul不能用于配置，可能会很方便，不会失败。设置`spring.cloud.consul.config.fail-fast=false`将导致配置模块记录一个警告而不是抛出一个异常。这将允许应用程序继续正常启动。

> 如果你设置了`spring.cloud.bootstrap.enabled=true`或`spring.config.use-legacy-processing=true`，或者包含了`spring-cloud-starter-bootstrap`，那么上述数值需要放在`bootstrap.yml`而不是`application.yml`中。

## 6. Consul 重试

如果你预计consul代理在你的应用程序启动时可能偶尔不可用，你可以要求它在失败后继续尝试。你需要把`spring-retry`和`spring-boot-starter-aop`添加到你的classpath中。默认行为是重试6次，初始回退间隔为1000ms，后续回退的指数乘数为1.1。你可以使用`spring.cloud.consul.retry.*`配置属性来配置这些属性（以及其他）。这在Spring Cloud Consul配置和发现注册中都适用。

> 为了完全控制重试，添加一个`@Bean`类型的`RetryOperationsInterceptor`，ID为 "consulRetryInterceptor"。Spring Retry有一个`RetryInterceptorBuilder`，可以很容易地创建一个。

## 7. Spring Cloud Bus 和 Consul

### 7.1. 如何激活

要开始使用Consul Bus，请使用 "org.springframework.cloud" group和 `spring-cloud-starter-consul-bus` artifact id的starter 。请参阅[Spring Cloud项目页面](https://projects.spring.io/spring-cloud/)，了解关于使用当前Spring Cloud发布列车设置构建系统的详细信息。

参见[Spring Cloud Bus](https://cloud.spring.io/spring-cloud-bus/)文档，了解可用的执行器端点以及如何发送自定义消息。

## 8. Circuit Breaker 和 Hystrix

应用程序可以使用由Spring Cloud Netflix项目提供的Hystrix熔断器，方法是在项目的pom.xml中包括这个启动器。`spring-cloud-starter-hystrix` 。Hystrix并不依赖Netflix发现客户端。`@EnableHystrix`注解应该放在一个配置类上（通常是主类）。然后方法可以用`@HystrixCommand`来注解，以受到熔断器的保护。更多细节见[文档](https://projects.spring.io/spring-cloud/spring-cloud.html#_circuit_breaker_hystrix_clients)。

### 9. Hystrix metrics aggregation with Turbine and Consul

Turbine（由Spring Cloud Netflix项目提供），聚合了多个实例Hystrix的指标流，因此仪表板可以显示一个聚合视图。Turbine使用`DiscoveryClient`接口来查找相关实例。要在Spring Cloud Consul中使用Turbine，请以类似于以下例子的方式配置Turbine应用程序。

pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-turbine</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

请注意，Turbine的依赖关系不是一个启动器。 turbine starter包括对Netflix Eureka的支持。

application.yml

```yaml
spring.application.name: turbine
applications: consulhystrixclient
turbine:
  aggregator:
    clusterConfig: ${applications}
  appConfig: ${applications}
```

`clusterConfig`和`appConfig`部分必须匹配，所以把逗号分隔的服务ID列表放到一个单独的配置属性中是很有用的。

Turbine.java

```java
@EnableTurbine
@SpringBootApplication
public class Turbine {
    public static void main(String[] args) {
        SpringApplication.run(DemoturbinecommonsApplication.class, args);
    }
}
```

## 10. 配置属性

要查看所有与Consul有关的配置属性列表，请查看[附录页面](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/appendix.html)。

{{#include ../license.md}}
