<a id="getting-started"></a>
# 二、入门

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

> 虽然 Spring Boot 支持 Java 1.6，但是如果可以，您应该考虑使用最新的 Java 版本。

<a id="getting-started-installation-instructions-for-java"></a>
### 10.1、针对 Java 开发人员的安装说明

您可以跟使用任何标准 Java 库的方式一样使用 Spring Boot。只需要在 classpath 下包含相应的 `spring-boot-*.jar` 文件即可。Spring Boot 不需要任何专用的工具来集成，因此您可以使用任何 IDE 或者文本编辑器，并且 Spring Boot 应用也没什么特殊之处，因此可以像任何其它 Java 程序一样运行和调试。

虽然您可以复制 Spring Boot 的 jar 文件，但我们通常建议您使用支持依赖管理的构建工具（比如 Maven 或者 Gradle）。

<a id="getting-started-maven-installation"></a>
### 10.1.1、使用 Maven 安装

Spring Boot 兼容 Apache Maven 3.2 或更高版本。如果您还没有安装 Maven，可以到 [maven.apache.org](https://maven.apache.org/) 上按照说明进行操作。

**提示**

> 在许多操作系统上，可以通过软件包管理器来安装 Maven。如果您是 OSX Homebrew 用户，请尝试使用 `brew install maven`。Ubuntu 用户可以运行 `sudo apt-get install maven`。

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

> `spring-boot-starter-parent` 是一个使用 Spring Boot 的好方式，但它并不是任何时候都适用。有时您可能需要继承不同的父 POM，或者您不喜欢我们的默认配置。请参见[第 13.2.2 节, “使用不带父 POM 的 Spring Boot”](#using-boot-maven-without-a-parent) 作为的替代方案，其使用了 `import` Scope。

<a id="getting-started-gradle-installation"></a>
### 10.1.2、使用 Gradle 安装

Spring Boot 兼容 Gradle 2 (2.9 或者更高版本)和 Gradle 3。如果您还没有安装 Gradle，您可以按照 [www.gradle.org/](http://www.gradle.org/) 上的说明进行操作。

Spring Boot 依赖 `org.springframework.boot` `group`。通常，您的项目将声明一个或者多个 [Starter](using-boot-starter.md) 的依赖。Spring Boot 提供了一个有用的 [Gradle 插件](#build-tool-plugins-gradle-plugin)，可用于简化依赖声明和创建可执行 jar 文件。

**Gradle Wrapper**

> 当您许需要构建项目时，Gradle Wrapper 提供了一个用于获取 Gradle 的好方法。它是由小脚本和库组成，您在提交的同时，您的代码将引导构建流程。更多详细信息，请参阅 [docs.gradle.org/2.14.1/userguide/gradle_wrapper.html](https://docs.gradle.org/2.14.1/userguide/gradle_wrapper.html)。

这是一个典型的 `build.gradle` 文件：

```groovy
plugins {
    id 'org.springframework.boot' version '1.5.4.RELEASE'
    id 'java'
}


jar {
    baseName = 'myproject'
    version =  '0.0.1-SNAPSHOT'
}

repositories {
    jcenter()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

<a id="getting-started-installing-the-cli"></a>
### 10.2、安装 Spring Boot CLI

Spring Boot CLI 是一个命令行工具，如果您想使用 Spring 快速搭建原型，可以选择它。它允许您运行 [Groovy](http://groovy.codehaus.org/) 脚本，这意味着您有可以有类 Java 语法且没有太多样板的代码。

您不需要使用 CLI 来配合 Spring Boot，但它确实是一个入门 Spring 应用的最快方式。

<a id="getting-started-manual-cli-installation"></a>
#### 10.2.1、手动安装

您可以从 Spring 软件仓库中下载 Spring CLI 发行版：

- [spring-boot-cli-1.5.9.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/1.5.9.RELEASE/spring-boot-cli-1.5.9.RELEASE-bin.zip)
- [spring-boot-cli-1.5.9.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/1.5.9.RELEASE/spring-boot-cli-1.5.9.RELEASE-bin.tar.gz)

[最新的快照发行版](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)也是可用的。

下载之后，请按照解压缩归档文件中的 [INSTALL.txt](https://raw.github.com/spring-projects/spring-boot/v1.5.9.RELEASE/spring-boot-cli/src/main/content/INSTALL.txt) 说明进行操作。总之：在 `.zip` 文件的 `bin/` 目录中有一个 spring 脚本（在 Windows 下为 `spring.bat`），或者也可以使用 `java -jar` 配合 `.jar` 文件（该脚本可以帮助您确保 classpath 设置正确）。

<a id="getting-started-sdkman-cli-installation"></a>
#### 10.2.2、使用 SDKMAN! 安装

SDKMAN!（软件开发包管理器）用于管理二进制 SDK 的多个版本，包括 Groovy 和 Spring Boot CLI。从 [sdkman.io](http://sdkman.io/) 获取 SDKMAN! 并安装 Spring Boot：

```bash
$ sdk install springboot
$ spring --version
Spring Boot v1.5.9.RELEASE
```

如果您正在为 CLI 开发功能，并希望够能轻松地访问刚创建的版本，请参照以下指令。

```bash
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-1.5.9.RELEASE-bin/spring-1.5.9.RELEASE/
$ sdk default springboot dev
$ spring --version
Spring CLI v1.5.9.RELEASE
```

以上操作将会安装一个名为 `dev` 的 `spring` 的本地实例。它指向您的目标构建位置，因此每次重新构建 Spring Boot 时，spring 都是最新的。

您可以这样做来相关信息：

```bash
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 1.5.9.RELEASE

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```

<a id="getting-started-homebrew-cli-installation"></a>
#### 10.2.3、使用 OSX Homebrew 安装

如果您是在 Mac 上工作并且使用了 [Homebrew](http://brew.sh/)，您安装 Spring Boot CLI 需要做的:

```bash
$ brew tap pivotal/tap
$ brew install springboot
```

Homebrew 将会把 spring 安装在 `/usr/local/bin`。

**注意**
> 如果您没有看到执行流程, 您安装的 brew 可能已经过期了。执行 `brew update` 并重新尝试。

<a id="getting-started-macports-cli-installation"></a>
#### 10.2.4、使用 MacPorts 安装

如果您是在 Mac 上工作并且使用了 [MacPorts](https://www.macports.org/)，您安装 Spring Boot CLI 所需要做的:

```bash
$ sudo port install spring-boot-cli
```

<a id="getting-started-cli-command-line-completion"></a>
#### 10.2.5、命令行完成

Spring Boot CLI 为 [BASH](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29) 和 [zsh](https://en.wikipedia.org/wiki/Zsh) 提供了命令完成脚本。您可以在任何 shell 中执行此脚本 (也称为 `spring`），或将其放在您个人或系统范围的 bash 中完成初始化。在 Debian 系统上，系统范围的脚本位于 `/shell-completion/bash` 中，当新的 shell 启动时，该目录中的所有脚本将被执行。要手动运行脚本, 例如：您已经使用 SDKMAN! 安装了

```bash
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

**注意**
> 如果您使用 Homebrew 或者 MacPorts 安装了 Spring Boot CLI，则命令行完成脚本将自动注册到您的 shell 中。

<a id="getting-started-cli-example"></a>
### 10.2.6、快速入门 Spring CLI 示例

这是一个非常简单的 web 应用程序，可以用于测试您的安装情况。创建一个名为 `app.groovy` 的文件：

```groovy
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

之后在 shell 中运行它：

```bash
$ spring run app.groovy
```
**注意**
> 第一次运行应用的时候需要一些时间，因为需要下载依赖。后续运行将会更快。

在您喜欢的浏览器中打开 localhost:8080，您应该会看到以下输出：

```
Hello World!
```

<a id="getting-started-upgrading-from-an-earlier-version"></a>
## 10.3、升级旧版 Spring Boot

如果您想从旧版的 Spring Boot 升级到此版本，请查看[项目 wiki](https://github.com/spring-projects/spring-boot/wiki) 上托管的 **release notes（发行说明）**。您将会找到升级说明以及每个版本的**新特性和注意事项**列表。

要升级现有的 CLI，请使用相应的包管理器命令（例如 `brew upgrade`） 或者, 如果您手动安装了 CLI，请按照[标准说明](#getting-started-manual-cli-installation)，记得更新您的 `PATH` 环境变量以删除任何旧的引用。

<a id="getting-started-first-application"></a>
## 11、开发第一个 Spring Boot 应用

让我们使用 Java 开发一个简单的 **Hello World!** web 应用程序，以便体现 Spring Boot 的一些关键特性。我们将使用 Maven 构建该项目，因为大多数 IDE 都支持它。

**提示**

> [spring.io](https://spring.io/) 网站上有许多使用 Spring Boot 的**入门指南**，如果您正在寻找具体问题的解决方案，可先从上面寻找。您可以到 [start.spring.io](https://start.spring.io/) 使用依赖搜索功能选择 `web` starter 来快速完成以下步骤。它将自动生成一个新的项目结构，以便您可以[立即开始编码](#getting-started-first-application-code)。[查看文档了解更多信息](https://github.com/spring-io/initializr)。

在开始之前，打开终端检查您是否安装了符合要求的 Java 版本和 Maven 版本。

```bash
$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
```

```bash
$ mvn -v
Apache Maven 3.2.3 (33f8c3e1027c3ddde99d3cdebad2656a31e8fdf4; 2014-08-11T13:58:10-07:00)
Maven home: /Users/user/tools/apache-maven-3.1.1
Java version: 1.7.0_51, vendor: Oracle Corporation
```

**提示**

> 此示例需要在您自己的文件夹中创建，后续的步骤说明假设您已经创建了这个文件夹，它是您的**当前目录**。

<a id="getting-started-first-application-pom"></a>
### 11.1 创建 POM

我们先要创建一个 Maven `pom.xml` 文件。`pom.xml` 是用于构建项目的**配方**。打开您最喜欢的编辑器并添加一下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <!-- Additional lines to be added here... -->

</project>
```

这应该会给您生成一个工作版本，您可以通过运行 `mvn package` 来测试它（此时您可以忽略 **jar will be empty - no content was marked for inclusion!** 警告信息)。

**注意**

> 此时，您可以将项目导入 IDE（大部分的现代 Java IDE 都内置 Maven 支持）。为了简单起见，我们将继续在这个例子中使用纯文本编辑器。

<a id="getting-started-first-application-dependencies"></a>

### 11.2、添加 Classpath 依赖

Spring Boot 提供了多个 “Starter”，可以让您方便地将 jar 添加到 classpath 下。我们的示例应用已经在 POM 的 `parent` 部分使用了 `spring-boot-starter-parent`。`spring-boot-starter-parent` 是一个特殊 Starter，它提供了有用的 Maven 默认配置。此外它还提供了[依赖管理](#using-boot-dependency-management)功能，您可以忽略这些依赖的版本（`version`）标签。

其他 Starter 只提供在开发特定应用时可能需要到的依赖。由于我们正在开发一个 web 应用，因此我们将添加一个 `spring-boot-starter-web` 依赖 ， 但在此之前，让我们来看看目前拥有的。

```
vn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

`mvn dependency:tree` 命令以树的形式打印项目的依赖。您可以看到 `spring-boot-starter-parent` 本身不提供依赖。我们可以在 `parent` 下方立即编辑 `pom.xml` 并添加 `spring-boot-starter-web` 依赖：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```

如果您再次运行 `mvn dependency:tree`，将会看到现在有许多附加的依赖，包括了 Tomcat web 服务器和 Spring Boot 本身。

<a id="getting-started-first-application-code"></a>

### 11.3、编码

要完成我们的应用，我们需要创建一个 Java 文件。默认情况下，Maven 将从 `src/main/java` 目录下编译源代码，因此您需要创建该文件夹结构，之后添加一个名为 `src/main/java/Example.java` 的文件：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```

虽然没有多少代码，但它仍然做了很多事情。让我们看看里面重要的部分。

<a id="getting-started-first-application-annotations"></a>

#### 11.3.1、@RestController 与 @RequestMapping 注解

`Example` 类中的第一个注解是 `@RestController`，该注解被称作 stereotype 注解。它能为代码阅读者提供一些提示，对于 Spring 而言，这个类具有特殊作用。在本示例中，我们的类是一个 web `@Controller`，因此 Spring 在处理传入的 web 请求时会考虑它。

`@RequestMapping` 注解提供了 **routing**（路由）信息。它告诉 Spring，任何具有路径为 `/` 的 HTTP 请求都应映射到 `home` 方法。`@RestController` 注解告知 Spring 渲染结果字符串直接返回给调用者。

**提示**

> `@RestController` 和 `@RequestMapping` 是 Spring MVC 注解（它们不是 Spring Boot 特有的）。有关更多详细信息，请参阅 Spring 参考文档中的 [MVC 章节](https://docs.spring.io/spring/docs/5.0.4.RELEASE/spring-framework-reference/web.html#mvc)

<a id="getting-started-first-application-auto-configuration"></a>

#### 11.3.2、@EnableAutoConfiguration 注解

第二个类级别注解是 `@EnableAutoConfiguration`。此注解告知 Spring Boot 根据您添加的 jar 依赖来“猜测”您想如何配置 Spring 并进行自动配置，由于 `spring-boot-starter-web` 添加了 Tomcat 和 Spring MVC，auto-configuration（自动配置）将假定您要开发 web 应用并相应设置了 Spring。

> **Starter 与自动配置** <br/> Auto-configuration 被设计与 Starter 配合使用，但这两个概念并不是直接相关的。您可以自由选择 starter 之外的 jar 依赖，Spring Boot 仍然会自动配置您的应用程序。

<a id="getting-started-first-application-main-method"></a>

#### 11.3.3、main 方法

应用的最后一部分是 `main` 方法。这只是一个标准方法，其遵循 Java 规范中定义的应用程序入口点。我们的 main 方法通过调用 `run` 来委托 Spring Boot 的 `SpringApplication` 类，`SpringApplication` 类将引导我们的应用，启动 Spring，然后启动自动配置的 Tomcat web 服务器。我们需要将 `Example.class`  作为一个参数传递给 `run` 方法来告知 `SpringApplication`，它是 Spring 主组件。同时还传递 `args` 数组以暴露所有命令行参数。

<a id="getting-started-first-application-run"></a>

### 11.4、运行示例

此时，我们的应用应该是可以工作了。由于您使用了 `spring-boot-starter-parent` POM，因此您可以使用 `run` 来启动应用程序。在根目录下输入 `mvn spring-boot:run` 以启动应用：

```
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.0.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果您用浏览器打开了 [`localhost:8080`](http://localhost:8080/)，您应该会看到以下输出：

```
Hello World!
```

要平滑退出程序，请按 `ctrl+c`。

<a id="getting-started-first-application-executable-jar"></a>

### 11.5、创建可执行 jar

我们通过创建一个完全自包含（self-contained）的可执行 jar 文件完成了示例。该 jar 文件可以在生产环境中运行。可执行 jar（有时又称为 `fat jars`）是包含了编译后的类以及代码运行时所需要相关的 jar 依赖的归档文件。

---

**可执行 jar 与 Java**

Java 不提供任何标准方式来加载嵌套的 jar 文件（比如本身包含在 jar 中的 jar 文件）。如果您想分发自包含的应用，这可能是个问题。

为了解决此问题，许多开发人员使用了 **uber** jar，uber jar 从所有应用的依赖中打包所有的类到一个归档中。这种方法的问题在于，您很难看出应用程序实际上使用到了哪些库。如果在多个 jar 中使用了相同的文件名（但内容不同），这也可能产生问题。

Spring Boot 采用了[不同方方式](#executable-jar)，可以直接对 jar 进行嵌套。 

---

要创建可执行 jar，我们需要将 `spring-boot-maven-plugin` 添加到 `pom.xml` 文件中。在 `dependencies` 下方插入以下配置：

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

**注意**

> `spring-boot-starter-parent` POM 包含了 `<executions>` 配置，用于绑定 `repackage` 。如果您没有使用父 POM，您需要自己声明此配置。有关详细的信息，请参阅[插件文档](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/maven-plugin/usage.html)。

保存 `pom.xml` 并在命令行中运行 `mvn package`：

```
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.0.0.RELEASE:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

如果您浏览 `target` 目录，您应该会看到 `myproject-0.0.1-SNAPSHOT.jar`。该文件的大小大约为 10 MB。如果您想要查看里面的内容，可以使用 `jar tvf`：

```bash
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

您应该还会在 `target` 目录中看到一个名为 `myproject-0.0.1-SNAPSHOT.jar.original` 的较小文件。这是在 Spring Boot 重新打包之前由 Maven 所创建的原始 jar 文件。

使用 `java -jar` 命令运行该应用：

```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.0.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```

跟之前一样, 要平滑退出应用，请按 `ctrl-c`。

<a id="getting-started-whats-next"></a>

## 12、下一步

希望您在本章节学到了一些 Spring Boot 的基础知识，并且开始编写自己的应用。如果您是一名面向任务（task-oriented）的开发人员，您可能想跳过 [spring.io](https://spring.io/) 而直接阅读一些能解决**如何使用 Spring 来解决？** 这类问题的[入门指南](https://spring.io/guides/)，此外我们还有 Spring Boot 专门的 [How-to](#howto) 参考文档。

[Spring Boot 仓库](https://github.com/spring-projects/spring-boot)还有很多您可以运行的[示例](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-samples)。示例与其余部分的代码是独立的（也就是说，您不需要构建其他的代码来运行或使用示例）。

接下来阅读的是第三部：[使用 Spring Boot](#using-boot)。如果您真的感到厌倦了，可以跳过该部分直接阅读 [Spring Boot 特性](#boot-features)。