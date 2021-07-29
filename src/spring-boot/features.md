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

#### 2.3.8. 处理多文档文件

Spring Boot允许你将一个物理文件分成多个逻辑文件，每个文件都是独立添加的。文件是按顺序处理的，从上到下。后面的文件可以覆盖前面文件中定义的属性。

对于`application.yml`文件，使用标准的YAML多文档语法。三个连续的连字符代表一个文件的结束，以及下一个文件的开始。

例如，下面的文件有两个逻辑文档。

```yaml
spring.application.name: MyApp
---
spring.config.activate.on-cloud-platform: kubernetes
spring.application.name: MyCloudApp
```

对于 `application.properties` 文件，一个特殊的 `#---`注释被用来标记文件的分割。

```properties
spring.application.name=MyApp
#---
spring.config.activate.on-cloud-platform=kubernetes
spring.application.name=MyCloudApp
```

属性文件的分隔符不能有任何前导空白，而且必须正好有三个连字符。分隔符的前后两行不能是注释。

多文档属性文件通常与激活属性一起使用，如`spring.config.activated.on-profile`。详见[下一节](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.activation-properties)。

不能通过使用`@PropertySource`或`@TestPropertySource`注解加载多文档属性文件。

#### 2.3.9. 激活配置

有时，只有在满足某些条件时才能激活一个给定的属性。例如，你可能有一些属性只有在特定的配置文件被激活时才相关。

你可以使用`spring.config.activation.*`有条件地激活一个属性文件。

以下的激活属性是可用的。

表 2. activation properties

|Property|Note|
| --- | --- |
|`on-profile`|一个配置文件表达式，必须与之匹配才能使文件处于活动状态。|
|`on-cloud-platform`|必须检测到的 `CloudPlatform`，以使文件处于活动状态。|

例如，下面指定第二个文件只有在`Kubernetes`上运行时才有效，并且只有在 `prod` 或 `staging` 配置文件激活时才有效。

```yaml
myprop:
  always-set
---
spring:
  config:
    activate:
      on-cloud-platform: "kubernetes"
      on-profile: "prod | staging"
myotherprop: sometimes-set
```

### 2.4. 加密配置属性

Spring Boot没有为加密属性值提供任何内置支持，但它确实提供了修改Spring `Environment`中包含的值所需的hook。`EnvironmentPostProcessor` 接口允许你在应用程序启动前对 `Environment` 进行操作。详见[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.application.customize-the-environment-or-application-context)。

如果你正在寻找一种安全的方式来存储证书和密码，[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)项目提供了对[HashiCorp Vault](https://www.vaultproject.io/)中存储外部化配置的支持。

### 2.5. 使用YAML工作

[YAML](https://yaml.org/)是JSON的超集，因此是指定分层配置数据的方便格式。只要你的classpath上有[SnakeYAML](https://bitbucket.org/asomov/snakeyaml)库，`SpringApplication`类就会自动支持YAML作为属性的替代品。

> 如果你使用 "Starter"，SnakeYAML将由`spring-boot-starter`自动提供。

#### 2.5.1. 将YAML映射到属性

YAML 文档需要从其分层格式转换为可与 Spring `Environment`一起使用的扁平结构。例如，考虑下面这个YAML文档。

```yaml
environments:
  dev:
    url: https://dev.example.com
    name: Developer Setup
  prod:
    url: https://another.example.com
    name: My Cool App
```

为了从 `Environment` 中访问这些属性，它们将被扁平化，如下所示。

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

同样地，YAML列表也需要进行扁平化处理。它们被表示为带有`[index]`脱引器的属性键。例如，考虑下面的YAML。

```yaml
my:
 servers:
 - dev.example.com
 - another.example.com
```

前面的例子将被转化为这些属性

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

使用 `[index]` 符号的属性可以使用Spring Boot的 `Binder` 类绑定到Java `List` 或 `Set` 对象。详情请见下面的 [类型安全的配置属性](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties)部分。

YAML文件不能通过使用`@PropertySource`或`@TestPropertySource`注解来加载。所以，在你需要以这种方式加载值的情况下，你需要使用一个properties文件。

#### 2.5.2. 直接加载YAML

Spring Framework提供了两个方便的类，可以用来加载YAML文档。`YamlPropertiesFactoryBean`将YAML作为`Properties` 加载，`YamlMapFactoryBean`将YAML作为`Map` 加载。

如果你想将YAML作为Spring的`PropertySource`来加载，你也可以使用`YamlPropertySourceLoader`类。

### 2.6. 配置随机值

`RandomValuePropertySource` 对于注入随机值很有用（例如，注入secrets或test cases）。它可以产生Integer、Long、UUID或字符串，如下面的例子所示。

```yaml
my:
  secret: "${random.value}"
  number: "${random.int}"
  bignumber: "${random.long}"
  uuid: "${random.uuid}"
  number-less-than-ten: "${random.int(10)}"
  number-in-range: "${random.int[1024,65536]}"
```

`random.int*`的语法是`OPEN value (,max) CLOSE`，其中`OPEN,CLOSE`是任何字符，`value,max`是整数。如果提供了`max`，那么`value`是最小值，`max`是最大值（独占）。

### 2.7. 配置系统环境属性

Spring Boot支持为环境属性设置一个前缀。如果系统环境被多个具有不同配置要求的Spring Boot应用程序共享，这将非常有用。系统环境属性的前缀可以直接在`SpringApplication`上设置。

例如，如果你将前缀设置为`input`，诸如`remote.timeout`这样的属性在系统环境中也将被解析为`input.remote.timeout`。

### 2.8. 类型安全的配置属性

使用`@Value("${property}")`注解来注入配置属性有时会很麻烦，特别是当你要处理多个属性或你的数据是分层的。Spring Boot提供了一种处理属性的替代方法，让强类型的Bean管理和验证你的应用程序的配置。

> 另请参见[`@Value`和类型安全配置属性的区别](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.vs-value-annotation)。

#### 2.8.1. JavaBean属性绑定

如下面的例子所示，可以绑定一个声明了标准JavaBean属性的bean。

```java
import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my.service")
public class MyProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() {
        return this.enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() {
            return this.username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return this.password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

        public List<String> getRoles() {
            return this.roles;
        }

        public void setRoles(List<String> roles) {
            this.roles = roles;
        }

    }

}
```

前面的POJO定义了以下属性。

* `my.service.enabled`, 默认值为`false`。
* `my.service.remote-address`, 具有一个可以从`String'强制的类型。
* `my.service.security.username` ，有一个嵌套的 `security` 对象，其名称由属性名称决定。特别是，那里根本没有使用返回类型，可以是`SecurityProperties`。
* `my.service.security.password` 。
* `my.service.security.role` ，有一个默认为`USER`的`String`集合。

映射到Spring Boot中可用的`@ConfigurationProperties`类的属性，通过属性文件、YAML文件、环境变量等进行配置，是公共API，但类本身的访问器（getters/setters）并不意味着可以直接使用。

这样的安排依赖于一个默认的空构造函数，而获取器和设置器通常是强制性的，因为绑定是通过标准的Java Beans属性描述符，就像Spring MVC中一样。在下列情况下，可以省略设置器。

* Map，只要它们被初始化，就需要一个getter，但不一定需要setter，因为它们可以被绑定器改变。
* Collection和Array可以通过索引（通常用YAML）或使用单个逗号分隔的值（属性）来访问。在后一种情况下，一个setter是必须的。我们建议总是为这类类型添加一个setter。如果你初始化一个集合，确保它不是不可变的（如前面的例子）。
* 如果嵌套的POJO属性被初始化（就像前面的例子中的`security`字段），就不需要setter了。如果你想让绑定器通过使用它的默认构造函数来即时创建实例，你需要一个setter。

有些人使用Project Lombok来自动添加getter和setter。确保Lombok不会为这样的类型生成任何特定的构造函数，因为容器会自动使用它来实例化对象。

最后，只考虑标准的Java Bean属性，不支持对静态属性的绑定。

#### 2.8.2. 构造函数绑定

上一节的例子可以用不可变的方式重写，如下例所示。

```java
import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;

@ConstructorBinding
@ConfigurationProperties("my.service")
public class MyProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public boolean isEnabled() {
        return this.enabled;
    }

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        public String getUsername() {
            return this.username;
        }

        public String getPassword() {
            return this.password;
        }

        public List<String> getRoles() {
            return this.roles;
        }

    }

}
```

在这个设置中，`@ConstructorBinding`注解被用来表示应该使用构造函数绑定。这意味着绑定器将期望找到一个具有你希望绑定的参数的构造函数。

`@ConstructorBinding`类的嵌套成员（比如上面例子中的`Security`）也将通过其构造函数被绑定。

默认值可以使用`@DefaultValue`来指定，同样的转换服务将被应用于将`String`值强制到缺失属性的目标类型。默认情况下，如果没有属性被绑定到`Security`，`MyProperties`实例将包含一个`security`的`null`值。如果你希望返回一个非空的`Security`实例，即使没有属性与之绑定，你可以使用一个空的`@DefaultValue`注释来实现。

```java
public MyProperties(boolean enabled, InetAddress remoteAddress, @DefaultValue Security security) {
    this.enabled = enabled;
    this.remoteAddress = remoteAddress;
    this.security = security;
}
```

要使用构造函数绑定，该类必须使用`@EnableConfigurationProperties`或配置属性扫描启用。你不能对通过常规Spring机制创建的Bean使用构造函数绑定（例如：`@Component`Bean，通过`@Bean`方法创建的Bean或使用`@Import`加载的Bean）。

如果你的类有一个以上的构造函数，你也可以直接在应该被绑定的构造函数上使用`@ConstructorBinding`。

不建议将`java.util.Optional`与`@ConfigurationProperties`一起使用，因为它主要是作为一个返回类型使用。因此，它并不适合配置属性注入。为了与其他类型的属性保持一致，如果你确实声明了一个`Optional`属性，但它没有值，`null`而不是一个空的`Optional`将被绑定。

#### 2.8.3. 启用@ConfigurationProperties-annotated类型

Spring Boot提供了绑定`@ConfigurationProperties`类型并将其注册为Bean的基础设施。你可以在逐个类的基础上启用配置属性，或者启用配置属性扫描，其工作方式与组件扫描类似。

有时，用`@ConfigurationProperties`注解的类可能不适合扫描，例如，如果你正在开发你自己的自动配置或者你想有条件地启用它们。在这些情况下，使用`@EnableConfigurationProperties` 注解指定要处理的类型列表。这可以在任何`@Configuration`类上进行，如下面的例子所示。

```java
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(SomeProperties.class)
public class MyConfiguration {

}
```

要使用配置属性扫描，请将`@ConfigurationPropertiesScan`注解添加到你的应用程序。通常情况下，它被添加到用`@SpringBootApplication`注解的主应用程序类中，但它也可以被添加到任何`@Configuration`类。默认情况下，扫描将从声明该注解的类的包中发生。如果你想定义特定的包来扫描，你可以这样做，如下面的例子所示。

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationPropertiesScan;

@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "com.example.another" })
public class MyApplication {

}
```

> 当`@ConfigurationProperties` Bean使用配置属性扫描或通过`@EnableConfigurationProperties`注册时，Bean有一个传统的名字：`<prefix>-<fqn>`，其中`<prefix>`是`@ConfigurationProperties`注解中指定的环境键前缀，`<fqn>`是Bean的完全限定名称。如果注解没有提供任何前缀，则只使用Bean的完全合格名称。
>
> 上面例子中的Bean名称是`com.example.app-com.example.app.SomeProperties`。

我们建议`@ConfigurationProperties`只处理环境，特别是不从上下文注入其他Bean。对于角落里的情况，可以使用设置器注入或框架提供的任何`*Aware`接口（如`EnvironmentAware`，如果你需要访问`Environment`）。如果你仍然想使用构造器注入其他Bean，配置属性Bean必须用`@Component`来注释，并使用基于JavaBean的属性绑定。

#### 2.8.4. 使用@ConfigurationProperties-annotated类型

这种配置风格与SpringApplication的外部YAML配置配合得特别好，如下例所示。

```yaml
my:
    service:
        remote-address: 192.168.1.1
        security:
            username: admin
            roles:
              - USER
              - ADMIN

```

要使用`@ConfigurationProperties` bean，你可以用与其他bean相同的方式注入它们，如下例所示

```java
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final SomeProperties properties;

    public MyService(SomeProperties properties) {
        this.properties = properties;
    }

    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        server.start();
        // ...
    }

    // ...

}
```

> 使用`@ConfigurationProperties`还可以让你生成元数据文件，可以被IDE用来为你自己的键提供自动完成。详见[附录](https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html#configuration-metadata)。

#### 2.8.5. 第三方配置

除了使用`@ConfigurationProperties`来注释一个类之外，你也可以在公共的`@Bean`方法上使用它。当你想把属性绑定到你控制之外的第三方组件时，这样做特别有用。

要从`Environment`属性中配置一个Bean，请将`@ConfigurationProperties`添加到其Bean注册中，如下例所示。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class ThirdPartyConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "another")
    public AnotherComponent anotherComponent() {
        return new AnotherComponent();
    }

}
```

任何用`another`前缀定义的JavaBean属性都会被映射到`AnotherComponent`Bean上，其方式类似于前面的`SomeProperties`例子。

#### 2.8.6. 宽松的绑定

Spring Boot在将 `Environment` 属性绑定到`@ConfigurationProperties`bean时使用了一些宽松的规则，因此 `Environment` 属性名称和bean属性名称之间不需要完全匹配。这很有用，常见的例子包括破折号分隔的环境属性（例如，`context-path`绑定到`contextPath`），和大写的环境属性（例如，`PORT`绑定到`port`）。

作为一个例子，考虑下面的`@ConfigurationProperties`类。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "my.main-project.person")
public class MyPersonProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

通过前面的代码，以下的属性名称都可以使用。

表 3. relaxed binding

|Property|Note|
| --- | --- |
|`my.main-project.person.first-name`|短横线案例，建议在`.properties`和`.yml`文件中使用。|
|`my.main-project.person.firstName`|标准的驼峰语法。|
|`my.main-project.person.first_name`|下划线符号，这是一种用于`.properties`和`.yml`文件的替代格式。|
|`MY_MAINPROJECT_PERSON_FIRSTNAME`|大写格式，在使用系统环境变量时建议使用大写格式。|

> 注释的 `prefix` 值必须是短横线（小写并以`-`分隔，如`my.main-project.person`）。

表 4. relaxed binding rules per property source

|配置源|Simple|List|
| --- | --- | --- |
|Properties 文件|驼峰、短横线或下划线|使用`[]`或逗号分隔值的标准列表语法|
|YAML 文件|驼峰、短横线或下划线|标准YAML列表语法或逗号分隔的值|
|Environment Variables|大写格式，下划线为分隔符（见[从环境变量绑定](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)）。|用下划线包围的数值（见[从环境变量绑定](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)）。|
|System properties|驼峰、短横线或下划线|使用`[]`或逗号分隔值的标准列表语法|

> 我们建议，在可能的情况下，属性应以小写的短横线格式存储，例如`my.person.first-name=Rod`。

Binding Maps

当绑定到`Map`属性时，你可能需要使用一个特殊的括号符号，以便保留原始的`key`值。如果键没有被`[]`包围，任何非字母数字、`-`或`.`的字符都会被删除。

例如，考虑将以下属性绑定到一个`Map<String,String>`。

```yaml
my:
  map:
    "[/key1]": "value1"
    "[/key2]": "value2"
    "/key3": "value3"

```

对于YAML文件，括号需要用引号包围，以使键被正确解析。

上面的属性将绑定到一个`Map`，`/key1`，`/key2`和`key3`作为地图的键。斜线已经从`key3`中移除，因为它没有被方括号包围。

如果你的`key`包含一个`.`，并且你要绑定到非标量值，你可能偶尔也需要使用括号符号。例如，将`a.b=c`绑定到`Map<String, Object>`将返回一个Map，其条目为`{"a"={"b"="c"}`，而`[a.b]=c`将返回一个Map，其条目为`{"a.b"="c"}`。

从环境变量绑定

大多数操作系统对可用于环境变量的名称有严格的规定。例如，Linux shell变量只能包含字母（`a`到`z`或`A`到`Z`）、数字（`0`到`9`）或下划线字符（`_`）。按照惯例，Unix shell变量的名称也将采用大写字母。

Spring Boot宽松的绑定规则被设计为尽可能与这些命名限制兼容。

要将规范形式的属性名称转换为环境变量名称，你可以遵循这些规则。

* 用下划线 ( `_` ) 替换点 ( `.` ) 。
* 删除任何破折号 ( `-` )。
* 转换为大写字母。

例如，配置属性`spring.main.log-startup-info`将是一个名为`SPRING_MAIN_LOGSTARTUPINFO`的环境变量。

环境变量也可以在绑定到对象列表时使用。要绑定到一个`List`，在变量名称中，元素编号应该用下划线包围。

例如，配置属性`my.service[0].other`将使用一个名为`MY_SERVICE_0_OTHER`的环境变量。

#### 2.8.7. 合并复杂类型

当列表被配置在多个地方时，覆盖的作用是替换整个列表。

例如，假设一个`MyPojo`对象的`name`和`description`属性默认为`null`。下面的例子从`MyProperties`暴露了一个`MyPojo`对象的列表。

```java
import java.util.ArrayList;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my")
public class MyProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

考虑以下配置。

```yaml
my:
  list:
  - name: "my name"
    description: "my description"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  list:
  - name: "my another name"
```

如果 `dev` 配置文件没有激活，`MyProperties.list` 包含一个 `MyPojo` 条目，如之前定义的。然而，如果 `dev` 配置文件被激活，`list` 仍然只包含一个条目（名字为 `my another name`，描述为 `null`）。这种配置*不会*在列表中添加第二个`MyPojo`实例，也不会合并项目。

当一个 `List` 在多个配置文件中被指定时，具有最高优先级的一个（而且只有那个）被使用。考虑一下下面的例子。

```yaml
my:
  list:
  - name: "my name"
    description: "my description"
  - name: "another name"
    description: "another description"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  list:
  - name: "my another name"

```

在前面的例子中，如果`dev`配置文件是激活的，`MyProperties.list`包含*一个*`MyPojo`条目（名字为`my another name`，描述为`null`）。对于YAML，逗号分隔的列表和YAML列表都可以用来完全覆盖列表的内容。

对于`Map`属性，你可以用从多个来源抽取的属性值进行绑定。然而，对于多个来源中的同一属性，使用具有最高优先级的那个。下面的例子从`MyProperties`暴露了一个`Map<String, MyPojo>`。

```java
import java.util.LinkedHashMap;
import java.util.Map;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my")
public class MyProperties {

    private final Map<String, MyPojo> map = new LinkedHashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```

考虑以下配置。

```yaml
my:
  map:
    key1:
      name: "my name 1"
      description: "my description 1"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  map:
    key1:
      name: "dev name 1"
    key2:
      name: "dev name 2"
      description: "dev description 2"

```

如果`dev`配置文件没有激活，`MyProperties.map`包含一个键`key1`的条目（名称为`my name 1`，描述为`my description 1`）。然而，如果 `dev` 配置文件被激活，`map`包含两个条目，键为`key1`（名称为`dev name 1`，描述为`my description 1`）和`key2`（名称为`dev name 2`，描述为`dev description 2`）。

> 前面的合并规则适用于所有属性源的属性，而不仅仅是文件。

#### 2.8.8. 属性转换

当Spring Boot与`@ConfigurationProperties` Bean绑定时，它试图将外部应用程序的属性胁迫为正确的类型。如果你需要自定义类型转换，你可以提供一个`ConversionService` bean（有一个名为 `conversionService` 的bean）或自定义属性编辑器（通过`CustomEditorConfigurer`bean）或自定义`Converters`（有注释为`@ConfigurationPropertiesBinding`的bean定义）。

由于这个Bean是在应用程序生命周期的早期被请求的，请确保限制你的`ConversionService'``ConversionService`不需要配置键强制，你可能想重命名它，并且只依赖用`@ConfigurationPropertiesBinding`限定的自定义转换器。

##### Converting Durations

Spring Boot对表达持续时间有专门的支持。如果你公开了一个`java.time.Duration`属性，应用程序属性中就有以下格式。

* 普通的`long`表示法（使用毫秒作为默认单位，除非指定了`@DurationUnit`）。
* 标准的ISO-8601格式[由`java.time.Duration`使用](https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html#parse-java.lang.CharSequence-)
* 一个更易读的格式，其中值和单位是耦合的（例如，`10s`表示10秒）

请考虑以下例子。

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.convert.DurationUnit;

@ConfigurationProperties("my")
public class MyProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```

要指定一个30秒的会话超时，`30`、`PT30S`和`30s`都是等价的。读取超时为500ms，可以用以下任何一种形式指定。`500`, `PT0.5S`和`500ms`。

你也可以使用任何支持的单位。这些单位是

* `ns`代表纳秒
* `us`代表微秒
* `ms`代表毫秒
* `s`代表秒
* `m`代表分钟
* `h`代表小时
* `d`代表天

默认单位是毫秒，可以使用`@DurationUnit`来重写，如上面的例子所示。

如果你喜欢使用构造函数绑定，同样的属性可以被暴露出来，如下面的例子所示。

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;
import org.springframework.boot.convert.DurationUnit;

@ConfigurationProperties("my")
@ConstructorBinding
public class MyProperties {

    private final Duration sessionTimeout;

    private final Duration readTimeout;

    public MyProperties(@DurationUnit(ChronoUnit.SECONDS) @DefaultValue("30s") Duration sessionTimeout,
            @DefaultValue("1000ms") Duration readTimeout) {
        this.sessionTimeout = sessionTimeout;
        this.readTimeout = readTimeout;
    }

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

}
```

如果你正在升级一个`Long属`性，如果它不是毫秒，请确保定义单位（使用`@DurationUnit`）。这样做提供了一个透明的升级路径，同时支持更丰富的格式。

Converting periods

除了期限，Spring Boot还可以使用`java.time.Period`类型。以下格式可以在应用程序属性中使用。

* 常规的`int`表示法（使用天作为默认单位，除非指定了`@PeriodUnit`）。
* 标准的ISO-8601格式[由`java.time.Period`使用](https://docs.oracle.com/javase/8/docs/api/java/time/Period.html#parse-java.lang.CharSequence-)
* 一个更简单的格式，其中值和单位对是耦合的（例如，`1y3d`表示1年3天)

简单格式支持以下单位。

* `y`代表年
* `m`代表月
* `w`代表周
* `d`代表天

`java.time.Period`类型实际上从未存储过周数，它是一个快捷方式，意味着 "7天"。

##### 转换数据大小

Spring Framework有一个`DataSize`值类型，以字节为单位表达大小。如果你公开了一个`DataSize`属性，在应用程序属性中可以使用以下格式。

* 常规的 `long` 表示（使用字节作为默认单位，除非指定了`@DataSizeUnit`）。
* 一个更易读的格式，其中值和单位是耦合的（例如，`10MB`意味着10兆字节）。

考虑一下下面的例子。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties("my")
public class MyProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```

要指定一个10兆字节的缓冲区大小，`10`和`10MB`是等价的。256字节的大小阈值可以指定为`256`或`256B`。

你也可以使用任何支持的单位。这些单位是

* `B`代表字节
* `KB`代表千字节
* `MB`表示兆字节
* `GB`代表千兆字节
* `TB`代表太字节

默认单位是字节，可以使用`@DataSizeUnit`来重写，如上面的例子所示。

如果你喜欢使用构造函数绑定，同样的属性可以被暴露出来，如下面的例子所示。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties("my")
@ConstructorBinding
public class MyProperties {

    private final DataSize bufferSize;

    private final DataSize sizeThreshold;

    public MyProperties(@DataSizeUnit(DataUnit.MEGABYTES) @DefaultValue("2MB") DataSize bufferSize,
            @DefaultValue("512B") DataSize sizeThreshold) {
        this.bufferSize = bufferSize;
        this.sizeThreshold = sizeThreshold;
    }

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

}
```

> 如果你正在升级一个`Long`属性，如果它不是字节，请确保定义单位（使用`@DataSizeUnit`）。这样做提供了一个透明的升级路径，同时支持更丰富的格式。

#### 2.8.9. @ConfigurationProperties 验证

只要使用Spring的`@Validated`注解，Spring Boot就会尝试验证`@ConfigurationProperties`类。你可以直接在你的配置类上使用JSR-303的`javax.validation`约束注解。要做到这一点，请确保你的classpath上有一个兼容的JSR-303实现，然后将约束注解添加到你的字段中，如下面的例子所示。

```java
import java.net.InetAddress;

import javax.validation.constraints.NotNull;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

}
```

> 你也可以通过在创建配置属性的`@Bean`方法上注解`@Validated`来触发验证。

为了确保总是触发嵌套属性的验证，即使没有找到属性，相关的字段必须用`@Valid`来注释。下面的例子建立在前面的 `MyProperties` 的基础上。

```java
import java.net.InetAddress;

import javax.validation.Valid;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        @NotEmpty
        private String username;

        public String getUsername() {
            return this.username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

    }

}
```

你也可以通过创建一个名为 `configurationPropertiesValidator` 的bean定义来添加一个自定义的Spring`Validator`。`@Bean`方法应该被声明为`static`。配置属性验证器是在应用程序生命周期的早期创建的，将`@Bean`方法声明为静态，可以让Bean被创建而不需要实例化`@Configuration`类。这样做可以避免早期实例化可能引起的任何问题。

> `spring-boot-actuator`模块包括一个端点，暴露了所有`@ConfigurationProperties` Bean。将你的网络浏览器指向`/actuator/configprops`或使用相应的JMX端点。详见 [生产就绪特性](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints) 部分。

#### 2.8.10. @ConfigurationProperties vs. @Value

`@Value`注解是一个核心的容器功能，它不提供与类型安全的配置属性相同的功能。下表总结了`@ConfigurationProperties`和`@Value`所支持的功能。

|Feature|`@ConfigurationProperties`|`@Value`|
| --- | --- | --- |
|[Relaxed binding](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding)|Yes|Limited (see [note below](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.vs-value-annotation.note))|
|[Meta-data support](https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html#configuration-metadata)|Yes|No|
|`SpEL` evaluation|No|Yes|

如果你想使用`@Value`，我们建议你使用属性名称的规范形式（只使用小写字母的kebab-case）。这将允许Spring Boot使用与放松绑定`@ConfigurationProperties`时一样的逻辑。例如，`@Value("{demo.item-price}")`将从`application.properties`文件中获取`demo.item-price`和`demo.itemPrice`形式，以及从系统环境中获取`DEMO_ITEMPRICE`。如果你用`@Value("{demo.itemPrice}")`代替，`demo.item-price`和`DEMO_ITEMPRICE`将不会被考虑。

> 如果你为自己的组件定义了一组配置键，我们建议你将它们归入一个用`@ConfigurationProperties`注解的POJO。这样做将为你提供结构化的、类型安全的对象，你可以将其注入到你自己的bean中。

来自[应用程序属性文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files)的`SpEL`表达式在解析这些文件和填充环境时不会被处理。然而，我们可以在`@Value`中写一个`SpEL`表达式。如果来自应用程序属性文件的属性值是一个`SpEL`表达式，当通过`@Value`消耗时，它将被评估。

## 3. Profiles

Spring Profiles提供了一种方法来隔离你的应用程序配置的一部分，使其只在某些环境下可用。任何`@Component`、`@Configuration` 或`@ConfigurationProperties`都可以用`@Profile`来标记，以限制它被加载的时间，如下面的例子所示。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

> 如果`@ConfigurationProperties` Bean是通过`@EnableConfigurationProperties`注册的，而不是自动扫描，则需要在具有`@EnableConfigurationProperties`注释的`@Configuration`类上指定`@Profile`注释。在`@ConfigurationProperties`被扫描的情况下，`@Profile` 可以在`@ConfigurationProperties` 类本身指定。

你可以使用`spring.profiles.active` `Environment`属性来指定哪些配置文件是活动的。你可以通过本章前面描述的任何方式来指定该属性。例如，你可以在你的`application.properties`中包含它，如下面的例子所示。

```yaml
spring:
  profiles:
    active: "dev,hsqldb"
```

你也可以通过使用以下开关在命令行中指定它：`--spring.profiles.active=dev,hsqldb` 。

如果没有激活配置文件，就会启用一个默认的配置文件。默认配置文件的名称是`default`，可以使用`spring.profiles.default``Environment`属性对其进行调整，如下面例子所示。

```yaml
spring:
  profiles:
    default: "none"
```

### 3.1. 添加活动配置文件

`spring.profiles.active`属性遵循与其他属性相同的排序规则。最高的`PropertySource`获胜。这意味着你可以在`application.properties`中指定活动配置文件，然后通过使用命令行开关替换它们。

有时，有一些属性可以添加到活动配置文件中，而不是替换它们，这很有用。`SpringApplication`入口点有一个Java API用于设置额外的配置文件（也就是在那些由`spring.profiles.active`属性激活的配置文件之上）。参见[SpringApplication](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/SpringApplication.html)中的`setAdditionalProfiles()`方法。配置文件组，在[下一节](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles.groups)中描述，如果一个给定的配置文件是激活的，也可以用来添加激活的配置文件。

### 3.2. Profile Groups

偶尔，你在你的应用程序中定义和使用的配置文件过于精细，使用起来就会很麻烦。例如，你可能有`proddb`和`prodmq`配置文件，用来独立启用数据库和消息传递功能。

为了帮助解决这个问题，Spring Boot允许你定义配置文件组。配置文件组允许你为相关的配置文件组定义一个逻辑名称。

例如，我们可以创建一个`production`组，由`proddb`和`prodmq`配置文件组成。

```yaml
spring:
  profiles:
    group:
      production:
      - "proddb"
      - "prodmq"
```

现在可以使用`--spring.profiles.active=production`来启动我们的应用程序，以一次性激活`production`、`proddb`和`prodmq`配置文件。

### 3.3. 以编程方式设置配置文件

你可以在应用运行前通过调用`SpringApplication.setAdditionalProfiles(...)`以编程方式设置活动配置文件。也可以通过使用Spring的`ConfigurableEnvironment`接口来激活配置文件。

### 3.4. 特定的配置文件

`application.properties`（或`application.yml`）和通过`@ConfigurationProperties`引用的文件的配置文件特定变体都被视为文件并加载。详情请见 [配置文件特定文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)。

## 4. Logging

Spring Boot在所有内部日志中使用[Commons Logging](https://commons.apache.org/logging)，但对底层日志的实现保持开放。为[Java Util Logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html)、[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback](https://logback.qos.ch/)提供默认配置。在每一种情况下，记录器都被预设为使用控制台输出，也可以选择文件输出。

默认情况下，如果你使用 "Starter"，Logback被用来做日志记录。适当的Logback路由也包括在内，以确保使用Java Util Logging、Commons Logging、Log4J或SLF4J的依赖库都能正确工作。

有很多适用于Java的日志框架。如果上面的列表看起来很混乱，请不要担心。一般来说，你不需要改变你的日志依赖，Spring Boot的默认值就很好用。

当你把你的应用程序部署到servlet容器或应用服务器时，通过Java Util Logging API执行的日志不会被传送到你的应用程序的日志。这可以防止由容器或其他已经部署到它的应用程序执行的日志出现在你的应用程序的日志中。

### 4.1. 日志格式

Spring Boot的默认日志输出类似于下面的例子。

```text
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出的项目如下。

* 日期和时间：精确到毫秒，易于排序。
* 日志级别: `ERROR`, `WARN`, `INFO`, `DEBUG`, 或`TRACE`.
* 进程ID。
* 一个`---`分隔符来区分实际日志信息的开始。
* 线程名称：包含在方括号中（对于控制台输出可能会被截断）。
* 记录器名称：这通常是源类的名称（通常是缩写）。
* 日志消息。

> Logback没有`FATAL`级别。它被映射到 `ERROR`。

### 4.2. 控制台输出

默认的日志配置是在写信息的时候向控制台echo。默认情况下，`ERROR` 级别、`WARN` 级别和 `INFO` 级别的消息被记录下来。你也可以通过使用`--debug`标志启动你的应用程序来启用 `调debug` 模式。

```shell
$ java -jar myapp.jar --debug
```

> 你也可以在你的 `application.properties`中指定`debug=true`。

当调试模式被启用时，一些核心记录器（嵌入式容器、Hibernate和Spring Boot）被配置为输出更多信息。启用调试模式并不配置你的应用程序以`DEBUG`级别记录所有信息。

另外，你可以通过在启动应用程序时设置`--trace`标志（或在`application.properties`中设置`trace=true`）来启用 `trace` 模式。这样做可以对一些核心记录器（嵌入式容器、Hibernate模式生成和整个Spring组合）进行跟踪记录。

#### 4.2.1. 彩色编码的输出

如果你的终端支持ANSI，就会使用彩色输出来帮助阅读。你可以将`spring.output.ansi.enabled`设置为一个[支持的值](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)来覆盖自动检测。

颜色编码是通过使用`%clr`转换词来配置的。在其最简单的形式下，转换器根据日志级别对输出进行着色，如下面的例子所示。

```text
%clr(%5p)
```

下表描述了日志级别与颜色的映射。

|级别|颜色|
| --- | --- |
|`FATAL`|红色|
|`ERROR`|红色|
|`WARN`|黄色|
|`INFO`|绿色|
|`DEBUG`|绿色|
|`TRACE`|绿色

另外，你也可以通过为转换提供一个选项来指定应该使用的颜色或样式。例如，要使文本为黄色，请使用以下设置。

```text
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式。

* `blue`
* `cyan`
* `faint`
* `green`
* `magenta`
* `red`
* `yellow`

### 4.3. 输出到文件

默认情况下，Spring Boot只向控制台记录日志，不写日志文件。如果你想在控制台输出之外写日志文件，你需要设置`logging.file.name`或`logging.file.path`属性（例如，在你的`application.properties`）。

下表显示了`logging.*`属性如何被一起使用。

Table 5. Logging properties

|`logging.file.name`|`logging.file.path`|Example|Description|
| --- | --- | --- | --- |
|*(none)*|*(none)*||只在控制台进行记录。|
|指定文件|*(none)*|`my.log`|写到指定的日志文件。名称可以是准确的位置，也可以是与当前目录的相对位置。|
|*(none)*|指定目录|`/var/log`|将`spring.log`写到指定目录。名称可以是准确的位置，也可以是与当前目录的相对位置。|

日志文件在达到10MB时就会轮换，与控制台输出一样，默认情况下会记录`ERROR`-级、`WARN`-级和`INFO`-级的信息。

> Logging properties 独立于实际的日志基础设施。因此，特定的配置KEY（如Logback的`logback.configurationFile`）不由spring Boot管理。

### 4.4. 文件轮换

如果你使用`Logback`，可以使用你的`application.properties`或`application.yaml`文件来微调日志轮换设置。对于所有其他的日志系统，你需要自己直接配置旋转设置（例如，如果你使用Log4J2，那么你可以添加一个`log4j.xml`文件）。

支持以下轮换策略属性。

|Name|Description|
| --- | --- |
|`logging.logback.rollingpolicy.file-name-pattern`|用于创建日志档案的文件名模式。|
|`logging.logback.rollingpolicy.clean-history-on-start`|如果应用程序启动时应进行日志归档清理。|
|`logging.logback.rollingpolicy.max-file-size`|日志文件归档前的最大尺寸。|
|`logging.logback.rollingpolicy.total-size-cap`|日志档案在被删除前的最大尺寸。|
|`logging.logback.rollingpolicy.max-history`|保存日志档案的天数（默认为7天）。|

### 4.5. 日志级别

所有支持的日志系统都可以通过使用`logging.level.<logger-name>=<level>`在Spring的`Environment`中（例如，在`application.properties`中）设置日志级别，其中`level`是TRACE, DEBUG, INFO, WARN, ERROR, FATAL, 或OFF之一。`root`记录器可以通过使用`logging.level.root`来配置。

下面的例子显示了`application.properties`中潜在的日志设置。

```yaml
logging:
  level:
    root: "warn"
    org.springframework.web: "debug"
    org.hibernate: "error"

```

也可以使用环境变量来设置日志级别。例如，`LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG`将设置`org.springframework.web`为`DEBUG`。

> 上述方法只适用于包级日志。由于放松的绑定总是将环境变量转换为小写字母，所以不可能用这种方式为单个类配置日志。如果你需要为一个类配置日志，你可以使用[the `SPRING_APPLICATION_JSON`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.application-json)变量。

### 4.6. Log Groups

能够将相关的日志记录器分组，以便同时对它们进行配置，这通常很有用。例如，你可能经常改变所有Tomcat相关的日志记录器的日志级别，但你不容易记住最高级别的包。

为了帮助解决这个问题，Spring Boot允许你在Spring `Environment`中定义日志组。例如，你可以这样定义一个 "tomcat" 组，把它添加到你的`application.properties`中。

```yaml
logging:
  group:
    tomcat: "org.apache.catalina,org.apache.coyote,org.apache.tomcat"
```

一旦定义，就可以用一行字来改变组中所有记录仪的级别。

```yaml
logging:
  level:
    tomcat: "trace"
```

Spring Boot包括以下预定义的日志组，可以开箱即用。

|Name|Loggers|
| --- | --- |
|web|`org.springframework.core.codec` , `org.springframework.http` , `org.springframework.web` , `org.springframework.boot.actuate.endpoint.web` , `org.springframework.boot.web.servlet.ServletContextInitializerBeans`|
|sql|`org.springframework.jdbc.core` , `org.hibernate.SQL` , `org.jooq.tools.LoggerListener`|

### 4.7. 使用日志停机Hook

为了在你的应用程序终止时释放日志资源，我们提供了一个关机Hook，它将在JVM退出时触发日志系统清理。这个关机钩子是自动注册的，除非你的应用程序是以war文件的形式部署。如果你的应用程序有复杂的上下文层次结构，关闭Hook可能无法满足你的需求。如果不能，请禁用关机Hook，并研究底层日志系统直接提供的选项。例如，Logback提供了[context selectors](http://logback.qos.ch/manual/loggingSeparation.html)，允许每个Logger在它自己的上下文中被创建。你可以使用`logging.register-shutdown-hook`属性来禁用关机Hook。将其设置为`false`将禁用注册。你可以在你的`application.properties`或`application.yaml`文件中设置该属性。

```yaml
logging:
  register-shutdown-hook: false
```

### 4.8. 自定义日志配置

各种日志系统可以通过在classpath上包含适当的库来激活，并且可以通过在classpath的根部或由以下Spring `Environment` 属性指定的位置提供合适的配置文件来进一步定制：`logging.config` 。

你可以通过使用`org.springframework.boot.logging.LoggingSystem`系统属性来强制Spring Boot使用一个特定的日志系统。该值应该是`LoggingSystem`实现的完全合格类名。你也可以通过使用`none`的值来完全禁用Spring Boot的日志配置。

由于日志是在创建`ApplicationContext`**之前**初始化的，所以不可能从Spring`@Configuration`文件中的`@PropertySources`控制日志。改变日志系统或完全禁用它的唯一方法是通过系统属性。

根据你的日志系统，会加载以下文件。

|Logging System|Customization|
| --- | --- |
|Logback|`logback-spring.xml` , `logback-spring.groovy` , `logback.xml` , or `logback.groovy`|
|Log4j2|`log4j2-spring.xml` or `log4j2.xml`|
|JDK (Java Util Logging)|`logging.properties`|

> 在可能的情况下，我们建议你使用`-spring`变体来进行日志配置（例如，`logback-spring.xml`而不是`logback.xml`）。如果你使用标准配置位置，Spring不能完全控制日志初始化。

当从 `executable jar`中运行时，Java Util Logging有一些已知的类加载问题，会导致问题。如果可能的话，我们建议你在从 "executable jar"中运行时避免它。

为了帮助定制，其他一些属性从Spring环境转移到系统属性，如下表所述。

|Spring Environment|System Property|Comments|
| --- | --- | --- |
|`logging.exception-conversion-word`|`LOG_EXCEPTION_CONVERSION_WORD`|记录异常时使用的转换词。|
|`logging.file.name`|`LOG_FILE`|如果定义了，它将用于默认的日志配置中。|
|`logging.file.path`|`LOG_PATH`|如果定义了，它将用于默认的日志配置中。|
|`logging.pattern.console`|`CONSOLE_LOG_PATTERN`|在控制台（stdout）使用的日志模式。|
|`logging.pattern.dateformat`|`LOG_DATEFORMAT_PATTERN`|日志日期格式的格式化。|
|`logging.charset.console`|`CONSOLE_LOG_CHARSET`|控制台使用的字符集。|
|`logging.pattern.file`|`FILE_LOG_PATTERN`|要在文件中使用的日志模式（如果`LOG_FILE`被启用）。|
|`logging.charset.file`|`FILE_LOG_CHARSET`|用于文件记录的字符集（如果`LOG_FILE`被启用）。|
|`logging.pattern.level`|`LOG_LEVEL_PATTERN`|呈现日志级别时使用的格式（默认为`%5p`）。|
|`PID`|`PID`|当前的进程ID（如果可能的话，在没有定义为操作系统环境变量的情况下被发现）。|

如果你使用Logback，以下属性也会被转移。

|Spring Environment|System Property|Comments|
| --- | --- | --- |
|`logging.logback.rollingpolicy.file-name-pattern`|`LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN`|滚动日志文件名的模式（默认为`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`）。|
|`logging.logback.rollingpolicy.clean-history-on-start`|`LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START`|是否在启动时清理归档日志文件。|
|`logging.logback.rollingpolicy.max-file-size`|`LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE`|最大日志文件大小。|
|`logging.logback.rollingpolicy.total-size-cap`|`LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP`|要保存的日志备份的总大小。|
|`logging.logback.rollingpolicy.max-history`|`LOGBACK_ROLLINGPOLICY_MAX_HISTORY`|要保留的最大归档日志文件数量。|

所有支持的日志系统在解析其配置文件时都可以查阅系统属性。例子见spring-boot.jar中的默认配置。

* [Logback](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
* [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
* [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

如果你想在日志属性中使用占位符，你应该使用[Spring Boot的语法](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.property-placeholders)，而不是底层框架的语法。值得注意的是，如果你使用Logback，你应该使用`:`作为属性名和其默认值之间的分隔符，而不是使用:`-`。

你可以只通过覆盖 `LOG_LEVEL_PATTERN` (或 Logback 的 `logging.pattern.level`)来向日志行添加 MDC 和其他临时内容。例如，如果你使用`logging.pattern.level=user:%X{user} %5p` ，那么默认的日志格式包含一个 "user" 的MDC条目，如果它存在的话，如下例所示。

```text
2019-08-30 12:30:04.031 user:someone INFO 22174 --- [  nio-8080-exec-0] demo.Controller
Handling authenticated request
```

### 4.9. Logback扩展

Spring Boot包括一些对Logback的扩展，可以帮助进行高级配置。你可以在你的`logback-spring.xml`配置文件中使用这些扩展。

> 因为标准的`logback.xml`配置文件被过早加载，你不能在其中使用扩展。你需要使用`logback-spring.xml`或者定义一个`logging.config`属性。

这些扩展不能与Logback的[配置扫描](https://logback.qos.ch/manual/configuration.html#autoScan)一起使用。如果你试图这样做，对配置文件进行修改会导致类似于以下的错误被记录下来。

```text
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

#### 4.9.1. 特定配置文件配置

`<springProfile>`标签让你可以根据活动的Spring配置文件选择性地包括或排除配置的部分。配置文件部分支持在`<configuration>`元素的任何地方。使用`name`属性来指定接受配置的配置文件。`<springProfile>`标签可以包含一个配置文件名称（例如`staging`）或一个配置文件表达式。配置文件表达式允许表达更复杂的配置逻辑，例如`production & (eu-central | eu-west)`。查看[参考指南](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/core.html#beans-definition-profiles-java)了解更多细节。下面的列表显示了三个配置文件的例子。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

#### 4.9.2. Environment Properties

`<springProperty>`标签让你从Spring `Environment`中公开属性，以便在Logback中使用。如果你想在Logback配置中访问`application.properties`文件中的值，这样做会很有用。该标签的工作方式与Logback的标准`<property>`标签类似。然而，你不是直接指定一个 `value`，而是指定该属性的 `source`（来自 `Environment`）。如果你需要在 `local` 范围以外的地方存储该属性，你可以使用 `scope` 属性。如果你需要一个后备值（万一该属性没有在`Environment`中设置），你可以使用`defaultValue`属性。下面的例子显示了如何公开属性以便在Logback中使用。

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

`source`必须以短横线格式指定（如`my.property-name`）。然而，属性可以通过使用宽松的规则添加到 `Environment` 中。

## 5. 国际化

{{#include ../license.md}}
