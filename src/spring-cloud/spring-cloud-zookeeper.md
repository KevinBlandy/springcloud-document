# Spring Cloud Zookeeper

- 当前版本：3.0.3
- 修改时间：2021年7月26日
- 官方文档：[https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-zookeeper](https://github.com/spring-cloud/spring-cloud-zookeeper)

该项目通过自动配置与Spring环境和其他Spring应用程序的绑定，为SpringBoot应用程序提供Zookeeper的集成。通过一些注解，你可以在你的应用程序中快速启用和配置常见的模式，并使用基于Zookeeper的组件构建大型分布式系统。所提供的模式包括服务发现和配置。该项目还通过与Spring Cloud LoadBalancer的集成提供了客户端的负载平衡。

## 1. 快速开始

这篇快速入门文章介绍了使用Spring Cloud Zookeeper进行服务发现和分布式配置。

### 1.1. Discovery Client 用法

要在应用程序中使用这些功能，你需要将`spring-cloud-zookeeper-core` 和 `spring-cloud-zookeeper-discovery`引入你的Spring Boot 应用程序依赖中。添加依赖关系的最方便方式是使用`Spring Boot starter`：`org.springframework.cloud:spring-cloud-starter-zookeeper-discovery`。我们推荐使用建议管理和`spring-boot-starter-parent`。下面的例子展示了一个典型的Maven配置：

**pom.xml**

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
      <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
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

下面的例子展示了一个典型的Gradle设置：

**build.gradle**

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
  implementation 'org.springframework.cloud:spring-cloud-starter-zookeeper-discovery'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

> 根据你使用的版本，你可能需要调整项目中使用费的Apache Zookeeper版本，你可以在安装Zookeeper(之后的章节)部分时查看更多信息。

现在，你可以创建标准的Spring Boot应用程序了，例如以下的HTTP Server：

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

此HTTP服务器运行时,它连接到Zookeeper，运行在默认的本地端口2181。要调整启动方式，你可以使用`appcalication.propertis`更改Zookeeper设置，如下面的示例：

```yml
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```

现在你可以使用 `DiscoveryClient`, `@LoadBalanced RestTemplate`, 或者 `@LoadBalanced WebClient.Builder` 从zookeeper检索服务和实例数据，如以下示例所示：

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

### 1.2. 分布式配置用法

要在应用程序中使用这些功能，你需要构建依赖于`spring-cloud-zookeeper-core` 和`spring-cloud-zookeeper-config`的Spring Boot 应用程序 。添加依赖最方便的方式是使用`Spring boot Starter`：`org.springframework.cloud:spring-cloud-starter-zookeeper-config`。我们推荐使用依赖管理和：`spring-boot-starter-parent`。以下示例显示了典型的Maven配置：

**pom.xml**

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
      <artifactId>spring-cloud-starter-zookeeper-config</artifactId>
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

以下示例显示了典型的Gradle Setup：

**build.gradle**

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
  implementation 'org.springframework.cloud:spring-cloud-starter-zookeeper-config'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

> 根据您使用的版本，您可能需要调整项目中使用的apache zookeeper版本。 您可以在安装zookeeper部分中了解更多信息。



## 2. 安装Zookeeper

有关如何安装[zookeeper](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)的说明，请参阅安装文档。

Spring Cloud Zookeeper在幕后使用Apache Curator。虽然Zookeeper 3.5.x仍被Zookeeper开发团队认为是“beta”版，但现实是它被许多用户使用到生产环境。然而，Zookeeper 3.4.x也在生产中使用。在Apache Curater 4.0之前，通过两个版本的Apache Cureator支持两个版本的ZooKeeper。从Curator 4.0开始，Zookeeper的两个版本都通过相同的Curator库来支持。

如果你集成的是3.4版本，你需要改变Zookeeper的依赖关系，这个依赖关系是与curator一起提供的，也就是`spring-cloud-zookeeper`。要做到这一点，只需排除该依赖关系，并添加3.4.x版本，如下所示:

**maven**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-all</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.12</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**gradle**

```groovy
compile('org.springframework.cloud:spring-cloud-starter-zookeeper-all') {
  exclude group: 'org.apache.zookeeper', module: 'zookeeper'
}
compile('org.apache.zookeeper:zookeeper:3.4.12') {
  exclude group: 'org.slf4j', module: 'slf4j-log4j12'
}
```

## 3. Zookeeper的服务发现

服务发现是基于微服务的架构的关键原则之一。试图手工配置每个客户端或某种形式的约定是很难做到的，而且可能出问题。[Curator](https://curator.apache.org/)（Zookeeper的一个Java库）通过服务发现扩展提供了服务发现。Spring Cloud Zookeeper使用这个扩展进行服务注册和发现。

### 3.1. 激活

包含`org.springframework.cloud:spring-cloud-starter-zookeeper-discovery`的依赖，可以实现自动配置，设置`Spring Cloud Zookeeper Discovery`。

> 对于Web功能，你仍然需要包括`org.springframework.boot:spring-boot-starter-web`。

> 当使用3.4版本的Zookeeper时，你需要改变包括依赖关系的方式，如[这里](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/#spring-cloud-zookeeper-install)所述。

### 3.2. 使用Zookeeper注册

当一个客户端在Zookeeper注册时，它提供关于自身的元数据（例如主机和端口，ID和名称）。

以下示例显示了zookeeper客户端：

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

> 上面的应用是一个普通的Spring Boot 应用程序

如果zookeeper位于`localhost：2181`以外的某个地方，则配置必须提供服务器的位置，如以下示例所示：

**application.yml**

```yml
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```

> 如果你使用`Spring Cloud Zookeeper`配置，前面的例子中显示的值需要放在bootstrap.yml而不是application.yml中。

默认的服务名称、实例ID和端口（取自`Environment`）分别为`${spring.application.name}`、Spring Contenxt ID和`${server.port}`。

在classpath上有`spring-cloud-starter-zookeeper-discovery`，使应用程序成为Zookeeper的 service（即它自己注册）和 "client"（即它可以查询Zookeeper以定位其他服务）。

如果你想禁用Zookeeper发现客户端，你可以将`spring.cloud.zookeeper.discovery.enabled`设置为`false`。

### 3.3. 使用DiscoveryClient

Spring Cloud支持[Feign](https://www.springcloud.io/docs/springcloud/spring-cloud-openfeign/)（一个REST客户端构建器）、[Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/ascii)和[Spring WebFlux](https://cloud.spring.io/spring-cloud-commons/reference/html/#loadbalanced-webclient)，使用逻辑服务名称而不是物理URL。

你也可以使用`org.springframework.cloud.client.discovery.DiscoveryClient`，它为发现客户端提供了一个简单的API，并不针对`Netflix`，如下例所示:

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

## 4. 使用Spring Cloud Zookeeper与Spring Cloud组件

使Feign、Spring Cloud Gateway和Spring Cloud LoadBalancer都与Spring Cloud Zookeeper一起工作。

### 4.1. Zookeeper的Spring Cloud LoadBalancer

`Spring Cloud Zookeeper`提供了一个`Spring Cloud LoadBalancer ServiceInstanceListSupplier`的实现。当你使用`spring-cloud-starter-zookeeper-discovery`时，`Spring Cloud LoadBalancer`被自动配置为默认使用`ZookeeperServiceInstanceListSupplier`。

> 如果你之前在Zookeeper中使用`StickyRule`，那么在当前的栈中，它的替代品是`SC LoadBalancer`中的`SameInstancePreferenceServiceInstanceListSupplier`。你可以在[Spring Cloud Commons](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)的文档中阅读如何设置它。



## 5. Spring Cloud Zookeeper 和服务注册

spring Cloud Zookeeper实现了`ServiceRegistry`接口，让开发者以编程方式注册任意的服务。

`ServiceInstanceRegistration`类提供了一个`builder()`方法来创建一个可以被`ServiceRegistry`使用的注册对象，如下面的例子所示:

```java
@Autowired
private ZookeeperServiceRegistry serviceRegistry;

public void registerThings() {
    ZookeeperRegistration registration = ServiceInstanceRegistration.builder()
            .defaultUriSpec()
            .address("anyUrl")
            .port(10)
            .name("/a/b/c/d/anotherservice")
            .build();
    this.serviceRegistry.register(registration);
}
```

### 5.1. 实例状态

`Netflix Eureka`支持拥有在服务器上注册的`OUT_OF_SERVICE`的实例。这些实例不会作为活动服务实例返回。这对诸如`blue/green`部署的行为很有用。(注意，Curator服务发现配置不支持这种行为）。) 利用灵活的有效载荷，Spring Cloud Zookeeper通过更新一些特定的元数据，然后在`Spring Cloud LoadBalancer ZookeeperServiceInstanceListSupplier`中过滤这些元数据，实现`OUT_OF_SERVICE`。`ZookeeperServiceInstanceListSupplier`会过滤掉所有不等于`UP`的非空的实例状态。如果实例状态字段为空，为了向后兼容，它被认为是`UP`。要改变一个实例的状态，请用`OUT_OF_SERVICE`向`ServiceRegistry`实例状态执行器端点发出POST，如下例所示：

```sh
$ http POST http://localhost:8081/service-registry status=OUT_OF_SERVICE
```

> 上面的例子使用了[httpie.org](https://httpie.org/)的`http`命令。



## 6. Zookeeper的依赖关系

下面的主题涵盖了如何处理Spring Cloud Zookeeper的依赖关系。

- [使用Zookeeper依赖项](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/#spring-cloud-zookeeper-dependencies-using)
- [激活Zookeeper依赖项](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/#spring-cloud-zookeeper-dependencies-activating)
- [设置Zookeeper依赖项](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/#spring-cloud-zookeeper-dependencies-setting-up)
- [配置Zookeeper依赖项](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/#spring-cloud-zookeeper-dependencies-configuring)



### 6.1. 使用Zookeeper依赖项

`Spring Cloud Zookeeper`为您提供了一种可能性，可以将您的应用程序的依赖性作为属性提供。作为依赖关系，你可以了解在Zookeeper中注册的其他应用程序，并且你想通过`Feign`（一个REST客户端构建器）、`Spring RestTemplate`和`Spring WebFlux`来调用这些应用程序。

你也可以使用Zookeeper的依赖观察者功能来控制和监测你的依赖关系的状态。

### 6.2. 激活Zookeeper依赖项

包括对 `org.springframework.cloud:spring-cloud-starter-zookeeper-discovery` 的依赖，可以实现自动配置，设置 `Spring Cloud Zookeeper` 的依赖。即使你在属性中提供了依赖性，你也可以关闭依赖性。要这样做，请将 `spring.cloud.zookeeper.dependency.enabled` 属性设置为 false（默认为 true）。

### 6.3. 设置Zookeeper的依赖项

以下依赖性表示的例子。

**application.yml**

```yml
spring.application.name: yourServiceName
spring.cloud.zookeeper:
  dependencies:
    newsletter:
      path: /path/where/newsletter/has/registered/in/zookeeper
      loadBalancerType: ROUND_ROBIN
      contentTypeTemplate: application/vnd.newsletter.$version+json
      version: v1
      headers:
        header1:
            - value1
        header2:
            - value2
      required: false
      stubs: org.springframework:foo:stubs
    mailing:
      path: /path/where/mailing/has/registered/in/zookeeper
      loadBalancerType: ROUND_ROBIN
      contentTypeTemplate: application/vnd.mailing.$version+json
      version: v1
      required: true
```

接下来的几节将逐一介绍依赖关系的各个部分。根属性名称是 `spring.cloud.zookeeper.dependencies`。

### 6.3.1 Aliases

在根属性下面，你必须将每个依赖关系表示为一个别名。这是由于`Spring Cloud LoadBalancer`的限制，它要求将应用程序的ID放在URL中。因此，你不能传递任何复杂的路径，(例如`/myApp/myRoute/name`。) 别名是你用来代替`DiscoveryClient`、`Feign`或`RestTemplate`的`serviceId`的名字。

在前面的例子中，别名是 `newsletter` 和 `mailing`。下面的例子显示了Feign`在通讯别名下的用法。

```java
@FeignClient("newsletter")
public interface NewsletterService {
        @RequestMapping(method = RequestMethod.GET, value = "/newsletter")
        String getNewsletters();
}
```

### 6.3.2. Path

路径由 YAML`path`属性表示，是依赖关系在Zookeeper下注册的路径。正如上一节所述，`Spring Cloud LoadBalancer`在URL上操作。因此，这个路径不符合其要求。这就是为什么Spring Cloud Zookeeper将别名映射到适当的路径。

#### 6.3.3. Load Balancer Type

负载均衡器的类型由 YAML的`loadBalancerType`属性表示。

如果你知道在调用这个特定的依赖关系时必须应用什么样的负载平衡策略，你可以在YAML文件中提供，它将被自动应用。你可以选择以下一种负载平衡策略。

- STICKY: 一旦选择了该实例，就会一直被调用。
- RANDOM: 随机地挑选一个实例。
- ROUND_ROBIN: 对实例进行轮询迭代。

#### 6.3.4. Content-Type 模板和版本

内容类型模板和版本由`contentType`Template和版本属性表示。

`content-Type`的模板和版本由YAML的属性`contentTypeTemplate` 和 `version`表示 

如果你在Content-Type头中对你的API进行了更改，你就不要想在你的每个请求中添加这个头。另外，如果你想调用一个新版本的API，你不想在你的代码中漫游以提高API的版本。这就是为什么你可以在`contentTypeTemplate`中提供一个特殊的`$version`占位符。该占位符将由YAML属性的版本值来填充。考虑一下下面这个`contentTypeTemplate`的例子。

```
application/vnd.newsletter.$version+json
```

进一步考虑以下`version`：

```
v1
```

`contentTypeTemplate`和版本的组合为每个请求创建一个`Content-Type`头，如下所示。

```
application/vnd.newsletter.v1+json
```

#### 6.3.5. 默认头

默认的头文件由YAML中的头文件映射表示。

有时，每次对依赖关系的调用都需要设置一些默认的头文件。为了不在代码中这样做，你可以在YAML文件中设置它们，如下面例子中的头文件部分所示。

```yml
headers:
    Accept:
        - text/html
        - application/xhtml+xml
    Cache-Control:
        - no-cache
```

该标题部分的结果是在你的HTTP请求中添加带有适当数值列表的`Accept`和`Cache-Control`标题。

#### 6.3.6. 必要的依赖

所需的依赖关系由YAML中的`required`属性来表示。

如果你的应用程序启动时，你的一个依赖项需要启动，你可以在YAML文件中设置`required: true`属性。

如果你的应用程序在启动时不能将所需的依赖关系本地化，它就会抛出一个异常，并且Spring Context无法设置。换句话说，如果所需的依赖性没有在Zookeeper中注册，你的应用程序就无法启动。

你可以在本文档后面阅读更多关于Spring Cloud Zookeeper Presence Checker的内容。

#### 6.3.7. Stubs

你可以提供一个以冒号分隔的路径，指向包含依赖关系的存根的JAR，如下面的例子中所示:

`stubs: org.springframework:myApp:stubs`

在：

* `org.springframework` 是 `groupId`。
* `myApp` 是 `artifactId`.
* `stubs` 是分类器. (请注意，stubs是默认值)

因为stubs是默认的分类器，所以前面的例子等同于下面的例子：

`stubs: org.springframework:myApp`

### 6.4.  配置Spring Cloud Zookeeper的依赖关系 

你可以设置以下属性来启用或禁用Zookeeper Dependencies的部分功能。

* `spring.cloud.zookeeper.dependencies`。如果你不设置这个属性，你就不能使用Zookeeper的依赖性。
* `spring.cloud.zookeeper.dependency.loadbalancer.enabled`（默认为启用）。开启 Zookeeper 特定的自定义负载平衡策略，包括 `ZookeeperServiceInstanceListSupplier` 和基于依赖的负载平衡` RestTemplate` 设置。
* `spring.cloud.zookeeper.dependency.headers.enabled`（默认为启用）。此属性注册了一个`FeignBlockingLoadBalancerClient`，它可以自动附加适当的头文件和内容类型的版本，如依赖性配置中提出的。如果没有这个设置，这两个参数不工作。
* `spring.cloud.zookeeper.dependency.resttemplate.enabled`（默认为启用）。启用后，该属性会修改 `@LoadBalanced-annotated RestTemplate` 的请求头，这样它就会传递头信息和内容类型与依赖性配置中设置的版本。没有这个设置，这两个参数就不起作用。

## 7.  Spring Cloud Zookeeper依赖性观察器

依赖观察者机制让你为你的依赖关系注册监听器。该功能实际上是观察者模式的一个实现。当一个依赖关系的状态发生变化（上升或下降）时，可以应用一些自定义逻辑。

### 7.1. 激活

需要启用`Spring Cloud Zookeeper Dependencies`功能才能使用`Dependency Watcher`机制。

### 7.2. 注册一个监听器

要注册一个监听器，你必须实现一个名为 `org.springframework.cloud.zookeeper.discovery.watcher.DependencyWatcherListener` 的接口，并将其注册为一个 bean。该接口给了你一个方法。

```java
void stateChanged(String dependencyName, DependencyState newState);
```

如果你想为一个特定的依赖关系注册一个监听器，依赖关系名称（`dependencyName`）将是你的具体实现的鉴别器。 `newState`为你提供关于你的依赖关系是否已经改变为`CONNECTED`或`DISCONNECTED`的信息。

### 7.3.  使用Presence Checker

与依赖观察器绑定的是名为 "Presence Checker "的功能。它可以让你在应用程序启动时提供自定义行为，以便根据你的依赖关系的状态做出反应。

抽象的默认实现是:

`org.springframework.cloud.zookeeper.discovery.watcher.presence.DependencyPresenceOnStartupVerifier`类是`org.springframework.cloud.zookeeper.discovery.watcher.presence.DefaultDependencyPresenceOnStartupVerifier`，它的工作方式如下:

1. 如果该依赖关系被标记为必需的，并且不在Zookeeper中，当你的应用程序启动时，它会抛出一个异常并关闭。
2. 如果依赖性不是必需的，`org.springframework.cloud.zookeeper.discovery.watcher.presence.LogMissingDependencyChecker`会在`WARN`级别记录依赖性的缺失。

因为`DefaultDependencyPresenceOnStartupVerifier`只在没有`DependencyPresenceOnStartupVerifier`类型的bean时才被注册，所以这个功能可以被重写。

## 8. 用Zookeeper进行分布式配置

Zookeeper提供了一个分层命名空间，让客户端存储任意的数据，如配置数据。Spring Cloud Zookeeper配置是配置服务器和客户端的替代品。配置是在特殊的 "bootstrap "阶段加载到Spring环境中的。配置默认存储在`/config`命名空间中。根据应用程序的名称和活动配置文件，创建多个`PropertySource`实例，以模仿Spring Cloud Config的属性解析顺序。例如，一个名称为`testApp`并具有`dev`配置文件的应用程序为其创建了以下属性源:

- `config/testApp,dev`
- `config/testApp`
- `config/application,dev`
- `config/application`

最具体的属性源在顶部，最不具体的在底部。`config/application`命名空间中的属性适用于所有使用zookeeper进行配置的应用程序。`config/testApp`命名空间中的属性只对名为`testApp`的服务实例可用。

目前，配置是在应用程序启动时读取的。向`/refresh`发送HTTP POST请求会导致配置被重新加载。观察配置命名空间（Zookeeper支持）目前还没有实现。

### 8.1. 激活

包括对 `org.springframework.cloud:spring-cloud-starter-zookeeper-config` 的依赖，可以实现自动配置Spring Cloud Zookeeper。

> 当使用3.4版本的Zookeeper时，你需要改变包括依赖关系的方式，如[这里](https://docs.spring.io/spring-cloud-zookeeper/docs/current/reference/html/#spring-cloud-zookeeper-install)所述。

### 8.2. Spring Boot配置数据导入

Spring Boot 2.4引入了一种通过`spring.config.import`属性导入配置数据的新方法。现在这是从Zookeeper获取配置的默认方式。

为了选择性地连接到Zookeeper进行配置，在`application.properties`中设置以下内容。

**application.properties**

```properties
spring.config.import=optional:zookeeper:
```

这将在"localhost:2181 "的默认位置连接到Zookeeper。移除`optional:` 前缀将导致Zookeeper配置在无法连接到Zookeeper时失败。要改变Zookeeper配置的连接属性，可以设置`spring.cloud.zookeeper.connect-string`或将连接字符串添加到`spring.config.import`语句中，如`spring.config.import=optional:zookeeper:myhost:2818`。`import`属性中的位置优先于`connect-string`属性。

Zookeeper 配置将尝试从基于 `spring.cloud.zookeeper.config.name`（默认为 spring.application.name 属性的值）和 `spring.cloud.zookeeper.config.default-context`（默认为应用）的四个自动上下文中加载值。如果你想指定上下文而不是使用计算出的上下文，你可以在`spring.config.import`语句中添加该信息。

**application.properties**

```properties
spring.config.import=optional:zookeeper:myhost:2181/contextone;/context/two
```

这将有选择地只从`/contextone`和`/context/two`加载配置。

> 通过`spring.config.import`导入的Spring Boot配置数据方法不需要一个引导文件（properties或yaml）

### 8.3. 定制

Zookeeper配置可以通过设置以下属性进行定制。

```yml
spring:
  cloud:
    zookeeper:
      config:
        enabled: true
        root: configuration
        defaultContext: apps
        profileSeparator: '::'
```

- `enabled`: 将此值设置为false，可禁用Zookeeper配置。
- `root`: 设置配置值的基础命名空间。
- `defaultContext`: 设置所有应用程序使用的名称。
- `profileSeparator`: 设置分隔符的值，用于在有配置文件的属性源中分隔配置文件的名称。

> 如果你设置了 `spring.cloud.bootstrap.enabled=true` 或 spring.config.use-legacy-processing=true`，或者包含了 `spring-cloud-starter-bootstrap`，那么上述值就需要放在 bootstrap.yml 而不是 application.yml 中。



### 8.4. 访问控制列表（ACLs)

你可以通过调用`CuratorFramework Bean`的`addAuthInfo`方法为Zookeeper ACL添加认证信息。实现这一目的的方法之一是提供你自己的`CuratorFramework` Bean，如下面的例子中所示:

```java
@BoostrapConfiguration
public class CustomCuratorFrameworkConfig {

  @Bean
  public CuratorFramework curatorFramework() {
    CuratorFramework curator = new CuratorFramework();
    curator.addAuthInfo("digest", "user:password".getBytes());
    return curator;
  }

}
```

请参考[ZookeeperAutoConfiguration](https://github.com/spring-cloud/spring-cloud-zookeeper/blob/master/spring-cloud-zookeeper-core/src/main/java/org/springframework/cloud/zookeeper/ZookeeperAutoConfiguration.java)类，看看`CuratorFramework` bean的默认配置。

另外，你也可以从一个依赖于现有`CuratorFramework` Bean的类中添加你的证书，如下面的例子中所示。

```java
@BoostrapConfiguration
public class DefaultCuratorFrameworkConfig {

  public ZookeeperConfig(CuratorFramework curator) {
    curator.addAuthInfo("digest", "user:password".getBytes());
  }

}
```

这个Bean的创建必须发生在`bootstrapping`阶段。你可以通过用 `@BootstrapConfiguration` 注释来注册配置类，并将它们包含在一个逗号分隔的列表中，你将该列表设置为资源`/META-INF/spring.plants` 文件中 `org.springframework.cloud.bootstrap.BootstrapConfiguration` 属性的值，如以下示例所示:

**resources/META-INF/spring.factories**

```properties
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
my.project.CustomCuratorFrameworkConfig,\
my.project.DefaultCuratorFrameworkConfig
```

{{#include ../license.md}}
