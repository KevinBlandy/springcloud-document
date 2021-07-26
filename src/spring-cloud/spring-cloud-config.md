# Spring Cloud Config

- 当前版本：3.0.4
- 修改时间：2021年7月23日
- 官方文档：[https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/](https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-netflix](https://github.com/spring-cloud/spring-cloud-netflix)

Spring Cloud Config为分布式系统中的外部化配置提供服务器和客户端支持。有了配置服务器，你就有了一个集中的地方来管理所有环境中的应用程序的外部属性。客户端和服务器上的概念与Spring Environment和PropertySource抽象完全一致，因此它们非常适用于Spring应用程序，但也可用于以任何语言运行的任何应用程序。当一个应用程序通过部署渠道从开发到测试再到生产时，你可以管理这些不同环境之间的配置，并确定应用程序在迁移时拥有运行所需的一切。服务器存储后端的默认实现使用git，因此它可以轻松地支持标签版本的配置环境，以及可用于管理内容的广泛工具。很容易添加其他的实现，并将它们与Spring配置连接起来。

## 1. 快速开始

这篇快速入门文章介绍了Spring Cloud Config Server的服务器和客户端的使用情况。

首先，启动服务器，如下:
```shell
$ cd spring-cloud-config-server
$ ../mvnw spring-boot:run
```

服务器是一个 Spring Boot 应用程序，因此如果您愿意，可以从 IDE 中运行它(主类是 `ConfigServerApplication`)。

接下来尝试一个客户端，如下所示:
```
$ curl localhost:8888/foo/development
{
  "name": "foo",
  "profiles": [
    "development"
  ]
  ....
  "propertySources": [
    {
      "name": "https://github.com/spring-cloud-samples/config-repo/foo-development.properties",
      "source": {
        "bar": "spam",
        "foo": "from foo development"
      }
    },
    {
      "name": "https://github.com/spring-cloud-samples/config-repo/foo.properties",
      "source": {
        "foo": "from foo props",
        "democonfigclient.message": "hello spring io"
      }
    },
    ....
```

定位属性源的默认策略是克隆一个 git 仓库（在`spring.cloud.config.server.git.uri`）并使用它来初始化一个迷你 SpringApplication。该迷你应用程序的环境被用来列举属性源，并在一个JSON端点上发布它们。

HTTP 服务以下面的形式提供资源:
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

例如：
```
curl localhost:8888/foo/development
curl localhost:8888/foo/development/master
curl localhost:8888/foo/development,db/master
curl localhost:8888/foo-development.yml
curl localhost:8888/foo-db.properties
curl localhost:8888/master/foo-db.properties
```

其中，`application`是作为`SpringApplication`中的`spring.config.name`注入的（通常是普通Spring Boot应用中的`application`），`profile`是一个活动的描述文件（或逗号分隔的属性列表），`label`是一个可选的git标签（默认为master)。

`Spring Cloud Config`服务器从各种来源为远程客户端提供配置。下面的示例从git仓库获取配置(必须提供) ，如下面的示例所示:

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
```

其他来源包括任何兼容JDBC的数据库、Subversion、Hashicorp Vault、Credhub和本地文件系统。


### 1.1. 使用客户端

要在应用程序中使用这些功能，你可以将其构建为依赖`spring-cloud-config-client`的Spring Boot应用程序（有关示例，请参见`config-client`的测试案例或示例应用程序）。添加依赖关系的最便捷方式是使用Spring Boot starter `org.springframework.cloud:spring-cloud-starter-config`。对于Maven用户，还有一个父级pom和BOM（`spring-cloud-starter-parent`），对于Gradle和Spring CLI用户，还有一个Spring IO版本管理属性文件。下面的例子展示了一个典型的Maven配置：

**pom.xml**

```xml
<parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>{spring-boot-docs-version}</version>
       <relativePath /> <!-- lookup parent from repository -->
   </parent>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>{spring-cloud-version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>

<build>
	<plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
	</plugins>
</build>

   <!-- repositories also needed for snapshots and milestones -->
```

现在您可以创建一个标准的 Spring Boot 应用程序，例如下面的 HTTP 服务器:

```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

当这个 HTTP 服务器运行时，它从端口8888上的默认本地配置服务器(如果它正在运行)获取外部配置。要修改启动行为，可以使用`application.properties`更改配置服务器的位置，如下面的示例所示:

```
spring.config.import=optional:configserver:http://myconfigserver.com
```

默认情况下，如果没有设置应用程序名，则将使用应用程序。要修改名称，可以将以下属性添加到 application.properties 文件中:

```
spring.application.name: myapp
```
> 当设置属性`${spring.application.name}`时，不要在你的应用程序名称前加上保留词`application-`，以防止在解决正确的属性源时出现问题。

Config Server属性作为高优先级属性源显示在/env 端点中，如下面的示例所示。

```
$ curl localhost:8080/env
{
  "activeProfiles": [],
  {
    "name": "servletContextInitParams",
    "properties": {}
  },
  {
    "name": "configserver:https://github.com/spring-cloud-samples/config-repo/foo.properties",
    "properties": {
      "foo": {
        "value": "bar",
        "origin": "Config Server https://github.com/spring-cloud-samples/config-repo/foo.properties:2:12"
      }
    }
  },
  ...
}
```

一个名为`configserver:<URL of remote repository>/<file name>`的属性源包含foo属性，值为bar。

> 属性源名称中的URL是 git仓库，而不是配置服务器URL

> 如果你使用`Spring Cloud Config` 客户端，你需要设置`spring.config.import`属性，以便与Config Server绑定。你可以在[《Spring Cloud Config参考指南》](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#config-data-import)中阅读更多相关内容。

## 2. Spring Cloud Config Server

`Spring Cloud Config Server`为外部配置（名-值对或等效的YAML内容）提供了一个基于HTTP资源的API。通过使用`@EnableConfigServer`注解，该服务器可以嵌入到Spring Boot应用程序中。因此，下面的应用程序就是一个配置服务器:

**ConfigServer.java**
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
  public static void main(String[] args) {
    SpringApplication.run(ConfigServer.class, args);
  }
}
```

与所有 Spring Boot 应用程序一样，它默认运行在端口8080上，但是您可以通过各种方式将其切换到更常规的端口8888。最简单的，也是设置默认配置存储库的方法，就是使用 `spring.Config.name=configserver`启动它(Config Server jar 中有一个`configserver.yml`)。另一种方法是使用自己的`application.properties`，如下面的示例所示:

**application.properties**
```properties
server.port: 8888
spring.cloud.config.server.git.uri: file://${user.home}/config-repo
```
其中`${ user.home }/config-repo`是一个包含YAML和属性文件的 git 仓库。

> 在Windows上，如果文件URL是绝对的，带有驱动器前缀，你需要在文件URL中多加一个"/"（例如，file:///${user.home}/config-repo）。

下面的列表显示了在前面的例子中创建git仓库的方法：
```
$ cd $HOME
$ mkdir config-repo
$ cd config-repo
$ git init .
$ echo info.foo: bar > application.properties
$ git add -A .
$ git commit -m "Add application.properties"
```
> 为 git仓库使用本地文件系统只是为了进行测试。你应该使用服务器来托管生产环境中的配置存储库

> 如果你在配置库中只保留文本文件，那么配置库的初始克隆可以快速有效地进行。如果你存储二进制文件，特别是大的二进制文件，你可能会在第一次请求配置时遇到延迟，或者在服务器中遇到内存不足的错误。

### 2.1. 环境仓库

你应该把配置服务器的配置数据存储在哪里？支持这种行为的策略是`EnvironmentRepository`，为`Environment`对象服务。这个`Environment`是来自`Spring Environment`（包括作为主要特征的`propertySources`）的领域的浅拷贝。环境资源是由三个变量参数化的。

* `{application}` 映射到客户端的`spring.application.name`。
* `{profile}` 映射到客户端上的`spring.profiles.active`(逗号分隔多个)。
* `{label}` 这是一个服务器端特性，标记了一组 “versioned” 的配置文件。

仓库实现通常表现得像Spring Boot应用程序，从等于`{application}`参数的`spring.config.name`和等于`{profiles}`参数的`spring.profiles.active`加载配置文件。配置文件的优先级规则也与普通Spring Boot应用程序相同。活跃的配置文件优先于默认值，如果有多个配置文件，最后一个优先（类似于向地图添加入口）。

下面的示例客户端应用程序具有这种引导配置:
```yml
spring:
  application:
    name: foo
  profiles:
    active: dev,mysql
```
> 和 Spring Boot 应用程序一样，这些属性也可以通过环境变量或命令行参数来设置

如果资源库是基于文件的，服务器会从`application.yml`（在所有客户端之间共享）和`foo.yml`（以`foo.yml`为优先）创建一个环境。如果YAML文件里面有指向Spring配置文件的文件，这些文件将以更高的优先级被应用（按照列出的配置文件的顺序）。如果有特定于配置文件的YAML（或属性）文件，这些文件也会以高于默认值的优先权被应用。更高的优先级转换为环境中较早列出的属性源。(这些规则同样适用于独立的Spring Boot应用程序）。)

你可以将`spring.cloud.config.server.accept-empty`设置为 false，这样如果没有找到应用程序，服务器将返回 HTTP 404 状态。默认情况下，该标志被设置为 true。

### 2.1.1 Git后端
`EnvironmentRepository`的默认实现使用 Git 后台，这对于管理升级和物理环境以及审计更改非常方便。要改变仓库的位置，你可以在配置服务器中设置`spring.cloud.config.server.git.uri`配置属性（例如在 application.yml 中）。如果你用前缀`file:`来设置它，它应该从本地仓库工作，这样你就可以在没有服务器的情况下快速、轻松地开始工作。然而，在这种情况下，服务器直接在本地资源库上操作，而不克隆它（如果它不是空的也没有关系，因为配置服务器从不对 "远程"资源库进行修改）。为了扩大配置服务器的规模并使其高可用，你需要让服务器的所有实例都指向同一个资源库，所以只有共享文件系统才能发挥作用。即使在这种情况下，最好使用共享文件系统资源库的`ssh:`协议，这样服务器就可以克隆它并使用本地工作拷贝作为缓存。

这个版本库的实现将HTTP资源的`{label}`参数映射到一个git标签（提交ID、分支名称或标签）。如果git分支或标签名称包含一个斜线（/），那么HTTP URL中的标签应该用特殊字符串（\_)来指定（以避免与其他URL路径产生歧义）。例如，如果标签是foo/bar，替换掉斜线会产生以下标签：foo(\_)bar。包含特殊字符串（\_）也可以应用于`{application}`参数。如果你使用命令行客户端，如curl，要注意URL中的括号--你应该在shell中用单引号（''）来转义它们。

#### 2.1.1.1. 跳过 SSL 证书验证

通过将`git.skipSslValidation`属性设置为 true (默认为 false) ，可以禁用配置服务器对 Git 服务器的 SSL 证书的验证。
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          skipSslValidation: true
```

#### 2.1.1.2. 设置 HTTP 连接超时
您可以配置配置服务器等待获取 HTTP 连接的时间(以秒为单位)。使用 git.timeout 属性。

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          timeout: 4
```

#### 2.1.1.3. Git URI 中的占位符
`Spring Cloud Config Server`支持git仓库的URL，其中包含`{application}`和`{profile}`的占位符。(如果你需要的话，还有`{label}`，但记住，无论如何，标签都是作为git标签应用的) 因此，你可以通过使用类似于以下的结构来支持 "每个应用一个仓库"的策略。

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/{application}
```

您还可以使用类似的模式(但是使用`{profile}`)支持“每个配置文件一个仓库”策略。

此外，在`{application}`参数中使用特殊的字符串“(_)”可以支持多个组织，如下面的示例所示:
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/{application}
```

其中`{application}`在请求时以下列格式提供: `organization (_) application`。

#### 2.1.1.4. 模式匹配和多个仓库

`Spring Cloud Config`还包括对应用程序和配置文件名称的模式匹配，以支持更复杂的要求。模式格式是一个逗号分隔的`{application}/{profile}`名称列表，并带有通配符（注意，以通配符开始的模式可能需要加引号），如下例所示。

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
```

如果`{application}/{profile}`不匹配任何模式，它将使用`spring.cloud.config.server.git.URI`下定义的默认URI。在上面的示例中，对于`simple`仓库，模式是`simple/*` (它只匹配所有概要文件中名为simple的一个应用程序)。`local`仓库匹配所有概要文件中以`local`开头的所有应用程序名称(/* 后缀会自动添加到任何没有配置文件匹配器的模式中)。

> 在 "simple"例子中使用的 "one-line"快捷方式，只有在唯一需要设置的属性是URI的情况下才可以使用。如果你需要设置其他东西（证书、模式等等），你需要使用完整的形式.

Repo 中的 pattern 属性实际上是一个数组，因此可以使用 YAML 数组(或者[0]、[1]等属性文件中的后缀)绑定到多个模式。如果你要运行多个配置文件的应用程序，你可能需要这样做，如下面的例子所示:

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            development:
              pattern:
                - '*/development'
                - '*/staging'
              uri: https://github.com/development/config-repo
            staging:
              pattern:
                - '*/qa'
                - '*/production'
              uri: https://github.com/staging/config-repo
```


> Spring Cloud猜测，包含一个不以\*结尾的配置文件的模式意味着你实际上想匹配以这个模式开始的配置文件列表（所以`*/staging`是`["*/staging", "*/staging,*"]`等的快捷方式）。这种情况很常见，例如，你需要在本地运行 `development`配置文件的应用程序，但也需要远程运行 `cloud`配置文件。

每个仓库还可以有选择地将配置文件存储在子目录中，搜索这些目录的模式可以指定为搜索路径。下面的示例在顶层显示一个配置文件:

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          search-paths:
            - foo
            - bar*
```
在前面的示例中，服务器在顶层和 foo/sub 目录中搜索配置文件，还搜索名称以 bar 开头的任何子目录。

默认情况下，服务器在首次请求配置时克隆远程仓库。可以将服务器配置为在启动时克隆仓库，如下面的示例所示:
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          repos:
            team-a:
                pattern: team-a-*
                cloneOnStart: true
                uri: https://git/team-a/config-repo.git
            team-b:
                pattern: team-b-*
                cloneOnStart: false
                uri: https://git/team-b/config-repo.git
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```

在前面的示例中，服务器在启动时克隆`team-a`的`config-repo`，然后接受任何请求。除非从存储库请求配置，否则不会克隆所有其他存储库。

> 设置服务器启动时克隆资源库有助于在配置服务器启动时快速识别配置错误的配置源（如无效的资源库URI）。如果不配置`cloneOnStart`，配置服务器可能会在配置错误或无效的配置源下成功启动，直到应用程序从该配置源请求配置时才会发现错误。


#### 2.1.1.5. 认证
要在远程存储库上使用 HTTP 基本身份验证，请分别添加用户名和密码属性(不在 URL 中) ，如下例所示:
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          username: trolley
          password: strongpassword
```

如果你不使用 HTTPS 和用户凭证，当你在默认目录（~/.ssh）中存储密钥，并且 URI 指向 SSH 位置，如 `git@github.com:configuration/cloud-configuration`时，SSH 也应该开箱工作。重要的是，在 `~/.ssh/known_hosts`文件中要有 Git 服务器的条目，并且是`ssh-rsa`格式。其他格式（如ecdsa-sha2-nistp256）不被支持。为了避免意外，你应该确保 Git 服务器的`known_hosts`文件中只有一个条目，并且它与你提供给配置服务器的 URL 相匹配。如果你在 URL 中使用了一个主机名，你应该在`known_hosts`文件中准确地写入这个主机名（而不是 IP）。仓库是通过使用JGit访问的，所以你找到的任何关于这个的文档都应该是适用的。HTTPS 代理设置可以在`~/.git/config `或（与任何其他 JVM 进程的方式相同）用系统属性（`-Dhttps.proxyHost`和 `-Dhttps.proxyPort`）来设置。

> 如果你不知道你的`~/.git`目录在哪里，可以使用`git config --global`来操作设置（例如，`git config --global http.sslVerify false`）。

JGit需要PEM格式的RSA密钥。下面是一个ssh-keygen(来自openssh)命令的示例，它将生成正确格式的密钥:
```
ssh-keygen -m PEM -t rsa -b 4096 -f ~/config_server_deploy_key.rsa
```

警告:当使用SSH密钥时，期望的SSH私钥必须以`----- begin RSA PRIVATE KEY-----`开头。如果密钥以`-----BEGIN OPENSSH PRIVATE key -----`开头，则在启动`spring-cloud-config`服务器时不会加载RSA密钥。错误如下所示:
```
- Error in object 'spring.cloud.config.server.git': codes [PrivateKeyIsValid.spring.cloud.config.server.git,PrivateKeyIsValid]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [spring.cloud.config.server.git.,]; arguments []; default message []]; default message [Property 'spring.cloud.config.server.git.privateKey' is not a valid private key]

```

为了纠正上述错误，必须将RSA密钥转换为PEM格式。上面提供了一个使用openssh的示例，以适当的格式生成新密钥。

#### 2.1.1.6. 使用AWS CodeCommit进行身份验证

`Spring Cloud Config`服务器也支持AWS CodeCommit认证。AWS CodeCommit在通过命令行使用Git时使用一个身份验证助手。这个助手没有和JGit库一起使用，所以如果Git URI匹配AWS CodeCommit模式，就会为AWS CodeCommit创建一个JGit CredentialProvider。AWS CodeCommit uri遵循以下模式:

```yml
https//git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${repo}.
```

如果您使用AWS CodeCommit URI提供用户名和密码，那么它们必须是`AWS accessKeyId`和`secretAccessKey`，它们提供对仓库的访问。如果不指定用户名和密码，则使用AWS默认凭据提供程序链`accessKeyId`和`secretAccessKey`。

如果您的Git URI与CodeCommit URI模式匹配(前面显示的)，则必须在用户名和密码中或在默认凭据提供者链支持的某个位置提供有效的AWS凭据。AWS EC2实例可能使用IAM Roles作为EC2实例。

> `aws-java-sdk-core jar`是一个可选依赖项。如果`AWS -java-sdk-core jar`不在类路径中，则不会创建AWS Code提交凭据提供程序，而与git服务器URI无关。

#### 2.1.1.7. 使用Google Cloud Source认证

Spring Cloud Config Server也支持对Google Cloud Source仓库进行认证。

如果你的Git URI使用http或https协议，且域名为`source.developer.google.com`，则将使用`Google Cloud Source`凭证提供商。Google Cloud Source 仓库的 URI 格式为 `https://source.developers.google.com/p/${GCP_PROJECT}/r/${REPO}`。要获得你的资源库的URI，点击Google Cloud Source用户界面的 "Clone"，并选择 "手动生成的凭证"。不要生成任何凭证，只需复制显示的URI。

Google Cloud Source凭证提供者将使用谷歌云平台应用程序的默认凭证。关于如何为一个系统创建应用默认凭证，请参见Google Cloud SDK文档。这种方法将适用于开发环境中的用户账户和生产环境中的服务账户。

> `com.google。Auth:google-auth-library-oauth2-http`是一个可选的依赖项。如果`Google -auth-library-oauth2-http`jar不在你的类路径，则不管git服务器URI如何，都不会创建谷歌Cloud Source凭据提供者。

#### 2.1.1.8. 使用属性进行SSH配置

默认情况下，Spring Cloud Config Server 使用的 JGit 库在使用 SSH URI 连接到 Git 仓库时使用 SSH 配置文件，如`~/.ssh/known_hosts`和`/etc/ssh/ssh_config`。在 Cloud Foundry 等云环境中，本地文件系统可能是短暂的或不容易访问。对于这些情况，SSH 配置可以通过使用 Java 属性来设置。为了激活基于属性的 SSH 配置，`spring.cloud.config.server.git.ignoreLocalSshSettings`属性必须设置为 true，如以下示例所示。

```yml
  spring:
    cloud:
      config:
        server:
          git:
            uri: git@gitserver.com:team/repo1.git
            ignoreLocalSshSettings: true
            hostKey: someHostKey
            hostKeyAlgorithm: ssh-rsa
            privateKey: |
                         -----BEGIN RSA PRIVATE KEY-----
                         MIIEpgIBAAKCAQEAx4UbaDzY5xjW6hc9jwN0mX33XpTDVW9WqHp5AKaRbtAC3DqX
                         IXFMPgw3K45jxRb93f8tv9vL3rD9CUG1Gv4FM+o7ds7FRES5RTjv2RT/JVNJCoqF
                         ol8+ngLqRZCyBtQN7zYByWMRirPGoDUqdPYrj2yq+ObBBNhg5N+hOwKjjpzdj2Ud
                         1l7R+wxIqmJo1IYyy16xS8WsjyQuyC0lL456qkd5BDZ0Ag8j2X9H9D5220Ln7s9i
                         oezTipXipS7p7Jekf3Ywx6abJwOmB0rX79dV4qiNcGgzATnG1PkXxqt76VhcGa0W
                         DDVHEEYGbSQ6hIGSh0I7BQun0aLRZojfE3gqHQIDAQABAoIBAQCZmGrk8BK6tXCd
                         fY6yTiKxFzwb38IQP0ojIUWNrq0+9Xt+NsypviLHkXfXXCKKU4zUHeIGVRq5MN9b
                         BO56/RrcQHHOoJdUWuOV2qMqJvPUtC0CpGkD+valhfD75MxoXU7s3FK7yjxy3rsG
                         EmfA6tHV8/4a5umo5TqSd2YTm5B19AhRqiuUVI1wTB41DjULUGiMYrnYrhzQlVvj
                         5MjnKTlYu3V8PoYDfv1GmxPPh6vlpafXEeEYN8VB97e5x3DGHjZ5UrurAmTLTdO8
                         +AahyoKsIY612TkkQthJlt7FJAwnCGMgY6podzzvzICLFmmTXYiZ/28I4BX/mOSe
                         pZVnfRixAoGBAO6Uiwt40/PKs53mCEWngslSCsh9oGAaLTf/XdvMns5VmuyyAyKG
                         ti8Ol5wqBMi4GIUzjbgUvSUt+IowIrG3f5tN85wpjQ1UGVcpTnl5Qo9xaS1PFScQ
                         xrtWZ9eNj2TsIAMp/svJsyGG3OibxfnuAIpSXNQiJPwRlW3irzpGgVx/AoGBANYW
                         dnhshUcEHMJi3aXwR12OTDnaLoanVGLwLnkqLSYUZA7ZegpKq90UAuBdcEfgdpyi
                         PhKpeaeIiAaNnFo8m9aoTKr+7I6/uMTlwrVnfrsVTZv3orxjwQV20YIBCVRKD1uX
                         VhE0ozPZxwwKSPAFocpyWpGHGreGF1AIYBE9UBtjAoGBAI8bfPgJpyFyMiGBjO6z
                         FwlJc/xlFqDusrcHL7abW5qq0L4v3R+FrJw3ZYufzLTVcKfdj6GelwJJO+8wBm+R
                         gTKYJItEhT48duLIfTDyIpHGVm9+I1MGhh5zKuCqIhxIYr9jHloBB7kRm0rPvYY4
                         VAykcNgyDvtAVODP+4m6JvhjAoGBALbtTqErKN47V0+JJpapLnF0KxGrqeGIjIRV
                         cYA6V4WYGr7NeIfesecfOC356PyhgPfpcVyEztwlvwTKb3RzIT1TZN8fH4YBr6Ee
                         KTbTjefRFhVUjQqnucAvfGi29f+9oE3Ei9f7wA+H35ocF6JvTYUsHNMIO/3gZ38N
                         CPjyCMa9AoGBAMhsITNe3QcbsXAbdUR00dDsIFVROzyFJ2m40i4KCRM35bC/BIBs
                         q0TY3we+ERB40U8Z2BvU61QuwaunJ2+uGadHo58VSVdggqAo0BSkH58innKKt96J
                         69pcVH/4rmLbXdcmNYGm6iu+MlPQk4BUZknHSmVHIFdJ0EPupVaQ8RHT
                         -----END RSA PRIVATE KEY-----
```

下表描述了SSH配置属性。

**表1：SSH配置属性**

| 属性名称                 | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| ignoreLocalSshSettings   | 如果为`true`，使用基于属性而不是基于文件的 SSH 配置。必须在 `spring.cloud.config.server.git.ignoreLocalSshSettings` 中设置，而不是在版本库定义中。 |
| privateKey               | 有效的SSH私钥。如果ignoreLocalSshSettings为true且Git URI为SSH格式，则必须设置。 |
| hostKey                  | 有效的SSH主机密钥。如果hostKeyAlgorithm也被设置，则必须被设置。 |
| hostKeyAlgorithm         | `ssh-dss`, `ssh-rsa`, `ecdsa-sha2-nistp256`, `ecdsa-sha2-nistp384`, 或 `ecdsa-sha2-nistp521`中的一个。如果hostKey也被设置，则必须被设置。 |
| strictHostKeyChecking    | `true` 或 `false`. 如果是`false`，则忽略主机钥匙的错误。     |
| knownHostsFile           | 自定义`.known_hosts`文件的位置。                             |
| preferredAuthentications | 覆盖服务器认证方法的顺序。如果服务器在`publickey`方法之前有键盘交互式认证，这应该允许规避登录提示。 |


#### 2.1.1.9. Git搜索路径中的占位符

Spring Cloud Config Server还支持带有`{appcaliton}`和`{profile}`占位符的搜索路径。(以及`{label}`（如果你需要的话），如下面的例子所示。

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          search-paths: '{application}'
```

#### 2.1.1.10. 强制拉取Git仓库

如前所述，Spring Cloud Config Server复制了一个远程git仓库，以防本地副本变脏（例如，操作系统进程改变了文件夹内容），导致Spring Cloud Config Server无法从远程仓库更新本地副本。

为了解决这个问题，有一个`force-pull`属性，如果本地副本脏了，它可以让Spring Cloud Config Server强制从远程仓库拉取，如下例所示。

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          force-pull: true
```

如果你有一个多仓库的配置，你可以在每个仓库配置`force-pull`属性，如下例所示。

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          force-pull: true
          repos:
            team-a:
                pattern: team-a-*
                uri: https://git/team-a/config-repo.git
                force-pull: true
            team-b:
                pattern: team-b-*
                uri: https://git/team-b/config-repo.git
                force-pull: true
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```
> `force-pull`属性默认为`false`

#### 2.1.1.11. 删除Git存储库中未被追踪的分支

由于Spring Cloud Config Server有一个远程git仓库的克隆，在将分支检出到本地repo后（例如通过标签获取属性），它将永远保留这个分支或直到下一次服务器重启（创建新的本地repo）。因此，可能会出现这样的情况：远程分支被删除，但其本地副本仍可用于获取。如果Spring云配置服务器客户端服务以`--spring.cloud.config.label=deletedRemoteBranch,master`启动，它将从`deleedRemoteBranch`本地分支获取属性，但不会从master获取。

为了保持本地版本库分支的干净和远程可以设置`deleteUntrackedBranches`属性。它将使Spring Cloud Config Server强制删除本地仓库中未跟踪的分支。例子:

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          deleteUntrackedBranches: true
```
> `deleteUntrackedBranches`属性默认为`false`

#### 2.1.1.12. Git刷新频率

你可以通过使用`spring.cloud.config.server.git.refreshRate`来控制配置服务器从你的 Git 后台获取更新配置数据的频率。该属性的值是以秒为单位的。默认值为0，意味着配置服务器将在每次请求时从Git repo中获取更新的配置。

### 2.1.2 版本控制后端文件系统的使用

对于基于VCS的后端（git、svn），文件被签出或克隆到本地文件系统。默认情况下，它们被放在系统的临时目录中，前缀为`config-repo-`。例如，在linux上，它可能是`/tmp/config-repo-<randomid>`。一些操作系统会定期清理临时目录。这可能会导致意想不到的行为，比如丢失属性。为了避免这个问题，通过将`spring.cloud.config.server.git.basedir` 或`spring.cloud.config.server.svn.basedir`设置为不在系统临时结构中的目录，改变配置服务器使用的目录。

### 2.1.3 文件系统后台
配置服务器中还有一个 `native`配置文件，它不使用Git，而是从本地classpath或文件系统（你想用`spring.cloud.config.server.native.searchLocations`指向的任何静态URL）加载配置文件。要使用本地配置文件，请使用 `spring.profiles.active=native`启动配置服务器。


> 记住要为文件资源使用`file:`前缀（没有前缀的默认值通常是classpath）。与任何Spring Boot配置一样，你可以嵌入`${}`式的环境占位符，但要记住，Windows中的绝对路径需要额外的/（例如，`file:///${user.home}/config-repo）`。

> `searchLocations`的默认值与本地Spring Boot应用程序相同（即`[classpath:/, classpath:/config, file:./, file:./config]`）。这不会将服务器上的`application.properties`s暴露给所有客户端，因为在发送到客户端之前，服务器中存在的任何属性源都会被删除。

> 文件系统后端非常适合快速入门并进行测试。 要在生产中使用它，您需要确保文件系统可靠并在Config Server的所有实例上共享。

搜索位置可以包含`{application}`，`{profile}`和`{label}`的占位符。 通过这种方式，您可以分离路径中的目录并选择一个对您有意义的策略（例如每个应用程序或每个配置文件子目录）。

如果你不在搜索位置使用占位符，这个资源库也会将HTTP资源的`{label}`参数附加到搜索路径的后缀，所以属性文件会从每个搜索位置和一个与标签同名的子目录中加载（在Spring环境中，被标记的属性优先）。因此，没有占位符的默认行为与添加一个以`/{label}/`结尾的搜索位置相同。例如，`file:/tmp/config`与`file:/tmp/config,file:/tmp/config/{label}`相同。这种行为可以通过设置`spring.cloud.config.server.native.addLabelLocations=false`来禁用。


### 2.1.4 Vault 后端
Spring Cloud Config Server还支持Vault作为后端。

> Vault是一个用于安全访问秘密的工具。秘密是任何你想严格控制访问的东西，如API密钥、密码、证书和其他敏感信息。Vault为任何秘密提供了一个统一的接口，同时提供严格的访问控制并记录详细的审计日志。

有关Vault的更多信息，请参阅[Vault快速入门指南](https://learn.hashicorp.com/vault/?track=getting-started#getting-started)。

为了使配置服务器能够使用Vault后端，你可以用vault配置文件运行你的配置服务器。例如，在你的配置服务器的`application.properties`中，你可以添加`spring.profiles.active=vault`。

默认情况下，配置服务器假设您的Vault服务器运行在http://127.0.0.1:8200。它还假定后端名称为`secret`，密钥为应用。所有这些默认值都可以在您的配置服务器的`application.properties`中进行配置。下表描述了可配置的Vault属性。

| 名称              | 默认值      |
| :---------------- | :---------- |
| host              | 127.0.0.1   |
| port              | 8200        |
| scheme            | http        |
| backend           | secret      |
| defaultKey        | application |
| profileSeparator  | ,           |
| kvVersion         | 1           |
| skipSslValidation | false       |
| timeout           | 5           |
| namespace         | null        |

> 前述表格中的所有属性都必须以`spring.cloud.config.server.vault`为前缀，或者放在复合配置中正确的 Vault 部分中。

所有可配置属性都可以在`org.springframework.cloud.config.server.environment.vaultenvironmentProperties`中找到

> Vault 0.10.0引入了一个版本化的键值后端（k/v后端版本2），它暴露了一个与早期版本不同的API，它现在需要在挂载路径和实际上下文路径之间有一个`data/`，并将秘密包裹在一个数据对象中。设置`spring.cloud.config.server.vault.kv-version=2` 将考虑到这一点。

（可选），支持`Vault Enterprise X-Vault-Namespace`标头。 要将其发送到Vault设置`namespace `属性。

使用您的CONFIG SERVER运行，您可以将HTTP请求向服务器制作以检索Vault后端的值。 为此，您需要一个为Vault Server提供令牌。

首先，将一些数据放在Vault中，如下例所示：

```
$ vault kv put secret/application foo=bar baz=bam
$ vault kv put secret/myapp foo=myappsbar
```
其次，对CONFIG Server进行HTTP请求以检索值，如以下示例所示：

```
$ curl -X "GET" "http://localhost:8888/myapp/default" -H "X-Config-Token: yourtoken"
```

您应该看到类似于以下内容的响应：

```
{
   "name":"myapp",
   "profiles":[
      "default"
   ],
   "label":null,
   "version":null,
   "state":null,
   "propertySources":[
      {
         "name":"vault:myapp",
         "source":{
            "foo":"myappsbar"
         }
      },
      {
         "name":"vault:application",
         "source":{
            "baz":"bam",
            "foo":"bar"
         }
      }
   ]
}
```

客户端提供必要的认证以让配置服务器与Vault对话的默认方式是设置`X-Config-Token`头。但是，您可以通过设置与`Spring Cloud Vault`相同的配置属性，省略该头文件并在服务器中配置认证。要设置的属性是`spring.cloud.config.server.vault.authentication`。它应该被设置为支持的认证方法之一。您可能还需要设置您所使用的认证方法的其他特定属性，方法是使用与`spring.cloud.vault`记载的相同的属性名称，但使用`spring.cloud.config.server.vault`前缀。请参阅[《Spring Cloud Vault参考指南》](https://cloud.spring.io/spring-cloud-vault/reference/html/#vault.config.authentication)了解更多细节。

如果你省略了`X-Config-Token`头并使用服务器属性来设置验证，配置服务器应用程序需要额外依赖`Spring Vault`来启用额外的验证选项。请参阅`Spring Vault`参考指南，了解如何添加该依赖关系。

#### 2.1.4.1 多个属性源

当使用Vault时，您可以为您的应用程序提供多个属性源。例如，假设你已将数据写入Vault的以下路径：
```
secret/myApp,dev
secret/myApp
secret/application,dev
secret/application
```

写入`secret/application`的属性对使用配置服务器的所有应用程序都是可用的。一个名称为myApp的应用程序将有任何写入`secret/myApp和secret/application`的属性对其可用。当myApp启用`dev` `profile`时，写入上述所有路径的属性对它来说都是可用的，列表中第一个路径中的属性比其他路径优先。

### 2.1.5. 通过代理访问后端
配置服务器可以通过HTTP或HTTPS代理访问Git或Vault的后端。对于 Git 或 Vault 来说，这种行为由 `proxy.http` 和 `proxy.https` 下的设置来控制。这些设置是针对每个仓库的，所以如果你使用的是复合环境仓库，你必须为复合环境中的每个后端单独配置代理设置。如果使用的网络需要为HTTP和HTTPS URLs提供单独的代理服务器，你可以为一个后端同时配置HTTP和HTTPS代理设置。

下表描述了HTTP和HTTPS代理的代理配置属性。所有这些属性必须以`proxy.http`或`proxy.https`为前缀。

**表格2：代理配置属性**

| 属性名称          | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| **host**          | 代理的主机                                                   |
| **port**          | 用于访问的端口                                               |
| **nonProxyHosts** | 配置服务器应该在代理之外访问的任何主机。如果同时为`proxy.http.nonProxyHosts`和`proxy.https.nonProxyHosts`提供了值，将使用`proxy.http`的值。 |
| **username**      | 用来验证代理的用户名。如果`proxy.http.username`和`proxy.https.username`都提供了值，将使用`proxy.http`的值。 |
| **password**      | 用来验证代理的密码。如果同时提供`proxy.http.password`和`proxy.https.password`的值，将使用`proxy.http`的值。. |


以下配置使用HTTPS代理访问Git仓库。
```yml
spring:
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          proxy:
            https:
              host: my-proxy.host.io
              password: myproxypassword
              port: '3128'
              username: myproxyusername
              nonProxyHosts: example.com
```

### 2.1.6. 使用所有应用程序共享配置

所有应用程序之间的共享配置根据您所采取的方法而异，如以下主题中所述：
* [File Based Repositories](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-server-file-based-repositories)

* [Vault Server](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-server-vault-server)

#### 2.1.6.1.  基于文件的存储库

对于基于文件的（git、svn和本地）仓库，文件名为`application`的资源（`application.properties`、`application.yml`、`application-.properties`等）在所有客户端应用程序之间共享。你可以使用这些文件名的资源来配置全局默认值，并在必要时让它们被特定的应用程序文件所覆盖。

属性覆盖功能也可用于设置全局默认值，允许占位符应用程序在本地覆盖它们。

> 使用“本机”配置文件（本地文件系统后端），您应该使用不属于服务器自己配置的显式搜索位置。 否则，默认搜索位置中的应用程序资源被删除，因为它们是服务器的一部分。

#### 2.1.6.2. Vault 服务器
当使用Vault作为后端时，您可以通过将配置放在`secret/application`中与所有应用程序共享配置。例如，如果你运行下面的Vault命令，所有使用配置服务器的应用程序将有属性foo和baz可供他们使用。

```sh
vault write secret/application foo=bar baz=bam
```

#### 2.1.6.3. CredHub 服务器

当使用`CredHub`作为后端时，你可以通过将配置放在`/application/`中，或将其放在应用程序的默认配置文件中，与所有应用程序共享配置。例如，如果你运行下面的CredHub命令，所有使用配置服务器的应用程序将有`shared.color1`和`shared.color2`的属性可供他们使用。

```sh
credhub set --name "/application/profile/master/shared" --type=json
value: {"shared.color1": "blue", "shared.color": "red"}
```

```sh
credhub set --name "/my-app/default/master/more-shared" --type=json
value: {"shared.word1": "hello", "shared.word2": "world"}
```

### 2.1.7. JDBC 后端

`Spring Cloud Config Server`支持将JDBC（关系型数据库）作为配置属性的后端。你可以通过在`classpath`中添加`spring-jdbc`并使用jdbc配置文件或添加`JdbcEnvironmentRepository`类型的`bean`来启用该功能。如果你在`classpath`上包含了正确的依赖关系（详情见用户指南），Spring Boot会配置一个数据源。

你可以通过将`spring.cloud.config.server.jdbc.enabled` 属性设置为`false`来禁用 `JdbcEnvironmentRepository` 的自动配置。

数据库需要有一个名为`PROPERTIES`的表，其列名为`APPLICATION`、`PROFILE`和`LABEL`（具有通常的环境含义），另外还有KEY和VALUE，用于属性风格中的键和值对。所有字段都是Java中的String类型，所以你可以把它们变成任何你需要的长度的`VARCHAR`。属性值的行为与它们来自Spring Boot名为`{application}-{profile}.properties`的属性文件的行为相同，包括所有的加密和解密，这将作为后处理步骤应用（也就是说，不直接在版本库实现中应用）。

### 2.1.8. Redis 后端

`Spring Cloud Config Server`支持Redis作为配置属性的后端。你可以通过添加对`Spring Data Redis`的依赖性来启用这一功能。

**pom.xml**

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-redis</artifactId>
	</dependency>
</dependencies>
```

以下配置使用`Spring Data Redistemplate`来访问Redis。 我们可以使用`spring.redis.*`属性覆盖默认连接设置。

```yml
spring:
  profiles:
    active: redis
  redis:
    host: redis
    port: 16379
```

该属性应存储为哈希中的字段。 哈希名称应与`spring.application.name`属性或`spring.application.name`和`spring.profiles.active[n]`相同。

```sh
HMSET sample-app server.port "8100" sample.topic.name "test" test.property1 "property1"
```

运行以上可见的命令后，哈希应包含带有值的以下键：

```
HGETALL sample-app
{
  "server.port": "8100",
  "sample.topic.name": "test",
  "test.property1": "property1"
}
```

> 当没有指定配置文件时，将使用默认值

### 2.1.9. AWS S3 后端

`Spring Cloud Config Server`支持`AWS S3`作为配置属性的后端。 您可以通过向Amazon S3添加对AWS Java SDK的依赖性来启用此功能。

**pom.xml**

```xml
<dependencies>
	<dependency>
		<groupId>com.amazonaws</groupId>
		<artifactId>aws-java-sdk-s3</artifactId>
	</dependency>
</dependencies>
```

以下配置使用AWS S3客户端访问配置文件。 我们可以使用`spring.aws3.*`属性以选择存储配置的存储桶。

```yml
spring:
  profiles:
    active: awss3
  cloud:
    config:
      server:
        awss3:
          region: us-east-1
          bucket: bucket1
```

也可以指定一个AWS URL，用`spring.awss3.endpoint`覆盖你的S3服务的标准端点。这允许支持S3的测试区，以及其他S3兼容的存储API。

使用[默认的AWS凭据](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)提供程序链找到凭据。 无需进一步配置，支持版本化和加密存储桶。

配置文件以`{application}-{profile}.properties`、`{application}-{profile}.yml`或`{application}-{profile}.json`的形式存储在你的桶中。可以提供一个可选的标签来指定文件的目录路径。

> 当没有指定配置文件时，将使用默认值

### 2.1.10. CredHub 后端

`Spring Cloud Config Server`支持CredHub作为配置属性的后端。你可以通过添加对`Spring CredHub`的依赖性来启用这一功能。

**pom.xml**

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.credhub</groupId>
		<artifactId>spring-credhub-starter</artifactId>
	</dependency>
</dependencies>
```

以下配置使用相互TLS访问CredHub：

```yml
spring:
  profiles:
    active: credhub
  cloud:
    config:
      server:
        credhub:
          url: https://credhub:8844
```

该属性应存储为JSON，例如：

```sh'
credhub set --name "/demo-app/default/master/toggles" --type=json
value: {"toggle.button": "blue", "toggle.link": "red"}
```

```sh
credhub set --name "/demo-app/default/master/abs" --type=json
value: {"marketing.enabled": true, "external.enabled": false}
```

所有应用名称为`spring.cloud.config.name = demo-app`将具有以下属性：

```
{
    toggle.button: "blue",
    toggle.link: "red",
    marketing.enabled: true,
    external.enabled: false
}
```

> 当没有指定配置文件时，将使用默认值；当没有指定标签时，将使用主值作为默认值。注意：添加到应用程序的值将被所有应用程序共享。



#### 2.1.10.1 OAuth 2.0

你可以使用[UAA](https://docs.cloudfoundry.org/concepts/architecture/uaa.html)作为提供程序使用[OAuth 2.0](https://oauth.net/2/)进行身份验证。

**pom.xml**

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-oauth2-client</artifactId>
	</dependency>
</dependencies>
```

以下配置使用OAuth 2.0和UAA访问CredHub：

```yml
spring:
  profiles:
    active: credhub
  cloud:
    config:
      server:
        credhub:
          url: https://credhub:8844
          oauth2:
            registration-id: credhub-client
  security:
    oauth2:
      client:
        registration:
          credhub-client:
            provider: uaa
            client-id: credhub_config_server
            client-secret: asecret
            authorization-grant-type: client_credentials
        provider:
          uaa:
            token-uri: https://uaa:8443/oauth/token
```

> 使用的UAA客户-ID应该有credhub.read作为范围。

### 2.1.11. 复合环境仓库

在某些情况下，你可能希望从多个环境仓库中提取配置数据。要做到这一点，你可以在配置服务器的应用程序属性或YAML文件中启用`composite`配置文件。例如，如果你想从一个`Subversion` 仓库以及两个 Git 仓库提取配置数据，你可以为你的配置服务器设置以下属性。

```yml
spring:
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
        -
          type: svn
          uri: file:///path/to/svn/repo
        -
          type: git
          uri: file:///path/to/rex/git/repo
        -
          type: git
          uri: file:///path/to/walter/git/repo
```

使用这种配置，优先级是由复合键下的仓库排列顺序决定的。在上面的例子中，Subversion 仓库被列在第一位，所以在 Subversion 仓库中发现的值会优先于在某个 Git 仓库中发现的相同属性的值。在`rex` Git 仓库中找到的值会在`walter` Git 仓库中找到的相同属性值之前使用。

如果你想只从不同类型的存储库中提取配置数据，你可以在配置服务器的应用程序属性或YAML文件中启用相应的配置文件，而不是`composite`配置文件。例如，如果你想从一个Git仓库和一个`HashiCorp Vault`服务器拉取配置数据，你可以为你的配置服务器设置以下属性。

```yml
spring:
  profiles:
    active: git, vault
  cloud:
    config:
      server:
        git:
          uri: file:///path/to/git/repo
          order: 2
        vault:
          host: 127.0.0.1
          port: 8200
          order: 1
```



使用这种配置，可以通过`order`属性来确定优先权。你可以使用`order`属性来为你的所有存储库指定优先顺序。`order`属性的数值越低，它的优先级就越高。存储库的优先顺序有助于解决包含相同属性值的存储库之间的任何潜在冲突。

> 如果你的复合环境包括一个Vault服务器，就像前面的例子一样，您必须在向配置服务器发出的每个请求中包括一个Vault令牌。参见Vault后台。

> 从环境资源库检索数值时，任何类型的失败都会导致整个复合环境的失败。

> 当使用复合环境时，重要的是所有版本库都包含相同的标签。如果你有一个类似于前面例子中的环境，你请求带有master标签的配置数据，但Subversion版本库不包含一个叫做master的分支，整个请求就会失败。



#### 2.1.11.1.自定义复合环境存储库

除了使用Spring Cloud的一个环境库外，你还可以提供你自己的`EnvironmentRepository Bean`，作为复合环境的一部分。要做到这一点，你的`Bean`必须实现`EnvironmentRepository`接口。如果你想在复合环境中控制你的自定义`EnvironmentRepository`的优先级，你也应该实现`Ordered`接口并重写`getOrdered`方法。如果你不实现`Ordered`接口，你的`EnvironmentRepository`将被赋予最低的优先级。



### 2.1.12. 属性覆盖

配置服务器有一个 "overrides "功能，可以让操作员为所有应用程序提供配置属性。被覆盖的属性不能被应用程序用正常的Spring Boot钩子方法改变。要声明覆盖，请向 `spring.cloud.config.server.overrides` 添加一个`key-value`对的映射，如下面的例子所示。

```yml
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```

上面的示例会导致配置客户端的所有应用程序读取foo = bar，与自己的配置无关。

> 配置系统不能强制应用程序以任何特定方式使用配置数据。因此，覆盖是不可执行的。然而，它们确实为Spring Cloud Config客户端提供了有用的默认行为。

> 通常情况下，带有`${}`的Spring环境占位符可以通过使用反斜杠`（\）`来转义`$`或{来转义`（`。并在客户端解析）。例如，`\${app.foo:bar}`会解析为`bar`，除非应用程序提供了自己的`app.foo`。

> 在YAML中，你不需要转义反斜杠本身。然而，在属性文件中，当你在服务器上配置重写时，你确实需要转义反斜杠。

你可以通过在远程资源库中设置 `spring.cloud.config.overrideNone=true `标志（默认为 false）来改变客户端中所有覆盖的优先级，使其更像默认值，让应用程序在环境变量或系统属性中提供自己的值。

### 2.2. 健康指示器

配置服务器带有一个健康指示器，用于检查配置的 `EnvironmentRepository` 是否在工作。默认情况下，它要求`EnvironmentRepository`提供一个名为`app`的应用程序、`default`配置文件和`EnvironmentRepository`实现提供的默认标签。

你可以配置健康指标，以检查更多的应用程序以及自定义配置文件和自定义标签，如以下例子所示。

```yml
spring:
  cloud:
    config:
      server:
        health:
          repositories:
            myservice:
              label: mylabel
            myservice-dev:
              name: myservice
              profiles: development
```

你可以通过设置`Management.Health.config.enabled = false`来禁用健康指示符。

### 2.3. Security

你可以用任何对你有意义的方式来保护你的配置服务器（从物理网络安全到OAuth2令牌），因为`Spring Security`和`Spring Boot`提供对许多安全安排的支持。

要使用默认的Spring Boot配置的`HTTP Basic`安全，请在`classpath`上包含`Spring Security`（例如，通过`spring-boot-starter-security`）。默认的用户名是`user`，密码是随机生成的。随机密码在实践中没有用，所以我们建议你配置密码（通过设置`spring.security.user.password`），并对其进行加密（关于如何做到这一点，见下文的说明）。

### 2.4. 加密和解密

> 为了使用加密和解密功能，你需要在你的JVM中安装全部的JCE（默认情况下不包括）。你可以从`Oracle `公司下载 "Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files"，并遵循安装说明（基本上，你需要用你下载的文件替换`JRE lib/security`目录中的两个策略文件）。

如果远程属性源包含加密的内容（以`{cipher}`开头的值），它们在通过HTTP发送给客户端之前会被解密。这种设置的主要优点是，当属性值处于 "静止状态 "时（例如，在git仓库中），它们不需要是纯文本。如果一个值不能被解密，它就会被从属性源中删除，并添加一个额外的属性，其密钥相同，但前缀为`invalid`和一个表示 "not applicable "的值（通常是<n/a>）。这主要是为了防止密码文本被用作密码而意外地泄露出去。

如果你为配置客户端应用程序设置了一个远程配置仓库，它可能包含一个类似于下面的`application.yml`。

**application.yml**

```yml
spring:
  datasource:
    username: dbuser
    password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'
```

`application.properties`文件中的加密值不能用引号包裹。否则，该值将不会被解密。下面的例子显示了可以工作的值。

**application.properties**

```
spring.datasource.username: dbuser
spring.datasource.password: {cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ
```

你可以安全地把这个纯文本推送到共享的git仓库，而秘密密码仍然受到保护。

服务器还公开了`/encrypt`和`/decrypt`端点（假设这些端点是安全的，只有授权的代理才能访问）。如果你编辑一个远程配置文件，你可以使用配置服务器通过POST到`/encrypt`端点来加密数值，如下例所示。

```sh
$ curl localhost:8888/encrypt -s -d mysecret
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
```

> 如果你用curl测试，那么使用`--data-urlencode`（而不是-d），并在要加密的值前加上`=`（curl需要这样做），或者设置一个明确的`Content-Type: text/plain`，以确保curl在有特殊字符（'+'特别棘手）时能正确编码数据。

> 要确保在加密的值中不包括任何curl命令的统计信息，这就是为什么例子中使用-s选项来沉默它们。将数值输出到文件中可以帮助避免这个问题。

解密操作也可以通过/decrypt进行（前提是服务器配置了对称密钥或全密钥对），如下例所示。

```sh
$ curl localhost:8888/decrypt -s -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

在你把它放在YAML或属性文件中，以及在你提交和推送到远程（可能是不安全的）商店之前，采取加密的值并添加`{cipher}`前缀。

`/encrypt`和`/decrypt`端点也都接受`/*/{application}/{profiles}`形式的路径，当客户端调用主环境资源时，可用于在每个应用（名称）和每个文件的基础上控制密码。

> 为了以这种精细的方式控制加密，你还必须提供一个`TextEncryptorLocator`类型的`@Bean`，它可以为每个名字和配置文件创建一个不同的加密器。默认提供的那个没有这样做（所有的加密都使用同一个密钥）。

`spring`命令行客户端（安装了`Spring Cloud CLI`扩展）也可以用来加密和解密，如下例所示。

```sh
$ spring encrypt mysecret --key foo
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ spring decrypt --key foo 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

要使用文件中的密钥（如用于加密的RSA公钥），在密钥值前加上"@"，并提供文件路径，如下例所示。

```sh
$ spring encrypt mysecret --key @${HOME}/.ssh/id_rsa.pub
AQAjPgt3eFZQXwt8tsHAVv/QHiY5sI2dRcR+...
```

> `--key`参数是强制性的（尽管有一个`--`前缀）。

### 2.5. Key管理

配置服务器可以使用对称（共享）密钥或非对称密钥（RSA密钥对）。非对称的选择在安全性方面更胜一筹，但使用对称密钥通常更方便，因为它是在`bootstrap.properties`中配置的单一属性值。

要配置对称密钥，你需要将`encrypt.key`设置为一个秘密的String（或者使用`ENCRYPT_KEY`环境变量来防止它进入纯文本配置文件）。

> 你不能使用encrypt.key配置非对称密钥。

要配置非对称钥匙，请使用钥匙库（例如，由 JDK 附带的 keytool 工具创建）。密钥库的属性是 `encrypt.keyStore.*`，`*`可选以下属性:

|            属性             |                 描述                  |
| :-------------------------: | :-----------------------------------: |
| `encrypt.keyStore.location` |           包含一个资源位置            |
| `encrypt.keyStore.password` |           解锁钥匙库的密码            |
|  `encrypt.keyStore.alias`   |       确定要使用商店中的哪个键        |
|   `encrypt.keyStore.type`   | 要创建的KeyStore的类型。默认为`jks`。 |

加密是用公钥完成的，而解密则需要私钥。因此，原则上，如果你只想加密（并准备在本地用私钥解密），你可以只在服务器上配置公钥。在实践中，你可能不想在本地进行解密，因为它将密钥管理过程分散到所有的客户端，而不是集中在服务器上。另一方面，如果你的配置服务器相对不安全，而且只有少数客户需要加密的属性，这可能是一个有用的选择。

### 2.6. 创建用于测试的Key Store

要创建用于测试的密钥库，可以使用类似于以下内容的命令：

```sh
$ keytool -genkeypair -alias mytestkey -keyalg RSA \
  -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
  -keypass changeme -keystore server.jks -storepass letmein
```

当使用JDK 11或以上版本时，你在使用上述命令时可能会得到以下警告。在这种情况下，你可能要确保keypass和storepass的值一致。

```sh
Warning:  Different store and key passwords not supported for PKCS12 KeyStores. Ignoring user-specified -keypass value.
```

把`server.jks`文件放在`classpath`中（例如），然后，在你的`bootstrap.yml`中，为配置服务器创建以下设置。

```yml
encrypt:
  keyStore:
    location: classpath:/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```

### 2.7.使用多个Key 和 轮换 key

除了加密属性值中的`{cipher}`前缀外，配置服务器在（Base64编码的）密码文本开始前寻找零个或多个`{name:value}`前缀。密钥被传递给`TextEncryptorLocator`，它可以做任何需要的逻辑来为密码定位一个`TextEncryptor`。如果你已经配置了一个钥匙库`（encrypt.keystore.location`），默认的定位器会寻找带有由钥匙前缀提供的别名的钥匙，其密码文本类似于以下内容。

```yml
foo:
  bar: `{cipher}{key:testkey}...`
```

该定位器寻找一个名为 "testkey "的密钥。也可以通过在前缀中使用`{secret:...}`值来提供一个秘密。然而，如果没有提供，默认情况下是使用钥匙库密码（当你建立一个钥匙库而没有指定一个秘密时，你会得到这样的密码）。如果你提供了一个秘密，你也应该用一个自定义的`SecretLocator`来加密这个秘密。

当钥匙只被用来加密几个字节的配置数据时（也就是说，它们没有被用在其他地方），从密码学的角度来看，钥匙的轮换几乎没有必要。然而，你可能偶尔需要改变密钥（例如，在安全漏洞的情况下）。在这种情况下，所有的客户端都需要改变他们的源配置文件（例如，在git中），并在所有密码中使用新的`{key:...}`前缀。请注意，客户需要首先检查配置服务器密钥库中的密钥别名是否可用。

> 如果你想让配置服务器处理所有的加密以及解密，`{name:value}`前缀也可以作为纯文本添加到`/encrypt`端点上。

### 2.8. 服务加密属性

有时你想让客户端在本地解密配置，而不是在服务器中进行解密。在这种情况下，如果你提供 `encrypt.* `配置来定位一个密钥，你仍然可以有` /encrypt` 和 `/decrypt` 端点，但你需要通过在 `bootstrap.[yml|properties] `中放置 `spring.cloud.config.server.encrypt.enabled=false` 明确地关闭对出站属性的解密。如果你不关心端点，如果你不配置密钥或启用的标志，它应该可以工作。
## 3. 提供其他形式的服务

来自`environment `端点的默认JSON格式非常适合Spring应用程序使用，因为它直接映射到`environment `抽象。如果你愿意，你可以通过在资源路径中添加一个后缀（".yml"、".yaml "或".properties"）来消费YAML或Java属性的相同数据。这对于那些不关心JSON端点的结构或其提供的额外元数据的应用程序的消费是非常有用的（例如，不使用Spring的应用程序可能会从这种方法的简单性中受益）。

YAML 和属性表示法有一个额外的标志（以`boolean `查询参数的形式提供，称为 `resolvePlaceholders`），表示源文档中的占位符（以标准的`Spring ${...}`形式）应该在渲染之前在输出中被解析，可能的话。对于不了解Spring占位符约定的消费者来说，这是一个有用的功能。

> 使用YAML或属性格式有一些限制，主要是与元数据的损失有关。例如JSON的结构是一个有序的属性源列表，其名称与源相关。YAML和属性形式被凝聚成一张图，即使值的来源有多个，原始源文件的名称也会丢失。另外，YAML表示法也不一定是支持仓库中YAML源的可靠表示。它是由平面属性源的列表构建的，必须对键的形式做出假设。

## 4. 服务于纯文本

你的应用程序可能需要为其环境量身定制的通用纯文本配置文件，而不是使用环境抽象（或YAML或properties格式中的一种替代表示）。配置服务器通过`/{application}/{profile}/{label}/{path}`的额外端点提供这些文件，其中`application`、`profile`和`label`的含义与常规环境端点相同，但path是一个文件名的路径（如`log.xml`）。这个端点的源文件的位置与环境端点的相同。属性和YAML文件也使用同样的搜索路径。然而，不是聚合所有匹配的资源，而是只返回第一个匹配的资源。

找到资源后，正常格式的占位符（`${...}`）通过使用提供的应用程序名称、配置文件和标签的有效环境来解决。通过这种方式，资源端点与环境端点紧密结合。



> 与环境配置的源文件一样，配置文件被用来解决文件名。因此，如果你想要一个特定的配置文件，`/*/development/*/logback.xml`可以被一个叫做`logback-development.xml`的文件解决（优先于logback.xml）。

> 如果你不想提供标签而让服务器使用默认标签，你可以提供一个`useDefaultLabel`请求参数。因此，前面的默认配置文件的例子可以是`/sample/default/nginx.conf?useDefaultLabel`。

目前，Spring Cloud Config可以为git、SVN、本地后端和AWS S3提供纯文本。对git、SVN和本地后端的支持是相同的。AWS S3的工作方式有些不同。下面的章节展示了每一种的工作方式。

- [Git, SVN, and Native Backends](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-serving-plain-text-git-svn-native-backends)
- [AWS S3](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-serving-plain-text-aws-s3)

### 4.1. GIT,SVN和Native 后端

Git或SVN存储库或Native后端的以下示例：

```
application.yml
nginx.conf
```

`nginx.conf`类似于以下列表：

```
server {
    listen              80;
    server_name         ${nginx.server.name};
}
```

`application.yml`类似于以下列表：

```yml
nginx:
  server:
    name: example.com
---
spring:
  profiles: development
nginx:
  server:
    name: develop.com
```

`/sample/default/master/nginx.conf`资源类似如下。

```
server {
    listen              80;
    server_name         example.com;
}
```

`/sample/development/master/nginx.conf` 类似如下：

```
server {
    listen              80;
    server_name         develop.com;
}
```

### 4.2. AWS S3

要为AWS s3提供纯文本服务，配置服务器应用程序需要包括一个对Spring Cloud AWS的依赖。关于如何设置该依赖关系的详细信息，请参见《Spring Cloud AWS参考指南》。然后你需要配置Spring Cloud AWS，如《Spring Cloud AWS参考指南》中所述。

### 4.3. 解密文本

默认情况下，纯文本文件中的加密值不会被解密。为了启用纯文本文件的解密，请在 bootstrap 中设置 `spring.cloud.config.server.encrypt.enabled=true` 和 `spring.cloud.config.server.encrypt.plainTextEncrypt=true[yml|properties]` 。

> 解密纯文本文件只支持YAML、JSON和属性文件扩展名。

如果该功能被启用，并且请求一个不支持的文件扩展，文件中的任何加密值将不会被解密。

## 5. 嵌入配置服务器

配置服务器最好作为一个独立的应用程序运行。然而，如果需要的话，你可以把它嵌入到另一个应用程序中。要做到这一点，请使用`@EnableConfigServer`注解。在这种情况下，一个名为`spring.cloud.config.server.bootstrap`的可选属性会很有用。它是一个标志，表示服务器是否应该从自己的远程资源库配置自己。默认情况下，该标志是关闭的，因为它可能延迟启动。然而，当嵌入到另一个应用程序中时，以与其他应用程序相同的方式初始化是有意义的。当设置` spring.cloud.config.server.bootstrap` 为 true 时，你还必须使用复合环境资源库配置。例如

```yml
spring:
  application:
    name: configserver
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
          - type: native
            search-locations: ${HOME}/Desktop/config
        bootstrap: true
```

> ​	如果你使用 bootstrap 标志，配置服务器需要在`bootstrap.yml` 中配置其名称和版本库 URI。

要改变服务器端点的位置，你可以（选择性地）设置`spring.cloud.config.server.prefix`（例如`/config`），以在一个前缀下提供资源。该前缀应以`/`开头，但不以`/`结尾。它被应用于配置服务器中的`@RequestMappings`（即在`Spring Boot server.servletPath`和`server.contextPath`的前缀之下）。

如果你想直接从后端资源库（而不是从配置服务器）读取应用程序的配置，你基本上想要一个没有端点的嵌入式配置服务器。你可以通过不使用`@EnableConfigServer`注解（设置`spring.cloud.config.server.bootstrap=true`）完全关闭端点。

## 6. 推送通知和Spring Cloud Bus

许多源代码库提供商（如Github、Gitlab、Gitea、Gitee、Gogs或Bitbucket）通过webhook通知你库中的变化。你可以通过供应商的用户界面将`webhook`配置为一个URL和一组你感兴趣的事件。例如，Github使用POST向webhook发送一个包含提交列表的JSON主体和一个设置为`push`的头（`X-Github-Event`）。如果你添加了对`spring-cloud-config-monitor`库的依赖，并在你的配置服务器中激活了`Spring Cloud Bus`，那么就会启用一个`/monitor`端点。

当`webhook`被激活时，配置服务器会向它认为可能发生变化的应用程序发送一个`RefreshRemoteApplicationEvent`。变化检测可以是战略性的。然而，默认情况下，它寻找与应用程序名称相匹配的文件中的变化（例如`foo.properties`是针对`foo`应用程序的，而`application.properties`是针对所有应用程序的）。当你想覆盖该行为时，要使用的策略是`PropertyPathNotificationExtractor`，它接受请求头和正文作为参数，并返回一个变化的文件路径列表。

默认配置可以在Github、Gitlab、Gitea、Gitee、Gogs或Bitbucket中使用。除了来自Github、Gitlab、Gitee或Bitbucket的JSON通知外，你还可以通过向`/monitor`发送带有表单编码的主体参数的`path={application}`模式来触发变更通知。这样做是为了向与`{application}`模式相匹配的应用程序进行广播（可以包含通配符）。

> 只有当配置服务器和客户端应用程序中的`spring-cloud-bus`都被激活时，才会传输`RefreshRemoteApplicationEven`t。

> 默认配置也会检测本地git仓库的文件系统变化。在这种情况下，不使用webhook。然而，只要你编辑一个配置文件，就会有一个刷新广播。



## 7. Spring Cloud Config Client

Spring Boot应用程序可以立即利用Spring配置服务器（或由应用程序开发人员提供的其他外部属性源）。它还可以获得一些与环境变化事件有关的额外的有用功能。

### 7.1. Spring Boot Config 数据导入

Spring Boot 2.4引入了一种通过·spring.config.import·属性导入配置数据的新方法。现在这是与配置服务器绑定的默认方式。

要选择连接到配置服务器，请在application.properties中设置以下内容：

**application.properties**

```properties
spring.config.import=optional:configserver:
```

这将在 "http://localhost:8888 "的默认位置连接到配置服务器。删除可选的: 前缀将导致配置客户端在无法连接到配置服务器时失败。要改变配置服务器的位置，可以设置`spring.cloud.config.uri`或在`spring.config.import`语句中添加url，如`spring.config.import=optional:configserver:http://myhost:8888`。import属性中的位置优先于uri属性。

> 通过`spring.config.import`导入的Spring Boot配置数据方法不需要一个引导文件（properties或yaml）。

### 7.2. 配置首个BootStrap

要使用传统的引导方式连接到配置服务器，必须通过一个属性或 `spring-cloud-starter-bootstrap` 启动器启用引导。该属性是` spring.cloud.bootstrap.enabled=true`。它必须被设置为系统属性或环境变量。一旦启用`bootstrap`，`classpath`上有`Spring Cloud Config Client`的任何应用程序将按如下方式连接到配置服务器。当配置客户端启动时，它会绑定到配置服务器（通过`spring.cloud.config.uri bootstrap`配置属性）并使用远程属性源初始化Spring环境。

这种行为的结果是，所有想要使用配置服务器的客户端应用程序都需要一个`bootstrap.yml`（或一个环境变量），并在`spring.cloud.config.uri `中设置服务器地址（它默认为 "http://localhost:8888"）。

#### 7.2.1 Discovery 查找首个

> 除非你使用`config first bootstrap`，否则你需要在配置属性中设置`spring.config.import`属性。例如，`spring.config.import=optional:configserver:`。

如果你使用`DiscoveryClient`实现，如Spring Cloud Netflix和Eureka Service Discovery或Spring Cloud Consul，你可以让配置服务器向发现服务注册。

如果你喜欢使用`DiscoveryClient`来定位配置服务器，你可以通过设置`spring.cloud.config.discovery.enabled=true`（默认是false）来实现。例如，使用`Spring Cloud Netflix`，你需要定义Eureka服务器地址（例如，在`eureka.client.serviceUrl.defaultZone`中）。使用这个选项的代价是在启动时要进行额外的网络往返，以定位服务注册。其好处是，只要发现服务是一个固定点，配置服务器就可以改变其坐标。默认的服务ID是`configserver`，但你可以在客户端通过设置`spring.cloud.config.discovery.serviceId`来改变它（在服务器上，以服务的通常方式，如设置`spring.application.name`）。

发现客户端的实现都支持某种`metadata map`（例如，我们有`Eureka.instance.metadataMap`用于`Eureka`）。配置服务器的一些额外属性可能需要在其服务注册元数据中进行配置，以便客户端能够正确连接。如果配置服务器用`HTTP Basic`来保证安全，你可以将凭证配置为`user`和`password`。另外，如果配置服务器有一个上下文路径，你可以设置`configPath`。例如，下面的YAML文件是针对作为Eureka客户端的配置服务器的。

```yml
eureka:
  instance:
    ...
    metadataMap:
      user: osufhalskjrtl
      password: lviuhlszvaorhvlo5847
      configPath: /config
```

####  7.2.2. 使用Eureka和WebClient的第一次Bootstrap Discovery

如果你使用`Spring Cloud Netflix`的`Eureka DiscoveryClient`，并且想使用`WebClient`而不是`Jersey`或`RestTemplate`，你需要在`classpath`上包含`WebClient`，并设置`eureka.client.webclient.enabled=true`。

### 7.3. 配置客户端快速失效

在某些情况下，如果服务不能连接到配置服务器，你可能想让它启动失败。如果这是需要的行为，请设置`bootstrap`配置属性`spring.cloud.config.fail-fast=true`，使客户端以异常停止。

> 要使用`spring.config.import`获得类似的功能，只需省略`optional: `前缀。

### 7.4. 配置客户端重试

如果你预计配置服务器在你的应用程序启动时可能偶尔不可用，你可以让它在失败后继续尝试。首先，你需要设置 `spring.cloud.config.fail-fast=true`。然后你需要将 `spring-retry` 和 `spring-boot-starter-aop` 添加到你的 `classpath`。默认行为是重试六次，初始回退间隔为1000ms，后续回退的指数乘数为`1.1`。你可以通过设置 `spring.cloud.config.retry.* `配置属性来配置这些属性（以及其他属性）。

> 要想完全控制重试行为，并且使用传统的`bootstrap`，请添加一个`RetryOperationsInterceptor`类型的`@Bean`，ID为`configServerRetryInterceptor`。`Spring Retry`有一个`RetryInterceptorBuilder`，支持创建一个。

### 7.5. 用spring.config.import配置客户端重试

重试在`Spring Boot的spring.config.import`语句中起作用，正常的属性也起作用。但是，如果导入语句是在配置文件中，比如`application-prod.properties`，那么你需要用不同的方式来配置重试。配置需要作为url参数放在导入语句中。

**application-prod.properties**

```properties
spring.config.import=configserver:http://configserver.example.com?fail-fast=true&max-attempts=10&max-interval=1500&multiplier=1.2&
```

这将设置 `spring.cloud.config.fail-fast=true`（注意上面缺少前缀）和所有可用的 `spring.cloud.config.retry.*` 配置属性。

### 7.6. 定位远程配置资源

配置服务为来自`/{application}/{profile}/{label}`的属性源提供服务，其中客户端应用程序的默认绑定如下。

- "application" = `${spring.application.name}`
- "profile" = `${spring.profiles.active}` (actually `Environment.getActiveProfiles()`)
- "label" = "master"

> ​	当设置属性`${spring.application.name}`时，不要在你的应用程序名称前加上保留词`application-`，以防止在解决正确的属性源时出现问题。

你可以通过设置`spring.cloud.config`.*（其中*是名称、配置文件或标签）来覆盖所有的配置。标签对于回滚到以前版本的配置非常有用。在默认的配置服务器实现中，它可以是一个git标签、分支名称或提交ID。标签也可以以逗号分隔的列表形式提供。在这种情况下，列表中的项目会被逐一尝试，直到有一个成功。这种行为在处理一个特性分支时很有用。例如，你可能想让配置标签与你的分支保持一致，但让它成为可选项（在这种情况下，使用 `spring.cloud.config.label=myfeature,develop`）。

如果你在配置服务器上使用 HTTP 基本安全，目前只有在你将凭证嵌入您在 `spring.cloud.config.uri `属性下指定的每个 URL 中时，才可能支持每个配置服务器的认证凭证。如果你使用任何其他类型的安全机制，你（目前）无法支持每台配置服务器的认证和授权。

### 7.7. 配置超时

如果要配置超时阈值：

* 读取超时可以通过使用属性`Spring.cloud.config.Request-read-timeout`来配置。

* 可以通过使用属性`Spring.cloud.config.Request-Connect-Timeout`来配置连接超时。

### 7.8. 安全

如果你在服务器上使用`HTTP Basic`安全，客户需要知道密码（如果不是默认的，还需要知道用户名）。你可以通过配置服务器URI或通过单独的用户名和密码属性来指定用户名和密码，如下面的例子中所示。

```yml
spring:
  cloud:
    config:
     uri: https://user:secret@myconfig.mycompany.com
```

以下示例显示了传递相同信息的备用方式：

```yml
spring:
  cloud:
    config:
     uri: https://myconfig.mycompany.com
     username: user
     password: secret
```

`spring.cloud.config.password` 和 `spring.cloud.config.username` 的值会覆盖 URI 中提供的任何内容。

如果您在 `Cloud Foundry` 上部署应用程序，提供密码的最佳方式是通过服务凭证（如 URI 中，因为它不需要在配置文件中）。以下示例适用于本地和 `Cloud Foundry` 上用户提供的名为 `configserver` 的服务。

```yml
spring:
  cloud:
    config:
     uri: ${vcap.services.configserver.credentials.uri:http://user:password@localhost:8888}
```

如果`Config Server`需要客户端TLS证书，则可以通过属性配置客户端TLS证书和信任库，如下示例所示：

```yml
spring:
  cloud:
    config:
      uri: https://myconfig.myconfig.com
      tls:
        enabled: true
        key-store: <path-of-key-store>
        key-store-type: PKCS12
        key-store-password: <key-store-password>
        key-password: <key-password>
        trust-store: <path-of-trust-store>
        trust-store-type: PKCS12
        trust-store-password: <trust-store-password>
```

`spring.cloud.config.tls.enabled`需要为true，以启用配置客户端TLS。当`spring.cloud.config.tls.trust-store`被省略时，会使用JVM默认的信任存储。`spring.cloud.config.tls.key-store-type `和 `spring.cloud.config.tls.trust-store-type` 的默认值是 PKCS12。当密码属性被省略时，假定为空密码。

如果你使用另一种形式的安全，你可能需要向`ConfigServicePropertySourceLocator`提供一个`RestTemplate`（例如，通过在`bootstrap`上下文中抓取并注入它）。

#### 7.8.1. 健康指示器

配置客户端提供了一个Spring Boot健康指示器，试图从配置服务器加载配置。可以通过设置`health.config.enabled=false`来禁用该健康指标。出于性能考虑，响应也会被缓存起来。默认的缓存生存时间是5分钟。要改变该值，请设置`health.config.time-to-live`属性（单位：毫秒）。

#### 7.8.2. 提供自定义RestTemplate

在某些情况下，你可能需要定制从客户端向配置服务器发出的请求。通常情况下，这样做涉及到传递特殊的授权头信息来验证对服务器的请求。要提供一个自定义的`RestTemplate`。

创建一个具有PropertySourceLocator实现的新配置，如下例所示。

**CustomConfigServiceBootstrapConfiguration.java**

```java
@Configuration
public class CustomConfigServiceBootstrapConfiguration {
    @Bean
    public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
        ConfigClientProperties clientProperties = configClientProperties();
       ConfigServicePropertySourceLocator configServicePropertySourceLocator =  new ConfigServicePropertySourceLocator(clientProperties);
        configServicePropertySourceLocator.setRestTemplate(customRestTemplate(clientProperties));
        return configServicePropertySourceLocator;
    }
}
```

> 对于添加授权头的简化方法，可以使用 `spring.cloud.config.headers.*`属性来代替。

在`Resources / META-INf`中，创建一个名为`Spring.factories`的文件并指定您的自定义配置，如下示例所示：

**spring.factories**

```properties
org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
```

#### 7.8.3. Vault

当使用Vault作为配置服务器的后端时，客户端需要为服务器提供一个令牌，以便从Vault检索值。这个令牌可以通过在`bootstrap.yml`中设置`spring.cloud.config.token`在客户端中提供，如以下例子所示。

```yml
spring:
  cloud:
    config:
      token: YourVaultToken
```

#### 7.8.4. 嵌套Keys中的Vault

`Vault`支持在存储在`Vault`中的值中嵌套键的能力，如下面的例子所示。

```
echo -n '{"appA": {"secret": "appAsecret"}, "bar": "baz"}' | vault write secret/myapp -
```

这个命令会将一个JSON对象写入你的Vault中。要在Spring中访问这些值，你可以使用传统的dot(.)注解，如以下例子所示:

```java
@Value("${appA.secret}")
String name = "World";
```

前面的代码将设置`name`变量的值为`appAsecret`。
