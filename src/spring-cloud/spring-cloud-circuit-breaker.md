# Spring Cloud Circuit Breaker

- 当前版本：2.0.2
- 修改时间：2021年7月22日
- 官方文档：[https://docs.spring.io/spring-cloud-circuitbreaker/docs/current/reference/html/](https://docs.spring.io/spring-cloud-circuitbreaker/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-circuitbreaker](https://github.com/spring-cloud/spring-cloud-circuitbreaker)

Spring Cloud Circuit breaker提供了一个跨越不同断路器实现的抽象。它提供了一个一致的API，可以在你的应用程序中使用，允许你的开发者选择最适合你的应用程序需求的断路器实现。

## 1. 使用说明

Spring Cloud CircuitBreaker项目包含Resilience4J和Spring Retry的实现。Spring Cloud CircuitBreaker中实现的API在Spring Cloud Commons中存在。这些API的使用文档位于[Spring Cloud Commons documentation](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-circuit-breaker)中。

### 1.1. 配置Resilience4J断路器

#### 1.1.1. Starters

Resilience4J的实现有两个启动器，一个用于reactive应用，一个用于非reactive应用。

- `org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j` - 非reactive应用程序
- `org.springframework.cloud:spring-cloud-starter-circuitbreaker-reactor-resilience4j` - reactive应用程序

#### 1.1.2. 自动配置

你可以通过设置`spring.cloud.circuitbreaker.resilience4j.enabled`为`false`来禁用Resilience4J自动配置。

#### 1.1.3. 默认配置

为了给所有的断路器提供一个默认的配置，创建一个`Customize` bean，它被传递给`Resilience4JCircuitBreakerFactory`或`ReactiveResilience4JCircuitBreakerFactory`。`configureDefault`方法可以用来提供一个默认的配置。

```java
@Bean
public Customizer<Resilience4JCircuitBreakerFactory> defaultCustomizer() {
    return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
            .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofSeconds(4)).build())
            .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
            .build());
}
```

Reactive Example

```java
@Bean
public Customizer<ReactiveResilience4JCircuitBreakerFactory> defaultCustomizer() {
    return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
            .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
            .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofSeconds(4)).build()).build());
}
```

#### 1.1.4. 具体的断路器配置

与提供默认配置类似，你可以创建一个`Customize` Bean，它被传递给一个 `Resilience4JCircuitBreakerFactory` 或 `ReactiveResilience4JCircuitBreakerFactory`。

```java
@Bean
public Customizer<Resilience4JCircuitBreakerFactory> slowCustomizer() {
    return factory -> factory.configure(builder -> builder.circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
            .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofSeconds(2)).build()), "slow");
}
```

除了配置被创建的断路器外，你还可以在断路器被创建后但被返回给调用者之前对其进行自定义。要做到这一点，你可以使用`addCircuitBreakerCustomizer`方法。这对于向Resilience4J断路器添加事件处理程序很有用。

```java
@Bean
public Customizer<Resilience4JCircuitBreakerFactory> slowCustomizer() {
    return factory -> factory.addCircuitBreakerCustomizer(circuitBreaker -> circuitBreaker.getEventPublisher()
    .onError(normalFluxErrorConsumer).onSuccess(normalFluxSuccessConsumer), "normalflux");
}
```

Reactive Example

```java
@Bean
public Customizer<ReactiveResilience4JCircuitBreakerFactory> slowCustomizer() {
    return factory -> {
        factory.configure(builder -> builder
        .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofSeconds(2)).build())
        .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults()), "slow", "slowflux");
        factory.addCircuitBreakerCustomizer(circuitBreaker -> circuitBreaker.getEventPublisher()
            .onError(normalFluxErrorConsumer).onSuccess(normalFluxSuccessConsumer), "normalflux");
     };
}
```

#### 1.1.5. 断路器属性配置

你可以在你的应用程序的配置属性文件中配置`CircuitBreaker`和`TimeLimiter`实例。属性配置比Java`Customizer`配置具有更高的优先级。

```yaml
resilience4j.circuitbreaker:
 instances:
     backendA:
         registerHealthIndicator: true
         slidingWindowSize: 100
     backendB:
         registerHealthIndicator: true
         slidingWindowSize: 10
         permittedNumberOfCallsInHalfOpenState: 3
         slidingWindowType: TIME_BASED
         recordFailurePredicate: io.github.robwin.exception.RecordFailurePredicate

resilience4j.timelimiter:
 instances:
     backendA:
         timeoutDuration: 2s
         cancelRunningFuture: true
     backendB:
         timeoutDuration: 1s
         cancelRunningFuture: false
```

关于Resilience4j属性配置的更多信息，见[Resilience4J Spring Boot 2配置](https://resilience4j.readme.io/docs/getting-started-3#configuration)。

#### 1.1.6. 隔离模式支持

如果`resilience4j-bulkhead`在classpath上，Spring Cloud CircuitBreaker将用Resilience4j Bulkhead来包装所有方法。你可以通过设置`spring.cluitbreaker.bulkhead.resilience4j.enabled`为`false`来禁用Resilience4j Bulkhead。

Spring Cloud CircuitBreaker Resilience4j提供两种隔离模式的实现。

- `SemaphoreBulkhead`，使用Semaphores。
- `FixedThreadPoolBulkhead`，它使用一个有界队列和一个固定的线程池。

默认情况下，Spring Cloud CircuitBreaker Resilience4j使用`FixedThreadPoolBulkhead`。更多关于实现Bulkhead模式的信息请参见[Resilience4j Bulkhead](https://resilience4j.readme.io/docs/bulkhead)。

`Customizer<Resilience4jBulkheadProvider>`可以用来提供默认的`Bulkhead`和`ThreadPoolBulkhead`配置。

```java
@Bean
public Customizer<Resilience4jBulkheadProvider> defaultBulkheadCustomizer() {
    return provider -> provider.configureDefault(id -> new Resilience4jBulkheadConfigurationBuilder()
        .bulkheadConfig(BulkheadConfig.custom().maxConcurrentCalls(4).build())
        .threadPoolBulkheadConfig(ThreadPoolBulkheadConfig.custom().coreThreadPoolSize(1).maxThreadPoolSize(1).build())
        .build()
);
}
```

#### 1.1.7. 具体的隔离配置

与证明默认的'Bulkhead'或'ThreadPoolBulkhead'配置类似，你可以创建一个`Customize` bean，这个bean被传递给`Resilience4jBulkheadProvider`。

```java
@Bean
public Customizer<Resilience4jBulkheadProvider> slowBulkheadProviderCustomizer() {
    return provider -> provider.configure(builder -> builder
        .bulkheadConfig(BulkheadConfig.custom().maxConcurrentCalls(1).build())
        .threadPoolBulkheadConfig(ThreadPoolBulkheadConfig.ofDefaults()), "slowBulkhead");
}
```

除了配置被创建的隔板外，你还可以在隔板和线程池被创建后，但在返回给调用者之前，对其进行自定义。要做到这一点，你可以使用`addBulkheadCustomizer`和`addThreadPoolBulkheadCustomizer`方法。

Bulkhead Example

```java
@Bean
public Customizer<Resilience4jBulkheadProvider> customizer() {
    return provider -> provider.addBulkheadCustomizer(bulkhead -> bulkhead.getEventPublisher()
        .onCallRejected(slowRejectedConsumer)
        .onCallFinished(slowFinishedConsumer), "slowBulkhead");
}
```

Thread Pool Bulkhead Example

```java
@Bean
public Customizer<Resilience4jBulkheadProvider> slowThreadPoolBulkheadCustomizer() {
    return provider -> provider.addThreadPoolBulkheadCustomizer(threadPoolBulkhead -> threadPoolBulkhead.getEventPublisher()
        .onCallRejected(slowThreadPoolRejectedConsumer)
        .onCallFinished(slowThreadPoolFinishedConsumer), "slowThreadPoolBulkhead");
}
```

#### 1.1.8. 隔板属性配置

你可以在你的应用程序的配置属性文件中配置 ThreadPoolBulkhead 和 SemaphoreBulkhead 实例。属性配置比Java`Customizer`配置具有更高的优先级。

```yaml
resilience4j.thread-pool-bulkhead:
    instances:
        backendA:
            maxThreadPoolSize: 1
            coreThreadPoolSize: 1
resilience4j.bulkhead:
    instances:
        backendB:
            maxConcurrentCalls: 10
```

关于Resilience4j属性配置的更多信息，见[Resilience4J Spring Boot 2配置](https://resilience4j.readme.io/docs/getting-started-3#configuration)。

#### 1.1.9. Collecting Metrics

Spring Cloud Circuit Breaker Resilience4j包含自动配置功能，只要classpath上有正确的依赖项，就可以设置度量衡收集。要启用指标收集，你必须包括`org.springframework.boot:spring-boot-starter-actuator`，和`io.github.resilience4j:resilience4j-micrometer`。关于存在这些依赖关系时产生的指标的更多信息，请参阅[Resilience4j文档](https://resilience4j.readme.io/docs/micrometer)。

> 你不必直接包括`micrometer-core`，因为它是由`spring-boot-starter-actuator`带来的。

### 1.2. 配置 Spring Retry Circuit Breakers

Spring Retry为Spring应用程序提供声明式重试支持。该项目中的一个子集包括实现断路器功能的能力。Spring Retry通过它的[`CircuitBreakerRetryPolicy`](https://github.com/spring-projects/spring-retry/blob/master/src/main/java/org/springframework/retry/policy/CircuitBreakerRetryPolicy.java)和[stateful retry](https://github.com/spring-projects/spring-retry#stateful-retry)的组合来提供断路器实现。所有使用Spring Retry创建的断路器将使用`CircuitBreakerRetryPolicy`和一个[`DefaultRetryState`](https://github.com/spring-projects/spring-retry/blob/master/src/main/java/org/springframework/retry/support/DefaultRetryState.java)。这两个类都可以使用`SpringRetryConfigBuilder`进行配置。

#### 1.2.1. 默认配置

为了给所有的断路器提供一个默认的配置，创建一个`Customize` bean，它被传递给一个`SpringRetryCircuitBreakerFactory`。`configureDefault`方法可以用来提供一个默认的配置。

```java
@Bean
public Customizer<SpringRetryCircuitBreakerFactory> defaultCustomizer() {
    return factory -> factory.configureDefault(id -> new SpringRetryConfigBuilder(id)
        .retryPolicy(new TimeoutRetryPolicy()).build());
}
```

#### 1.2.2. 具体的断路器配置

与提供默认配置类似，你可以创建一个 `Customize` Bean，它被传递给一个 `SpringRetryCircuitBreakerFactory`。

```java
@Bean
public Customizer<SpringRetryCircuitBreakerFactory> slowCustomizer() {
    return factory -> factory.configure(builder -> builder.retryPolicy(new SimpleRetryPolicy(1)).build(), "slow");
}
```

除了配置被创建的断路器外，你还可以在断路器被创建后但被返回给调用者之前对其进行自定义。要做到这一点，你可以使用`addRetryTemplateCustomizers`方法。这对于向`RetryTemplate`添加事件处理程序很有用。

```java
@Bean
public Customizer<SpringRetryCircuitBreakerFactory> slowCustomizer() {
    return factory -> factory.addRetryTemplateCustomizers(retryTemplate -> retryTemplate.registerListener(new RetryListener() {

        @Override
        public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
            return false;
        }

        @Override
        public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {

        }

        @Override
        public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {

        }
    }));
}
```

## 2. 构建

### 2.1. 基本编译和测试

要构建源代码，你需要安装JDK 1.8。

Spring Cloud使用Maven进行大多数构建相关活动，你可以通过克隆你感兴趣的项目并键入

```bash
$ ./mvnw install
```

> 您也可以自己安装Maven（>=3.3.3），运行`mvn`命令来代替下面例子中的`./mvnw`。如果你这样做，如果你的本地Maven设置不包含spring预发布工件的仓库声明，你可能还需要添加`-P spring`。

> 请注意，您可能需要通过设置`MAVEN_OPTS`环境变量，如`Xmx512m -XX:MaxPermSize=128m`来增加Maven的可用内存量。我们试图在`.mvn`配置中涵盖这一点，所以如果你发现你必须这样做才能使构建成功，请提出一个票据，将设置添加到源控制中。

关于如何构建项目的提示，请看`.travis.yml`，如果有的话。应该有一个 "script"，也许还有 "install"命令。也可以看看 "services"部分，看看是否有任何服务需要在本地运行（例如mongo或rabbit）。忽略你可能在 "before_install"中发现的与git有关的部分，因为它们与设置git证书有关，而你已经有了这些证书。

需要中间件的项目通常包括`docker-compose.yml`，所以考虑使用[Docker Compose](https://docs.docker.com/compose/)在Docker容器中运行中间件服务器。关于mongo、rabbit和redis的常见情况，请参见[scripts demo repository](https://github.com/spring-cloud-samples/scripts)中的README，以了解具体说明。

> 如果其他都失败了，用`.travis.yml`的命令来构建（通常是`./mvnw install`）。

### 2.2. 文档

spring-cloud-build模块有一个 "docs"配置文件，如果你打开它，它将尝试从`src/main/asciidoc`构建asciidoc源。作为这个过程的一部分，它将寻找 "README.adoc"，并通过加载所有内容来处理它，但不解析或渲染它，只是将它复制到"${main.baseir}"（默认为"$/tmp/releaser-1622150702029-0/spring-cloud-circuitbreaker/docs"，即项目的根）。如果README有任何改动，在Maven构建后会在正确位置显示为修改过的文件。提交并推送修改内容即可。

### 2.3. 使用代码工作

如果你没有IDE的偏好，我们建议你在处理代码时使用[Spring Tools Suite](https://www.springsource.com/developer/sts)或[Eclipse](https://eclipse.org/)。我们使用[m2eclipse](https://eclipse.org/m2e/) eclipse插件来支持maven。其他IDE和工具只要使用Maven 3.3.3或更高版本，也应能顺利工作。

#### 2.3.1. 激活Spring Maven配置文件

Spring Cloud项目需要激活 "spring" Maven配置文件，以解决spring里程碑和快照库的问题。使用你喜欢的IDE将该配置文件设置为激活状态，否则你可能会遇到构建错误。

#### 2.3.2. 用m2eclipse导入到eclipse中

在使用eclipse时，我们推荐[m2eclipse](https://eclipse.org/m2e/) eclipse插件。如果你还没有安装m2eclipse，它可以从 "eclipse marketplace"获得。

> 旧版本的m2e不支持Maven 3.3，所以一旦项目被导入Eclipse，你还需要告诉m2eclipse为项目使用正确的配置文件。如果你看到项目中与POMs有关的许多不同的错误，请检查你是否有一个最新的安装。如果你不能升级m2e，在你的`settings.xml`中加入 "spring "配置文件。或者你可以从父pom的 "spring "配置文件中复制版本库设置到你的`settings.xml`。

#### 2.3.3. 在没有m2eclipse的情况下导入到eclipse中

如果你不愿意使用m2eclipse，你可以用以下命令生成eclipse项目元数据。

 ```bash
 $ ./mvnw eclipse:eclipse
 ```

 生成的eclipse项目可以通过在文件菜单中选择导入现有项目来导入。

{{#include ../license.md}}
