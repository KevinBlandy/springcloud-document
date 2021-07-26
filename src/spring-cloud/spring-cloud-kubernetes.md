# Spring Cloud Kubernetes

- 当前版本：2.0.3
- 修改时间：2021年7月24日
- 官方文档：[https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-projects/spring-cloud](https://github.com/spring-projects/spring-cloud)

## 1. 为什么需要Spring Cloud Kubernetes？

Spring Cloud Kubernetes提供了众所周知的Spring Cloud接口的实现，允许开发者在Kubernetes上构建和运行Spring Cloud应用。虽然这个项目在构建云原生应用时可能对你有用，但它也不是在Kubernetes上部署Spring Boot应用的必要条件。如果你刚刚开始在Kubernetes上运行你的Spring Boot应用，你只需要一个基本的Spring Boot应用和Kubernetes本身就可以完成很多事情。要想了解更多，你可以通过阅读[Spring Boot部署到Kubernetes的参考文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#cloud-deployment-kubernetes)，以及通过研讨会材料[Spring和Kubernetes](https://hackmd.io/@ryanjbaxter/spring-on-k8s-workshop)来开始。

## 2. Starters

Starters是方便的依赖性描述符，你可以在你的应用程序中包含它。包括一个starter，以获得功能集的依赖性和Spring Boot自动配置。以`spring-cloud-starter-kubernetes-fabric8`开头的启动器提供了使用[Fabric8 Kubernetes Java Client](https://github.com/fabric8io/kubernetes-client)的实现。以`spring-cloud-starter-kubernetes-client`开头的启动器提供了使用[Kubernetes Java Client](https://github.com/kubernetes-client/java)的实现。

### 2.1. 服务发实

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-fabric8</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-client</artifactId>
</dependency>
```

[Discovery Client](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#discoveryclient-for-kubernetes)实现，将服务名称解析为Kubernetes服务。

### 2.2. 配置中心

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-fabric8-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-client-config</artifactId>
</dependency>
```

从Kubernetes[ConfigMaps](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#configmap-propertysource)和[Secrets](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#secrets-propertysource)加载应用属性。当ConfigMap或Secret发生变化时，[重新加载](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#propertysource-reload)应用程序属性。

### 2.3. 所有Spring Cloud Kubernetes功能。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-fabric8-all</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-client-all</artifactId>
</dependency>
```

## 3. 用于Kubernetes的DiscoveryClient

该项目为[Kubernetes](https://kubernetes.io/)提供了[Discovery Client](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/DiscoveryClient.java)的实现。这个客户端可以让你按名称查询Kubernetes端点（见[services](https://kubernetes.io/docs/user-guide/services/)）。服务通常由Kubernetes API服务器公开，作为代表 `http` 和 `https` 地址的端点的集合，客户端可以从作为pod运行的Spring Boot应用程序中访问。

通过在你的项目中添加以下依赖关系，你可以免费获得这个东西。

Fabric8 Kubernetes Client

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-fabric8</artifactId>
</dependency>
```

Kubernetes Java Client

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-client</artifactId>
</dependency>
```

要启用`DiscoveryClient`的加载，请将`@EnableDiscoveryClient`添加到相应的配置或应用类中，如下例所示。

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

如下面的例子所示，你可以简单地通过自装配将客户端注入你的代码中。

```java
@Autowired
private DiscoveryClient discoveryClient;
```

你可以通过在`application.properties`中设置以下属性来选择从所有命名空间启用`DiscoveryClient`。

```properties
spring.cloud.kubernetes.discovery.all-namespaces=true
```

要发现未被kubernetes api服务器标记为 "ready" 的服务端点地址，可以在`application.properties`中设置以下属性（默认：`false`）。

```properties
spring.cloud.kubernetes.discovery.include-not-ready-addresses=true
```

> 这在为监控目的而发现服务时可能很有用，并能检查未准备好的服务实例的`/health`端点。

如果你的服务暴露了多个端口，你将需要指定`DiscoveryClient`应该使用哪个端口。`DiscoveryClient`将使用以下逻辑选择端口。

1. 如果服务有一个标签`primary-port-name`，它将使用标签值中指定名称的端口。
2. 如果没有标签，那么将使用`spring.cloud.kubernetes.discovery.primary-port-name`中指定的端口名称。
3. 如果以上两者都没有指定，它将使用名为 `https` 的端口。
4. 如果上述条件都不满足，它将使用名为 `http` 的端口。
5. 作为最后手段，它将选择端口列表中的第一个端口。

> 最后一个选项可能会导致非确定性的行为。请确保对你的服务和/或应用程序进行相应的配置。

默认情况下，所有的端口及其名称将被添加到`ServiceInstance`的元数据中。

如果出于任何原因，你需要禁用`DiscoveryClient`，你可以在`application.properties`中设置以下属性。

```properties
spring.cloud.kubernetes.discovery.enabled=false
```

一些Spring Cloud组件使用`DiscoveryClient`，以获取本地服务实例的信息。要做到这一点，你需要将Kubernetes服务名称与`spring.application.name`属性对齐。

> `spring.application.name`对于在Kubernetes中为应用程序注册的名称没有影响。

Spring Cloud Kubernetes还可以观察Kubernetes服务目录的变化，并相应地更新`DiscoveryClient`实现。为了启用这一功能，你需要在你的应用程序中的配置类上添加`@EnableScheduling`。

## 4. Kubernetes本地服务发现

Kubernetes本身能够进行（服务器端）服务发现（参见：[kubernetes.io/docs/concepts/services-networking/service/#discovering-services](https://kubernetes.io/docs/concepts/services-networking/service/#discovering-services)）。使用原生的kubernetes服务发现可以确保与其他工具的兼容性，例如Istio（[istio.io](https://istio.io/)），一个能够实现负载均衡、断路器、故障转移等功能的服务网。

然后，调用者服务只需要引用特定Kubernetes集群中可解析的名称。一个简单的实现可以使用一个指向完全合格域名（FQDN）的spring `RestTemplate`，例如`{service-name}.{namespace}.svc.{cluster}.local:{service-port}`。

此外，你可以使用Hystrix进行熔断。

- 通过在spring boot应用类中注解`@EnableCircuitBreaker`，在调用方实现熔断器。
- Fallback 功能，通过用`@HystrixCommand(fallbackMethod=`) 注释相应的方法。

## 5. Kubernetes PropertySource的实现

配置Spring Boot应用程序的最常见方法是创建`application.properties`或`application.yaml`或`application-profile.properties`或`application-profile.yaml`文件，其中包含为应用程序或Spring Boot启动器提供定制值的键值对。你可以通过指定系统属性或环境变量来覆盖这些属性。

### 5.1. 使用ConfigMap PropertySource

Kubernetes提供了一个名为[`ConfigMap`](https://kubernetes.io/docs/user-guide/configmap/)的资源，以键值对或嵌入的`application.properties`或`application.yaml`文件的形式将参数外化，传递给你的应用程序。[Spring Cloud Kubernetes Config](https://github.com/spring-cloud/spring-cloud-kubernetes/tree/master/spring-cloud-kubernetes-fabric8-config)项目使Kubernetes的`ConfigMap`实例在应用启动过程中可用，并在发现观察到的`ConfigMap`实例发生变化时，触发Bean或Spring上下文的热重载。

默认行为是基于Kubernete的`ConfigMap`创建`Fabric8ConfigMapPropertySource`，其`metadata.name`值为Spring应用程序的名称（由其`spring.application.name`属性定义）或在`bootstrap.properties`文件中定义的自定义名称，键为：`spring.cloud.kubernetes.config.name`。

然而，更高级的配置是可能的，你可以使用多个`ConfigMap`实例。`spring.cloud.kubernetes.config.sources`列表使这成为可能。例如，你可以定义以下`ConfigMap`实例。

```yaml
spring:
  application:
    name: cloud-k8s-app
  cloud:
    kubernetes:
      config:
        name: default-name
        namespace: default-namespace
        sources:
         # Spring Cloud Kubernetes looks up a ConfigMap named c1 in namespace default-namespace
         - name: c1
         # Spring Cloud Kubernetes looks up a ConfigMap named default-name in whatever namespace n2
         - namespace: n2
         # Spring Cloud Kubernetes looks up a ConfigMap named c3 in namespace n3
         - namespace: n3
           name: c3
```

在前面的例子中，如果没有设置`spring.cloud.kubernetes.config.namespace`，名为`c1`的`ConfigMap`将在应用程序运行的命名空间中被查找。

找到的任何匹配的`ConfigMap`将被处理如下。

- 应用单个配置属性。
- 将任何名为`application.yaml`的属性内容作为`yaml`应用。
- 将任何名为`application.properties`的属性内容作为一个属性文件应用。

上述流程的唯一例外是，当`ConfigMap`包含一个表明文件是YAML或属性文件的**single** key时。在这种情况下，键的名称不必是`application.yaml`或`application.properties`（它可以是任何东西），并且属性的值被正确处理。这一特点有助于通过使用类似以下内容来创建`ConfigMap`的用例。

```bash
kubectl create configmap game-config --from-file=/path/to/app-config.yaml
```

假设我们有一个名为`demo`的Spring Boot应用程序，它使用以下属性来读取其线程池配置。

- `pool.size.core
- `pool.size.maximum`。

这可以外化为`yaml`格式的config map，如下。

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  pool.size.core: 1
  pool.size.max: 16
```

单独的属性在大多数情况下都能正常工作。然而，有时，嵌入`yaml`更方便。在这种情况下，我们使用一个名为`application.yaml`的单一属性来嵌入我们的`yaml`，如下所示。

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  application.yaml: |-
    pool:
      size:
        core: 1
        max:16
```

下面的例子也可以。

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  custom-name.yaml: |-
    pool:
      size:
        core: 1
        max:16
```

你也可以根据读取`ConfigMap`时合并在一起的活动配置文件，以不同方式配置Spring Boot应用程序。你可以通过使用`application.properties`或`application.yaml`属性，为不同的profile提供不同的属性值，指定特定的profile值，每个都在自己的文档中（用`---`序列表示），如下所示。

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  application.yml: |-
    greeting:
      message: Say Hello to the World
    farewell:
      message: Say Goodbye
    ---
    spring:
      profiles: development
    greeting:
      message: Say Hello to the Developers
    farewell:
      message: Say Goodbye to the Developers
    ---
    spring:
      profiles: production
    greeting:
      message: Say Hello to the Ops
```

在前面的案例中，通过开发配置文件加载到你的Spring应用程序的配置是这样的。

```yaml
  greeting:
    message: Say Hello to the Developers
  farewell:
    message: Say Goodbye to the Developers
```

然而，如果`production`配置文件处于`active`状态，配置就会变成。

```yaml
  greeting:
    message: Say Hello to the Ops
  farewell:
    message: Say Goodbye
```

如果两个配置文件都处于活动状态，在`ConfigMap`中最后出现的属性将覆盖前面的任何值。

另一个选择是为每个配置文件创建一个不同的配置map，spring boot会根据活动的配置文件自动获取它

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  application.yml: |-
    greeting:
      message: Say Hello to the World
    farewell:
      message: Say Goodbye
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo-development
data:
  application.yml: |-
    spring:
      profiles: development
    greeting:
      message: Say Hello to the Developers
    farewell:
      message: Say Goodbye to the Developers
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo-production
data:
  application.yml: |-
    spring:
      profiles: production
    greeting:
      message: Say Hello to the Ops
    farewell:
      message: Say Goodbye
```

要告诉Spring Boot在启动时应该启用哪个配置文件，你可以通过`SPRING_PROFILES_ACTIVE`环境变量。要做到这一点，你可以用环境变量启动你的Spring Boot应用程序，你可以在容器规范的`PodSpec`中定义它。部署资源文件，如下所示。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-name
  labels:
    app: deployment-name
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment-name
  template:
    metadata:
      labels:
        app: deployment-name
    spec:
        containers:
        - name: container-name
          image: your-image
          env:
          - name: SPRING_PROFILES_ACTIVE
            value: "development"
```

> 你应该检查安全配置部分。要从pod内部访问配置maps，你需要有正确的Kubernetes服务账户、角色和角色绑定。

使用`ConfigMap`实例的另一个选择是通过运行Spring Cloud Kubernetes应用程序并让Spring Cloud Kubernetes从文件系统中读取它们，将它们挂载到Pod中。这种行为是由`spring.cloud.kubernetes.config.paths`属性控制的。你可以使用它来补充或替代前面描述的机制。你可以在`spring.cloud.kubernetes.config.paths`中使用`,`分隔符来指定多个（精确）文件路径。

> 你必须提供每个属性文件的完整准确路径，因为目录没有被递归解析。

> 如果你使用`spring.cloud.kubernetes.config.paths`或`spring.cloud.kubernetes.secrets.path`，自动重载功能将不起作用。你将需要向`/actuator/refresh`端点发出`POST`请求，或者重新启动/重新部署应用程序。

Table 1. Properties:

|Name|Type|Default|Description|
| --- | --- | --- | --- |
|`spring.cloud.kubernetes.config.enabled`|`Boolean`|`true`|启用ConfigMaps `PropertySource`。|
|`spring.cloud.kubernetes.config.name`|`String`|`${spring.application.name}`|设置 `ConfigMap` 的名称以进行查询。|
|`spring.cloud.kubernetes.config.namespace`|`String`|Client namespace|设置查询的Kubernetes命名空间|
|`spring.cloud.kubernetes.config.paths`|`List`|`null`|设置 `ConfigMap` 实例的挂载路径。|
|`spring.cloud.kubernetes.config.enableApi`|`Boolean`|`true`|启用或禁用通过API消费`ConfigMap`实例的功能|

### 5.2. Secrets PropertySource

Kubernetes有[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)的概念，用于存储敏感数据，如密码、OAuth令牌等。这个项目提供了与 "秘密 "的集成，使秘密可以被Spring Boot应用程序访问。你可以通过设置`spring.cloud.kubernetes.secrets.enabled`属性明确地启用或禁用该功能。

启用后，`Fabric8SecretsPropertySource`从以下来源查找Kubernetes的`Secrets`。

1. 递归地从secrets挂载中读取
2. 2.以应用程序命名（由`spring.application.name`定义）。
3. 匹配一些标签

**注意**

默认情况下，由于安全原因，通过API消费Secrets（上面第2和第3点）**不启用**。Secrets上的权限 "list"允许客户端检查指定命名空间中的secrets值。此外，我们建议容器通过加载的卷来共享秘密。

如果你通过API启用消耗秘密，我们建议你通过使用授权策略来限制对秘密的访问，例如RBAC。关于通过API消费Secrets时的风险和最佳做法的更多信息，请参考[本文档](https://kubernetes.io/docs/concepts/configuration/secret/#best-practices)。

如果发现了秘密，它们的数据就会被提供给应用程序。

假设我们有一个名为 `demo` 的spring boot应用程序，它使用属性来读取其数据库配置。我们可以通过使用以下命令创建一个Kubernetes secret。

```bash
kubectl create secret generic db-secret --from-literal=username=user --from-literal=password=p455w0rd
```

前面的命令将创建以下secret（你可以通过使用`kubectl get secrets db-secret -o yaml`查看）。

```yaml
apiVersion: v1
data:
  password: cDQ1NXcwcmQ=
  username: dXNlcg==
kind: Secret
metadata:
  creationTimestamp: 2017-07-04T09:15:57Z
  name: db-secret
  namespace: default
  resourceVersion: "357496"
  selfLink: /api/v1/namespaces/default/secrets/db-secret
  uid: 63c89263-6099-11e7-b3da-76d6186905a8
type: Opaque
```

注意，该数据包含由创建命令提供的字面意思的Base64编码版本。

然后，你的应用程序可以使用这个 `secret` - 例如，通过导出`secret`的值作为环境变量。

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: ${project.artifactId}
spec:
   template:
     spec:
       containers:
         - env:
            - name: DB_USERNAME
              valueFrom:
                 secretKeyRef:
                   name: db-secret
                   key: username
            - name: DB_PASSWORD
              valueFrom:
                 secretKeyRef:
                   name: db-secret
                   key: password
```

你可以通过多种方式选择要消费的`secret`。

1. 通过列出`secret`被映射的目录

    ```bash
    -Dspring.cloud.kubernetes.secrets.paths=/etc/secrets/db-secret,etc/secrets/postgresql
    ```

    如果你把所有的`secret`都映射到一个共同的根上，你可以像这样设置它们。

    ```bash
    -Dspring.cloud.kubernetes.secrets.paths=/etc/secrets
    ```

2. 通过设置一个命名的`secret`

    ```bash
    -Dspring.cloud.kubernetes.secrets.name=db-secret
    ```

3. 通过定义一个标签列表

    ```bash
    -Dspring.cloud.kubernetes.secrets.labels.broker=activemq
    -Dspring.cloud.kubernetes.secrets.labels.db=postgresql
    ```

如同 `ConfigMap` 的情况，更高级的配置也是可能的，你可以使用多个 `secret` 实例。`spring.cloud.kubernetes.secrets.sources`列表使这成为可能。例如，你可以定义以下`Secret`实例。

```yaml
spring:
  application:
    name: cloud-k8s-app
  cloud:
    kubernetes:
      secrets:
        name: default-name
        namespace: default-namespace
        sources:
         # Spring Cloud Kubernetes looks up a Secret named s1 in namespace default-namespace
         - name: s1
         # Spring Cloud Kubernetes looks up a Secret named default-name in whatever namespace n2
         - namespace: n2
         # Spring Cloud Kubernetes looks up a Secret named s3 in namespace n3
         - namespace: n3
           name: s3
```

在前面的例子中，如果没有设置`spring.cloud.kubernetes.secrets.namespace`，命名为`s1`的`Secret`将在应用程序运行的命名空间中被查找到。

Table 2. Properties:

|Name|Type|Default|Description|
| --- | --- | --- | --- |
|`spring.cloud.kubernetes.secrets.enabled`|`Boolean`|`true`|启用 Secrets `PropertySource`|
|`spring.cloud.kubernetes.secrets.name`|`String`|`${spring.application.name}`|设置要查询的`Secrets`的名称|
|`spring.cloud.kubernetes.secrets.namespace`|`String`|Client namespace|设置Kubernetes的命名空间，用于查询|
|`spring.cloud.kubernetes.secrets.labels`|`Map`|`null`|设置用于查询`Secrets`的标签|
|`spring.cloud.kubernetes.secrets.paths`|`List`|`null`|设置安装`Secrets`的路径（例子1）。|
|`spring.cloud.kubernetes.secrets.enableApi`|`Boolean`|`false`|启用或禁用通过API消费`Secrets`的功能（例子2和3）。|

**注意事项**

- `spring.cloud.kubernetes.secrets.labels`属性的行为与[Map-based binding.](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-Configuration-Binding#map-based-binding)所定义的一致。
- `spring.cloud.kubernetes.secrets.paths`属性的行为与[Collection-based binding.](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-Configuration-Binding#collection-based-binding)所定义的一样。
- 出于安全原因，通过API访问秘密可能受到限制。首选的方法是将秘密挂载到Pod上。

你可以在[spring-boot-camel-config](https://github.com/fabric8-quickstarts/spring-boot-camel-config)找到一个使用secrets的应用实例（尽管它还没有被更新到使用新的`spring-cloud-kubernetes`项目）。

### 5.3. PropertySource Reload

> 这一功能在2020.0版本中已被弃用。请参阅[Spring Cloud Kubernetes Configuration Watcher](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#spring-cloud-kubernetes-configuration-watcher)控制器，以获取实现相同功能的替代方法。

一些应用程序可能需要检测外部属性源上的变化，并更新其内部状态以反映新的配置。Spring Cloud Kubernetes的重载功能能够在相关的`ConfigMap`或`Secret`发生变化时触发应用重载。

默认情况下，这个功能是禁用的。你可以通过使用`spring.cloud.kubernetes.reload.enabled=true`配置属性来启用它（例如，在`application.properties`文件中）。

支持以下级别的重载（通过设置`spring.cloud.kubernetes.reload.strategy`属性）。

- `refresh`（默认）。只有用`@ConfigurationProperties`或`@RefreshScope`注释的配置豆被重新加载。这个重载级别利用了Spring Cloud Context的刷新功能。
- `restart_context`：整个Spring`ApplicationContext`被优雅地重新启动。豆类以新的配置重新创建。为了使重启上下文功能正常工作，你必须启用并公开重启执行器端点

```yaml
management:
  endpoint:
    restart:
      enabled: true
  endpoints:
    web:
      exposure:
        include: restart
```

- `shutdown`：Spring `ApplicationContext`被关闭以激活容器的重新启动。当你使用这个级别时，请确保所有非daemon线程的生命周期被绑定到ApplicationContext，并且复制控制器或复制集被配置为重启pod。

假设在默认设置下启用了重载功能（刷新模式），当config map发生变化时，会刷新以下bean。

```java
@Configuration
@ConfigurationProperties(prefix = "bean")
public class MyConfig {

    private String message = "a message that can be changed live";

    // getter and setters

}
```

为了看到变化的有效发生，你可以创建另一个Bean，定期打印消息，如下所示

```java
@Component
public class MyBean {

    @Autowired
    private MyConfig config;

    @Scheduled(fixedDelay = 5000)
    public void hello() {
        System.out.println("The message is: " + config.getMessage());
    }
}
```

你可以通过使用`ConfigMap`来改变应用程序打印的信息，如下所示。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: reload-example
data:
  application.properties: |-
    bean.message=Hello World!
```

与pod相关的`ConfigMap'中名为`bean.message'的属性的任何变化都会反映在输出中。更一般地说，与以`@ConfigurationProperties`注解的`prefix`字段定义的值为前缀的属性相关的变化会被检测并反映在应用程序中。本章前面解释了[将`ConfigMap`与pod相关联](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#configmap-propertysource)。

完整的例子见[`spring-cloud-kubernetes-reload-example`](https://github.com/spring-cloud/spring-cloud-kubernetes/tree/main/spring-cloud-kubernetes-examples/kubernetes-reload-example)。

重载功能支持两种操作模式:

- `event`（默认）。通过使用Kubernetes API（网络套接字）来观察配置图或秘密的变化。任何事件都会产生对配置的重新检查，如果有变化，则重新加载。为了监听配置图的变化，需要服务账户上的`view`角色。秘密需要一个更高级别的角色（如`编辑`）（默认情况下，秘密不被监控）。

- `polling`。定期从配置图和秘密中重新创建配置，看它是否有变化。你可以通过使用`spring.cloud.kubernetes.reload.period`属性来配置轮询周期，默认为15秒。它要求与被监控的属性源的角色相同。这意味着，例如，在文件挂载的秘密源上使用轮询不需要特别的权限。

Table 3. Properties:

|Name|Type|Default|Description|
| --- | --- | --- | --- |
|`spring.cloud.kubernetes.reload.enabled`|`Boolean`|`false`|启用对配置资源的监控和配置重载|
|`spring.cloud.kubernetes.reload.monitoring-config-maps`|`Boolean`|`true`|允许监控 `config maps` 的变化|
|`spring.cloud.kubernetes.reload.monitoring-secrets`|`Boolean`|`false`|允许监控 `secrets`的变化|
|`spring.cloud.kubernetes.reload.strategy`|`Enum`|`refresh`|触发重载时使用的策略 ( `refresh` , `restart_context` , or `shutdown` )|
|`spring.cloud.kubernetes.reload.mode`|`Enum`|`event`|指定如何监听属性源的变化（`event`或`polling`）。|
|`spring.cloud.kubernetes.reload.period`|`Duration`|`15s`|使用 `polling` 策略时，验证变化的周期|

**注意事项**

- 你不应该在`config maps`或`secrets`中使用`spring.cloud.kubernetes.reload`下的属性。在运行时改变这些属性可能会导致意外的结果。
- 当你使用`refresh`级别时，删除一个属性或整个配置图不会恢复bean的原始状态。

## 6. Kubernetes生态系统意识

无论你的应用程序是否在Kubernetes内运行，本指南前面描述的所有功能都同样有效。这对开发和故障排除真的很有帮助。从开发的角度来看，这可以让你启动你的Spring Boot应用，并调试这个项目中的一个模块。你不需要在Kubernetes中部署它，因为项目的代码依赖于[Fabric8 Kubernetes Java客户端](https://github.com/fabric8io/kubernetes-client)，它是一个流畅的DSL，可以通过使用`http`协议与Kubernetes服务器的REST API进行通信。

要禁用与Kubernetes的集成，你可以将`spring.cloud.kubernetes.enabled`设为`false`。请注意，当`spring-cloud-kubernetes-config`在classpath上时，`spring.cloud.kubernetes.enabled`应该设置在`bootstrap.{properties|yml}`（或特定的配置文件），否则应该在`application.{properties|yml}`（或特定的配置文件）。还要注意这些属性。`spring.cloud.kubernetes.config.enabled`和`spring.cloud.kubernetes.secrets.enabled`只有在`bootstrap.{properties|yml}`中设置才会生效。

### 6.1 Kubernetes配置文件自动配置

当应用程序在Kubernetes内以`pod`形式运行时，一个名为kubernetes的Spring配置文件会自动被激活。这让你可以定制配置，定义Spring Boot应用在Kubernetes平台上部署时应用的bean（例如，不同的开发和生产配置）。

### 6.2. Istio意识

当你在应用程序的classpath中包含`spring-cloud-kubernetes-fabric8-istio`模块时，一个新的配置文件被添加到应用程序中，前提是该应用程序在安装了[Istio](https://istio.io/)的Kubernetes集群中运行。然后你可以在你的Bean和`@Configuration`类中使用spring `@Profile("istio")`注释。

Istio意识模块使用`me.snowdrop:istio-client`与Istio API交互，让我们发现流量规则、断路器等，使我们的Spring Boot应用程序能够轻松地消费这些数据，根据环境动态地配置自己。

## 7. Pod 健康指标

Spring Boot使用[`HealthIndicator`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthEndpoint.java)来暴露应用程序的健康信息。这使得它在向用户公开健康相关信息方面非常有用，并使它很适合作为[准备就绪探针](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)使用。

Kubernetes健康指标（是核心模块的一部分）暴露了以下信息。

- Pod名称、IP地址、命名空间、服务账户、节点名称及其IP地址。
- 指示Spring Boot应用程序是在Kubernetes内部还是外部的一个标志。

## 8. 信息贡献者

Spring Cloud Kubernetes包括一个`InfoContributor`，它将Pod信息添加到Spring Boot的`/info` Acturator 端点。

你可以通过在`bootstrap.[properties | yaml]`中设置`management.info.kubernetes.enabled`为`false`来禁用这个`InfoContributor`。

## 9. Leader选举

Spring Cloud Kubernetes领导者选举机制使用Kubernetes ConfigMap实现了Spring Integration的领导者选举API。

多个应用实例竞争领导权，但领导权只授予一个。当被授予领导权时，领导者应用会收到一个带有领导权`Context`的`OnGrantedEvent`应用事件。应用程序会定期尝试获得领导权，并将领导权授予第一个调用者。一个领导者将一直是一个领导者，直到它被从集群中移除，或者它放弃了它的领导权。当领导权被移除时，前一个领导者会收到`OnRevokedEvent`应用事件。移除后，集群中的任何实例都可以成为新的领导者，包括旧的领导者。

要在你的项目中包含它，请添加以下依赖关系。

Fabric8 Leader Implementation

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-kubernetes-fabric8-leader</artifactId>
</dependency>
```

要指定用于Leader选举的configmap的名称，请使用以下property。

```properties
spring.cloud.kubernetes.leader.config-map-name=leader
```

## 10. 用于Kubernetes的负载均衡器。

这个项目包括Spring Cloud Load Balancer，用于基于Kubernetes Endpoints的负载平衡，并提供了基于Kubernetes Service的负载平衡器的实现。要将其纳入你的项目，请添加以下依赖关系。

Fabric8 Implementation

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-fabric8-loadbalancer</artifactId>
</dependency>
```

Kubernetes Java Client Implementation

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-client-loadbalancer</artifactId>
</dependency>
```

要启用基于Kubernetes服务名称的负载均衡，请使用以下属性。然后负载均衡器将尝试使用地址调用应用程序，例如`service-a.default.svc.cluster.local`。

```properties
spring.cloud.kubernetes.loadbalancer.mode=SERVICE
```

要启用所有命名空间的负载均衡，请使用以下属性。尊重`spring-cloud-kubernetes-discovery`模块的属性。

```properties
spring.cloud.kubernetes.discovery.all-namespaces=true
```

## 11. Kubernetes内部的安全配置

### 11.1. Namespace

本项目中提供的大多数组件都需要知道命名空间。对于Kubernetes（1.3以上），命名空间作为服务账户`secret`的一部分提供给pod，并由客户端自动检测。对于早期版本，它需要作为环境变量指定给pod。做到这一点的快速方法如下。

```yaml
      env:
      - name: "KUBERNETES_NAMESPACE"
        valueFrom:
          fieldRef:
            fieldPath: "metadata.namespace"
```

### 11.2. 服务账户

或支持集群内更精细的基于角色的访问的Kubernetes发行版，你需要确保使用`spring-cloud-kubernetes`运行的pod可以访问Kubernetes API。对于你分配给部署或pod的任何服务账户，你需要确保他们有正确的角色。

根据要求，你需要在以下资源上有`get`、`list`和`watch`的权限。

Table 4. Kubernetes Resource Permissions

|Dependency|Resources|
| --- | --- |
|spring-cloud-starter-kubernetes-fabric8|pods, services, endpoints|
|spring-cloud-starter-kubernetes-fabric8-config|configmaps, secrets|
|spring-cloud-starter-kubernetes-client|pods, services, endpoints|
|spring-cloud-starter-kubernetes-client-config|configmaps, secrets|

为了开发的目的，你可以给你的 `default` 服务账户增加 `cluster-reader` 的权限。在生产系统中，你可能想提供更细的权限。

下面的Role和RoleBinding是一个为`default`账户命名的权限的例子。

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: YOUR-NAME-SPACE
  name: namespace-reader
rules:
  - apiGroups: ["", "extensions", "apps"]
    resources: ["configmaps", "pods", "services", "endpoints", "secrets"]
    verbs: ["get", "list", "watch"]

---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: namespace-reader-binding
  namespace: YOUR-NAME-SPACE
subjects:
- kind: ServiceAccount
  name: default
  apiGroup: ""
roleRef:
  kind: Role
  name: namespace-reader
  apiGroup: ""
```

## 12. 服务注册的实施

在Kubernetes中，服务注册由平台控制，应用程序本身并不像其他平台那样控制注册。因此，使用`spring.cloud.service-registry.auto-registration.enabled`或设置`@EnableDiscoveryClient(autoRegister=false)`在Spring Cloud Kubernetes中没有效果。

## 13. Spring Cloud Kubernetes配置Watcher

Kubernetes提供了在应用程序的容器中[将ConfigMap或Secret作为卷装载](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)的能力。当ConfigMap或Secret的内容发生变化时，[挂载的卷就会根据这些变化进行更新](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically)。

然而，除非你重新启动应用程序，否则Spring Boot不会自动更新这些变化。Spring Cloud提供了刷新应用程序上下文的能力，无需重启应用程序，方法是点击执行器端点`/refresh`或通过使用Spring Cloud Bus发布`RefreshRemoteApplicationEvent`。

为了实现在Kubernetes上运行的Spring Cloud应用程序的配置刷新，你可以将Spring Cloud Kubernetes配置观察者控制器部署到你的Kubernetes集群。

该应用以容器形式发布，可在[Docker Hub](https://hub.docker.com/repository/docker/springcloud/spring-cloud-kubernetes-configuration-watcher)上使用。

Spring Cloud Kubernetes Configuration Watcher可以通过两种方式向应用程序发送刷新通知。

1. 通过HTTP，在这种情况下，被通知的应用程序必须有"/refresh "执行器端点暴露，并可从集群内访问。
2. 2.使用Spring Cloud Bus，在这种情况下，你将需要一个部署在你的托管中心的消息代理，以便应用程序使用。

### 13.1. Deployment YAML

下面是一个部署YAML的样本，你可以用来将Kubernetes配置观察器部署到Kubernetes。

```yaml
---
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: spring-cloud-kubernetes-configuration-watcher
      name: spring-cloud-kubernetes-configuration-watcher
    spec:
      ports:
        - name: http
          port: 8888
          targetPort: 8888
      selector:
        app: spring-cloud-kubernetes-configuration-watcher
      type: ClusterIP
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      labels:
        app: spring-cloud-kubernetes-configuration-watcher
      name: spring-cloud-kubernetes-configuration-watcher
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      labels:
        app: spring-cloud-kubernetes-configuration-watcher
      name: spring-cloud-kubernetes-configuration-watcher:view
    roleRef:
      kind: Role
      apiGroup: rbac.authorization.k8s.io
      name: namespace-reader
    subjects:
      - kind: ServiceAccount
        name: spring-cloud-kubernetes-configuration-watcher
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: namespace-reader
    rules:
      - apiGroups: ["", "extensions", "apps"]
        resources: ["configmaps", "pods", "services", "endpoints", "secrets"]
        verbs: ["get", "list", "watch"]
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: spring-cloud-kubernetes-configuration-watcher-deployment
    spec:
      selector:
        matchLabels:
          app: spring-cloud-kubernetes-configuration-watcher
      template:
        metadata:
          labels:
            app: spring-cloud-kubernetes-configuration-watcher
        spec:
          serviceAccount: spring-cloud-kubernetes-configuration-watcher
          containers:
          - name: spring-cloud-kubernetes-configuration-watcher
            image: springcloud/spring-cloud-kubernetes-configuration-watcher:2.0.1-SNAPSHOT
            imagePullPolicy: IfNotPresent
            readinessProbe:
              httpGet:
                port: 8888
                path: /actuator/health/readiness
            livenessProbe:
              httpGet:
                port: 8888
                path: /actuator/health/liveness
            ports:
            - containerPort: 8888
```

服务账户和相关的角色绑定对于Spring Cloud Kubernetes配置的正常工作非常重要。控制器需要读取Kubernetes集群中的ConfigMaps、Pod、服务、端点和Secrets数据的权限。

### 13.2. 监控 ConfigMaps 和 Secrets

Spring Cloud Kubernetes Configuration Watcher将对标签为`spring.cloud.kubernetes.config`且值为`true`的ConfigMaps或标签为`spring.cloud.kubernetes.secret`且值为`true`的任何Secret中的变化做出反应。如果ConfigMap或Secret没有这些标签，或者这些标签的值不是`true`，那么任何改变都会被忽略。

Spring Cloud Kubernetes Configuration Watcher在ConfigMaps和Secrets上寻找的标签可以分别通过设置`spring.cloud.kubernetes.config.watcher.configLabel`和`spring.cloud.kubernetes.config.watcher.secretLabel`来改变。

如果对具有有效标签的ConfigMap或Secret进行了更改，那么Spring Cloud Kubernetes Configuration Watcher将采用ConfigMap或Secret的名称，并以该名称向应用程序发送通知。

### 13.3. HTTP的实现

HTTP实现是默认使用的。当使用该实现时，Spring Cloud Kubernetes Configuration Watcher和ConfigMap或Secret发生变化，那么HTTP实现将使用Spring Cloud Kubernetes Discovery Client来获取与ConfigMap或Secret名称匹配的所有应用程序实例，并向应用程序的actuator`/refresh`端点发送HTTP POST请求。默认情况下，它将使用发现客户端中注册的端口向`/actuator/refresh`发送post请求。

#### 13.3.1. 非默认的管理端口和执行器路径

如果应用程序使用非默认的执行器路径和/或为管理端点使用不同的端口，应用程序的Kubernetes服务可以添加一个名为`boot.spring.io/actuator`的注解，并将其值设置为应用程序使用的路径和端口。例如

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: config-map-demo
  name: config-map-demo
  annotations:
    boot.spring.io/actuator: http://:9090/myactuator/home
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
    app: config-map-demo
```

你可以选择配置执行器路径和/或管理端口的另一种方式是设置`spring.cloud.kubernetes.configuration.watcher.actuatorPath`和`spring.cloud.kubernetes.configuration.watcher.actuatorPort`。

### 13.4. Messaging Implementation

当Spring Cloud Kubernetes配置观察者应用程序部署到Kubernetes时，可以通过将配置文件设置为`bus-amqp`（RabbitMQ）或`bus-kafka`（Kafka）来启用消息传递实现。

### 13.5. RabbitMQ 配置

当启用 `bus-amqp` 配置文件时，您将需要配置 Spring RabbitMQ 以将其指向您希望使用的 RabbitMQ 实例的位置，以及任何必要的验证凭证。这可以通过设置标准 Spring RabbitMQ 属性来完成，例如

```yaml
spring:
  rabbitmq:
    username: user
    password: password
    host: rabbitmq
```

### 13.6. Kafka 配置

启用`bus-kafka`配置文件后，你需要配置Spring Kafka，将其指向你想使用的Kafka Broker实例的位置。这可以通过设置标准的Spring Kafka属性来完成，例如

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: localhost:9092
```

## 14. Examples

Spring Cloud Kubernetes试图通过遵循Spring Cloud的接口，使你的应用程序在消费Kubernetes原生服务时变得透明。

在你的应用程序中，你需要在classpath中添加`spring-cloud-kubernetes-discovery`依赖项，并删除任何包含`DiscoveryClient`实现（即Eureka发现客户端）的其他依赖项。这同样适用于 `PropertySourceLocator`，你需要在classpath中添加 `spring-cloud-kubernetes-config`，并删除任何包含 `PropertySourceLocator` 实现（即配置服务器客户端）的其他依赖。

以下项目强调了这些依赖关系的用法，并演示了你如何从任何Spring Boot应用程序中使用这些库。

- [Spring Cloud Kubernetes Examples](https://github.com/sring-cloud/spring-cloud-kubernetes/tree/master/spring-cloud-kubernetes-examples)：位于这个资源库里面的。
- Spring Cloud Kubernetes完整示例。Minions和Boss
  - [Minion](https://github.com/salaboy/spring-cloud-k8s-minion)
  - [Boss](https://github.com/salaboy/spring-cloud-k8s-boss)

- Spring Cloud Kubernetes完整示例：[SpringOne Platform Tickets Service](https://github.com/salaboy/s1p_docs)
- [Spring Cloud Gateway with Spring Cloud Kubernetes Discovery and Config](https://github.com/salaboy/s1p_gateway)
- [Spring Boot Admin with Spring Cloud Kubernetes Discovery and Config](https://github.com/salaboy/showcase-admin-tool)

## 15. 其他资源

本节列出了其他资源，如关于Spring Cloud Kubernetes的演讲（slides）和视频。

- [S1P Spring Cloud on PKS](https://salaboy.com/2018/09/27/the-s1p-experience/)
- [Spring Cloud, Docker, Kubernetes → London Java Community July 2018](https://salaboy.com/2018/07/18/ljc-july-18-spring-cloud-docker-k8s/)

请随时通 pull requests 向[本仓库](https://github.com/spring-cloud/spring-cloud-kubernetes)提交其他资源。

## 16. 配置属性

要查看所有与Kubernetes相关的配置属性列表，请查看[附录页面](https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/appendix.html)。
