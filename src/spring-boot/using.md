# 使用SpringBoot

本节更详细地介绍了你应该如何使用Spring Boot。它涵盖了诸如构建系统、自动配置、以及如何运行你的应用程序等主题。我们还介绍了一些Spring Boot的最佳实践。虽然Spring Boot没有什么特别之处（它只是另一个你可以使用的库），但有一些建议，如果遵循这些建议，可以使你的开发过程更容易一些。

如果你开始使用Spring Boot，在进入本节之前，你可能应该先阅读[入门](/spring-boot/getting-started.html)指南。

## 1.  构建系统

强烈建议您选择一个支持[*依赖性管理*](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.dependency-management)的构建系统，并且能够消费发布到 "Maven Central" 资源库的工件。我们建议你选择Maven或Gradle。让Spring Boot与其他构建系统（例如Ant）配合使用也是可能的，但它们的支持度不是特别高。

### 1.1. 依赖管理

Spring Boot的每个版本都提供了一个它所支持的依赖关系的精选列表。在实践中，你不需要在构建配置中为这些依赖提供一个版本，因为Spring Boot会为你管理这些。当你升级Spring Boot本身时，这些依赖项也会以一致的方式被升级。

> 你仍然可以指定一个版本，并在需要时覆盖Spring Boot的建议。

> 这个精心策划的列表包含了所有可以在Spring Boot中使用的Spring模块，以及第三方库的精炼列表。该列表以标准材料清单的形式提供（`spring-boot-dependencies`），可用于[Maven](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.maven)和[Gradle](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.gradle)。

### 1.2. Maven

要了解如何在Maven中使用Spring Boot，请参考Spring Boot的Maven插件的文档。

* Reference （[HTML](https://docs.spring.io/spring-boot/docs/2.5.3/maven-plugin/reference/htmlsingle/)和[PDF](https://docs.spring.io/spring-boot/docs/2.5.3/maven-plugin/reference/pdf/spring-boot-maven-plugin-reference.pdf))
* [API](https://docs.spring.io/spring-boot/docs/2.5.3/maven-plugin/api/)

### 1.3. Gradle

要了解使用Spring Boot与Gradle的情况，请参考Spring Boot的Gradle插件的文档。

* Reference（[HTML](https://docs.spring.io/spring-boot/docs/2.5.3/gradle-plugin/reference/htmlsingle/)和[PDF](https://docs.spring.io/spring-boot/docs/2.5.3/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf))
* [API](https://docs.spring.io/spring-boot/docs/2.5.3/gradle-plugin/api/)

### 1.4. Ant

可以使用Apache Ant+Ivy来构建Spring Boot项目。`spring-boot-antlib` "AntLib" 模块也可以用来帮助Ant创建可执行的jar。

为了声明依赖关系，一个典型的`ivy.xml`文件看起来像下面的例子。

```xml
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```

一个典型的`build.xml`看起来像下面的例子。

```xml
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="2.5.3" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>

```

如果你不想使用`spring-boot-antlib`模块，请参阅[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.build.build-an-executable-archive-with-ant-without-using-spring-boot-antlib) “How-to”。

### 1.5. Starters

Starters是一组方便的依赖描述符，你可以在你的应用程序中包含它们。你可以获得所有你需要的Spring和相关技术的一站式服务，而不需要在样本代码中寻找和复制粘贴大量的dependency。例如，如果你想开始使用Spring和JPA进行数据库访问，在你的项目中包括`spring-boot-starter-data-jpa`依赖项。

starter包含了很多你需要的依赖，以使项目快速启动和运行，并拥有一套一致的、受支持的可管理的过渡性依赖。

> 名字里有什么
>
>所有**官方的**starter都遵循一个类似的命名模式：`spring-boot-starter-*`，其中`*`是一个特定的应用程序类型。这种命名结构是为了帮助你找到starter。许多IDE中的Maven集成可以让你按名称搜索依赖关系。例如，如果安装了相应的Eclipse或Spring Tools插件，你可以在POM编辑器中按下`ctrl-space`，然后输入 "spring-boot-starter"，就能得到完整的列表。
>
>正如"[创建你自己的starter](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.custom-starter) "一节所解释的，第三方starter不应该以`spring-boot`开头，因为它是为官方Spring Boot工件保留的。相反，第三方starter通常以项目的名称开始。例如，一个名为`thirdpartyproject`的第三方starter项目通常被命名为`thirdpartyproject-spring-boot-starter`。

**以下是Spring Boot在`org.springframework.boot` group下提供的application starters。**

表1. Spring Boot application starters

|Name|Description|
| --- | --- |
|`spring-boot-starter`|核心starter，包括自动配置支持、日志和YAML|
|`spring-boot-starter-activemq`|使用Apache ActiveMQ的JMS消息传递的Starter|
|`spring-boot-starter-amqp`|使用Spring AMQP和Rabbit MQ的Starter|
|`spring-boot-starter-aop`|使用Spring AOP和AspectJ进行面向切面的编程的Starter|
|`spring-boot-starter-artemis`| 使用Apache Artemis的JMS消息传递的starter|
|`spring-boot-starter-batch`|使用Spring Batch的starter|
|`spring-boot-starter-cache`|使用Spring框架的缓存支持的starter|
|`spring-boot-starter-data-cassandra`|使用Cassandra分布式数据库和Spring Data Cassandra的starter|
|`spring-boot-starter-data-cassandra-reactive`|使用Cassandra分布式数据库和Spring Data Cassandra Reactive的starter|
|`spring-boot-starter-data-couchbase`|使用Couchbase面向文档的数据库和Spring Data Couchbase的starter|
|`spring-boot-starter-data-couchbase-reactive`|使用Couchbase面向文档的数据库和Spring Data Couchbase Reactive的starter|
|`spring-boot-starter-data-elasticsearch`|使用Elasticsearch搜索和分析引擎以及Spring Data Elasticsearch的starter|
|`spring-boot-starter-data-jdbc`|使用Spring Data JDBC的starter|
|`spring-boot-starter-data-jpa`|使用Spring Data JPA与Hibernate的starter|
|`spring-boot-starter-data-ldap`|使用Spring Data LDAP的starter|
|`spring-boot-starter-data-mongodb`|使用MongoDB面向文档的数据库和Spring Data MongoDB的starter|
|`spring-boot-starter-data-mongodb-reactive`|使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive的starter|
|`spring-boot-starter-data-neo4j`|使用Neo4j图形数据库和Spring Data Neo4j的starter|
|`spring-boot-starter-data-r2dbc`|使用Spring Data R2DBC的starter|
|`spring-boot-starter-data-redis`|用Spring Data Redis和Lettuce客户端使用Redis键值数据存储的starter|
|`spring-boot-starter-data-redis-reactive`|使用Redis键值数据存储与Spring Data Redis Reactive和Lettuce客户端的starter|
|`spring-boot-starter-data-rest`|使用Spring Data REST通过REST暴露Spring Data存储库的starter|
|`spring-boot-starter-freemarker`|使用FreeMarker视图构建MVC网络应用程序的starter|
|`spring-boot-starter-groovy-templates`|使用Groovy模板视图构建MVC网络应用的starter|
|`spring-boot-starter-hateoas`|使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful网络应用程序的starter|
|`spring-boot-starter-integration`|使用Spring集成的starter|
|`spring-boot-starter-jdbc`|使用JDBC与HikariCP连接池的starter|
|`spring-boot-starter-jersey`|使用JAX-RS和Jersey构建RESTful网络应用的入门级软件。是[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-web)的替代品。|
|`spring-boot-starter-jooq`|使用jOOQ访问SQL数据库的starter。是[`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-data-jpa)或[`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-jdbc)的替代品。|
|`spring-boot-starter-json`|读取和写入json的starter|
|`spring-boot-starter-jta-atomikos`|使用Atomikos的JTA事务的starter|
|`spring-boot-starter-mail`|使用Java Mail和Spring Framework的电子邮件发送支持的starter|
|`spring-boot-starter-mustache`|使用Mustache视图构建网络应用的starter|
|`spring-boot-starter-oauth2-client`|使用Spring Security的OAuth2/OpenID连接客户端功能的starter|
|`spring-boot-starter-oauth2-resource-server`|使用Spring Security的OAuth2资源服务器功能的starter|
|`spring-boot-starter-quartz`|使用Quartz调度器的starter|
|`spring-boot-starter-rsocket`|构建RSocket客户端和服务器的starter|
|`spring-boot-starter-security`|使用Spring Security的starter|
|`spring-boot-starter-test`|使用包括JUnit Jupiter、Hamcrest和Mockito在内的库来测试Spring Boot应用的入门者|
|`spring-boot-starter-thymeleaf`|使用Thymeleaf视图构建MVC网络应用的starter|
|`spring-boot-starter-validation`|使用Hibernate验证器的Java Bean验证的starter|
|`spring-boot-starter-web`|使用Spring MVC构建Web（包括RESTful）应用程序的starter。使用Tomcat作为默认的嵌入式容器|
|`spring-boot-starter-web-services`|使用Spring Web services的starter|
|`spring-boot-starter-webflux`|使用Spring框架的Reactive Web支持构建WebFlux应用程序的starter|
|`spring-boot-starter-websocket`|使用Spring框架的WebSocket支持构建WebSocket应用的starter|

**除了pplication starters，以下starter可用于添加[production ready](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator)功能。**

表 2. Spring Boot production starters

|Name|Description|
| --- | --- |
|`spring-boot-starter-actuator`|使用Spring Boot的Actuator的starter，它提供了生产准备的功能，帮助你监控和管理你的应用程序。|

**最后，Spring Boot还包括以下starter，如果你想排除或调换特定的实现，可以使用。**

表 3. Spring Boot technical starters

|Name|Description|
| --- | --- |
|`spring-boot-starter-jetty`|使用Jetty作为嵌入式servlet容器的starter。是[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-tomcat)的替代品。|
|`spring-boot-starter-log4j2`|使用Log4j2进行日志记录的starter。是[`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-logging)的替代品。|
|`spring-boot-starter-logging`|使用Logback进行日志记录的starter。默认的日志starter|
|`spring-boot-starter-reactor-netty`|使用Reactor Netty作为嵌入式Reactor HTTP服务器的starter。|
|`spring-boot-starter-tomcat`|使用Tomcat作为嵌入式servlet容器的starter。默认的servlet容器starter由[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-web)使用。|
|`spring-boot-starter-undertow`|使用Undertow作为嵌入式Servlet容器的starter。是[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#spring-boot-starter-tomcat)的替代品。|

如果需要更换实现，请看[更换服务器](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.webserver.use-another)和[日志系统](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.logging.log4j)的操作文档。

> 关于其他社区贡献的启动器列表，请参见GitHub上`spring-boot-starters`模块的[README文件](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-starters/README.adoc)。

## 2. 构建你的代码

Spring Boot并不要求任何特定的代码布局来工作。但是，有一些最佳实践可以帮助。

### 2.1. 使用e “default package”

当一个类不包括 `package` 的声明时，它被认为是在 `default package` 中。通常不鼓励使用 `default package`，应避免使用。对于使用`@ComponentScan`、`@ConfigurationPropertiesScan`、`@EntityScan`或`@SpringBootApplication` 注解的Spring Boot应用程序，它可能会造成特别的问题，因为每个jar中的每个类都会被读取。

> 我们建议你遵循Java推荐的包的命名惯例，并使用反转的域名（例如，`com.example.project`）。

### 2.2. Main方法类

我们通常建议你将你的Main方法类放在根包中，高于其他类。[`@SpringBootApplication`注解](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.using-the-springbootapplication-annotation)经常被放在你的Main类上，它隐含地定义了某些项目的基础 "搜索包"。例如，如果你正在编写一个JPA应用程序，`@SpringBootApplication`注解类的包被用来搜索`@Entity`项目。使用一个根包也允许组件扫描只适用于你的项目。

> 如果你不想使用`@SpringBootApplication`，它所导入的`@EnableAutoConfiguration`和`@ComponentScan`注解定义了该行为，所以你也可以使用这些来代替。

下面列出了一个典型的布局。

```text
com
 +- example
     +- myapplication
         +- MyApplication.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`MyApplication.java`文件将声明`main`方法，以及基本的`@SpringBootApplication`，如下所示。

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

## 3. Configuration 类

Spring Boot倾向于基于Java的配置。虽然可以使用XML源的`SpringApplication`，但我们一般建议你的主要源是一个`@Configuration`类。通常，定义`main`方法的类是作为主要`@Configuration`的良好候选者。

> 互联网上已经发布了许多使用XML配置的Spring配置实例。如果可能的话，总是尝试使用基于Java的同等配置。搜索`Enable*`注解可以是一个很好的起点。

### 3.1. 导入额外的 Configuration 类

你不需要把所有的`@Configuration`放在一个单一的类中。`@Import`注解可以用来导入额外的配置类。另外，你可以使用`@ComponentScan`来自动拾取所有Spring组件，包括`@Configuration`类。

### 3.2. 导入 XML Configuration

如果你绝对必须使用基于XML的配置，我们建议你仍然从`@Configuration`类开始。然后你可以使用`@ImportResource`注解来加载XML配置文件。

## 4. Auto-configuration

Spring Boot自动配置试图根据你所添加的jar依赖项自动配置你的Spring应用程序。例如，如果`HSQLDB`在你的classpath上，而且你没有手动配置任何数据库连接Bean，那么Spring Boot就会自动配置内存数据库。

你需要将`@EnableAutoConfiguration`或`@SpringBootApplication`注解添加到你的`@Configuration`类中，从而选择加入自动配置。

> 你应该只添加一个`@SpringBootApplication`或`@EnableAutoConfiguration`注解。我们通常建议你只在你的主`@Configuration`类中添加一个或另一个。

### 4.1. 逐步替换 Auto-configuration

自动配置是非侵入性的。在任何时候，你都可以开始定义你自己的配置来替换自动配置的特定部分。例如，如果你添加了你自己的`DataSource`bean，默认的嵌入式数据库支持就会退缩。

如果你需要找出当前正在应用的自动配置，以及为什么，用`--debug`开关启动你的应用程序。这样做可以为选定的核心记录器启用调试日志，并将条件报告记录到控制台。

### 4.2. 4.2. 禁用指定的 Auto-configuration 类

如果你发现你不想要的特定自动配置类被应用，你可以使用`@SpringBootApplication`的`exclude`属性来禁用它们，如下例所示。

```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApplication {

}
```

如果该类不在classpath上，你可以使用注解的`excludeName`属性并指定完全合格的名称来代替。如果你喜欢使用`@EnableAutoConfiguration`而不是`@SpringBootApplication`，`exclude`和`excludeName`也可用。最后，你也可以通过使用`spring.autoconfigure.exclude`属性来控制要排除的自动配置类列表。

> 注解 + 配置的方式可以一起使用

> 即使自动配置类是 `public` 的，唯一被认为是公共API的方面是该类的名称，它可以用于禁用自动配置。这些类的实际内容，如嵌套的配置类或Bean方法，只供内部使用，我们不建议直接使用这些。

## 5. Spring Bean 依赖和注入

你可以自由地使用任何标准的Spring框架技术来定义你的Bean及其注入的依赖关系。我们通常推荐使用构造函数注入来连接依赖关系，并使用`@ComponentScan`来查找Bean。

如果你按照上面的建议构造你的代码（将你的应用类定位在顶级包中），你可以添加`@ComponentScan`而不需要任何参数，或者使用`@SpringBootApplication`注解来隐含地包含它。你所有的应用组件（`@Component`、`@Service`、`@Repository`、`@Controller`等）都会自动注册为Spring Bean。

下面的例子显示了一个`@Service`Bean，它使用构造函数注入来获得一个所需的`RiskAssessor`Bean。

```java
import org.springframework.stereotype.Service;

@Service
public class MyAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果一个Bean有多个构造函数，你需要用`@Autowired`来标记你希望Spring使用的那个。

```java
import java.io.PrintStream;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MyAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    private final PrintStream out;

    @Autowired
    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
        this.out = System.out;
    }

    public MyAccountService(RiskAssessor riskAssessor, PrintStream out) {
        this.riskAssessor = riskAssessor;
        this.out = out;
    }

    // ...

}
```

> 请注意，使用构造器注入让`riskAssessor`字段被标记为`final`，表明它随后不能被改变。

## 6. 使用 @SpringBootApplication 注解

许多Spring Boot开发者希望他们的应用程序能够使用自动配置、组件扫描，并且能够在他们的 "application class" 上定义额外的配置。一个`@SpringBootApplication`注解就可以用来启用这三个功能，也就是。

* `@EnableAutoConfiguration` ：启用[Spring Boot的自动配置机制](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.auto-configuration)
* `@ComponentScan` : 在应用程序所在的包上启用`@Component`扫描（参见[最佳实践](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code))
* `@SpringBootConfiguration`：启用在上下文中注册额外的Bean或导入额外的配置类。这是Spring标准`@Configuration`的替代方案，有助于在你的集成测试中进行[配置检测](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.detecting-configuration)。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @SpringBootConfiguration @EnableAutoConfiguration
                        // @ComponentScan
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

> `@SpringBootApplication`也提供了别名来定制`@EnableAutoConfiguration`和`@ComponentScan`的属性。

这些功能都不是强制性的，你可以选择用它所启用的任何功能来代替这个单一的注解。例如，你可能不希望在你的应用程序中使用组件扫描或配置属性扫描。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Import;

@SpringBootConfiguration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ SomeConfiguration.class, AnotherConfiguration.class })
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

> 在这个例子中，`MyApplication`和其他Spring Boot应用程序一样，只是`@Component`-注解的类和`@ConfigurationProperties`-注解的类没有被自动检测到，用户定义的Bean被明确导入（见`@Import`）。

## 7. 运行你的应用程序

将你的应用程序打包成jar并使用嵌入式HTTP服务器的最大优势之一是，你可以像其他应用程序一样运行你的应用程序。该样本适用于调试Spring Boot应用程序。你不需要任何特殊的IDE插件或扩展。

> 本节只涉及基于jar的打包。如果你选择将你的应用程序打包成war文件，你应该参考你的服务器和IDE文档。

### 7.1. 在 IDE 中运行

你可以从你的IDE中把Spring Boot应用作为一个Java应用运行。不过，你首先需要导入你的项目。导入步骤因你的IDE和构建系统而异。大多数IDE可以直接导入Maven项目。例如，Eclipse用户可以从`File`菜单中选择`Import...`→`Existing Maven Projects`。

如果你不能直接将项目导入IDE，你可以通过使用构建插件来生成IDE元数据。Maven包括用于[Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA](https://maven.apache.org/plugins/maven-idea-plugin/)的插件。Gradle为[各种IDE](https://docs.gradle.org/current/userguide/userguide.html)提供插件。

> 如果你不小心运行了两次web application，你会看到一个 "Port already in use"的错误。Spring Tools用户可以使用`Relaunch`按钮而不是`Run`按钮来确保任何现有的实例被关闭。

### 7.2. 以打包的方式运行的应用程序

如果你使用Spring Boot的Maven或Gradle插件来创建一个可执行的jar，你可以使用`java -jar`来运行你的应用程序，如下例所示。

```shell
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

也可以在启用远程调试支持的情况下运行一个打包的应用程序。这样做可以让你把调试器附加到你的打包的应用程序上，如下面的例子中所示。

```shell
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar

```

### 7.3. 使用Maven插件

Spring Boot Maven插件包括一个`run`目标，可用于快速编译和运行您的应用程序。应用程序以爆炸的形式运行，就像在IDE中一样。下面的例子显示了运行Spring Boot应用程序的典型Maven命令。

```shell
$ mvn spring-boot:run
```

你可能还想使用`MAVEN_OPTS`操作系统环境变量，如下例所示。

```shell
$ export MAVEN_OPTS=-Xmx1024m
```

### 7.4. 使用Gradle插件

Spring Boot Gradle插件还包括一个`bootRun`任务，可以用来以爆炸的形式运行你的应用程序。只要你应用`org.springframework.boot`和`java`插件，就会添加`bootRun`任务，如下例所示。

```shell
$ gradle bootRun
```

你可能还想使用`JAVA_OPTS`操作系统环境变量，如下例所示。

```shell
$ export JAVA_OPTS=-Xmx1024m
```

### 7.5. 热部署

由于Spring Boot应用程序是普通的Java应用程序，JVM热部署应该可以开箱即用。JVM热部署所能替换的字节码有一定的限制。要获得更完整的解决方案，可以使用[JRebel](https://www.jrebel.com/products/jrebel)。

> `spring-boot-devtools`模块还包括对快速重启应用程序的支持。详情请见[Hot swapping "How-to"](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.hotswapping)。

## 8. Developer Tools

Spring Boot包括一套额外的工具，可以使应用程序开发的体验更加愉快。`spring-boot-devtools`模块可以包含在任何项目中，以提供额外的开发时间功能。要包含devtools支持，请将模块依赖性添加到你的构建中，如下面Maven和Gradle的列表中所示。

Maven

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

Gradle

```groovy
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

> 当运行一个完全打包的应用程序时，开发者工具会被自动禁用。如果你的应用程序是从`java -jar`启动的，或者是从一个特殊的classloader启动的，那么它就被认为是一个 `production application`。你可以通过使用`spring.devtools.restart.enabled`系统属性来控制这一行为。要启用devtools，无论使用何种类加载器来启动你的应用程序，设置`-Dspring.devtools.restart.enabled=true`系统属性。在生产环境中不能这样做，因为运行devtools会有安全风险。要禁用devtools，请排除该依赖关系或设置`-Dspring.devtools.restart.enabled=false`系统属性。

> 在Maven中把该依赖标记为可选，或在Gradle中使用 `developmentOnly` 配置（如上所示），可以防止devtools被过渡应用到使用你的项目的其他模块。

> 重新打包的归档文件默认不包含devtools。如果你想使用[某个远程devtools功能](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.remote-applications)，你需要包含它。使用Maven插件时，将`excludeDevtools`属性设置为`false`。使用Gradle插件时，[配置任务的classpath以包括`developmentOnly`配置](https://docs.spring.io/spring-boot/docs/2.5.3/gradle-plugin/reference/htmlsingle/#packaging-executable-configuring-including-development-only-dependencies)。

### 8.1. 默认属性值

Spring Boot支持的几个库使用缓存来提高性能。例如，[模板引擎](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.template-engines)对编译的模板进行缓存，以避免重复解析模板文件。另外，Spring MVC在提供静态资源时可以在响应中添加HTTP缓存头。

虽然缓存在生产中非常有利，但在开发过程中可能会产生反作用，使你无法看到你刚刚在应用中的变化。由于这个原因，spring-boot-devtools默认禁用了缓存选项。

缓存选项通常由你的`application.properties`文件中的设置来配置。例如，Thymeleaf提供了`spring.thymeleaf.cache`属性。与其需要手动设置这些属性，`spring-boot-devtools`模块会自动应用合理的开发时配置。

因为你在开发Spring MVC和Spring WebFlux应用程序时需要更多关于Web请求的信息，开发者工具将为`web`日志组启用`DEBUG`日志。这将给你提供关于传入请求的信息，哪个处理程序正在处理它，响应结果等。如果你希望记录所有的请求细节（包括潜在的敏感信息），你可以打开`spring.mvc.log-request-details`或`spring.codec.log-request-details`配置属性。

> 如果你不希望应用属性默认值，你可以在你的`application.properties`中设置`spring.devtools.add-perties`为`false`。

> 关于devtools应用的属性的完整列表，见[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)。

TODO 