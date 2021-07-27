# Spring Cloud Security

- 当前版本：2.2.5
- 修改时间：2021年7月25日
- 官方文档：[https://docs.spring.io/spring-cloud-security/docs/current/reference/html/](https://docs.spring.io/spring-cloud-security/docs/current/reference/html/)
- 源码仓库：[https://github.com/spring-cloud/spring-cloud-security](https://github.com/spring-cloud/spring-cloud-security)

Spring Cloud Security提供了一组原语来构建安全的应用程序和服务。一个可以在外部（或中央）进行大量配置的声明性模型，适合于实现由合作的远程组件组成的大型系统，通常有一个中央身份管理服务。它在Cloud Foundry这样的服务平台上也非常容易使用。在Spring Boot和Spring Security OAuth2的基础上，我们可以快速创建系统，实现单点登录、令牌中继和令牌交换等常见模式。

> 在未来的主要版本中，这个项目所包含的功能将转移到各自的项目中。

> Spring Cloud是在非限制性的Apache 2.0许可下发布的。如果你想为这部分文档做出贡献，或者发现错误，请在[github](https://github.com/spring-cloud/spring-cloud-security)的项目中找到源代码和问题跟踪器。

## 快速开始

这里有一个Spring Cloud的 "Hello World "应用，采用HTTP Basic认证和单点用户账户。

### OAuth2 单点登陆

**app.groovy**

```java
@Grab('spring-boot-starter-security')
@Controller
class Application {

  @RequestMapping('/')
  String home() {
    'Hello World'
  }

}
```

你可以用`spring run app.groovy`运行它，观察日志中的密码（用户名是 "user"）。到目前为止，这只是Spring Boot应用的默认情况。

这是一个带有Oauth2 sso的Spring Cloud 应用程序：

**app.groovy**

```java
@Controller
@EnableOAuth2Sso
class Application {

  @RequestMapping('/')
  String home() {
    'Hello World'
  }

}
```

发现区别了吗？这个应用的行为实际上与之前的应用完全一样，因为它还不知道自己的OAuth2凭证。

你可以很容易用github账号注册一个应用程序，所以如果你想在你自己的域名上生产一个应用程序，可以试试。如果你愿意在`localhost:8080`上测试，那么在你的应用程序配置中设置这些属性。

**application.yml**

```yml
security:
  oauth2:
    client:
      clientId: bd1c0a783ccdd1c9b9e4
      clientSecret: 1a9030fbca47a5b2c28e92f19050bb77824b5ad1
      accessTokenUri: https://github.com/login/oauth/access_token
      userAuthorizationUri: https://github.com/login/oauth/authorize
      clientAuthenticationScheme: form
    resource:
      userInfoUri: https://api.github.com/user
      preferTokenInfo: false
```

运行上面的应用程序，它将重定向到github进行授权。如果你已经登录了github，你甚至不会注意到它已经认证了。这些凭证只有在你的应用程序运行在8080端口时才会起作用。

为了限制客户端在获得访问令牌时要求的范围，你可以设置`security.oauth2.client.scope`（在YAML中逗号分隔或一个数组）。默认情况下，范围是空的，由授权服务器决定默认值，通常根据客户端注册中的设置。

> 上面的例子都是Groovy脚本。如果你想用Java（或Groovy）写同样的代码，你需要在classpath中添加Spring Security OAuth2（例如，见这里的示例）。

### OAuth2 受保护的资源

你想用OAuth2令牌来保护一个API资源？这里有一个简单的例子（与上面的客户端相配）。

**app.groovy**

```java
@Grab('spring-cloud-starter-security')
@RestController
@EnableResourceServer
class Application {

  @RequestMapping('/')
  def home() {
    [message: 'Hello World']
  }

}
```

和

**application.yml**

```yml
security:
  oauth2:
    resource:
      userInfoUri: https://api.github.com/user
      preferTokenInfo: false
```

## 更多详情

### 单点登陆

所有的OAuth2 SSO和资源服务器功能都在1.3版本中转移到Spring Boot。你可以在Spring Boot[用户指南](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)中找到文档。

### Token Relay

Token Relay是指OAuth2消费者作为客户端，将传入的令牌转发给传出的资源请求。消费者可以是一个纯粹的客户端（如SSO应用程序）或一个资源服务器。

#### 在Spring Cloud Gateway客户端中使用Token Relay

如果你的应用程序也有一个Spring Cloud Gateway反向代理应用程序，那么你可以要求它将OAuth2访问令牌转发到它所代理的服务下游。因此，上面的SSO应用可以像这样简单地增强。

**App.java**

```java
@Autowired
private TokenRelayGatewayFilterFactory filterFactory;

@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
            .route("resource", r -> r.path("/resource")
                    .filters(f -> f.filter(filterFactory.apply()))
                    .uri("http://localhost:9000"))
            .build();
}
```

或者这样

**application.yaml**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: resource
        uri: http://localhost:9000
        predicates:
        - Path=/resource
        filters:
        - TokenRelay=
```

它将（除了登录用户和抓取令牌外）把认证令牌传递给下游的服务（在这里是`/resource`）。

要为Spring Cloud Gateway启用这个功能，需要添加以下依赖项

- `org.springframework.boot:spring-boot-starter-oauth2-client`
- `org.springframework.cloud:spring-cloud-starter-security`

它是如何工作的？该过滤器从当前认证的用户中提取访问令牌，并将其放在下游请求的请求头中。

完整的工作样本见[该项目](https://github.com/spring-cloud-samples/sample-gateway-oauth2login)。

> `TokenRelayGatewayFilterFactory`使用的`ReactiveOAuth2AuthorizedClientService`的默认实现使用了一个内存数据存储。如果你需要一个更强大的解决方案，你将需要提供你自己的实现`ReactiveOAuth2AuthorizedClientService`。

#### 客户端 Token Relay

如果你的应用程序是一个面向用户的OAuth2客户端（即已声明`@EnableOAuth2Sso`或`@EnableOAuth2Client`），那么它在Spring Boot的请求范围内有一个`OAuth2ClientContext`。你可以从这个上下文和一个自动连接的`OAuth2ProtectedResourceDetails`中创建你自己的`OAuth2RestTemplate`，然后该上下文将一直向下游转发访问令牌，如果访问令牌过期，也会自动刷新。(这些都是Spring Security和Spring Boot的特性）。)

> 如果你使用`client_credentials`令牌，Spring Boot（1.4.1）不会自动创建一个`OAuth2ProtectedResourceDetails`。在这种情况下，你需要创建自己的`ClientCredentialsResourceDetails`，并用`@ConfigurationProperties("security.oauth2.client")`来配置它。

#### 在zuul代理客户端中使用Token Relay

如果你的应用程序也有一个`Spring Cloud Zuul`嵌入式反向代理（使用`@EnableZuulProxy`），那么你可以要求它将OAuth2访问令牌转发给它所代理的服务。因此，上面的SSO应用可以像这样简单地增强。

**app.groovy**

```java
@Controller
@EnableOAuth2Sso
@EnableZuulProxy
class Application {

}
```

它将（除了登录用户和获取令牌外）把认证令牌传递给下游的`/proxy/*`服务。如果这些服务是用`@EnableResourceServer`实现的，那么它们将在正确的头中得到一个有效的令牌。

它是如何工作的？`@EnableOAuth2Sso`注解拉入了`spring-cloud-starter-security`（你可以在传统应用中手动完成），这反过来又触发了`ZuulFilter`的一些自动配置，它本身被激活了，因为Zuul在classpath中（通过`@EnableZuulProxy`）。该过滤器只是从当前认证的用户中提取一个访问令牌，并将其放在下游请求的请求头中。

Spring Boot不会自动创建刷新令牌所需的`OAuth2RestOperations`。在这种情况下，你需要创建你自己的`OAuth2RestOperations`，以便`OAuth2TokenRelayFilter`在需要时可以刷新令牌。

#### 资源服务器中的Token Relay

如果你的应用程序有`@EnableResourceServer`，你可能想把传入的令牌转发到下游的其他服务。如果你使用`RestTemplate`来联系下游服务，那么这只是一个如何用正确的上下文创建模板的问题。

如果你的服务使用`UserInfoTokenServices`来验证传入的令牌（即使用`security.oauth2.user-info-uri`配置），那么你可以简单地使用自动连接的`OAuth2ClientContext`创建一个`OAuth2RestTemplate`（它将在进入后端代码之前由认证过程填充）。同样地（使用Spring Boot 1.4），你可以注入一个`UserInfoRestTemplateFactory`，并在配置中抓取其`OAuth2RestTemplate`。比如说。

**MyConfiguration.java**

```java
@Bean
public OAuth2RestTemplate restTemplate(UserInfoRestTemplateFactory factory) {
    return factory.getUserInfoRestTemplate();
}
```

然后，此REST模板将具有身份验证过滤器使用的相同的`OAuth2ClientContext`（请求范围），因此你可以使用它来发送具有相同访问令牌的请求。

如果你的应用程序没有使用`UserInfoTokenServices`，但仍然是一个客户端（即它声明了`@EnableOAuth2Client`或@`EnableOAuth2Sso`），那么通过Spring Security Cloud，用户从`@Autowired OAuth2Context`创建的任何`OAuth2RestOperations`也将转发令牌。这个功能默认是作为MVC处理程序拦截器实现的，所以它只在Spring MVC中工作。如果你不使用MVC，你可以使用自定义过滤器或AOP拦截器包装`AccessTokenContextRelay`来提供同样的功能。

这里有一个基本的例子，显示了在其他地方创建的自动连接REST模板的使用（"foo.com "是一个资源服务器，接受与周围应用程序相同的令牌）：

**MyController.java**

```java
@Autowired
private OAuth2RestOperations restTemplate;

@RequestMapping("/relay")
public String relay() {
    ResponseEntity<String> response =
      restTemplate.getForEntity("https://foo.com/bar", String.class);
    return "Success! (" + response.getBody() + ")";
}
```

如果你不想转发令牌（这是一个有效的选择，因为你可能想作为你自己，而不是向你发送令牌的客户端），那么你只需要创建自己的`OAuth2Context`，而不是自动连接默认的。

如果`OAuth2ClientContext`可用的话，Feign客户端也会拾取一个使用`OAuth2ClientContext`的拦截器，所以他们也应该在`RestTemplate`的任何地方做一个令牌中继。

## 配置Zuul Proxy下游的身份验证

你可以通过`proxy.auth.*`设置来控制`@EnableZuulProxy`下游的授权行为。例子：

**application.yml**

```yml
proxy:
  auth:
    routes:
      customers: oauth2
      stores: passthru
      recommendations: none
```

在这个例子中，"customers"服务得到一个OAuth2令牌中继，"stores"服务得到一个passthrough（授权头只是在下游传递），而 "recommendations"服务的授权头被删除。默认行为是，如果有可用的令牌，就进行令牌中继，否则就进行穿透。

详细情况见[ProxyAuthenticationProperties](https://github.com/spring-cloud/spring-cloud-security/tree/master/src/main/java/org/springframework/cloud/security/oauth2/proxy/ProxyAuthenticationProperties)。

{{#include ../license.md}}
