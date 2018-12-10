<a id="boot-features"></a>

# 四、Spring Boot 功能

本部分将介绍 Spring Boot 相关的细节内容。在这里，您可以学习到可能需要使用和自定义的主要功能。您如果还没有做好充分准备，可能需要阅读[第二部分：入门](#getting-started)和[第三部分：使用 Spring Boot](#using-boot)，以便打下前期基础。

<a id="boot-features-spring-application"></a>

## 23、SpringApplication

`SpringApplication` 类提供了一种可通过运行 `main()` 方法来启动 Spring 应用的简单方式。多数情况下，您只需要委托给静态的 `SpringApplication.run` 方法：

```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

当应用启动时，您应该会看到类似以下的内容输出：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.1.1.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下，将显示 `INFO` 级别的日志信息，包括一些应用启动相关信息。如果您需要修改 `INFO` 日志级别，请参考 [26.4 部分：日志等级](#boot-features-custom-log-levels)。

<a id="boot-features-startup-failure"></a>

### 23.1、启动失败

如果您的应用无法启动，注册的 `FailureAnalyzers` 可能会提供有相关的错误信息和解决问题的具体方法。例如，如果您在已经被占用的 `8080` 端口上启动了一个 web 应用，会看到类似以下的错误信息：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

**注意**

> Spring Boot 提供了许多的 `FailureAnalyzer` 实现，您也可以[添加自己的实现](#howto-failure-analyzer)。

如果没有失败分析器能够处理的异常，您仍然可以显示完整的条件报告以便更好地了解出现的问题。为此，您需要针对 `org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` [启用 `debug` 属性](#boot-features-external-config)或者[开启 `DEBUG` 日志](#boot-features-custom-log-levels)。

例如，如果您使用 `java -jar` 运行应用，可以按以下方式启用 `debug` 属性：

```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

<a id="boot-features-banner"></a>

### 23.2、自定义 banner

可以通过在 classpath 下添加一个 `banner.txt` 文件，或者将 `spring.banner.location` 属性指向该文件的位置来更改启动时打印的 banner。如果文件采用了非 UTF-8 编码，您可以设置 `spring.banner.charset` 来解决。除了文本文件，您还可以将 `banner.gif`、`banner.jpg` 或者 `banner.png` 图片文件添加到 classpath 下，或者设置 `spring.banner.image.location` 属性。指定的图片将会被转换成 ASCII 形式并打印在 banner 文本上方。

您可以在 `banner.txt` 文件中使用以下占位符：

| 变量 | 描述 |
| ---- | ---- |
| `${application.version}` | 您的应用版本号，声明在 `MANIFEST.MF` 中。例如，`Implementation-Version: 1.0` 将被打印为 `1.0`。 |
| `${application.formatted-version}` | 您的应用版本号，声明在 `MANIFEST.MF` 中，格式化之后打印（用括号括起来，以 `v` 为前缀），例如 (`v1.0`)。 |
| `${spring-boot.version}` | 您使用的 Spring Boot 版本。例如 `2.1.1.RELEASE.`。 |
| `${spring-boot.formatted-version}` | 您使用的 Spring Boot 版本格式化之后显示（用括号括起来，以 `v` 为前缀）。例如 (`v2.1.1.RELEASE`)。 |
| `${Ansi.NAME}`（或 `${AnsiColor.NAME}`、<br/>`${AnsiBackground.NAME}`、<br/>`${AnsiStyle.NAME}`）| 其中 `NAME` 是 ANSI 转义码的名称。有关详细信息，请参阅 [AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。 |
| `${application.title}` | 您的应用标题，声明在 `MANIFEST.MF` 中，例如 `Implementation-Title: MyApp` 打印为 `MyApp`。 |


**提示**

> 如果您想以编程的方式生成 banner，可以使用 `SpringApplication.setBanner(​...)` 方法。使用 `org.springframework.boot.Banner` 接口并实现自己的 `printBanner()` 方法。

您还可以使用 `spring.main.banner-mode` 属性来确定是否必须在 `System.out`（`console`）上打印 banner，还是使用日志记录器（`log`）或者都不打印（`off`）。

打印的 banner 被注册名为 `springBootBanner` 的单例 bean。

**注意**

> YAML 将 `off` 映射为 `false`，因此如果要禁用应用程序 banner，请确保属性添加引号。

```yaml
spring:
    main:
        banner-mode: "off"
```

<a id="boot-features-customizing-spring-application"></a>

### 23.3、自定义 SpringApplication

如果 `SpringApplication` 的默认设置不符合您的想法，您可以创建本地实例进行定制化。例如，要关闭 banner，您可以这样：

```java
public static void main(String[] args) {
	SpringApplication app = new SpringApplication(MySpringConfiguration.class);
	app.setBannerMode(Banner.Mode.OFF);
	app.run(args);
}
```

**注意**

> 传入 `SpringApplication` 的构造参数是 spring bean 的配置源。大多情况下是引用 `@Configuration` 类，但您也可以引用 XML 配置或者被扫描的包。

也可以使用 `application.properties` 文件配置 `SpringApplication`。有关详细信息，请参见[第 24 章：外部化配置](#boot-features-external-config)。

关于配置选项的完整列表，请参阅 [SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/SpringApplication.html)。

<a id="boot-features-customizing-spring-application"></a>

### 23.4、Fluent Builder API

如果您需要构建一个有层级关系的 `ApplicationContext`（具有父/子关系的多上下文），或者偏向使用 **fluent**（流畅）构建器 API，可以使用 `SpringApplicationBuilder`。

`SpringApplicationBuilder` 允许您链式调用多个方法，包括能创建出具有层次结构的 `parent` 和 `child` 方法。

例如：

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

**注意**

> 创建层级的 `ApplicationContext` 时有部分限制，比如 Web 组件**必须**包含在子上下文中，并且相同的 `Environment` 将作用于父子上下文。有关详细信息，请参阅 [SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html)。

<a id="boot-features-application-events-and-listeners"></a>

### 23.5、应用程序事件与监听器

除了常见的 Spring Framework 事件，比如 [`ContextRefreshedEvent`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，`SpringApplication` 还会发送其他应用程序事件。

**注意**

> 在 `ApplicationContext` 创建之前，实际上触发了一些事件，因此您不能像 `@Bean` 一样注册监听器。您可以通过 `SpringApplication.addListeners(​...)` 或者 `SpringApplicationBuilder.listeners(...​)` 方法注册它们。如果您希望无论应用使用何种创建方式都能自动注册这些监听器，您都可以将 `META-INF/spring.factories` 文件添加到项目中，并使用 `org.springframework.context.ApplicationListener ` 属性键指向您的监听器。比如：`org.springframework.context.ApplicationListener=com.example.project.MyListener`

当您运行应用时，应用程序事件将按照以下顺序发送：

1. 在开始应用开始运行但还没有进行任何处理时（除了注册监听器和初始化器[initializer]），将发送 `ApplicationStartingEvent`。
2. 当 `Environment` 被上下文使用，但是在上下文创建之前，将发送 `ApplicationEnvironmentPreparedEvent`。
3. 在开始刷新之前，bean 定义被加载之后发送 `ApplicationPreparedEvent`。
4. 在上下文刷新之后且所有的应用和命令行运行器（command-line runner）被调用之前发送 `ApplicationStartedEvent`。
5. 在应用程序和命令行运行器（command-line runner）被调用之后，将发出 `ApplicationReadyEvent`，该事件用于通知应用已经准备处理请求。
6. 如果启动时发生异常，将发送 `ApplicationFailedEvent`。

**提示**

> 您可能不会经常使用应用程序事件，但了解他们的存在还是很有必要的。在框架内部，Spring Boot 使用这些事件来处理各种任务。

应用程序事件发送使用了 Spring Framework 的事件发布机制。该部分机制确保在子上下文中发布给监听器的事件也会发布给所有祖先上下文中的监听器。因此，如果您的应用程序使用有层级结构的 SpringApplication 实例，则监听器可能会收到同种类型应用程序事件的多个实例。

为了让监听器能够区分其上下文事件和后代上下文事件，您应该注入其应用程序上下文，然后将注入的上下文与事件的上下文进行比较。可以通过实现 `ApplicationContextAware` 来注入上下文，如果监听器是 bean，则使用 `@Autowired` 注入上下文。

<a id="boot-features-web-environment"></a>

### 23.6、Web 环境

`SpringApplication` 试图为您创建正确类型的 `ApplicationContext`。确定 `WebApplicationType` 的算法非常简单：

- 如果存在 Spring MVC，则使用 `AnnotationConfigServletWebServerApplicationContext`
- 如果 Spring MVC 不存在且存在 Spring WebFlux，则使用 `AnnotationConfigReactiveWebServerApplicationContext`
- 否则，使用 `AnnotationConfigApplicationContext`

这意味着如果您在同一个应用程序中使用了 Spring MVC 和 Spring WebFlux 中的新 `WebClient`，默认情况下将使用 Spring MVC。您可以通过调用 `setWebApplicationType(WebApplicationType)` 修改默认行为。

也可以调用 `setApplicationContextClass(...)` 来完全控制 `ApplicationContext` 类型。

**提示**

> 在 JUnit 测试中使用 `SpringApplication` 时，通常需要调用 `setWebApplicationType(WebApplicationType.NONE)`。

<a id="boot-features-application-arguments"></a>

### 23.7、访问应用程序参数

如果您需要访问从 `SpringApplication.run(​...) ` 传入的应用程序参数，可以注入一个 `org.springframework.boot.ApplicationArguments` bean。`ApplicationArguments` 接口提供了访问原始 `String[]` 参数以及解析后的 `option` 和 `non-option` 参数的方法：

```java
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;

@Component
public class MyBean {

	@Autowired
	public MyBean(ApplicationArguments args) {
		boolean debug = args.containsOption("debug");
		List<String> files = args.getNonOptionArgs();
		// if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
	}

}
```

**提示**

> Spring Boot 还向 Spring `Environment` 注册了一个 `CommandLinePropertySource`。这允许您可以使用 `@Value` 注解注入单个应用参数。

<a id="boot-features-command-line-runner"></a>

### 23.8、使用 ApplicationRunner 或 ApplicationRunner

如果您需要在 SpringApplication 启动时运行一些代码，可以实现 `ApplicationRunner` 或者 `CommandLineRunner` 接口。这两个接口的工作方式是一样的，都提供了一个单独的 `run` 方法，它将在 `SpringApplication.run(​...)` 完成之前调用。

`CommandLineRunner` 接口提供了访问应用程序字符串数组形式参数的方法，而 `ApplicationRunner` 则使用了上述的 `ApplicationArguments` 接口。以下示例展示 `CommandLineRunner` 和 `run` 方法的使用：

```java
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

	public void run(String... args) {
		// Do something...
	}

}
```

如果您定义了多个 `CommandLineRunner` 或者 `ApplicationRunner` bean，则必须指定调用顺序，您可以实现 `org.springframework.core.Ordered` 接口，也可以使用 `org.springframework.core.annotation.Order` 注解解决顺序问题。

<a id="boot-features-application-exit"></a>

### 23.9、应用程序退出

每个 `SpringApplication` 注册了一个 JVM 关闭钩子，以确保 `ApplicationContext` 在退出时可以优雅关闭。所有标准的 Spring 生命周期回调（比如 `DisposableBean` 接口，或者 `@PreDestroy` 注解）都可以使用。

此外，如果希望在调用 `SpringApplication.exit()` 时返回特定的退出码，则 bean 可以实现 `org.springframework.boot.ExitCodeGenerator` 接口。之后退出码将传递给 `System.exit()` 以将其作为状态码返回，如示例所示：

```java
@SpringBootApplication
public class ExitCodeApplication {

	@Bean
	public ExitCodeGenerator exitCodeGenerator() {
		return () -> 42;
	}

	public static void main(String[] args) {
		System.exit(SpringApplication
				.exit(SpringApplication.run(ExitCodeApplication.class, args)));
	}

}
```

此外，`ExitCodeGenerator` 接口可以通过异常实现。遇到这类异常时，Spring Boot 将返回实现的 `getExitCode()` 方法提供的退出码。

<a id="boot-features-application-admin"></a>

### 23.10、管理功能

可以通过指定 `spring.application.admin.enabled` 属性来为应用程序启用管理相关的功能。其将在 `MBeanServer` 平台上暴露 [`SpringApplicationAdminMXBean`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java)。您可以使用此功能来远程管理 Spring Boot 应用。该功能对服务包装器的实现也是非常有用的。

**提示**

> 如果您想知道应用程序在哪一个 HTTP 端口上运行，请使用 `local.server.port` 键获取该属性。

**注意**

> 启用此功能时请小心，因为 MBean 暴露了关闭应用程序的方法。

<a id="boot-features-external-config"></a>

## 24、外部化配置

Spring Boot 可以让您的配置外部化，以便可以在不同环境中使用相同的应用程序代码。您可以使用 properties 文件、YAML 文件、环境变量或者命令行参数来外部化配置。可以使用 `@Value` 注解将属性值直接注入到 bean 中，可通过 Spring 的 `Environment` 访问，或者通过 `@ConfigurationProperties` 绑定到[结构化对象](#boot-features-external-config-typesafe-configuration-properties)。

Spring Boot 使用了一个非常特别的 `PropertySource` 指令，用于智能覆盖默认值。属性将按照以下顺序处理：

1. 在您的主目录（当 devtools 被激活，则为 `~/.spring-boot-devtools.properties` ）中的 [Devtools 全局设置属性](#using-boot-devtools-globalsettings)。
2. 在测试中使用到的 `@TestPropertySource` 注解。
3. 在测试中使用到的 `properties` 属性，可以是 [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/test/context/SpringBootTest.html) 和[用于测试应用程序某部分的测试注解](#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)。
4. 命令行参数。
5. 来自 `SPRING_APPLICATION_JSON` 的属性（嵌入在环境变量或者系统属性【system propert】中的内联 JSON）。
6. `ServletConfig` 初始化参数。
7. `ServletContext` 初始化参数。
8. 来自 `java:comp/env` 的 JNDI 属性。
9. Java 系统属性（`System.getProperties()`）。
10. 操作系统环境变量。
11. 只有 `random.*` 属性的 `RandomValuePropertySource`。
12. 在已打包的 jar 外部的[指定 profile 的应用属性文件](#boot-features-external-config-profile-specific-properties)（`application-{profile}.properties` 和 YAML 变量）。
13. 在已打包的 jar 内部的[指定 profile 的应用属性文件](#boot-features-external-config-profile-specific-properties)（`application-{profile}.properties` 和 YAML 变量）。
14. 在已打包的 jar 外部的应用属性文件（`application.properties` 和 YAML 变量）。
15. 在已打包的 jar 内部的应用属性文件（`application.properties` 和 YAML 变量）。
16. 在 `@Configuration` 类上的 `@PropertySource` 注解。
17. 默认属性（使用 `SpringApplication.setDefaultProperties` 指定）。

举个例子，假设开发的 `@Component` 使用了 `name` 属性，可以这样：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在您的应用程序的 classpath 中（比如在 jar 中），您可以有一个 `application.properties`，它为 `name` 提供了一个合适的默认属性值。当在新环境中运行时，您可以在 jar 外面提供一个 `application.properties` 来覆盖 `name`。对于一次性测试，您可以使用命令行指定形式启动（比如 `java -jar app.jar --name="Spring"`）。

**提示**

<blockquote>

`SPRING_APPLICATION_JSON` 属性可以在命令行中提供一个环境变量。比如在 UN*X shell 中：

```
$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
```

在此示例中，您可以在 Spring `Environment` 中使用 `acme.name=test`，也可以在系统属性（System property）中将 JSON 作为 `spring.application.json` 属性提供：

```
$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
```

或者以命令行参数形式：

```
$ java -jar myapp.jar --spring.application.json='{"name":"test"}'
```

或者将 JSON 作为一个 JNDI 变量：`java:comp/env/spring.application.json`。

</blockquote>

<a id="boot-features-developing-web-applications"></a>

## 28、开发 Web 应用程序

Spring Boot 非常适合用于开发 web 应用程序。您可以使用嵌入式 Tomcat、Jetty 或者 Undertow 来创建一个独立（self-contained）的 HTTP 服务器。大多数 web 应用程序使用 `spring-boot-starter-web` 模块来快速搭建和运行，您也可以选择使用 `spring-boot-starter-webflux` 模块来构建响应式（reactive） web 应用程序。

如果您尚未开发过 Spring Boot web 应用程序，则可以按照[入门](#getting-started-first-application)章节中的“Hello World!”示例进行操作。

<a id="boot-features-spring-mvc"></a>

### 28.1、Spring Web MVC 框架

[Spring Web MVC 框架](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc)（通常简称“Spring MVC”）是一个富**模型-视图-控制器**的 web 框架。Spring MVC 允许您创建 `@Controller` 或者 `@RestController` bean 来处理传入的 HTTP 请求。控制器中的方法通过 `@RequestMapping` 注解映射到 HTTP。

以下是一个使用了 `@RestController` 来响应 JSON 数据的典型示例：

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

	@RequestMapping(value="/{user}", method=RequestMethod.GET)
	public User getUser(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
	List<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}", method=RequestMethod.DELETE)
	public User deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

Spring MVC 是 Spring Framework 核心的一部分，详细介绍可参考其[参考文档](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc)。[spring.io/guides](https://spring.io/guides) 还提供了几个 Spring MVC 相关的指南。

<a id="boot-features-spring-mvc-auto-configuration"></a>

#### 28.1.1、Spring MVC 自动配置

Spring Boot 提供了适用于大多数 Spring MVC 应用的自动配置（auto-configuration）。

自动配置在 Spring 默认功能上添加了以下功能：

- 引入 `ContentNegotiatingViewResolver` 和 `BeanNameViewResolver` bean。
- 支持服务静态资源，包括对 WebJar 的支持（[见下文](#boot-features-spring-mvc-static-content)）。
- 自动注册 `Converter`、`GenericConverter` 和 `Formatter` bean。
- 支持 `HttpMessageConverter`（见[下文](#boot-features-spring-mvc-message-converters)）。
- 自动注册 `MessageCodesResolver`（见[下文](#boot-features-spring-message-codes)）。
- 支持静态 index.html。
- 支持自定义 Favicon （见[下文](#boot-features-spring-mvc-favicon)）。
- 自动使用 `ConfigurableWebBindingInitializer` bean（见[下文](#boot-features-spring-mvc-web-binding-initializer)）。

如果您想保留 Spring Boot MVC 的功能，并且需要添加其他 [MVC 配置](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc)（interceptor、formatter 和视图控制器等），可以添加自己的 `WebMvcConfigurerAdapter` 类型的 `@Configuration` 类，但**不能**带 `@EnableWebMvc` 注解。如果您想自定义 `RequestMappingHandlerMapping`、`RequestMappingHandlerAdapter` 或者 `ExceptionHandlerExceptionResolver` 实例，可以声明一个 `WebMvcRegistrationsAdapter` 实例来提供这些组件。

如果您想完全掌控 Spring MVC，可以添加自定义注解了 `@EnableWebMvc` 的 @Configuration 配置类。

<a id="boot-features-spring-mvc-message-converters"></a>

#### 28.1.2、HttpMessageConverters

Spring MVC 使用 `HttpMessageConverter` 接口来转换 HTTP 的请求和响应。开箱即用功能包含了合适的默认值，比如对象可以自动转换为 JSON（使用 Jackson 库）或者 XML（优先使用 Jackson XML 扩展，其次为 JAXB）。字符串默认使用 `UTF-8` 编码。

如果您需要添加或者自定义转换器（converter），可以使用 Spring Boot 的 `HttpMessageConverters` 类：

```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration
public class MyConfiguration {

	@Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}

}
```

上下文中的所有 `HttpMessageConverter` bean 都将被添加到转换器列表中。您也可以用这种方式来覆盖默认转换器。

<a id="boot-features-json-components"></a>

#### 28.1.3、自定义 JSON Serializer 和 Deserializer

如果您使用 Jackson 序列化和反序列化 JSON 数据，可能需要自己编写 `JsonSerializer` 和 `JsonDeserializer` 类。自定义序列化器（serializer）的做法通常是通过[一个模块来注册 Jackson](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers)， 然而 Spring Boot 提供了一个备选的 `@JsonComponent` 注解，它可以更加容易地直接注册 Spring Bean。

您可以直接在 `JsonSerializer` 或者 `JsonDeserializer` 实现上使用 `@JsonComponent` 注解。您也可以在将序列化器/反序列化器（deserializer）作为内部类的类上使用。例如：

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

	public static class Serializer extends JsonSerializer<SomeObject> {
		// ...
	}

	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// ...
	}

}
```

`ApplicationContext` 中所有的 `@JsonComponent` bean 将被自动注册到 Jackson 中，由于 `@JsonComponent` 使用 `@Component` 注解标记，因此组件扫描（component-scanning）规则将对其生效。

Spring Boot 还提供了 [JsonObjectSerializer](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java) 和 [JsonObjectDeserializer](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java) 基类，它们在序列化对象时为标准的 Jackson 版本提供了有用的替代方案。有关详细信息，请参阅 Javadoc 中的 [JsonObjectSerializer](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/jackson/JsonObjectSerializer.html) 和 [JsonObjectDeserializer](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/jackson/JsonObjectDeserializer.html)。

<a id="boot-features-spring-message-codes"></a>

#### 28.1.4、MessageCodesResolver

Spring MVC 有一个从绑定错误中生成错误码的策略，用于渲染错误信息：`MessageCodesResolver`。如果您设置了 `spring.mvc.message-codes-resolver.format` 属性值为 `PREFIX_ERROR_CODE` 或 `POSTFIX_ERROR_CODE`，Spring Boot 将为你创建该策略（请参阅 [DefaultMessageCodesResolver.Format](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html) 中的枚举）。

<a id="boot-features-spring-mvc-static-content"></a>

#### 28.1.5、静态内容

默认情况下，Spring Boot 将在 classpath 或者 `ServletContext` 根目录下从名为 `/static` （`/public`、`/resources` 或 `/META-INF/resources`）目录中服务静态内容。它使用了 Spring MVC 的 `ResourceHttpRequestHandler`，因此您可以通过添加自己的 `WebMvcConfigurerAdapter` 并重写 `addResourceHandlers` 方法来修改此行为。

在一个独立的（stand-alone） web 应用程序中，来自容器的默认 servlet 也是被启用的，并充当一个回退支援，Spring 决定不处理 `ServletContext` 根目录下的静态资源，容器的默认 servlet 也将会处理。大多情况下，这是不会发生的（除非您修改了默认的 MVC 配置），因为 Spring 始终能通过 `DispatcherServlet` 来处理请求。

默认情况下，资源被映射到 `/**`，但可以通过 `spring.mvc.static-path-pattern` 属性调整。比如，将所有资源重定位到 `/resources/**`：

```
spring.mvc.static-path-pattern=/resources/**
```

您还可以使用 `spring.resources.static-locations` 属性来自定义静态资源的位置（使用一个目录位置列表替换默认值）。根 Servlet context path `/` 自动作为一个 location 添加进来。

除了上述提到的**标准**静态资源位置之外，还有一种特殊情况是用于 [Webjar 内容](https://www.webjars.org/)。如果以 Webjar 格式打包，则所有符合 `/webjars/**` 的资源都将从 jar 文件中服务。

**提示**

> 如果您的应用程序要包成 jar，请不要使用 `src/main/webapp` 目录。虽然此目录是一个通用标准，但它**只**适用于 war 打包，如果生成的是一个 jar，它将被绝大多数的构建工具所忽略。

Spring Boot 还支持 Spring MVC 提供的高级资源处理功能，允许使用例如静态资源缓存清除（cache busting）或者 Webjar 版本无关 URL。

要使用 Webjar 版本无关 URL 功能，只需要添加 `webjars-locator-core` 依赖。然后声明您的 Webjar，以 jQuery 为例，添加的 `"/webjars/jquery/dist/jquery.min.js"` 将变成 `"/webjars/jquery/x.y.z/dist/jquery.min.js"`，其中 `x.y.z` 是 Webjar 的版本。

**注意**

> 如果您使用 JBoss，则需要声明 `webjars-locator-jboss-vfs` 依赖，而不是 `webjars-locator-core`，否则所有 Webjar 将被解析成 `404`。

要使用缓存清除功能，以下配置为所有静态资源配置了一个缓存清除方案，实际上是在 URL 上添加了一个内容哈希，例如 `<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`：

```
pring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

**注意**

> 模板中的资源链接在运行时被重写，这得益于 `ResourceUrlEncodingFilter` 为 Thymeleaf 和 FreeMarker 自动配置。在使用 JSP 时，您应该手动声明此过滤器。其他模板引擎现在还不会自动支持，但可以与自定义模板宏（macro）/helper 和 [`ResourceUrlProvider`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceUrlProvider.html) 结合使用。

当使用例如 Javascript 模块加载器动态加载资源时，重命名文件是不可选的。这也是为什么支持其他策略并且可以组合使用的原因。**fixed**策略将在 URL 中添加一个静态版本字符串，而不是更改文件名：

```ini
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

使用此配置，JavaScript 模块定位在 `"/js/lib/"` 下使用固定版本策略（`"/v12/js/lib/mymodule.js"`），而其他资源仍使用内容策略（`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`）。

有关更多支持选项，请参阅 [ResourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java)。

**提示**

> 该功能已经在一个专门的[博客文章](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources)和 Spring 框架的[参考文档](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources)中进行了详细描述。

<a id="boot-features-spring-mvc-welcome-page"></a>

#### 28.1.6、欢迎页面

Spring Boot 支持静态和模板化的欢迎页面。它首先在配置的静态内容位置中查找 `index.html` 文件。如果找不到，则查找 `index` 模板。如果找到其中任何一个，它将自动用作应用程序的欢迎页面。

<a id="boot-features-spring-mvc-favicon"></a>

#### 28.1.7、自定义 Favicon

Spring Boot 在配置的静态内容位置和根 classpath 中查找 `favicon.ico`（按顺序）。如果该文件存在，则将被自动用作应用程序的 favicon。

<a id="boot-features-spring-mvc-pathmatch"></a>

#### 28.1.8、路径匹配与内容协商

Spring MVC 可以通过查看请求路径并将其与应用程序中定义的映射相匹配，将传入的 HTTP 请求映射到处理程序（例如 Controller 方法上的 `@GetMapping` 注解）。

Spring Boot 默认选择禁用后缀模式匹配，这意味着像 `"GET /projects/spring-boot.json"` 这样的请求将不会与 `@GetMapping("/projects/spring-boot")` 映射匹配。这被视为是 [Spring MVC 应用程序的最佳实践](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)。此功能在过去对于 HTTP 客户端没有发送正确的 **Accept** 请求头的情况还是很有用的，我们需要确保将正确的内容类型发送给客户端。如今，内容协商（Content Negotiation）更加可靠。

还有其他方法可以处理 HTTP 客户端发送不一致 **Accept** 请求头问题。我们可以使用查询参数来确保像 `"GET /projects/spring-boot?format=json"` 这样的请求映射到 `@GetMapping("/projects/spring-boot")`，而不是使用后缀匹配：

```ini
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

如果您了解相关注意事项并仍希望应用程序使用后缀模式匹配，则需要以下配置：

```ini
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

或者，不打开所有后缀模式，仅打开支持已注册的后缀模式更加安全：

```ini
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

<a id="boot-features-spring-mvc-web-binding-initializer"></a>

#### 28.1.9、ConfigurableWebBindingInitializer

Spring MVC 使用一个 `WebBindingInitializer` 为特定的请求初始化 `WebDataBinder`。如果您创建了自己的 `ConfigurableWebBindingInitializer` `@Bean`，Spring Boot 将自动配置 Spring MVC 使用它。

<a id="boot-features-spring-mvc-template-engines"></a>

#### 28.1.10、模板引擎

除了 REST web 服务之外，您还可以使用 Spring MVC 来服务动态 HTML 内容。Spring MVC 支持多种模板技术，包括 Thymeleaf、FreeMarker 和 JSP。当然，许多其他模板引擎也有自己的 Spring MVC 集成。

Spring Boot 包含了以下的模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.apache.org/docs/)
- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)
- [Thymeleaf](http://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

**提示**

> 如果可以，请尽量避免使用 JSP，当使用了内嵌 servlet 容器，会有几个[已知限制](#boot-features-jsp-limitations)。

当您使用这些模板引擎的其中一个并附带了默认配置时，您的模板将从 `src/main/resources/templates` 自动获取。

**提示**

> IntelliJ IDEA 根据您运行应用程序的方式来对 classpath 进行不同的排序。在 IDE 中通过 main 方法来运行应用程序将导致与使用 Maven 或 Gradle 或来以 jar 包方式引用程序的排序有所不同，可能会导致 Spring Boot 找不到 classpath 中的模板。如果您碰到到此问题，可以重新排序 IDE 的 classpath 来放置模块的 classes 和 resources 到首位。或者，您可以配置模板前缀来搜索 classpath 中的每一个 `templates` 目录，比如：`classpath*:/templates/`。 

<a id="boot-features-error-handling"></a>

#### 28.1.11、错误处理

默认情况下，Spring Boot 提供了一个使用了比较合理的方式来处理所有错误的 `/error` 映射，其在 servlet 容器中注册了一个**全局**错误页面。对于机器客户端而言，它将产生一个包含错误、HTTP 状态和异常消息的 JSON 响应。对于浏览器客户端而言，将以 HTML 格式呈现相同数据的 **whitelabel** 错误视图（可添加一个解析到 `error` 的 `View` 进行自定义）。要完全替换默认行为，您可以实现 `ErrorController` 并注册该类型的 bean，或者简单地添加一个类型为 `ErrorAttributes` 的 bean 来替换内容，但继续使用现用机制。

**提示**

> `BasicErrorController` 可以作为自定义 `ErrorController` 的基类，这非常有用，尤其是在您想添加一个新的内容类型（默认专门处理 `text/html`，并为其他内容提供后备）处理器的情况下。要做到这点，您只需要继承 `BasicErrorController` 并添加一个带有 `produces` 属性的 `@RequestMapping` 注解的公共方法，之后创建一个新类型的 bean。

您还可以定义一个带有 `@ControllerAdvice` 注解的类来自定义为特定控制器或异常类型返回的 JSON 文档：

```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

	@ExceptionHandler(YourException.class)
	@ResponseBody
	ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
	}

	private HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		return HttpStatus.valueOf(statusCode);
	}

}
```

以上示例中，如果同包下定义的控制器 `AcmeController` 抛出了 `YourException`，则将使用 `CustomerErrorType` 类型的 POJO 来代替 `ErrorAttributes` 做 JSON 呈现。

<a id="boot-features-error-handling-custom-error-pages"></a>

##### 28.1.11.1、自定义错误页面

如果您想在自定义的 HTML 错误页面上显示给定的状态码，请将文件添加到 `/error` 文件夹中。错误页面可以是静态 HTML（添加在任意静态资源文件夹下) 或者使用模板构建。文件的名称应该是确切的状态码或者一个序列掩码。

例如，要将 `404` 映射到一个静态 HTML 文件，文件夹结构可以如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

使用 FreeMarker 模板来映射所有 `5xx` 错误，文件夹的结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftl
             +- <other templates>
```

对于更复杂的映射，您还通过可以添加实现了 `ErrorViewResolver` 接口的 bean 来处理：

```java
public class MyErrorViewResolver implements ErrorViewResolver {

	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request,
			HttpStatus status, Map<String, Object> model) {
		// Use the request or status to optionally return a ModelAndView
		return ...
	}

}
```

您还可以使用常规的 Spring MVC 功能，比如 [`@ExceptionHandler` 方法](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers)和 [@ControllerAdvice](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-ann-controller-advice)`。之后，ErrorController` 将能接收任何未处理的异常。

<a id="boot-features-error-handling-mapping-error-pages-without-mvc"></a>

##### 28.1.11.2、映射到 Spring MVC 之外的错误页面

对于不使用 Spring MVC 的应用程序，您可以使用 `ErrorPageRegistrar` 接口来直接注册 `ErrorPages`。抽象部分直接与底层的内嵌 servlet 容器一起工作，即使您没有 Spring MVC `DispatcherServlet` 也能使用。

```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
	return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
	}

}
```

**注意**

> 如果您注册了一个 `ErrorPage`，它的路径最终由一个 `Filter`（例如，像一些非 Spring web 框架一样，比如 Jersey 和 Wicket）处理，则必须将 Filter 显式注册为一个 `ERROR` dispatcher，如下示例：

```java
@Bean
public FilterRegistrationBean myFilter() {
	FilterRegistrationBean registration = new FilterRegistrationBean();
	registration.setFilter(new MyFilter());
	...
	registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
	return registration;
}
```

请注意，默认的 `FilterRegistrationBean` 不包含 `ERROR` 调度器（dispatcher）类型。

**当心**：当部署到 servlet 容器时，Spring Boot 使用其错误页面过滤器会将有错误状态的请求转发到相应的错误页面。如果尚未提交响应，则只能将请求转发到正确的错误页面。默认情况下，WebSphere Application Server 8.0 及更高版本在成功完成 servlet 的 service 方法后提交响应。您应该将 `com.ibm.ws.webcontainer.invokeFlushAfterService` 设置为 `false` 来禁用此行为。

<a id="boot-features-spring-hateoas"></a>

#### 28.1.12、Spring HATEOAS

如果您想开发一个使用超媒体（hypermedia）的 RESTful API，Spring Boot 提供的 Spring HATEOAS 自动配置在大多数应用程序都工作得非常好。自动配置取代了 `@EnableHypermediaSupport` 的需要，并注册了一些 bean，以便能轻松构建基于超媒体的应用程序，其包括了一个 `LinkDiscoverers` （用于客户端支持）和一个用于配置将响应正确呈现的 `ObjectMapper`。`ObjectMapper` 可以通过设置 `spring.jackson.*` 属性或者 `Jackson2ObjectMapperBuilder` bean （如果存在）自定义。

您可以使用 `@EnableHypermediaSupport` 来控制 Spring HATEOAS 的配置。请注意，这使得上述的自定义 ObjectMapper 被禁用。

<a id="boot-features-cors"></a>

#### 28.1.13、CORS 支持

[跨域资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（Cross-origin resource sharing，CORS）是[大多数浏览器](https://caniuse.com/#feat=cors)实现的一个 [W3C 规范](https://www.w3.org/TR/cors/)，其可允许您以灵活的方式指定何种跨域请求可以被授权，而不是使用一些不太安全和不太强大的方式（比如 IFRAME 或者 JSONP）。

Spring MVC 从 4.2 版本起开始[支持 CORS](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#cors)。您可在 Spring Boot 应用程序中使用 [`@CrossOrigin`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#controller-method-cors-configuration) 注解配置[控制器方法启用 CORS](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#controller-method-cors-configuration)。还可以通过注册一个 WebMvcConfigurer bean 并自定义 `addCorsMappings(CorsRegistry)` 方法来定义[全局 CORS 配置](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#global-cors-configuration)：

```java
@Configuration
public class MyConfiguration {

	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
}
```

<a id="boot-features-webflux"></a>

### 28.2、Spring WebFlux 框架

Spring WebFlux 是 Spring Framework 5.0 中新引入的一个响应式 Web 框架。与 Spring MVC 不同，它不需要 Servlet API，完全异步且无阻塞，并通过 [Reactor 项目](http://www.reactive-streams.org/)实现[响应式流（Reactive Streams）](http://www.reactive-streams.org/)规范。

Spring WebFlux 有两个版本：函数式和基于注解。基于注解的方式非常接近 Spring MVC 模型，如下所示：

```java
@RestController
@RequestMapping("/users")
public class MyRestController {

	@GetMapping("/{user}")
	public Mono<User> getUser(@PathVariable Long user) {
		// ...
	}

	@GetMapping("/{user}/customers")
	public Flux<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@DeleteMapping("/{user}")
	public Mono<User> deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

**WebFlux.fn** 为函数式调用方式，它将路由配置与请求处理分开，如下所示：

```java
@Configuration
public class RoutingConfiguration {

	@Bean
	public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
		return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
				.andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
				.andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
	}

}

@Component
public class UserHandler {

	public Mono<ServerResponse> getUser(ServerRequest request) {
		// ...
	}

	public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
		// ...
	}

	public Mono<ServerResponse> deleteUser(ServerRequest request) {
		// ...
	}
}
```

WebFlux 是 Spring Framework 的一部分，详细信息可查看其[参考文档](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn)。

**提示**

> 您可以根据需要定义尽可能多的 `RouterFunction` bean 来模块化路由定义。如果需要设定优先级，Bean 可以指定顺序。

首先，将 `spring-boot-starter-webflux` 模块添加到您的应用程序中。

**注意**

> 在应用程序中同时添加 `spring-boot-starter-web` 和 `spring-boot-starter-webflux` 模块会导致Spring Boot 自动配置 Spring MVC，而不是使用 WebFlux。这样做的原因是因为许多 Spring 开发人员将 `spring-boot-starter-webflux` 添加到他们的 Spring MVC 应用程序中只是为了使用响应式 `WebClient`。 您仍然可以通过设置 `SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)` 来强制执行您选择的应用程序类型。

<a id="boot-features-webflux-auto-configuration"></a>

#### 28.2.1、Spring WebFlux 自动配置

Spring Boot 为 Spring WebFlux 提供自动配置，适用于大多数应用程序。

自动配置在 Spring 的默认基础上添加了以下功能：

- 为 `HttpMessageReader` 和 `HttpMessageWriter` 实例配置编解码器（[稍后将介绍](#boot-features-webflux-httpcodecs)）。
- 支持提供静态资源，包括对 WebJars 的支持（[后面将介绍](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-static-content)）。

如果你要保留 Spring Boot WebFlux 功能并且想要添加其他 [WebFlux 配置](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#web-reactive)，可以添加自己的 `@Configuration` 类，类型为 `WebFluxConfigurer`，但不包含 `@EnableWebFlux`。

如果您想完全控制 Spring WebFlux，可以将 `@EnableWebFlux` 注解到自己的 @Configuration。

<a id="boot-features-webflux-httpcodecs"></a>

#### 28.2.2、使用 HttpMessageReader 和 HttpMessageWriter 作为 HTTP 编解码器

Spring WebFlux 使用 `HttpMessageReader` 和 `HttpMessageWriter` 接口来转换 HTTP 的请求和响应。它们通过检测 classpath 中可用的类库，配置了 `CodecConfigurer` 生成合适的默认值。

Spring Boot 通过使用 `CodecCustomizer` 实例加强定制。例如，`spring.jackson.*` 配置 key 应用于 Jackson 编解码器。

如果需要添加或自定义编解码器，您可以创建一个自定义的 `CodecCustomizer` 组件，如下所示：

```java
import org.springframework.boot.web.codec.CodecCustomizer;

@Configuration
public class MyConfiguration {

	@Bean
	public CodecCustomizer myCodecCustomizer() {
		return codecConfigurer -> {
			// ...
		}
	}

}
```

您还可以利用 [Boot 的自定义 JSON 序列化器和反序列化器](#boot-features-json-components)。

<a id="boot-features-webflux-static-content"></a>

#### 28.2.3、静态内容

默认情况下，Spring Boot 将在 classpath 或者 `ServletContext` 根目录下从名为 `/static` （`/public`、`/resources` 或 `/META-INF/resources`）目录中服务静态内容。它使用了 Spring WebFlux 的 `ResourceWebHandler`，因此您可以通过添加自己的 `WebFluxConfigurer` 并重写 `addResourceHandlers` 方法来修改此行为。

默认情况下，资源被映射到 `/**`，但可以通过 `spring.webflux.static-path-pattern` 属性调整。比如，将所有资源重定位到 `/resources/**`：

```
spring.webflux.static-path-pattern=/resources/**
```

您还可以使用 `spring.resources.static-locations` 属性来自定义静态资源的位置（使用一个目录位置列表替换默认值），如果这样做，默认的欢迎页面检测会切换到您自定义的位置。因此，如果启动时有任何其中一个位置存在 `index.html`，那么它将是应用程序的主页。

除了上述提到的**标准**静态资源位置之外，还有一种特殊情况是用于 [Webjar 内容](https://www.webjars.org/)。如果以 Webjar 格式打包，则所有符合 `/webjars/**` 的资源都将从 jar 文件中服务。

**提示**

> Spring WebFlux 应用程序并不严格依赖于 Servlet API，因此它们不能作为 war 文件部署，也不能使用 `src/main/webapp` 目录。

<a id="boot-features-webflux-template-engines"></a>

#### 28.2.4、模板引擎

除了 REST web 服务之外，您还可以使用 Spring WebFlux 来服务动态 HTML 内容。Spring WebFlux 支持多种模板技术，包括 Thymeleaf、FreeMarker 和 Mustache。

Spring Boot 包含了以下的模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.apache.org/docs/)
- [Thymeleaf](http://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

当您使用这些模板引擎的其中一个并附带了默认配置时，您的模板将从 `src/main/resources/templates` 自动获取。

<a id="boot-features-webflux-error-handling"></a>

#### 28.2.5、错误处理

Spring Boot 提供了一个 `WebExceptionHandler`，它以合理的方式处理所有错误。它在处理顺序中的位置紧接在 WebFlux 提供的处理程序之前，这些处理器排序是最后的。对于机器客户端，它会生成一个 JSON 响应，其中包含错误详情、HTTP 状态和异常消息。对于浏览器客户端，有一个 **whitelabel** 错误处理程序，它以 HTML 格式呈现同样的数据。您还可以提供自己的 HTML 模板来显示错误（[请参阅下一节](#boot-features-webflux-error-handling-custom-error-pages)）。

自定义此功能的第一步通常会沿用现有机制，但替换或扩充了错误内容。为此，您可以添加 `ErrorAttributes` 类型的 bean。

想要更改错误处理行为，可以实现 `ErrorWebExceptionHandler` 并注册该类型的 bean。因为 `WebExceptionHandler` 是一个非常底层的异常处理器，所以 Spring Boot 还提供了一个方便的 `AbstractErrorWebExceptionHandler` 来让你以 WebFlux 的方式处理错误，如下所示：

```java
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

	// Define constructor here

	@Override
	protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {

		return RouterFunctions
				.route(aPredicate, aHandler)
				.andRoute(anotherPredicate, anotherHandler);
	}

}
```

要获得更完整的功能，您还可以直接继承 `DefaultErrorWebExceptionHandler` 并覆盖相关方法。

<a id="boot-features-webflux-error-handling-custom-error-pages"></a>

##### 28.2.5.1、自定义错误页面

如果您想在自定义的 HTML 错误页面上显示给定的状态码，请将文件添加到 `/error` 文件夹中。错误页面可以是静态 HTML（添加在任意静态资源文件夹下) 或者使用模板构建。文件的名称应该是确切的状态码或者一个序列掩码。

例如，要将 `404` 映射到一个静态 HTML 文件，文件夹结构可以如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

使用 Mustache 模板来映射所有 `5xx` 错误，文件夹的结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.mustache
             +- <other templates>
```

<a id="boot-features-webflux-web-filters"></a>

#### 28.2.6、Web 过滤器

Spring WebFlux 提供了一个 `WebFilter` 接口，可以通过实现该接口来过滤 HTTP 请求/响应消息交换。在应用程序上下文中找到的 WebFilter bean 将自动用于过滤每个消息交换。

如果过滤器的执行顺序很重要，则可以实现 `Ordered` 接口或使用 `@Order` 注解来指定顺序。Spring Boot 自动配置可能为您配置了几个 Web 过滤器。执行此操作时，将使用下表中的顺序：

| Web 过滤器 | 顺序 |
|----|----|
| `MetricsWebFilter` | `Ordered.HIGHEST_PRECEDENCE + 1` |
| `WebFilterChainProxy`（Spring Security）| `-100` |
| `HttpTraceWebFilter` | `Ordered.LOWEST_PRECEDENCE - 10` |

<a id="boot-features-jersey"></a>

### 28.3、JAX-RS 与 Jersey

如果您喜欢 JAX-RS 编程模型的 REST 端点，则可以使用一个实现来替代 Spring MVC。[Jersey](https://jersey.github.io/) 和 [Apache CXF](https://cxf.apache.org/) 都能开箱即用。CXF 要求在应用程序上下文中以 `@Bean` 的方式将它注册为一个 `Servlet` 或者 `Filter`。Jersey 有部分原生 Spring 支持，所以我们也在 starter 中提供了与 Spring Boot 整合的自动配置支持。

要使用 Jersey，只需要将 `spring-boot-starter-jersey` 作为依赖引入，然后您需要一个 `ResourceConfig` 类型的 `@Bean`，您可以在其中注册所有端点：

```java
@Component
public class JerseyConfig extends ResourceConfig {

	public JerseyConfig() {
		register(Endpoint.class);
	}

}
```

**警告**

> Jersey 对于扫描可执行归档文件的支持是相当有限的。例如，它无法扫描一个[完整的可执行 jar 文件](#deployment-install)中的端点，同样，当运行一个可执行的 war 文件时，它也无法扫包中 `WEB-INF/classes` 下的端点。为了避免该限制，您不应该使用 `packages` 方法，应该使用上述的 `register` 方法来单独注册每一个端点。

您可以注册任意数量实现了 `ResourceConfigCustomizer` 的 bean，以实现更高级的定制化。

所有注册的端点都应注解了 `@Components` 并具有 HTTP 资源注解（ `@GET` 等），例如：

```java
@Component
@Path("/hello")
public class Endpoint {

	@GET
	public String message() {
		return "Hello";
	}

}
```

由于 `Endpoint` 是一个 Spring `@Component`，它的生命周期由 Spring 管理，您可以使用 `@Autowired` 注入依赖并使用 `@Value` 注入外部配置。默认情况下，Jersey servlet 将被注册并映射到 `/*`。您可以通过将 `@ApplicationPath` 添加到 `ResourceConfig` 来改变此行为。

默认情况下，Jersey 在 `ServletRegistrationBean` 类型的 `@Bean` 中被设置为一个名为 `jerseyServletRegistration` 的 Servlet。默认情况下，该 servlet 将被延迟初始化，您可以使用 `spring.jersey.servlet.load-on-startup` 自定义。您可以禁用或通过创建一个自己的同名 bean 来覆盖该 bean。您还可以通过设置 `spring.jersey.type=filter` 使用过滤器替代 servlet（该情况下， 替代或覆盖 `@Bean` 的为`jerseyFilterRegistration`）。该过滤器有一个 `@Order`，您可以使用 `spring.jersey.filter.order` 设置。可以使用 `spring.jersey.init.*` 指定一个 map 类型的 property 以给定 servlet 和过滤器的初始化参数。

这里有一个 [Jersey 示例](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-samples/spring-boot-sample-jersey)，您可以解如何设置。

<a id="boot-features-embedded-container"></a>

### 28.4、内嵌 Servlet 容器支持

Spring Boot 包含了对内嵌 [Tomcat](https://tomcat.apache.org/)、[Jetty](https://www.eclipse.org/jetty/) 和 [Undertow](http://undertow.io/) 服务器的支持。大部分开发人员只需简单地使用对应的 Starter 来获取完整的配置实例。默认情况下，内嵌服务器将监听 `8080` 上的 HTTP 请求。

**警告**

> 如果您选择在 [CentOS](https://www.centos.org/) 使用 Tomcat，请注意，默认情况下，临时目录用于储存编译后的 JSP、上传的文件等。当您的应用程序运行时发生了故障，该目录可能会被 `tmpwatch` 删除。为了避免出现该情况，您可能需要自定义 `tmpwatch` 配置，使 `tomcat.* `目录不被删除，或者配置 `server.tomcat.basedir` 让 Tomcat 使用其他位置。

<a id="boot-features-embedded-container-servlets-filters-listeners"></a>

#### 28.4.1、Servlet、Filter 与 Listener

使用内嵌 servlet 容器时，您可以使用 Spring bean 或者扫描方式来注册 Servlet 规范中的 Servlet、Filter 和所有监听器（比如 `HttpSessionListener`）。

<a id="boot-features-embedded-container-servlets-filters-listeners-beans"></a>

##### 28.4.1.1、将 Servlet、Filter 和 Listener 注册为 Spring Bean

任何 `Servlet`、`Filter` 或 `*Listener` 的 Spring bean 实例都将被注册到内嵌容器中。如果您想引用 `application.properties` 中的某个值，这可能会特别方便。

默认情况下，如果上下文只包含单个 Servlet，它将映射到 `/`。在多个 Servlet bean 的情况下，bean 的名称将用作路径的前缀。Filter 将映射到 `/*`。

如果基于约定配置的映射不够灵活，您可以使用 `ServletRegistrationBean`、`FilterRegistrationBean` 和 `ServletListenerRegistrationBean` 类来完全控制。

Spring Boot 附带了许多可以定义 Filter bean 的自动配置。以下是部分过滤器及其执行顺序的（顺序值越低，优先级越高）：

| Servlet Filter | 顺序 |
| ---- | ----|
| `OrderedCharacterEncodingFilter` | `Ordered.HIGHEST_PRECEDENCE` |
| `WebMvcMetricsFilter` | `Ordered.HIGHEST_PRECEDENCE + 1` |
| `ErrorPageFilter` | `Ordered.HIGHEST_PRECEDENCE + 1` |
| `HttpTraceFilter` | `Ordered.LOWEST_PRECEDENCE - 10` |

通常 Filter bean 无序放置也是安全的。

如果需要指定顺序，则应避免在 `Ordered.HIGHEST_PRECEDENCE` 顺序点配置读取请求体的过滤器，因为它的字符编码可能与应用程序的字符编码配置不一致。如果一个 Servlet 过滤器包装了请求，则应使用小于或等于 `OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER `的顺序点对其进行配置。

<a id="boot-features-embedded-container-context-initializer"></a>

#### 28.4.2、Servlet 上下文初始化

内嵌 servlet 容器不会直接执行 Servlet 3.0+ 的 `javax.servlet.ServletContainerInitializer` 接口或 Spring 的 `org.springframework.web.WebApplicationInitializer` 接口。这是一个有意的设计决策，旨在降低在 war 内运行时第三方类库产生的风险，防止破坏 Sring Boot 应用程序。

如果您需要在 Spring Boot 应用程序中执行 servlet 上下文初始化，则应注册一个实现了 `org.springframework.boot.context.embedded.ServletContextInitializer` 接口的 bean。`onStartup` 方法提供了针对 `ServletContext` 的访问入口，如果需要，它可以容易作为现有 `WebApplicationInitializer` 的适配器。

<a id="boot-features-embedded-container-servlets-filters-listeners-scanning"></a>

##### 28.4.2.1、扫描 Servlet、Filter 和 Listener

使用内嵌容器时，可以使用 `@ServletComponentScan` 启用带 `@WebServlet`、`@WebFilter` 和 `@WebListener` 注解的类自动注册。

**提示**

`@ServletComponentScan` 在独立（standalone）容器中不起作用，因容器将使用内置发现机制来代替。

<a id="boot-features-embedded-container-application-context"></a>

##### 28.4.3、ServletWebServerApplicationContext

Spring Boot 底层使用了一个不同的 `ApplicationContext` 类型来支持内嵌 servlet。`ServletWebServerApplicationContext` 是一个特殊 `WebApplicationContext` 类型，它通过搜索单个 `ServletWebServerFactory` bean 来引导自身。通常，`TomcatServletWebServerFactory`、 `JettyServletWebServerFactory` 或者 `UndertowServletWebServerFactory` 中的一个将被自动配置。

**注意**

> 通常，你不需要知道这些实现类。大部分应用程序会自动配置，并为您创建合适的 `ApplicationContext` 和 `ServletWebServerFactory`。

**待续……**