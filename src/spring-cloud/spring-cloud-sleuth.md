# Spring Cloud Sleuth

- 当前版本：3.0.3
- 修改时间：2021年7月23日
- 官方文档：[https://docs.spring.io/spring-cloud-sleuth/docs/current/reference/html/](https://docs.spring.io/spring-cloud-sleuth/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-sleuth](https://github.com/spring-cloud/spring-cloud-sleuth)

## 1. 法律条款

本文件的副本可供你自己使用和分发给其他人，但你不得对这些副本收取任何费用，而且每份副本都要包含本版权声明，无论是以印刷品还是电子方式分发。

## 2. 开始使用

如果你刚开始使用Spring Cloud Sleuth或Spring，请先阅读本节。它回答了 "什么？"、"如何？"和 "为什么？"等基本问题。它包括对Spring Cloud Sleuth的介绍，以及安装说明。然后，我们将引导你构建你的第一个Spring Cloud Sleuth应用程序，在此过程中讨论一些核心原则。

### 2.1. Spring Cloud Sleuth 介绍

Spring Cloud Sleuth为[Spring Cloud](https://cloud.spring.io/)提供分布式追踪解决方案的API。它与[OpenZipkin Brave](https://github.com/openzipkin/brave) 集成

Spring Cloud Sleuth能够追踪你的请求和消息，这样你就可以将该通信与相应的日志条目联系起来。你还可以将追踪信息导出到外部系统，以可视化延迟。Spring Cloud Sleuth直接支持[OpenZipkin](https://zipkin.io/)兼容系统。

#### 2.1.1. 术语介绍

Spring Cloud Sleuth 借鉴了 [Dapper’s](https://research.google.com/pubs/pub36356.html) 术语表.

**Span**: 工作的基本单位。例如，发送一个RPC是一个新的跨度，正如发送对RPC的响应一样。span也有其他数据，如描述、有时间戳的事件、键值注释（标签）、引起它们的span的ID和进程ID（通常是IP地址）。

跨度可以被启动和停止，并保持跟踪其时间信息。一旦你创建了一个跨度，你必须在未来的某个时间点停止它。

**Tree**：一组跨度形成一个树状结构。例如，如果你运行一个分布式大数据存储，一个跟踪可能由一个PUT请求形成。

**Annotation/Event**：用来记录一个事件在时间上的存在。

从概念上讲，在一个典型的RPC场景中，我们标记这些事件是为了强调发生了什么样的动作（这并不意味着在物理上这样的事件会被设置在一个跨度上）。

- **cs**：客户端发送。客户端已经发出了请求。这个注解表示跨度的开始。
- **sr**：服务器收到了。服务器端得到了请求并开始处理它。从这个时间戳中减去`cs`时间戳可以看出网络延迟。
- **ss**：服务器发送。在请求处理完成后的注释（当响应被发回给客户端）。从这个时间戳中减去`sr`时间戳，可以看出服务器端处理该请求所需的时间。
- **cr**：客户端收到了。标志着跨度的结束。客户端已成功收到服务器端的响应。从这个时间戳中减去`cs`时间戳，显示出客户端从服务器端接收响应所需的全部时间。

下图显示了**Span**和**Trace**在一个系统中的样子。

![ ](https://cdn.jsdelivr.net/gh/KevinBlandy/springcloud-images/2021/07/24/b4a2078f5e0c4b7aa82691e6a4cf33f3.png)

一个`note`的每种颜色代表一个跨度（有七个跨度--从**A**到**G**）。请看下面这个`note`。

```text
Trace Id = X
Span Id = D
Client Sent
```

这个说明表明，当前跨度的**Trace Id**设置为**X**，**Span Id**设置为**D**。另外，从RPC的角度来看，发生了 `Client Sen`事件。

让我们考虑更多的note。

```text
Trace Id = X
Span Id = A
(no custom span)

Trace Id = X
Span Id = C
(custom span)
```

你可以继续使用已创建的跨度（`no custom span`的例子），或者你可以手动创建子跨度（有`custom span`指示的例子）。

下面的图片显示了跨度的父子关系的样子。

![ ](https://cdn.jsdelivr.net/gh/KevinBlandy/springcloud-images/2021/07/24/a4c26b86588447759c6f6baa744b256b.png)

### 2.2. 发你的第一个基于Spring Cloud sleuth的应用程序

本节介绍了如何开发一个小型的 "Hello World!"网络应用，其中突出了Spring Cloud Sleuth的一些关键功能。我们使用Maven来构建这个项目，因为大多数IDE都支持它。作为追踪器的实现，我们将使用[OpenZipkin Brave](https://github.com/openzipkin/brave)。

> 你可以通过进入[start.spring.io](https://start.spring.io/)并从依赖项搜索器中选择 `Web`和 `Spring Cloud Sleuth` 启动器来缩短下面的步骤。这样做会生成一个新的项目结构，这样你就可以[立即开始编码](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#getting-started-first-application-code)。

#### 2.2.1. 创建 POM

我们需要先创建一个Maven pom.xml文件。pom.xml是用于构建项目的配置。打开你喜欢的文本编辑器，添加以下内容。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <!-- Use the latest compatible Spring Boot version. You can check https://spring.io/projects/spring-cloud for more information -->
        <version>2.4.6</version>
    </parent>

    <!-- Spring Cloud Sleuth requires a Spring Cloud BOM -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <!-- Provide the latest stable Spring Cloud release train version (e.g. 2020.0.0) -->
                <version>${release.train.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- (you don't need this if you are using a GA version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>https://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

前面的列表应该给你一个工作的构建。你可以通过运行`mvn package`来测试它（现在，你可以忽略 "“jar will be empty - no content was marked for inclusion!"的警告）。

> 这时，你可以把项目导入IDE（大多数现代Java IDE都包含对Maven的内置支持）。为简单起见，我们在本例中继续使用纯文本编辑器。

#### 2.2.2. Adding Classpath 依赖

要添加必要的依赖，编辑你的`pom.xml`并在`parent`部分下面添加`spring-boot-starter-web`依赖。

```xml
<dependencies>
    <!-- Boot's Web support -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Sleuth with Brave tracer implementation -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
</dependencies>
```

#### 2.2.3. 编写代码

为了完成我们的应用，我们需要创建一个单独的Java文件。默认情况下，Maven从`src/main/java`编译源代码，所以你需要创建该目录结构，然后添加一个名为`src/main/java/Example.java`的文件，包含以下代码。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    private static final Logger log = LoggerFactory.getLogger(Backend.class);

    @RequestMapping("/")
    String home() {
        log.info("Hello world!");
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Example.class, args);
    }

}
```

虽然这里的代码不多，但有相当多的事情要做。我们在接下来的几节中逐步介绍重要的部分。虽然这里的代码不多，但有相当多的事情要做。我们在接下来的几节中逐步介绍重要的部分。

**@RestController和@RequestMapping注解**

Spring Boot设置了Rest Controller，并使我们的应用程序绑定到Tomcat端口。带有Brave追踪器的Spring Cloud Sleuth将提供对传入请求的检测。

#### 2.2.4. 运行该实例

在这一点上，你的应用程序应该工作。由于你使用了`spring-boot-starter-parent`POM，你有一个有用的`run`目标，你可以用它来启动该应用程序。在项目根目录下输入`SPRING_APPLICATION_NAME=backend mvn spring-boot:run`来启动应用程序。你应该看到类似以下的输出。

```bash
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 ...
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果你打开网页浏览器到`localhost:8080`，你应该看到以下输出。

```text
Hello World!
```

如果你检查日志，你应该看到一个类似的输出

```text
2020-10-21 12:01:16.285  INFO [backend,0b6aaf642574edd3,0b6aaf642574edd3] 289589 --- [nio-9000-exec-1] Example              : Hello world!
```

你可以注意到，日志格式已经更新，有以下信息`[backend,0b6aaf642574edd3,0b6aaf642574edd3`。这个条目对应于`[application name,trace id, span id]`。应用程序名称从`SPRING_APPLICATION_NAME`环境变量中读取。

> 你可以设置`logging.level.org.springframework.web.servlet.DispatcherServlet=DEBUG`，而不是在处理程序中明确记录请求。

要优雅地退出应用程序，按`ctrl-c`。

### 2.3. 接下来的步骤

希望本节提供了一些Spring Cloud Sleuth的基础知识，让你开始编写自己的应用程序。如果你是一个面向任务的开发者，你可能想跳到[spring.io](https://spring.io/)并查看一些[入门](https://spring.io/guides/)指南，这些指南解决了 "我如何用Spring做这个？"的具体问题。我们也有专门针对Spring Cloud Sleuth的"[how-to](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#howto)"参考文档。

否则，下一个合乎逻辑的步骤是阅读[使用Spring Cloud Sleuth](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#using)。如果你真的没有耐心，你也可以提前阅读[Spring Cloud Sleuth功能](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#project-features)。

你可以在[samples](https://github.com/spring-cloud/spring-cloud-sleuth/tree/%7Bbranch%7D/spring-cloud-sleuth-samples)找到默认项目的样本。

## 3. 使用 Spring Cloud Sleuth

本节更详细地介绍了应该如何使用Spring Cloud Sleuth。它涵盖了一些主题，如用Spring Cloud Sleuth API或通过注解控制跨度生命周期。我们还介绍了一些Spring Cloud Sleuth的最佳实践。

如果你开始使用Spring Cloud Sleuth，在进入本节之前，你可能应该先阅读[入门](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#getting-started)指南。

### 3.1. Span生的命周期和Spring Cloud Sleuth的API

Spring Cloud Sleuth Core在其 `api` 模块中包含了所有需要由追踪器实现的必要接口。该项目带有OpenZipkin Brave实现。你可以通过查看`org.springframework.cloud.sleuth.brave.bridge`来检查追踪器是如何与Sleuth的API桥接的。

最常用的接口是。

- `org.springframework.cloud.sleuth.Tracer` - 使用追踪器，你可以创建一个根跨度，捕捉请求的关键路径。
- `org.springframework.cloud.sleuth.Span` - Span是一个需要启动和停止的单一工作单元。包含时间信息和事件及标签。

你也可以直接使用你的追踪器实现的API。

让我们来看看下面的Span生命周期动作。

- [start](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#using-creating-and-ending-spans)：当你开始一个跨度时，它的名字被指定，开始的时间戳被记录。
- [end](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#using-creating-and-ending-spans):：跨度结束（跨度的结束时间被记录下来），如果跨度被取样，它就有资格被收集（例如，到Zipkin）。
- [continue](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#using-continuing-spans)：该跨度被继续，例如在另一个线程中。
- [create with explicit parent](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#using-creating-spans-with-explicit-parent)：你可以创建一个新的跨度，并为它设置一个明确的父节点。

> Spring Cloud Sleuth为你创建一个`Tracer`的实例。为了使用它，你可以自动连接它。

#### 3.1.1. Creating and Ending Spans

你可以通过使用Tracer手动创建Span，如下面的例子所示。

```java
// 开始一个Span。如果这个线程中存在一个跨度，它将成为
// `newSpan`的父节点。
Span newSpan = this.tracer.nextSpan().name("calculateTax");
try (Tracer.SpanInScope ws = this.tracer.withSpan(newSpan.start())) {
    // ...
    // 你可以标记一个Span
    newSpan.tag("taxValue", taxValue);
    // ...
    // 你可以在一个`span`上记录一个事件
    newSpan.event("taxCalculated");
}
finally {
    // 一旦完成，记得要结束`span'。这将允许收集
    // `span`，将其发送到分布式跟踪系统，例如Zipkin。
    newSpan.end();
}
```

在前面的例子中，我们可以看到如何创建一个新的span的实例。如果这个线程中已经有一个span，它就会成为新span的父级。

> 创建`Span` 后一定要记得 `clean`

> 如果你的`span`包含一个大于50个字符的名字，这个名字会被截断为50个字符。你的名字必须是明确和具体的。大的名字会导致延迟问题，有时甚至会出现异常。

#### 3.1.2. Continuing Spans

有时，你不想创建一个新的`span`，但你想继续一个。这种情况的一个例子可能如下。

- **AOP**: 如果在达到一个`aspect`之前已经有一个`span`被创建，你可能不想创建一个新的`span`。

为了继续一个 `span`，你可以在一个线程中存储 `span`，并将其传递给另一个线程，如下例所示。

```java
Span spanFromThreadX = this.tracer.nextSpan().name("calculateTax");
try (Tracer.SpanInScope ws = this.tracer.withSpan(spanFromThreadX.start())) {
    executorService.submit(() -> {
        // Pass the span from thread X
        Span continuedSpan = spanFromThreadX;
        // ...
        // You can tag a span
        continuedSpan.tag("taxValue", taxValue);
        // ...
        // You can log an event on a span
        continuedSpan.event("taxCalculated");
    }).get();
}
finally {
    spanFromThreadX.end();
}
```

#### 3.1.3. Creating a Span with an explicit Parent

你可能想启动一个新的 "span"，并为该span提供一个明确的父节点。假设一个跨度的父级在一个线程中，而你想在另一个线程中启动一个新的跨度。每当你调用`Tracer.nextSpan()`时，它就会创建一个参考当前范围内的span的span。你可以把span放在作用域中，然后调用`Tracer.nextSpan()`，如下面的例子所示。

```java
// let's assume that we're in a thread Y and we've received
// the `initialSpan` from thread X. `initialSpan` will be the parent
// of the `newSpan`
Span newSpan = null;
try (Tracer.SpanInScope ws = this.tracer.withSpan(initialSpan)) {
    newSpan = this.tracer.nextSpan().name("calculateCommission");
    // ...
    // You can tag a span
    newSpan.tag("commissionValue", commissionValue);
    // ...
    // You can log an event on a span
    newSpan.event("commissionCalculated");
}
finally {
    // Once done remember to end the span. This will allow collecting
    // the span to send it to e.g. Zipkin. The tags and events set on the
    // newSpan will not be present on the parent
    if (newSpan != null) {
        newSpan.end();
    }
}
```

> 创建这样的`span`后，你必须完成它。否则，它不会被报告（例如向Zipkin）。

你也可以使用`Tracer.nextSpan(Span parentSpan)`版本来明确提供父级`span`。

### 3.2. Spans 命名

挑选一个`span`名称不是一件简单的事情。一个`span`名称应该描述一个操作名称。这个名字应该是低cardinality，所以它不应该包括标识符。

因为有很多工具正在进行，所以一些跨度名称是人为的。

- `controller-method-name`当被一个方法名为`controllerMethodName`的Controller接收时。
- `async`用于用包装好的`Callable`和`Runnable`接口进行异步操作。
- 用`@Scheduled`注解的方法会返回该类的简单名称。

幸运的是，对于异步处理，你可以提供明确的命名。

#### 3.2.1. @SpanName Annotation

你可以通过使用`@SpanName`注解来明确地命名`span`，如下例所示。

```java
@SpanName("calculateTax")
class TaxCountingRunnable implements Runnable {

    @Override
    public void run() {
        // perform logic
    }

}
```

在这种情况下，当以下列方式处理时，跨度被命名为 `calculateTax`.

```java
Runnable runnable = new TraceRunnable(this.tracer, spanNamer, new TaxCountingRunnable());
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

#### 3.2.2. toString() Method

为 `Runnable` 或 `Callable` 创建单独的类是很罕见的。通常情况下，人们会创建这些类的匿名实例。你不能对这样的类进行注释。为了克服这个限制，如果没有 `@SpanName` 注释，我们会检查该类是否有 `toString()` 方法的自定义实现。

运行这样的代码会导致创建一个名为 `calculateTax` 的span，如下面的例子所示。

```java
Runnable runnable = new TraceRunnable(this.tracer, spanNamer, new Runnable() {
    @Override
    public void run() {
        // perform logic
    }

    @Override
    public String toString() {
        return "calculateTax";
    }
});
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

### 3.3. 用注解管理Span

用注解来管理 `Span` 有很多好的理由，包括。

- 与跨度协作的API无关的手段。使用注解可以让用户在添加跨度时不依赖跨度api的库。这样做可以让Sleuth改变其核心API，对用户代码产生更少的影响。
- 减少了基本跨度操作的表面积。如果没有这个功能，你必须使用span api，它的生命周期命令可能被错误地使用。通过只暴露范围、标签和日志功能，你可以在不意外地破坏span生命周期的情况下进行协作。
- 与运行时生成的代码进行协作。通过Spring Data和Feign这样的库，接口的实现是在运行时生成的。因此，对对象进行跨度包装是很乏味的。现在你可以为接口和这些接口的参数提供注解。

#### 3.3.1. 创建新的Span

如果你不想手动创建本地`Span`，你可以使用`@NewSpan`注解。另外，我们还提供了`@SpanTag`注解来自动添加标签。

现在我们可以考虑一些使用的例子。

```java
@NewSpan
void testMethod();
```

在没有任何参数的情况下注解方法，会导致创建一个新的跨度，其名称等于注解的方法名称。

```java
@NewSpan("customNameOnTestMethod4")
void testMethod4();
```

如果你在注解中提供值（直接或通过设置`name`参数），创建的跨度将以提供的值作为名称。

```java
// method declaration
@NewSpan(name = "customNameOnTestMethod5")
void testMethod5(@SpanTag("testTag") String param);

// and method execution
this.testBean.testMethod5("test");
```

你可以把名字和标签都结合起来。让我们关注一下后者。在这种情况下，被注解的方法的参数运行时的值成为标签的值。在我们的例子中，标签键是`testTag`，标签值是`test`。

```java
@NewSpan(name = "customNameOnTestMethod3")
@Override
public void testMethod3() {
}
```

你可以把`@NewSpan`注解放在类和接口上。如果你覆盖了接口的方法，并为`@NewSpan`注解提供了不同的值，那么最具体的那一个会获胜（在这个例子中，`customNameOnTestMethod3`被设置）。

#### 3.3.2. Continuing Spans

如果你想给现有的跨度添加标签和注释，你可以使用`@ContinueSpan`注释，如下例所示。

```java
// method declaration
@ContinueSpan(log = "testMethod11")
void testMethod11(@SpanTag("testTag11") String param);

// method execution
this.testBean.testMethod11("test");
this.testBean.testMethod13();
```

(注意，与`@NewSpan`注释相反，你也可以用`log`参数添加日志)。

这样一来，跨度就会得到延续，并且。

- 创建名为`testMethod11.before`和`testMethod11.after`的日志条目。
- 如果一个异常被抛出，也会创建一个名为`testMethod11.afterFailure`的日志条目。
- 创建一个键值为`testTag11`、值为`test`的标签。

#### 3.3.3. Advanced Tag Setting

有3种不同的方式来添加标签到span中。所有这些都是由`SpanTag`注释控制的。其优先级如下。

1. 尝试使用`TagValueResolver`类型的bean和提供的名称。
2. 如果没有提供bean的名称，则尝试评估一个表达式。我们搜索一个`TagValueExpressionResolver`豆。默认实现使用SPEL表达式解析。**重要的是**你只能从SPEL表达式中引用属性。由于安全限制，不允许执行方法。
3. 如果我们没有找到任何可以评估的表达式，则返回参数的`toString()`值。

**自定义Extractor**

以下方法的标签值是由`TagValueResolver` 接口的实现计算出来的。它的类名必须作为 `resolver` 属性的值被传递。

考虑一下下面的注解方法。

```java
@NewSpan
public void getAnnotationForTagValueResolver(
        @SpanTag(key = "test", resolver = TagValueResolver.class) String test) {
}
```

现在进一步考虑以下`TagValueResolver` bean的实现。

```java
@Bean(name = "myCustomTagValueResolver")
public TagValueResolver tagValueResolver() {
    return parameter -> "Value from myCustomTagValueResolver";
}
```

前面的两个例子导致了设置一个等于来自`myCustomTagValueResolver`的标签值。

**解析一个值的表达式**

考虑以下注释的方法:

```java
@NewSpan
public void getAnnotationForTagValueExpression(
        @SpanTag(key = "test", expression = "'hello' + ' characters'") String test) {
}
```

没有自定义实现`TagValueExpressionResolver`就会导致SPEL表达式的评估，并且在span上设置一个值为`4个字符`的标签。如果你想使用一些其他的表达式解析机制，你可以创建你自己的bean实现。

**使用 The toString() 方法**

考虑以下注释的方法。

```java
@NewSpan
public void getAnnotationForArgumentToString(@SpanTag("test") Long param) {
}
```

以`15`的值运行前面的方法，会导致设置一个字符串值为`"15"`的标签。

### 3.4. 接下来要读什么

现在你应该明白如何使用Spring Cloud Sleuth以及应该遵循的一些最佳实践。现在你可以继续了解具体的[Spring Cloud Sleuth功能](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#project-features)，或者你可以跳过前面的内容，阅读[Spring Cloud Sleuth中可用的集成](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/integrations)。

## 4. Spring Cloud Sleuth的特性

本节深入探讨了Spring Cloud Sleuth的细节。在这里，你可以了解到你可能想要使用和定制的关键功能。如果你还没有这样做，你可能需要阅读"[入门](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#getting-started)"和"[使用Spring Cloud Sleuth](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#using)"部分，以便你有一个良好的基础知识。

### 4.1. Context 传播

追踪使用标头传播从服务连接到服务。默认格式是[B3](https://github.com/openzipkin/b3-propagation)。与数据格式类似，你也可以配置替代的头格式，只要跟踪和跨度ID与B3兼容。最值得注意的是，这意味着跟踪ID和跨度ID是小写的十六进制，而不是UUID。除了跟踪标识符，其他属性（Baggage）也可以与请求一起传递。远程Baggage必须是预定义的，但在其他方面是灵活的。

要使用提供的默认值，你可以设置`spring.sleuth.propagation.type`属性。该值可以是一个列表，在这种情况下，你将传播更多的跟踪头信息。

对于Brave，我们支持`AWS`、`B3`、`W3C`的传播类型。

你可以在这个"[how to section](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#how-to-change-context-propagation) "中阅读更多关于如何提供自定义上下文传播的信息。

### 4.2. 采样

Spring Cloud Sleuth将抽样决策推给了追踪器的实现。然而，在有些情况下，你可以在运行时改变采样决定。

其中一种情况是跳过某些客户端的跨度报告。为了实现这一点，你可以在`spring.sleuth.web.client.skip-pattern`中设置要跳过的路径模式。另一个选择是提供你自己的自定义`org.springframework.cloud.sleuth.SamplerFunction<org.springframework.cloud.sleuth.http.HttpRequest>`实现并定义何时不应对给定的`HttpRequest`进行采样。

### 4.3. Baggage

分布式跟踪是通过传播服务内部和跨服务的字段来进行的，这些字段将跟踪连接在一起：traceId和spanId值得注意。持有这些字段的上下文可以选择推送其他字段，这些字段需要保持一致，无论触及的服务有多少。这些额外字段的简单名称是 "Baggage"。

Sleuth允许你定义哪些Baggage允许存在于跟踪上下文中，包括使用什么样的头名称。

下面的例子展示了使用Spring Cloud Sleuth的API设置Baggage值。

```java
try (Tracer.SpanInScope ws = this.tracer.withSpan(initialSpan)) {
    BaggageInScope businessProcess = this.tracer.createBaggage(BUSINESS_PROCESS).set("ALM");
    BaggageInScope countryCode = this.tracer.createBaggage(COUNTRY_CODE).set("FO");
    try {
```

> 目前对Baggage项目的数量或大小没有限制。请记住，过多的Baggage会降低系统的吞吐量或增加RPC的延迟。在极端情况下，过多的Baggage会使应用程序崩溃，原因是超过了传输级的消息或标题的容量。

你可以使用属性来定义没有特殊配置的字段，如名称映射。

- `spring.sleuth.baggage.remote-fields`是一个接受并传播给远程服务的头名称的列表。
- `spring.sleuth.baggage.local-fields`是一个要在本地传播的名称列表。

这些键不适用前缀。你所设置的就是实际使用的。

在这两个属性中的任何一个中设置的名称都会导致一个相同名称的 "Baggage"。

为了自动将Baggage值设置为Slf4j的MDC，你必须用一个允许的本地或远程键的列表来设置`spring.sleuth.baggage.correlation-fields`属性。例如，`spring.sleuth.baggage.correlation-fields=country-code`将把`country-code`包袱的值设置到MDC。

注意，额外的字段会从下一个下游跟踪上下文开始传播并添加到 MDC 中。要在当前跟踪上下文中立即将额外字段添加到 MDC 中，请将该字段配置为更新时刷新。

```java
// configuration
@Bean
BaggageField countryCodeField() {
    return BaggageField.create("country-code");
}

@Bean
ScopeDecorator mdcScopeDecorator() {
    return MDCScopeDecorator.newBuilder()
            .clear()
            .add(SingleCorrelationField.newBuilder(countryCodeField())
                    .flushOnUpdate()
                    .build())
            .build();
}

// service
@Autowired
BaggageField countryCodeField;

countryCodeField.updateValue("new-value");
```

> 请记住，向MDC添加条目会大大降低你的应用程序的性能!

如果你想把Baggage条目作为标签加入，以便能够通过Baggage条目搜索跨度，你可以用允许的行李键列表设置`spring.sleuth.baggage.tag-fields`的值。要禁用该功能，你必须通过`spring.sleuth.propagation.tag.enabled=false`属性。

#### 4.3.1 Baggage versus Tags

与跟踪ID一样，Baggage被附加到消息或请求中，通常作为头文件。标签是以Span的形式发送到Zipkin的键值对。Baggage值默认不加入span，这意味着除非你选择加入，否则你不能根据Baggage进行搜索。

要使Baggage也成为标签，请使用属性`spring.sleuth.baggage.tag-fields`，像这样。

```yaml
spring:
  sleuth:
    baggage:
      foo: bar
      remoteFields:
        - country-code
        - x-vcap-request-id
      tagFields:
        - country-code
```

### 4.4. OpenZipkin Brave Tracer集成

Spring Cloud Sleuth通过 `spring-cloud-sleuth-brave`模块中的bridge与OpenZipkin Brave追踪器集成。在本节中，你可以了解到具体的Brave集成。

你可以选择在你的代码中直接使用Sleuth的API或Brave的API（例如，Sleuth的`Tracer`或Brave的`Tracer`）。如果你想直接使用这个追踪器实现的API，请阅读[他们的文档以了解更多信息](https://github.com/openzipkin/brave)。

#### 4.4.1. Brave Basics

以下是你可能使用的最核心的类型。

- `brave.SpanCustomizer` - 改变当前正在进行的span。
- `brave.Tracer` - 开始临时性的新span。

以下是OpenZipkin Brave项目中最相关的链接。

- [Brave’s core library](https://github.com/openzipkin/brave/tree/master/brave)
- [Baggage (propagated fields)](https://github.com/openzipkin/brave/tree/master/brave#baggage)
- [HTTP tracing](https://github.com/openzipkin/brave/tree/master/instrumentation/http)

#### 4.4.2. Brave Sampling

采样只适用于追踪后端，如Zipkin。无论采样率如何，日志中都会出现跟踪ID。采样是一种防止系统过载的方法，即持续追踪一些但不是所有的请求。

默认速率为每秒10条，由spring.sleuth.sampler.rate属性控制，当我们知道Sleuth是用于记录以外的原因时，就会适用。使用每秒100条以上的采样率时要特别小心，因为它可能会使你的跟踪系统超载。

采样器也可以通过Java配置来设置，如下面的例子所示。

```java
@Bean
public Sampler defaultSampler() {
    return Sampler.ALWAYS_SAMPLE;
}
```

> 你可以把HTTP标头b3设置为1，或者在进行信息传递时，你可以把spanFlags标头设置为1。这样做可以强制对当前请求进行采样，而不考虑配置。

默认情况下，采样器将与刷新范围机制一起工作。这意味着你可以在运行时改变采样属性，刷新应用程序，这些变化将得到反映。然而，有时围绕采样器创建一个代理并过早地调用它（从`@PostConstruct`注释的方法）可能会导致死锁。在这种情况下，要么明确地创建一个采样器bean，要么将属性`spring.sleuth.sampler.refresh.enabled`设置为`false`以禁用刷新范围支持。

#### 4.4.3. Brave Baggage Java configuration

如果你需要做比上面更高级的事情，请不要定义属性，而是为你使用的包袱字段使用`@Bean`配置。

- `BaggagePropagationCustomizer`设置baggage字段
- 添加一个`SingleBaggageField`来控制一个`Baggage`的标题名称。
- `CorrelationScopeCustomizer`设置了MDC字段
- 添加一个`SingleCorrelationField`来改变一个`baggage`的MDC名称，或者如果更新刷新。

#### 4.4.4. Brave Customizations

`brave.Tracer`对象是由sleuth完全管理的，所以你很少需要影响它。也就是说，Sleuth支持一些`Customizer`类型，允许你配置任何Sleuth尚未用自动配置或属性完成的东西。

如果你把下面的一个定义为 "Bean"，Sleuth将调用它来定制行为。

- `RpcTracingCustomizer` - 用于RPC标记和采样策略
- `HttpTracingCustomizer` - 用于HTTP标记和采样策略
- `MessagingTracingCustomizer` - 用于消息标记和取样策略
- `CurrentTraceContextCustomizer` - 集成装饰器，如关联性。
- `BaggagePropagationCustomizer` - 用于在进程中和在标题上传播包袱字段
- `CorrelationScopeDecoratorCustomizer` - 用于范围装饰，如MDC（日志）字段的相关。

**Brave Sampling Customizations**

如果需要客户端/服务器采样，只需注册一个`brave.sampler.SamplerFunction<HttpRequest>`类型的Bean，并将该Bean命名为`sleuthHttpClientSampler`用于客户端采样，`sleuthHttpServerSampler`用于服务器采样。

为了方便起见，`@HttpClientSampler`和`@HttpServerSampler`注解可以用来注入适当的Bean或通过其静态字符串`NAME`字段来引用Bean名称。

查看Brave的代码，看看如何制作一个基于路径的采样器的例子[github.com/openzipkin/brave/tree/master/instrumentation/http#sampling-policy](https://github.com/openzipkin/brave/tree/master/instrumentation/http#sampling-policy)

如果你想完全重写`HttpTracing`bean，你可以使用`SkipPatternProvider`接口来检索那些不应该被采样的跨度的URL`Pattern`。下面你可以看到一个在服务器端使用`SkipPatternProvider`的例子，`Sampler<HttpRequest>`。

```java
@Configuration(proxyBeanMethods = false)
    class Config {
  @Bean(name = HttpServerSampler.NAME)
  SamplerFunction<HttpRequest> myHttpSampler(SkipPatternProvider provider) {
      Pattern pattern = provider.skipPattern();
      return request -> {
          String url = request.path();
          boolean shouldSkip = pattern.matcher(url).matches();
          if (shouldSkip) {
              return false;
          }
          return null;
      };
  }
}
```

#### 4.4.5. Brave Messaging

Sleuth自动配置了`MessagingTracing`bean，作为Kafka或JMS等消息传递工具的基础。

如果需要定制生产者/消费者的消息跟踪采样，只需注册一个`brave.sampler.SamplerFunction<MessagingRequest>`类型的bean，并为生产者采样的bean命名为`sleuthProducerSampler`，为消费者采样命名为`sleuthConsumerSampler`。

为了方便起见，`@ProducerSampler`和`@ConsumerSampler`注解可用于注入适当的Bean或通过其静态字符串`NAME`字段引用Bean名称。

例如。这是一个每秒追踪100个消费者请求的采样器，除了 "alerts"通道。其他请求将使用由`Tracing`组件提供的全局速率。

```java
@Configuration(proxyBeanMethods = false)
    class Config {
  @Bean(name = ConsumerSampler.NAME)
  SamplerFunction<MessagingRequest> myMessagingSampler() {
      return MessagingRuleSampler.newBuilder().putRule(channelNameEquals("alerts"), Sampler.NEVER_SAMPLE)
              .putRule(Matchers.alwaysMatch(), RateLimitingSampler.create(100)).build();
  }
}
```

更多内容请见[github.com/openzipkin/brave/tree/master/instrumentation/messaging#sampling-policy](https://github.com/openzipkin/brave/tree/master/instrumentation/messaging#sampling-policy)

#### 4.4.6. Brave Opentracing

你可以通过`io.opentracing.brave:brave-opentracing` bridge 与Brave和[OpenTracing](https://opentracing.io/)集成。只要把它添加到classpath中，OpenTracing的`Tracer`就会被自动设置。

### 4.5. 向Zipkin发送Span

Spring Cloud Sleuth提供了与[OpenZipkin](https://zipkin.io/)分布式跟踪系统的各种集成。无论选择何种追踪器实现，只需在classpath中添加`spring-cloud-sleuth-zipkin`，就可以开始向Zipkin发送Span。你可以选择通过HTTP或消息传递来实现。你可以在"[how to section](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#how-to-set-up-sleuth-with-brave-zipkin-messaging) "中阅读更多关于如何做到这一点的信息。

当span被关闭时，它将通过HTTP发送给Zipkin。该通信是异步的。你可以通过设置`spring.zipkin.baseUrl`属性来配置URL，如下所示。

```yaml
spring.zipkin.baseUrl: https://192.168.99.100:9411/
```

如果想通过服务发现找到Zipkin，可以在URL内传递Zipkin的服务ID，如下例所示为`zipkinserver`服务ID。

```yaml
spring.zipkin.baseUrl: https://zipkinserver/
```

要禁用该功能，只需将`spring.zipkin.discovery-client-enabled`设置为`false`。

当发现客户端功能启用时，Sleuth使用`LoadBalancerClient`来查找Zipkin服务器的URL。这意味着你可以设置负载平衡的配置。

如果你在classpath上有`web`、`rabbit`、`activemq`或`kafka`在一起，你可能需要挑选你想发送spans到zipkin的方式。为此，将`web`、`rabbit`、`activemq`或`kafka`设置为`spring.zipkin.sender.type`属性。下面的例子显示了设置`web`的sender类型。

```yaml
spring.zipkin.sender.type: web
```

要定制通过HTTP向Zipkin发送span的`RestTemplate`，可以注册`ZipkinRestTemplateCustomizer` bean。

```java
@Configuration(proxyBeanMethods = false)
    class MyConfig {
    @Bean ZipkinRestTemplateCustomizer myCustomizer() {
        return new ZipkinRestTemplateCustomizer() {
            @Override
            void customize(RestTemplate restTemplate) {
                // customize the RestTemplate
            }
        };
    }
}
```

然而，如果你想控制创建`RestTemplate`对象的整个过程，你必须创建一个`zipkin2.reporter.Sender`类型的bean。

```java
@Bean Sender myRestTemplateSender(ZipkinProperties zipkin,
        ZipkinRestTemplateCustomizer zipkinRestTemplateCustomizer) {
    RestTemplate restTemplate = mySuperCustomRestTemplate();
    zipkinRestTemplateCustomizer.customize(restTemplate);
    return myCustomSender(zipkin, restTemplate);
}
```

默认情况下，api路径将被设置为`api/v2/spans`或`api/v1/spans`，取决于编码器版本。如果你想使用自定义的api路径，你可以使用以下属性进行配置（空的情况下，设置""）。

```yaml
spring.zipkin.api-path: v2/path2
```

#### 4.5.1. 自定义 service 名称

默认情况下，Sleuth假定，当你向Zipkin发送span时，你希望span的服务名称等于`spring.application.name`属性的值。不过，情况并非总是如此。在有些情况下，你想为来自你的应用程序的所有span明确提供一个不同的服务名称。为了达到这个目的，你可以向你的应用程序传递以下属性来覆盖这个值（这个例子是针对一个名为`myService`的服务）。

```yaml
spring.zipkin.service.name: myService
```

#### 4.5.2. Host Locator

> 本节是关于从服务发现中定义主机。它不是关于通过服务发现找到Zipkin。

为了定义对应于一个特定`span'的主机，我们需要解决主机名和端口。默认的方法是从服务器属性中获取这些值。如果这些没有设置，我们尝试从网络接口检索主机名。

如果你启用了发现客户端，并且喜欢从服务注册表中的注册实例检索主机地址，你必须设置`spring.zipkin.locator.discovery.enabled`属性（它适用于基于HTTP和基于流的跨度报告），如下所示。

```yaml
spring.zipkin.locator.discovery.enabled: true
```

#### 4.5.3. Customization of Reported Spans

在Sleuth中，我们用一个固定的名字来生成Span。有些用户希望根据标签的值来修改名称。

Sleuth注册了一个`SpanFilter`bean，可以自动跳过报告给定名称模式的Span。属性`spring.sleuth.span-filter.span-name-patterns-to-skip`包含span名称的默认跳过模式。属性`spring.sleuth.span-filter.additional-span-name-patterns-to-skip`将把提供的Span名称模式追加到现有的模式中。为了禁用这个功能，只需将`spring.sleuth.span-filter.enabled`设为`false`。

**Brave Customization of Reported Spans**

> 本节仅适用于Brave追踪器。

在报告Span之前（例如，向Zipkin），你可能想以某种方式修改该Span。你可以通过实现 "SpanHandler"来实现。

下面的例子显示了如何注册两个实现`SpanHandler`的bean。

```java
@Bean
SpanHandler handlerOne() {
    return new SpanHandler() {
        @Override
        public boolean end(TraceContext traceContext, MutableSpan span, Cause cause) {
            span.name("foo");
            return true; // keep this span
        }
    };
}

@Bean
SpanHandler handlerTwo() {
    return new SpanHandler() {
        @Override
        public boolean end(TraceContext traceContext, MutableSpan span, Cause cause) {
            span.name(span.name() + " bar");
            return true; // keep this span
        }
    };
}
```

前面的例子导致报告的span的名称改为`foo bar`，就在它被报告之前（例如，给Zipkin）。

#### 4.5.4. 覆盖Zipkin的自动配置

从2.1.0版本开始，Spring Cloud Sleuth支持向多个追踪系统发送追踪信息。为了使其发挥作用，每个追踪系统都需要有一个`Reporter<Span>`和`Sender`。如果你想覆盖所提供的bean，你需要给它们一个特定的名字。要做到这一点，你可以分别使用`ZipkinAutoConfiguration.Reporter_BEAN_NAME`和`ZipkinAutoConfiguration.SENDER_BEAN_NAME`。

```java
@Configuration(proxyBeanMethods = false)
protected static class MyConfig {

    @Bean(ZipkinAutoConfiguration.REPORTER_BEAN_NAME)
    Reporter<zipkin2.Span> myReporter(@Qualifier(ZipkinAutoConfiguration.SENDER_BEAN_NAME) MySender mySender) {
        return AsyncReporter.create(mySender);
    }

    @Bean(ZipkinAutoConfiguration.SENDER_BEAN_NAME)
    MySender mySender() {
        return new MySender();
    }

    static class MySender extends Sender {

        private boolean spanSent = false;

        boolean isSpanSent() {
            return this.spanSent;
        }

        @Override
        public Encoding encoding() {
            return Encoding.JSON;
        }

        @Override
        public int messageMaxBytes() {
            return Integer.MAX_VALUE;
        }

        @Override
        public int messageSizeInBytes(List<byte[]> encodedSpans) {
            return encoding().listSizeInBytes(encodedSpans);
        }

        @Override
        public Call<Void> sendSpans(List<byte[]> encodedSpans) {
            this.spanSent = true;
            return Call.create(null);
        }

    }

}
```

### 4.6. 日志整合

Sleuth用包括服务名称（`%{spring.zipkin.service.name}`或`%{spring.application.name}`，如果之前没有设置）、span ID（`%{spanId}`）和trace ID（`%{traceId}`）等变量配置日志上下文。这些有助于你将日志与分布式跟踪联系起来，并允许你选择使用什么工具来排除服务的故障。

一旦你发现任何有错误的日志，你可以在消息中寻找跟踪ID。把它粘贴到你的分布式跟踪系统中，以可视化整个跟踪，不管第一个请求最终击中了多少个服务。

```log
backend.log:  2020-04-09 17:45:40.516 ERROR [backend,5e8eeec48b08e26882aba313eb08f0a4,dcc1df555b5777b3] 97203 --- [nio-9000-exec-1] o.s.c.s.i.web.ExceptionLoggingFilter     : Uncaught exception thrown
frontend.log:2020-04-09 17:45:40.574 ERROR [frontend,5e8eeec48b08e26882aba313eb08f0a4,82aba313eb08f0a4] 97192 --- [nio-8081-exec-2] o.s.c.s.i.web.ExceptionLoggingFilter     : Uncaught exception thrown
```

上面，你会注意到跟踪ID是`5e8eeec48b08e26882aba313eb08f0a4` ，例如。这个日志配置是由Sleuth自动设置的。你可以通过`spring.sleuth.enabled=false`属性禁用Sleuth，或者放入你自己的`logging.pattern.level`属性来禁用它。

如果你使用一个日志聚合工具（如[Kibana](https://www.elastic.co/products/kibana), [Splunk](https://www.splunk.com/), 和其他），你可以对发生的事件进行排序。来自Kibana的一个例子类似于下面的图片。

![ ](https://cdn.jsdelivr.net/gh/KevinBlandy/springcloud-images/2021/07/25/3c975295b96f4b65ad8911fd52f2740f.png)

如果你想使用[Logstash](https://www.elastic.co/guide/en/logstash/current/index.html)，下面列出了Logstash的Grok模式。

```text
filter {
  # pattern matching logback pattern
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
  }
  date {
    match => ["timestamp", "ISO8601"]
  }
  mutate {
    remove_field => ["timestamp"]
  }
}
```

> 如果您想将 Grok 与 Cloud Foundry 的日志一起使用，您必须使用以下模式。

```text
filter {
  # pattern matching logback pattern
  grok {
    match => { "message" => "(?m)OUT\s+%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
  }
  date {
    match => ["timestamp", "ISO8601"]
  }
  mutate {
    remove_field => ["timestamp"]
  }
}
```

#### 4.6.1. 使用Logstash的JSON Logback

通常情况下，你不希望将你的日志存储在文本文件中，而是存储在Logstash可以立即提取的JSON文件中。要做到这一点，你必须做到以下几点（为了便于阅读，我们用`groupId:artifactId:version`的符号来传递依赖关系）。

**Dependencies Setup**

1. 确保 Logback 在 classpath 上 ( `ch.qos.logback:logback-core` ) 。
2. 2.添加 Logstash Logback 编码。例如，要使用`4.6`版本，添加`net.logstash.logback:logstash-logback-encoder:4.6`。

**Logback设置**

考虑一下下面这个Logback配置文件的例子（logback-spring.xml）。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>
    <!-- Example for logging into the build folder of your project -->
    <property name="LOG_FILE" value="${BUILD_FOLDER:-build}/${springAppName}"/>

    <!-- You can override this to have a custom pattern -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>

    <!-- Appender to log to console -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <!-- Minimum logging level to be presented in the console logs-->
            <level>DEBUG</level>
        </filter>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- Appender to log to file -->
    <appender name="flatfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>
    <!-- Appender to log to file in a JSON format -->
    <appender name="logstash" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.json.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "timestamp": "@timestamp",
                        "severity": "%level",
                        "service": "${springAppName:-}",
                        "trace": "%X{traceId:-}",
                        "span": "%X{spanId:-}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "rest": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="console"/>
        <!-- uncomment this to have also JSON logs -->
        <!--<appender-ref ref="logstash"/>-->
        <!--<appender-ref ref="flatfile"/>-->
    </root>
</configuration>
```

即Logback配置文件。

- 将应用程序的信息以JSON格式记录到`build/${spring.application.name}.json`文件。
- 已经注释了两个额外的应用者：控制台和标准日志文件。
- 具有与上一节中介绍的相同的日志模式。

> 如果你使用一个自定义的`logback-spring.xml`，你必须在`bootstrap`中传递`spring.application.name`，而不是`application`属性文件。否则，你的自定义logback文件就不能正确读取该属性。

### 4.7. 接下来要读的内容

如果你想进一步了解本节讨论的任何一个类，你可以直接浏览[源代码](https://github.com/spring-cloud/spring-cloud-sleuth/tree/main)。如果你有具体问题，请看[如何做](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#howto)部分。

如果你对Spring Cloud Sleuth的核心功能感到满意，你可以继续阅读[Spring Cloud Sleuth的集成](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#integrations.adoc)。

## 5. “How-to” Guides

本节为使用Spring Cloud Sleuth时经常出现的一些常见的 "如何做...... "问题提供答案。它的覆盖面并不全面，但确实涵盖了相当多的内容。

如果你有一个特定的问题，而我们在这里没有涉及，你可能想去看看[stackoverflow.com](https://stackoverflow.com/tags/spring-cloud-sleuth)，看看是否有人已经提供了答案。Stack Overflow也是一个提出新问题的好地方（请使用`spring-cloud-sleuth`标签）。

我们也非常乐意扩展这个部分。如果你想添加一个 "如何做"，请向我们发送一个[拉动请求](https://github.com/spring-cloud/spring-cloud-sleuth/tree/main)。

### 5.1. 如何用Brave设置Sleuth？

将Sleuth starter 添加到classpath。

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

### 5.2. 如何通过HTTP设置Sleuth与Brave & Zipkin？

将Sleuth启动器和Zipkin添加到classpath。

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

## 5.3. 如何通过信息传递将Sleuth与Brave & Zipkin设置在一起？

如果你想使用RabbitMQ、Kafka或ActiveMQ而不是HTTP，请添加`spring-rabbit`、`spring-kafka`或`org.apache.activemq:activemq-client`依赖项。默认的目标名称是`Zipkin`。

如果使用Kafka，你必须添加Kafka依赖性。

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

另外，你需要相应地设置`spring.zipkin.sender.type`属性。

```yaml
spring.zipkin.sender.type: kafka
```

如果你想在RabbitMQ上使用Sleuth，请添加`spring-cloud-starter-sleuth`、`spring-cloud-sleuth-zipkin`和`spring-rabbit`依赖项。

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
```

如果你想在ActiveMQ上使用Sleuth，请添加`spring-cloud-starter-sleuth`、`spring-cloud-sleuth-zipkin`和`activemq-client`依赖项。

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-client</artifactId>
</dependency>
```

另外，你需要相应地设置`spring.zipkin.sender.type`属性。

```yaml
spring.zipkin.sender.type: activemq
```

### 5.4. 如何在一个外部系统中看到Span？

如果你看不到Span被报告给外部系统（如Zipkin），那么很可能是由于以下原因。

- [你的Span没有被采样](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#not-sampled-span)
- [你忘记添加向外部系统报告的依赖关系（如`spring-cloud-sleuth-zipkin`）](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#missing-dependency)
- [你错误地配置了与外部系统的连接](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#connection-misconfiguration)

#### 5.4.1. 你的Span没有被采样

为了检查跨度是否没有被采样，只要看看可导出的标志是否被设置就可以了。让我们看一下下面的例子。

```log
2020-10-21 12:01:16.285  INFO [backend,0b6aaf642574edd3,0b6aaf642574edd3,true] 289589 --- [nio-9000-exec-1] Example              : Hello world!
```

如果`[backend,0b6aaf642574edd3,0b6aaf642574edd3,true]`部分的布尔值为 "true"，意味着该跨度正在被采样，应该被报告。

#### 5.4.2. 缺少依赖性

直到Sleuth 3.0.0，`spring-cloud-starter-zipkin`这个依赖项包括`spring-cloud-starter-sleuth`依赖项和`spring-cloud-sleuth-zipkin`依赖项。在3.0.0版本中，`spring-cloud-starter-zipkin`被移除，所以你需要把它改为`spring-cloud-sleuth-zipkin`。

#### 5.4.3. 连接错误配置

仔细检查远程系统地址是否正确（例如`spring.zipkin.baseUrl`），如果试图通过broker进行通信，你的broker连接设置正确。

### 5.5. 如何使RestTemplate、WebClient等。工作？

如果你观察到追踪上下文没有被传播，那么原因是以下之一。

- 我们没有对指定的库进行检测
- 我们正在检测该库，但是你错误地配置了设置。

如果缺乏检测能力，请提交[问题](https://github.com/spring-cloud/spring-cloud-sleuth/issues)，要求增加这种检测。

在配置错误的情况下，请确保你用来通信的客户端是一个Spring Bean。如果你通过 `new` 操作符手动创建客户端，那么仪表功能将不起作用。

工具化工作的例子。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration(proxyBeanMethods = false)
class MyConfiguration {
    @Bean RestTemplate myRestTemplate() {
        return new RestTemplate();
    }
}

@Service
class MyService {
    private final RestTemplate restTemplate;

    MyService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    String makeACall() {
        return this.restTemplate.getForObject("http://example.com", String.class);
    }

}
```

仪表盘不起作用的例子。

```java
@Service
class MyService {

    String makeACall() {
        // This will not work because RestTemplate is not a bean
        return new RestTemplate().getForObject("http://example.com", String.class);
    }

}
```

### 5.6. 如何在HTTP服务器响应中添加头信息？

注册一个`HttpResponseParser`类型的bean，其名称为`HttpServerResponseParser.NAME`。

```java
import org.springframework.cloud.sleuth.http.HttpResponseParser;
import org.springframework.cloud.sleuth.instrument.web.HttpServerResponseParser;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
class MyConfig {

    @Bean(name = HttpServerResponseParser.NAME)
    HttpResponseParser myHttpResponseParser() {
        return (response, context, span) -> {
            Object unwrap = response.unwrap();
            if (unwrap instanceof HttpServletResponse) {
                HttpServletResponse resp = (HttpServletResponse) unwrap;
                resp.addHeader("MyCustom", "Header");
            }
        };
    }

}
```

> 你的Span需要被取样，以便解析器工作。这意味着你需要能够将Span导出到例如Zipkin。

### 5.7. 如何自定义HTTP客户端的Span？

注册一个`HttpRequestParser`类型的Bean，其名称为`HttpClientRequestParser.NAME`，以便为请求方添加定制。注册一个`HttpResponseParser`类型的Bean，其名称为`HttpClientRequestParser.NAME`，以便为响应端添加定制功能。

```java
@Configuration(proxyBeanMethods = false)
public static class ClientParserConfiguration {

    // example for Feign
    @Bean(name = HttpClientRequestParser.NAME)
    HttpRequestParser myHttpClientRequestParser() {
        return (request, context, span) -> {
            // Span customization
            span.name(request.method());
            span.tag("ClientRequest", "Tag");
            Object unwrap = request.unwrap();
            if (unwrap instanceof feign.Request) {
                feign.Request req = (feign.Request) unwrap;
                // Span customization
                span.tag("ClientRequestFeign", req.httpMethod().name());
            }
        };
    }

    // example for Feign
    @Bean(name = HttpClientResponseParser.NAME)
    HttpResponseParser myHttpClientResponseParser() {
        return (response, context, span) -> {
            // Span customization
            span.tag("ClientResponse", "Tag");
            Object unwrap = response.unwrap();
            if (unwrap instanceof feign.Response) {
                feign.Response resp = (feign.Response) unwrap;
                // Span customization
                span.tag("ClientResponseFeign", String.valueOf(resp.status()));
            }
        };
    }

}
```

### 5.8. 如何自定义HTTP服务器的Span？

注册一个`HttpRequestParser`类型的Bean，其名称为`HttpServerRequestParser.NAME`，以便为请求方添加定制。注册一个`HttpResponseParser`类型的Bean，其名称为`HttpServerResponseParser.NAME`，以便为响应端添加定制功能。

```java
@Configuration(proxyBeanMethods = false)
public static class ServerParserConfiguration {

    @Bean(name = HttpServerRequestParser.NAME)
    HttpRequestParser myHttpRequestParser() {
        return (request, context, span) -> {
            // Span customization
            span.tag("ServerRequest", "Tag");
            Object unwrap = request.unwrap();
            if (unwrap instanceof HttpServletRequest) {
                HttpServletRequest req = (HttpServletRequest) unwrap;
                // Span customization
                span.tag("ServerRequestServlet", req.getMethod());
            }
        };
    }

    @Bean(name = HttpServerResponseParser.NAME)
    HttpResponseParser myHttpResponseParser() {
        return (response, context, span) -> {
            // Span customization
            span.tag("ServerResponse", "Tag");
            Object unwrap = response.unwrap();
            if (unwrap instanceof HttpServletResponse) {
                HttpServletResponse resp = (HttpServletResponse) unwrap;
                // Span customization
                span.tag("ServerResponseServlet", String.valueOf(resp.getStatus()));
            }
        };
    }

}
```

> 你的Span需要被取样，以便解析器工作。这意味着你需要能够将跨度导出到例如Zipkin。

### 5.9. 如何在日志中看到应用程序的名称？

假设你没有改变默认的日志格式，在`bootstrap.yml`中设置`spring.application.name`属性，而不是在`application.yml`。

> 有了新的Spring Cloud配置Bootstrap，这应该不再需要了，因为不再有Bootstrap Context。

### 5.10. How to Change The Context Propagation Mechanism?

要使用提供的默认值，你可以设置`spring.sleuth.propagation.type`属性。该值可以是一个列表，在这种情况下，你将传播更多的跟踪头信息。

对于Brave，我们支持`AWS`、`B3`、`W3C`的传播类型。

如果你想提供一个自定义的传播机制，将`spring.sleuth.propagation.type`属性设置为`CUSTOM`并实现你自己的bean（对于Brave来说是`Propagation.Factory`）。下面你可以找到这些例子。

```java
@Component
class CustomPropagator extends Propagation.Factory implements Propagation<String> {

    @Override
    public List<String> keys() {
        return Arrays.asList("myCustomTraceId", "myCustomSpanId");
    }

    @Override
    public <R> TraceContext.Injector<R> injector(Setter<R, String> setter) {
        return (traceContext, request) -> {
            setter.put(request, "myCustomTraceId", traceContext.traceIdString());
            setter.put(request, "myCustomSpanId", traceContext.spanIdString());
        };
    }

    @Override
    public <R> TraceContext.Extractor<R> extractor(Getter<R, String> getter) {
        return request -> TraceContextOrSamplingFlags.create(TraceContext.newBuilder()
                .traceId(HexCodec.lowerHexToUnsignedLong(getter.get(request, "myCustomTraceId")))
                .spanId(HexCodec.lowerHexToUnsignedLong(getter.get(request, "myCustomSpanId"))).build());
    }

    @Override
    public <K> Propagation<K> create(KeyFactory<K> keyFactory) {
        return StringPropagationAdapter.create(this, keyFactory);
    }

}
```

### 5.11. 如何实现我自己的追踪器？

Spring Cloud Sleuth API包含所有需要由追踪器实现的必要接口。该项目带有OpenZipkin Brave的实现。你可以通过查看`org.springframework.cloud.sleuth.brave.bridge`模块来检查这两个追踪器是如何与Sleuth的API连接的。

## 6. Spring Cloud Sleuth的定制

在本节中，我们将介绍如何定制Spring Cloud Sleuth的各个部分。

### 6.1. 异步通信

在本节中，我们将介绍如何用Spring Cloud Sleuth定制异步通信。

#### 6.1.1. @Async注解的方法

该功能适用于所有追踪器的实现。

在Spring Cloud Sleuth中，我们对异步相关的组件进行检测，以便在线程之间传递跟踪信息。你可以通过将`spring.sleuth.async.enabled`的值设置为`false`来禁用这种行为。

如果你用`@Async`来注释你的方法，我们会自动修改现有的Span，如下所示。

- 如果方法被注解为`@SpanName`，注解的值就是Span的名字。
- 如果该方法没有用`@SpanName`注释，那么Span的名字就是被注释的方法的名字。
- Span被标记为该方法的类名和方法名。

由于我们正在修改现有的span，如果你想保持它的原始名称（例如通过接收HTTP请求创建的span），你应该用`@Async`注解的方法包裹你的`@NewSpan`注解或者手动创建一个新的span。

#### 6.1.2. @Scheduled 注释的方法

该功能适用于所有追踪器的实现。

在Spring Cloud Sleuth中，我们对预定方法的执行进行了检测，这样追踪信息就会在线程之间传递。你可以通过将`spring.sleuth.schedulated.enabled`的值设置为`false`来禁用这种行为。

如果你用`@Scheduled`来注释你的方法，我们会自动创建一个新的跨度，具有以下特征。

- span的名字是被注释的方法的名字。
- span被标记为该方法的类名和方法名。

如果你想跳过对某些`@Scheduled`注释类的span创建，你可以将`spring.sleuth.schedulated.skipPattern`设置为正则表达式，与`@Scheduled`注释类的完全合格名称相匹配。

#### 6.1.3. Executor, ExecutorService, 和 ScheduledExecutorService

该功能适用于所有追踪器的实现。

我们提供`LazyTraceExecutor`，`TraceableExecutorService`，和`TraceableScheduledExecutorService`。这些实现在每次提交、调用或安排新任务时都会创建跨度。

下面的例子显示了在使用`CompletableFuture`时如何用`TraceableExecutorService`传递跟踪信息。

```java
CompletableFuture<Long> completableFuture = CompletableFuture.supplyAsync(() -> {
    // perform some logic
    return 1_000_000L;
}, new TraceableExecutorService(beanFactory, executorService,
        // 'calculateTax' explicitly names the span - this param is optional
        "calculateTax"));
```

> Sleuth不能与`parallelStream()`一起工作。如果你想让追踪信息通过流传播，你必须使用`supplyAsync(..)`的方法，如前面所示。

如果有一些实现了`Executor`接口的Bean被你排除在span创建之外，你可以使用`spring.sleuth.async.ignored-beans`属性，你可以提供一个Bean名称的列表。

你可以通过将`spring.sleuth.async.enabled`的值设置为`false`来禁用这种行为。

**自定义Executors**

有时，你需要设置一个自定义的 `AsyncExecutor` 实例。下面的例子展示了如何设置这样一个自定义的`Executor`。

```java
@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@EnableAsync
// add the infrastructure role to ensure that the bean gets auto-proxied
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public static class CustomExecutorConfig extends AsyncConfigurerSupport {

    @Autowired
    BeanFactory beanFactory;

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // CUSTOMIZE HERE
        executor.setCorePoolSize(7);
        executor.setMaxPoolSize(42);
        executor.setQueueCapacity(11);
        executor.setThreadNamePrefix("MyExecutor-");
        // DON'T FORGET TO INITIALIZE
        executor.initialize();
        return new LazyTraceExecutor(this.beanFactory, executor);
    }

}
```

> 为了确保你的配置得到后处理，记得在你的`@Configuration`类上添加`@Role(BeanDefinition.ROLE_INFRASTRUCTURE)`。

### 6.2. HTTP客户端集成

这一部分的功能可以通过设置`spring.sleuth.web.client.enabled`属性值为`false`来禁用。

#### 6.2.1. 同步 Rest Template

这个功能对所有追踪器的实现都是可用的。

我们注入一个`RestTemplate`拦截器，以确保所有的追踪信息被传递给请求。每次调用时，都会创建一个新的Span。它在收到响应时被关闭。为了阻止同步的`RestTemplate`功能，将`spring.sleuth.web.client.enabled`设置为`false`。

> 你必须把`RestTemplate`注册为一个Bean，这样拦截器才会被注入。如果你用 `new` 关键字创建了一个`RestTemplate`实例，仪表就不会起作用。

#### 6.2.2. 异步 Rest Template

这个功能对所有追踪器的实现都是可用的。

从Sleuth `2.0.0`开始，我们不再注册`AsyncRestTemplate`类型的bean。这取决于你是否能创建这样一个Bean。然后我们对它进行检测。

要阻止 `AsyncRestTemplate` 功能，将`spring.sleuth.web.async.client.enabled`设置为`false`。要禁止创建默认的`TraceAsyncClientHttpRequestFactoryWrapper`，请将`spring.sleuth.web.async.client.factory.enabled`设置为`false`。如果你根本不想创建`AsyncRestClient`，请将`spring.sleuth.web.async.client.template.enabled`设置为`false`。

**Multiple Asynchronous Rest Templates**

有时你需要使用异步Rest模板的多个实现。在下面的片段中，你可以看到一个例子，说明如何设置这样一个自定义的`AsyncRestTemplate`。

```java
@Configuration(proxyBeanMethods = false)
public static class TestConfig {

    @Bean(name = "customAsyncRestTemplate")
    public AsyncRestTemplate traceAsyncRestTemplate() {
        return new AsyncRestTemplate(asyncClientFactory(), clientHttpRequestFactory());
    }

    private ClientHttpRequestFactory clientHttpRequestFactory() {
        ClientHttpRequestFactory clientHttpRequestFactory = new CustomClientHttpRequestFactory();
        // CUSTOMIZE HERE
        return clientHttpRequestFactory;
    }

    private AsyncClientHttpRequestFactory asyncClientFactory() {
        AsyncClientHttpRequestFactory factory = new CustomAsyncClientHttpRequestFactory();
        // CUSTOMIZE HERE
        return factory;
    }

}
```

**WebClient**

这个功能对所有追踪器的实现都是可用的。

我们注入了一个`ExchangeFilterFunction`实现，它创建了一个跨度，并通过成功和错误的回调，负责关闭客户端的跨度。

要阻止这个功能，请将`spring.sleuth.web.client.enabled`设置为`false`。

你必须把`WebClient`注册为一个Bean，这样才能应用追踪工具。如果你用 `new` 关键字创建一个`WebClient`实例，追踪工具就不会工作。

**Traverson**

这个功能对所有追踪器的实现都是可用的。

如果你使用[Traverson](https://docs.spring.io/spring-hateoas/docs/current/reference/html/#client.traverson)库，你可以把`RestTemplate`作为一个bean注入你的Traverson对象。由于`RestTemplate`已经被拦截了，你可以在你的客户端获得对追踪的完全支持。下面的伪代码显示了如何做到这一点。

```java
@Autowired RestTemplate restTemplate;

Traverson traverson = new Traverson(URI.create("https://some/address"),
    MediaType.APPLICATION_JSON, MediaType.APPLICATION_JSON_UTF8).setRestOperations(restTemplate);
// use Traverson
```

**Apache HttpClientBuilder 和 HttpAsyncClientBuilder**

这个功能适用于Brave追踪器的实现。

我们对`HttpClientBuilder`和`HttpAsyncClientBuilder`进行检测，以便追踪上下文被注入到发送的请求中。

要阻止这些功能，请将`spring.sleuth.web.client.enabled`设置为`false`。

**Netty HttpClient**

这个功能对所有追踪器的实现都是可用的。

我们使用Netty的`HttpClient`。

要阻止这个功能，将`spring.sleuth.web.client.enabled`设置为`false`。

你必须把`HttpClient`注册为一个Bean，这样才会有仪器分析。如果你用 `new` 关键字创建一个`HttpClient`实例，仪器分析就不会发生。

**UserInfoRestTemplateCustomizer**

这个功能对所有追踪器的实现都是可用的。

我们使用Spring Security的`UserInfoRestTemplateCustomizer`。

要阻止这个功能，请将`spring.sleuth.web.client.enabled`设置为`false`。

### 6.3. HTTP服务器集成

这一部分的功能可以通过设置`spring.sleuth.web.enabled`属性的值等于`false`来禁用。

#### 6.3.1. HTTP Filter

这个功能适用于所有追踪器的实现。

通过`TracingFilter`，所有被采样的传入请求都会导致创建一个Span。你可以通过设置`spring.sleuth.web.skipPattern`属性来配置你想跳过的URI。如果你在classpath上有`ManagementServerProperties`，它的`contextPath`值会被附加到所提供的跳过模式上。如果你想重复使用Sleuth的默认跳过模式，只需添加你自己的模式，通过使用`spring.sleuth.web.extraSkipPattern`传递这些模式。

默认情况下，所有的spring boot执行器端点都被自动添加到跳过模式中。如果你想禁用这种行为，请将`spring.sleuth.web.ignore-auto-configured-skip-patterns`设为`true`。

要改变跟踪过滤器的注册顺序，请设置`spring.sleuth.web.filter-order`属性。

要禁用记录未捕获异常的过滤器，可以禁用`spring.sleuth.web.exception-throwing-filter-enabled`属性。

#### 6.3.2. HandlerInterceptor

这个功能适用于所有追踪器的实现。

由于我们希望span的名称是精确的，我们使用一个`TraceHandlerInterceptor`，它或者包裹一个现有的`HandlerInterceptor`，或者直接添加到现有的`HandlerInterceptors`列表中。`TraceHandlerInterceptor`为给定的`HttpServletRequest`添加一个特殊的请求属性。如果`TracingFilter`没有看到这个属性，它就会创建一个 "fallback" span，这是一个在服务器端创建的额外span，以便在UI中正确显示跟踪。如果发生这种情况，可能是缺少仪器。在这种情况下，请在Spring Cloud Sleuth中提交一个问题。

#### 6.3.3. Async Servlet support

这个功能适用于所有追踪器的实现。

如果你的控制器返回一个`Callable`或`WebAsyncTask`，Spring Cloud Sleuth会继续现有的跨度，而不是创建一个新的。

#### 6.3.4. WebFlux support

这个功能适用于所有追踪器的实现。

通过`TraceWebFilter`，所有被采样的传入请求都会产生一个Span。这个Span的名字是`http:` + 请求被发送到的路径。例如，如果请求被发送到`/this/that`，其名称就是`http:/this/that`。你可以通过使用`spring.sleuth.web.skipPattern`属性来配置你想跳过的URI。如果你在classpath上有`ManagementServerProperties`，它的`contextPath`值会被附加到所提供的跳过模式上。如果你想重复使用Sleuth的默认跳过模式，并附加你自己的模式，可以通过使用`spring.sleuth.web.extraSkipPattern`来传递这些模式。

为了在性能和上下文传播方面达到最佳效果，我们建议你将`spring.sleuth.reactor.instrumation-type`切换为`MANUAL`。为了在span的范围内执行代码，你可以调用`WebFluxSleuthOperators.withSpanInScope`。例子。

```java
@GetMapping("/simpleManual")
public Mono<String> simpleManual() {
    return Mono.just("hello").map(String::toUpperCase).doOnEach(WebFluxSleuthOperators
            .withSpanInScope(SignalType.ON_NEXT, signal -> log.info("Hello from simple [{}]", signal.get())));
}
```

要改变追踪过滤器的注册顺序，请设置`spring.sleuth.web.filter-order`属性。

### 6.4. Messaging

这一部分的功能可以通过设置`spring.sleuth.messaging.enabled`属性值为`false`来禁用。

#### 6.4.1. Spring 集成

该功能适用于所有追踪器的实现。

Spring Cloud Sleuth与[Spring Integration](https://projects.spring.io/spring-integration/)集成。它为发布和订阅事件创建跨度。要禁用Spring集成工具，请将`spring.sleuth.integration.enabled`设为`false`。

你可以提供`spring.sleuth.integration.pattern`模式来明确提供你想包括在追踪中的通道的名字。默认情况下，除了`hystrixStreamOutput`通道外，所有通道都被包括在内。

> 当使用`Executor`构建Spring Integration`IntegrationFlow`时，你必须使用`Executor`的未跟踪版本。用`TraceableExecutorService`装饰Spring Integration Executor通道会导致Span被不适当地关闭。

如果你想自定义从消息头读取和写入追踪上下文的方式，你只需注册一些类型的Bean。

- `Propagator.Setter<MessageHeaderAccessor>` - 用于向消息写入头文件
- `Propagator.Getter<MessageHeaderAccessor>` - 用于从消息中读取标题。

**Spring Integration Customization**

**Customizing messaging spans**

为了改变默认的Span名称和标签，只需注册一个`MessageSpanCustomizer`类型的bean。你也可以覆盖现有的`DefaultMessageSpanCustomizer`来扩展现有行为。

```java
@Component
  class MyMessageSpanCustomizer extends DefaultMessageSpanCustomizer {
      @Override
      public Span customizeHandle(Span spanCustomizer,
              Message<?> message, MessageChannel messageChannel) {
          return super.customizeHandle(spanCustomizer, message, messageChannel)
                  .name("changedHandle")
                  .tag("handleKey", "handleValue")
                  .tag("channelName", channelName(messageChannel));
      }

      @Override
      public Span.Builder customizeSend(Span.Builder builder,
              Message<?> message, MessageChannel messageChannel) {
          return super.customizeSend(builder, message, messageChannel)
                  .name("changedSend")
                  .tag("sendKey", "sendValue")
                  .tag("channelName", channelName(messageChannel));
      }
  }
```

#### 6.4.2. Spring Cloud Function和Spring Cloud Stream

该功能适用于所有追踪器的实现。

Spring Cloud Sleuth可以检测Spring Cloud Function。实现的方法是提供一个以`Message` 为参数的`Function`、`Consumer` 或 `Supplier`，例如 `Function<Message<String>, Message<Integer>>`。如果类型不是 `Message`，那么将不会进行仪器分析。在处理基于Reactor的流时，开箱即用的仪器不会发生，例如`Function<Flux<Message<String>>`, `Flux<Message<Integer>>`。

由于Spring Cloud Stream重用了Spring Cloud Function，你将获得开箱即用的仪器。

你可以通过将`spring.sleuth.function.enabled`的值设置为`false`来禁用这种行为。

为了与 reactive Stream函数一起工作，你可以利用`MessagingSleuthOperators`实用类，它允许你操作输入和输出的消息，以便继续跟踪上下文，并在跟踪上下文中执行自定义代码。

```java
class SimpleReactiveManualFunction implements Function<Flux<Message<String>>, Flux<Message<String>>> {

    private static final Logger log = LoggerFactory.getLogger(SimpleReactiveFunction.class);

    private final BeanFactory beanFactory;

    SimpleReactiveManualFunction(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    @Override
    public Flux<Message<String>> apply(Flux<Message<String>> input) {
        return input.map(message -> (MessagingSleuthOperators.asFunction(this.beanFactory, message))
                .andThen(msg -> MessagingSleuthOperators.withSpanInScope(this.beanFactory, msg, stringMessage -> {
                    log.info("Hello from simple manual [{}]", stringMessage.getPayload());
                    return stringMessage;
                })).andThen(msg -> MessagingSleuthOperators.afterMessageHandled(this.beanFactory, msg, null))
                .andThen(msg -> MessageBuilder.createMessage(msg.getPayload().toUpperCase(), msg.getHeaders()))
                .andThen(msg -> MessagingSleuthOperators.handleOutputMessage(this.beanFactory, msg)).apply(message));
    }

}
```

#### 6.4.3. Spring RabbitMq

他的功能适用于Brave追踪器的实现。

我们对`RabbitTemplate`进行检测，这样追踪头就会被注入消息中。

要阻止这个功能，请将`spring.sleuth.messaging.rabbit.enabled`设为`false`。

#### 6.4.4. Spring Kafka

这个功能适用于Brave追踪器的实现。

我们对Spring Kafka的 `ProducerFactory` 和 `ConsumerFactory` 进行检测，这样追踪头就会被注入创建的Spring Kafka的 `Producer` 和 `Consumer` 中。

要阻止这个功能，请将`spring.sleuth.messaging.kafka.enabled`设为`false`。

#### 6.4.5. Spring Kafka Streams

这个功能适用于Brave追踪器的实现。

我们对`KafkaStreams``KafkaClientSupplier`进行检测，这样追踪头就会被注入到`Producer`和`Consumer`s。`KafkaStreamsTracing`bean允许通过额外的`TransformerSupplier`和`ProcessorSupplier`方法进行进一步追踪。

要阻止这个功能，请将`spring.sleuth.messaging.kafka.streams.enabled`设为`false`。

#### 6.4.6. Spring JMS

这个功能适用于Brave追踪器的实现。

我们对`JmsTemplate`进行分析，以便将追踪的头信息注入到消息中。我们还支持消费者方面的`@JmsListener`注解方法。

要阻止这个功能，请将`spring.sleuth.messaging.jms.enabled`设为`false`。

> 我们不支持JMS的baggage传播

### 6.5. OpenFeign

该功能适用于所有追踪器的实现。

默认情况下，Spring Cloud Sleuth通过`TraceFeignClientAutoConfiguration`提供与Feign的集成。你可以通过设置`spring.sleuth.feign.enabled`为`false`来完全禁用它。如果你这样做，就不会有任何与Feign相关的检测发生。

部分Feign工具化是通过`FeignBeanPostProcessor`完成的。你可以通过设置`spring.sleuth.feign.processor.enabled`为`false`来禁用它。如果你把它设置为 `false`，Spring Cloud Sleuth就不会对你的任何自定义Feign组件进行检测。然而，所有默认的检测仍然存在。

### 6.6. OpenTracing

该功能适用于所有追踪器的实现。

Spring Cloud Sleuth与[OpenTracing](https://opentracing.io/)兼容。如果你在classpath上有OpenTracing，我们会自动注册OpenTracing的`Tracer`bean。如果你想禁用它，请将`spring.sleuth.opentracing.enabled`设为`false`。

### 6.7. Quartz

该功能适用于所有追踪器的实现。

我们通过向quartz 添加Job/Trigge来检测Quartz Scheduler。

要关闭该功能，请将`spring.sleuth.quartz.enabled`属性设置为`false`。

### 6.8. Reactor

该功能适用于所有追踪器的实现。

我们有以下基于reactor的应用程序的仪器化模式，可以通过`spring.sleuth.reactor.instrumation-type`属性来设置。

- `DECORATE_QUEUES` - 通过新的Reactor[队列包装机制](https://github.com/reactor/reactor-core/pull/2566) (Reactor 3.4.3)，我们对Reactor切换线程的方式进行仪表化。这将导致与`ON_EACH`的功能相同，对性能影响较小。
- `DECORATE_ON_EACH'--将每个Reactor操作者包裹在一个跟踪表示中。在大多数情况下会传递跟踪上下文。这种模式可能会导致性能急剧下降。
- `DECORATE_ON_LAST` - 在跟踪表示中包装最后一个Reactor操作符。在某些情况下会传递跟踪上下文，因此访问MDC上下文可能不工作。这种模式可能导致中等程度的性能下降。
- `MANUAL` - 以最不具侵略性的方式包装每个Reactor，不传递跟踪上下文。这取决于用户的操作。

目前默认的是`ON_EACH`，因为向后兼容的原因，但是我们鼓励用户迁移到`MANUAL`工具，从`WebFluxSleuthOperators`和`MessagingSleuthOperators`中获益。性能的提高是巨大的。

Example

```java
@GetMapping("/simpleManual")
public Mono<String> simpleManual() {
    return Mono.just("hello").map(String::toUpperCase).doOnEach(WebFluxSleuthOperators
            .withSpanInScope(SignalType.ON_NEXT, signal -> log.info("Hello from simple [{}]", signal.get())));
}
```

### 6.9. Redis

这个功能适用于Brave追踪器的实现。

我们将`tracing`属性设置为Lettuce`ClientResources`实例，以启用Lettuce中内置的Brave追踪功能。要禁用Redis支持，将`spring.sleuth.redis.enabled`属性设为`false`。

### 6.10. Runnable and Callable

这个功能对所有追踪器的实现都是可用的。

如果你把你的逻辑包在`Runnable`或`Callable`中，你可以把这些类包在它们的Sleuth代表中，如下面`Runnable`的例子所示。

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        // do some work
    }

    @Override
    public String toString() {
        return "spanNameFromToStringMethod";
    }
};
// Manual `TraceRunnable` creation with explicit "calculateTax" Span name
Runnable traceRunnable = new TraceRunnable(this.tracer, spanNamer, runnable, "calculateTax");
```

下面的例子显示了如何为`Callable`做到这一点。

```java
Callable<String> callable = new Callable<String>() {
    @Override
    public String call() throws Exception {
        return someLogic();
    }

    @Override
    public String toString() {
        return "spanNameFromToStringMethod";
    }
};
// Manual `TraceCallable` creation with explicit "calculateTax" Span name
Callable<String> traceCallable = new TraceCallable<>(tracer, spanNamer, callable, "calculateTax");
```

这样，你就能确保每次执行时都能创建和关闭一个新的Span。

### 6.11. RPC

这个功能适用于Brave追踪器的实现。

Sleuth自动配置了`RpcTracing`bean，作为RPC工具的基础，如gRPC或Dubbo。

如果需要定制客户端/服务器的RPC跟踪采样，只需注册一个`brave.sampler.SamplerFunction<RpcRequest>`类型的bean，并将该bean命名为`sleuthRpcClientSampler`用于客户端采样，`sleuthRpcServerSampler`用于服务器采样。

为了方便起见，`@RpcClientSampler`和`@RpcServerSampler`注解可用于注入适当的Bean，或通过静态字符串`NAME`字段引用Bean名称。

例如。这里有一个采样器，每秒追踪100个 `GetUserToken` 服务器请求。这不会启动对健康检查服务请求的新追踪。其他请求将使用全局采样配置。

```java
@Configuration(proxyBeanMethods = false)
    class Config {
  @Bean(name = RpcServerSampler.NAME)
  SamplerFunction<RpcRequest> myRpcSampler() {
      Matcher<RpcRequest> userAuth = and(serviceEquals("users.UserService"), methodEquals("GetUserToken"));
      return RpcRuleSampler.newBuilder().putRule(serviceEquals("grpc.health.v1.Health"), Sampler.NEVER_SAMPLE)
              .putRule(userAuth, RateLimitingSampler.create(100)).build();
  }
}
```

更多内容请见[github.com/openzipkin/brave/tree/master/instrumentation/rpc#sampling-policy](https://github.com/openzipkin/brave/tree/master/instrumentation/rpc#sampling-policy)

#### 6.11.1. Dubbo RPC support

通过与Brave的集成，Spring Cloud Sleuth支持[Dubbo](https://dubbo.apache.org/)。只需添加`brave-instrumentation-dubbo`的依赖关系即可。

```xml
<dependency>
    <groupId>io.zipkin.brave</groupId>
    <artifactId>brave-instrumentation-dubbo</artifactId>
</dependency>
```

你还需要设置一个`dubbo.properties`文件，内容如下。

```properties
dubbo.provider.filter=tracing
dubbo.consumer.filter=tracing
```

你可以阅读更多关于Brave - Dubbo整合的信息[这里](https://github.com/openzipkin/brave/tree/master/instrumentation/dubbo-rpc)。可以找到一个Spring Cloud Sleuth和Dubbo的例子[这里](https://github.com/openzipkin/sleuth-webmvc-example/compare/add-dubbo-tracing)。

#### 6.11.2. gRPC

Spring Cloud Sleuth通过Brave追踪器为[gRPC](https://grpc.io/)提供了工具。你可以通过设置`spring.sleuth.grpc.enabled`为`false`来完全禁用它。

**Variant 1**

Dependencies

> gRPC集成依赖于两个外部库来检测客户端和服务器，这两个库都必须在类的路径上，以实现检测。

```xml
        <dependency>
            <groupId>io.github.lognet</groupId>
            <artifactId>grpc-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.brave</groupId>
            <artifactId>brave-instrumentation-grpc</artifactId>
        </dependency>
```

Server Instrumentation

Spring Cloud Sleuth利用grpc-spring-boot-starter将Brave的gRPC服务器拦截器与所有用`@GRpcService`注解的服务进行注册。

Client Instrumentation

gRPC客户端利用`ManagedChannelBuilder`来构建`ManagedChannel`，用于与gRPC服务器通信。本机的 `ManagedChannelBuilder` 提供了静态方法作为构建 `ManagedChannel` 实例的入口，然而，这种机制是在Spring应用环境的影响之外。

> Spring Cloud Sleuth提供了一个 `SpringAwareManagedChannelBuilder`，可以通过Spring application context进行定制，并由gRPC客户端进行注入。**在创建 `ManagedChannel`实例时，必须使用这个构建器**。

Sleuth创建了一个`TracingManagedChannelBuilderCustomizer`，将Brave的客户端拦截器注入到`SpringAwareManagedChannelBuilder`。

**Variant 2**

[Grpc Spring Boot Starter](https://github.com/yidongnan/grpc-spring-boot-starter)自动检测Spring Cloud Sleuth和Brave的gRPC仪器的存在，并注册了必要的客户端和/或服务器工具。

### 6.12. RxJava

这个功能对所有追踪器的实现都是可用的。

我们注册了一个自定义的[`RxJavaSchedulersHook`](https://github.com/ReactiveX/RxJava/wiki/Plugins#rxjavaschedulershook)，将所有的`Action0`实例包裹在它们的Sleuth代表中，这被称为`TraceAction`。这个钩子要么开始，要么继续，取决于在Action被安排之前跟踪是否已经在进行。要禁用自定义的`RxJavaSchedulersHook`，将`spring.sleuth.rxjava.schedulers.hook.enabled`设置为`false`。

你可以为不希望创建跨度的线程名称定义一个正则表达式列表。要做到这一点，请在`spring.sleuth.rxjava.schedulers.ignoredthreads`属性中提供一个逗号分隔的正则表达式列表。

> 对反应式编程和Sleuth的建议方法是使用Reactor支持。

### 6.13. Spring Cloud CircuitBreaker

这个功能对所有追踪器的实现都是可用的。

如果你在classpath上有Spring Cloud CircuitBreaker，我们将把传递的命令`Supplier`和fallback的`Function`包裹在它的跟踪表示中。为了禁用这个工具，请将`spring.sleuth.circuitbreaker.enabled`设为`false`。

## 7. 配置属性

要查看所有Spring Cloud Netflix相关配置属性的列表，请检查[附录页面](https://docs.spring.io/spring-cloud-sleuth/docs/3.0.3/reference/htmlsingle/#common-application-properties)。
