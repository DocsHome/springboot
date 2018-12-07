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
| `${spring-boot.formatted-version}` | 你使用的 Spring Boot 版本格式化之后显示（用括号括起来，以 `v` 为前缀）。例如 (`v2.1.1.RELEASE`)。 |
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

**待续……**