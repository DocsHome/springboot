<a id="boot-features"></a>

# 四、Spring Boot 特性

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

如果您需要构建一个有层级关系的 `ApplicationContext`（具有父/子关系的多上下文），或者偏向使用 **fluent**（流式）构建器 API，可以使用 `SpringApplicationBuilder`。

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

<a id="boot-features-external-config-random-values"></a>

### 24.1、配置随机值

`RandomValuePropertySource` 对于随机值注入非常有用（比如在保密场景或者测试用例中)。它可以产生 integer、long、uuid 和 string。如下示例：

```ini
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

`random.int*` 语法为 `OPEN value (,max) CLOSE`，`OPEN,CLOSE` 可为任意字符，`value,max` 为整数。如果使用了 `max`，`value` 则为最小值，`max` 为最大值。

<a id="boot-features-external-config-command-line-args"></a>

### 24.2、访问命令行属性

默认情况下，`SpringApplication` 将所有命令行选项参数（即以 `--` 开头的参数，比如 `--server.port=9000`）转换为属性，并将它们添加到 Spring `Environment` 中。如之前所述，命令行属性始终优先于其他属性源。

如果您不希望将命令行属性添加到 `Environment`，可以使用 `SpringApplication.setAddCommandLineProperties(false)` 来禁用它们。

<a id="boot-features-external-config-application-property-files"></a>

### 24.3、应用程序属性文件

`SpringApplication` 从以下位置的 `application.properties` 文件中加载属性（properties），并将它们添加到 Spring `Environment` 中：

1. 当前目录的 `/config` 子目录
2. 当前目录
3. classpath 上的 `/config` 包
4. classpath 根路径

列表按序号优先级排序，序号越小，优先级越高。

**注意**

> 您还可以使用 [YAML（.yml）](#boot-features-external-config-yaml)文件来替代 **.properties**。

如果您不喜欢 `application.properties` 作为配置文件名，则可以通过指定 `spring.config.name` 环境属性来切换到另一个文件名。您还可以使用 `spring.config.location` 环境属性来引用一个显式位置（以逗号分隔的目录位置或文件路径列表）。以下示例展示了如何指定其他文件名：

```
$ java -jar myproject.jar --spring.config.name=myproject
```

以下示例展示了如何指定两个位置：

```
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```
**警告**

> `spring.config.name` 和 `spring.config.location` 在程序启动早期就用来确定哪些文件必须加载，因此必须将它们定义为环境属性（通常是 OS 环境变量、系统属性或命令行参数）。

如果 `spring.config.location` 包含目录（而不是文件），则它们应该以 `/` 结尾（并且在运行期间，在加载之前追加从 `spring.config.name` 生成的名称，包括指定 profile 的文件名）。 `spring.config.location` 中指定的文件按原样使用，不支持指定 profile 形式，并且可被任何指定 profile 的文件的属性所覆盖。

配置位置以相反的顺序搜索。默认情况下，配置的位置为 `classpath:/,classpath:/config/,file:./,file:./config/`。生成的搜索顺序如下：

1. `file:./config/`
2. `file:./`
3. `classpath:/config/`
4. `classpath:/`

使用了 `spring.config.location` 配置自定义配置位置时，默认位置配置将被替代。例如，如果 `spring.config.location` 配置为 `classpath:/custom-config/,file:./custom-config/`，搜索顺序将变为以下：

1. `file:./custom-config/`
2. `classpath:custom-config/`

或者，当使用 `spring.config.additional-location` 配置自定义配置位置时，除了使用默认位置外，还会使用它们。这些其他（additional）位置将在默认位置之前搜索。例如，如果将其他位置配置为 `classpath:/custom-config/,file:./custom-config/`，则搜索顺序将变为以下内容：

1. `file:./custom-config/`
2. `classpath:custom-config/`
3. `file:./config/`
4. `file:./`
5. `classpath:/config/`
6. `classpath:/`

该搜索顺序允许您在一个配置文件中指定默认值，然后有选择地覆盖另一个配置文件中的值。您可以在 `application.properties`（或您使用 `spring.config.name` 指定的其他文件）中的某个默认位置为应用程序提供默认值。之后，在运行时，这些默认值将被自定义位置中的某个文件所覆盖。

**注意**

> 如果您使用的是环境变量而不是系统属性，大部分操作系统都不允许使用 `.` 分隔的键名，但您可以使用下划线来代替（例如，使用 `SPRING_CONFIG_NAME` 而不是 `spring.config.name`）。

**注意**

> 如果应用程序在容器中运行，则可以使用 JNDI 属性（`java:comp/env`）或 servlet 上下文初始化参数来代替环境变量或系统属性。

<a id="boot-features-external-config-profile-specific-properties"></a>

### 24.4、特定 Profile 的属性文件

除 `application.properties` 文件外，还可以使用以下命名约定定义特定 profile 的属性文件：`application-{profile}.properties`。`Environment` 有一组默认配置文件（默认情况下为 `default`），如果未设置激活的（active）profile，则使用这些配置文件。换句话说，如果没有显式激活 profile，则会加载 `application-default.properties` 中的属性。

特定 profile 的属性文件从与标准 `application.properties` 相同的位置加载，特定 profile 的属性文件无论是否在打包的 jar 内部，都始终覆盖非特定文件。

如果指定了多个配置文件，则应用 last-wins 策略（优先采取最后一个）。例如，`spring.profiles.active` 属性指定的配置文件是在使用 `SpringApplication` API 配置的配置文件之后添加的，因此优先应用。

**注意**

> 如果在 `spring.config.location` 中指定了文件，则不考虑这些文件的特定 profile 形式。如果您还想使用特定 profile 的属性文件，请在 `spring.config.location` 中使用目录形式。

<a id="boot-features-external-config-placeholders-in-properties"></a>

### 24.5、属性中的占位符

`application.properties` 中的值在使用时通过现有的 `Environment` 进行过滤，因此您可以返回之前定义的值（例如，从系统属性）。

```ini
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

**提示**

> 您还可以使用此技术创建现有 Spring Boot 属性的**简短**形式。有关详细信息，请参见[第 77.4 章节：使用**简短**命令行参数](hwo-to.md#howto-use-short-command-line-arguments)。

<a id="boot-features-encrypting-properties"></a>

### 24.6、加密属性

Spring Boot 没有为加密属性值提供任何内置支持，然而，它提供了修改 Spring `Environment` 包含的值所必需的钩子。`EnvironmentPostProcessor` 接口允许您在应用程序启动之前操作 `Environment`。有关详细信息，请参见[第 76.3 章节：在启动前自定义 Environment 或 ApplicationContext](how-to.md#howto-customize-the-environment-or-application-context)。

如果您正在寻找一种可用于存储凭据和密码的安全方法，[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/) 项目支持在 [HashiCorp Vault](https://www.vaultproject.io/) 中存储外部化配置。


<a id="boot-features-external-config-yaml"></a>

### 24.7、使用 YAML 代替属性文件

[YAML](http://yaml.org/) 是 JSON 的超集，是一个可用于指定层级配置数据的便捷格式。只要在 classpath 上有 [SnakeYAML](http://www.snakeyaml.org/) 库，`SpringApplication` 类就会自动支持 YAML 作为属性文件（properties）的替代。

**注意**

> 如果使用 **starter**，则 `spring-boot-starter` 会自动提供 SnakeYAML。

<a id="boot-features-external-config-loading-yaml"></a>

#### 24.7.1、加载 YAML

Spring Framework 提供了两个便捷类，可用于加载 YAML 文档。`YamlPropertiesFactoryBean` 将 YAML 加载为 `Properties`，`YamlMapFactoryBean` 将 YAML 加载为 `Map`。

例如以下 YAML 文档：

```yaml
environments:
  dev:
	url: http://dev.example.com
    name: Developer Setup
  prod:
	url: http://another.example.com
	name: My Cool App
```

前面的示例将转换为以下属性（properties）：

```ini
environments.dev.url=http://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=http://another.example.com
environments.prod.name=My Cool App
```

YAML 列表表示带有 `[index]` 下标引用的属性键。例如以下 YAML：

```yaml
my:
servers:
  - dev.example.com
  - another.example.com
```

以上示例将转成以下属性：

```ini
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

要使用 Spring Boot 的 `Binder` 工具来绑定这样配置到属性（这是 `@ConfigurationProperties` 所做的），你需要在目标 bean 中有一个 `java.util.List`（或 `Set`）类型的属性，你需要为其提供一个 setter 或者使用可变值初始化它。 例如，以下示例展示将上述的配置与属性绑定：

```java
@ConfigurationProperties(prefix="my")
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```

<a id="boot-features-external-config-exposing-yaml-to-spring"></a>

#### 24.7.2、在 Spring Environment 中将 YAML 暴露为属性

`YamlPropertySourceLoader` 类可用于在 Spring `Environment` 中将 YAML 暴露为 `PropertySource`。这样做可以让您使用带占位符语法的 `@Value` 注解来访问 YAML 属性。

<a id="boot-features-external-config-multi-profile-yaml"></a>

#### 24.7.3、多 profile YAML 文档

您可以使用 `spring.profiles` key 在单个文件中指定多个特定 profile 的 YAML 文档，以指示文档何时应用，如下所示：

```yaml
server:
  address: 192.168.1.100
---
spring:
  profiles: development
server:
  address: 127.0.0.1
---
spring:
  profiles: production & eu-central
server:
  address: 192.168.1.120
```

在前面示例中，如果 `development` profile 处于激活状态，则 `server.address` 属性得值为 `127.0.0.1`。 同样，如果 `production` 和 `eu-central` profile 处于激活状态，则 `server.address` 属性的值为 `192.168.1.120`。如果未激活 `development`、`production` 或 `eu-central` profile，则该属性的值为 `192.168.1.100`。

**注意**

> 因此，`spring.profiles` 可以包含一个简单的 profile 名称（例如 `production`）或一个 profile 表达式。profile 表达式允许表达更复杂的 profile 逻辑，例如 `production & (eu-central | eu-west)`。有关详细信息，请查阅[参考指南](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。

如果在应用程序上下文启动时没有显式激活，则激活默认 profile。因此，在以下 YAML 中，我们为 `spring.security.user.password` 设置了一个值，该值**仅**在 `default` profile 中可用：

```yaml
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
      password: weak
```

然而，在以下示例中，始终设置密码，因为它未附加到任何 profile，如果需要更改，必须在所有其他 profile 中显式重置：

```yaml
server:
  port: 8000
spring:
  security:
    user:
      password: weak
```

使用 `spring.profiles` 元素来指定 Spring profile 可以选择通过使用 `!` 字符来取反（否定）。如果为单个文档指定了否定和非否定的 profile，则至少一个非否定的 profile 必须匹配，没有否定的 profile 可以匹配。

<a id="boot-features-external-config-yaml-shortcomings"></a>

#### 24.7.4、YAML 的缺点

无法使用 `@PropertySource` 注解加载 YAML 文件。因此，如果您需要以这种方式加载值，请使用属性文件（properties）。

<a id="boot-features-external-config-typesafe-configuration-properties"></a>

### 24.8、类型安全的配置属性

使用 `@Value("${property}")` 注解来注入配置属性有时会很麻烦，特别是如果您使用了多个属性或者您的数据本质上是分层结构。Spring Boot 提供了另一种使用属性的方法，该方法使用强类型的 bean 来管理和验证应用程序的配置，如下所示：

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

	private boolean enabled;

	private InetAddress remoteAddress;

	private final Security security = new Security();

	public boolean isEnabled() { ... }

	public void setEnabled(boolean enabled) { ... }

	public InetAddress getRemoteAddress() { ... }

	public void setRemoteAddress(InetAddress remoteAddress) { ... }

	public Security getSecurity() { ... }

	public static class Security {

		private String username;

		private String password;

		private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

		public String getUsername() { ... }

		public void setUsername(String username) { ... }

		public String getPassword() { ... }

		public void setPassword(String password) { ... }

		public List<String> getRoles() { ... }

		public void setRoles(List<String> roles) { ... }

	}
}
```

前面的 POJO 定义了以下属性：

- `acme.enabled`，默认值为 `false`。
- `acme.remote-address`，可以从 String 强制转换的类型。
- `acme.security.username`，内嵌一个 **security** 对象，其名称由属性名称决定。特别是，返回类型根本没有使用，可能是 `SecurityProperties`。
- `acme.security.password`。
- `acme.security.roles`，String 集合。

**注意**

<blockquote>

getter 和 setter 通常是必需的，因为绑定是通过标准的 Java Bean 属性描述符来完成，就像在 Spring MVC 中一样。以下情况可以省略 setter：

- Map，只要它们要初始化，就需要一个 getter 但不一定需要setter，因为它们可以被 binder 修改。
- 集合和数组可以通过一个索引（通常使用 YAML）或使用单个逗号分隔值（属性）进行访问。最后一种情况必须使用 setter。我们建议始终为此类型添加 setter。如果初始化集合，请确保它是可变的（如上例所示）。
- 如果初始化嵌套的 POJO 属性（如前面示例中的 `Security` 字段），则不需要 setter。如果您希望 binder 使用其默认构造函数动态创建实例，则需要一个 setter。

有些人可能会使用 Project Lombok 来自动生成 getter 和 setter。请确保 Lombok 不为此类型生成任何特定构造函数，因为容器会自动使用它来实例化对象。

最后，考虑到标准 Java Bean 属性，不支持对静态属性的绑定。

</blockquote>

**提示**

> 另请参阅 [@Value 和 @ConfigurationProperties 之间的差异](#boot-features-external-config-vs-value)。

您还需要列出要在 `@EnableConfigurationProperties` 注解中注册的属性类，如下所示：

```java
@Configuration
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

**注意**

<blockquote>

当以这种方式注册 `@ConfigurationProperties` bean 时，bean 具有一个固定格式的名称：`<prefix>-<fqn>`，其中 `<prefix>` 是 `@ConfigurationProperties` 注解中指定的环境 key 前缀，`<fqn>` 是 bean 的完全限定类名。如果注解未提供任何前缀，则仅使用 bean 的完全限定类名。

上面示例中的 bean 名称为 `acme-com.example.AcmeProperties`。

</blockquote>

即使前面的配置为 `AcmeProperties` 创建了一个 bean，我们也建议 `@ConfigurationProperties` 只处理环境（environment），特别是不要从上下文中注入其他 bean。话虽如此，`@EnableConfigurationProperties` 注解也会自动应用到您的项目，以便从 `Environment` 配置使用了 `@ConfigurationProperties` 注解的所有**现有**的 bean。您可以通过确保 `AcmeProperties` 已经是一个 bean 来快捷生成 `MyConfiguration`，如下所示：

```java
@Component
@ConfigurationProperties(prefix="acme")
public class AcmeProperties {

	// ... see the preceding example

}
```

这种配置风格特别适用于 `SpringApplication` 外部 YAML 配置，如下所示：

```yaml
# application.yml

acme:
	remote-address: 192.168.1.1
	security:
		username: admin
		roles:
		  - USER
		  - ADMIN

# additional configuration as required
```

要使用 `@ConfigurationProperties` bean，您可以使用与其他 bean 相同的方式注入它们，如下所示：

```java
@Service
public class MyService {

	private final AcmeProperties properties;

	@Autowired
	public MyService(AcmeProperties properties) {
	    this.properties = properties;
	}

 	//...

	@PostConstruct
	public void openConnection() {
		Server server = new Server(this.properties.getRemoteAddress());
		// ...
	}

}
```

**提示**

> 使用 `@ConfigurationProperties` 还可以生成元数据文件，IDE 可以通过这些文件来为您自己的 key 提供自动完成功能。有关详细信息，请参阅[附录 B：配置元数据](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#configuration-metadata)。

<a id="boot-features-external-config-3rd-party-configuration"></a>

#### 24.8.1、第三方配置

`@ConfigurationProperties` 除了可以使用来注解类之外，您还可以在公共的 @Bean 方法上使用。当您想要将属性绑定到您掌控之外的第三方组件时，这样做特别有用。

要使用 `Environment` 属性配置 bean，请将 `@ConfigurationProperties` 添加到 bean 注册上，如下所示：

```java
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
	...
}
```

使用 `another` 前缀定义的所有属性都使用与前面的 `AcmeProperties` 示例类似的方式映射到 `AnotherComponent` bean。

<a id="boot-features-external-config-relaxed-binding"></a>

#### 24.8.2、宽松绑定

Spring Boot 使用一些宽松的规则将 `Environment` 属性绑定到 `@ConfigurationProperties` bean，因此 `Environment` 属性名不需要和 bean 属性名精确匹配。常见的示例包括使用了 `-` 符号分割的环境属性（例如，`context-path` 绑定到 `contextPath`）和大写环境属性（例如，`PORT` 绑定到 `port`）。

如下 `@ConfigurationProperties` 类：

```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

	private String firstName;

	public String getFirstName() {
		return this.firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

}
```

在上述示例中，同样可以使用以下属性名称：

**表 24.1、宽松绑定**

| 属性 | 描述 |
| --- | --- |
| `acme.my-project.person.first-name` | Kebab 风格（短横线命名），建议在 `.properties` 和 `.yml` 文件中使用。 |
| `acme.myProject.person.firstName` | 标准驼峰式风格。 |
| `acme.my_project.person.first_name` | 下划线表示法，`.properties` 和 `.yaml` 文件中的另外一种格式。 |
| `ACME_MYPROJECT_PERSON_FIRSTNAME` | 大写风格，当使用系统环境变量时推荐使用该风格。 |

**注意**

> 注解的 `prefix` 值必须是 kebab (短横线命名)风格（小写并用 `-` 分隔，例如 `acme.my-project.person`）。

**表 24.2、每种属性源（property source）的宽松绑定规则**

| 属性源 | 简单类型 | 列表集合类型 |
| --- | --- | --- |
| properties 文件 | 驼峰式、短横线式或下划线式 | 标准列表语法使用 `[]` 或逗号分隔值 |
| YAML 文件 | 驼峰式、短横线式或者下划线式 | 标准 YAML 列表语法或者逗号分隔值 |
| 环境变量 | 大写并且以下划线作为定界符，`_` 不能放在属性名之间使用 | 数字值两边使用下划线连接，例如 `MY_ACME_1_OTHER = my.acme[1].other` |
| 系统属性 | 驼峰式、短横线式或者下划线式 | 标准列表语法使用 `[]` 或逗号分隔值 |

**提示**

> 我们建议，属性尽可能以小写的短横线格式存储，比如 `my.property-name=acme`。

当绑定到 `Map` 属性时，如果 key 包含除小写字母数字字符或 `-` 以外的任何内容，则需要使用括号表示法来保留原始值。如果 key 没有使用 `[]` 包裹，则里面的任何非字母数字字符或 `-` 的字符都将被删除。例如，将以下属性绑定到一个 `Map`：

```yaml
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```

上面的属性将绑定到一个 `Map` 上，其中 `/key1`，`/key2` 和 `key3` 作为 map 的 key。

<a id="boot-features-external-config-complex-type-merge"></a>

#### 24.8.3、合并复杂类型

当列表集合（list）在多个地方配置时，整个列表集合将被替换。

例如，假设带有 `name` 和 `description` 属性的 `MyPojo` 对象默认为 `null`。以下示例中，`AcmeProperties` 暴露了一个 `MyPojo` 对象列表集合：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final List<MyPojo> list = new ArrayList<>();

	public List<MyPojo> getList() {
		return this.list;
	}

}
```

配置可以如下：

```yaml
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

如果 `dev` 配置文件未激活，则 `AcmeProperties.list` 只包含一条 `MyPojo` 条目，如之前所述。但是，如果激活了 `dev` 配置文件，列表集合仍然只包含一个条目（`name` 属性值为 `my another name`，`description` 为 `null`）。此配置不会向列表集合中添加第二个 `MyPojo` 实例，也不会合并条目。

在多个配置文件中指定一个 `List` 时，最高优先级（并且只有一个）的列表集合将被使用。可做如下配置：

```yaml
acme:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

在前面示例中，如果 `dev` 配置文件处于活动状态，则 `AcmeProperties.list` 包含一个 `MyPojo` 条目（`name` 为 `my another name`，`description` 为 `null`）。对于 YAML 而言，逗号分隔的列表和YAML 列表同样会完全覆盖列表集合的内容。

对于 `Map` 属性，您可以绑定来自多个源中提取的属性值。但是，对于多个源中的相同属性，则使用高优先级最高的属性。以下示例从 `AcmeProperties` 暴露了一个 `Map<String, MyPojo>`：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final Map<String, MyPojo> map = new HashMap<>();

	public Map<String, MyPojo> getMap() {
		return this.map;
	}

}
```

可以考虑以下配置：

```yaml
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```

如果 `dev` 配置文件未激活，则 `AcmeProperties.map` 只包含一个带 `key1` key 的条目（`name` 为 `my name 1`，`description` 为 `my description 1`）。但是，如果激活了 dev 配置文件，则 `map` 将包含两个条目， key 分别为 `key1`（`name` 为 `dev name 1` 和 `description` 为 `my description 1`）和 `key2`（`name` 为 `dev name 2` 和 `description` 为 `dev description 2`）。

**注意**

> 前面的合并规则适用于所有不同属性源的属性，而不仅仅是 YAML 文件。

<a id="boot-features-external-config-conversion"></a>

#### 24.8.4、属性转换

当外部应用程序属性（application properties） 绑定到 `@ConfigurationProperties` bean 时，Spring Boot 会尝试将其属性强制转换为正确的类型。如果需要自定义类型转换，可以提供 `ConversionService` bean（名为 `conversionService` 的 bean）或自定义属性编辑器（通过 `CustomEditorConfigurer` bean）或自定义转换器（带有注解为 `@ConfigurationPropertiesBinding` 的 bean 定义）。

**注意**

由于该 bean 在应用程序生命周期早期就被请求 ，因此请限制 `ConversionService` 使用的依赖。您在创建时可能无法完全初始化所需的依赖。如果配置 key 为非强制需要，您可能希望重命名自定义的 `ConversionService`，并仅依赖于使用 `@ConfigurationPropertiesBinding` 限定的自定义转换器。

<a id="boot-features-external-config-conversion-duration"></a>

##### 转换 duration

Spring Boot 支持持续时间（duration）表达。如果您暴露一个 `java.time.Duration` 属性，则可以在应用程序属性中使用以下格式：

- 常规 `long` 表示（除非指定 `@DurationUnit`，否则使用毫秒作为默认单位）
- [`java.util.Duration`](https://docs.oracle.com/javase/8/docs/api//java/time/Duration.html#parse-java.lang.CharSequence-) 使用的标准 ISO-8601 格式
- 一种更易读的格式，值和单位在一起（例如 `10s` 表示 10 秒）

思考以下示例：

```java
@ConfigurationProperties("app.system")
public class AppSystemProperties {

	@DurationUnit(ChronoUnit.SECONDS)
	private Duration sessionTimeout = Duration.ofSeconds(30);

	private Duration readTimeout = Duration.ofMillis(1000);

	public Duration getSessionTimeout() {
		return this.sessionTimeout;
	}

	public void setSessionTimeout(Duration sessionTimeout) {
		this.sessionTimeout = sessionTimeout;
	}

	public Duration getReadTimeout() {
		return this.readTimeout;
	}

	public void setReadTimeout(Duration readTimeout) {
		this.readTimeout = readTimeout;
	}

}
```

指定一个会话超时时间为 30 秒，使用 `30`、`PT30S` 和 `30s` 等形式都是可以的。读取超时时间设置为 500ms，可以采用以下任何一种形式：`500`、`PT0.5S` 和 `500ms`。

您也可以使用任何支持的单位来标识：

- `ns` 为纳秒
- `us` 为微秒
- `ms` 为毫秒
- `s` 为秒
- `m` 为分钟
- `h` 为小时
- `d` 为天

默认单位是毫秒，可以使用 `@DurationUnit` 配合上面的单位示例重写。

**提示**

> 要从先前仅使用 `Long` 来表示持续时间的版本进行升级，如果切换到 `Duration` 时不是毫秒，请定义单位（使用 `@DurationUnit`）。这样做可以提供透明的升级路径，同时支持更丰富的格式。

<a id="boot-features-external-config-conversion-datasize"></a>

##### 转换 Data Size

Spring Framework 有一个 `DataSize` 值类型，允许以字节表示大小。如果暴露一个 `DataSize` 属性，则可以在应用程序属性中使用以下格式：

- 常规的 `Long` 表示（使用字节作为默认单位，除非指定了 `@DataSizeUnit`）
- 更具有可读性的格式，值和单位在一起（例如 `10MB` 表示 `10` 兆字节）

请思考以下示例：

```java
@ConfigurationProperties("app.io")
public class AppIoProperties {

	@DataSizeUnit(DataUnit.MEGABYTES)
	private DataSize bufferSize = DataSize.ofMegabytes(2);

	private DataSize sizeThreshold = DataSize.ofBytes(512);

	public DataSize getBufferSize() {
		return this.bufferSize;
	}

	public void setBufferSize(DataSize bufferSize) {
		this.bufferSize = bufferSize;
	}

	public DataSize getSizeThreshold() {
		return this.sizeThreshold;
	}

	public void setSizeThreshold(DataSize sizeThreshold) {
		this.sizeThreshold = sizeThreshold;
	}

}
```

要指定 10 兆字节的缓冲大小，使用 `10` 和 `10MB` 是等效的。256 字节的大小可以指定为 `256` 或 `256B`。

您也可以使用任何支持的单位：

- `B` 表示字节
- `KB` 为千字节
- `MB` 为兆字节
- `GB` 为千兆字节
- `TB` 为兆兆字节

默认单位是字节，可以使用 `@DataSizeUnit` 配合上面的示例单位重写。

**提示**

> 要从先前仅使用 `Long` 来表示大小的版本进行升级，请确保在切换到 `DataSize` 不是字节的情况下定义单位（使用 `@DataSizeUnit`）。这样做可以提供透明的升级路径，同时支持更丰富的格式。

<a id="boot-features-external-config-validation"></a>

#### 24.8.5、@ConfigurationProperties 验证

只要使用了 Spring 的 `@Validated` 注解，Spring Boot 就会尝试验证 `@ConfigurationProperties` 类。您可以直接在配置类上使用 JSR-303 `javax.validation` 约束注解。为此，请确保 JSR-303 实现在 classpath 上，然后将约束注解添加到字段上，如下所示：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	// ... getters and setters

}
```

**提示**

> 您还可以通过使用 `@Validated` 注解创建配置属性的 `@Bean` 方法来触发验证。

虽然绑定时也会验证嵌套属性，但最好的做法还是将关联字段注解上 `@Valid`。这可确保即使未找到嵌套属性也会触发验证。以下示例基于前面的 `AcmeProperties` 示例：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	@Valid
	private final Security security = new Security();

	// ... getters and setters

	public static class Security {

		@NotEmpty
		public String username;

		// ... getters and setters

	}

}
```

您还可以通过创建一个名为 `configurationPropertiesValidator` 的 bean 定义来添加自定义 Spring `Validator`。应该将 `@Bean` 方法声明为 `static`。配置属性验证器在应用程序生命周期的早期创建，将 `@Bean` 方法声明为 `static` 可以无需实例化 `@Configuration` 类来创建 bean。这样做可以避免早期实例化可能导致的意外问题。这里有一个[属性验证示例](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-samples/spring-boot-sample-property-validation)，讲解了如何设置。

**提示**

> `spring-boot-actuator` 模块包括一个暴露所有 `@ConfigurationProperties` bean 的端点。可将 Web 浏览器指向 `/actuator/configprops` 或使用等效的 JMX 端点。有关详细信息，请参阅[生产就绪功能](#production-ready-endpoints)部分。

<a id="boot-features-external-config-vs-value"></a>

#### 24.8.6、@ConfigurationProperties 与 @Value 对比

`@Value` 注解是核心容器功能，它不提供与类型安全配置属性相同的功能。下表总结了 `@ConfigurationProperties` 和 `@Value` 支持的功能：

| 功能 | `@ConfigurationProperties` | `@Value` |
| --- | --- | --- |
| [宽松绑定](#boot-features-external-config-relaxed-binding) | 是 | 否 |
| [元数据支持](#configuration-metadata) | 是 | 否 |
| `SpEL` 表达式 | 否 | 是 |

如果您要为自己的组件定义一组配置 key，我们建议您将它们分组到使用 `@ConfigurationProperties` 注解的 POJO 中。您应该知道，由于 `@Value` 不支持宽松绑定，因此如果您需要通过环境变量来提供值，它并不是一个好的选择。

最后，虽然您可以在 `@Value` 中编写 `SpEL` 表达式，但来自应用程序属性文件的此类表达式并不会被处理。

<a id="boot-features-profiles"></a>

## 25、Profile

Spring Profile 提供了一种应用程序配置部分隔离并使其仅在特定环境中可用的方法。可以使用 `@Profile` 来注解任何 `@Component` 或 `@Configuration` 以指定何时加载它，如下所示：

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {

	// ...

}
```

您可以使用 `spring.profiles.active` `Environment` 属性指定哪些配置文件处于激活状态。您可以使用本章前面介绍的任何方法指定属性。例如，您可以将其包含在 `application.properties` 中，如下所示：

```ini
spring.profiles.active=dev,hsqldb
```

您还可以在命令行上使用以下开关指定它：`--spring.profiles.active=dev,hsqldb`。

<a id="boot-features-adding-active-profiles"></a>

### 25.1、添加激活 Profile

`spring.profiles.active` 属性遵循与其他属性相同的排序规则：应用优先级最高的 `PropertySource`。这意味着您可以在 `application.properties` 中指定激活配置文件，然后使用命令行开关**替换**它们。

有时，将特定 profile 的属性**添加**到激活配置文件而不是替换它们，这种方式也是很有用的。`spring.profiles.include` 属性可无条件地添加激活配置文件。`SpringApplication` 入口还有一个 Java API，用于设置其他 profile（即，在 `spring.profiles.active` 属性激活的 profile 之上）。请参阅SpringApplication 中的 `setAdditionalProfiles()` 方法。

例如，当使用开关 `--spring.profiles.active=prod` 运行有以下属性的应用程序时，`proddb` 和 `prodmq` 配置文件也会被激活：

```yaml
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
  - proddb
  - prodmq
```

**注意**

> 请记住，可以在 YAML 文档中定义 `spring.profiles` 属性，以确定此特定文档何时包含在配置中。有关更多详细信息，请参见第 77.7 章节：[根据环境更改配置](howto.md#howto-change-configuration-depending-on-the-environment)。

<a id="boot-features-programmatically-setting-profiles"></a>

### 25.2、以编程方式设置 Profile

您可以在应用程序运行之前以编程方式通过调用 `SpringApplication.setAdditionalProfiles(...)` 设置激活 profile。也可以使用 Spring 的 `ConfigurableEnvironment` 接口激活 profile。

<a id="boot-features-profile-specific-configuration"></a>

### 25.3、特定 Profile 的配置文件

特定 profile 的 `application.properties`（或 `application.yml`）和通过 `@ConfigurationProperties` 引用的文件被当做文件并加载。有关详细信息，请参见[第 24.4 章节：特定 Profile 的属性文件](#boot-features-external-config-profile-specific-properties)。

<a id="boot-features-logging"></a>

## 26、日志记录

Spring Boot 使用 [Commons Logging](https://commons.apache.org/logging) 记录所有内部日志，但开放日志的底层实现。其为 [Java Util Logging
](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)、[Log4J2](https://logging.apache.org/log4j/2.x/) 和 [Logback](http://logback.qos.ch/) 提供了默认配置。在每种情况下，日志记录器都预先配置为使用控制台输出，并且还提供可选的文件输出。

默认情况下，如果您使用了 **Starter**，则使用 Logback 进行日志记录。还包括合适的 Logback 路由，以确保在使用 Java Util Logging、Commons Logging、Log4J 或 SLF4J 的依赖库都能正常工作。

**提示**

> Java 有很多日志框架可供使用。如果以上列表让您感到困惑，请不要担心。通常，您不需要更改日志依赖，并且 Spring Boot 提供的默认配置可以保证日志正常工作。

<a id="boot-features-logging-format"></a>

### 26.1、日志格式

Spring Boot 默认日志输出类似于以下示例：

```
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出以下项：

- 日期和时间：毫秒精度，易于排序。
- 日志级别：`ERROR`、`WARN`、`INFO`、`DEBUG` 或 `TRACE`。
- 进程 ID。
- 一个 `---` 分隔符，用于区分实际日志内容的开始。
- 线程名称：在方括号中（可能会截断控制台输出）。
- 日志记录器名称：这通常是源类名称（通常为缩写）。
- 日志内容。

**注意**

> Logback 没有 `FATAL` 级别。该级别映射到 `ERROR`。

<a id="boot-features-logging-console-output"></a>

### 26.2、控制台输出

默认日志配置会在写入时将消息回显到控制台。默认情况下，会记录 `ERROR`、`WARN` 和 `INFO` 级别的日志。您还可以通过使用 `--debug` 标志启动应用程序来启用**调试**模式。

```bash
$ java -jar myapp.jar --debug
```

**注意**

> 您还可以在 `application.properties` 中指定 `debug=true`。

启用调试模式后，核心日志记录器（内嵌容器、Hibernate 和 Spring Boot）将被配置为输出更多日志信息。启用调试模式不会将应用程序配置为使用 `DEBUG` 级别记录所有日志内容。

或者，您可以通过使用 `--trace` 标志（或在 `application.properties` 中的设置 `trace=true`）启动应用程序来启用**跟踪**模式。这样做可以为选择的核心日志记录器（内嵌容器、Hibernate 模式生成和整个 Spring 组合）启用日志追踪。

<a id="boot-features-logging-color-coded-output"></a>

### 26.2.1、着色输出

如果您的终端支持 ANSI，则可以使用颜色输出来提高可读性。您可以将 `spring.output.ansi.enabled` 设置为[受支持的值](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)以覆盖自动检测。

可使用 `％clr` 转换字配置颜色编码。最简单形式是，转换器根据日志级别对输出进行着色，如下所示：

```
%clr(%5p)
```

下表描述日志级别与颜色的映射关系：

| 级别 | 颜色 |
| --- | --- |
| `FATAL` | 红（Red） |
| `ERROR` | 红（Red） |
| `WARN` | 黄（Yellow） |
| `INFO` | 绿（Green） |
| `DEBUG` | 绿（Green） |
| `TRACE` | 绿（Green） |

或者，您可以通过将其作为转换选项指定应使用的颜色或样式。例如，要将文本变为黄色，请使用以下设置：

```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式：

- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`

<a id="boot-features-logging-file-output"></a>

### 26.3、文件输出

默认情况下，Spring Boot 仅记录到控制台，不会写入日志文件。想除了控制台输出之外还要写入日志文件，则需要设置 `logging.file` 或 `logging.path` 属性（例如，在 `application.properties` 中）。

下表展示了如何与 `logging.*` 属性一起使用：

**表 26.1、Logging 属性**

| `logging.file` | `logging.path` | 示例 | 描述 |
| --- | --- | --- | --- |
| （无） | （无） | | 仅在控制台输出 |
| 指定文件 | (无) | `my.log` | 写入指定的日志文件。名称可以是绝对位置或相对于当前目录。 |
| （无） | 指定目录 | `/var/log` | 将 `spring.log` 写入指定的目录。名称可以是绝对位置或相对于当前目录。 |

日志文件在达到 10MB 时会轮转，并且与控制台输出一样，默认情况下会记录 `ERROR`、`WARN` 和 `INFO` 级别的内容。可以使用 `logging.file.max-size` 属性更改大小限制。除非已设置 `logging.file.max-history` 属性，否则以前轮转的文件将无限期归档。

**注意**

> 日志记录系统在应用程序生命周期的早期开始初始化。因此，通过 `@PropertySource` 注解加载的属性文件中是找不到日志属性的。

**提示**

> 日志属性独立于实际的日志底层。因此，spring Boot 不管理特定的配置 key（例如 Logback 的 `logback.configurationFile`）。


<a id="boot-features-custom-log-levels"></a>

### 26.4、日志等级

所有受支持的日志记录系统都可以使用 `logging.level.<logger-name>=<level>` 来设置 Spring `Environment` 中的记录器等级（例如，在 `application.properties` 中）。其中 `level` 是 TRACE、DEBUG、INFO、WARN、ERROR、FATAL 和 OFF 其中之一。可以使用 `logging.level.root` 配置 `root` 记录器。

以下示例展示了 `application.properties` 中默认的日志记录设置：

```ini
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

<a id="boot-features-custom-log-groups"></a>

### 26.5、日志组

将相关记录器组合在一起以便可以同时配置，这通常很有用。例如，您可以更改**所有** Tomcat 相关记录器的日志记录级别，但您无法轻松记住顶层的包名。

为了解决这个问题，Spring Boot 允许您在 Spring `Environment` 中定义日志记录组。例如，以下通过将 **tomcat** 组添加到 `application.properties` 来定义 **tomcat** 组：

```ini
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

定义后，您可以使用一行配置来更改组中所有记录器的级别：

```ini
logging.level.tomcat=TRACE
```

Spring Boot 包含以下预定义的日志记录组，可以直接使用：

| 名称 | 日志记录器 |
| --- | --- |
| web | `org.springframework.core.codec`、`org.springframework.http`、`org.springframework.web` |
| sql | `org.springframework.jdbc.core`、`org.hibernate.SQL` |


<a id="boot-features-custom-log-configuration"></a>

### 26.6、自定义日志配置

可以通过在 classpath 中引入适合的库来激活各种日志记录系统，并且可以通过在 classpath 的根目录中或在以下 Spring `Environment` 属性指定的位置提供合适的配置文件来进一步自定义：`logging.config`。

您可以使用 `org.springframework.boot.logging.LoggingSystem` 系统属性强制 Spring Boot 使用特定的日志记录系统。该值应该是一个实现了 `LoggingSystem` 的类的完全限定类名。您还可以使用 `none` 值完全禁用 Spring Boot 的日志记录配置。

**注意**

> 由于日志记录在创建 `ApplicationContext` 之前初始化，因此无法在 Spring `@Configuration` 文件中控制来自 `@PropertySources` 的日志记录。更改日志记录系统或完全禁用它的唯一方法是通过系统属性设置。

根据您的日志记录系统，将加载以下文件：

| 日志记录系统 | 文件 |
| --- | --- |
| Logback | `logback-spring.xml`、`logback-spring.groovy`、`logback.xml` 或者 `logback.groovy` |
| Log4j2 | `log4j2-spring.xml` 或者 `log4j2.xml` |
| JDK（Java Util Logging） | `logging.properties` |

**注意**

> 如果可能，我们建议您使用 `-spring` 的形式来配置日志记录（比如 `logback-spring.xml` 而不是 `logback.xml`）。如果使用标准的配置位置，Spring 无法完全控制日志初始化。

**警告**

> Java Util Logging 存在已知的类加载问题，这些问题在以**可执行 jar** 运行时会触发。如果可能的话，我们建议您在使用**可执行 jar** 方式运行时避免使用它。

为了进行自定义，部分其他属性会从 Spring `Environment` 传输到 System 属性，如下表所述：

| Spring Environment | 系统属性 | 说明 |
| --- | --- | --- |
| `logging.exception-conversion-word` | `LOG_EXCEPTION_CONVERSION_WORD` | 记录异常时使用的转换字。 |
| `logging.file` | `LOG_FILE` | 如果已定义，则在默认日志配置中使用它。 |
| `logging.file.max-size` | `LOG_FILE_MAX_SIZE` | 最大日志文件大小（如果启用了 LOG_FILE）。（仅支持默认的 Logback 设置。） |
|`logging.file.max-history` | `LOG_FILE_MAX_HISTORY` | 要保留的归档日志文件最大数量（如果启用了 LOG_FILE）。（仅支持默认的 Logback 设置。） |
| `logging.path` | `LOG_PATH` | 如果已定义，则在默认日志配置中使用它。 |
| `logging.pattern.console` | `CONSOLE_LOG_PATTERN` | 要在控制台上使用的日志模式（stdout）。（仅支持默认的 Logback 设置。） |
| `logging.pattern.dateformat` | `LOG_DATEFORMAT_PATTERN` | 日志日期格式的 Appender 模式。（仅支持默认的 Logback 设置。） |
| `logging.pattern.file` | `FILE_LOG_PATTERN` | 要在文件中使用的日志模式（如果启用了 `LOG_FILE`）。（仅支持默认的 Logback 设置。） |
| `logging.pattern.level` | `LOG_LEVEL_PATTERN` | 渲染日志级别时使用的格式（默认值为 `％5p`）。（仅支持默认的 Logback 设置。） |
| `PID` | `PID` | 当前进程 ID（如果可能，则在未定义为 OS 环境变量时发现）。 | 

所有受支持的日志记录系统在解析其配置文件时都可以参考系统属性。有关示例，请参阅 `spring-boot.jar` 中的默认配置：

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

**提示**

> 如果要在日志记录属性中使用占位符，则应使用 [Spring Boot 的语法](#boot-features-external-config-placeholders-in-properties)，而不是使用底层框架的语法。值得注意的是，如果使用 Logback，则应使用 `:` 作为属性名称与其默认值之间的分隔符，而不是使用 `:-`。

**提示**

<blockquote>

您可以通过仅覆盖 `LOG_LEVEL_PATTERN`（或带 Logback 的 `logging.pattern.level`）将 MDC 和其他特别的内容添加到日志行。例如，如果使用 `logging.pattern.level=user:%X{user} %5p`，则默认日志格式包含 **user** MDC 项（如果存在），如下所示:

```
2015-09-30 12:30:04.031 user:someone INFO 22174 --- [  nio-8080-exec-0] demo.Controller
Handling authenticated request
```

</blockquote>

<a id="boot-features-logback-extensions"></a>

### 26.7、Logback 扩展

Spring Boot 包含许多 Logback 扩展，可用于进行高级配置。您可以在 `logback-spring.xml` 配置文件中使用这些扩展。

**注意**

> 由于标准的 logback.xml 配置文件加载过早，因此无法在其中使用扩展。您需要使用 `logback-spring.xml` 或定义 `logging.config` 属性。

**警告**

> 扩展不能与 Logback 的[配置扫描](http://logback.qos.ch/manual/configuration.html#autoScan)一起使用。如果尝试这样做，更改配置文件会导致发生类似以下错误日志：

```
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```
 
<a id="_profile_specific_configuration"></a>

### 26.7.1、特定 Profile 配置

`<springProfile>` 标签允许您根据激活的 Spring profile 选择性地包含或排除配置部分。在 `<configuration>` 元素中的任何位置都支持配置 profile。使用 `name` 属性指定哪个 proifle 接受配置。`<springProfile>` 标记可以包含简单的 proifle 名称（例如 `staging`）或 profile 表达式。profile 表达式允许表达更复杂的 profile 逻辑，例如 `production & (eu-central | eu-west)`。有关详细信息，请查阅[参考指南](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。以下清单展示了三个示例 profile：

```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

<a id="_environment_properties"></a>

### 26.7.2、环境属性

使用 `<springProperty>` 标记可以让您暴露 Spring 环境（`Environment`）中的属性，以便在 Logback 中使用。如果在 Logback 配置中访问来自 `application.properties` 文件的值，这样做很有用。标签的工作方式与 Logback 的标准 `<property>` 标签类似。但是，您可以指定属性（来自 `Environment`）的 `source`，而不是指定直接的 `value`。如果需要将属性存储在 `local` 范围以外的其他位置，则可以使用 `scope` 属性。如果需要回退值（如果未在 `Environment` 中设置该属性），则可以使用 `defaultValue` 属性。以下示例展示了如何暴露属性以便在 Logback 中使用：

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
		defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```

**注意**

> 必须以 kebab 风格（短横线小写风格）指定 `source`（例如 `my.property-name`）。但可以使用宽松规则将属性添加到 `Environment` 中。

<a id="boot-features-json"></a>

## 27、JSON

Spring Boot 为三个 JSON 映射库提供了内置集成：

- GSON
- Jackson
- JSON-B

Jackson 是首选和默认的库。

<a id="boot-features-json-jackson"></a>

## 27.1、Jackson

Spring Boot 提供了 Jackson 的自动配置，Jackson 是 `spring-boot-starter-json` 的一部分。当 Jackson 在 classpath 上时，会自动配置 `ObjectMapper` bean。Spring Boot 提供了几个配置属性来[自定义 `ObjectMapper` 的配置](howto.md#howto-customize-the-jackson-objectmapper)。

<a id="boot-features-json-gson"></a>

## 27.2、Gson

Spring Boot 提供 Gson 的自动配置。当 Gson 在 classpath 上时，会自动配置 `Gson` bean。Spring Boot 提供了几个 `spring.gson.*` 配置属性来自定义配置。为了获得更多控制，可以使用一个或多个 `GsonBuilderCustomizer` bean。

<a id="boot-features-json-json-b"></a>

## 27.3、JSON-B

Spring Boot 提供了 JSON-B 的自动配置。当 JSON-B API 和实现在 classpath 上时，将自动配置 `Jsonb` bean。首选的 JSON-B 实现是 Apache Johnzon，它提供了依赖管理。

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

> Jersey 对于扫描可执行归档文件的支持是相当有限的。例如，它无法扫描一个[完整的可执行 jar 文件](#deployment-install)中的端点，同样，当运行一个可执行的 war 文件时，它也无法扫描包中 `WEB-INF/classes` 下的端点。为了避免该限制，您不应该使用 `packages` 方法，应该使用上述的 `register` 方法来单独注册每一个端点。

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

#### 28.4.3、ServletWebServerApplicationContext

Spring Boot 底层使用了一个不同的 `ApplicationContext` 类型来支持内嵌 servlet。`ServletWebServerApplicationContext` 是一个特殊 `WebApplicationContext` 类型，它通过搜索单个 `ServletWebServerFactory` bean 来引导自身。通常，`TomcatServletWebServerFactory`、 `JettyServletWebServerFactory` 或者 `UndertowServletWebServerFactory` 中的一个将被自动配置。

**注意**

> 通常，你不需要知道这些实现类。大部分应用程序会自动配置，并为您创建合适的 `ApplicationContext` 和 `ServletWebServerFactory`。

<a id="boot-features-customizing-embedded-containers"></a>

##### 28.4.4、自定义内嵌 Servlet 容器

可以使用 Spring `Environment` 属性来配置通用的 servlet 容器设置。通常，您可以在 `application.properties` 文件中定义这些属性。

通用的服务器设置包括：

- 网络设置：监听 HTTP 请求的端口（`server.port`），绑定接口地址到 `server.address` 等。
- 会话设置：是否持久会话（`server.session.persistence`）、session 超时（`server.session.timeout`）、会话数据存放位置（`server.session.store-dir`）和 session-cookie 配置（`server.session.cookie.*`）。
- 错误管理：错误页面位置（`server.error.path`）等。
- [SSL](#howto-configure-ssl)
- [HTTP 压缩](#how-to-enable-http-response-compression)

Spring Boot 尽可能暴露通用的设置，但并不总是都可以。针对这些情况，专用的命名空间为特定的服务器提供了自定义功能（请参阅 `server.tomcat` 和 `server.undertow`）。例如，您可以使用内嵌 servlet 容器的特定功能来配置[访问日志](#howto-configure-accesslogs)。

**提示**

> 有关完整的内容列表，请参阅 [ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java) 类。

<a id="boot-features-programmatic-embedded-container-customization"></a>

###### 28.4.4.1、以编程方式自定义

如果您需要以编程的方式配置内嵌 servlet 容器，可以注册一个是实现了 `WebServerFactoryCustomizer` 接口的 Spring bean。`WebServerFactoryCustomizer` 提供了对 `ConfigurableServletWebServerFactory` 的访问入口，其中包含了许多自定义 setter 方法。以下示例使用了编程方式来设置端口：

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

	@Override
	public void customize(ConfigurableServletWebServerFactory server) {
		server.setPort(9000);
	}

}
```

**注意**

> `TomcatServletWebServerFactory`、`JettyServletWebServerFactory` 和 `UndertowServletWebServerFactory` 是 ConfigurableServletWebServerFactory 的具体子类，它们分别为 Tomcat、Jetty 和 Undertow 提供了额外的自定义 setter 方法。


<a id="boot-features-customizing-configurableservletwebserverfactory-directly"></a>

###### 28.4.4.2、直接自定义 ConfigurableServletWebServerFactory

如果上述的自定义方式太局限，您可以自己注册 `TomcatServletWebServerFactory`、`JettyServletWebServerFactory` 或 `UndertowServletWebServerFactory` bean。

```java
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
	TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
	factory.setPort(9000);
	factory.setSessionTimeout(10, TimeUnit.MINUTES);
	factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
	return factory;
}
```

Setter 方法提供了许多配置选项。还有几个 **hook** 保护方法供您深入定制。有关详细信息，请参阅[源码文档](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html)。

<a id="boot-features-jsp-limitations"></a>

#### 28.4.5、JSP 局限

当运行使用了内嵌 servlet 容器的 Spring Boot 应用程序时（打包为可执行归档文件），JSP 支持将存在一些限制。

- 如果您使用 war 打包，在 Jetty 和 Tomcat 中可以正常工作，使用 `java -jar` 启动时，可执行的 war 可正常使用，并且还可以部署到任何标准容器。使用可执行 jar 时不支持 JSP。
- Undertow 不支持 JSP。
- 创建自定义的 error.jsp 页面不会覆盖默认[错误处理](#boot-features-error-handling)视图，应该使用[自定义错误页面](#boot-features-error-handling-custom-error-pages)来代替。

这里有一个 [JSP 示例](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-samples/spring-boot-sample-web-jsp)，您可以了解到如何配置。

<a id="boot-features-reactive-server"></a>

### 28.5、内嵌响应式服务器支持

Spring Boot 包括对以下内嵌响应式 Web 服务器的支持：Reactor Netty、Tomcat、Jetty 和 Undertow。大多数开发人员使用对应的 **Starter** 来获取一个完全配置的实例。默认情况下，内嵌服务器在 8080 端口上监听 HTTP 请求。

<a id="boot-features-reactive-server-resources"></a>

### 28.6、响应式服务器资源配置

在自动配置 Reactor Netty 或 Jetty 服务器时，Spring Boot 将创建特定的 bean 为服务器实例提供 HTTP 资源：`ReactorResourceFactory` 或 `JettyResourceFactory`。

默认情况下，这些资源也将与 Reactor Netty 和 Jetty 客户端共享以获得最佳性能，具体如下：

- 用于服务器和客户端的的相同技术
- 客户端实例使用了 Spring Boot 自动配置的 `WebClient.Builder` bean 构建。

开发人员可以通过提供自定义的 `ReactorResourceFactory` 或 `JettyResourceFactory` bean 来重写 Jetty 和 Reactor Netty 的资源配置 —— 将应用于客户端和服务器。

您可以在 [WebClient Runtime](#boot-features-webclient-runtime) 章节中了解有关客户端资源配置的更多内容。

<a id="boot-features-security"></a>

## 29、安全

默认情况下，如果 [Spring Security](https://projects.spring.io/spring-security/) 在 classpath 上，则 Web 应用程序是受保护的。Spring Boot 依赖 Spring Security 的内容协商策略来确定是使用 `httpBasic` 还是 `formLogin`。要给 Web 应用程序添加方法级别的安全保护，可以使用 `@EnableGlobalMethodSecurity` 注解设置。有关更多其他信息，您可以在 [Spring Security 参考指南](https://docs.spring.io/spring-security/site/docs/5.1.2.RELEASE/reference/htmlsingle#jc-method)中找到。

默认的 `UserDetailsS​​ervice` 只有一个用户。用户名为 `user`，密码是随机的，在应用程序启动时会以 INFO 级别打印出来，如下所示：

```
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

**注意**

> 如果您对日志配置进行微调，请确保将 `org.springframework.boot.autoconfigure.security` 的级别设置为 `INFO`。否则，默认密码不会打印出来。

您可以通过提供 `spring.security.user.name` 和 `spring.security.user.password` 来更改用户名和密码。

您在 Web 应用程序中默认会获得以下基本功能：

- 一个 `UserDetailsS​​ervice`（或 WebFlux 应用程序中的 `ReactiveUserDetailsS​​ervice`）bean，采用内存存储形式，有一个自动生成密码的用户（有关用户属性，请参阅 [`SecurityProperties.User`](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/autoconfigure/security/SecurityProperties.User.html)）。
- 用于整个应用程序（如果 actuator 在 classpath 上，则包括 actuator 端点）基于表单登录或 HTTP Basic 认证（取决于 Content-Type）。
- 一个用于发布身份验证事件的 `DefaultAuthenticationEventPublisher`。

您可以通过为其添加一个 bean 来提供不同的 `AuthenticationEventPublisher`。

<a id="boot-features-security-mvc"></a>

### 29.1、MVC 安全

默认的安全配置在 `SecurityAutoConfiguration` 和 `UserDetailsS​​erviceAutoConfiguration` 中实现。 `SecurityAutoConfiguration` 导入用于 Web 安全的 `SpringBootWebSecurityConfiguration`，`UserDetailsS​​erviceAutoConfiguration` 配置身份验证，这同样适用于非 Web 应用程序。要完全关闭默认的 Web 应用程序安全配置，可以添加 `WebSecurityConfigurerAdapter` 类型的 bean（这样做不会禁用 `UserDetailsS​​ervice` 配置或 Actuator 的安全保护）。

要同时关闭 `UserDetailsS​​ervice` 配置，您可以添加 `UserDetailsS​​ervice`、`AuthenticationProvider` 或 `AuthenticationManager` 类型的 bean。[Spring Boot 示例](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-samples/)中有几个使用了安全保护的应用程序，他们或许可以帮助到您。

可以通过添加自定义 `WebSecurityConfigurerAdapter` 来重写访问规则。Spring Boot 提供了便捷方法，可用于重写 actuator 端点和静态资源的访问规则。`EndpointRequest` 可用于创建一个基于 `management.endpoints.web.base-path` 属性的 `RequestMatcher`。`PathRequest` 可用于为常用位置中的资源创建一个 `RequestMatcher`。

<a id="boot-features-security-webflux"></a>

### 29.2、WebFlux 安全

与 Spring MVC 应用程序类似，您可以通过添加 `spring-boot-starter-security` 依赖来保护 WebFlux 应用程序。默认的安全配置在 `ReactiveSecurityAutoConfiguration` 和 `UserDetailsServiceAutoConfiguration` 中实现。`ReactiveSecurityAutoConfiguration` 导入用于 Web 安全的 `WebFluxSecurityConfiguration`，`UserDetailsServiceAutoConfiguration` 配置身份验证，这同样适用于非 Web 应用程序。要完全关闭默认的 Web 应用程序安全配置，可以添加 `WebFilterChainProxy` 类型的 bean（这样做不会禁用 `UserDetailsS​​ervice` 配置或 Actuator 的安全保护）。

要同时关闭 `UserDetailsS​​ervice` 配置，您可以添加 `ReactiveUserDetailsService` 或 `ReactiveAuthenticationManager` 类型的 bean。

可以通过添加自定义 `SecurityWebFilterChain` 来重写访问规则。Spring Boot 提供了便捷方法，可用于重写 actuator 端点和静态资源的访问规则。`EndpointRequest` 可用于创建一个基于 `management.endpoints.web.base-path` 属性的 `ServerWebExchangeMatcher`。

`PathRequest` 可用于为常用位置中的资源创建一个 `ServerWebExchangeMatcher`。

例如，您可以通过添加以下内容来自定义安全配置：

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
	return http
		.authorizeExchange()
			.matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
			.pathMatchers("/foo", "/bar")
				.authenticated().and()
			.formLogin().and()
		.build();
}
```

<a id="boot-features-security-oauth2"></a>

### 29.3、OAuth2

[OAuth2](https://oauth.net/2/) 是 Spring 支持的一种广泛使用的授权框架。

<a id="boot-features-security-oauth2-client"></a>

#### 29.3.1、客户端

如果您的 classpath 上有 `spring-security-oauth2-client`，则可以利用一些自动配置来轻松设置 `OAuth2/Open ID Connect 客户端。该配置使用 `OAuth2ClientProperties` 的属性。相同的属性适用于 servlet 和响应式应用程序。

您可以在 `spring.security.oauth2.client` 前缀下注册多个 OAuth2 客户端和提供者（provider），如下所示：

```ini
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=http://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=http://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=http://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=http://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```

对于支持 [OpenID Connect 发现](https://openid.net/specs/openid-connect-discovery-1_0.html)的 OpenID Connect 提供者，可以进一步简化配置。需要使用 `issuer-uri` 配置提供者，`issuer-uri` 是其 Issuer Identifier 的 URI。例如，如果提供的 `issuer-uri` 是 “https://example.com”，则将对 “https://example.com/.well-known/openid-configuration” 发起一个 `OpenID Provider Configuration Request`。期望结果是一个 `OpenID Provider Configuration Response`。以下示例展示了如何使用 `issuer-uri` 配置一个 OpenID Connect Provider：

```
spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

默认情况下，Spring Security 的 `OAuth2LoginAuthenticationFilter` 仅处理与 `/login/oauth2/code/*` 相匹配的 URL。如果要自定义 `redirect-uri` 以使用其他匹配模式，则需要提供配置以处理该自定义模式。例如，对于 servlet 应用程序，您可以添加类似于以下 `WebSecurityConfigurerAdapter`：

```java
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.oauth2Login()
				.redirectionEndpoint()
					.baseUri("/custom-callback");
	}
}
```

<a id="boot-features-security-oauth2-common-providers"></a>

**OAuth2 客户端注册常见的提供者**

对于常见的 OAuth2 和 OpenID 提供者（provider），包括 Google、Github、Facebook 和 Okta，我们提供了一组提供者默认设置（分别是 `google`、`github`、`facebook` 和 `okta`）。

如果您不需要自定义这些提供者，则可以将 `provider` 属性设置为您需要推断默认值的属性。此外，如果客户端注册的 key 与默认支持的提供者匹配，则 Spring Boot 也会推断出来。

换而言之，以下示例中的两个配置使用了 Google 提供者：

```ini
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```

<a id="boot-features-security-oauth2-server"></a>

#### 29.3.2、资源服务器

如果在 classpath 上有 `spring-security-oauth2-resource-server`，只要指定了 JWK Set URI 或 OIDC Issuer URI，Spring Boot 就可以设置 OAuth2 资源服务器，如下所示：

```ini
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys
```

```ini
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

相同的属性适用于 servlet 和响应式应用程序。

或者，您可以为 servlet 应用程序定义自己的 `JwtDecoder` bean，或为响应式应用程序定义 `ReactiveJwtDecoder`。

<a id="_authorization_server"></a>

#### 29.3.3、授权服务器

目前，Spring Security 没有提供 OAuth 2.0 授权服务器实现。但此功能可从 [Spring Security OAuth](https://projects.spring.io/spring-security-oauth/) 项目获得，该项目最终会被 Spring Security 所取代。在此之前，您可以使用 `spring-security-oauth2-autoconfigure` 模块轻松设置 OAuth 2.0 授权服务器，请参阅其[文档](https://docs.spring.io/spring-security-oauth2-boot)以获取详细信息。

<a id="boot-features-security-actuator"></a>

### 29.4、Actuator 安全

出于安全考虑，默认情况下禁用除 `/health` 和 `/info` 之外的所有 actuator。可用 `management.endpoints.web.exposure.include` 属性启用 actuator。

如果 Spring Security 位于 classpath 上且没有其他 `WebSecurityConfigurerAdapter`，则除了 `/health` 和 `/info` 之外的所有 actuator 都由 Spring Boot 自动配置保护。如果您定义了自定义 `WebSecurityConfigurerAdapter`，则 Spring Boot 自动配置将不再生效，您可以完全控制 actuator 的访问规则。

**注意**

> 在设置 `management.endpoints.web.exposure.include` 之前，请确保暴露的 actuator 没有包含敏感信息和 `/` 或被防火墙保护亦或受 Spring Security 之类的保护。

<a id="boot-features-security-csrf"></a>

#### 29.4.1、跨站请求伪造保护

由于 Spring Boot 依赖 Spring Security 的默认值配置，因此默认情况下会启用 CSRF 保护。这意味着当使用默认安全配置时，需要 `POST`（shutdown 和 loggers 端点）、`PUT` 或 `DELETE` 的 actuator 端点将返回 403 禁止访问错误。

**注意**

> 我们建议仅在创建非浏览器客户端使用的服务时才完全禁用 CSRF 保护。

有关 CSRF 保护的其他信息，请参阅 [Spring Security 参考指南](https://docs.spring.io/spring-security/site/docs/5.1.2.RELEASE/reference/htmlsingle#csrf)。

<a id="boot-features-sql"></a>

## 30、使用 SQL 数据库

[Spring Framework](https://projects.spring.io/spring-framework/) 为 SQL 数据库提供了广泛的支持。从直接使用 `JdbcTemplate` 进行 JDBC 访问到完全的**对象关系映射**（object relational mapping）技术，比如 Hibernate。[Spring Data](https://projects.spring.io/spring-data/) 提供了更多级别的功能，直接从接口创建的 `Repository` 实现，并使用了约定从方法名生成查询。

<a id="boot-features-configure-datasource"></a>

### 30.1、配置数据源

Java 的 `javax.sql.DataSource` 接口提供了一个使用数据库连接的标准方法。通常，数据源使用 `URL` 和一些凭据信息来建立数据库连接。

**提示**

> 查看 [How-to](#howto-configure-a-datasource) 部分获取更多高级示例，通常您可以完全控制数据库的配置。

<a id="boot-features-embedded-database-support"></a>

#### 30.1.1、内嵌数据库支持

使用内嵌内存数据库来开发应用程序非常方便的。显然，内存数据库不提供持久存储。在应用启动时，您需要填充数据库，并在应用程序结束时丢弃数据。

**提示**

> **How-to** 部分包含了[如何初始化数据库](#howto-database-initialization)方面的内容。

Spring Boot 可以自动配置内嵌 [H2](http://www.h2database.com/)、[HSQL](http://hsqldb.org/) 和 [Derby](https://db.apache.org/derby/) 数据库。您不需要提供任何连接 URL，只需为您想要使用的内嵌数据库引入特定的构建依赖。

**注意**

> 如果您在测试中使用此功能，您可能会注意到，无论使用了多少应用程序上下文，整个测试套件都会重复使用相同的数据库。如果您想确保每个上下文都有一个单独的内嵌数据库，则应该将 `spring.datasource.generate-unique-name` 设置为 `true`。

以下是 POM 依赖示例：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.hsqldb</groupId>
	<artifactId>hsqldb</artifactId>
	<scope>runtime</scope>
</dependency>
```

**注意**

> 要自动配置内嵌数据库，您需要一个 `spring-jdbc` 依赖。在这个例子中，它是通过 `spring-boot-starter-data-jpa` 引入。

**提示**

> 如果出于某些原因，您需要配置内嵌数据库的连接 URL，则应注意确保禁用数据库的自动关闭功能。如果您使用 H2，则应该使用 `DB_CLOSE_ON_EXIT=FALSE` 来设置。如果您使用 `HSQLDB`，则确保不使用 `shutdown=true`。禁用数据库的自动关闭功能允许 Spring Boot 控制数据库何时关闭，从而确保一旦不再需要访问数据库时就触发。

<a id="boot-features-connect-to-production-database"></a>

#### 30.1.2、连接生产数据库

生产数据库连接也可以使用使用 `DataSource` 自动配置。Spring Boot 使用以下算法来选择一个特定的实现：

- 出于性能和并发性的考虑，我们更喜欢 [HikariCP](https://github.com/brettwooldridge/HikariCP) 连接池。如果 HikariCP 可用，我们总是选择它。
- 否则，如果 Tomcat 池 `DataSource` 可用，我们将使用它。
- 如果 HikariCP 和 Tomcat 池数据源不可用，但 [Commons DBCP](https://commons.apache.org/proper/commons-dbcp/) 可用，我们将使用它。

如果您使用了 `spring-boot-starter-jdbc` 或者 `spring-boot-starter-data-jpa` starter，您将自动得到 `HikariCP` 依赖。

**注意**

> 您完全可以绕过该算法，并通过 `spring.datasource.type` 属性指定要使用的连接池。如果您在 Tomcat 容器中运行应用程序，默认提供 `tomcat-jdbc`，这点尤其重要。

**提示**

> 可以手动配置其他连接池。如果您定义了自己的 `DataSource` bean，则自动配置将不会触发。

数据源配置由 `spring.datasource.*` 中的外部属性所控制。例如，您可以在 `application.properties` 中声明以下部分：

```ini
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

**注意**

> 您至少应该使用 `spring.datasource.url` 属性来指定 URL，否则 Spring Boot 将尝试自动配置内嵌数据库。

**提示**

> 通常您不需要指定 `driver-class-name`，因为 Spring boot 可以从 `url` 推导出大多数数据库。

**注意**

> 对于要创建的池 `DataSource`，我们需要能够验证有效的 `Driver` 类是否可用，因此我们在使用之前进行检查。例如，如果您设置了 `spring.datasource.driver-class-name=com.mysql.jdbc.Driver`，那么该类必须可加载。

有关更多支持选项，请参阅 [DataSourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)。这些都是标准选项，与实际的实现无关。还可以使用各自的前缀（`spring.datasource.hikari.*`、`spring.datasource.tomcat.*` 和 `spring.datasource.dbcp2.*`）微调实现特定的设置。请参考您现在使用的连接池实现的文档来获取更多信息。

例如，如果你使用 [Tomcat 连接池](https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html#Common_Attributes)，则可以自定义许多其他设置，如下：

```ini
# Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

# Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

# Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```

<a id="boot-features-connecting-to-a-jndi-datasource"></a>

#### 30.1.3、连接 JNDI 数据源

如果要将 Spring Boot 应用程序部署到应用服务器（Application Server）上，您可能想使用应用服务器的内置功能和 JNDI 访问方式来配置和管理数据源。

`spring.datasource.jndi-name` 属性可作为 `spring.datasource.url`、`spring.datasource.username` 和 `spring.datasource.password` 属性的替代方法，用于从特定的 JNDI 位置访问 `DataSource`。例如，`application.properties` 中的以下部分展示了如何访问 JBoss AS 定义的 `DataSource`：

```ini
spring.datasource.jndi-name=java:jboss/datasources/customers
```

<a id="boot-features-using-jdbc-template"></a>

### 30.2、使用 JdbcTemplate

Spring 的 `JdbcTemplate` 和 `NamedParameterJdbcTemplate` 类是自动配置的，您可以使用 `@Autowire` 将它们直接注入您的 bean 中：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final JdbcTemplate jdbcTemplate;

	@Autowired
	public MyBean(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}

	// ...

}
```

您可以使用 `spring.jdbc.template.*` 属性来自定义一些 template 的属性，如下：

```ini
spring.jdbc.template.max-rows=500
```

**注意**

> `NamedParameterJdbcTemplate` 在底层重用了相同的 `JdbcTemplate` 实例。如果定义了多个 `JdbcTemplate` 且没有声明 primary 主候选，则不会自动配置 `NamedParameterJdbcTemplate`。

<a id="boot-features-jpa-and-spring-data"></a>

### 30.3、JPA 与 Spring Data JPA

Java Persistence API（Java 持久化 API）是一项标准技术，可让您将对象**映射**到关系数据库。`spring-boot-starter-data-jpa` POM 提供了一个快速起步的方法。它提供了以下关键依赖：

- **Hibernate**  ——  最受欢迎的 JPA 实现之一。
- **Spring Data JPA** ——  可以轻松地实现基于 JPA 的资源库。
- **Spring ORM**  ——  Spring Framework 的核心 ORM 支持

**提示**

> 我们不会在这里介绍太多关于 JPA 或者 [Spring Data](https://projects.spring.io/spring-data/) 的相关内容。您可以在 [spring.io](https://spring.io/) 上查看[使用 JPA 访问数据](https://spring.io/guides/gs/accessing-data-jpa/)，获取阅读 [Spring Data JPA](https://projects.spring.io/spring-data-jpa/) 和 [Hibernate](https://hibernate.org/orm/documentation/) 的参考文档。

<a id="boot-features-jpa-and-spring-data"></a>

#### 30.3.1、实体类

通常，JPA **Entity**（实体）类是在 `persistence.xml` 文件中指定的。使用了 Spring Boot，该文件将不是必需的，可以使用 **Entity Scanning**（实体扫描）来代替。默认情况下，将搜索主配置类（使用了 `@EnableAutoConfiguration` 或 `@SpringBootApplication` 注解）下面的所有包。

任何用了 `@Entity`、`@Embeddable` 或者 `@MappedSuperclass` 注解的类将被考虑。一个典型的实体类如下：

```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

	@Id
	@GeneratedValue
	private Long id;

	@Column(nullable = false)
	private String name;

	@Column(nullable = false)
	private String state;

	// ... additional members, often include @OneToMany mappings

	protected City() {
		// no-args constructor required by JPA spec
		// this one is protected since it shouldn't be used directly
	}

	public City(String name, String state) {
		this.name = name;
		this.state = state;
	}

	public String getName() {
		return this.name;
	}

	public String getState() {
		return this.state;
	}

	// ... etc

}
```

**提示**

> 您可以使用 `@EntityScan` 注解自定义实体类的扫描位置。请参见[84.4、从 Spring configuration 配置中分离 @Entity 定义](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#howto-separate-entity-definitions-from-spring-configuration)章节。

<a id="boot-features-spring-data-jpa-repositories"></a>

#### 30.3.2、Spring Data JPA 资源库

[Spring Data JPA](https://projects.spring.io/spring-data-jpa/) 资源库（repository）是接口，您可以定义用于访问数据。JAP 查询是根据您的方法名自动创建。例如，`CityRepository` 接口可以声明 `findAllByState(String state)` 方法来查找指定状态下的所有城市。

对于更加复杂的查询，您可以使用 Spring Data 的 [`Query`](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html) 注解

Spring Data 资源库通常继承自 [Repository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html) 或者 [CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) 接口。如果您使用了自动配置，则将从包含主配置类（使用了 `@EnableAutoConfiguration` 或 `@SpringBootApplication` 注解）的包中搜索资源库：

以下是一个典型的 Spring Data 资源库接口定义：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

	Page<City> findAll(Pageable pageable);

	City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

Spring Data JPA 资源库支持三种不同的引导模式：default、deferred 和 lazy。要启用延迟或懒惰引导，请将 `spring.data.jpa.repositories.bootstrap-mode` 分别设置为 `deferred` 或 `lazy`。使用延迟或延迟引导时，自动配置的 `EntityManagerFactoryBuilder` 将使用上下文的异步任务执行器（如果有）作为引导程序执行器。

**提示**

> 我们几乎没有接触到 Spring Data JPA 的表面内容。有关详细信息，请查阅 [Spring Data JPA 参考文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)。


<a id="boot-features-creating-and-dropping-jpa-databases"></a>

#### 30.3.3、创建和删除 JPA 数据库

默认情况下，**仅**当您使用了内嵌数据库（H2、HSQL 或 Derby）时才会自动创建 JPA 数据库。您可以使用 `spring.jpa.*` 属性显式配置 JPA 设置。例如，要创建和删除表，您可以将以下内容添加到 `application.properties` 中：

```ini
spring.jpa.hibernate.ddl-auto=create-drop
```

**注意**

> 关于上述功能，Hibernate 自己的内部属性名称（如果您记住更好）为 `hibernate.hbm2ddl.auto`。您可以使用 `spring.jpa.properties.*`（在添加到实体管理器之前，该前缀将被删除）来将 Hibernate 原生属性一同设置：

```ini
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

上面示例中将 `true` 值设置给 `hibernate.globally_quoted_identifiers` 属性，该属性将传给 Hibernate 实体管理器。

默认情况下，DDL 执行（或验证）将延迟到 `ApplicationContext` 启动后。还有一个 `spring.jpa.generate-ddl` 标志，如果 Hibernate 自动配置是激活的，那么它将不会被使用，因为 `ddl-auto` 设置更细粒度。

<a id="boot-features-jpa-in-web-environment"></a>

#### 30.3.4、在视图中打开 EntityManager

如果您正在运行 web 应用程序，Spring Boot 将默认注册 [`OpenEntityManagerInViewInterceptor`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html) 用于**在视图中打开 EntityManager** 模式，即运允许在 web 视图中延迟加载。如果您不想开启这个行为，则应在 `application.properties` 中将 `spring.jpa.open-in-view` 设置为 `false`。

<a id="boot-features-data-jdbc"></a>

### 30.4、Spring Data JDBC

Spring Data 包含了对 JDBC 资源库的支持，并将自动为 `CrudRepository` 上的方法生成 SQL。对于更高级的查询，它提供了 `@Query` 注解。

当 classpath 下存在必要的依赖时，Spring Boot 将自动配置 Spring Data 的 JDBC 资源库。可以通过添加单个 `spring-boot-starter-data-jdbc` 依赖引入到项目中。如有必要，可通过在应用程序中添加 `@EnableJdbcRepositories` 注解或 `JdbcConfiguration` 子类来控制 Spring Data JDBC 的配置。

**提示**

> 有关 Spring Data JDBC 的完整详细信息，请参阅[参考文档](https://projects.spring.io/spring-data-jdbc/)。

<a id="boot-features-sql-h2-console"></a>

### 30.5、使用 H2 的 Web 控制台

[H2 数据库](http://www.h2database.com/)提供了一个[基于浏览器的控制台](http://www.h2database.com/html/quickstart.html#h2_console)，Spring Boot 可以为您自动配置。当满足以下条件时，控制台将自动配置：

- 您开发的是一个基于 servlet 的 web 应用程序
- `com.h2database:h2` 在 classpath 上
- 您使用了 [Spring Boot 的开发者工具](#using-boot-devtools)

**提示**

> 如果您不使用 Spring Boot 的开发者工具，但仍希望使用 H2 的控制台，则可以通过将 `spring.h2.console.enabled` 属性设置为 `true` 来实现。

**注意**

> H2 控制台仅用于开发期间，因此应注意确保 `spring.h2.console.enabled` 在生产环境中**没有**设置为 `true`。

<a id="boot-features-sql-h2-console-custom-path"></a>

#### 30.5.1、更改 H2 控制台的路径

默认情况下，控制台的路径为 `/h2-console`。你可以使用 `spring.h2.console.path` 属性来自定义控制台的路径。

<a id="boot-features-jooq"></a>

### 30.6、使用 jOOQ

Java 面向对象查询（[Java Object Oriented Querying，jOOQ](http://www.jooq.org/)）是一款广受欢迎的产品，出自 [Data Geekery](http://www.datageekery.com/)，它可以通过数据库生成 Java 代码，并允许您使用流式 API 来构建类型安全的 SQL 查询。商业版和开源版都可以与 Spring Boot 一起使用。

<a id="_code_generation"></a>

#### 30.6.1、代码生成

要使用 jOOQ 的类型安全查询，您需要从数据库模式生成 Java 类。您可以按照 [jOOQ 用户手册](https://www.jooq.org/doc/3.11.7/manual-single-page/#jooq-in-7-steps-step3)中的说明进行操作。如果您使用了 `jooq-codegen-maven` 插件，并且还使用了 `spring-boot-starter-parent` 父 POM，则可以安全地省略掉插件的 `<version>` 标签。您还可以使用 Spring Boot 定义的版本变量（例如 `h2.version`）来声明插件的数据库依赖。以下是一个示例：

```xml
<plugin>
	<groupId>org.jooq</groupId>
	<artifactId>jooq-codegen-maven</artifactId>
	<executions>
		...
	</executions>
	<dependencies>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>${h2.version}</version>
		</dependency>
	</dependencies>
	<configuration>
		<jdbc>
			<driver>org.h2.Driver</driver>
			<url>jdbc:h2:~/yourdatabase</url>
		</jdbc>
		<generator>
			...
		</generator>
	</configuration>
</plugin>
```

<a id="_using_dslcontext"></a>

#### 30.6.2、使用 DSLContext

jOOQ 提供的流式 API 是通过 `org.jooq.DSLContext` 接口初始化的。Spring Boot 将自动配置一个 `DSLContext` 作为 Spring Bean，并且将其连接到应用程序的 `DataSource`。`要使用 DSLContext`，您只需要 `@Autowire` 它：

```java
@Component
public class JooqExample implements CommandLineRunner {

	private final DSLContext create;

	@Autowired
	public JooqExample(DSLContext dslContext) {
		this.create = dslContext;
	}

}
```

**提示**

> jOOQ 手册建议使用名为 `create` 的变量来保存 `DSLContext`。

您可以使用 `DSLContext` 构建查询：

```java
public List<GregorianCalendar> authorsBornAfter1980() {
	return this.create.selectFrom(AUTHOR)
		.where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
		.fetch(AUTHOR.DATE_OF_BIRTH);
}
```

<a id="_jooq_sql_dialect"></a>

#### 30.6.3、jOOQ SQL 方言

除非配置了 `spring.jooq.sql-dialect` 属性，否则 Spring Boot 会自动判定用于数据源的 SQL 方言。如果 Spring Boot 无法检测到方言，则使用 `DEFAULT`。

**注意**

> Spring Boot 只能自动配置 jOOQ 开源版本支持的方言。

<a id="_customizing_jooq"></a>

#### 30.6.4、自定义 jOOQ

可通过定义自己的 `@Bean` 来实现更高级的功能，这些自定义将在创建 jOOQ `Configuration` 时使用。您可以为以下 jOOQ 类型定义 bean：

- `ConnectionProvider`
- `ExecutorProvider`
- `TransactionProvider`
- `RecordMapperProvider`
- `RecordUnmapperProvider`
- `RecordListenerProvider`
- `ExecuteListenerProvider`
- `VisitListenerProvider`
- `TransactionListenerProvider`

如果要完全控制 jOOQ 配置，您可以创建自己的 `org.jooq.Configuration` `@Bean`。

<a id="boot-features-nosql"></a>

## 31、使用 NoSQL 技术

Spring Data 提供了其他项目，可以帮助您访问各种 NoSQL 技术，包括 [MongoDB](https://projects.spring.io/spring-data-mongodb/)、[Neo4J](https://projects.spring.io/spring-data-neo4j/)、[Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch/)、[Solr](https://projects.spring.io/spring-data-solr/)、[Redis](https://projects.spring.io/spring-data-redis/)、[Gemfire](https://projects.spring.io/spring-data-gemfire/)、[Cassandra](https://projects.spring.io/spring-data-cassandra/)、[Couchbase](https://projects.spring.io/spring-data-couchbase/) 和 [LDAP](https://projects.spring.io/spring-data-ldap/)。Spring Boot 为 Redis、MongoDB、Neo4j、Elasticsearch、Solr、Cassandra、Couchbase 和 LDAP 提供了自动配置。您也可以使用其他项目，但您需要自行配置他们。请参阅 [projects.spring.io/spring-data](https://projects.spring.io/spring-data) 中相应的参考文档。

<a id="boot-features-redis"></a>

### 31.1、Redis

[Redis](http://redis.io/) 是一个集缓存、消息代理和键值存储等丰富功能的数据库。Spring Boot 为 [Lettuce](https://github.com/lettuce-io/lettuce-core/) 和 [Jedis 客户端类库](https://github.com/xetorthio/jedis/)提供了基本自动配置，[Spring Data Redis](https://github.com/spring-projects/spring-data-redis) 为他们提供了上层抽象。

使用 `spring-boot-starter-data-redis` starter 可方便地引入相关依赖。默认情况下，它使用 [Lettuce](https://github.com/lettuce-io/lettuce-core/)。该 starter 可处理传统应用程序和响应式应用程序。

**提示**

> 我们还提供了一个 `spring-boot-starter-data-redis-reactive` starter，以便与其他带有响应式支持的存储保持一致。

<a id="boot-features-connecting-to-redis"></a>

#### 31.1.1、连接 Redis

您可以像所有 Spring Bean 一样注入自动配置的 `RedisConnectionFactory`、`StringRedisTemplate` 或普通的 `RedisTemplate` 实例。默认情况下，实例将尝试在 `localhost:6379` 上连接 Redis 服务器，以下是 bean 示例：

```java
@Component
public class MyBean {

	private StringRedisTemplate template;

	@Autowired
	public MyBean(StringRedisTemplate template) {
		this.template = template;
	}

	// ...

}
```

**提示**

> 您还可以注册任意数量个实现了 `LettuceClientConfigurationBuilderCustomizer` 的 bean，以进行更高级的自定义。如果你使用 Jedis，则可以使用 `JedisClientConfigurationBuilderCustomizer`。

如果您添加了自己的任何一个自动配置类型的 `@Bean`，它将替换默认设置（除了 `RedisTemplate`，由于排除是基于 bean 名称，而 `redisTemplate` 不是它的类型）。默认情况下，如果 `commons-pool2` 在 classpath 上，您将获得一个连接池工厂。

<a id="boot-features-mongodb"></a>

### 31.2、MongoDB

[MongoDB](https://www.mongodb.com/) 是一个开源的 NoSQL 文档数据库，其使用了类似 JSON 的模式（schema）来替代传统基于表的关系数据。Spring Boot 为 MongoDB 提供了几种便利的使用方式，包括 `spring-boot-starter-data-mongodb` 和 `spring-boot-starter-data-mongodb-reactive` starter。

<a id="boot-features-connecting-to-mongodb"></a>

#### 31.2.1、连接 MongoDB 数据库

您可以注入一个自动配置的 `org.springframework.data.mongodb.MongoDbFactory` 来访问 Mongo 数据库。默认情况下，该实例将尝试在 `mongodb://localhost/test` 上连接 MongoDB 服务器，以下示例展示了如何连接到 MongoDB 数据库：

```java
import org.springframework.data.mongodb.MongoDbFactory;
import com.mongodb.DB;

@Component
public class MyBean {

	private final MongoDbFactory mongo;

	@Autowired
	public MyBean(MongoDbFactory mongo) {
		this.mongo = mongo;
	}

	// ...

	public void example() {
		DB db = mongo.getDb();
		// ...
	}

}
```

您可以通过设置 `spring.data.mongodb.uri` 属性来更改 URL 和配置其他设置，如**副本集**（replica set）：

```ini
spring.data.mongodb.uri=mongodb://user:secret@mongo1.example.com:12345,mongo2.example.com:23456/test
```

另外，只要您使用了 Mongo 2.x，请指定 `host`/`port`。比如，您可能要在 `application.properties` 中声明以下内容：

```ini
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017
```

如果您已经定义了自己的 `MongoClient`，它将被用于自动配置合适的 `MongoDbFactory`。支持 `com.mongodb.MongoClient` 和 `com.mongodb.client.MongoClient`。

**注意**

> 如果您使用 Mongo 3.0 Java 驱动，则不支持 `spring.data.mongodb.host` 和 `spring.data.mongodb.port`。这种情况下，应该使用 `spring.data.mongodb.uri` 来提供所有配置。

**提示**

> 如果未指定 `spring.data.mongodb.port`，则使用默认值 `27017`。您可以将上述示例中的改行配置删除掉。

**提示**

> 如果您不使用 Spring Data Mongo，则可以注入 `com.mongodb.MongoClient` bean 来代替 `MongoDbFactory`。如果要完全控制建立 MongoDB 连接，您还可以声明自己的 `MongoDbFactory` 或者 `MongoClient` bean。

**注意**

> 如果您使用的是响应式驱动，则 SSL 需要 Netty。 如果 Netty 可用且 factory 尚未自定义，则自动配置会自动配置此 factory。

<a id="boot-features-mongo-template"></a>

#### 31.2.2、MongoTemplate

[Spring Data Mongo](https://projects.spring.io/spring-data-mongodb/) 提供了一个 [`MongoTemplate`](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) 类，它的设计与 Spring 的 `JdbcTemplate` 非常相似。与 `JdbcTemplate` 一样，Spring Boot 会自动配置一个 bean，以便您能注入模板：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final MongoTemplate mongoTemplate;

	@Autowired
	public MyBean(MongoTemplate mongoTemplate) {
		this.mongoTemplate = mongoTemplate;
	}

	// ...

}
```

更多详细信息，参照 [`MongoOperations`](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoOperations.html) Javadoc。

<a id="boot-features-spring-data-mongo-repositories"></a>

#### 31.2.3、Spring Data MongoDB 资源库

Spring Data 包含了对 MongoDB 资源库（repository）的支持。与之前讨论的 JPA 资源库一样，基本原理是根据方法名称自动构建查询。

事实上，Spring Data JPA 和 Spring Data MongoDB 共享通用的底层代码，因此你可以拿之前提到的 JPA 示例作为基础，假设 `City` 现在是一个 Mongo 数据类，而不是一个 JPA `@Entity`，他们方式工作相同：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

	Page<City> findAll(Pageable pageable);

	City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

**提示**

> 您可以使用 `@EntityScan` 注解来自定义文档扫描位置。

**提示**

> 有关 Spring Data MongoDB 的完整详细内容，包括其丰富的对象关系映射技术，请参考其[参考文档](https://projects.spring.io/spring-data-mongodb/)。


<a id="boot-features-mongo-embedded"></a>

#### 31.2.4、内嵌 Mongo

Spring Boot 提供了[内嵌 Mongo](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo) 的自动配置。要在 Spring Boot 应用程序中使用它，请添加依赖 `de.flapdoodle.embed:de.flapdoodle.embed.mongo`。

可以使用 `spring.data.mongodb.port` 属性来配置 Mongo 的监听端口。如果想随机分配空闲端口，请把值设置为 0。`MongoAutoConfiguration` 创建的 `MongoClient` 将自动配置随机分配的端口。

**注意**

> 如果您不配置一个自定义端口，内嵌支持将默认使用一个随机端口（而不是 27017）。

如果您的 classpath 上有 SLF4J，Mongo 产生的输出将自动路由到名为 `org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongo` 的 logger。

您可以声明自己的 `IMongodConfig` 和 `IRuntimeConfig` bean 来控制 Mongo 实例的配置和日志路由。

<a id="boot-features-neo4j"></a>

### 31.3、Neo4j

[Neo4j](http://neo4j.com/) 是一个开源的 NoSQL 图形数据库，它使用了一个节点由关系连接的富数据模型，比传统 RDBMS 的方式更适合连接大数据。Spring Boot 为 Neo4j 提供了便捷引入方式，包括 `spring-boot-starter-data-neo4j` starter。

<a id="boot-features-connecting-to-neo4j"></a>

#### 31.3.1、连接 Neo4j 数据库

您可以像任何 Spring Bean 一样注入一个自动配置的 `org.neo4j.ogm.session.Session`。默认情况下， 该实例将尝试使用在 `localhost:7687` 上使用 Bolt 协议连接到 Neo4j 服务器，以下示例展示了如何注入 一个 Neo4j `Session`：

```java
@Component
public class MyBean {

	private final Session session;

	@Autowired
	public MyBean(Session session) {
		this.session = session;
	}

	// ...

}
```

您可以通过配置 `spring.data.neo4j.*` 属性来设置 uri 和凭据：

```ini
spring.data.neo4j.uri=bolt://my-server:7687
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=secret
```

您可以通过添加自己的 `org.neo4j.ogm.config.Configuration` @Bean 来完全控制 session 创建。此外，添加 `SessionFactory` 类型的 @Bean 会禁用自动配置，因此您可以掌控所有。

<a id="boot-features-connecting-to-neo4j-embedded"></a>

#### 31.3.2、使用内嵌模式

如果您将 `org.neo4j:neo4j-ogm-embedded-driver` 添加到应用程序的依赖中，Spring Boot 将自动配置一个进程内内嵌的 Neo4j 实例，当您的应用程序关闭时，该实例将不会保留任何数据。

**注意**

> 内嵌 Neo4j OGM 驱动本身不提供 Neo4j 您必须自己声明 `org.neo4j:neo4j` 依赖，请参考 [Neo4j OGM 文档](https://neo4j.com/docs/ogm-manual/current/reference/#reference:getting-started) 获取兼容版本列表。

当 classpath 上有多个驱动时，内嵌驱动优先于其他驱动。您可以通过设置 `spring.data.neo4j.embedded.enabled=false` 来显式禁用内嵌模式。

如果内嵌驱动和 Neo4j 内核如上所述位于 classpath 上，则 [Data Neo4j 测试](#boot-features-testing-spring-boot-applications-testing-autoconfigured-neo4j-test) 会自动使用内嵌 Neo4j 实例。

**注意**

> 您可以通过在配置中提供数据库文件路径来为内嵌模式启用持久化，例如：`spring.data.neo4j.uri=file://var/tmp/graph.db`。

<a id="boot-features-neo4j-ogm-session"></a>

#### 31.3.3、Neo4jSession

默认情况下，如果您正在运行 Web 应用程序，会话（session）将被绑定到当前请求的整个处理线程（即 **Open Session in View** 模式）。如果不希望此行为，您可以在 `application.properties` 中添加以下内容：

```ini
spring.data.neo4j.open-in-view=false
```

<a id="boot-features-spring-data-neo4j-repositories"></a>

#### 31.3.4、Spring Data Neo4j 资源库

Spring Data 包括了对 Neo4j 资源库的支持。

Spring Data Neo4j 与 Spring Data JPA 共享相同的通用底层代码，因此您可以直接把之前的 JPA 示例作为基础，假设 `City` 现在是一个 Neo4j OGM `@NodeEntity`，而不是一个 JPA `@Entity`，并且资源库抽象以相同的方式工作：

```java
package com.example.myapp.domain;

import java.util.Optional;

import org.springframework.data.neo4j.repository.*;

public interface CityRepository extends Neo4jRepository<City, Long> {

	Optional<City> findOneByNameAndState(String name, String state);

}
```

`spring-boot-starter-data-neo4j` starter 支持资源库和事务管理。您可以在 `@Configuration` bean 上分别使用 `@EnableNeo4jRepositories` 和 `@EntityScan` 来自定义位置以查找资源库和实体。

**提示**

有关 Spring Data Neo4j 的完整详细信息，包括其对象映射技术，请参阅[参考文档](https://projects.spring.io/spring-data-neo4j/)。

<a id="boot-features-gemfire"></a>

### 31.4、Gemfire

[Spring Data Gemfire](https://github.com/spring-projects/spring-data-gemfire) 提供了便捷的 Spring 整合工具，用于访问 [Pivotal Gemfire](https://pivotal.io/big-data/pivotal-gemfire#details) 数据管理平台。		`spring-boot-starter-data-gemfire` starter 包含了相关依赖。目前没有针对 Gemfire 的自动配置支持，但您可以使用一个[单独的注解（@EnableGemfireRepositories）](https://github.com/spring-projects/spring-data-gemfire/blob/master/src/main/java/org/springframework/data/gemfire/repository/config/EnableGemfireRepositories.java)来启用 Spring Data 资源库。

<a id="boot-features-solr"></a>

### 31.5、Solr

[Apache Solr](https://lucene.apache.org/solr/) 是一个搜素引擎。Spring Boot 为 Solr 5 客户端类库提供了基本的自动配置，并且 [Spring Data Solr](https://github.com/spring-projects/spring-data-solr) 为其提供给了顶层抽象。相关的依赖包含在了 `spring-boot-starter-data-solr` starter 中。

<a id="boot-features-solr"></a>

#### 31.5.1、连接 Solr

您可以像其他 Spring Bean 一样注入一个自动配置的 `SolrClient` 实例。默认情况下，该实例将尝试通过 [`localhost:8983/solr`](http://localhost:8983/solr) 连接到服务器，以下示例展示了如何注入一个 Solr bean：

```java
@Component
public class MyBean {

	private SolrClient solr;

	@Autowired
	public MyBean(SolrClient solr) {
		this.solr = solr;
	}

	// ...

}
```

如果您添加了自己的 `SolrClient` 类型的 `@Bean`，它将替换掉默认配置。

<a id="boot-features-spring-data-solr-repositories"></a>

#### 31.5.2、Spring Data Solr 资源库

Spring Data 包含了对 Apache Solr 资源库的支持。与之前讨论的 JPA 资源库一样，基本原理是根据方法名称自动构造查询。

事实上，Spring Data JPA 和 Spring Data Solr 共享了相同的通用底层代码，因此您可以使用之前的 JPA 示例作为基础，假设 `City` 现在是一个 `@SolrDocument` 类，而不是一个 JPA `@Entity`，它的工作方式相同。

**提示**

> 有关 Spring Data Solr 的完整详细内容，请参考其[参考文档](https://projects.spring.io/spring-data-solr/)。

<a id="boot-features-elasticsearch"></a>

### 31.6、Elasticsearch

[Elasticsearch](https://www.elastic.co/products/elasticsearch) 是一个开源、分布式、RESTful 的实时搜索分析引擎。Spring Boot 为 Elasticsearch 提供了基本的自动配置。

Spring Boot 支持以下 HTTP 客户端：

- 官方 Java **Low Level（低级）** 和 **High Level（高级）** REST 客户端
- [Jest](https://github.com/searchbox-io/Jest)

[Spring Data Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch) 依旧使用传输客户端，您可以使用 `spring-boot-starter-data-elasticsearch` starter 引入使用它。

<a id="boot-features-connecting-to-elasticsearch-rest"></a>

#### 31.6.1、使用 REST 客户端连接 Elasticsearch

Elasticsearch 提供了两个可用于查询集群的 [REST 客户端](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)：**Low Level（低级）** 和 **High Level（高级）**。

如果您的 classpath 上存在 `org.elasticsearch.client:elasticsearch-rest-client` 依赖，则 Spring Boot 将自动配置并注册默认目标为 [`localhost:9200`](http://localhost:9200/) 的 `RestClient` bean。您可以进一步调整 `RestClient` 的配置，如下所示：

```ini
spring.elasticsearch.rest.uris=http://search.example.com:9200
spring.elasticsearch.rest.username=user
spring.elasticsearch.rest.password=secret
```

您还可以注册实现任意数量的 `RestClientBuilderCustomizer` bean，以进行更高级的自定义。要完全控制注册流程，请定义 `RestClient` bean。

如果你 classpath 上有 `org.elasticsearch.client:elasticsearch-rest-high-level-client` 依赖，Spring Boot 将自动配置一个 `RestHighLevelClient`，它包装了所有现有的 `RestClient` bean，重用其 HTTP 配置。

<a id="boot-features-connecting-to-elasticsearch-jest"></a>

#### 31.6.2、使用 Jest 连接 Elasticsearch

如果您的 classpath 上存在 `Jest`，则可以注入一个默认目标为 [`localhost:9200`](http://localhost:9200/) 的自动配置 `JestClient`。您还可以进一步调整客户端配置：

```ini
spring.elasticsearch.jest.uris=http://search.example.com:9200
spring.elasticsearch.jest.read-timeout=10000
spring.elasticsearch.jest.username=user
spring.elasticsearch.jest.password=secret
```

您还可以注册任何数量实现了 `HttpClientConfigBuilderCustomizer` 的 bean，以进行更加高级的自定义。以下示例调整了其他 HTTP 设置：

```java
static class HttpSettingsCustomizer implements HttpClientConfigBuilderCustomizer {

    @Override
    public void customize(HttpClientConfig.Builder builder) {
        builder.maxTotalConnection(100).defaultMaxTotalConnectionPerRoute(5);
    }

}
```
要完全控制注册流程，请定义一个 `JestClient` bean。

<a id="boot-features-connecting-to-elasticsearch-spring-data"></a>

#### 31.6.3、使用 Spring Data 连接 Elasticsearch

要连接 Elasticsearch，您必须提供一个或多个群集节点的地址。可以通过将 `spring.data.elasticsearch.cluster-nodes` 属性设置为以逗号分隔的 `host:port` 列表来指定地址。使用此配置，可以像其他 Spring bean 一样注入 `ElasticsearchTemplate` 或 `TransportClient`，如下所示：

```java
@Component
public class MyBean {

	private final ElasticsearchTemplate template;

	public MyBean(ElasticsearchTemplate template) {
		this.template = template;
	}

	// ...

}
```

如果您添加了自己的 `ElasticsearchTemplate` 或者 `TransportClient` `@Bean`，则其将替代默认配置。

<a id="boot-features-spring-data-elasticsearch-repositories"></a>

#### 31.6.4、Spring Data Elasticsearch 资源库

Spring Data 包含了对 Elasticsearch 资源库的支持，与之前讨论的 JPA 资源库一样，其原理是根据方法名称自动构造查询。

事实上，Spring Data JPA 与 Spring Data Elasticsearch 共享了相同的通用底层代码，因此您可以使用之前的 JPA 示例作为基础，假设 `City` 此时是一个 Elasticsearch `@Document` 类，而不是一个 JPA `@Entity`，它以相同的方式工作。

**提示**

> 有关 Spring Data Elasticsearch 的完整详细内容，请参阅其[参考文档](https://docs.spring.io/spring-data/elasticsearch/docs/)。

<a id="boot-features-cassandra"></a>

### 31.7、Cassandra

[Cassandra](https://cassandra.apache.org/) 是一个开源的分布式数据库管理系统，旨在处理商用服务器上的大量数据。Spring Boot 为 Cassandra 提供了自动配置，且 [Spring Data Cassandra](https://github.com/spring-projects/spring-data-cassandra) 为其提供了顶层抽象。相关依赖包含在 `spring-boot-starter-data-cassandra` starter 中。

<a id="boot-features-connecting-to-cassandra"></a>

#### 31.7.1、连接 Cassandra

您可以像其他 Spring Bean 一样注入一个自动配置的 `CassandraTemplate` 或 Cassandra `Session` 实例。`spring.data.cassandra.*` 属性可用于自定义连接。通常，您会提供 `keyspace-name` 和 `contact-points `属性：

```ini
spring.data.cassandra.keyspace-name=mykeyspace
spring.data.cassandra.contact-points=cassandrahost1,cassandrahost2
```

您还可以注册任意数量实现了 ClusterBuilderCustomizer 的 bean，以进行更高级的自定义。

以下代码展示了如何注入一个 Cassandra bean：

```java
@Component
public class MyBean {

	private CassandraTemplate template;

	@Autowired
	public MyBean(CassandraTemplate template) {
		this.template = template;
	}

	// ...

}
```
如果您添加了自己的类的为 `@CassandraTemplate` 的 `@Bean`，则其将替代默认值。

<a id="boot-features-spring-data-cassandra-repositories"></a>

#### 31.7.2、Spring Data Cassandra 资源库

Spring Data 包含了基本的 Cassandra 资源库支持。目前，其限制要比之前讨论的 JPA 资源库要多，并且需要在 finder 方法上使用 `@Query` 注解。

**提示**

> 有关 Spring Data Cassandra 的完整详细内容，请参阅其[参考文档](https://docs.spring.io/spring-data/cassandra/docs/)。

<a id="boot-features-couchbase"></a>

### 31.8、Couchbase

[Couchbase](https://www.couchbase.com/) 是一个开源、分布式多模型的 NoSQL 面向文档数据库，其针对交互式应用程序做了优化。Spring Boot 为 Couchbase 提供了自动配置，且 [Spring Data Couchbase](https://github.com/spring-projects/spring-data-couchbase) 为其提供了顶层抽象。相关的依赖包含在了 `spring-boot-starter-data-couchbase` starter 中。

<a id="boot-features-connecting-to-couchbase"></a>

#### 31.8.1、连接 Couchbase

您可以通过添加 Couchbase SDK 和一些配置来轻松获取 `Bucket` 和 `Cluster`。`spring.couchbase.*` 属性可用于自定义连接。通常您会配置 bootstrap host、bucket name 和 password：

```ini
spring.couchbase.bootstrap-hosts=my-host-1,192.168.1.123
spring.couchbase.bucket.name=my-bucket
spring.couchbase.bucket.password=secret
```

> 您**至少**需要提供 bootstrap host，这种情况下，bucket name 为 `default` 且 password 为空字符串。或者，您可以定义自己的 `org.springframework.data.couchbase.config.CouchbaseConfigurer` @Bean 来控制整个配置。

也可以自定义一些 `CouchbaseEnvironment` 设置。例如，以下配置修改了打开一个新 `Bucket` 的超时时间和开启了 SSL 支持：

```ini
spring.couchbase.env.timeouts.connect=3000
spring.couchbase.env.ssl.key-store=/location/of/keystore.jks
spring.couchbase.env.ssl.key-store-password=secret
```
查看 `spring.couchbase.env.*` 获取更多详细内容。

<a id="boot-features-spring-data-couchbase-repositories"></a>

#### 31.8.2、Spring Data Couchbase 资源库

Spring Data 包含了 Couchbase 资源库支持。有关 Spring Data Couchbase 的完整详细信息，请参阅其[参考文档](https://docs.spring.io/spring-data/couchbase/docs/current/reference/html/)。

您可以像使用其他 Spring Bean 一样注入自动配置的 `CouchbaseTemplate` 实例，前提是有一个默认的`CouchbaseConfigurer`（当您启用 Couchbase 支持时会发生这种情况，如之前所述）。

以下示例展示了如何注入一个 Couchbase bean：

```java
@Component
public class MyBean {

	private final CouchbaseTemplate template;

	@Autowired
	public MyBean(CouchbaseTemplate template) {
		this.template = template;
	}

	// ...

}
```

您可以在自己的配置中定义以下几个 bean，以覆盖自动配置提供的配置：

- 一个名为 `couchbaseTemplate` 的 `CouchbaseTemplate` @Bean
- 一个名为 `couchbaseIndexManager` 的 `IndexManager` @Bean
- 一个名为 `couchbaseCustomConversions` 的 `CustomConversions` @Bean

为了避免在自己的配置中硬编码这些名称，您可以重用 Spring Data Couchbase 提供的 `BeanNames`，例如，您可以自定义转换器，如下：

```java
@Configuration
public class SomeConfiguration {

	@Bean(BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
	public CustomConversions myCustomConversions() {
		return new CustomConversions(...);
	}

	// ...

}
```

**提示**

如果您想要安全绕开 Spring Data Couchbase 的自动配置，请提供自己的 `org.springframework.data.couchbase.config.AbstractCouchbaseDataConfiguration` 实现。

<a id="boot-features-ldap"></a>

### 31.9、LDAP

[LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)（Lightweight Directory Access Protocol，轻量级目录访问协议）是一个开放、厂商中立的行业标准应用协议，其通过 IP 网络访问和维护分布式目录信息服务。Spring Boot 为兼容 LDAP 服务器提供了自动配置，以及支持从 [UnboundID](https://www.ldap.com/unboundid-ldap-sdk-for-java) 内嵌内存式 LDAP 服务器。

[Spring Data LDAP](https://github.com/spring-projects/spring-data-ldap) 提供了 LDAP 抽象。相关依赖包含在了 `spring-boot-starter-data-ldap` starter 中。

<a id="boot-features-ldap-connecting"></a>

#### 31.9.1、连接 LDAP 服务器

要连接 LDAP 服务器，请确保您已经声明了 `spring-boot-starter-data-ldap` starter 或者 `spring-ldap-core` 依赖，然后在 `application.properties` 声明服务器的 URL：

```ini
spring.ldap.urls=ldap://myserver:1235
spring.ldap.username=admin
spring.ldap.password=secret
```

如果需要自定义连接设置，您可以使用 `spring.ldap.base` 和 `spring.ldap.base-environment` 属性。

`LdapContextSource` 将根据这些设置自动配置。如果您需要自定义它，例如使用一个 `PooledContextSource`，则仍然可以注入自动配置的 `LdapContextSource`。确保将自定义的 `ContextSource` 标记为 `@Primary`，以便自动配置的 `LdapTemplate` 能使用它。

<a id="boot-features-ldap-spring-data-repositories"></a>

#### 31.9.2、Spring Data LDAP 资源库

Spring Data 包含了 LDAP 资源库支持。有关 Spring Data LDAP 的完整详细信息，请参阅其[参考文档](https://docs.spring.io/spring-data/ldap/docs/1.0.x/reference/html/)。

您还可以像其他 Spring Bean 一样注入一个自动配置的 `LdapTemplate` 实例：

```java
@Component
public class MyBean {

	private final LdapTemplate template;

	@Autowired
	public MyBean(LdapTemplate template) {
		this.template = template;
	}

	// ...

}
```

<a id="boot-features-ldap-embedded"></a>

#### 31.9.3、内嵌内存式 LDAP 服务器

为了测试目的，Spring Boot 支持从 [UnboundID](https://www.ldap.com/unboundid-ldap-sdk-for-java) 自动配置一个内存式 LDAP 服务器。要配置服务器，请添加 `com.unboundid:unboundid-ldapsdk` 依赖并声明一个 `base-dn` 属性：

```ini
spring.ldap.embedded.base-dn=dc=spring,dc=io
```

**注意**

<blockquote>

可以定义多个 `base-dn` 值，但是，由于名称包含逗号，存在歧义，因此必须使用正确的符号来定义它们。

在 yaml 文件中，您可以使用 yaml 列表表示法：

```yaml
spring.ldap.embedded.base-dn:
  - dc=spring,dc=io
  - dc=pivotal,dc=io
```

在属性文件中，必须使用索引方式：

```ini
spring.ldap.embedded.base-dn[0]=dc=spring,dc=io
spring.ldap.embedded.base-dn[1]=dc=pivotal,dc=io
```

</blockquote>

默认情况下，服务器将在一个随机端口上启动，并触发常规的 LDAP 支持（不需要指定 `spring.ldap.urls` 属性）。

如果您的 classpath 上存在一个 `schema.ldif` 文件，其将用于初始化服务器。如果您想从不同的资源中加载脚本，可以使用 `spring.ldap.embedded.ldif` 属性。

默认情况下，将使用一个标准模式（schema）来校验 `LDIF` 文件。您可以使用 `spring.ldap.embedded.validation.enabled` 属性来关闭所有校验。如果您有自定义的属性，则可以使用 `spring.ldap.embedded.validation.schema` 来定义自定义属性类型或者对象类。

<a id="boot-features-influxdb"></a>

### 31.10、InfluxDB

[InfluxDB](https://www.influxdata.com/) 是一个开源时列数据库，其针对运营监控、应用程序指标、物联网传感器数据和实时分析等领域中的时间序列数据在速度、高可用存储和检索方面进行了优化。

<a id="boot-features-connecting-to-influxdb"></a>

#### 31.10.1、连接 InfluxDB

Spring Boot 自动配置 `InfluxDB` 实例，前提是 `Influxdb-java` 客户端在 classpath 上并且设置了数据库的 URL，如下所示：

```ini
spring.influx.url=HTTP://172.0.0.1:8086
```

如果与 InfluxDB 的连接需要用户和密码，则可以相应地设置 `spring.influx.user` 和 `spring.influx.password` 属性。

InfluxDB 依赖于 OkHttp。如果你需要调整 `InfluxDB` 在底层使用的 http 客户端，则可以注册一个 `InfluxDbOkHttpClientBuilderProvider` bean。

<a id="boot-features-caching"></a>

## 32、缓存

Spring Framework 支持以透明的方式向应用程序添加缓存。从本质上讲，将缓存应用于方法上，根据缓存数据减少方法的执行次数。缓存逻辑是透明的，不会对调用者造成任何干扰。通过 `@EnableCaching` 注解启用缓存支持，Spring Boot 就会自动配置缓存设置。

**注意**

> 有关更多详细信息，请查看 Spring Framework 参考文档的[相关部分](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#cache)。

简而言之，为服务添加缓存的操作就像在其方法中添加注解一样简单，如下所示：

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Component;

@Component
public class MathService {

	@Cacheable("piDecimals")
	public int computePiDecimal(int i) {
		// ...
	}

}
```

此示例展示了如何在代价可能高昂的操作上使用缓存。在调用 `computePiDecimal` 之前，缓存支持会在 `piDecimals` 缓存中查找与 `i` 参数匹配的项。如果找到，则缓存中的内容会立即返回给调用者，并不会调用该方法。否则，将调用该方法，并在返回值之前更新缓存。

**注意**

> 您还可以使用标准 JSR-107（JCache）注解（例如 `@CacheResult`）。但是，我们强烈建议您**不要**将 Spring Cache 和 JCache 注解混合使用。

如果您不添加任何指定的缓存库，Spring Boot 会自动配置一个使用并发 map 的 [simple provider](#boot-features-caching-provider-simple)。当需要缓存时（例如前面示例中的 `piDecimals`），该 simple provider 会为您创建缓存。不推荐将 simple provider 用于生产环境，但它非常适合入门并帮助您了解这些功能。当您决定使用缓存提供者时，请务必阅读其文档以了解如何配置应用程序。几乎所有提供者都要求您显式配置应用程序中使用的每个缓存。有些提供了自定义 `spring.cache.cache-names` 属性以定义默认缓存。

**提示**

> 还可以透明地[更新](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#cache-annotations-put)或[回收](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#cache-annotations-evict)缓存中的数据。

<a id="boot-features-caching-provider"></a>

### 32.1、支持的缓存提供者

缓存抽象不提供存储实现，其依赖于 `org.springframework.cache.Cache` 和 `org.springframework.cache.CacheManager` 接口实现的抽象。

如果您未定义 `CacheManager` 类型的 bean 或名为 `cacheResolver` 的 `CacheResolver`（请参阅 [`CachingConfigurer`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/cache/annotation/CachingConfigurer.html)），则 Spring Boot 会尝试检测以下提供者（按序号顺序）：

1. [Generic](#boot-features-caching-provider-generic)
2. [JCache (JSR-107)](#boot-features-caching-provider-jcache)(EhCache 3、Hazelcast、Infinispan 或其他)
3. [EhCache 2.x](#boot-features-caching-provider-ehcache2)
4. [Hazelcast](#boot-features-caching-provider-hazelcast)
5. [Infinispan](#boot-features-caching-provider-infinispan)
6. [Couchbase](#boot-features-caching-provider-couchbase)
7. [Redis](#boot-features-caching-provider-redis)
8 [Caffeine](#boot-features-caching-provider-caffeine)
9. [Simple](#boot-features-caching-provider-simple)

**提示**

> 也可以通过设置 `spring.cache.type` 属性来强制指定缓存提供者。如果您需要在某些环境（比如测试）中[完全禁用缓存](#boot-features-caching-provider-none)，请使用此属性。

**提示**

> 使用 `spring-boot-starter-cache` starter 快速添加基本的缓存依赖。starter 引入了 `spring-context-support`。如果手动添加依赖，则必须包含 `spring-context-support` 才能使用 JCache、EhCache 2.x 或 Guava 支持。

如果通过 Spring Boot 自动配置 `CacheManager`，则可以通过暴露一个实现了 `CacheManagerCustomizer` 接口的 bean，在完全初始化之前进一步调整其配置。以下示例设置了一个 flag，表示应将 null 值传递给底层 map：

```java
@Bean
public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
	return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {
		@Override
		public void customize(ConcurrentMapCacheManager cacheManager) {
			cacheManager.setAllowNullValues(false);
		}
	};
}
```

**注意**

> 在前面示例中，需要一个自动配置的 `ConcurrentMapCacheManager`。如果不是这种情况（您提供了自己的配置或自动配置了不同的缓存提供者），则根本不会调用 customizer。您可以拥有多个 customizer，也可以使用 `@Order` 或 `Ordered` 来排序它们。

<a id="boot-features-caching-provider-generic"></a>

#### 32.1.1、Generic

如果上下文定义了**至少**一个 `org.springframework.cache.Cache` bean，则使用 Generic 缓存。将创建一个包装所有该类型 bean 的 `CacheManager`。

<a id="boot-features-caching-provider-jcache"></a>

#### 32.1.2、JCache (JSR-107)

[JCache](https://jcp.org/en/jsr/detail?id=107) 通过 classpath 上的 `javax.cache.spi.CachingProvider`（即 classpath 上存在符合 JSR-107 的缓存库）来引导，`jCacheCacheManager` 由 `spring-boot-starter-cache` starter 提供。您可以使用各种兼容库，Spring Boot 为 Ehcache 3、Hazelcast 和 Infinispan 提供依赖管理。您还可以添加任何其他兼容库。

可能存在多个提供者，在这种情况下必须明确指定提供者。即使 JSR-107 标准没有强制规定一个定义配置文件位置的标准化方法，Spring Boot 也会尽其所能设置一个包含实现细节的缓存，如下所示：

```ini
# 存在多个 provider 才需要这样做
spring.cache.jcache.provider=com.acme.MyCachingProvider
spring.cache.jcache.config=classpath:acme.xml
```

**注意**

> 当缓存库同时提供原生实现和 JSR-107 支持时，Spring Boot 更倾向 JSR-107 支持，因此当您切换到不同的 JSR-107 实现时，还可以使用相同的功能。

**提示**

> Spring Boot 对 [Hazelcast](#boot-features-hazelcast) 的支持一般。如果有一个 `HazelcastInstance` 可用，它也会自动为 `CacheManager` 复用，除非指定了 `spring.cache.jcache.config` 属性。

有两种方法可以自定义底层的 `javax.cache.cacheManager`：

- 可以通过设置 `spring.cache.cache-names` 属性在启动时创建缓存。如果定义了自定义 `javax.cache.configuration.Configuration` bean，则会使用它来自定义。
- 使用 `CacheManager` 的引用调用 `org.springframework.boot.autoconfigure.cache.JCacheManagerCustomizer` bean 以进行完全自定义。

**提示**

> 如果定义了一个标准的 `javax.cache.CacheManager` bean，它将自动包装进一个抽象所需的 `org.springframework.cache.CacheManager` 实现中，而不会应用自定义配置。

<a id="boot-features-caching-provider-ehcache2"></a>

#### 32.1.3、EhCache 2.x

如果可以在 classpath 的根目录中找到名为 `ehcache.xml` 的文件，则使用 EhCache 2.x。如果找到 EhCache 2.x，则使用 `spring-boot-starter-cache` starter 提供的 `EhCacheCacheManager` 来启动缓存管理器。还可以提供其他配置文件，如下所示：

```ini
spring.cache.ehcache.config=classpath:config/another-config.xml
```

<a id="boot-features-caching-provider-hazelcast"></a>

#### 32.1.4、Hazelcast

Spring Boot 对 [Hazelcast](#boot-features-hazelcast) 的支持一般。如果自动配置了一个 `HazelcastInstance`，它将自动包装进 `CacheManager` 中。

<a id="boot-features-caching-provider-infinispan"></a>

#### 32.1.5、Infinispan

[Infinispan](http://infinispan.org/) 没有默认的配置文件位置，因此必须明确指定。否则将使用默认配置引导。

```ini
spring.cache.infinispan.config=infinispan.xml
```

可以通过设置 `spring.cache.cache-names` 属性在启动时创建缓存。如果定义了自定义 `ConfigurationBuilder` bean，则它将用于自定义缓存。

**注意**

> Infinispan 在 Spring Boot 中的支持仅限于内嵌模式，非常简单。如果你想要更多选项，你应该使用官方的 Infinispan Spring Boot starter。有关更多详细信息，请参阅 [Infinispan 文档](https://github.com/infinispan/infinispan-spring-boot)。

<a id="boot-features-caching-provider-couchbase"></a>

#### 32.1.6、Couchbase

如果 [Couchbase](https://www.couchbase.com/) Java 客户端和 `couchbase-spring-cache` 实现可用且已经[配置](#boot-features-couchbase)了 Couchbase，则应用程序会自动配置一个 ` CouchbaseCacheManager`。也可以通过设置 `spring.cache.cache-names` 属性在启动时创建其他缓存。这些缓存在自动配置的 Bucket 上操作。您**还**可以使用 customizer 在其他 `Bucket` 上创建缓存。假设您需要在 **main** `Bucket` 上有两个缓存（`cache1` 和 `cache2`），并且 **another** `Bucket` 有一个过期时间为 2 秒的缓存（`cache3`）。您可以通过配置创建这两个缓存，如下所示：

```ini
spring.cache.cache-names=cache1,cache2
```

然后，您可以定义一个 `@Configuration` 类来配置其他 `Bucket` 和 `cache3` 缓存，如下所示：

```java
@Configuration
public class CouchbaseCacheConfiguration {

	private final Cluster cluster;

	public CouchbaseCacheConfiguration(Cluster cluster) {
		this.cluster = cluster;
	}

	@Bean
	public Bucket anotherBucket() {
		return this.cluster.openBucket("another", "secret");
	}

	@Bean
	public CacheManagerCustomizer<CouchbaseCacheManager> cacheManagerCustomizer() {
		return c -> {
			c.prepareCache("cache3", CacheBuilder.newInstance(anotherBucket())
					.withExpiration(2));
		};
	}

}
```

此示例配置复用了通过自动配置创建的 `Cluster`。

<a id="boot-features-caching-provider-redis"></a>

#### 32.1.7、Redis

如果 [Redis](http://redis.io/) 可用并已经配置，则应用程序会自动配置一个 `RedisCacheManager`。通过设置 `spring.cache.cache-names` 属性可以在启动时创建其他缓存，并且可以使用 `spring.cache.redis.*` 属性配置缓存默认值。例如，以下配置创建 `cache1` 和 `cache2` 缓存，他们的**有效时间**为 10 分钟：

```ini
spring.cache.cache-names=cache1,cache2
spring.cache.redis.time-to-live=600000
```

**注意**

> 默认情况下，会添加一个 key 前缀，这样做是因为如果两个单独的缓存使用了相同的键，Redis 不支持重叠 key，而缓存也不能返回无效值。如果您创建自己的 `RedisCacheManager`，我们强烈建议您启用此设置。

**提示**

> 您可以通过添加自己的 `RedisCacheConfiguration` `@Bean` 来完全控制配置。如果您想自定义序列化策略，这种方式可能很有用。

<a id="boot-features-caching-provider-caffeine"></a>

#### 32.1.8、Caffeine

[Caffeine](https://github.com/ben-manes/caffeine) 是一个使用了 Java 8 重写 Guava 缓存，用于取代 Guava 支持的缓存库。如果 Caffeine 存在，则应用程序会自动配置一个 `CaffeineCacheManager`（由 `spring-boot-starter-cache` starter 提供）。可以通过设置 `spring.cache.cache-names` 属性在启动时创建缓存，并且可以通过以下方式之一（按序号顺序）自定义缓存：

1. 一个由 `spring.cache.caffeine.spec` 定义的缓存规范
2. 一个已定义的 `com.github.benmanes.caffeine.cache.CaffeineSpec` bean
3. 一个已定义的 `com.github.benmanes.caffeine.cache.Caffeine` bean

例如，以下配置创建 `cache1` 和 `cache2` 缓存，最大大小为 500，**有效时间** 为 10 分钟：

```ini
spring.cache.cache-names=cache1,cache2
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```

如果定义了 `com.github.benmanes.caffeine.cache.CacheLoader` bean，它将自动与 `CaffeineCacheManager` 关联。由于 `CacheLoader` 将与缓存管理器管理的**所有**缓存相关联，因此必须将其定义为 `CacheLoader<Object, Object>`。自动配置会忽略所有其他泛型类型。

<a id="boot-features-caching-provider-simple"></a>

#### 32.1.9、Simple

如果找不到其他提供者，则配置使用一个 `ConcurrentHashMap` 作为缓存存储的简单实现。如果您的应用程序中没有缓存库，则该项为默认值。默认情况下，会根据需要创建缓存，但您可以通过设置 `cache-names` 属性来限制可用缓存的列表。例如，如果只需要 `cache1` 和 `cache2` 缓存，请按如下设置 `cache-names` 属性：

```ini
spring.cache.cache-names=cache1,cache2
```

如果这样做了，并且您的应用程序使用了未列出的缓存，则运行时在它需要缓存时会触发失败，但在启动时则不会。这类似于**真实**缓存提供者在使用未声明的缓存时触发的行为方式。

<a id="boot-features-caching-provider-none"></a>

#### 32.1.10、None

当配置中存在 `@EnableCaching` 时，也需要合适的缓存配置。如果需要在某些环境中完全禁用缓存，请将缓存类型强制设置为 `none` 以使用 no-op 实现，如下所示：

```ini
spring.cache.type=none
```

<a id="boot-features-messaging"></a>

## 33、消息传递

Spring Framework 为消息传递系统集成提供了广泛的支持，从使用 `JmsTemplate` 简化 JMS API 的使用到异步接收消息的完整基础设施。Spring AMQP 为高级消息队列协议（Advanced Message Queuing Protocol，AMQP）提供了类似的功能集合。Spring Boot 还为 `RabbitTemplate` 和 RabbitMQ 提供自动配置选项。Spring WebSocket 本身包含了对 STOMP 消息传递的支持，Spring Boot 通过 starter 和少量自动配置即可支持它。Spring Boot 同样支持 Apache Kafka。

<a id="boot-features-jms"></a>

### 33.1、JMS

`javax.jms.ConnectionFactory` 接口提供了一种创建 `javax.jms.Connection` 的标准方法，可与 JMS broker（代理）进行交互。虽然 Spring 需要一个 `ConnectionFactory` 来与 JMS 一同工作，但是您通常不需要自己直接使用它，而是可以依赖更高级别的消息传递抽象。（有关详细信息，请参阅 Spring Framework 参考文档的[相关部分](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#jms)。）Spring Boot 还会自动配置发送和接收消息所需的基础设施。

<a id="boot-features-activemq"></a>

#### 33.1.1、ActiveMQ 支持

当 [ActiveMQ](http://activemq.apache.org/) 在 classpath 上可用时，Spring Boot 也可以配置一个 `ConnectionFactory`。如果 broker 存在，则会自动启动并配置一个内嵌式 broker（前提是未通过配置指定 broder URL）。

**注意**

> 如果使用 `spring-boot-starter-activemq`，则提供了连接到 ActiveMQ 实例必须依赖或内嵌一个 ActiveMQ 实例，以及与 JMS 集成的 Spring 基础设施。

ActiveMQ 配置由 `spring.activemq.*` 中的外部配置属性控制。例如，您可以在 `application.properties` 中声明以下部分：

```ini
spring.activemq.broker-url=tcp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret
```

默认情况下，`CachingConnectionFactory` 将原生的 `ConnectionFactory` 使用可由 `spring.jms.*` 中的外部配置属性控制的合理设置包装起来：

```ini
spring.jms.cache.session-cache-size=5
```

如果您更愿意使用原生池，则可以通过向 `org.messaginghub:pooled-jms` 添加一个依赖并相应地配置 `JmsPoolConnectionFactory` 来实现，如下所示：

```ini
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=50
```

**提示**

> 有关更多支持的选项，请参阅 [`ActiveMQProperties`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java)。您还可以注册多个实现了 `ActiveMQConnectionFactoryCustomizer` 的的 bean，以进行更高级的自定义。

默认情况下，ActiveMQ 会创建一个 destination（目标）（如果它尚不存在），以便根据提供的名称解析 destination。

<a id="boot-features-artemis"></a>

#### 33.1.2、Artemis 支持

Spring Boot 可以在检测到 [Artemis](http://activemq.apache.org/artemis/) 在 classpath 上可用时自动配置一个 `ConnectionFactory`。如果存在 broker，则会自动启动并配置一个内嵌 broker（除非已明确设置 mode 属性）。支持的 mode 为 `embedded`（明确表示需要一个内嵌 broker，如果 broker 在 classpath 上不可用则发生错误）和 `native`（使用 netty 传输协议连接到 broker）。配置后者后，Spring Boot 会使用默认设置配置一个 `ConnectionFactory`，该 `ConnectionFactory` 连接到在本地计算机上运行的 broker。

**注意**

> 如果使用了 `spring-boot-starter-artemis`，则会提供连接到现有的 Artemis 实例的必须依赖，以及与 JMS 集成的Spring 基础设施。将 `org.apache.activemq:artemis-jms-server` 添加到您的应用程序可让您使用内嵌模式。

Artemis 配置由 `spring.artemis.*` 中的外部配置属性控制。例如，您可以在 `application.properties` 中声明以下部分：

```ini
spring.artemis.mode=native
spring.artemis.host=192.168.1.210
spring.artemis.port=9876
spring.artemis.user=admin
spring.artemis.password=secret
```

内嵌 broker 时，您可以选择是否要启用持久化并列出应该可用的 destination。可以将这些指定为以逗号分隔的列表，以使用默认选项创建它们，也可以定义类型为 `org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration` 或 `org.apache.activemq.artemis.jms.server.config.TopicConfiguration` 的 bean，分别用于高级队列和 topic（主题）配置。

默认情况下，`CachingConnectionFactory` 将原生的 `ConnectionFactory` 使用可由 `spring.jms.*` 中的外部配置属性控制的合理设置包装起来：

```ini
spring.jms.cache.session-cache-size=5
```

如果您更愿意使用原生池，则可以通过向 `org.messaginghub:pooled-jms` 添加一个依赖并相应地配置 `JmsPoolConnectionFactory` 来实现，如下所示：

```ini
spring.artemis.pool.enabled=true
spring.artemis.pool.max-connections=50
```

有关更多支持的选项，请参阅 [`ArtemisProperties`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/artemis/ArtemisProperties.java)。

不涉及 JNDI 查找，使用 Artemis 配置中的 `name` 属性或通过配置提供的名称来解析目标（destination）名称。

<a id="boot-features-jms-jndi"></a>

#### 33.1.3、使用 JNDI ConnectionFactory

如果您在应用程序服务器中运行应用程序，Spring Boot 会尝试使用 JNDI 找到 JMS `ConnectionFactory`。默认情况下，将检查 `java:/JmsXA` 和 `java:/XAConnectionFactory` 这两个位置。如果需要指定其他位置，可以使用 `spring.jms.jndi-name` 属性，如下所示：

```ini
spring.jms.jndi-name=java:/MyConnectionFactory
```

<a id="boot-features-using-jms-sending"></a>

#### 33.1.4、发送消息

Spring 的 `JmsTemplate` 是自动配置的，你可以直接将它注入到你自己的 bean 中，如下所示：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final JmsTemplate jmsTemplate;

	@Autowired
	public MyBean(JmsTemplate jmsTemplate) {
		this.jmsTemplate = jmsTemplate;
	}

	// ...

}
```

**注意**

> `JmsMessagingTemplate` 可以以类似的方式注入。如果定义了 `DestinationResolver` 或 `MessageConverter` bean，它将自动关联到自动配置的 `JmsTemplate`。

<a id="boot-features-using-jms-receiving"></a>

#### 33.1.5、接收消息

当存在 JMS 基础设施时，可以使用 `@JmsListener` 对任何 bean 进行注解以创建监听器（listener）端点。如果未定义 `JmsListenerContainerFactory`，则会自动配置一个默认的（factory）。如果定义了 `DestinationResolver` 或 `MessageConverter` bean，它将自动关联到默认的 factory。

默认情况下，默认 factory 是具有事务特性的。如果您在存在有 `JtaTransactionManager` 的基础设施中运行，则默认情况下它与监听器容器相关联。如果不是，则 `sessionTransacted` flag 将为启用（enabled）。在后一种情况下，您可以通过在监听器方法（或其委托）上添加 `@Transactional`，将本地数据存储事务与传入消息的处理相关联。这确保了在本地事务完成后传入消息能被告知。这还包括了发送已在同一 JMS 会话上执行的响应消息。

以下组件在 `someQueue` destination 上创建一个监听器端点：

```java
@Component
public class MyBean {

	@JmsListener(destination = "someQueue")
	public void processMessage(String content) {
		// ...
	}

}
```

**提示**

> 有关更多详细信息，请参阅 [`@EnableJms` 的 Javadoc](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/jms/annotation/EnableJms.html)。

如果需要创建更多 `JmsListenerContainerFactory` 实例或覆盖缺省值，Spring Boot 会提供一个 `DefaultJmsListenerContainerFactoryConfigurer`，您可以使用它来初始化 `DefaultJmsListenerContainerFactory`，其设置与自动配置的 factory 设置相同。

例如，以下示例暴露了另一个使用特定 `MessageConverter` 的 factory：

```java
@Configuration
static class JmsConfiguration {

	@Bean
	public DefaultJmsListenerContainerFactory myFactory(
			DefaultJmsListenerContainerFactoryConfigurer configurer) {
		DefaultJmsListenerContainerFactory factory =
				new DefaultJmsListenerContainerFactory();
		configurer.configure(factory, connectionFactory());
		factory.setMessageConverter(myMessageConverter());
		return factory;
	}

}
```

然后，您可以在任何 `@JmsListener` 注解的方法中使用该 factory，如下所示：

```java
@Component
public class MyBean {

	@JmsListener(destination = "someQueue", containerFactory="myFactory")
	public void processMessage(String content) {
		// ...
	}

}
```

<a id="boot-features-amqp"></a>

### 33.2、AMQP

高级消息队列协议（Advanced Message Queuing Protocol，AMQP）是一个平台无关，面向消息中间件的连接级协议。Spring AMQP 项目将核心 Spring 概念应用于基于 AMQP 消息传递解决方案的开发。Spring Boot 为通过 RabbitMQ 使用 AMQP 提供了一些快捷方法，包括 `spring-boot-starter-amqp` starter。

<a id="boot-features-rabbitmq"></a>

#### 33.2.1、RabbitMQ 支持

[RabbitMQ](https://www.rabbitmq.com/) 是一个基于 AMQP 协议的轻量级、可靠、可扩展且可移植的消息代理。Spring 使用 RabbitMQ 通过 AMQP 协议进行通信。

RabbitMQ 配置由 `spring.rabbitmq.*` 中的外部配置属性控制。例如，您可以在 `application.properties` 中声明以下部分：

```ini
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=secret
```

如果上下文中存在 `ConnectionNameStrategy` bean，它将自动用于命名由自动配置的 `ConnectionFactory` 所创建的连接。有关更多支持的选项，请参阅 [RabbitProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java)。

**提示**

> 有关详细信息，请参阅[理解 AMQP、RabbitMQ 使用的协议](https://spring.io/blog/2010/06/14/understanding-amqp-the-protocol-used-by-rabbitmq/)。

<a id="boot-features-using-amqp-sending"></a>

#### 33.2.2、发送消息

Spring 的 `AmqpTemplate` 和 `AmqpAdmin` 是自动配置的，您可以将它们直接注入自己的 bean 中，如下所示：

```java
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final AmqpAdmin amqpAdmin;
	private final AmqpTemplate amqpTemplate;

	@Autowired
	public MyBean(AmqpAdmin amqpAdmin, AmqpTemplate amqpTemplate) {
		this.amqpAdmin = amqpAdmin;
		this.amqpTemplate = amqpTemplate;
	}

	// ...

}
```

**注意**

> RabbitMessagingTemplate 可以以类似的方式注入。如果定义了 `MessageConverter` bean，它将自动关联到自动配置的 `AmqpTemplate`。

如有必要，所有定义为 bean 的 `org.springframework.amqp.core.Queue` 都会自动在 RabbitMQ 实例上声明相应的队列。

要重试操作，可以在 `AmqpTemplate` 上启用重试（例如，在 broker 连接丢失的情况下）：

```ini
spring.rabbitmq.template.retry.enabled=true
spring.rabbitmq.template.retry.initial-interval=2s
```

默认情况下禁用重试。您还可以通过声明 `RabbitRetryTemplateCustomizer` bean 以编程方式自定义 `RetryTemplate`。

<a id="boot-features-using-amqp-receiving"></a>

#### 33.2.3、接收消息

当 Rabbit 基础设施存在时，可以使用 `@RabbitListener` 注解任何 bean 以创建监听器端点。如果未定义 `RabbitListenerContainerFactory`，则会自动配置一个默认的 `SimpleRabbitListenerContainerFactory`，您可以使用 `spring.rabbitmq.listener.type` 属性切换到一个直接容器。如果定义了 `MessageConverter` 或 `MessageRecoverer` bean，它将自动与默认 factory 关联。

以下示例组件在 `someQueue` 队列上创建一个侦听监听器端点：

```java
@Component
public class MyBean {

	@RabbitListener(queues = "someQueue")
	public void processMessage(String content) {
		// ...
	}

}
```

**提示**

> 有关更多详细信息，请参阅 [`@EnableRabbit` 的 Javadoc](https://docs.spring.io/spring-amqp/docs/current/api/org/springframework/amqp/rabbit/annotation/EnableRabbit.html)。

如果需要创建更多 `RabbitListenerContainerFactory` 实例或覆盖缺省值，Spring Boot 提供了一个 `SimpleRabbitListenerContainerFactoryConfigurer` 和一个 `DirectRabbitListenerContainerFactoryConfigurer`，您可以使用它来初始化 `SimpleRabbitListenerContainerFactory` 和 `DirectRabbitListenerContainerFactory`，其设置与使用自动配置的 factory 相同。

**提示**

> 这两个 bean 与您选择的容器类型没有关系，它们通过自动配置暴露。

例如，以下配置类暴露了另一个使用特定 `MessageConverter` 的 factory：

```java
@Configuration
static class RabbitConfiguration {

	@Bean
	public SimpleRabbitListenerContainerFactory myFactory(
			SimpleRabbitListenerContainerFactoryConfigurer configurer) {
		SimpleRabbitListenerContainerFactory factory =
				new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		factory.setMessageConverter(myMessageConverter());
		return factory;
	}

}
```

然后，您可以在任何 `@RabbitListener` 注解的方法中使用该 factory，如下所示：

```java
@Component
public class MyBean {

	@RabbitListener(queues = "someQueue", containerFactory="myFactory")
	public void processMessage(String content) {
		// ...
	}

}
```

您可以启用重试机制来处理监听器的异常抛出情况。默认情况下使用 `RejectAndDontRequeueRecoverer`，但您可以定义自己的 `MessageRecoverer`。如果 broker 配置了重试机制，当重试次数耗尽时，则拒绝消息并将其丢弃或路由到死信（dead-letter）exchange 中。默认情况下重试机制为禁用。您还可以通过声明 `RabbitRetryTemplateCustomizer` bean 以编程方式自定义 `RetryTemplate`。

**重要**

> 默认情况下，如果禁用重试并且监听器异常抛出，则会无限期地重试传递。您可以通过两种方式修改此行为：将 `defaultRequeueRejected` 属性设置为 `false`，以便尝试零重传或抛出 `AmqpRejectAndDontRequeueException` 以指示拒绝该消息。后者是启用重试并且达到最大传递尝试次数时使用的机制。

<a id="boot-features-kafka"></a>

### 33.3、Apache Kafka 支持

通过提供 `spring-kafka` 项目的自动配置来支持 [Apache Kafka](https://kafka.apache.org/)。

Kafka 配置由 `spring.kafka.*` 中的外部配置属性控制。例如，您可以在 `application.properties` 中声明以下部分：

```ini
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=myGroup
```

**提示**

> 要在启动时创建主题（topic），请添加 `NewTopic` 类型的 Bean。如果主题已存在，则忽略该 bean。

有关更多支持的选项，请参阅 [`KafkaProperties`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/kafka/KafkaProperties.java)。

<a id="boot-features-kafka-sending-a-message"></a>

#### 33.3.1、发送消息

Spring 的 `KafkaTemplate` 是自动配置的，您可以直接在自己的 bean 中装配它，如下所示：

```java
@Component
public class MyBean {

	private final KafkaTemplate kafkaTemplate;

	@Autowired
	public MyBean(KafkaTemplate kafkaTemplate) {
		this.kafkaTemplate = kafkaTemplate;
	}

	// ...

}
```

**注意**

> 如果定义了属性 `spring.kafka.producer.transaction-id-prefix`，则会自动配置一个 `KafkaTransactionManager`。此外，如果定义了 `RecordMessageConverter` bean，它将自动关联到自动配置的 `KafkaTemplate`。

<a id="boot-features-kafka-receiving-a-message"></a>

#### 33.3.2、接收消息

当存在 Apache Kafka 基础设施时，可以使用 `@KafkaListener` 注解任何 bean 以创监听器端点。如果未定义 `KafkaListenerContainerFactory`，则会使用 `spring.kafka.listener.*` 中定义的 key 自动配置一个默认的 factory。

以下组件在 `someTopic` topic 上创建一个监听器端点：

```java
@Component
public class MyBean {

	@KafkaListener(topics = "someTopic")
	public void processMessage(String content) {
		// ...
	}

}
```

如果定义了 `KafkaTransactionManager` bean，它将自动关联到容器 factory。同样，如果定义了 `RecordMessageConverter`、`ErrorHandler` 或 `AfterRollbackProcessor` bean，它将自动关联到默认的 factory。

**提示**

> 自定义 `ChainedKafkaTransactionManager` 必须标记为 `@Primary`，因为它通常引用自动配置的 `KafkaTransactionManager` bean。

<a id="boot-features-kafka-streams"></a>

### 33.3.3、Kafka Stream

Spring for Apache Kafka 提供了一个工厂 bean 来创建 `StreamsBuilder` 对象并管理其 stream（流）的生命周期。只要 `kafka-streams` 在 classpath 上并且通过 `@EnableKafkaStreams` 注解启用了 Kafka Stream，Spring Boot 就会自动配置所需的 `KafkaStreamsConfiguration` bean。

启用 Kafka Stream 意味着必须设置应用程序 id 和引导服务器（bootstrap server）。可以使用 `spring.kafka.streams.application-id` 配置前者，如果未设置则默认为 `spring.application.name`。后者可以全局设置或专门为 stream 而重写。

使用专用 properties 可以设置多个其他属性，可以使用 `spring.kafka.streams.properties` 命名空间设置其他任意 Kafka 属性。有关更多信息，另请参见[第 33.3.4 节：其他 Kafka 属性](#boot-features-kafka-extra-props)。

要使用 factory bean，只需将 `StreamsBuilder` 装配到您的 `@Bean` 中，如下所示：

```java
@Configuration
@EnableKafkaStreams
static class KafkaStreamsExampleConfiguration {

	@Bean
	public KStream<Integer, String> kStream(StreamsBuilder streamsBuilder) {
		KStream<Integer, String> stream = streamsBuilder.stream("ks1In");
		stream.map((k, v) -> new KeyValue<>(k, v.toUpperCase())).to("ks1Out",
				Produced.with(Serdes.Integer(), new JsonSerde<>()));
		return stream;
	}

}
```

默认情况下，由其创建的 `StreamBuilder` 对象管理的流会自动启动。您可以使用 `spring.kafka.streams.auto-startup` 属性自定义此行为。

<a id="boot-features-kafka-extra-props"></a>

### 33.3.4、其他 Kafka 属性

自动配置支持的属性可在[附录 A、常见应用程序属性](#common-application-properties)中找到。请注意，在大多数情况下，这些属性（连接符或驼峰命名）直接映射到 Apache Kafka 点连形式属性。有关详细信息，请参阅 Apache Kafka 文档。

这些属性中的前几个适用于所有组件（生产者【producer】、使用者【consumer】、管理者【admin】和流【stream】），但如果您希望使用不同的值，则可以在组件级别指定。Apache Kafka 重要性（优先级）属性设定为 HIGH、MEDIUM 或 LOW。Spring Boot 自动配置支持所有 HIGH 重要性属性，一些选择的 MEDIUM 和 LOW 属性，以及所有没有默认值的属性。

只有 Kafka 支持的属性的子集可以直接通过 `KafkaProperties` 类获得。如果您希望使用不受支持的其他属性配置生产者或消费者，请使用以下属性：

```ini
spring.kafka.properties.prop.one=first
spring.kafka.admin.properties.prop.two=second
spring.kafka.consumer.properties.prop.three=third
spring.kafka.producer.properties.prop.four=fourth
spring.kafka.streams.properties.prop.five=fifth
```

这将常见的 `prop.one` Kafka 属性设置为 `first`（适用于生产者、消费者和管理者），`prop.two` 管理者属性为 `second`，`prop.three` 消费者属性为 `third`，`prop.four` 生产者属性为 `fourth`，`prop.five` 流属性为 `fifth`。

您还可以按如下方式配置 Spring Kafka `JsonDeserializer`：

```ini
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.Invoice
spring.kafka.consumer.properties.spring.json.trusted.packages=com.example,org.acme
```

同样，您可以禁用 `JsonSerializer` 在 header 中发送类型信息的默认行为：

```ini
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.properties.spring.json.add.type.headers=false
```

**重要**

> 以这种方式设置的属性将覆盖 Spring Boot 明确支持的任何配置项。

<a id="boot-features-resttemplate"></a>

## 34、使用 `RestTemplate` 调用 REST 服务

如果您的应用程序需要调用远程 REST 服务，这可以使用 Spring Framework 的 [`RestTemplate`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html) 类。由于 `RestTemplate` 实例在使用之前通常需要进行自定义，因此 Spring Boot 不提供任何自动配置的 `RestTemplate` bean。但是，它会自动配置 `RestTemplateBuilder`，可在需要时创建 `RestTemplate` 实例。自动配置的 `RestTemplateBuilder` 确保将合适的 `HttpMessageConverters` 应用于 `RestTemplate` 实例。

以下代码展示了一个典型示例：

```java
@Service
public class MyService {

	private final RestTemplate restTemplate;

	public MyService(RestTemplateBuilder restTemplateBuilder) {
		this.restTemplate = restTemplateBuilder.build();
	}

	public Details someRestCall(String name) {
		return this.restTemplate.getForObject("/{name}/details", Details.class, name);
	}

}
```

**提示**

> `RestTemplateBuilder` 包含许多可用于快速配置 ·RestTemplate· 的方法。例如，要添加 BASIC auth 支持，可以使用 `builder.basicAuthentication("user", "password").build()`。

<a id="boot-features-resttemplate-customization"></a>

### 34.1、自定义 RestTemplate

`RestTemplate` 自定义有三种主要方法，具体取决于您希望自定义的程度。

要想自定义的范围尽可能地窄，请注入自动配置的 `RestTemplateBuilder`，然后根据需要调用其方法。每个方法调用都返回一个新的 `RestTemplateBuilder` 实例，因此自定义只会影响当前构建器。

要在应用程序范围内添加自定义配置，请使用 `RestTemplateCustomizer` bean。所有这些 bean 都会自动注册到自动配置的 `RestTemplateBuilder`，并应用于使用它构建的所有模板。

以下示例展示了一个 customizer，它为除 `192.168.0.5` 之外的所有主机配置代理：

```java
static class ProxyCustomizer implements RestTemplateCustomizer {

	@Override
	public void customize(RestTemplate restTemplate) {
		HttpHost proxy = new HttpHost("proxy.example.com");
		HttpClient httpClient = HttpClientBuilder.create()
				.setRoutePlanner(new DefaultProxyRoutePlanner(proxy) {

					@Override
					public HttpHost determineProxy(HttpHost target,
							HttpRequest request, HttpContext context)
							throws HttpException {
						if (target.getHostName().equals("192.168.0.5")) {
							return null;
						}
						return super.determineProxy(target, request, context);
					}

				}).build();
		restTemplate.setRequestFactory(
				new HttpComponentsClientHttpRequestFactory(httpClient));
	}

}
```

最后，最极端（也很少使用）的选择是创建自己的 `RestTemplateBuilder` bean。这样做会关闭 `RestTemplateBuilder` 的自动配置，并阻止使用任何 `RestTemplateCustomizer` bean。

<a id="boot-features-webclient"></a>

## 35、使用 `WebClient` 调用 REST 服务

如果在 classpath 上存在 Spring WebFlux，则还可以选择使用 `WebClient` 来调用远程 REST 服务。与 `RestTemplate` 相比，该客户端更具函数式风格并且完全响应式。您可以在 [Spring Framework 文档的相关部分](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web-reactive.html#webflux-client)中了解有关 `WebClient` 的更多信息。

Spring Boot 为您创建并预配置了一个 `WebClient.Builder`。强烈建议将其注入您的组件中并使用它来创建 `WebClient` 实例。Spring Boot 配置该构建器以共享 HTTP 资源，以与服务器相同的方式反射编解码器设置（请参阅 [WebFlux HTTP 编解码器自动配置](#boot-features-webflux-httpcodecs)）等。

以下代码是一个典型示例：

```java
@Service
public class MyService {

	private final WebClient webClient;

	public MyService(WebClient.Builder webClientBuilder) {
		this.webClient = webClientBuilder.baseUrl("http://example.org").build();
	}

	public Mono<Details> someRestCall(String name) {
		return this.webClient.get().uri("/{name}/details", name)
						.retrieve().bodyToMono(Details.class);
	}

}
```

<a id="boot-features-webclient-runtime"></a>

### 35.1、WebClient 运行时

Spring Boot 将自动检测用于驱动 `WebClient` 的 `ClientHttpConnector`，具体取决于应用程序 classpath 上可用的类库。目前支持 Reactor Netty 和 Jetty RS 客户端。

默认情况下 `spring-boot-starter-webflux` starter 依赖于 `io.projectreactor.netty:reactor-netty`，它包含了服务器和客户端的实现。如果您选择将 `Jetty` 用作响应式服务器，则应添加 Jetty Reactive HTTP 客户端库依赖项 `org.eclipse.jetty:jetty-reactive-httpclient`。服务器和客户端使用相同的技术具有一定优势，因为它会自动在客户端和服务器之间共享 HTTP 资源。

开发人员可以通过提供自定义的 `ReactorResourceFactory` 或 `JettyResourceFactory` bean 来覆盖 Jetty 和 Reactor Netty 的资源配置 —— 这将同时应用于客户端和服务器。

如果您只希望覆盖客户端选项，则可以定义自己的 `ClientHttpConnector` bean 并完全控制客户端配置。

您可以在 [Spring Framework 参考文档中了解有关 `WebClient` 配置选项的更多信息](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web-reactive.html#webflux-client-builder)。

<a id="boot-features-webclient-customization"></a>

### 35.2、自定义 WebClient

`WebClient` 自定义有三种主要方法，具体取决于您希望自定义的程度。

要想自定义的范围尽可能地窄，请注入自动配置的 `WebClient.Builder`，然后根据需要调用其方法。`WebClient.Builder` 实例是有状态的：构建器上的任何更改都会影响到之后所有使用它创建的客户端。如果要使用相同的构建器创建多个客户端，可以考虑使用 `WebClient.Builder other = builder.clone();` 的方式克隆构建器。

要在应用程序范围内对所有 `WebClient.Builder` 实例添加自定义，可以声明 `WebClientCustomizer` bean 并在注入点局部更改 `WebClient.Builder`。

最后，您可以回退到原始 API 并使用 `WebClient.create()`。在这种情况下，不会应用自动配置或 `WebClientCustomizer`。

<a id="boot-features-validation"></a>

## 36、验证

只要 classpath 上存在 JSR-303 实现（例如 Hibernate 验证器），就会自动启用 Bean Validation 1.1 支持的方法验证功能。这允许 bean 方法在其参数和/或返回值上使用 `javax.validation` 约束进行注解。带有此类注解方法的目标类需要在类级别上使用 `@Validated` 进行注解，以便搜索其内联约束注解的方法。

例如，以下服务触发第一个参数的验证，确保其大小在 8 到 10 之间：

```java
@Service
@Validated
public class MyBean {

	public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code,
			Author author) {
		...
	}

}
```

<a id="boot-features-email"></a>

## 37、发送邮件

Spring Framework 提供了一个使用 `JavaMailSender` 接口发送电子邮件的简单抽象，Spring Boot 为其提供了自动配置以及一个 starter 模块。

**提示**

> 有关如何使用 `JavaMailSender` 的详细说明，请参阅[参考文档](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#mail)。

如果 `spring.mail.host` 和相关库（由 `spring-boot-starter-mail` 定义）可用，则创建默认的 `JavaMailSender`（如果不存在）。可以通过 `spring.mail` 命名空间中的配置项进一步自定义发件人。有关更多详细信息，请参阅 [`MailProperties`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java)。

特别是，某些默认超时时间的值是无限的，您可能想更改它以避免线程被无响应的邮件服务器阻塞，如下示例所示：

```ini
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=3000
spring.mail.properties.mail.smtp.writetimeout=5000
```

也可以使用 JNDI 中的现有 `Session` 配置一个 `JavaMailSender`：

```ini
spring.mail.jndi-name=mail/Session
```

设置 `jndi-name` 时，它优先于所有其他与 `Session` 相关的设置。

<a id="boot-features-jta"></a>

## 38、JTA 分布式事务

Spring Boot 通过使用 [Atomikos](http://www.atomikos.com/) 或 [Bitronix](https://github.com/bitronix/btm) 嵌入式事务管理器来支持跨多个 XA 资源的分布式 JTA 事务。部署在某些 Java EE 应用服务器（Application Server）上也支持 JTA 事务。

当检测到 JTA 环境时，Spring 的 `JtaTransactionManager` 将用于管理事务。自动配置的 JMS、DataSource 和 JPA bean 已升级为支持 XA 事务。您可以使用标准的 Spring 方式（例如 `@Transactional`）来使用分布式事务。如果您处于 JTA 环境中并且仍想使用本地事务，则可以将 `spring.jta.enabled` 属性设置为 `false` 以禁用 JTA 自动配置。

<a id="boot-features-jta-atomikos"></a>

### 38.1、使用 Atomikos 事务管理器

[Atomikos](https://www.atomikos.com/) 是一个流行的开源事务管理器，可以嵌入到 Spring Boot 应用程序中。您可以使用 `spring-boot-starter-jta-atomikos` starter 来获取相应的 Atomikos 库。Spring Boot 自动配置 Atomikos 并确保将合适的依赖设置应用于 Spring bean，以确保启动和关闭顺序正确。

默认情况下，Atomikos 事务日志将写入应用程序主目录（应用程序 jar 文件所在的目录）中的 `transaction-logs` 目录。您可以通过在 `application.properties` 文件中设置 `spring.jta.log-dir` 属性来自定义此目录的位置。也可用 `spring.jta.atomikos.properties` 开头的属性来自定义 Atomikos `UserTransactionServiceImp`。有关完整的详细信息，请参阅 [AtomikosProperties Javadoc](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html)。

**注意**

> 为确保多个事务管理器可以安全地协调相同的资源管理器，必须为每个 Atomikos 实例配置唯一 ID。默认情况下，此 ID 是运行 Atomikos 的计算机的 IP 地址。在生产环境中要确保唯一性，应为应用程序的每个实例配置 `spring.jta.transaction-manager-id` 属性，并使用不同的值。

<a id="boot-features-jta-bitronix"></a>

### 38.2、使用 Bitronix 事务管理器

[Bitronix](https://github.com/bitronix/btm) 是一个流行的开源 JTA 事务管理器实现。您可以使用 `spring-boot-starter-jta-bitronix` starter 为您的项目添加合适的 Bitronix 依赖。与 Atomikos 一样，Spring Boot 会自动配置 Bitronix 并对 bean 进行后处理（post-processes），以确保启动和关闭顺序正确。

默认情况下，Bitronix 事务日志文件（`part1.btm` 和 `part2.btm`）将写入应用程序主目录中的 `transaction-logs` 目录。您可以通过设置 `spring.jta.log-dir` 属性来自定义此目录的位置。以 `spring.jta.bitronix.properties` 开头的属性绑定到了 `bitronix.tm.Configuration` bean，允许完全自定义。有关详细信息，请参阅 [Bitronix 文档](https://github.com/bitronix/btm/wiki/Transaction-manager-configuration)。

**注意**

> 为确保多个事务管理器能够安全地协调相同的资源管理器，必须为每个 Bitronix 实例配置唯一的 ID。默认情况下，此 ID 是运行 Bitronix 的计算机的 IP 地址。生产环境要确保唯一性，应为应用程序的每个实例配置 `spring.jta.transaction-manager-id` 属性，并使用不同的值。

<a id="boot-features-jta-javaee"></a>

### 38.3、使用 Java EE 管理的事务管理器

如果将 Spring Boot 应用程序打包为 `war` 或 `ear` 文件并将其部署到 Java EE 应用程序服务器，则可以使用应用程序服务器的内置事务管理器。Spring Boot 尝试通过查找常见的 JNDI 位置（`java:comp/UserTransaction`、`java:comp/TransactionManager` 等）来自动配置事务管理器。如果使用应用程序服务器提供的事务服务，通常还需要确保所有资源都由服务器管理并通过 JNDI 暴露。Spring Boot 尝试通过在 JNDI 路径（`java:/JmsXA` 或 `java:/JmsXA`）中查找 `ConnectionFactory` 来自动配置 JMS，并且可以使用 [`spring.datasource.jndi-name` 属性](#boot-features-connecting-to-a-jndi-datasource)来配置 `DataSource`。

<a id="boot-features-jta-mixed-jms"></a>

### 38.4、混合使用 XA 与非 XA JMS 连接

使用 JTA 时，主 JMS `ConnectionFactory` bean 可识别 XA 并参与分布式事务。在某些情况下，您可能希望使用非 XA `ConnectionFactory` 处理某些 JMS 消息。例如，您的 JMS 处理逻辑可能需要比 XA 超时时间更长的时间。

如果要使用非 XA `ConnectionFactory`，可以注入 `nonXaJmsConnectionFactory` bean 而不是 `@Primary` `jmsConnectionFactory` bean。为了保持一致性，提供的 `jmsConnectionFactory` bean 还需要使用 `xaJmsConnectionFactory` 别名。

以下示例展示了如何注入 `ConnectionFactory` 实例：

```java
// Inject the primary (XA aware) ConnectionFactory
@Autowired
private ConnectionFactory defaultConnectionFactory;

// Inject the XA aware ConnectionFactory (uses the alias and injects the same as above)
@Autowired
@Qualifier("xaJmsConnectionFactory")
private ConnectionFactory xaConnectionFactory;

// Inject the non-XA aware ConnectionFactory
@Autowired
@Qualifier("nonXaJmsConnectionFactory")
private ConnectionFactory nonXaConnectionFactory;
```

<a id="boot-features-jta-supporting-alternative-embedded"></a>

### 38.5、支持嵌入式事务管理器

[`XAConnectionFactoryWrapper`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jms/XAConnectionFactoryWrapper.java) 和 [`XADataSourceWrapper`](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/XADataSourceWrapper.java) 接口可用于支持其他嵌入式事务管理器。接口负责包装 `XAConnectionFactory` 和 `XADataSource` bean，并将它们公开为普通的 `ConnectionFactory` 和 `DataSource` bean，它们透明地加入分布式事务。`DataSource` 和 JMS 自动配置使用 JTA 变体，前提是您需要有一个 `JtaTransactionManager` bean 和在 `ApplicationContext` 中注册有的相应 XA 包装器（wrapper） bean。

[BitronixXAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXAConnectionFactoryWrapper.java) 和 [BitronixXADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXADataSourceWrapper.java) 为如何编写 XA 包装器提供了很好示例。

<a id="boot-features-hazelcast"></a>

## 39、Hazelcast

如果 [Hazelcast](https://hazelcast.com/) 在 classpath 上并有合适的配置，则 Spring Boot 会自动配置一个可以在应用程序中注入的 `HazelcastInstance`。

如果定义了 `com.hazelcast.config.Config` bean，则 Spring Boot 会使用它。如果您的配置定义了实例名称，Spring Boot 会尝试查找现有的实例，而不是创建新实例。

您还可以通过配置指定要使用的 `hazelcast.xml` 配置文件，如下所示：

```ini
spring.hazelcast.config=classpath:config/my-hazelcast.xml
```

否则，Spring Boot 会尝试从默认位置查找 Hazelcast 配置：工作目录或 classpath 根目录中的 `hazelcast.xml` 。我们还检查是否设置了 `hazelcast.config` 系统属性。有关更多详细信息，请参阅 [Hazelcast 文档](http://docs.hazelcast.org/docs/latest/manual/html-single/)。

如果 classpath 中存在 `hazelcast-client`，则 Spring Boot 会首先尝试通过检查以下配置项来创建客户端：

- 存在 `com.hazelcast.client.config.ClientConfig` bean。
- `spring.hazelcast.config` 属性定义的配置文件。
- 存在 `hazelcast.client.config` 系统属性。
- 工作目录中或 classpath 根目录下的 `hazelcast-client.xml`。

**注意**

> Spring Boot 还为 Hazelcast 提供了[缓存支持](#boot-features-caching-provider-hazelcast)。如果启用了缓存，`HazelcastInstance` 将自动包装在 `CacheManager` 实现中。

<a id="boot-features-quartz"></a>

## 40、Quartz 调度器

Spring Boot 提供了几种使用 [Quartz 调度器](http://www.quartz-scheduler.org/)的便捷方式，它们来自 `spring-boot-starter-quartz` starter。如果 Quartz 可用，则 Spring Boot 将自动配置 `Scheduler`（通过 `SchedulerFactoryBean` 抽象）。

自动选取以下类型的 Bean 并将其与 `Scheduler` 关联起来：

- `JobDetail`：定义一个特定的 job。可以使用 `JobBuilder` API 构建 `JobDetail` 实例。
- `Calendar`。
- `Trigger`：定义何时触发 job。

默认使用内存存储方式的 `JobStore`。 但如果应用程序中有 `DataSource` bean，并且配置了 `spring.quartz.job-store-type` 属性，则可以配置基于 JDBC 的存储，如下所示：

```ini
spring.quartz.job-store-type=jdbc
```

使用 JDBC 存储时，可以在启动时初始化 schema（表结构），如下所示：

```ini
spring.quartz.jdbc.initialize-schema=always
```

**警告**

> 默认将使用 Quartz 库提供的标准脚本检测并初始化数据库。这些脚本会删除现有表，在每次重启时删除所有触发器。可以通过设置 `spring.quartz.jdbc.schema` 属性来提供自定义脚本。

要让 Quartz 使用除应用程序主 `DataSource` 之外的 `DataSource`，请声明一个 `DataSource` bean，使用 `@QuartzDataSource` 注解其 `@Bean` 方法。这样做可确保 `SchedulerFactoryBean` 和 schema 初始化都使用 Quartz 指定的 `DataSource`。

默认情况下，配置创建的 job 不会覆盖已从持久 job 存储读取的已注册的 job。要启用覆盖现有的 job 定义，请设置 `spring.quartz.overwrite-existing-jobs` 属性。

Quartz 调取器配置可以使用 `spring.quartz` 属性和 `SchedulerFactoryBeanCustomizer` bean 进行自定义，它们允许以编程方式的 SchedulerFactoryBean 自定义。可以使用 `spring.quartz.properties.*` 自定义高级 Quartz 配置属性。

**注意**

> 需要强调的是，`Executor` bean 与调度程序没有关联，因为 Quartz 提供了通过 `spring.quartz.properties` 配置调度器的方法。如果需要自定义执行器，请考虑实现 `SchedulerFactoryBeanCustomizer`。

job 可以定义 `setter` 以注入数据映射属性。也可以以类似的方式注入常规的 bean，如下所示：

```java
public class SampleJob extends QuartzJobBean {

	private MyService myService;

	private String name;

	// Inject "MyService" bean
	public void setMyService(MyService myService) { ... }

	// Inject the "name" job data property
	public void setName(String name) { ... }

	@Override
	protected void executeInternal(JobExecutionContext context)
			throws JobExecutionException {
		...
	}

}
```

<a id="boot-features-task-execution-scheduling"></a>

## 41、任务执行与调度

在上下文中没有 `Executor` bean 的情况下，Spring Boot 会自动配置一个有合理默认值的 `ThreadPoolTask​​Executor`，它可以自动与异步任务执行（`@EnableAsync`）和 Spring MVC 异步请求处理相关联。

**提示**

<blockquote>

如果您在上下文中定义了自定义 `Executor`，则常规任务执行（即 `@EnableAsync`）将透明地使用它，但不会配置 Spring MVC 支持，因为它需要 `AsyncTaskExecutor` 实现（名为 `applicationTaskExecutor`）。根据您的目标安排，您可以将 `Executor` 更改为 `ThreadPoolTask​​Executor`，或者定义 `Executor的ThreadPoolTask​​Executor` 和 `AsyncConfigurer` 来包装自定义的 `Executor`。

您可以使用自动配置的 `TaskExecutorBuilder` 来轻松创建实例，以复制默认的自动配置。

</blockquote>

线程池使用 8 个核心线程，可根据负载情况增加和减少。可以使用 `spring.task.execution` 命名空间对这些默认设置进行微调，如下所示：

```ini
spring.task.execution.pool.max-threads=16
spring.task.execution.pool.queue-capacity=100
spring.task.execution.pool.keep-alive=10s
```

这会将线程池更改为使用有界队列，在队列满（100 个任务）时，线程池将增加到最多 16 个线程。当线程在闲置10 秒（而不是默认的 60 秒）时回收线程，池的收缩更为明显。

如果需要与调度任务执行（`@EnableScheduling`）相关联，可以自动配置一个 `ThreadPoolTaskScheduler`。默认情况下，线程池使用一个线程，可以使用 `spring.task.scheduling` 命名空间对这些设置进行微调。

如果需要创建自定义执行器或调度器，则在上下文中可以使用 `TaskExecutorBuilder` bean 和 `TaskSchedulerBuilder` bean。

<a id="boot-features-integration"></a>

## 42、Spring Integration

Spring Boot 为 [Spring Integration](https://projects.spring.io/spring-integration/) 提供了一些便捷的使用方式，它们包含在 `spring-boot-starter-integration` starter 中。Spring Integration 为消息传递以及其他传输（如 HTTP、TCP 等）提供了抽象。如果 classpath 上存在 Spring Integration，则 Spring Boot 会通过 `@EnableIntegration` 注解对其进行初始化。

Spring Boot 还配置了一些由其他 Spring Integration 模块触发的功能。如果 `spring-integration-jmx` 也在 classpath 上，则消息处理统计信息将通过 JMX 发布。如果 `spring-integration-jdbc` 可用，则可以在启动时创建默认数据库模式，如下所示：

```ini
spring.integration.jdbc.initialize-schema=always
```

有关更多详细信息，请参阅 [IntegrationAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java) 和 [IntegrationProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationProperties.java) 类。

默认情况下，如果存在 Micrometer `meterRegistry` bean，则 Micrometer 将管理 Spring Integration 的指标。如果您希望使用旧版 Spring Integration 度量，请将 `DefaultMetricsFactory` bean 添加到应用程序上下文中。

<a id="boot-features-session"></a>

## 43、Spring Session

Spring Boot 为各种数据存储提供 [Spring Session](https://projects.spring.io/spring-session/) 自动配置。在构建 Servlet Web 应用程序时，可以自动配置以下存储：

- JDBC
- Redis
- Hazelcast
- MongoDB

构建响应式 Web 应用程序时，可以自动配置以下存储：

- Redis
- MongoDB

如果 classpath 上存在单个 Spring Session 模块，则 Spring Boot 会自动使用该存储实现。如果您有多个实现，则必须选择要用于存储会话的 [`StoreType`](https://github.com/spring-projects/spring-boot/tree/v2.1.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/session/StoreType.java)。 例如，要使用 JDBC 作为后端存储，您可以按如下方式配置应用程序：

```ini
spring.session.store-type=jdbc
```

**提示**

> 可以将 `store-type` 设置为 `none` 来禁用 Spring Session。

每个 store 都有自己的额外设置。例如，可以为 JDBC 存储定制表的名称，如下所示：

```ini
spring.session.jdbc.table-name=SESSIONS
```

可以使用 `spring.session.timeout` 属性来设置会话的超时时间。如果未设置该属性，则自动配置将使用 `server.servlet.session.timeout` 的值。

<a id="boot-features-jmx"></a>

## 44、通过 JMX 监控和管理

Java Management Extensions（JMX，Java 管理扩展）提供了一种监视和管理应用程序的标准机制。默认情况下，Spring Boot 会创建一个 ID 为 `mbeanServer` 的 `MBeanServer` bean，并暴露使用 Spring JMX 注解（`@ManagedResource`、`@ManagedAttribute` 或 `@ManagedOperation`）的 bean。

有关更多详细信息，请参阅 [`JmxAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.1.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java) 类。

<a id="boot-features-testing"></a>

## 45、测试

<a id="boot-features-websockets"></a>

## 46、WebSocket

Spring Boot 为内嵌式 Tomcat、Jetty 和 Undertow 提供了 WebSocket 自动配置。如果将 war 文件部署到独立容器，则 Spring Boot 假定容器负责配置其 WebSocket 支持。

Spring Framework 为 MVC Web 应用程序提供了[丰富的 WebSocket 支持](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/web.html#websocket)，可以通过 `spring-boot-starter-websocket` 模块轻松访问。

WebSocket 支持也可用于[响应式 Web 应用程序](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/web-reactive.html#webflux-websocket)，并且引入 WebSocket API 以及 `spring-boot-starter-webflux`：

```xml
<dependency>
	<groupId>javax.websocket</groupId>
	<artifactId>javax.websocket-api</artifactId>
</dependency>
```

<a id="boot-features-webservices"></a>

## 47、Web Service

Spring Boot 提供 Web Service 自动配置，因此您要做的就是定义 `Endpoints`。

可以使用 `spring-boot-starter-webservices` 模块轻松访问 [Spring Web Service 功能](https://docs.spring.io/spring-ws/docs/3.0.6.RELEASE/reference/)。

可以分别为 WSDL 和 XSD 自动创建 `SimpleWsdl11Definition` 和 `SimpleXsdSchema` bean。为此，请配置其位置，如下所示：

```ini
spring.webservices.wsdl-locations=classpath:/wsdl
```

<a id="boot-features-webservices-template"></a>

### 47.1、使用 `WebServiceTemplate` 调用 Web Service

如果您需要从应用程序调用远程 Web 服务，则可以使用 `WebServiceTemplate` 类。由于 `WebServiceTemplate` 实例在使用之前通常需要进行自定义，因此 Spring Boot 不提供任何自动配置的 `WebServiceTemplate` bean。但是，它会自动配置 `WebServiceTemplateBuilder`，可在需要创建 `WebServiceTemplate` 实例时使用。

以下代码为一个典型示例：

```java
@Service
public class MyService {

	private final WebServiceTemplate webServiceTemplate;

	public MyService(WebServiceTemplateBuilder webServiceTemplateBuilder) {
		this.webServiceTemplate = webServiceTemplateBuilder.build();
	}

	public DetailsResp someWsCall(DetailsReq detailsReq) {
		 return (DetailsResp) this.webServiceTemplate.marshalSendAndReceive(detailsReq, new SoapActionCallback(ACTION));

	}

}
```

默认情况下，`WebServiceTemplateBuilder` 使用 classpath 上的可用 HTTP 客户端库检测合适的基于 HTTP 的 `WebServiceMessageSender`。您还可以按如下方式自定义读取和连接的超时时间：

```java
@Bean
public WebServiceTemplate webServiceTemplate(WebServiceTemplateBuilder builder) {
	return builder.messageSenders(new HttpWebServiceMessageSenderBuilder()
			.setConnectTimeout(5000).setReadTimeout(2000).build()).build();
}
```

<a id="boot-features-developing-auto-configuration"></a>

## 49、创建自己的自动配置

如果您在公司负责开发公共类库，或者如果您在开发一个开源或商业库，您可能希望开发自己的自动配置。自动配置类可以捆绑在外部 jar 中，他仍然可以被 Spring Boot 获取。

自动配置可以与提供自动配置代码的 starter 以及您将使用的类库库相关联。我们首先介绍构建自己的自动配置需要了解的内容，然后我们将继续介绍[创建自定义 starter 所需的步骤](#boot-features-custom-starter)。

**提示**

> 这里有一个[演示项目](https://github.com/snicoll-demos/spring-boot-master-auto-configuration)展示了如何逐步创建 starter。

<a id="boot-features-understanding-auto-configured-beans"></a>

### 49.1、理解自定配置 Bean

在内部，自动配置使用了标准的 `@Configuration` 类来实现。`@Conditional` 注解用于约束何时应用自动配置。通常，自动配置类使用 `@ConditionalOnClass` 和 `@ConditionalOnMissingBean` 注解。这可确保仅在找到相关类时以及未声明您自己的 `@Configuration` 时才应用自动配置。

您可以浏览 [`spring-boot-autoconfigure`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure) 的源代码，以查看 Spring 提供的 `@Configuration` 类（请参阅 [`META-INF/spring.factories`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) 文件）。

<a id="boot-features-locating-auto-configuration-candidates"></a>

### 49.2、找到候选的自动配置

Spring Boot 会检查已发布 jar 中是否存在 `META-INF/spring.factories` 文件。该文件应列出 `EnableAutoConfiguration` key 下的配置类，如下所示：

```ini
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

**注意**

> **必须**以这种方式加载自动配置。确保它们在特定的包空间中定义，并且它们不能是组件扫描的目标。此外，自动配置类不应启用组件扫描以查找其他组件。应该使用特定的`@Imports` 来代替。

如果需要按特定顺序应用配置，则可以使用 [`@AutoConfigureAfter`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java) 或 [`@AutoConfigureBefore`](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java) 注解。例如，如果您提供特定于 Web 的配置，则可能需要在`WebMvcAutoConfiguration` 之后应用您的类。

如果您想排序某些不应该彼此直接了解的自动配置，您也可以使用 `@AutoConfigureOrder`。该注解与常规 `@Order` 注解有相同的语义，但它为自动配置类提供了专用顺序。

<a id="boot-features-condition-annotations"></a>

### 49.3、条件注解

您几乎总希望在自动配置类中包含一个或多个 `@Conditional` 注解。`@ConditionalOnMissingBean` 是一个常用的注解，其允许开发人员在对您的默认值不满意用于覆盖自动配置。

Spring Boot 包含许多 `@Conditional` 注解，您可以通过注解 `@Configuration` 类或单独的 `@Bean` 方法在您自己的代码中复用它们。这些注解包括：

- [第 49.3.1 节，类条件](#boot-features-class-conditions)
- [第 49.3.2 节，Bean 条件](#boot-features-bean-conditions)
- [第 49.3.3 节，属性条件](#boot-features-property-conditions)
- [第 49.3.4 节，资源条件](#boot-features-resource-conditions)
- [第 49.3.5 节，Web 应用程序条件](#boot-features-web-application-conditions)
- [第 49.3.6 节，SpEL 表达式条件](#boot-features-spel-conditions)

<a id="boot-features-class-conditions"></a>

#### 49.3.1、类条件

`@ConditionalOnClass` 和 `@ConditionalOnMissingClass` 注解允许根据特定类的是否存在来包含 `@Configuration` 类。由于使用 [ASM](http://asm.ow2.org/) 解析注解元数据，您可以使用 `value` 属性来引用真实类，即使该类实际上可能不会出现在正在运行的应用程序的 classpath 中。如果您希望使用 `String` 值来指定类名，也可以使用 `name` 属性。

此机制不会以相同的方式应用于返回类型是条件的目标的 `@Bean` 方法：在方法上的条件应用之前，JVM 将加载类和可能处理的方法引用，如果找不到类，将发生失败。

要处理这种情况，可以使用单独的 `@Configuration` 类来隔离条件，如下所示：

```java
@Configuration
// Some conditions
public class MyAutoConfiguration {

	// Auto-configured beans

	@Configuration
	@ConditionalOnClass(EmbeddedAcmeService.class)
	static class EmbeddedConfiguration {

		@Bean
		@ConditionalOnMissingBean
		public EmbeddedAcmeService embeddedAcmeService() { ... }

	}

}
```

**提示**

> 如果使用 `@ConditionalOnClass` 或 `@ConditionalOnMissingClass` 作为元注解的一部分来组成自己的组合注解，则必须使用 `name` 来引用类，在这种情况将不作处理。

<a id="boot-features-bean-conditions"></a>

#### 49.3.2、Bean 条件

`@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 注解允许根据特定 bean 是否存在来包含 bean。您可以使用 `value` 属性按类型或使用 `name` 来指定 bean。`search` 属性允许您限制在搜索 bean 时应考虑的 `ApplicationContext` 层次结构。

放置在 `@Bean` 方法上时，目标类型默认为方法的返回类型，如下所示：

```java
@Configuration
public class MyAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public MyService myService() { ... }

}
```

在前面的示例中，如果 `ApplicationContext` 中不包含 `MyService` 类型的 bean，则将创建 `myService` bean。

**提示**

> 您需要非常小心地添加 bean 定义的顺序，因为这些条件是根据到目前为止已处理的内容进行计算的。因此，我们建议在自动配置类上仅使用 `@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 注解（因为这些注解保证在添加所有用户定义的 bean 定义后加载）。

**注意**

> `@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 不会阻止创建 `@Configuration` 类。在类级别使用这些条件并使用注解标记每个包含 `@Bean` 方法的唯一区别是，如果条件不匹配，前者会阻止将 `@Configuration` 类注册为 bean。

<a id="boot-features-property-conditions"></a>

#### 49.3.3、属性条件

`@ConditionalOnProperty` 注解允许基于 Spring Environment 属性包含配置。使用 `prefix` 和 `name` 属性指定需要检查的属性。默认情况下，匹配存在且不等于 `false` 的所有属性。您还可以使用 `havingValue` 和 `matchIfMissing` 属性创建更高级的检查。

<a id="boot-features-resource-conditions"></a>

#### 49.3.4、资源条件

`@ConditionalOnResource` 注解仅允许在存在特定资源时包含配置。可以使用常用的 Spring 约定来指定资源，如下所示：`file:/home/user/test.dat`。

<a id="boot-features-web-application-conditions"></a>

#### 49.3.5、Web 应用程序条件

`@ConditionalOnWebApplication` 和 `@ConditionalOnNotWebApplication` 注解在应用程序为 **Web 应用程序**的情况下是否包含配置。Web 应用程序是使用 Spring `WebApplicationContext`，定义一个 `session` 范围或具有 `StandardServletEnvironment` 的任何应用程序。

<a id="boot-features-spel-conditions"></a>

#### 49.3.6、SpEL 表达式条件

`@ConditionalOnExpression` 注解允许根据 [SpEL 表达式](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#expressions)的结果包含配置。

<a id="boot-features-test-autoconfig"></a>

#### 49.4、测试自动配置

自动配置可能受许多因素的影响：用户配置（`@Bean` 定义和 `Environment` 自定义）、条件评估（存在特定的类库）等。具体而言，每个测试都应该创建一个定义良好的 `ApplicationContext`，它表示这些自定义的组合。`ApplicationContextRunner` 提供了一个好的实现方法。

`ApplicationContextRunner` 通常被定义为测试类的一个字段，用于收集基本的通用配置。以下示例确保始终调用 `UserServiceAutoConfiguration`：

```java
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
		.withConfiguration(AutoConfigurations.of(UserServiceAutoConfiguration.class));
```

**提示**

> 如果必须定义多个自动配置，则无需按照与运行应用程序时完全相同的顺序调用它们的声明。

每个测试都可以使用 runner 来表示特定的用例。例如，下面的示例调用用户配置（`UserConfiguration`）并检查自动配置是否正确退回。调用 `run` 提供了一个可以与 `Assert4J` 一起使用的回调上下文。

```java
@Test
public void defaultServiceBacksOff() {
	this.contextRunner.withUserConfiguration(UserConfiguration.class)
			.run((context) -> {
				assertThat(context).hasSingleBean(UserService.class);
				assertThat(context.getBean(UserService.class)).isSameAs(
						context.getBean(UserConfiguration.class).myUserService());
			});
}

@Configuration
static class UserConfiguration {

	@Bean
	public UserService myUserService() {
		return new UserService("mine");
	}

}
```

也可以轻松自定义 `Environment`，如下所示：

```java
@Test
public void serviceNameCanBeConfigured() {
	this.contextRunner.withPropertyValues("user.name=test123").run((context) -> {
		assertThat(context).hasSingleBean(UserService.class);
		assertThat(context.getBean(UserService.class).getName()).isEqualTo("test123");
	});
}
```

runner 还可用于展示 `ConditionEvaluationReport`。报告可以在 `INFO` 或 `DEBUG` 级别下打印。以下示例展示如何使用 `ConditionEvaluationReportLoggingListener` 在自动配置测试中打印报表。

```java
@Test
public void autoConfigTest {
	ConditionEvaluationReportLoggingListener initializer = new ConditionEvaluationReportLoggingListener(
			LogLevel.INFO);
	ApplicationContextRunner contextRunner = new ApplicationContextRunner()
			.withInitializer(initializer).run((context) -> {
					// Do something...
			});
}
```

<a id="_simulating_a_web_context"></a>

#### 49.4.1、模拟一个 Web 上下文

如果需要测试一个仅在 Servlet 或响应式 Web 应用程序上下文中运行的自动配置，请分别使用 `WebApplicationContextRunner` 或 `ReactiveWebApplicationContextRunner`。

<a id="_overriding_the_classpath"></a>

#### 49.4.2、覆盖 Classpath

还可以测试在运行时不存在特定类和/或包时发生的情况。 Spring Boot附带了一个可以由跑步者轻松使用的FilteredClassLoader。 在以下示例中，我们声明如果UserService不存在，则会正确禁用自动配置：

```java
@Test
public void serviceIsIgnoredIfLibraryIsNotPresent() {
	this.contextRunner.withClassLoader(new FilteredClassLoader(UserService.class))
			.run((context) -> assertThat(context).doesNotHaveBean("userService"));
}
```

<a id="boot-features-custom-starter"></a>

### 49.5、创建自己的 Starter

一个完整的 Spring Boot starter 类库可能包含以下组件：

- `autoconfigure` 模块，包含自动配置代码。
- `starter` 模块，它提供对 `autoconfigure` 模块依赖关系以及类库和常用的其他依赖关系。简而言之，添加 starter 应该提供该库开始使用所需的一切。

**提示**

> 如果您不想将这两个模块分开，则可以将自动配置代码和依赖关系管理组合在一个模块中。

<a id="boot-features-custom-starter-naming"></a>

#### 49.5.1、命名

您应该确保为您的 starter 提供一个合适的命名空间。即使您使用其他 Maven `groupId`，也不要使用 `spring-boot` 作为模块名称的开头。我们可能会为您以后自动配置的内容提供官方支持。

根据经验，您应该在 starter 后命名一个组合模块。例如，假设您正在为 **acme** 创建一个 starter，并且您将自动配置模块命名为 `acme-spring-boot-autoconfigure`，将 starter 命名为 `acme-spring-boot-starter`。如果您只有一个组合这两者的模块，请将其命名为 `acme-spring-boot-starter`。

此外，如果您的 starter 提供配置 key，请为它们使用唯一的命名空间。尤其是，不要将您的 key 包含在 Spring Boot 使用的命名空间中（例如 `server`、`management`、`spring` 等）。如果您使用了相同的命名空间，我们将来可能会以破坏您的模块的方式来修改这些命名空间。

确[保触发元数据生成](#configuration-metadata-annotation-processor)，以便为您的 key 提供 IDE 帮助。您可能想查看生成的元数据（`META-INF/spring-configuration-metadata.json`）以确保您的 key 记录是否正确。

<a id="boot-features-custom-starter-module-autoconfigure"></a>

#### 49.5.2、`autoconfigure` 模块

`autoconfigure` 模块包含类库开始使用所需的所有内容。它还可以包含配置 key 定义（例如 `@ConfigurationProperties`）和任何可用于进一步自定义组件初始化方式的回调接口。

**提示**

> 您应该将类库的依赖项标记为可选，以便您可以更轻松地在项目中包含 `autoconfigure` 模块。如果以这种方式执行，则不提供类库，默认情况下，Spring Boot 将会退出。

Spring Boot 使用注解处理器来收集元数据文件（`META-INF/spring-autoconfigure-metadata.properties`）中自动配置的条件。如果该文件存在，则用于快速过滤不匹配的自动配置，缩短启动时间。建议在包含自动配置的模块中添加以下依赖项：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-autoconfigure-processor</artifactId>
	<optional>true</optional>
</dependency>
```

使用 Gradle 4.5 及更早版本时，应在 `compileOnly` 配置中声明依赖项，如下所示：

```groovy
dependencies {
	compileOnly "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

使用 Gradle 4.6 及更高版本时，应在 `annotationProcessor` 配置中声明依赖项，如下所示：

```groovy
dependencies {
	annotationProcessor "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

<a id="boot-features-custom-starter-module-starter"></a>

#### 49.5.3、Starter 模块

starter 真的是一个空 jar。它的唯一目的是为使用类库提供必要的依赖项。您可以将其视为使用类库的一切基础。

不要对添加 starter 的项目抱有假设想法。如果您自动配置的库经常需要其他 starter，请一并声明它们。如果可选依赖项的数量很多，则提供一组适当的默认依赖项可能很难，因为您本应该避免包含对常用库的使用不必要的依赖项。换而言之，您不应该包含可选的依赖项。

**注意**

> 无论哪种方式，您的 starter 必须直接或间接引用核心 Spring Boot starter（`spring-boot-starter`）（如果您的 starter 依赖于另一个 starter ，则无需添加它）。如果只使用自定义 starter 创建项目，则 Spring Boot 的核心功能将通过存在的核心 starter 来实现。


<a id="boot-features-kotlin"></a>

## 50、Kotlin 支持

[Kotlin](https://kotlinlang.org/) 是一种针对 JVM（和其他平台）的静态类型语言，它可编写出简洁而优雅的代码，同时提供与使用 Java 编写的现有库的[互操作性](https://kotlinlang.org/docs/reference/java-interop.html)。

Spring Boot 通过利用其他 Spring 项目（如 Spring Framework、Spring Data 和 Reactor）的支持来提供 Kotlin 支持。有关更多信息，请参阅 [Spring Framework Kotlin 支持文档](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/languages.html#kotlin)。

开始学习 Spring Boot 和 Kotlin 最简单方法是遵循这个[全面教程](https://spring.io/guides/tutorials/spring-boot-kotlin/)。您可以通过 [start.spring.io](https://start.spring.io/#!language=kotlin) 创建新的 Kotlin 项目。如果您需要支持，请免费加入 [Kotlin Slack](http://slack.kotlinlang.org/) 的 #spring 频道或使用 [Stack Overflow](https://stackoverflow.com/questions/tagged/spring+kotlin) 上的 `spring` 和 `kotlin` 标签提问。

<a id="boot-features-kotlin-requirements"></a>

### 50.1、要求

Spring Boot 支持 Kotlin 1.2.x。要使用 Kotlin，classpath 下必须存在 `org.jetbrains.kotlin:kotlin-stdlib` 和 `org.jetbrains.kotlin:kotlin-reflect`。也可以使用 `kotlin-stdlib` 的变体 `kotlin-stdlib-jdk7` 和 `kotlin-stdlib-jdk8`。

由于 [Kotlin 类默认为 final](https://discuss.kotlinlang.org/t/classes-final-by-default/166)，因此您可能需要配置 [kotlin-spring](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support) 插件以自动打开 `Spring-annotated` 类，以便可以代理它们。

在 Kotlin 中序列化/反序列化 JSON 数据需要使用 [Jackson 的 Kotlin 模块](https://github.com/FasterXML/jackson-module-kotlin)。在 classpath 中找到它时会自动注册。如果 Jackson 和 Kotlin 存在但 Jackson Kotlin 模块不存在，则会记录警告消息。

**提示**

> 如果在 [start.spring.io](https://start.spring.io/#!language=kotlin) 上创建 Kotlin 项目，则默认提供这些依赖项和插件。

<a id="boot-features-kotlin-null-safety"></a>

### 50.2、Null 安全

Kotlin 的一个关键特性是 [null 安全](https://kotlinlang.org/docs/reference/null-safety.html)。它在编译时处理空值，而不是将问题推迟到运行时并遇到 `NullPointerException`。这有助于消除常见的错误来源，而无需支付像 `Optional` 这样的包装器的成本。Kotlin 还允许使用有可空值的，如 [Kotlin null 安全综合指南](http://www.baeldung.com/kotlin-null-safety)中所述。

虽然 Java 不允许在其类型系统中表示 null 安全，但 Spring Framework、Spring Data 和 Reactor 现在通过易于使用的工具的注解提供其 API 的安全性。默认情况下，Kotlin 中使用的 Java API 类型被识别为放宽空检查的[平台类型](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)。[Kotlin 对 JSR 305 注解的支持](https://kotlinlang.org/docs/reference/java-interop.html#jsr-305-support)与可空注解相结合，为 Kotlin 中 Spring API 相关的代码提供了空安全。

可以通过使用以下选项添加 `-Xjsr305` 编译器标志来配置 JSR 305 检查：`-Xjsr305={strict|warn|ignore}`。默认行为与 `-Xjsr305=warn` 相同。在从 Spring API 推断出的 Kotlin 类型中需要考虑 null 安全的 `strict` 值，但是应该使用 Spring API 可空声明甚至可以在次要版本之间发展并且将来可能添加更多检查的方案。

**警告**

> 尚不支持泛型类型参数、`varargs` 和数组元素可空性。有关最新信息，请参见 [SPR-15942](https://jira.spring.io/browse/SPR-15942)。另请注意，Spring Boot 自己的 API [尚未注解](https://github.com/spring-projects/spring-boot/issues/10712)。

<a id="boot-features-kotlin-api"></a>

### 50.3、Kotlin API

<a id="boot-features-kotlin-api-runapplication"></a>

#### 50.3.1、runApplication

Spring Boot 提供了使用 `runApplication<MyApplication>(*args)` 运行应用程序的惯用方法，如下所示：

```kotlin
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
	runApplication<MyApplication>(*args)
}
```

这是 `SpringApplication.run(MyApplication::class.java, *args)` 的替代方式。它还允许自定义应用程序，如下所示：

```kotlin
runApplication<MyApplication>(*args) {
	setBannerMode(OFF)
}
```

<a id="boot-features-kotlin-api-extensions"></a>

#### 50.3.2、扩展

Kotlin [扩展](https://kotlinlang.org/docs/reference/extensions.html)提供了使用附加功能扩展现有类的能力。Spring Boot Kotlin API 利用这些扩展为现有 API 添加新的 Kotlin 特定便利。

提供的 `TestRestTemplate` 扩展类似于 Spring Framework 为 `RestOperations` 提供的。除此之外，扩展使得利用 Kotlin reified 类型参数变为可能。

<a id="boot-features-kotlin-dependency-management"></a>

### 50.4、依赖管理

为了避免在 classpath 上混合不同版本的 Kotlin 依赖项，提供了以下 Kotlin 依赖项的依赖项管理：

- `kotlin-reflect`
- `kotlin-runtime`
- `kotlin-stdlib`
- `kotlin-stdlib-jdk7`
- `kotlin-stdlib-jdk8`
- `kotlin-stdlib-jre7`
- `kotlin-stdlib-jre8`

使用 Maven，可以通过 `kotlin.version` 属性自定义 Kotlin 版本，并为 `kotlin-maven-plugin` 提供插件管理。使用 Gradle，Spring Boot 插件会自动将 `kotlin.version` 与 Kotlin 插件的版本保一致。

<a id="boot-features-kotlin-configuration-properties"></a>

### 50.5、`@ConfigurationProperties`

`@ConfigurationProperties` 目前仅适用于 `lateinit` 或可空的 `var` 属性（建议使用前者），因为[尚不支持](https://github.com/spring-projects/spring-boot/issues/8762)由构造函数初始化的不可变类。

```kotlin
@ConfigurationProperties("example.kotlin")
class KotlinExampleProperties {

	lateinit var name: String

	lateinit var description: String

	val myService = MyService()

	class MyService {

		lateinit var apiToken: String

		lateinit var uri: URI

	}

}
```

**提示**

> 要使用注解处理器生成[您自己的元数据](#configuration-metadata-annotation-processor)，应使用 `spring-boot-configuration-processor` 依赖[配置 `kapt`](https://kotlinlang.org/docs/reference/kapt.html)。

<a id="boot-features-kotlin-testing"></a>

### 50.6、测试

虽然可以使用 JUnit 4（`spring-boot-starter-test` 提供的默认配置）来测试 Kotlin 代码，但建议使用 JUnit 5。JUnit 5 允许测试类实例化一次，并在所有类的测试中复用。这使得可以在非静态方法上使用 `@BeforeAll` 和 `@AfterAll` 注解，这非常适合 Kotlin。

要使用 JUnit 5，请从 `spring-boot-starter-test` 中排除 `junit:junit` 依赖项，然后添加 JUnit 5 依赖项，并相应地配置 Maven 或 Gradle 插件。有关更多详细信息，请参阅 [JUnit 5 文档](https://junit.org/junit5/docs/current/user-guide/#dependency-metadata-junit-jupiter-samples)。您还需要[将测试实例生命周期切换为 **per-class**](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle-changing-default)。

为了模拟 Kotlin 类，建议使用 [Mockk](https://mockk.io/)。如果需要 Mockk 等效的 Mockito 特定的 [`@MockBean` 和 `@SpyBean` 注解](boot-features-testing-spring-boot-applications-mocking-beans)，则可以使用 [SpringMockK](https://github.com/Ninja-Squad/springmockk)，它提供类似的 `@MockkBean` 和 `@SpykBean` 注解。

<a id="boot-features-kotlin-resources"></a>

### 50.7、资源

<a id="boot-features-kotlin-resources-further-reading"></a>

#### 50.7.1、进阶阅读

- [Kotlin 语言参考](https://kotlinlang.org/docs/reference/)
- [Kotlin Slack](http://slack.kotlinlang.org/)（有专用的 #spring 频道）
- [Stackoverflow 上 `spring` 和 `kotlin` 标签](https://stackoverflow.com/questions/tagged/spring+kotlin)
- [在浏览器中尝试 Kotlin](https://try.kotlinlang.org/)
- [Kotlin 博客](https://blog.jetbrains.com/kotlin/)
- [Awesome Kotlin](https://kotlin.link/)
- [教程：使用 Spring Boot 和 Kotlin 构建 Web 应用程序](https://spring.io/guides/tutorials/spring-boot-kotlin/)
- [使用 Kotlin 开发 Spring Boot 应用程序](https://spring.io/blog/2016/02/15/developing-spring-boot-applications-with-kotlin)
- [使用 Kotlin、Spring Boot 和 PostgreSQL 开发地理信使](https://spring.io/blog/2016/03/20/a-geospatial-messenger-with-kotlin-spring-boot-and-postgresql)
- [在 Spring Framework 5.0 中引入 Kotlin 支持](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0)
- [Spring Framework 5 Kotlin API 实现函数式](https://spring.io/blog/2017/08/01/spring-framework-5-kotlin-apis-the-functional-way)

<a id="boot-features-kotlin-resources-examples"></a>

#### 50.7.2、示例

- [spring-boot-kotlin-demo](https://github.com/sdeleuze/spring-boot-kotlin-demo)：常规的 Spring Boot + Spring Data JPA 项目
- [mixit](https://github.com/mixitconf/mixit)：Spring Boot 2 + WebFlux + Reactive Spring Data MongoDB
- [spring-kotlin-fullstack](https://github.com/sdeleuze/spring-kotlin-fullstack)：WebFlux Kotlin 全栈示例，在前端使用 Kotlin2js 代替 JavaScript 和 TypeScript
- [spring-petclinic-kotlin](https://github.com/spring-petclinic/spring-petclinic-kotlin)：Spring PetClinic 示例应用的 Kotlin 版本
- [spring-kotlin-deepdive](https://github.com/sdeleuze/spring-kotlin-deepdive)：将Boot 1.0 + Java 逐步迁移到 Boot 2.0 + Kotlin

<a id="boot-features-whats-next"></a>

## 51、下一步

如果您想了解本节中讨论的任何类目的更多信息，可以查看 [Spring Boot API 文档](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/api)，也可以直接浏览[源代码](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE)。如果您有具体问题，请查看 [how-to](howto.md) 部分。

如果您对 Spring Boot 的核心功能感到满意，可以继续阅读有关生产就绪功能的内容。
