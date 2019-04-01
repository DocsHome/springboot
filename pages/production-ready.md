<a id="production-ready"></a>

# 五、Spring Boot Actuator: 生产就绪功能

Spring Boot 包含许多其他功能，可帮助您在将应用程序推送到生产环境时监控和管理应用程序。您可以选择使用 HTTP 端点或 JMX 来管理和监控应用程序。审计、健康和指标收集也可以自动应用于您的应用程序。

<a id="production-ready-enabling"></a>

## 52、启用生产就绪功能

[`spring-boot-actuator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator) 模块提供了 Spring Boot 的所有生产就绪功能。启用这些功能的最简单方法是添加 `spring-boot-starter-actuator` starter 到依赖中。

<blockquote>

**Actuator 的定义**

Actuator 是制造术语，指的是用于移动或控制某物的机械装置。Actuator 可以通过一个小的变化产生大量的运动。

</blockquote>

要将 actuator 添加到基于 Maven 的项目，请添加以下 starter 依赖项：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
</dependencies>
```

对于 Gradle，请使用以下声明：

```groovy
dependencies {
	compile("org.springframework.boot:spring-boot-starter-actuator")
}
```

<a id="production-ready-endpoints"></a>

## 53、端点

通过 Actuator 端点，您可以监控应用程序并与之交互。Spring Boot 包含许多内置端点，也允许您添加自己的端点。例如，`health` 端点提供基本的应用程序健康信息。

可以[启用或禁用](#production-ready-endpoints-enabling-endpoints)每个端点。它可控制当其 bean 存在于应用程序上下文中是否创建端点。要进行远程访问，必须[通过 JMX 或 HTTP 暴露端点](#production-ready-endpoints-exposing-endpoints)。大多数应用程序选择 HTTP 方式，端点的 ID 以及 `/actuator` 的前缀映射到一个 URL。例如，默认情况下，`health` 端点映射到 `/actuator/health`。

可以使用以下与技术无关的端点：

| ID | 描述 | 默认启用 |
| --- | --- | :---: |
| `auditevents` | 暴露当前应用程序的审计事件信息。 | 是 |
| `beans` | 显示应用程序中所有 Spring bean 的完整列表。 | 是 |
| `caches` | 暴露可用的缓存。 | 是 |
| `conditions` | 显示在配置和自动配置类上评估的条件以及它们匹配或不匹配的原因。 | 是 |
| `configprops` | 显示所有 `@ConfigurationProperties` 的校对清单。 | 是 | 
| `env` | 暴露 Spring `ConfigurableEnvironment` 中的属性。 | 是 |
| `flyway` | 显示已应用的 Flyway 数据库迁移。 | 是 |
| `health` | 显示应用程序健康信息 | 是 |
| `httptrace` | 显示 HTTP 追踪信息（默认情况下，最后 100 个 HTTP 请求/响应交换）。 | 是 |
| `info` |  显示应用程序信息。 | 是 |
| `integrationgraph` |  显示 Spring Integration 图。 | 是 |
| `loggers` |  显示和修改应用程序中日志记录器的配置。 | 是 |
| `liquibase` |  显示已应用的 Liquibase 数据库迁移。 | 是 |
| `metrics` |  显示当前应用程序的指标度量信息。 | 是 |
| `mappings` |  显示所有 `@RequestMapping` 路径的整理清单。 | 是 |
| `scheduledtasks` |  显示应用程序中的调度任务。 | 是 |
| `sessions` |  允许从 Spring Session 支持的会话存储中检索和删除用户会话。当使用 Spring Session 的响应式 Web 应用程序支持时不可用。 | 是 |
| `shutdown` |  正常关闭应用程序。 | 否 |
| `threaddump` |  执行线程 dump。 | 是 |

如果您的应用程序是 Web 应用程序（Spring MVC、Spring WebFlux 或 Jersey），则可以使用以下附加端点：

| ID | 描述 | 默认启用 |
| --- | --- | :---: |
| `heapdump` | 返回一个 `hprof` 堆 dump 文件。 | 是 |
| `jolokia` | 通过 HTTP 暴露 JMX bean（当 Jolokia 在 classpath 上时，不适用于 WebFlux）。
 | 是 |
| `logfile` | 返回日志文件的内容（如果已设置 `logging.file` 或 `logging.path` 属性）。支持使用 HTTP `Range` 头来检索部分日志文件的内容。 | 是 |
| `prometheus` | 以可以由 Prometheus 服务器抓取的格式暴露指标。 | 是 |

想了解有关 Actuator 端点及其请求和响应格式的更多信息，请参阅单独的 API 文档（[HTML](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/actuator-api//html) 或 [PDF](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）。

<a id="production-ready-endpoints-enabling-endpoints"></a>

### 53.1、启用端点

默认情况下，Actuator 启用除 `shutdown` 之外的所有端点。要配置端点的启用，请使用其 `management.endpoint.<id>.enabled` 属性。以下示例展示了如何启用关闭端点：

```ini
management.endpoint.shutdown.enabled=true
```

如果您希望端点启用是选择性加入而不是选择性退出，请将 `management.endpoints.enabled-by-default` 属性设置为 `false`，并使用各个端点的 `enabled` 属性重新加入。以下示例启用 `info` 端点并禁用所有其他端点：

```ini
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
```

**注意**

> 已完全从应用程序上下文中删除已禁用的端点。如果只想更改端点所暴露的技术，请改用 [`include` 和 `exclude` 属性](#production-ready-endpoints-exposing-endpoints)。

<a id="production-ready-endpoints-exposing-endpoints"></a>

### 53.2、暴露端点

由于端点可能包含敏感信息，因此应仔细考虑何时暴露它们。下表显示了内置端点和默认暴露情况：

| ID | JMX | Web |
| --- | --- | :---: |
| `auditevents` | 是 | 否 |
| `beans` | 是 | 否 |
| `caches` | 是 | 否 |
| `conditions` | 是 | 否 |
| `configprops` | 是 | 否 |
| `env` | 是 | 否 |
| `flyway` | 是 | 否 |
| `health` | 是 | 是 |
| `heapdump` | N/A | 否 |
| `httptrace` | 是 | 否 |
| `info` | 是 | 是 |
| `integrationgraph` | 是 | 否 |
| `jolokia` | N/A | 否 |
| `logfile` | N/A | 否 |
| `loggers` | 是 | 否 |
| `liquibase` | 是 | 否 |
| `metrics` | 是 | 否 |
| `mappings` | 是 | 否 |
| `prometheus` | N/A | 否 |
| `scheduledtasks` | 是 | 否 |
| `sessions` | 是 | 否 |
| `shutdown` | 是 | 否 |
| `threaddump` | 是 | 否 |

要更改暴露的端点，请使用以下特定的 `include` 和 `exclude` 属性：

| 属性 | 默认 |
| --- | --- | 
| `management.endpoints.jmx.exposure.exclude` | |
| `management.endpoints.jmx.exposure.include` | `*` |
| `management.endpoints.web.exposure.exclude` | |
| `management.endpoints.web.exposure.include` | `info, health` |

`include` 属性列出了暴露的端点的 ID。`exclude` 属性列出了不应暴露的端点的 ID。`exclude` 属性优先于 `include` 属性。可以使用端点 ID 列表配置 `include` 和 `exclude` 属性。

例如，要停止通过 JMX 暴露所有端点并仅暴露 `health` 和 `info` 端点，请使用以下属性：

```ini
management.endpoints.jmx.exposure.include=health,info
```

`*` 可用于选择所有端点。例如，要通过 HTTP 暴露除 `env` 和 `beans` 之外的所有端点，请使用以下属性：

```ini
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans
```

**注意**

<blockquote>

`*` 在 YAML 中具有特殊含义，因此如果要包含（或排除）所有端点，请务必添加引号，如下所示：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

</blockquote>

**注意**

> 如果您的应用程序是公开的，我们强烈建议您也[保护您的端点](#production-ready-endpoints-security)。

**提示**

> 如果要在暴露端点时实现自己的策略，可以注册一个 `EndpointFilter` bean。

<a id="production-ready-endpoints-security"></a>

### 53.3、保护 HTTP 端点

您应该像保护所有其他敏感 URL 一样注意保护 HTTP 端点。如果存在 Spring Security，则默认使用 Spring Security 的内容协商策略来保护端点。例如，如果您希望为 HTTP 端点配置自定义安全策略，只允许具有特定角色身份的用户访问它们，Spring Boot 提供了方便的 `RequestMatcher` 对象，可以与 Spring Security 结合使用。

典型的 Spring Security 配置可能如下：

```java
@Configuration
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests()
				.anyRequest().hasRole("ENDPOINT_ADMIN")
				.and()
			.httpBasic();
	}

}
```

上面的示例使用 `EndpointRequest.toAnyEndpoint()` 将请求与所有端点进行匹配，然后确保所有端点都具有 `ENDPOINT_ADMIN` 角色。`EndpointRequest` 上还提供了其他几种匹配器方法。有关详细信息，请参阅 API 文档（[HTML](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/actuator-api//html)或 [PDF](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）。

如果应用程序部署在有防火墙的环境，您可能希望无需身份验证即可访问所有 Actuator 端点。您可以通过更改 `management.endpoints.web.exposure.include` 属性来执行此操作，如下所示：

**application.properties**

```ini
management.endpoints.web.exposure.include=*
```

此外，如果存在 Spring Security，则需要添加自定义安全配置，以允许对端点进行未经身份验证的访问，如下所示：

```java
@Configuration
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests()
			.anyRequest().permitAll();
	}

}
```

<a id="production-ready-endpoints-caching"></a>

### 53.4、配置端点

端点对不带参数读取操作的响应自动缓存。要配置端点缓存响应的时间长度，请使用其 `cache.time-to-live` 属性。以下示例将beans端点缓存的生存时间设置为10秒：

**application.properties**

```ini
management.endpoint.beans.cache.time-to-live=10s
```

**注意**

> 前缀 `management.endpoint.<name>` 用于唯一标识配置的端点。

**注意**

> 在进行一个身份验证 HTTP 请求时，`Principal` 被视为端点的输入，因此不会缓存响应。

<a id="production-ready-endpoints-hypermedia"></a>

### 53.5、Actuator Web 端点超媒体

添加 **discovery page**，其包含指向所有端点的链接。默认情况下，**discovery page** 在 `/actuator` 上可访问。

配置一个自定义管理上下文（management context）路径时，**discovery page** 会自动从 `/actuator` 移动到管理上下文的根目录。例如，如果管理上下文路径是 `/management`，则可以从 `/management` 获取 discovery page。当管理上下文路径设置为 `/` 时，将禁用发现页面以防止与其他映射冲突。

<a id="production-ready-endpoints-cors"></a>

### 53.6、跨域支持

[跨源资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是一个 [W3C 规范](https://www.w3.org/TR/cors/)，允许您以灵活的方式指定授权的跨域请求类型。如果您使用 Spring MVC 或 Spring WebFlux，则可以配置 Actuator 的 Web 端点以支持此类方案。

默认情况下 CORS 支持被禁用，仅在设置了 `management.endpoints.web.cors.allowed-origins` 属性后才启用 CORS 支持。以下配置允许来自 `example.com` 域的 GET 和 POST 调用：

```ini
management.endpoints.web.cors.allowed-origins=http://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

**提示**

> 有关选项的完整列表，请参阅 [CorsEndpointProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/endpoint/web/CorsEndpointProperties.java)。

<a id="production-ready-endpoints-custom"></a>

### 53.7、实现自定义端点

如果您添加一个使用了 `@Endpoint` 注解的 `@Bean`，则使用 `@ReadOperation`、`@WritOperation` 或 `@DeleteOperation` 注解的所有方法都将通过 JMX 自动暴露，并且在 Web 应用程序中也将通过 HTTP 暴露。可以使用 Jersey、Spring MVC 或 Spring WebFlux 通过 HTTP 暴露端点。

您还可以使用 `@JmxEndpoint` 或 `@WebEndpoint` 编写特定技术的端点。这些端点仅限于各自的技术。例如，`@WebEndpoint` 仅通过 HTTP 暴露，而不是 JMX。

您可以使用 `@EndpointWebExtension` 和 `@EndpointJmxExtension` 编写特定技术的扩展。通过这些注解，您可以提供特定技术的操作来扩充现有端点。

最后，如果您需要访问特定 Web 框架的功能，则可以实现 Servlet 或 Spring `@Controller` 和 `@RestController` 端点，但代价是它们无法通过 JMX 或使用其他 Web 框架。

<a id="production-ready-endpoints-custom-input"></a>

#### 53.7.1、接收输入

端点上的操作通过参数接收输入。通过 Web 暴露时，这些参数的值取自 URL 的查询参数和 JSON 请求体。通过 JMX 暴露时，参数将映射到 MBean 操作的参数。默认情况下参数是必须的。可以使用 `@org.springframework.lang.Nullable` 对它们进行注解，使它们成为可选项。

JSON 请求体中的每个根属性都可以映射到端点的参数。考虑以下 JSON 请求体：

```json
{
	"name": "test",
	"counter": 42
}
```

这可用于调用带有 `String name` 和 `int counter` 参数的写操作。

**提示**

> 由于端点与技术无关，因此只能在方法签名中指定简单类型。特别是不支持使用定义一个 `name` 和 `counter` 属性的自定义类型声明单个参数。

**注意**

> 要允许将输入映射到操作方法的参数，应使用 `-parameters` 编译实现端点的 Java 代码，并且应使用 `-java-parameters` 编译实现端点的 Kotlin 代码。如果您使用的是 Spring Boot 的 Gradle 插件，或者是 Maven 和 spring-boot-starter-parent，则它们会自动执行此操作。

<a id="production-ready-endpoints-custom-input-conversion"></a>

##### 输入类型转换

如有必要，传递给端点操作方法的参数将自动转换为所需类型。在调用操作方法之前，使用 `ApplicationConversionService` 实例将通过 JMX 或 HTTP 请求接收的输入转换为所需类型。

<a id="production-ready-endpoints-custom-web"></a>

#### 53.7.2、自定义 Web 端点

`@Endpoint`、`@WebEndpoint` 或 `@EndpointWebExtension` 上的操作将使用 Jersey、Spring MVC 或 Spring WebFlux 通过 HTTP 自动暴露。

<a id="production-ready-endpoints-custom-web-predicate"></a>

##### Web 端点请求谓词

为 Web 暴露的端点上的每个操作自动生成请求谓词。

<a id="production-ready-endpoints-custom-web-predicate-path"></a>

##### 路径

谓词的路径由端点的 ID 和 Web 暴露的端点的基础路径确定。默认基础路径是 `/actuator`。例如，有 ID 为 `sessions` 的端点将使用 `/actuator/sessions` 作为其在谓词中的路径。

通过使用 `@Selector` 注解操作方法的一个或多个参数，可以进一步自定义路径。这样的参数作为路径变量添加到路径谓词中。调用端点操作时，变量的值将传递给操作方法。

<a id="production-ready-endpoints-custom-web-predicate-http-method"></a>

##### HTTP 方法

谓词的 HTTP 方法由操作类型决定，如下表所示：

| 操作 | HTTP 方法 |
| --- | --- |
| `@ReadOperation` | `GET` |
| `@WriteOperation` | `POST` |
| `@DeleteOperation` | `DELETE` |

<a id="production-ready-endpoints-custom-web-predicate-consumes"></a>

##### Consume

对于使用请求体的 `@WriteOperation`（HTTP `POST`），谓词的 consume 子句是 `application/vnd.spring-boot.actuator.v2+json, application/json`。对于所有其他操作，consume 子句为空。

<a id="production-ready-endpoints-custom-web-predicate-produces"></a>

##### Produce

谓词的 produce 子句可以由 `@DeleteOperation`、`@ReadOperation` 和 `@WriteOperation` 注解的 `produce` 属性确定。该属性是可选的。如果未使用，则自动确定 produce 子句。

如果操作方法返回 `void` 或 `Void`，则 produce 子句为空。如果操作方法返回 `org.springframework.core.io.Resource`，则 produce 子句为 `application/octet-stream`。对于所有其他操作，produce 子句是 `application/vnd.spring-boot.actuator.v2+json, application/json`。

<a id="production-ready-endpoints-custom-web-response-status"></a>

##### Web 端点响应状态

端点操作的默认响应状态取决于操作类型（读取、写入或删除）以及操作返回的内容（如果有）。

`@ReadOperation` 返回一个值，响应状态为 200（OK）。如果它未返回值，则响应状态将为 404（未找到）。

如果 `@WriteOperation` 或 `@DeleteOperation` 返回值，则响应状态将为 200（OK）。如果它没有返回值，则响应状态将为 204（无内容）。

如果在没有必需参数的情况下调用操作，或者使用无法转换为所需类型的参数，则不会调用操作方法，并且响应状态将为 400（错误请求）。

<a id="production-ready-endpoints-custom-web-range-requests"></a>

##### Web 端点范围请求

可用 HTTP 范围请求请求部分 HTTP 资源。使用 Spring MVC 或 Spring Web Flux 时，返回 `org.springframework.core.io.Resource` 的操作会自动支持范围请求。

**注意**

> 使用 Jersey 时不支持范围请求。

<a id="production-ready-endpoints-custom-web-security"></a>

##### Web 端点安全

Web 端点或特定 Web 的端点扩展上的操作可以接收当前的 `java.security.Principal` 或 `org.springframework.boot.actuate.endpoint.SecurityContext` 作为方法参数。前者通常与 `@Nullable` 结合使用，为经过身份验证和未经身份验证的用户提供不同的行为。后者通常用于使用其 `isUserInRole(String)` 方法执行授权检查。

<a id="production-ready-endpoints-custom-servlet"></a>

#### 53.7.3、Servlet 端点

通过实现一个带有 `@ServletEndpoint` 注解的类，`Servlet` 可以作为端点暴露，该类也实现了 `Supplier<EndpointServlet>`。Servlet 端点提供了与 Servlet 容器更深层次的集成，但代价是可移植性。它们旨在用于将现有 Servlet 作为端点暴露。对于新端点，应尽可能首选 `@Endpoint` 和 `@WebEndpoint` 注解。

<a id="production-ready-endpoints-custom-controller"></a>

#### 53.7.4、控制器端点

`@ControllerEndpoint` 和 `@RestControllerEndpoint` 可用于实现仅由 Spring MVC 或 Spring WebFlux 暴露的端点。使用 Spring MVC 和 Spring WebFlux 的标准注解（如 `@RequestMapping` 和 `@GetMapping`）映射方法，并将端点的 ID 用作路径的前缀。控制器端点提供了与 Spring 的 Web 框架更深层次的集成，但代价是可移植性。应尽可能首选 `@Endpoint` 和 `@WebEndpoint` 注解。

<a id="production-ready-health"></a>

### 53.8、健康信息

您可以使用健康信息来检查正在运行的应用程序的状态。监控软件经常在生产系统出现故障时使用它提醒某人。`health` 端点暴露的信息取决于 `management.endpoint.health.show-details` 属性，该属性可以使用以下值之一进行配置：

| 名称 | 描述 |
| --- | --- |
| `never` | 永远不会显示细节。 |
| `when-authorized` | 详细信息仅向授权用户显示。可以使用 `management.endpoint.health.roles` 配置授权角色。 |
| `always` | 向所有用户显示详细信息。 |

默认值为 `never`。当用户处于一个或多个端点的角色时，将被视为已获得授权。如果端点没有配置角色（默认值），则认为所有经过身份验证的用户都已获得授权。可以使用 `management.endpoint.health.roles` 属性配置角色。

**注意**

> 如果您已保护应用程序并希望使用 `always`，则安全配置必须允许经过身份验证和未经身份验证的用户对健康端点的访问。

健康信息是从 [`HealthIndicatorRegistry`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicatorRegistry.java) 的内容中收集的（默认情况下，`ApplicationContext` 中定义的所有 [`HealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) 实例）。Spring Boot 包含许多自动配置的 `HealthIndicators`，您也可以自己编写。默认情况下，最终系统状态由 `HealthAggregator` 根据状态的有序列表对每个 `HealthIndicator` 的状态进行排序。排序列表中的第一个状态作为整体健康状态。如果没有 `HealthIndicator` 返回一个 `HealthAggregator` 已知的状态，则使用 `UNKNOWN` 状态。

**提示**

> `HealthIndicatorRegistry` 可用于在运行时注册和注销健康指示器。

<a id="_auto_configured_healthindicators"></a>

#### 53.8.1、自动配置的 HealthIndicator

适当时，Spring Boot 会自动配置以下 `HealthIndicator`：

| 名称 | 描述 |
| --- | --- |
| [`CassandraHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.java) | 检查 Cassandra 数据库是否已启动。 |
| [`CouchbaseHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseHealthIndicator.java) | 检查 Couchbase 集群是否已启动。 |
| [`DiskSpaceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java) | 检查是否磁盘空间不足。 |
| [`DataSourceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.java) | 检查是否可以获得与 `DataSource` 的连接。 |
| [`ElasticsearchHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticsearchHealthIndicator.java) | 检查 Elasticsearch 集群是否已启动。 |
| [`InfluxDbHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/influx/InfluxDbHealthIndicator.java) | 检查 InfluxDB 服务器是否已启动。 |
| [`JmsHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java) | 检查 JMS broker 是否已启动。 |
| [`MailHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java) | 检查邮件服务器是否已启动。 |
| [`MongoHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java) | 检查 Mongo 数据库是否已启动。 |
| [`Neo4jHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/neo4j/Neo4jHealthIndicator.java) | 检查 Neo4j 服务器是否已启动。 |
| [`RabbitHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java) | 检查 Rabbit 服务器是否已启动。 |
| [`RedisHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java) | 检查 Redis 服务器是否已启动。 |
| [`SolrHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java) | 检查 Solr 服务器是否已启动。 |

**提示**

> 您可以通过设置 `management.health.defaults.enabled` 属性来禁用它们。

<a id="_writing_custom_healthindicators"></a>

#### 53.8.2、编写自定义 HealthIndicator

要提供自定义健康信息，可以注册实现 [`HealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) 接口的 Spring bean。您需要提供 `health()` 方法的实现并返回一个 `Health` 响应。`Health` 响应应包括一个状态，并且可以选择包括要显示的其他详细信息。以下代码展示了一个 HealthIndicator 实现示例：

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

	@Override
	public Health health() {
		int errorCode = check(); // 执行某些特定的健康检查
		if (errorCode != 0) {
			return Health.down().withDetail("Error Code", errorCode).build();
		}
		return Health.up().build();
	}

}
```

**注意**

> 给定 `HealthIndicator` 的标识符是没有 `HealthIndicator` 后缀的 bean 的名称（如果存在）。在前面的示例中，健康信息在名为 `my` 的条目中可用。

除了 Spring Boot 的预定义 [`Status`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java) 类型之外，`Health` 还可以返回一个表示新系统状态的自定义 `Status`。在这种情况下，还需要提供 [`HealthAggregator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthAggregator.java) 接口的自定义实现，或者必须使用 `management.health.status.order` 配置属性配置默认实现。

例如，假设在您的一个 `HealthIndicator` 实现中使用了代码为 `FATAL` 的新 `Status`。需要配置严重性顺序，请将以下属性添加到应用程序属性：

```ini
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UNKNOWN, UP
```

响应中的 HTTP 状态码反映了整体运行状况（例如，`UP` 映射到 200，而 `OUT_OF_SERVICE` 和 `DOWN` 映射到 503）。如果通过 HTTP 访问健康端点，则可能还需要注册自定义状态映射。例如，以下属性将 `FATAL` 映射到 503（服务不可用）：

```ini
management.health.status.http-mapping.FATAL=503
```

**提示**

> 如果需要控制更多，可以定义自己的 `HealthStatusHttpMapper` bean。

下表展示了内置状态的默认状态映射：

| 状态 | 映射 |
| --- | --- |
| DOWN | SERVICE_UNAVAILABLE (503) |
| OUT_OF_SERVICE | SERVICE_UNAVAILABLE (503) |
| UP | 默认没有映射，因此状态码为 200 |
| UNKNOWN | 默认没有映射，因此状态码为 200 |

<a id="reactive-health-indicators"></a>

#### 53.8.3、响应式健康指示器

对于响应式应用程序，例如使用 Spring WebFlux 的应用程序，`ReactiveHealthIndicator` 提供了一个非阻塞的接口来获取应用程序健康信息。与传统的 `HealthIndicator` 类似，健康信息从 [`ReactiveHealthIndicatorRegistry`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicatorRegistry.java) 的内容中收集（默认情况下，`ApplicationContext` 中定义的所有 [`HealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) 和 [`ReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java) 实例）。不检查响应式 API 的常规 `HealthIndicator`在弹性调度程序上执行。

**提示**

> 在响应式应用程序中，`ReactiveHealthIndicatorRegistry` 可用于在运行时注册和取消注册健康指示器。

要从响应式 API 提供自定义健康信息，可以注册实现 [`ReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java) 接口的 Spring bean。以下代码展示了 `ReactiveHealthIndicator` 实现的示例：

```java
@Component
public class MyReactiveHealthIndicator implements ReactiveHealthIndicator {

	@Override
	public Mono<Health> health() {
		return doHealthCheck() // 执行一些特定的健康检查并返回一个 Mono<Health>
			.onErrorResume(ex -> Mono.just(new Health.Builder().down(ex).build())));
	}

}
```

**提示**

> 要自动处理错误，请考虑从 `AbstractReactiveHealthIndicator` 进行扩展。

<a id="_auto_configured_reactivehealthindicators"></a>

#### 53.8.4、自动配置的 ReactiveHealthIndicator

适当时，Spring Boot 会自动配置以下 `ReactiveHealthIndicator`：

| 名称 | 描述 |
| --- | --- |
| [`CassandraReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraReactiveHealthIndicator.java) | 检查 Cassandra 数据库是否已启动。 |
| [`CouchbaseReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseReactiveHealthIndicator.java) | 检查 Couchbase 集群是否已启动。 |
| [`MongoReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoReactiveHealthIndicator.java) | 检查 Mongo 数据库是否已启动。 |
| [`RedisReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisReactiveHealthIndicator.java) | 检查 Redis 服务器是否已启动。 |

**提示**

> 必要时，响应式指示器取代常规指示器。此外，任何未明确处理的 `HealthIndicator` 都会自动换行。

<a id="production-ready-application-info"></a>

### 53.9、应用程序信息

应用程序信息暴露从 `ApplicationContext` 中定义的所有 [`InfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java) bean 收集的各种信息。Spring Boot 包含许多自动配置的 `InfoContributor` bean，您可以编写自己的 bean。

<a id="production-ready-application-info"></a>

#### 53.9.1、自动配置的 InfoContributor

适当时，Spring Boot 会自动配置以下 `InfoContributor` bean：

| 名称 | 描述 |
| --- | --- |
| [`EnvironmentInfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/EnvironmentInfoContributor.java) | 在 `info` key 下显示 `Environment` 中的所有 key。 |
| [`GitInfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/GitInfoContributor.java) | 如果 `git.properties` 可用则暴露 git 信息。 |
| [`BuildInfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/BuildInfoContributor.java) | 如果 `META-INF/build-info.properties` 可用则暴露构建信息。 |

**提示**

> 可以通过设置 `management.info.defaults.enabled` 属性来禁用它们。

<a id="production-ready-application-info-env"></a>

#### 53.9.2、自定义应用程序信息

您可以通过设置 `info.*` 字符串属性来自定义 `info`端点暴露的数据。`info` key 下的所有 `Environment` 属性都会自动暴露。例如，您可以将以下设置添加到 `application.properties` 文件中：

```ini
info.app.encoding=UTF-8
info.app.java.source=1.8
info.app.java.target=1.8
```

**提示**

<blockquote>

您可以在[构建时扩展 info 属性](how-to.md#howto-automatic-expansion)，而不是对这些值进行硬编码。
假设您使用 Maven，您可以按如下方式重写前面的示例：

```ini
info.app.encoding=@project.build.sourceEncoding@
info.app.java.source=@java.version@
info.app.java.target=@java.version@
```

</blockquote>

<a id="production-ready-application-info-git"></a>

#### 53.9.3、Git 提交信息

`info` 端点的另一个有用功能是它能够在构建项目时发布 git 源码仓库相关的状态的信息。如果 `GitProperties` bean 可用，则暴露 `git.branch`、`git.commit.id` 和 `git.commit.time` 属性。

**提示**

> 如果 `git.properties` 文件在 classpath 的根目录中可用，则会自动配置 `GitProperties` bean。有关更多详细信息，请参阅[**生成 git 信息**](how-to.md#howto-git-info)。

如果要显示完整的 git 信息（即 `git.properties` 的完整内容），请使用 `management.info.git.mode` 属性，如下所示：

```ini
management.info.git.mode=full
```

<a id="production-ready-application-info-build"></a>

#### 53.9.4、构建信息

如果 `BuildProperties` bean 可用，则 `info` 端点还可以发布构建相关的信息。如果 classpath 中有 `META-INF/build-info.properties` 文件，则会发生这种情况。

**提示**

> Maven 和 Gradle 插件都可以生成该文件。有关更多详细信息，请参阅[**生成构建信息**](how-to.md#howto-build-info)。

<a id="production-ready-application-info-build"></a>

#### 53.9.5、编写自定义 InfoContributor

要提供自定义应用程序信息，可以注册实现 [`InfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java) 接口的 Spring bean。

以下示例提供了具有单个值的 `example` 条目：

```java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

	@Override
	public void contribute(Info.Builder builder) {
		builder.withDetail("example",
				Collections.singletonMap("key", "value"));
	}

}
```

如果访问 `info` 端点，您应该能看到包含以下附加条目的响应：

```json
{
	"example": {
		"key" : "value"
	}
}
```

**待续……**