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

```console
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

```console
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

```console
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

```console
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

Java 面向对象查询（[Java Object Oriented Querying，jOOQ](http://www.jooq.org/)）是一款广受欢迎的产品，出自 [Data Geekery](http://www.datageekery.com/)，它可以通过数据库生成 Java 代码，并允许您使用流畅 API 来构建类型安全的 SQL 查询。商业版和开源版都可以与 Spring Boot 一起使用。

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

jOOQ 提供的流畅 API 是通过 `org.jooq.DSLContext` 接口初始化的。Spring Boot 将自动配置一个 `DSLContext` 作为 Spring Bean，并且将其连接到应用程序的 `DataSource`。`要使用 DSLContext`，您只需要 `@Autowire` 它：

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


**待续……**