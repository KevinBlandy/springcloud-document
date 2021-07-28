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

Spring Boot让你将配置外部化，这样你就可以在不同的环境中使用相同的应用程序代码。你可以使用各种外部配置源，包括Java properties文件、YAML文件、环境变量和命令行参数。

属性值可以通过使用`@Value`注解直接注入你的Bean，通过Spring的`Environment`抽象访问，或者通过`@ConfigurationProperties`[绑定到结构化对象](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties)。

Spring Boot使用一个非常特殊的`PropertySource`顺序，旨在允许合理地覆盖值。属性是按以下顺序考虑的（较低项目的值会覆盖较早项目的值）。

1. 默认属性（通过设置`SpringApplication.setDefaultProperties`指定）。
2. `@Configuration`类上的[@PropertySource](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/annotation/PropertySource.html)注解。请注意，这样的属性源直到application context被刷新时才会被添加到环境中。这对于配置某些属性来说已经太晚了，比如`logging.*`和`spring.main.*`，它们在刷新开始前就已经被读取了。
3. 配置数据（如`application.properties`文件）。
4. 一个`RandomValuePropertySource`，它的属性只有`random.*`。
5. 操作系统环境变量。
6. Java系统属性（`System.getProperties()`）。
7. 来自`java:comp/env`的JNDI属性。
8. `ServletContext`初始参数。
9. `ServletConfig`初始参数。
10. 来自`SPRING_APPLICATION_JSON`的属性（嵌入环境变量或系统属性中的内联JSON）。
11. 命令行参数
12. `properties`属性在你的`tests`上。在[`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/test/context/SpringBootTest.html)和[用于测试你的应用程序的特定片断的测试注释](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-tests)上可用。
13. [`@TestPropertySource`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/test/context/TestPropertySource.html)对你的测试进行注解。
14. 当devtools处于活动状态时，在`$HOME/.config/spring-boot`目录下的[Devtools全局设置属性](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.globalsettings)。

配置数据文件按以下顺序考虑。

1. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files)打包在你的jar里面(`application.properties`和YAML变体)。
2. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)打包在你的jar中（`application-{profile}.properties`和YAML变量）。
3. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files) 在你打包的jar之外(`application.properties`和YAML变体)。
4. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)，在你打包的jar之外（`application-{profile}.properties`和YAML变体）。

> 建议你在整个应用程序中坚持使用一种格式。如果你在同一地点有`.properties`和`.yml`格式的配置文件，`.properties`优先。

为了提供一个具体的例子，假设你开发了一个`@Component`，使用了`name`属性，如下例所示。

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在你的应用程序的classpath（例如，在你的jar中），你可以有一个`application.properties`文件，为`name`提供一个合理的默认属性值。当在一个新的环境中运行时，可以在jar之外提供一个`application.properties`文件，覆盖`name`。对于一次性的测试，你可以用一个特定的命令行开关来启动（例如，`java -jar app.jar --name="Spring"`）。

> `env`和`configprops`端点在确定一个属性为什么有一个特定的值时很有用。你可以使用这两个端点来诊断意外的属性值。详见"[Production ready features](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints) "部分。

### 2.1.  访问命令行属性

默认情况下，`SpringApplication`会将任何命令行选项参数（即以`--`开头的参数，如`--server.port=9000`）转换为`property`并将其添加到Spring`Environment`中。如前所述，命令行属性总是优先于基于文件的属性源。

如果你不希望命令行属性被添加到 `Environment` 中，你可以通过使用`SpringApplication.setAddCommandLineProperties(false)`来禁用它们。

### 2.2. JSON Application Properties

环境变量和系统属性往往有限制，这意味着有些属性名称不能使用。为了帮助解决这个问题，Spring Boot允许你将一个属性块编码为一个单一的JSON结构。

当你的应用程序启动时，任何`spring.application.json`或`SPRING_APPLICATION_JSON`属性将被解析并添加到`Environment`中。

例如，`SPRING_APPLICATION_JSON`属性可以在UN*X shell的命令行中作为环境变量提供。

```shell
$ SPRING_APPLICATION_JSON='{"my":{"name":"test"}}' java -jar myapp.jar
```

在前面的例子中，你在Spring的`Environment`中最终得到了`my.name=test`。

同样的JSON也可以作为一个系统属性提供。

```shell
$ java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```

或者你可以通过使用一个命令行参数来提供JSON。

```shell
$ java -jar myapp.jar --spring.application.json='{"my":{"name":"test"}}'
```

如果你要部署到一个经典的应用服务器，你也可以使用一个名为`java:comp/env/spring.application.json`的JNDI变量。

> 尽管JSON中的 `null` 值将被添加到生成的属性源中，但`PropertySourcesPropertyResolver`将 "`null` 属性视为缺失值。这意味着JSON不能用 `null` 值覆盖来自低阶属性源的属性。

### 2.3. 外部应用属性

当你的应用程序启动时，Spring Boot会自动从以下位置找到并加载`application.properties`和`application.yaml`文件。

1. classpath
    1. classpath的根路径
    2. classpath `/config`包
2. 当前目录
    1. 当前目录根路径
    2. 当前目录下的`/config`子目录
    3. `/config`子目录的直接子目录。

列表按优先级排序（较低项目的值覆盖较早项目的值）。加载的文件被作为 `PropertySources` 添加到Spring的 `Environment` 中。

如果你不喜欢`application`作为配置文件的名称，你可以通过指定`spring.config.name`环境属性来切换到另一个文件名。例如，为了寻找`myproject.properties`和`myproject.yaml`文件，你可以按以下方式运行你的应用程序。

```shell
$ java -jar myproject.jar --spring.config.name=myproject
```

你也可以通过使用`spring.config.location`环境属性来引用一个明确的位置。该属性接受一个逗号分隔的列表，其中包含一个或多个要检查的位置。

下面的例子显示了如何指定两个不同的文件。

```shell
$ java -jar myproject.jar --spring.config.location=\
    optional:classpath:/default.properties,\
    optional:classpath:/override.properties
```

> 如果[位置是可选的](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.optional-prefix)并且你不介意它们不存在，请使用前缀`optional:`。

`spring.config.name` , `spring.config.location` , 和`spring.config.extra-location`很早就用来确定哪些文件必须被加载。它们必须被定义为环境属性（通常是操作系统环境变量，系统属性，或命令行参数）。

如果`spring.config.location`包含目录（而不是文件），它们应该以`/`结尾。在运行时，它们将被附加上由`spring.config.name`生成的名称，然后被加载。在`spring.config.location`中指定的文件被直接导入。

> 目录和文件位置值也都被扩展，以检查[配置文件特定文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)。例如，如果你的`spring.config.location`是`classpath:myconfig.properties`，你也会发现适当的`classpath:myconfig-<profile>.properties`文件被加载。

在大多数情况下，你添加的每个`spring.config.location`项目将引用一个文件或目录。位置是按照定义的顺序来处理的，后面的位置可以覆盖前面的位置的值。

如果你有一个复杂的位置设置，并且你使用特定的配置文件，你可能需要提供进一步的提示，以便Spring Boot知道它们应该如何分组。一个位置组是一个位置的集合，这些位置都被认为是在同一级别。例如，你可能想把所有classpath位置分组，然后是所有外部位置。一个位置组内的项目应该用`;`分开。更多细节见 [配置文件特定文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific) 一节中的例子。

通过使用`spring.config.location`配置的位置将取代默认位置。例如，如果`spring.config.location`的配置值为`optional:classpath:/custom-config/,optional:file:./custom-config/`，则完整位置集为。

1. `optional:classpath:custom-config/`
2. `optional:file:./custom-config/`

如果你喜欢添加额外的位置，而不是替换它们，你可以使用`spring.config.extra-location`。从附加位置加载的属性可以覆盖默认位置的属性。例如，如果`spring.config.extra-location`的配置值为`optional:classpath:/custom-config/,optional:file:./custom-config/`，那么完整位置集是。

1. `optional:classpath:/;optional:classpath:/config/`
2. `optional:file:./;optional:file:./config/;optional:file:./config/*/`
3. `optional:classpath:custom-config/`
4. `optional:file:./custom-config/`

这种搜索排序让你在一个配置文件中指定默认值，然后在另一个文件中选择性地覆盖这些值。你可以在`application.properties`（或你用`spring.config.name`选择的任何其他basename）中为你的应用程序提供默认值，在默认位置之一。然后，这些默认值可以在运行时被位于其中一个自定义位置的不同文件所覆盖。

如果你使用环境变量而不是系统属性，大多数操作系统不允许使用句点分隔的键名，但你可以使用下划线代替（例如，`SPRING_CONFIG_NAME`代替`spring.config.name`）。详见[从环境变量绑定](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)。

如果你的应用程序在servlet容器或应用服务器中运行，那么JNDI属性（在`java:comp/env`中）或servlet上下文初始化参数可以代替环境变量或系统属性，或者与之一样。

#### 2.3.1. 可选路径

默认情况下，当指定的配置数据位置不存在时，Spring Boot将抛出一个`ConfigDataLocationNotFoundException`，你的应用程序将无法启动。

如果你想指定一个位置，但你不介意它并不总是存在，你可以使用`optional:`前缀。你可以在`spring.config.location`和`spring.config.extra-location`属性中使用这个前缀，也可以在[`spring.config.import`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.importing)声明中使用。

例如，`spring.config.import`值为`optional:file:./myconfig.properties`允许你的应用程序启动，即使`myconfig.properties`文件丢失。

如果你想忽略所有的`ConfigDataLocationNotFoundExceptions`并总是继续启动你的应用程序，你可以使用`spring.config.onnot-found`属性。使用`SpringApplication.setDefaultProperties(..)`或使用系统/环境变量将该值设置为`ignore`。

#### 2.3.2. 通配符路径

如果一个配置文件的位置在最后一个路径段中包括`*`字符，它就被认为是一个通配符位置。通配符在配置被加载时被扩展，因此，直接的子目录也被检查。通配符位置在Kubernetes这样的环境中特别有用，因为有多个来源的配置属性。

例如，如果你有一些Redis配置和一些MySQL配置，你可能想把这两块配置分开，同时要求这两块都存在于一个`application.properties`文件中。这可能会导致两个独立的`application.properties`文件挂载在不同的位置，如`/config/redis/application.properties`和`/config/mysql/application.properties`。在这种情况下，有一个`config/*/`的通配符位置，将导致两个文件都被处理。

默认情况下，Spring Boot将`config/*/`列入默认搜索位置。这意味着你的jar之外的`/config`目录的所有子目录都会被搜索到。

你可以通过`spring.config.location`和`spring.config.extra-location`属性自己使用通配符位置。

通配符位置必须只包含一个 `*`，并以 `*/`结尾，用于搜索属于目录的位置，或 `*/<filename>`用于搜索属于文件的位置。带有通配符的位置将根据文件名的绝对路径按字母顺序排序。

> 通配符位置只对外部目录起作用。你不能在`classpath:`位置中使用通配符。

#### 2.3.3. Profile Specific Files

除了`application`属性文件，Spring Boot还将尝试使用命名惯例`application-{profile}`加载profile特定文件。例如，如果你的应用程序激活了名为`prod`的配置文件并使用YAML文件，那么`application.yml`和`application-prod.yml`都将被考虑。

特定于配置文件的属性与标准的`application.properties`的位置相同，特定于配置文件的文件总是优先于非特定文件。如果指定了几个配置文件，则采用最后胜出的策略。例如，如果配置文件`prod,live`是由`spring.profiles.active`属性指定的，`application-prod.properties`中的值可以被`application-live.properties`中的值覆盖。

最后获胜的策略适用于[location group](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.location-groups)级别。`spring.config.location`为`classpath:/cfg/,classpath:/ext/`将不会有与`classpath:/cfg/;classpath:/ext/`一样的覆盖规则。

例如，继续我们上面的 `prod,live` 例子，我们可能有以下文件。

```text
/cfg
  application-live.properties
/ext
  application-live.properties
  application-prod.properties
```

当我们的`spring.config.location`为`classpath:/cfg/,classpath:/ext/`时，我们会在所有`/ext`文件之前处理所有`cfg`文件。

1. `/cfg/application-live.properties`。
2. `/ext/application-prod.properties`。
3. `/ext/application-live.properties`。

当我们用`classpath:/cfg/;classpath:/ext/`代替时（用`;`分隔符），我们在同一级别处理`/cfg`和`/ext`。

1. `/ext/application-prod.properties`。
2. `/cfg/application-live.properties`。
3. `/ext/application-live.properties`。

`Environment`有一组默认的配置文件（默认为[default]），如果没有设置活动的配置文件，就会使用这些配置文件。换句话说，如果没有明确激活的配置文件，那么就会考虑来自`application-default`的属性。

> 属性文件只被加载一次。如果你已经直接[导入](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.importing)一个配置文件的特定属性文件，那么它将不会被第二次导入。

#### 2.3.4. 导入额外的数据

应用程序属性可以使用`spring.config.import`属性从其他位置导入进一步的配置数据。导入在被发现时被处理，并被视为紧接着声明导入的文件下面插入的额外文件。

例如，你在classpath的`application.properties`文件中可能有以下内容。

```properties
spring.application.name=myapp
spring.config.import=optional:file:./dev.properties
```

这将触发导入当前目录下的 `dev.properties` 文件（如果存在这样的文件）。导入的`dev.properties`中的值将优先于触发导入的文件。在上面的例子中，`dev.properties`可以将`spring.application.name`重新定义为一个不同的值。

一个导入只能被导入一次，无论它被声明多少次。一个导入在properties/yaml文件中的单个文件中被定义的顺序并不重要。例如，下面的两个例子产生相同的结果。

```properties
spring.config.import=my.properties
my.property=value
```

```properties
my.property=value
spring.config.import=my.properties
```

在上述两个例子中，`my.properties`文件的值将优先于触发其导入的文件。

在一个单一的`spring.config.import`键下可以指定多个位置。位置将按照它们被定义的顺序进行处理，后来的导入将被优先考虑。

在适当的时候，[Profile-specific variants](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific) 也被考虑导入。上面的例子将导入 `my.properties` 以及任何 `my-<profile>.properties` 变体。

Spring Boot包括可插入的API，允许支持各种不同的位置地址。默认情况下，你可以导入Java属性、YAML和 [configuration trees](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.configtree)。

第三方的jars可以提供对其他技术的支持（不要求文件是本地的）。例如，你可以想象配置数据来自外部存储，如Consul、Apache ZooKeeper或Netflix Archaius。

如果你想支持你自己的位置，请参阅`org.springframework.boot.context.config`包中的`ConfigDataLocationResolver`和`ConfigDataLoader`类。

#### 2.3.5. 导入无扩展名的文件

有些云平台不能为卷装文件添加文件扩展名。要导入这些无扩展名的文件，你需要给Spring Boot一个提示，以便它知道如何加载它们。你可以通过在方括号里放一个扩展名提示来做到这一点。

例如，假设你有一个`/etc/config/myconfig`文件，你希望以yaml形式导入。你可以用下面的方法从你的`application.properties`中导入它。

```properties
spring.config.import=file:/etc/config/myconfig[.yaml]
```

#### 2.3.6. 使用Configuration Trees

当在云平台（如Kubernetes）上运行应用程序时，你经常需要读取平台提供的配置值。为这种目的使用环境变量并不罕见，但这可能有缺点，特别是如果该值被认为是secret的。

作为环境变量的替代品，许多云平台现在允许你将配置映射到安装的数据卷中。例如，Kubernetes可以将[`ConfigMaps`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#populate-a-volume-with-data-stored-in-a-configmap)和[`Secrets`](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)都装入卷中。

有两种常见的卷装模式可以使用。

1. 一个单一的文件包含一套完整的属性（通常写成YAML）。
2. 多个文件被写入一个目录树，文件名成为 "key"，内容成为 "value"。

对于第一种情况，你可以使用`spring.config.import`直接导入YAML或属性文件，如[上文](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.importing)所述。对于第二种情况，你需要使用`configtree:`前缀，以便Spring Boot知道它需要将所有文件作为属性公开。

作为一个例子，让我们想象一下，Kubernetes已经安装了以下卷。

```text
etc/
  config/
    myapp/
      username
      password
```

`username`文件的内容将是一个配置值，而`password`的内容将是一个secret。

要导入这些属性，你可以在你的`application.properties`或`application.yaml`文件中添加以下内容。

```yaml
spring:
  config:
    import: "optional:configtree:/etc/config/"
```

然后你可以从 `Environment` 中以常规方式访问或注入`myapp.username`和`myapp.password`属性。

> 配置树的值可以被绑定到字符串`String`和`byte[]`类型，这取决于预期的内容。

如果你有多个配置树要从同一个父文件夹导入，你可以使用通配符快捷方式。任何以`/*/`结尾的`configtree`:位置都将导入所有直接的子文件夹作为配置树。

例如，给定以下卷。

```text
etc/
  config/
    dbconfig/
      db/
        username
        password
    mqconfig/
      mq/
        username
        password
```

你可以使用`configtree:/etc/config/*/`作为导入位置。

```properties
spring.config.import=optional:configtree:/etc/config/*/
```

这将添加`db.username`、`db.password`、`mq.username`和`mq.password`属性。

> 使用通配符加载的目录是按字母顺序排列的。如果你需要一个不同的顺序，那么你应该把每个位置作为一个单独的导入列出

配置树也可以用于Docker secrets。当Docker swarm服务被授予对secrets的访问权时，该secrets会被装载到容器中。例如，如果一个名为`db.password`的secrets被挂载在`/run/secrets/`的位置，你可以用以下方法让`db.password`对Spring环境可用。

```properties
spring.config.import=optional:configtree:/run/secrets/
```

#### 2.3.7. 属性占位符

`application.properties`和`application.yml`中的值在使用时通过现有的`Environment`过滤，所以你可以参考以前定义的值（例如，从系统属性）。标准的`${name}`属性占位符语法可以在一个值的任何地方使用。

例如，下面的文件将把`app.description`设置为 "MyApp is a Spring Boot application"。

```yaml
app:
  name: "MyApp"
  description: "${app.name} is a Spring Boot application"
```

> 你也可以使用这种技术来创建现有Spring Boot属性的 "short" 变体。详见[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.properties-and-configuration.short-command-line-arguments)的方法。

TODO

{{#include ../license.md}}
