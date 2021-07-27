# 开始使用

如果你正在开始使用Spring Boot，或一般的 "Spring"，请从阅读本节开始。它回答了 "什么？"、"如何？"和 "为什么？"这些基本问题。它包括对Spring Boot的介绍，以及安装说明。然后，我们将引导你构建你的第一个Spring Boot应用程序，在此过程中讨论一些核心原则。

## 1. Spring Boot 介绍

Spring Boot帮助你创建可以运行的独立的、基于Spring的生产级应用程序。我们对Spring平台和第三方库采取了有主见的观点，这样你就能以最少的麻烦开始工作。大多数Spring Boot应用程序只需要很少的Spring配置。

你可以使用Spring Boot创建Java应用程序，通过使用`java -jar`或更传统的war部署来启动。我们还提供一个运行 "spring scripts "的命令行工具。

我们的主要目标是。

* 为所有的Spring开发提供一个根本性的更快、更广泛的启动体验。
* 开箱即有意见，但当需求开始偏离默认值时，很快就能摆脱困境。
* 提供一系列大类项目常见的非功能特性（如嵌入式服务器、安全、度量、健康检查和外部化配置）。
* 绝对没有代码生成，也不要求XML配置。

## 2. 系统要求

Spring Boot 2.5.3需要[Java 8](https://www.java.com/)，并且兼容到Java 16，包括Java 16。还需要[Spring Framework 5.3.9](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/)或以上版本。

为以下构建工具提供了明确的构建支持。Spring Boot 2.5.3需要[Java 8](https://www.java.com/)，并且兼容到Java 16，包括Java 16。还需要[Spring Framework 5.3.9](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/)或以上版本。

为以下构建工具提供了明确的构建支持。

|Build Tool|Version|
| --- | --- |
|Maven|3.5+|
|Gradle|6.8.x, 6.9.x, and 7.x|

### 2.1. Servlet 容器

Spring Boot支持以下嵌入式Servlet容器。

|Name|Servlet Version|
| --- | --- |
|Tomcat 9.0|4.0|
|Jetty 9.4|3.1|
|Jetty 10.0|4.0|
|Undertow 2.0|4.0|

你也可以将Spring Boot应用部署到任何兼容Servlet 3.1+的容器中。

## 3. 安装 Spring Boot

Spring Boot可以使用 "经典" Java开发工具，也可以作为命令行工具安装。无论哪种方式，你都需要Java SDK v1.8或更高版本。在你开始之前，你应该使用以下命令检查你当前的Java安装。

```shell
$ java -version
```

如果你是Java开发的新手，或者你想尝试使用Spring Boot，你可能想先试试[Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.installing.cli)（命令行界面）。否则，请继续阅读 "经典"安装说明。

### 3.1. 为Java开发者提供的安装说明

你可以以与任何标准Java库相同的方式使用Spring Boot。要做到这一点，在你的classpath上包括适当的spring-boot-*.jar文件。Spring Boot不需要任何特殊的工具集成，所以你可以使用任何IDE或文本编辑器。另外，Spring Boot应用程序没有什么特别之处，所以你可以像运行其他Java程序一样运行和调试Spring Boot应用程序。

虽然你可以复制Spring Boot的jars，但我们一般建议你使用支持依赖性管理的构建工具（如Maven或Gradle）。

#### 3.1.1. Maven 安装

Spring Boot与Apache Maven 3.3或以上版本兼容。如果你还没有安装Maven，可以按照[maven.apache.org](https://maven.apache.org/)上的说明进行。

> 在许多操作系统上，Maven可以通过软件包管理器来安装。如果你使用OSX Homebrew，可以尝试`brew install maven` 。Ubuntu用户可以运行`sudo apt-get install maven` 。使用[Chocolatey](https://chocolatey.org/)的Windows用户可以在elevated (administrator)提示下运行`choco install maven`。

Spring Boot的依赖项使用`org.springframework.boot` `groupId`。通常，你的Maven POM文件继承自`spring-boot-starter-parent`项目，并声明对一个或多个["Starters"](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)的依赖。Spring Boot还提供了一个可选的[Maven插件](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins.html#build-tool-plugins.maven)来创建可执行的jar。

关于开始使用Spring Boot和Maven的更多细节，可以在Maven插件参考指南的[入门部分](https://docs.spring.io/spring-boot/docs/2.5.3/maven-plugin/reference/htmlsingle/#getting-started)中找到。

#### 3.1.2 Gradle 安装

Spring Boot与Gradle 6.8、6.9和7.x兼容。如果你还没有安装Gradle，可以按照[gradle.org](https://gradle.org/)上的说明进行。

Spring Boot的依赖关系可以通过使用`org.springframework.boot` `group`来声明。通常，你的项目会声明对一个或多个["启动器"](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)的依赖关系。Spring Boot提供了一个有用的[Gradle插件](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins.html#build-tool-plugins.gradle)，可以用来简化依赖性声明和创建可执行的jar。

> **Gradle封装器**
>
>当你需要构建一个项目时，Gradle Wrapper提供了一种 "obtaining" Gradle的好方法。它是一个小脚本和库，你可以在你的代码旁边提交，以引导构建过程。详情见[docs.gradle.org/current/userguide/gradle_wrapper.html](https://docs.gradle.org/current/userguide/gradle_wrapper.html)。

关于开始使用Spring Boot和Gradle的更多细节，可以在Gradle插件参考指南的[入门部分](https://docs.spring.io/spring-boot/docs/2.5.3/gradle-plugin/reference/htmlsingle/#getting-started)中找到。

### 3.2. 安装Spring Boot CLI

Spring Boot CLI（命令行界面）是一个命令行工具，你可以用它来快速建立Spring的原型。它可以让你运行[Groovy](https://groovy-lang.org/)脚本，这意味着你有一个熟悉的类似Java的语法，而没有那么多模板代码。

你不需要使用CLI来使用Spring Boot，但它是在没有IDE的情况下快速启动Spring应用的一种方法。

#### 3.2.1. 手动安装

你可以从Spring软件仓库下载Spring CLI发行版。

* [spring-boot-cli-2.5.3-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.5.3/spring-boot-cli-2.5.3-bin.zip)
* [spring-boot-cli-2.5.3-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.5.3/spring-boot-cli-2.5.3-bin.tar.gz)

前沿技术的的[snapshot distributions](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)也是可用的。

下载后，按照解压后的存档中的[INSTALL.txt](https://raw.githubusercontent.com/spring-projects/spring-boot/v2.5.3/spring-boot-project/spring-boot-cli/src/main/content/INSTALL.txt)指示操作。总之，在`.zip`文件中的`bin/`目录下有一个`spring`脚本（Windows的`spring.bat`）。另外，你可以使用`java -jar`和`.jar`文件（该脚本帮助你确定classpath设置正确）。

#### 3.2.2. 使用SDKMAN进行安装

SDKMAN! (The Software Development Kit Manager)可用于管理各种二进制SDK的多个版本，包括Groovy和Spring Boot CLI。从[sdkman.io](https://sdkman.io/)获取SDKMAN！并通过使用以下命令安装Spring Boot。

```bash
$ sdk install springboot
$ spring --version
Spring Boot v2.5.3
```

如果你为CLI开发功能，并希望访问你建立的版本，请使用以下命令。

```bash
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.5.3-bin/spring-2.5.3/
$ sdk default springboot dev
$ spring --version
Spring CLI v2.5.3
```

前面的说明安装了一个`spring`的本地实例，称为`dev`实例。它指向你的目标构建位置，所以每次你重建Spring Boot时，`spring`都是最新的。

你可以通过运行以下命令看到它。

```shell
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 2.5.3

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================

```

#### 3.2.3. OSX Homebrew 安装

如果你是在Mac上，并且使用[Homebrew](https://brew.sh/)，你可以通过使用以下命令来安装Spring Boot CLI。

```shell
$ brew tap spring-io/tap
$ brew install spring-boot
```

Homebrew将`spring`安装到`/usr/local/bin`。

> 如果你没有看到这个公式，你安装的brew可能已经过时了。在这种情况下，运行 `brew update` 并再试一次。

#### 3.2.4. MacPorts 安装

如果你在Mac上并使用[MacPorts](https://www.macports.org/)，你可以通过使用以下命令安装Spring Boot CLI。

```shell
$ sudo port install spring-boot-cli
```

#### 3.2.5. Command-line Completion

Spring Boot CLI包括为[BASH](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29)和[zsh](https://en.wikipedia.org/wiki/Z_shell) `shell`提供命令完成的脚本。你可以在任何shell中`source`脚本（也被命名为`spring`），或者把它放在你的个人或系统范围的bash完成初始化中。在Debian系统中，全系统的脚本都在`/shell-completion/bash`中，当一个新的shell启动时，该目录中的所有脚本都会被执行。例如，如果你是通过使用SDKMAN！安装的，要手动运行该脚本，请使用以下命令。

```shell
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

> 如果你通过使用Homebrew或MacPorts安装Spring Boot CLI，命令行完成脚本会自动注册到你的shell中。

#### 3.2.6. Windows Scoop 安装

如果你在Windows上并使用[Scoop](https://scoop.sh/)，你可以通过使用以下命令安装Spring Boot CLI。

```text
> scoop bucket add extras
> scoop install springboot
```

Scoop将`spring`安装到`~/scoop/apps/springboot/current/bin`。

> 如果你没有看到应用程序清单，你安装的Scoop可能已经过期。在这种情况下，请运行 `scoop update` 并重新尝试。

#### 3.2.7. 快速启动Spring CLI实例

你可以使用下面的网络应用程序来测试你的安装。首先，创建一个名为`app.groovy`的文件，如下所示。

```groovy
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

然后从`shell`运行它，如下所示。

```shell
$ spring run app.groovy
```

> 你的应用程序的第一次运行很慢，因为要下载依赖。随后的运行会快得多。

在你喜欢的网络浏览器中打开`localhost:8080`。你应该看到以下输出。

```text
Hello World!
```

## 4. 开发你的第一个Spring Boot应用程序

本节介绍如何开发一个小型的 "Hello World!"网络应用，突出Spring Boot的一些关键特性。我们使用Maven来构建这个项目，因为大多数IDE都支持它。

> [spring.io](https://spring.io/)网站上有许多使用Spring Boot的 "入门" [指南](https://spring.io/guides)。如果你需要解决一个特定的问题，请先在那里查看。
>
> 你可以通过进入[start.spring.io](https://start.spring.io/)并从依赖项搜索器中选择 "Web" starter来缩短下面的步骤。这样做会生成一个新的项目结构，这样你就可以[立即开始编码](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.first-application.code)。查看 [start.spring.io 用户指南](https://github.com/spring-io/start.spring.io/blob/main/USING.adoc) 了解更多细节。

在我们开始之前，请打开终端并运行以下命令，以确保你安装了有效的Java和Maven版本。

```shell
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

```shell
$ mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```

> 这个案例需要在它自己的目录中创建。后面的说明假定你已经创建了一个合适的目录，并且它是你的当前目录。

### 4.1. 创建 pom 文件

我们需要先创建一个Maven的`pom.xml`文件。`pom.xml`是用于构建项目的配置。打开你喜欢的文本编辑器，添加以下内容。

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
        <version>2.5.3</version>
    </parent>

    <!-- Additional lines to be added here... -->

</project>

```

前面的列表应该给你一个工作的构建。你可以通过运行`mvn package`来测试它（现在，你可以忽略 "jar will be empty - no content was marked for inclusion!"的警告）。

> 这时，你可以把项目导入IDE（大多数现代Java IDE都包含对Maven的内置支持）。为简单起见，我们在本例中继续使用纯文本编辑器。

### 4.2. 添加Classpath依赖项

Spring Boot提供了一些 "Starters"，让你在classpath中添加jars。我们用于smoke tests的应用程序在POM的`parent`部分使用`spring-boot-starter-parent`。`spring-boot-starter-parent`是一个特殊的starter，提供有用的Maven默认值。它还提供了一个[`依赖管理`](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.dependency-management)部分，这样你就可以省略 "blessed"依赖的`version`标签。

其他 "starter"提供了你在开发特定类型的应用程序时可能需要的依赖项。由于我们正在开发一个Web应用程序，我们添加一个`spring-boot-starter-web`依赖项。在此之前，我们可以通过运行下面的命令来看看我们目前拥有什么。

```shell
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

`mvn dependency:tree`命令打印了一个项目依赖的树状图。你可以看到`spring-boot-starter-parent`本身没有提供任何依赖性。要添加必要的依赖，编辑你的`pom.xml`并在`parent`部分下面添加`spring-boot-starter-web`的依赖。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

如果你再次运行`mvn dependency:tree`，你会看到现在有一些额外的依赖，包括Tomcat网络服务器和Spring Boot本身。

### 4.3. 编写代码

为了完成我们的应用，我们需要创建一个单独的Java文件。默认情况下，Maven从`src/main/java`编译源代码，所以你需要创建该目录结构，然后添加一个名为`src/main/java/MyApplication.java`的文件，包含以下代码。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class MyApplication {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}

```

虽然这里的代码不多，但有相当多的事情要做。我们在接下来的几节中逐步介绍重要的部分。

#### 4.3.1. @RestController和@RequestMapping注解

我们的 `MyApplication` 类上的第一个注解是`@RestController`。这被称为*stereotype*注解。它为阅读代码的人和Spring提供了提示，说明这个类扮演了一个特定的角色。在本例中，我们的类是一个网络`@Controller`，所以Spring在处理传入的网络请求时考虑它。

`@RequestMapping`注解提供了 "路由"信息。它告诉Spring，任何带有`/`路径的HTTP请求都应该被映射到`home`方法中。`@RestController`注解告诉Spring将结果字符串直接渲染给调用者。

> `@RestController`和`@RequestMapping`注解是Spring MVC注解（它们不是Spring Boot特有的）。详情请参见Spring参考文档中的[MVC部分](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc)。

#### 4.3.2. @EnableAutoConfiguration 注解

第二个类级注解是`@EnableAutoConfiguration`。这个注解告诉Spring Boot根据你添加的jar依赖项 "猜测"你想如何配置Spring。由于`spring-boot-starter-web`添加了Tomcat和Spring MVC，自动配置会假定你正在开发一个Web应用，并相应地设置Spring。

> **Starters和Auto-configuration**
>
>自动配置被设计成与 "Starters"一起工作，但这两个概念并不直接联系在一起。你可以自由地在 "Starters"之外挑选jar依赖项。Spring Boot仍然会尽力自动配置你的应用程序。

#### 4.3.3. Main 方法

我们应用程序的最后部分是`main`方法。这是一个标准的方法，遵循应用程序入口点的Java惯例。我们的main方法通过调用`run`委托给Spring Boot的`SpringApplication`类。`SpringApplication`引导我们的应用程序，启动Spring，而Spring又会启动自动配置的Tomcat网络服务器。我们需要将`MyApplication.class`作为参数传递给`run`方法，以告诉`SpringApplication`哪个是主要的Spring组件。`args`数组也被传入，以显示任何命令行参数。

### 4.4. 运行案例

在这一点上，你的应用程序应该工作了。由于你使用了`spring-boot-starter-parent` POM，你有一个有用的运行目标，你可以用它来启动应用程序。在项目根目录下输入`mvn spring-boot:run`来启动应用程序。你应该看到类似以下的输出。

```shell
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.5.3)
....... . . .
....... . . . (log output here)
....... . . .
........ Started MyApplication in 2.222 seconds (JVM running for 6.514)
```

如果你打开网页浏览器到`localhost:8080`，你应该看到以下输出。

```text
Hello World!
```

要优雅地退出应用程序，按`ctrl-c`。

### 4.5. 创建一个可执行Jar

我们通过创建一个完全独立的可执行jar文件来完成我们的例子，我们可以在生产中运行。可执行的jar文件（有时称为 "fat jars"）是包含你的编译类以及你的代码运行所需的所有jar依赖项的存档。

> **可执行的jar和Java**
>
>Java没有提供一个标准的方法来加载嵌套的jar文件（jar文件本身包含在jar中）。如果你想发布一个独立的应用程序，这可能是个问题。
>
>为了解决这个问题，许多开发者使用 "uber" jar。uber jar将所有应用程序依赖的所有类打包成一个单一的存档。这种方法的问题是，很难看到哪些库在你的应用程>序中。如果在多个jar中使用相同的文件名（但内容不同），也会产生问题。
>
>Spring Boot采取了一种[不同的方法](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html#executable-jar)，让你实际上直接嵌套jars。

为了创建一个可执行的jar，我们需要在`pom.xml`中添加`spring-boot-maven-plugin`。为此，在 `dependencies` 部分下面插入以下几行。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

`spring-boot-starter-parent`POM包括`<executions>`配置来绑定`repackage`目标。如果你不使用父POM，你需要自己声明这个配置。详情请见[plugin documentation](https://docs.spring.io/spring-boot/docs/2.5.3/maven-plugin/reference/htmlsingle/#getting-started)。

保存你的pom.xml并从命令行运行`mvn package`，如下所示。

```shell
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.5.3:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------

```

如果你查看目标目录，你应该看到`myproject-0.0.1-SNAPSHOT.jar`。这个文件的大小应该在`10MB`左右。如果你想偷看里面，你可以使用`jar tvf`，如下所示。

```shell
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

你还应该在目标目录中看到一个更小的文件，名为`myproject-0.0.1-SNAPSHOT.jar.original`。这是Maven在被Spring Boot重新打包之前创建的原始jar文件。

要运行该应用程序，请使用`java -jar`命令，如下所示。

```shell
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.5.3)
....... . . .
....... . . . (log output here)
....... . . .
........ Started MyApplication in 2.536 seconds (JVM running for 2.864)
```

和以前一样，要退出应用程序，按`ctrl-c`。

## 5. 接下来要读什么？

希望本节提供了一些Spring Boot的基础知识，让你开始编写自己的应用程序。如果你是一个面向任务的开发者，你可能想跳到[spring.io](https://spring.io/)并查看一些[入门](https://spring.io/guides/)指南，这些指南解决了 "我如何用Spring做这个？"的具体问题。我们还有针对Spring Boot的"[How-to](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto) "参考文档。

否则，下一个合乎逻辑的步骤是阅读 [使用spring-boot](/spring-boot/using.html) 。如果你真的没有耐心，你也可以提前阅读[Spring Boot特性](/spring-boot/features.html)。

{{#include ../license.md}}
