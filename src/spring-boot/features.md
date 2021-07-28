# SringBoot特性

本节深入介绍了Spring Boot的细节。在这里你可以了解到你可能想要使用和定制的关键功能。如果你还没有这样做，你可能需要阅读[开始使用](/spring-boot/getting-started.html) 和 [使用Spring Boot](/spring-boot/using.html) 部分，以便你对基础知识有一个良好的了解。

## 1. SpringApplication

`SpringApplication`类提供了一种方便的方式来引导一个从`main()`方法启动的Spring应用程序。在许多情况下，你可以委托给静态的`SpringApplication.run`方法，如以下例子所示。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

当你的应用程序启动时，你应该看到类似于以下的输出。

```text
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.5.3

2021-02-03 10:33:25.224  INFO 17321 --- [           main] o.s.b.d.s.s.SpringAppplicationExample    : Starting SpringAppplicationExample using Java 1.8.0_232 on mycomputer with PID 17321 (/apps/myjar.jar started by pwebb)
2021-02-03 10:33:25.226  INFO 17900 --- [           main] o.s.b.d.s.s.SpringAppplicationExample    : No active profile set, falling back to default profiles: default
2021-02-03 10:33:26.046  INFO 17321 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-02-03 10:33:26.054  INFO 17900 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-02-03 10:33:26.055  INFO 17900 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.41]
2021-02-03 10:33:26.097  INFO 17900 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-02-03 10:33:26.097  INFO 17900 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 821 ms
2021-02-03 10:33:26.144  INFO 17900 --- [           main] s.tomcat.SampleTomcatApplication         : ServletContext initialized
2021-02-03 10:33:26.376  INFO 17900 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-02-03 10:33:26.384  INFO 17900 --- [           main] o.s.b.d.s.s.SpringAppplicationExample    : Started SampleTomcatApplication in 1.514 seconds (JVM running for 1.823)
```

默认情况下，显示`INFO`日志信息，包括一些相关的启动细节，例如启动应用程序的用户。如果你需要一个除`INFO`以外的日志级别，你可以设置它，如[日志级别](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging.log-levels)中所述。应用程序的版本是通过主应用程序类的包的实现版本来确定的。启动信息的记录可以通过设置`spring.main.log-startup-info`为`false`来关闭。这也将关闭应用程序的活动配置文件的日志记录。

> 为了在启动过程中增加额外的日志记录，你可以在`SpringApplication`的子类中覆盖`logStartupInfo(boolean)`。

### 1.1. 启动失败

如果你的应用程序启动失败，注册的`FailureAnalyzers`有机会提供专门的错误信息和具体的行动来解决这个问题。例如，如果你在端口`8080`上启动一个网络应用程序，而该端口已经被使用，你应该看到类似于下面的信息。

```text
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

> Spring Boot提供了许多 "FailureAnalyzers" 的实现，你也可以[添加你自己的](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.application.failure-analyzer)。

如果没有故障分析器能够处理异常，你仍然可以显示完整的条件报告，以更好地了解出错的原因。为此，你需要为`org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` [启用debug属性](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)或[启用DEBUG日志记录](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging.log-levels)。

例如，如果你通过使用`java -jar`运行你的应用程序，你可以启用`debug`属性，如下所示。

```bash
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 1.2. 延迟初始化

`SpringApplication`允许应用程序被延迟初始化。当启用延迟初始化时，Bean在需要时被创建，而不是在应用程序启动时。因此，启用延迟初始化可以减少应用程序的启动时间。在一个Web应用程序中，启用延迟初始化将导致许多与Web相关的Bean在收到HTTP请求之前不会被初始化。

延迟初始化的一个缺点是，它可以延迟发现应用程序的问题。如果一个配置错误的Bean被初始化，在启动过程中就不会再出现故障，问题只有在Bean被初始化时才会显现。还必须注意确保JVM有足够的内存来容纳应用程序的所有Bean，而不仅仅是那些在启动时被初始化的Bean。由于这些原因，默认情况下不启用延迟初始化，建议在启用延迟初始化之前，对JVM的堆大小进行微调。

可以使用`SpringApplicationBuilder`上的`lazyInitialization`方法或`SpringApplication`上的`setLazyInitialization`方法以编程方式启用延迟初始化。另外，也可以使用`spring.main.lazy-initialization`属性来启用，如下例所示。

```yaml
spring:
  main:
    lazy-initialization: true
```

如果你想禁用某些Bean的延迟初始化，同时对应用程序的其他部分使用延迟初始化，你可以使用`@Lazy(false)`注解将它们的懒惰属性明确地设置为false。

### 1.3. 自定义Banner

启动时打印的Banner可以通过在classpath中添加`banner.txt`文件或将`spring.banner.location`属性设置为此类文件的位置来改变。如果该文件的编码不是UTF-8，你可以设置`spring.banner.charset`。除了文本文件，你还可以在classpath中添加`banner.gif`, `banner.jpg`, 或`banner.png`图像文件，或者设置`spring.banner.image.location`属性。图像被转换为ASCII艺术表现，并打印在任何文本横幅之上。

在你的`banner.txt`文件中，你可以使用以下任何一个占位符。

表 1. Banner 中可以使用的变量

|Variable|Description|
| --- | --- |
|`${application.version}`|你的应用程序的版本号，正如在`MANIFEST.MF`中声明的那样。例如，`Implementation-Version: 1.0`被打印为`1.0`。|
|`${application.formatted-version}`|你的应用程序的版本号，如在`MANIFEST.MF`中声明的那样，并以格式化的方式显示（用括号包围并以`v`为前缀）。例如 `(v1.0)` 。|
|`${spring-boot.version}`|你正在使用的Spring Boot版本。例如`2.5.3`。|
|`${spring-boot.formatted-version}`|您正在使用的Spring Boot版本，并以格式化显示（用括号包围，前缀为`v`）。例如，`(v2.5.3)` 。|
|`${Ansi.NAME}` (or `${AnsiColor.NAME}` , `${AnsiBackground.NAME}` , `${AnsiStyle.NAME}` )|其中`NAME`是一个ANSI转义代码的名称。详见[`AnsiPropertySource`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。|
|`${application.title}`|你的应用程序的标题，正如在`MANIFEST.MF`中声明的那样。例如，`Implementation-Title: MyApp`被打印成 `MyApp`。|

> 如果你想以编程方式生成一个Banner，可以使用`SpringApplication.setBanner(..)`方法。使用`org.springframework.boot.Banner`接口并实现你自己的`printBanner()`方法。

你也可以使用`spring.main.banner-mode`属性来决定是否将banner打印到`System.out`（`console`），发送到配置的logger（`log`），或根本不产生（`off`）。

打印的Banner被注册为一个单例Bean，名字是：`springBootBanner`。

> `${application.version}`和`${application.formatted-version}`属性只有在使用Spring Boot启动器时才可用。如果你运行一个未打包的jar并使用`java -cp <classpath> <mainclass>`启动它，这些值将不会被解析。
>
> 这就是为什么我们建议你总是使用`java org.springframework.boot.loader.JarLauncher`来启动未打包的jar。这将在构建classpath和启动你的应用程序之前初始化`application.*`的Banner变量。

### 1.4. 定制SpringApplication

如果 `SpringApplication` 的默认值不符合你的口味，你可以创建一个本地实例并对其进行自定义。例如，要关闭Banner，你可以这样写。

```java
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
    }

}
```

> 传递给`SpringApplication`的构造参数是Spring Bean的配置源。在大多数情况下，这些是对`@Configuration`类的引用，但它们也可能是对`@Component`类的直接引用。

也可以通过使用 `application.properties` 文件来配置`SpringApplication`。参见[外部化配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)以了解详情。

关于配置选项的完整列表，请参见[SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/SpringApplication.html)。

### 1.5. Fluent Builder API

如果你需要建立一个`ApplicationContext`层次结构（具有父/子关系的多个Context），或者你喜欢使用一个 `Fluent` 构建器API，你可以使用`SpringApplicationBuilder`。

`SpringApplicationBuilder`允许你将多个方法调用串联起来，并包括`parent`和`child`方法，让你创建一个层次结构，如以下例子所示。

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);


```

> 在创建 `ApplicationContext` 层次结构时，有一些限制。例如，Web组件**必须**包含在子context，并且父Context和子Context使用相同的`Environment`。请参阅[SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/builder/SpringApplicationBuilder.html)以了解全部细节。

### 1.6. 应用程序的可用性

在平台上部署时，应用程序可以使用[Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)等基础设施向平台提供有关其可用性的信息。Spring Boot包括对常用的 "liveness"和 "readiness"可用性状态的开箱即用支持。如果你使用Spring Boot的`actuator`支持，那么这些状态将作为健康端点组暴露出来。

此外，你也可以通过将 `ApplicationAvailability` 接口注入到你自己的Bean中来获得可用性状态。

#### 1.6.1. Liveness State

一个应用程序的 "Liveness"状态告诉我们它的内部状态是否允许它正常工作，或者在当前失败的情况下自行恢复。一个破碎的 "Liveness"状态意味着应用程序处于无法恢复的状态，基础设施应该重新启动该应用程序。

> 一般来说，"Liveness" 状态不应该基于外部检查，比如[健康检查](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health)。如果是这样，一个失败的外部系统（数据库、Web API、外部缓存）将引发大规模的重启和整个平台的级联故障。

Spring Boot应用程序的内部状态大多由Spring `ApplicationContext`表示。如果应用程序Context已经成功启动，Spring Boot就认为应用程序处于有效状态。一旦Context被刷新，应用程序就被认为是活的，见[Spring Boot应用程序生命周期和相关应用程序事件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-events-and-listeners)。

#### 1.6.2. Readiness State

一个应用程序的 "Readiness"状态告诉人们该应用程序是否准备好处理流量。一个失败的 "Readiness"状态告诉平台，它暂时不应该把流量发送到该应用程序。这通常发生在启动过程中，当`CommandLineRunner`和`ApplicationRunner`组件被处理时，或者在任何时候，如果应用程序决定它太忙了，无法处理额外的流量。

一旦应用程序和命令行运行器被调用，就认为应用程序已经准备好了，见[Spring Boot应用程序生命周期和相关的应用程序事件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-events-and-listeners)。

#### 1.6.3. 管理应用程序的可用性状态

应用组件可以在任何时候通过注入`ApplicationAvailability`接口并调用其上的方法来检索当前的可用性状态。更多时候，应用程序会想要监听状态更新或更新应用程序的状态。

例如，我们可以把应用程序的 "Readiness"状态导出到一个文件，这样Kubernetes的 "exec Probe"就可以查看这个文件了。

```java
import org.springframework.boot.availability.AvailabilityChangeEvent;
import org.springframework.boot.availability.ReadinessState;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class MyReadinessStateExporter {

    @EventListener
    public void onStateChange(AvailabilityChangeEvent<ReadinessState> event) {
        switch (event.getState()) {
        case ACCEPTING_TRAFFIC:
            // create file /tmp/healthy
            break;
        case REFUSING_TRAFFIC:
            // remove file /tmp/healthy
            break;
        }
    }

}
```

当应用程序中断而无法恢复时，我们还可以更新应用程序的状态。

```java
import org.springframework.boot.availability.AvailabilityChangeEvent;
import org.springframework.boot.availability.LivenessState;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Component;

@Component
public class MyLocalCacheVerifier {

    private final ApplicationEventPublisher eventPublisher;

    public MyLocalCacheVerifier(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void checkLocalCache() {
        try {
            // ...
        }
        catch (CacheCompletelyBrokenException ex) {
            AvailabilityChangeEvent.publish(this.eventPublisher, ex, LivenessState.BROKEN);
        }
    }

}
```

Spring Boot提供了[Kubernetes HTTP探测 "Liveness"和 "Readiness"与Actuator Health Endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.kubernetes-probes)。你可以得到更多关于[在Kubernetes上部署Spring Boot应用程序的专门章节](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.cloud.kubernetes)的指导。

### 1.7. Application Events and Listeners

除了常见的Spring框架事件，如[`ContextRefreshedEvent`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，一个`SpringApplication`会发送一些额外的应用事件。

有些事件实际上是在`ApplicationContext`被创建之前被触发的，所以你不能以`@Bean`的形式注册一个监听器。你可以通过`SpringApplication.addListeners(...)`方法或`SpringApplicationBuilder.listeners(...)`方法注册它们。

如果你希望这些监听器被自动注册，无论应用程序是以何种方式创建的，你可以在项目中添加一个`META-INF/spring.plants`文件，并通过使用`org.springframework.context.ApplicationListener`键来引用你的监听器，如以下例子所示。

```properties
org.springframework.context.ApplicationListener=com.example.project.MyListener
```

当你的应用程序运行时，应用程序事件按以下顺序发送：

1. `ApplicationStartingEvent`在运行开始时被发送，但在任何处理之前，除了监听器和初始化器的注册之外。
2. 当已知将在Context中使用的环境，但在创建Context之前，将发送一个 `ApplicationEnvironmentPreparedEvent`。
3. 当 `ApplicationContext` 被准备好并且 `ApplicationContextInitializers` 被调用，但在任何 bean 定义被加载之前，`ApplicationContextInitializedEvent` 被发送。
4. `ApplicationPreparedEvent`在刷新开始前但在Bean定义被加载后被发送。
5. `ApplicationStartedEvent`在Context被刷新之后，但在任何应用程序和命令行运行程序被调用之前被发送。
6. 紧接着发送一个`AvailabilityChangeEvent`，并注明`LivenessState.CORRECT`，以表明应用程序被认为是有效的。
7. 在任何[application and command-line runners](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.command-line-runner)被调用后，将发送一个 `ApplicationReadyEvent`。
8. 紧接着发送一个带有`ReadinessState.ACCEPTING_TRAFFIC`的`AvailabilityChangeEvent`，以表明应用程序已经准备好为请求提供服务。
9. 如果在启动时出现异常，将发送一个`ApplicationFailedEvent`。

以上列表仅包括与 `SpringApplication` 相关的 `SpringApplicationEvent`。除此以外，以下事件也会在`ApplicationPreparedEvent`之后和`ApplicationStartedEvent`之前发布。

* `WebServerInitializedEvent`是在`WebServer`准备好后发送的。`ServletWebServerInitializedEvent`和`ReactiveWebServerInitializedEvent`分别是Servlet和Reactive的变体。
* `ContextRefreshedEvent`在`ApplicationContext`被刷新时发送。

你通常不需要使用应用程序事件，但知道它们的存在会很方便。在内部，Spring Boot使用事件来处理各种任务。

> Event listeners不应该运行潜在的冗长任务，因为它们默认是在同一个线程中执行。考虑使用[application and command-line runners](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.command-line-runner)来代替。

Application event是通过使用Spring框架的事件发布机制来发送的。该机制的一部分确保了发布给子Context中的listener的事件也会发布给任何祖先Context中的listener。因此，如果你的应用程序使用`SpringApplication`实例的层次结构，一个监听器可能会收到同一类型Application event的多个实例。

为了让你的监听器区分其Context的事件和后代Context的事件，它应该请求其应用程序Context被注入，然后将注入的Context与事件的Context进行比较。Context可以通过实现`ApplicationContextAware`来注入，或者，如果listener是一个Bean，可以通过使用`@Autowired`。

### 1.8. Web 环境

`SpringApplication`试图帮你你创建正确类型的`ApplicationContext`。用于确定`WebApplicationType`的算法如下。

* 如果Spring MVC存在，就会使用`AnnotationConfigServletWebServerApplicationContext`。
* 如果Spring MVC不存在而Spring WebFlux存在，则使用 `AnnotationConfigReactiveWebServerApplicationContext`。
* 否则，将使用 `AnnotationConfigApplicationContext`。

这意味着，如果你在同一个应用程序中使用Spring MVC和Spring WebFlux的新`WebClient`，将默认使用Spring MVC。你可以通过调用`setWebApplicationType(WebApplicationType)`轻松覆盖。

也可以通过调用`setApplicationContextClass(..)`来完全控制使用的`ApplicationContext`类型。

> 当在JUnit测试中使用`SpringApplication`时，通常需要调用`setWebApplicationType(WebApplicationType.NONE)`。

### 1.9. 访问 Application 参数

如果你需要访问传递给`SpringApplication.run(..)`的应用程序参数，你可以注入一个`org.springframework.boot.ApplicationArguments`bean。`ApplicationArguments`接口提供了对原始`String[]`参数以及经过解析的`option`和`non-option`参数的访问，如以下例子所示。

```java
import java.util.List;

import org.springframework.boot.ApplicationArguments;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        if (debug) {
            System.out.println(files);
        }
        // if run with "--debug logfile.txt" prints ["logfile.txt"]
    }

}
```

Spring Boot还在Spring的`Environment` 中注册了一个`CommandLinePropertySource`。这让你也可以通过使用`@Value`注解来注入单个应用参数。

### 1.10. 使用 ApplicationRunner 或者 CommandLineRunner

如果你需要在`SpringApplication`启动后运行一些特定的代码，你可以实现`ApplicationRunner`或`CommandLineRunner`接口。这两个接口以相同的方式工作，并提供一个单一的`run`方法，该方法在`SpringApplication.run(...)`完成之前被调用。

> 这个约定很适合那些应该在应用程序启动后但在其开始接受访问之前运行的任务。

`CommandLineRunner`接口提供对应用程序参数的访问，作为一个字符串数组，而`ApplicationRunner`使用前面讨论的`ApplicationArguments`接口。下面的例子显示了一个带有`run`方法的`CommandLineRunner`。

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // Do something...
    }

}
```

如果定义了几个 `CommandLineRunner` 或 `ApplicationRunner` Bean，它们必须以特定的顺序被调用，你可以额外实现`org.springframework.core.Ordered`接口或使用`org.springframework.core.annotation.Order`注解。

### 1.11. 退出程序

每个 `SpringApplication` 都向JVM注册了一个shutdown hook，以确保 `ApplicationContext` 在退出时优雅地关闭。所有标准的Spring生命周期回调（如`DisposableBean`接口或`@PreDestroy`注解）都可以使用。

此外，如果Bean希望在调用`SpringApplication.exit()`时返回特定的退出代码，可以实现`org.springframework.boot.ExitCodeGenerator`接口。然后，这个退出代码可以被传递给`System.exit()`，将其作为状态代码返回，如下面的例子所示。

```java
import org.springframework.boot.ExitCodeGenerator;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class MyApplication {

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> 42;
    }

    public static void main(String[] args) {
        System.exit(SpringApplication.exit(SpringApplication.run(MyApplication.class, args)));
    }

}
```

另外，`ExitCodeGenerator`接口可以由异常实现。当遇到这种异常时，Spring Boot会返回由实现的`getExitCode()`方法提供的退出代码。

### 1.12. 管理功能

可以通过指定`spring.application.admin.enabled`属性来启用应用程序的管理相关功能。这暴露了平台上的[`SpringApplicationAdminMXBean`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java) `MBeanServer`。你可以使用这个功能来远程管理你的Spring Boot应用程序。这个功能对任何服务封装器的实现也很有用。

> 如果你想知道应用程序是在哪个HTTP端口上运行的，可以读取KEY为`local.server.port`的属性值。

### 1.13. 应用程序启动跟踪

在应用启动期间，`SpringApplication`和`ApplicationContext`执行许多与应用生命周期、Bean生命周期甚至处理应用事件有关的任务。通过[`ApplicationStartup`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/core/metrics/ApplicationStartup.html)，Spring框架[允许你用StartupStep 对象跟踪应用程序的启动顺序](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/core.html#context-functionality-startup)。这些数据可以为分析目的而收集，或者只是为了更好地了解应用程序的启动过程。

你可以在设置 `SpringApplication` 实例时选择一个 `ApplicationStartup` 实现。例如，为了使用`BufferingApplicationStartup`，你可以写。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.metrics.buffering.BufferingApplicationStartup;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setApplicationStartup(new BufferingApplicationStartup(2048));
        application.run(args);
    }

}
```

第一个可用的实现，`FlightRecorderApplicationStartup`是由Spring框架提供的。它将Spring特有的启动事件添加到Java Flight Recorder会话中，旨在对应用程序进行分析，并将其Spring上下文生命周期与JVM事件（如分配、GC、类加载......）联系起来。一旦配置好，你就可以通过启用Flight Recorder运行应用程序来记录数据。

```shell
$ java -XX:StartFlightRecording:filename=recording.jfr,duration=10s -jar demo.jar
```

Spring Boot提供了 `BufferingApplicationStartup` 变体；这个实现是为了缓冲启动步骤，并将其排入外部度量系统。应用程序可以在任何组件中要求获得`BufferingApplicationStartup`类型的bean。

Spring Boot也可以被配置为公开一个[startup端点](https://docs.spring.io/spring-boot/docs/2.5.3/actuator-api/htmlsingle/#startup)，以JSON文档的形式提供这一信息。

## 2. 外部化配置

TODO

{{#include ../license.md}}
