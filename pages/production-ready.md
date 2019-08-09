<a id="production-ready"></a>

# 五、Spring Boot Actuator: 生产就绪功能

Spring Boot 包含许多其他功能，可帮助你在将应用程序推送到生产环境时监控和管理应用程序。你可以选择使用 HTTP 端点或 JMX 来管理和监控应用程序。审计、健康和指标收集也可以自动应用于你的应用程序。

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

通过 Actuator 端点，你可以监控应用程序并与之交互。Spring Boot 包含许多内置端点，也允许你添加自己的端点。例如，`health` 端点提供基本的应用程序健康信息。

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

如果你的应用程序是 Web 应用程序（Spring MVC、Spring WebFlux 或 Jersey），则可以使用以下附加端点：

| ID | 描述 | 默认启用 |
| --- | --- | :---: |
| `heapdump` | 返回一个 `hprof` 堆 dump 文件。 | 是 |
| `jolokia` | 通过 HTTP 暴露 JMX bean（当 Jolokia 在 classpath 上时，不适用于 WebFlux）。 | 是 |
| `logfile` | 返回日志文件的内容（如果已设置 `logging.file` 或 `logging.path` 属性）。支持使用 HTTP `Range` 头来检索部分日志文件的内容。 | 是 |
| `prometheus` | 以可以由 Prometheus 服务器抓取的格式暴露指标。 | 是 |

想了解有关 Actuator 端点及其请求和响应格式的更多信息，请参阅单独的 API 文档（[HTML](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/actuator-api//html) 或 [PDF](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）。

<a id="production-ready-endpoints-enabling-endpoints"></a>

### 53.1、启用端点

默认情况下，Actuator 启用除 `shutdown` 之外的所有端点。要配置端点的启用，请使用其 `management.endpoint.<id>.enabled` 属性。以下示例展示了如何启用关闭端点：

```ini
management.endpoint.shutdown.enabled=true
```

如果你希望端点启用是选择性加入而不是选择性退出，请将 `management.endpoints.enabled-by-default` 属性设置为 `false`，并使用各个端点的 `enabled` 属性重新加入。以下示例启用 `info` 端点并禁用所有其他端点：

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

> 如果你的应用程序是公开的，我们强烈建议你也[保护你的端点](#production-ready-endpoints-security)。

**提示**

> 如果要在暴露端点时实现自己的策略，可以注册一个 `EndpointFilter` bean。

<a id="production-ready-endpoints-security"></a>

### 53.3、保护 HTTP 端点

你应该像保护所有其他敏感 URL 一样注意保护 HTTP 端点。如果存在 Spring Security，则默认使用 Spring Security 的内容协商策略来保护端点。例如，如果你希望为 HTTP 端点配置自定义安全策略，只允许具有特定角色身份的用户访问它们，Spring Boot 提供了方便的 `RequestMatcher` 对象，可以与 Spring Security 结合使用。

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

如果应用程序部署在有防火墙的环境，你可能希望无需身份验证即可访问所有 Actuator 端点。你可以通过更改 `management.endpoints.web.exposure.include` 属性来执行此操作，如下所示：

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

[跨源资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是一个 [W3C 规范](https://www.w3.org/TR/cors/)，允许你以灵活的方式指定授权的跨域请求类型。如果你使用 Spring MVC 或 Spring WebFlux，则可以配置 Actuator 的 Web 端点以支持此类方案。

默认情况下 CORS 支持被禁用，仅在设置了 `management.endpoints.web.cors.allowed-origins` 属性后才启用 CORS 支持。以下配置允许来自 `example.com` 域的 GET 和 POST 调用：

```ini
management.endpoints.web.cors.allowed-origins=http://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

**提示**

> 有关选项的完整列表，请参阅 [CorsEndpointProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/endpoint/web/CorsEndpointProperties.java)。

<a id="production-ready-endpoints-custom"></a>

### 53.7、实现自定义端点

如果你添加一个使用了 `@Endpoint` 注解的 `@Bean`，则使用 `@ReadOperation`、`@WritOperation` 或 `@DeleteOperation` 注解的所有方法都将通过 JMX 自动暴露，并且在 Web 应用程序中也将通过 HTTP 暴露。可以使用 Jersey、Spring MVC 或 Spring WebFlux 通过 HTTP 暴露端点。

你还可以使用 `@JmxEndpoint` 或 `@WebEndpoint` 编写特定技术的端点。这些端点仅限于各自的技术。例如，`@WebEndpoint` 仅通过 HTTP 暴露，而不是 JMX。

你可以使用 `@EndpointWebExtension` 和 `@EndpointJmxExtension` 编写特定技术的扩展。通过这些注解，你可以提供特定技术的操作来扩充现有端点。

最后，如果你需要访问特定 Web 框架的功能，则可以实现 Servlet 或 Spring `@Controller` 和 `@RestController` 端点，但代价是它们无法通过 JMX 或使用其他 Web 框架。

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

> 要允许将输入映射到操作方法的参数，应使用 `-parameters` 编译实现端点的 Java 代码，并且应使用 `-java-parameters` 编译实现端点的 Kotlin 代码。如果你使用的是 Spring Boot 的 Gradle 插件，或者是 Maven 和 spring-boot-starter-parent，则它们会自动执行此操作。

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

你可以使用健康信息来检查正在运行的应用程序的状态。监控软件经常在生产系统出现故障时使用它提醒某人。`health` 端点暴露的信息取决于 `management.endpoint.health.show-details` 属性，该属性可以使用以下值之一进行配置：

| 名称 | 描述 |
| --- | --- |
| `never` | 永远不会显示细节。 |
| `when-authorized` | 详细信息仅向授权用户显示。可以使用 `management.endpoint.health.roles` 配置授权角色。 |
| `always` | 向所有用户显示详细信息。 |

默认值为 `never`。当用户处于一个或多个端点的角色时，将被视为已获得授权。如果端点没有配置角色（默认值），则认为所有经过身份验证的用户都已获得授权。可以使用 `management.endpoint.health.roles` 属性配置角色。

**注意**

> 如果你已保护应用程序并希望使用 `always`，则安全配置必须允许经过身份验证和未经身份验证的用户对健康端点的访问。

健康信息是从 [`HealthIndicatorRegistry`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicatorRegistry.java) 的内容中收集的（默认情况下，`ApplicationContext` 中定义的所有 [`HealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) 实例）。Spring Boot 包含许多自动配置的 `HealthIndicators`，你也可以自己编写。默认情况下，最终系统状态由 `HealthAggregator` 根据状态的有序列表对每个 `HealthIndicator` 的状态进行排序。排序列表中的第一个状态作为整体健康状态。如果没有 `HealthIndicator` 返回一个 `HealthAggregator` 已知的状态，则使用 `UNKNOWN` 状态。

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

> 你可以通过设置 `management.health.defaults.enabled` 属性来禁用它们。

<a id="_writing_custom_healthindicators"></a>

#### 53.8.2、编写自定义 HealthIndicator

要提供自定义健康信息，可以注册实现 [`HealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) 接口的 Spring bean。你需要提供 `health()` 方法的实现并返回一个 `Health` 响应。`Health` 响应应包括一个状态，并且可以选择包括要显示的其他详细信息。以下代码展示了一个 HealthIndicator 实现示例：

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

例如，假设在你的一个 `HealthIndicator` 实现中使用了代码为 `FATAL` 的新 `Status`。需要配置严重性顺序，请将以下属性添加到应用程序属性：

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

> 必要时，响应式指示器取代常规指示器。此外，任何未明确处理的 `HealthIndicator` 都会自动包装。

<a id="production-ready-application-info"></a>

### 53.9、应用程序信息

应用程序信息暴露从 `ApplicationContext` 中定义的所有 [`InfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java) bean 收集的各种信息。Spring Boot 包含许多自动配置的 `InfoContributor` bean，你可以编写自己的 bean。

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

你可以通过设置 `info.*` 字符串属性来自定义 `info`端点暴露的数据。`info` key 下的所有 `Environment` 属性都会自动暴露。例如，你可以将以下设置添加到 `application.properties` 文件中：

```ini
info.app.encoding=UTF-8
info.app.java.source=1.8
info.app.java.target=1.8
```

**提示**

<blockquote>

你可以在[构建时扩展 info 属性](how-to.md#howto-automatic-expansion)，而不是对这些值进行硬编码。
假设你使用 Maven，你可以按如下方式重写前面的示例：

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

如果访问 `info` 端点，你应该能看到包含以下附加条目的响应：

```json
{
	"example": {
		"key" : "value"
	}
}
```

<a id="production-ready-monitoring"></a>

## 54、通过 HTTP 监控和管理

如果你正在开发 Web 应用程序，Spring Boot Actuator 会自动配置所有已启用的端点以通过 HTTP 暴露。默认约定是使用前缀为 `/actuator` 的端点的 `id` 作为 URL 路径。例如，`health` 以 `/actuator/health` 暴露。提示：Spring MVC、Spring WebFlux 和 Jersey 本身支持 Actuator。

<a id="production-ready-customizing-management-server-context-path"></a>

### 54.1、自定义 Management 端点路径

有时，自定义 management 端点的前缀很有用。例如，你的应用程序可能已将 `/actuator` 用于其他目的。你可以使用 `management.endpoints.web.base-path` 属性更改 management 端点的前缀，如下所示：

```ini
management.endpoints.web.base-path=/manage
```

前面的 `application.properties` 示例将端点从 `/actuator/{id}` 更改为 `/manage/{id}`（例如，`/manage/info`）。

**注意**

> 除非已将 management 端口配置为[使用其他 HTTP 端口暴露端点](production-ready-customizing-management-server-port)，否则 `management.endpoints.web.base-path` 与 `server.servlet.context-path` 相关联。如果配置了 `management.server.port`，则 `management.endpoints.web.base-path` 与 `management.server.servlet.context-path` 相关联。

如果要将端点映射到其他路径，可以使用 `management.endpoints.web.path-mapping` 属性。

以下示例将 `/actuator/health` 重新映射到 `/healthcheck`：

**application.properties**

```ini
management.endpoints.web.base-path=/
management.endpoints.web.path-mapping.health=healthcheck
```

<a id="production-ready-customizing-management-server-port"></a>

### 54.2、自定义 Management 服务器端口

使用默认 HTTP 端口暴露 management 端点是基于云部署的明智选择。但是，如果应用程序是在自己的数据中心内运行，你可能更喜欢使用其他 HTTP 端口暴露端点。

你可以设置 `management.server.port` 属性以更改 HTTP 端口，如下所示：

```ini
management.server.port=8081
```

<a id="production-ready-management-specific-ssl"></a>

### 54.3、配置 Management 的 SSL

当配置为使用自定义端口时，还可以使用各种 `management.server.ssl.*` 属性为 management 服务器配置自己的 SSL。例如，这样做可以在主应用程序使用 HTTPS 时可通过 HTTP 使用 management 服务器，如以下属性设置所示：

```ini
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:store.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=false
```

或者，主服务器和 management 服务器都可以使用 SSL，但他们的 key store 不同，如下所示：

```ini
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:main.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=true
management.server.ssl.key-store=classpath:management.jks
management.server.ssl.key-password=secret
```

<a id="production-ready-customizing-management-server-address"></a>

### 54.4、配置 Management 服务器地址

你可以通过设置 `management.server.address` 属性来自定义 management 端点可用的地址。如果你只想在内部或操作的网络上监听或仅监听来自 `localhost` 的连接，那么这样做会非常有用。

**注意**

> 仅当端口与主服务器端口不同时，才能监听不同的地址。

以下 `application.properties` 示例不允许远程连接 management：

```ini
management.server.port=8081
management.server.address=127.0.0.1
```

<a id="production-ready-disabling-http-endpoints"></a>

### 54.5、禁用 HTTP 端点

如果你不希望通过 HTTP 暴露端点，则可以将 management 端口设置为 `-1`，如下所示：

```ini
management.server.port=-1
```

这可以使用 `management.endpoints.web.exposure.exclude` 属性来实现，如下所示：

```ini
management.endpoints.web.exposure.exclude=*
```

<a id="production-ready-jmx"></a>

## 55、通过 JMX 监控和管理

Java 管理扩展（Java Management Extensions，JMX）提供了一种监控和管理应用程序的标准机制。默认情况下，Spring Boot 将 management 端点暴露为 `org.springframework.boot` 域下的 JMX MBean。

<a id="production-ready-custom-mbean-names"></a>

### 55.1、自定义 MBean 名称

MBean 的名称通常是从端点的 `id` 生成的。例如，`health` 端点公开为 `org.springframework.boot:type=Endpoint,name=Health`。

如果你的应用程序包含多个 Spring `ApplicationContext`，可能会发生名称冲突。要解决此问题，可以将 `spring.jmx.unique-names` 属性设置为 `true`，以保证 MBean 名称始终唯一。

你还可以自定义暴露端点的 JMX 域。以下设置展示了在 `application.properties` 中执行此操作的示例：

```ini
spring.jmx.unique-names=true
management.endpoints.jmx.domain=com.example.myapp
```

<a id="production-ready-disable-jmx-endpoints"></a>

### 55.2、禁用 JMX 端点

如果你不想通过 JMX 暴露端点，可以将 `management.endpoints.jmx.exposure.exclude` 属性设置为 `*`，如下所示：

```ini
management.endpoints.jmx.exposure.exclude=*
```

<a id="production-ready-jolokia"></a>

### 55.3、通过 HTTP 使用 Jolokia 访问 JMX

Jolokia 是一个 JMX-HTTP 桥，它提供了一种访问 JMX bean 的新方式。要使用 Jolokia，请引入依赖：`org.jolokia:jolokia-core`。例如，使用 Maven，你将添加以下依赖：

```xml
<dependency>
	<groupId>org.jolokia</groupId>
	<artifactId>jolokia-core</artifactId>
</dependency>
```

之后可以通过将 `jolokia` 或 `*` 添加到 `management.endpoints.web.exposure.include` 属性来暴露 Jolokia 端点。最后，你可以在 management HTTP 服务器上使用 `/actuator/jolokia` 访问它。

<a id="production-ready-customizing-jolokia"></a>

#### 55.3.1、自定义 Jolokia

Jolokia 有许多设置，你可以通过设置 servlet 参数来使用传统方式进行配置。使用 Spring Boot 时，你可以使用 `application.properties` 文件配置。请在参数前加上 `management.endpoint.jolokia.config`。如下所示：

```ini
management.endpoint.jolokia.config.debug=true
```

<a id="production-ready-disabling-jolokia"></a>

#### 55.3.2、禁用 Jolokia

如果你使用 Jolokia 但不希望 Spring Boot 配置它，请将 `management.endpoint.jolokia.enabled` 属性设置为 `false`，如下所示：

```ini
management.endpoint.jolokia.enabled=false
```

<a id="production-ready-loggers"></a>

## 56、日志记录器

Spring Boot Actuator 有可在运行时查看和配置应用程序日志级别的功能。你可以查看全部或单个日志记录器的配置，该配置由显式配置的日志记录级别以及日志记录框架为其提供的有效日志记录级别组成。这些级别可以是以下之一：

- `TRACE`
- `DEBUG`
- `INFO`
- `WARN`
- `ERROR`
- `FATAL`
- `OFF`
- `null`

`null` 表示没有显式配置。

<a id="production-ready-logger-configuration"></a>

### 56.1、配置一个日志记录器

要配置日志记录器，请将部分实体 `POST` 到资源的 URI，如下所示：

```json
{
	"configuredLevel": "DEBUG"
}
```

**提示**

> 要**重置**日志记录器的特定级别（并使用默认配置代替），可以将 `null` 值作为 `configuredLevel` 传递。

<a id="production-ready-metrics"></a>

## 57、指标

Spring Boot Actuator 为 [Micrometer](https://micrometer.io/) 提供了依赖管理和自动配置，Micrometer 是一个支持众多监控系统的应用程序指标门面，包括：

- [AppOptics](#production-ready-metrics-export-appoptics)
- [Atlas](#production-ready-metrics-export-atlas)
- [Datadog](#production-ready-metrics-export-datadog)
- [Dynatrace](#production-ready-metrics-export-dynatrace)
- [Elastic](#production-ready-metrics-export-elastic)
- [Ganglia](#production-ready-metrics-export-ganglia)
- [Graphite](#production-ready-metrics-export-graphite)
- [Humio](#production-ready-metrics-export-humio)
- [Influx](#production-ready-metrics-export-influx)
- [JMX](#production-ready-metrics-export-jmx)
- [KairosDB](#production-ready-metrics-export-kairos)
- [New Relic](#production-ready-metrics-export-newrelic)
- [Prometheus](#production-ready-metrics-export-prometheus)
- [SignalFx](#production-ready-metrics-export-signalfx)
- [Simple (in-memory)](#production-ready-metrics-export-simple)
- [StatsD](#production-ready-metrics-export-statsd)
- [Wavefront](#production-ready-metrics-export-wavefront)

**提示**

要了解有关 Micrometer 功能的更多信息，请参阅其[参考文档](https://micrometer.io/docs)，特别是[概念部分](https://micrometer.io/docs/concepts)。

<a id="production-ready-metrics-getting-started"></a>

### 57.1、入门

Spring Boot 自动配置了一个组合的 `MeterRegistry`，并为 classpath 中每个受支持的实现向该组合注册一个注册表。在运行时，只需要 classpath 中有 `micrometer-registry-{system}` 依赖即可让 Spring Boot 配置该注册表。

大部分注册表都有共同点 例如，即使 Micrometer 注册实现位于 classpath 上，你也可以禁用特定的注册表。例如，要禁用 Datadog：

```ini
management.metrics.export.datadog.enabled=false
```

Spring Boot 还会将所有自动配置的注册表添加到 `Metrics` 类的全局静态复合注册表中，除非你明确禁止：

```ini
management.metrics.use-global-registry=false
```

在注册表中注册任何指标之前，你可以注册任意数量的 `MeterRegistryCustomizer` bean 以进一步配置注册表，例如通用标签：

```java
@Bean
MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
	return registry -> registry.config().commonTags("region", "us-east-1");
}
```

你可以通过指定泛型类型，自定义注册表实现：

```java
@Bean
MeterRegistryCustomizer<GraphiteMeterRegistry> graphiteMetricsNamingConvention() {
	return registry -> registry.config().namingConvention(MY_CUSTOM_CONVENTION);
}
```

使用该设置，你可以在组件中注入 `MeterRegistry` 并注册指标：

```java
@Component
public class SampleBean {

	private final Counter counter;

	public SampleBean(MeterRegistry registry) {
		this.counter = registry.counter("received.messages");
	}

	public void handleMessage(String message) {
		this.counter.increment();
		// 处理消息实现
	}

}
```

Spring Boot 还[配置内置的测量工具](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#production-ready-metrics-meter)（即 `MeterBinder` 实现），你可以通过配置或专用注解标记来控制。

<a id="production-ready-metrics-export"></a>

### 57.2、支持的监控系统

<a id="production-ready-metrics-export-appoptics"></a>

#### 57.2.1、AppOptics

默认情况下，AppOptics 注册表会定期将指标推送到 [api.appoptics.com/v1/measurements](https://api.appoptics.com/v1/measurements)。要将指标导出到 SaaS [AppOptics](https://micrometer.io/docs/registry/appoptics)，你必须提供 API 令牌：

```ini
management.metrics.export.appoptics.api-token=YOUR_TOKEN
```

<a id="production-ready-metrics-export-atlas"></a>

#### 57.2.2、Atlas

默认情况下，度量标准将导出到本地的 [Atlas](https://micrometer.io/docs/registry/atlas)。可以使用以下方式指定 [Atlas 服务器](https://github.com/Netflix/atlas)的位置：

```ini
management.metrics.export.atlas.uri=https://atlas.example.com:7101/api/v1/publish
```

<a id="production-ready-metrics-export-datadog"></a>

#### 57.2.3、Datadog

Datadog 注册表会定期将指标推送到 [datadoghq](https://www.datadoghq.com/)。要将指标导出到 [Datadog](https://micrometer.io/docs/registry/datadog)，你必须提供 API 密钥：

```ini
management.metrics.export.datadog.api-key=YOUR_KEY
```

你还可以更改度量标准发送到 Datadog 的间隔时间：

```ini
management.metrics.export.datadog.step=30s
```

<a id="production-ready-metrics-export-dynatrace"></a>

#### 57.2.4、Dynatrace

Dynatrace 注册表定期将指标推送到配置的 URI。要将指标导出到 [Dynatrace](https://micrometer.io/docs/registry/dynatrace)，必须提供 API 令牌、设备 ID 和 URI：

```ini
management.metrics.export.dynatrace.api-token=YOUR_TOKEN
management.metrics.export.dynatrace.device-id=YOUR_DEVICE_ID
management.metrics.export.dynatrace.uri=YOUR_URI
```

你还可以更改度量标准发送到 Dynatrace 的间隔时间：

```ini
management.metrics.export.dynatrace.step=30s
```

<a id="production-ready-metrics-export-elastic"></a>

#### 57.2.5、Elastic

默认情况下，度量将导出到本地的 [Elastic](https://micrometer.io/docs/registry/elastic)。可以使用以下属性提供 Elastic 服务器的位置：

```ini
management.metrics.export.elastic.host=https://elastic.example.com:8086
```

<a id="production-ready-metrics-export-ganglia"></a>

#### 57.2.6、Ganglia

默认情况下，度量将导出到本地的 [Ganglia](https://micrometer.io/docs/registry/ganglia)。可以使用以下方式提供 [Ganglia 服务器](http://ganglia.sourceforge.net/)主机和端口：

```ini
management.metrics.export.ganglia.host=ganglia.example.com
management.metrics.export.ganglia.port=9649
```

<a id="production-ready-metrics-export-graphite"></a>

#### 57.2.7、Graphite

默认情况下，度量将导出到本地的 [Graphite](https://micrometer.io/docs/registry/graphite)。可以使用以下方式提供 [Graphite 服务器](https://graphiteapp.org/)主机和端口：

```ini
management.metrics.export.graphite.host=graphite.example.com
management.metrics.export.graphite.port=9004
```

Micrometer 提供了一个默认的 `HierarchicalNameMapper`，它管理维度计数器 id 如何[映射到平面分层名称](https://micrometer.io/docs/registry/graphite#_hierarchical_name_mapping)。

**提示**

> 要控制此行为，请定义 `GraphiteMeterRegistry` 并提供自己的 `HierarchicalNameMapper`。除非你自己定义，否则使用自动配置的 `GraphiteConfig` 和 `Clock` bean：

```java
@Bean
public GraphiteMeterRegistry graphiteMeterRegistry(GraphiteConfig config, Clock clock) {
	return new GraphiteMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

<a id="production-ready-metrics-export-humio"></a>

#### 57.2.8、Humio

默认情况下，Humio 注册表会定期将指标推送到 [cloud.humio.com](https://cloud.humio.com/)。要将指标导出到 SaaS [Humio](https://micrometer.io/docs/registry/humio)，你必须提供 API 令牌：

```ini
management.metrics.export.humio.api-token=YOUR_TOKEN
```

你还应配置一个或多个标记，以标识要推送指标的数据源：

```ini
management.metrics.export.humio.tags.alpha=a
management.metrics.export.humio.tags.bravo=b
```

<a id="production-ready-metrics-export-influx"></a>

#### 57.2.9、Influx

默认情况下，度量将导出到本地的 [Influx](https://micrometer.io/docs/registry/influx)。要指定 [Influx 服务器](https://www.influxdata.com/)的位置，可以使用：

```ini
management.metrics.export.influx.uri=https://influx.example.com:8086
```

<a id="production-ready-metrics-export-jmx"></a>

#### 57.2.10、JMX

Micrometer 提供了与 [JMX](https://micrometer.io/docs/registry/jmx) 的分层映射，主要为了方便在本地查看指标且可移植。默认情况下，度量将导出到 `metrics` JMX 域。可以使用以下方式提供要使用的域：

```ini
management.metrics.export.jmx.domain=com.example.app.metrics
```

Micrometer 提供了一个默认的 `HierarchicalNameMapper`，它管理维度计数器 id 如何[映射到平面分层名称](https://micrometer.io/docs/registry/jmx#_hierarchical_name_mapping)。

**提示**

> 要控制此行为，请定义 `JmxMeterRegistry` 并提供自己的 `HierarchicalNameMapper`。除非你自己定义，否则使用自动配置的 `JmxConfig` 和 `Clock` bean：

```java
@Bean
public JmxMeterRegistry jmxMeterRegistry(JmxConfig config, Clock clock) {
	return new JmxMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

<a id="production-ready-metrics-export-kairos"></a>

#### 57.2.11、KairosDB

默认情况下，度量将导出到本地的 [KairosDB](https://micrometer.io/docs/registry/kairos)。可以使用以下方式提供 [KairosDB 服务器](https://kairosdb.github.io/)的位置：

```ini
management.metrics.export.kairos.uri=https://kairosdb.example.com:8080/api/v1/datapoints
```

<a id="production-ready-metrics-export-newrelic"></a>

#### 57.2.12、New Relic

New Relic 注册表定期将指标推送到 [New Relic](https://micrometer.io/docs/registry/new-relic)。要将指标导出到 [New Relic](https://newrelic.com/)，你必须提供 API 密钥和帐户 ID：

```ini
management.metrics.export.newrelic.api-key=YOUR_KEY
management.metrics.export.newrelic.account-id=YOUR_ACCOUNT_ID
```

你还可以更改将度量发送到 New Relic 的间隔时间：

```ini
management.metrics.export.newrelic.step=30s
```

<a id="production-ready-metrics-export-prometheus"></a>

#### 57.2.13、Prometheus

[Prometheus](https://micrometer.io/docs/registry/prometheus) 希望抓取或轮询各个应用实例以获取指标数据。Spring Boot 在 `/actuator/prometheus` 上提供 actuator 端点，以适当的格式呈现 [Prometheus scrape ](https://prometheus.io/)。

**提示**

> 默认情况下端点不可用，必须暴露，请参阅[暴露端点](#production-ready-endpoints-exposing-endpoints)以获取更多详细信息。

以下是要添加到 `prometheus.yml` 的示例 `scrape_config`：

```yaml
scrape_configs:
  - job_name: 'spring'
	metrics_path: '/actuator/prometheus'
	static_configs:
	  - targets: ['HOST:PORT']
```

<a id="production-ready-metrics-export-signalfx"></a>

#### 57.2.14、SignalFx

SignalFx 注册表定期将指标推送到 [SignalFx](https://micrometer.io/docs/registry/signalfx)。要将指标导出到 [SignalFx](https://signalfx.com/)，你必须提供访问令牌：

```ini
management.metrics.export.signalfx.access-token=YOUR_ACCESS_TOKEN
```

你还可以更改将指标发送到 SignalFx 的间隔时间：

```ini
management.metrics.export.signalfx.step=30s
```

<a id="production-ready-metrics-export-simple"></a>

#### 57.2.15、Simple

Micrometer 附带一个简单的内存后端，如果没有配置其他注册表，它将自动用作后备。这使你可以查看[指标端点](#production-ready-metrics-endpoint)中收集的指标信息。

只要你使用了任何其他可用的后端，内存后端就会自动禁用。你也可以显式禁用它：

```ini
management.metrics.export.simple.enabled=false
```

<a id="production-ready-metrics-export-statsd"></a>

#### 57.2.16、StatsD

StatsD 注册表将 UDP 上的指标推送到 [StatsD](https://micrometer.io/docs/registry/statsd) 代理。 默认情况下，度量将导出到本地的 StatsD 代理，可以使用以下方式提供 StatsD 代理主机和端口：

```ini
management.metrics.export.statsd.host=statsd.example.com
management.metrics.export.statsd.port=9125
```

你还可以更改要使用的 StatsD 线路协议（默认为 Datadog）：

```ini
management.metrics.export.statsd.flavor=etsy
```

<a id="production-ready-metrics-export-wavefront"></a>

#### 57.2.17、Wavefront

Wavefront 注册表定期将指标推送到 [Wavefront](https://micrometer.io/docs/registry/wavefront)。如果要将指标直接导出到 [Wavefront](https://www.wavefront.com/)，则你必须提供 API 令牌：

```ini
management.metrics.export.wavefront.api-token=YOUR_API_TOKEN
```

或者，你可以在环境中使用 Wavefront sidecar 或内部代理设置，将指标数据转发到 Wavefront API 主机：

```ini
management.metrics.export.wavefront.uri=proxy://localhost:2878
```

**提示**

> 如果将度量发布到 Wavefront 代理（如[文档](https://docs.wavefront.com/proxies_installing.html)中所述），则主机必须采用 `proxy://HOST:PORT` 格式。

你还可以更改将指标发送到 Wavefront 的间隔时间：

```ini
management.metrics.export.wavefront.step=30s
```

<a id="production-ready-metrics-meter"></a>

### 57.3、支持的指标

Spring Boot 在适当的环境注册以下核心指标：

- JVM 指标，报告利用率：
	- 各种内存和缓冲池
	- 与垃圾回收有关的统计
	- 线程利用率
	- 加载/卸载 class 的数量
- CPU 指标
- 文件描述符指标
- Kafka 消费者指标
- Log4j2 指标：记录每个级别记录到 Log4j2 的事件数
- Logback 指标：记录每个级别记录到 Logback 的事件数
- 正常运行时间指标：报告正常运行时间和表示应用程序绝对启动时间的固定计量值
- Tomcat 指标
- [Spring Integration](https://docs.spring.io/spring-integration/docs/current/reference/html/system-management-chapter.html#micrometer-integration) 指标

<a id="production-ready-metrics-spring-mvc"></a>

#### 57.3.1、Spring MVC 指标

自动配置启用 Spring MVC 处理的请求的指标记录。当 `management.metrics.web.server.auto-time-requests` 为 `true` 时，将对所有请求进行此检测。或者，当设置为 `false` 时，你可以通过将 `@Timed` 添加到请求处理方法来启用检测：

```java
@RestController
@Timed // <1>
public class MyController {

	@GetMapping("/api/people")
	@Timed(extraTags = { "region", "us-east-1" }) // <2>
	@Timed(value = "all.people", longTask = true) // <3>
	public List<Person> listPeople() { ... }

}
```

1. 一个控制器类，为控制器中的每个请求处理程序启用计时。
2. 启用单个端点。如果你在类上使用了它，就不需要在方法上再次声明，但可以用它来进一步自定义该特定端点的计时器。
3. 使用 `longTask = true` 的方法为该方法启用长任务计时器。长任务计时器需要单独的指标名称，并且可以使用短任务计时器进行堆叠。

默认情况下，使用名称为 `http.server.requests` 生成度量指标。可以通过设置 `management.metrics.web.server.requests-metric-name` 属性来自定义名称。

默认情况下，Spring MVC 相关指标使用了以下标签标记：

| 标签 | 描述 |
| --- | --- |
| `exception` | 处理请求时抛出的异常的简单类名。 |
| `method` | 请求的方法（例如，`GET` 或 `POST`） |
| `outcome` | 根据响应状态码生成结果。1xx 是 `INFORMATIONAL`，2xx 是 `SUCCESS`，3xx 是 `REDIRECTION`，4xx 是 `CLIENT_ERROR`，5xx 是 `SERVER_ERROR` |
| `status` | 响应的 HTTP 状态码（例如，`200` 或 `500`） |
| `uri` | 如果可能，在变量替换之前请求 URI 模板（例如，`/api/person/{id}`） |

要自定义标签，请提供一个实现了 `WebMvcTagsProvider` 的 `@Bean`。

<a id="production-ready-metrics-web-flux"></a>

#### 57.3.2、Spring WebFlux 指标

自动配置启用了 WebFlux 控制器和函数式处理程序处理的所有请求的指标记录功能。

默认情况下，使用名为 `http.server.requests` 生成度量指标。你可以通过设置 `management.metrics.web.server.requests-metric-name` 属性来自定义名称。

默认情况下，与 WebFlux 相关的指标使用以下标签标记：

| 标签 | 描述 |
| --- | --- |
| `exception` | 处理请求时抛出的异常的简单类名。 |
| `method` | 请求方法（例如，`GET`或 `POST`） |
| `outcome` | 根据响应状态码生成请求结果。1xx 是 `INFORMATIONAL`，2xx 是 `SUCCESS`，3xx 是 `REDIRECTION`，4xx 是 `CLIENT_ERROR`，5xx 是 `SERVER_ERROR` |
| `status` | 响应的 HTTP 状态码（例如，`200` 或 `500`） |
| `uri` | 如果可能，在变量替换之前请求 URI 模板（例如，`/api/person/{id}`） |

要自定义标签，请提供一个实现了 `WebFluxTagsProvider` 的 `@Bean`。

<a id="production-ready-metrics-jersey-server"></a>

#### 57.3.3、Jersey Server 指标

自动配置启用了由 Jersey JAX-RS 实现处理的请求的指标记录功能。当 `management.metrics.web.server.auto-time-requests` 为 `true` 时，将对所有请求进行该项检测。当设置为 `false` 时，你可以通过将 `@Timed` 添加到请求处理方法上来启用检测：

```java
@Component
@Path("/api/people")
@Timed // <1>
public class Endpoint {
	@GET
	@Timed(extraTags = { "region", "us-east-1" }) // <2>
	@Timed(value = "all.people", longTask = true) // <3>
	public List<Person> listPeople() { ... }
}
```

1. 在资源类上，为资源中的每个请求处理程序启用计时。
2. 在方法上则启用单个端点。如果你在类上使用了它，则不需在方法上再次声明，但可以用它来进一步自定义该特定端点的计时器。
3. 在有 `longTask = true` 的方法上，为该方法启用长任务计时器。长任务计时器需要单独的度量名称，并且可以使用短任务计时器进行堆叠。

默认情况下，使用名为 `http.server.requests` 生成度量指标。可以通过设置 `management.metrics.web.server.requests-metric-name` 属性来自定义名称。

默认情况下，Jersey 服务器指标使用以下标签标记：

| 标签 | 描述 |
| --- | --- |
| `exception` | 处理请求时抛出的异常的简单类名。 |
| `method` | 请求的方法（例如，`GET` 或 `POST`） |
| `outcome` | 根据响应状态码生成的请求结果。1xx 是 `INFORMATIONAL`，2xx 是 `SUCCESS`，3xx 是 `REDIRECTION`，4xx 是 `CLIENT_ERROR`，5xx 是 `SERVER_ERROR` |
| `status` | 响应的 HTTP 状态码（例如，`200`或 `500`） |
| `uri` | 如果可能，在变量替换之前请求 URI 模板（例如，`/api/person/{id}`） |

要自定义标签，请提供一个实现了 `JerseyTagsProvider` 的 `@Bean`。

<a id="production-ready-metrics-http-clients"></a>

#### 57.3.4、HTTP Client 指标

Spring Boot Actuator 管理 RestTemplate 和 WebClient 的指标记录。为此，你必须注入一个自动配置的 builder 并使用它来创建实例：

- `RestTemplateBuilder` 用于 `RestTemplate`
- `WebClient.Builder` 用于 `WebClient`

也可以手动指定负责此指标记录的自定义程序，即 `MetricsRestTemplateCustomizer` 和 `MetricsWebClientCustomizer`。

默认情况下，使用名为 `http.client.requests` 生成度量指标。可以通过设置 `management.metrics.web.client.requests-metric-name` 属性来自定义名称。

默认情况下，已指标记录客户端生成的度量指标使用以下标签标记：

- `method`，请求的方法（例如，`GET`或 `POST`）。
- `uri`，变量替换之前的请求 URI 模板（如果可能）（例如，`/api/person/{id}`）。
- `status`，响应的 HTTP 状态码（例如，`200` 或 `500`）。
- `clientName`，URI 的主机部分 。

要根据你选择的客户端自定义标签，你可以提供一个实现了 `RestTemplateExchangeTagsProvider` 或 `WebClientExchangeTagsProvider` 的 `@Bean`。`RestTemplateExchangeTags` 和 `WebClientExchangeTags` 中有便捷的静态函数。

<a id="production-ready-metrics-cache"></a>

#### 57.3.5、Cache 指标

在启动时，自动配置启动所有可用 `Cache` 的指标记录功能，指标以 `cache` 为前缀。缓存指标记录针对一组基本指标进行了标准化。此外，还提供了缓存特定的指标。

支持以下缓存库：

- Caffeine
- EhCache 2
- Hazelcast
- 所有兼容 JCache（JSR-107）的实现

度量指标由缓存的名称和从 bean 名称派生的 `CacheManager` 的名称标记。

**注意**

> 只有启动时可用的缓存才会绑定到注册表。对于在启动阶段之后即时或以编程方式创建的缓存，需要显式注册。可用 `CacheMetricsRegistrar` bean 简化该过程。

<a id="production-ready-metrics-jdbc"></a>

#### 57.3.6、数据源指标

自动配置启用对所有可用 `DataSource` 对象进行指标记录功能，指标的名称为 `jdbc`。数据源指标记录会生成表示池中当前活动、大允许和最小允许连接的计量器（gauge）。这些计量器都有一个以 `jdbc` 为前缀的名称。

度量指标也由基于 bean 名称计算的 `DataSource` 的名称标记。

**提示**

> 默认情况下，Spring Boot 为所有支持的数据源提供了元数据。如果开箱即用不支持你喜欢的数据源，则可以添加其他 `DataSourcePoolMetadataProvider` bean。有关示例，请参阅 `DataSourcePoolMetadataProvidersConfiguration`。

此外，Hikari 特定的指标用 `hikaricp` 前缀暴露。每个度量指标都由池名称标记（可以使用 `spring.datasource.name` 控制）。

<a id="production-ready-metrics-hibernate"></a>

#### 57.3.7、Hibernate 指标

自动配置启用所有可用 Hibernate `EntityManagerFactory` 实例的指标记录功能，这些实例使用名为 `hibernate` 的度量指标统计信息。

度量指标也由从 bean 名称派生的 `EntityManagerFactory` 的名称标记。

要启用信息统计，必须将标准 JPA 属性 `hibernate.generate_statistics` 设置为 `true`。你可以在自动配置的 `EntityManagerFactory` 上启用它，如下所示：

```ini
spring.jpa.properties.hibernate.generate_statistics=true
```

<a id="production-ready-metrics-rabbitmq"></a>

#### 57.3.8、RabbitMQ 指标

自动配置将使用名为 `rabbitmq` 的度量指标启用对所有可用 RabbitMQ 连接工厂进行指标记录。

<a id="production-ready-metrics-custom"></a>

### 57.4、注册自定义指标

要注册自定义指标，请将 `MeterRegistry` 注入你的组件中，如下所示：

```java
class Dictionary {

	private final List<String> words = new CopyOnWriteArrayList<>();

	Dictionary(MeterRegistry registry) {
		registry.gaugeCollectionSize("dictionary.size", Tags.empty(), this.words);
	}

	// …

}
```

如果你发现跨组件或应用程序重复记录一套度量指标，则可以将此套件封装在 `MeterBinder` 实现中。默认情况下，所有 `MeterBinder` bean 的指标都将自动绑定到 Spring 管理的 `MeterRegistry`。

<a id="production-ready-metrics-per-meter-properties"></a>

### 57.5、自定义单独指标

如果需要将自定义特定的 `Meter` 实例，可以使用 `io.micrometer.core.instrument.config.MeterFilter` 接口。默认情况下，所有 `MeterFilter` bean 都将自动应用于 `MeterRegistry.Config`。

例如，如果要将 `mytag.region` 标记重命名为 `mytag.area` 以获取以 `com.example` 开头的所有 meter ID，则可以执行以下操作：

```java
@Bean
public MeterFilter renameRegionTagMeterFilter() {
	return MeterFilter.renameTag("com.example", "mytag.region", "mytag.area");
}
```

<a id="production-ready-metrics-common-tags"></a>

#### 57.5.1、自定义标签

通用标签通常用于操作环境中的维度下钻，例如主机、实例、区域、堆栈等。通用标签应用于所有 meter，并且可以按照以下示例进行配置：

```ini
management.metrics.tags.region=us-east-1
management.metrics.tags.stack=prod
```

上面的示例将 `region` 和 `stack` 标签添加到所有 meter 中，其值分别为 `us-east-1` 和 `prod`。

**注意**

> 如果你使用 Graphite，那么标签的顺序很重要。由于使用此方法无法保证通用标签的顺序，因此建议 Graphite 用户定义自定义 `MeterFilter`。

<a id="_per_meter_properties"></a>

#### 57.5.2、Per-meter 属性

除了 `MeterFilter` bean 之外，还可以使用 properties 在 per-meter 基础上自定义。Per-meter 定义适用于以给定名称开头的所有 meter ID。例如，以下将禁用任何以 `example.remote` 开头的 ID 的 meter：

```ini
management.metrics.enable.example.remote=false
```

以下属性允许 per-meter 自定义：

**表 57.1、Per-meter 自定义**

| 属性 | 描述 |
| `management.metrics.enable` | 是否拒绝 meter 发布任何指标。 |
| `management.metrics.distribution.percentiles-histogram` | 是否发布一个适用于计算可聚合（跨维度）的百分比近似柱状图。 |
| `management.metrics.distribution.minimum-expected-value`，<br/> `management.metrics.distribution.maximum-expected-value`| 通过限制预期值的范围来发布较少的柱状图桶。 |
| `management.metrics.distribution.percentiles` | 发布在你自己的应用程序中计算的百分比数值 |
| `management.metrics.distribution.sla` | 发一个使用 SLA 定义的存储桶发布累积柱状图。 |

有关 `percentiles-histogram`、`percentiles` 和 `sla` 概念的更多详细信息，请参阅「[柱状图与百分位数](https://micrometer.io/docs/concepts#_histograms_and_percentiles)」部分的文档。

<a id="production-ready-metrics-endpoint"></a>

### 57.6、指标端点

Spring Boot 提供了一个 `metrics` 端点，可以在诊断中用于检查应用程序收集的度量指标。默认情况下端点不可用，必须手动暴露，请参阅[暴露端点](#production-ready-endpoints-exposing-endpoints)以获取更多详细信息。

访问 `/actuator/metrics` 会显示可用的 meter 名称列表。你可以查看某一个 meter 的信息，方法是将其名称作为选择器，例如，`/actuator/metrics/jvm.memory.max`。

**提示**

> 你在此处使用的名称应与代码中使用的名称相匹配，而不是在命名约定规范化后的名称 —— 为了发送到监控系统。换句话说，如果 `jvm.memory.max` 由于 Prometheus 命名约定而显示为 `jvm_memory_max`，则在审查度量指标端点中的 meter 时，应仍使用 `jvm.memory.max` 作为选择器。

你还可以在 URL 的末尾添加任意数量的 `tag=KEY:VALUE` 查询参数，以便多维度向下钻取 meter，例如 `/actuator/metrics/jvm.memory.max?tag=area:nonheap`。

**提示**

报告的测量值是与 meter 名称和已应用的任何标签匹配的所有 meter 的统计数据的总和。因此，在上面的示例中，返回的 Value 统计信息是堆的 Code Cache，Compressed Class Space 和 Metaspace 区域的最大内存占用量的总和。如果你只想查看 Metaspace 的最大大小，可以添加一个额外的 `tag=id:Metaspace`，即 `/actuator/metrics/jvm.memory.max?tag=area:nonheap&tag=id:Metaspace`。

<a id="production-ready-auditing"></a>

## 58、审计

一旦 Spring Security 生效，Spring Boot Actuator 就拥有一个灵活的审计框架，它可以发布事件（默认情况下，`authentication success`、`failure` 和 `access denied` 例外）。此功能对事件报告和基于身份验证失败实现一个锁定策略非常有用。要自定义发布的安全事件，你可以提供自己的 `AbstractAuthenticationAuditListener` 和 `AbstractAuthorizationAuditListener` 实现。

你还可以将审计服务用于自己的业务事件。为此，请将现有的 `AuditEventRepository` 注入自己的组件并直接使用它或使用 Spring `ApplicationEventPublisher`（通过实现 `ApplicationEventPublisherAware`）发布 `AuditApplicationEvent`。

<a id="production-ready-http-tracing"></a>

## 59、HTTP 追踪

所有 HTTP 请求将自动启用追踪功能。你可以通过查看 `httptrace` 端点来获取最近相关的 100 个请求响应信息。

<a id="production-ready-http-tracing-custom"></a>

### 59.1、自定义 HTTP 追踪

要自定义每个追踪信息中包含的项，请使用 `management.trace.http.include` 属性配置。对于高级自定义，请考虑注册自己的 `HttpExchangeTracer` 实现。

默认情况下，使用一个 `InMemoryHttpTraceRepository` 存储最新的 100 个请求响应信息。如果需要扩展容量，可定义自己的 `InMemoryHttpTraceRepository` bean 实例。你还可以创建自己的 `HttpTraceRepository` 实现来替代默认配置。

<a id="production-ready-process-monitoring"></a>

## 60、进程监控

在 `spring-boot` 模块中，你可以找到两个类来创建文件，他们通常用于进程监控：

- `ApplicationPidFileWriter` 创建一个包含应用程序 PID 的文件（默认在应用程序目录中，文件名为 `application.pid`）。
- `WebServerPortFileWriter` 创建一个或多个文件，其包含正在运行的 Web 服务器的端口（默认在应用程序目录中，文件名为 `application.port`）。

默认情况下，这些 writer 未激活，但你可以启用：

- [扩展配置](#production-ready-process-monitoring-configuration)
- [第 60.2 节、编程方式](#production-ready-process-monitoring-programmatically)

<a id="production-ready-process-monitoring-configuration"></a>

### 60.1、扩展配置

你可以在 `META-INF/spring.factories` 文件中激活生成和写入 PID 文件的监听器（Listener），如下所示：

```
org.springframework.context.ApplicationListener=\
org.springframework.boot.context.ApplicationPidFileWriter,\
org.springframework.boot.web.context.WebServerPortFileWriter
```

<a id="production-ready-process-monitoring-configuration"></a>

### 60.2、编程方式

你还可以通过调用 `SpringApplication.addListeners(...)` 方法并传递相应的 `Writer` 对象来激活监听器。此方法还允许你在 `Writer` 构造方法中自定义文件名和路径。

<a id="production-ready-cloudfoundry"></a>

## 61、Cloud Foundry 支持

当你部署到一个兼容 Cloud Foundry 的实例时，Spring Boot 的 Actuator 模块包含的其他支持将被激活。`/cloudfoundryapplication` 路径为所有 `@Endpoint` bean 提供了另外一个安全路由。

该扩展支持允许使用 Spring Boot Actuator 信息扩充 Cloud Foundry 管理 UI（例如可用于查看已部署应用的 Web 应用）。比如，应用程序状态页面可以包括完整的健康信息而不是常见的 running 或 stop 状态。

**注意**

> 常规用户无法直接访问 `/cloudfoundryapplication` 路径。为了能访问端点，你必须在请求时传递一个有效的 UAA 令牌。

<a id="production-ready-cloudfoundry-disable"></a>

### 61.1、禁用 Cloud Foundry Actuator 扩展支持

如果要完全禁用 `/cloudfoundryapplication` 端点，可以将以下设置添加到 `application.properties` 文件中：

**application.properties**

```ini
management.cloudfoundry.enabled=false
```

<a id="production-ready-cloudfoundry-ssl"></a>

### 61.2、Cloud Foundry 自签名证书

默认情况下，`/cloudfoundryapplication` 端点的安全验证会对各种 Cloud Foundry 服务进行 SSL 调用。如果你的 Cloud Foundry UAA 或 Cloud Controller 服务使用自签名证书，则需要设置以下属性：

**application.properties**

```ini
management.cloudfoundry.skip-ssl-validation=true
```

<a id="_custom_context_path"></a>

### 61.3、自定义上下文路径

如果服务器的 context-path 已配置为 `/` 以外的其他内容，则 Cloud Foundry 端点将无法在应用程序的根目录中使用。例如，如果 `server.servlet.context-path=/app`，Cloud Foundry 端点将在 `/app/cloudfoundryapplication/*` 上可用。

如果你希望 Cloud Foundry 端点始终在 `/cloudfoundryapplication/*` 上可用，则无论服务器的 context-path 如何，你都需要在应用程序中明确配置它。配置因使用的 Web 服务器而有所不同。针对 Tomcat，可以添加以下配置：

```java
@Bean
public TomcatServletWebServerFactory servletWebServerFactory() {
	return new TomcatServletWebServerFactory() {

		@Override
		protected void prepareContext(Host host,
				ServletContextInitializer[] initializers) {
			super.prepareContext(host, initializers);
			StandardContext child = new StandardContext();
			child.addLifecycleListener(new Tomcat.FixContextListener());
			child.setPath("/cloudfoundryapplication");
			ServletContainerInitializer initializer = getServletContextInitializer(
					getContextPath());
			child.addServletContainerInitializer(initializer, Collections.emptySet());
			child.setCrossContext(true);
			host.addChild(child);
		}

	};
}

private ServletContainerInitializer getServletContextInitializer(String contextPath) {
	return (c, context) -> {
		Servlet servlet = new GenericServlet() {

			@Override
			public void service(ServletRequest req, ServletResponse res)
					throws ServletException, IOException {
				ServletContext context = req.getServletContext()
						.getContext(contextPath);
				context.getRequestDispatcher("/cloudfoundryapplication").forward(req,
						res);
			}

		};
		context.addServlet("cloudfoundry", servlet).addMapping("/*");
	};
}
```

<a id="production-ready-whats-next"></a>

## 62、下一步

如果你想了解本章中讨论的一些概念，你可以查看 actuator [示例应用程序](https://github.com/spring-projects/spring-boot/tree/v2.1.5.RELEASE/spring-boot-samples)。或许你还想了解 [Graphite](https://graphite.wikidot.com/) 等图形工具的相关知识。

此外，你可以继续阅读[应用部署](deployment.md)相关内容，或继续阅读有关 Spring Boot [构建工具插件](build-tool-plugins.md)的相关内容。
