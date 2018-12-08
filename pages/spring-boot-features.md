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

> `SPRING_APPLICATION_JSON` 属性可以在命令行中提供一个环境变量。比如在 UN*X shell 中：
```
$$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
```
> 在此示例中，您可以在 Spring `Environment` 中使用 `acme.name=test`，也可以在系统属性（System property）中将 JSON 以 `spring.application.json` 属性提供：
```
$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
```
或者以命令行参数形式：
```
$ java -jar myapp.jar --spring.application.json='{"name":"test"}'
```
或者将 JSON 作为一个 JNDI 变量：`java:comp/env/spring.application.json`。

**待续……**