<a id="boot-documentation"></a>
# II、入门

如果您是刚开始使用 Spring Boot，或者对 Spring 有点印象，那么这部分内容是为您准备的！在这里我们将给出基本的“是什么？”、“怎么做？”、“为什么？”这类问题的答案。这是一份友好的 Spring Boot 简介和安装说明。当我们在讨论一些核心原理之后，我们将构建第一个 Spring Boot 应用。

<a id="getting-started-introducing-spring-boot"></a>
## 8、Spring Boot 简介

使用 Spring Boot 可以很容易地创建出能直接运行的独立的、生产级别的基于 Spring 的应用。我们对 Spring 平台和第三方类库有自己的考虑，因此您可以从最基本的开始。大多数 Spring Boot 应用只需要很少的 Spring 配置。

您可以使用 Spring Boot 来创建一个可以使用 `java -jar` 命令来运行或者基于传统的 war 包部署的应用程序。我们还提供了一个用于运行 spring scripts 的命令行工具。

我们的主要目标是：

- 为所有 Spring Boot 开发提供一个更快、更全面的入门体验。
- 坚持自我虽好，但当需求出现偏离，您需要能迅速摆脱出来。
- 提供大量非功能性特性相关项目（例如：内嵌服务器、安全、指标、健康检查、外部配置)。
- 绝对没有代码生成，也不要求 XML 配置。

<a id="getting-started-system-requirements"></a>
## 9、系统要求

默认情况下，Spring Boot 1.5.4.RELEASE 需要 [Java 7](https://www.java.com/) 和 Spring Framework 4.3.9.RELEASE 或者更高版本。您可以通过其他配置使 Java 6 配合 Spring Boot。更多详细信息，请参见[第 84.11节，“如何使用 Java 6”](howto.md#howto-use-java-6)。Spring Boot 为 Maven（3.2+），和 Gradle 2（2.9 或者更高版本）和 Gradle 3 提供了显式构建支持。

**提示**
> 虽然您可以在 Java 6 或者 Java 7 上使用 Spring Boot，但我们还是强烈推荐您使用 Java 8。

<a id="_servlet_containers"></a>
### 9.1、Servlet 容器

支持以下嵌入式容器开箱即用：

| 名称 | Servlet 版本 | Java 版本 |
|:------|:------|:-----|
| Tomcat 8 | 3.1 | Java 7+ |
| Tomcat 7 | 3.0 | Java 6+ |
| Jetty 9.3 | 3.1 | Java 8+ |
| Jetty 9.2 | 3.1 | Java 7+ |
| Jetty 8 | 3.0 | Java 6+ |
| Undertow 1.3 | 3.1 | Java 7+ |

您可以将 Spring Boot 应用部署到任何一个 Servlet 3.0+ 兼容容器中。

<a id="getting-started-installing-spring-boot"></a>
## 10、安装 Spring Boot

Spring Boot 可以与**经典**的 Java 开发工具一起使用或者作为命令行工具安装。无论如何，您将需要 [Java SDK 1.6](https://www.java.com/) 或者更高版本。在开始之前您应该检查 Java 的安装情况：

```bash
$ java -version
```

如果您是 Java 开发新手，或者您只想尝试使用 Spring Boot，您可能需要首先尝试使用 [Spring Boot CLI](#getting-started-installing-the-cli)，否则请阅读经典的安装说明。

**提示**

虽然 Spring Boot 支持 Java 1.6，但是如果可以，您应该考虑使用最新的 Java 版本。

<a id="getting-started-installation-instructions-for-java"></a>
### 10.1、针对 Java 开发人员的安装说明

您可以跟使用任何标准 Java 库的方式一样使用 Spring Boot。只需要在 classpath 下包含相应的 `spring-boot-*.jar` 文件即可。Spring Boot 不需要任何专用的工具来集成，因此您可以使用任何 IDE 或者文本编辑器，并且 Spring Boot 应用也没什么特殊之处，因此可以像任何其它 Java 程序一样运行和调试。

虽然您可以复制 Spring Boot 的 jar 文件，但我们通常建议您使用支持依赖管理的构建工具（比如 Maven 或者 Gradle）。

<a id="getting-started-maven-installation"></a>
### 10.1.1、使用 Maven 安装

Spring Boot 兼容 Apache Maven 3.2 或更高版本。如果您还没有安装 Maven，可以到 [maven.apache.org](https://maven.apache.org/) 上按照说明进行操作。

**提示**

在许多操作系统上，可以通过软件包管理器来安装 Maven。如果您是 OSX Homebrew 用户，请尝试使用 `brew install maven`。Ubuntu 用户可以运行 `sudo apt-get install maven`。

Spring Boot 依赖使用到了 `org.springframework.boot` `groupId`。通常，您的 Maven POM 文件将从 `spring-boot-starter-parent` 项目继承，并声明一个或多个 [Starter](using-boot-starter.md) 依赖。Spring Boot 还提供了一个可选的 [Maven 插件](#build-tool-plugins-maven-plugin)来创建可执行 jar。

这是一个典型的 `pom.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**提示**

`spring-boot-starter-parent` 是一个使用 Spring Boot 的好方式，但它并不是任何时候都适用。有时您可能需要继承不同的父 POM，或者您不喜欢我们的默认配置。请参见[第 13.2.2 节, “使用不带父 POM 的 Spring Boot”](#using-boot-maven-without-a-parent) 作为的替代方案，其使用了 `import` Scope。


