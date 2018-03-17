<a id="using-boot"></a>

# 三、使用 Spring Boot

本章节将详细介绍如何使用 Spring Boot。它覆盖了诸如构建系统、自动配置和如何运行应用等主题。我们还介绍一些 Spring Boot 最佳实践。虽然 Spring Boot 并没有什么特别（它只是另一个您可以使用的类库），但仍然有一些建议可以让您的开发工作变得更加容易。

如果您是刚开始使用 Spring Boot，那么在深入本部分之前，您应该先阅读[入门部分](#getting-started)。

<a id="using-boot-build-systems"></a>

## 13、构建系统

强烈推荐您选择一个支持[依赖管理](#using-boot-dependency-management)的构建系统, 您可以使用它将 artifact 发布到 Maven Central 仓库。我们建议您选择 Maven 或者 Gradle。虽然可以让 Spring Boot 与其它构建系统（如 Ant）配合工作，但它们不会得到特别好的支持。

<a id="using-boot-dependency-management"></a>

### 13.1、依赖管理

每一次 Spring Boot 发行都提供了一个它所支持的依赖清单。实际上，您不需要为构建配置提供任何依赖的版本，因为 Spring Boot 已经帮您管理这些了。当您升级 Spring Boot 时，这些依赖也将以一致的方式进行升级。

**注意**

> 如果您觉得有必要，您仍然可以指定一个版本并覆盖 Spring Boot 所推荐的。

该清单包含了全部可以与 Spring Boot 一起使用的 spring 模块以及第三方类库，可作为标准[材料清单（`spring-boot-dependencies`）](#using-boot-maven-without-a-parent)，并且可以与 [Maven](#using-boot-maven-parent-pom) 和 [Gradle](#using-boot-gradle) 一起使用。

**警告**

> Spring Boot 的每一次发行都会基于一个 Spring Framework 版本，因此我们**强烈**建议您不要指定指定它的版本。

<a id="using-boot-maven"></a>

### 13.2、Maven

Maven 用户可以继承 `spring-boot-starter-parent` 项目以获取合适的默认值，父项目提供了以下功能：

- Java 1.8 作为默认编译器级别。
- 源代码使用 UTF-8 编码。
- [依赖管理部分](#using-boot-dependency-management)继承自 `spring-boot-dependencies` 的 POM，允许您省略常见依赖的 `<version>` 标签。
- 合理的[资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)。
- 合适的插件配置（[exec plugin](http://www.mojohaus.org/exec-maven-plugin/)、[Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin)、[shade](https://maven.apache.org/plugins/maven-shade-plugin/)）。
- 针对 `application.properties` 和 `application.yml` 资源的合理过滤，包括特定 profile 的文件（例如 `application-foo.properties` 和 `application-foo.yml`）

注意：由于 `application.properties` 和 `application.yml` 文件接受 Spring 风格的占位符（`${​...}`），因此 Maven 过滤改为使用 `@..@` 占位符（您可以使用 Maven 的 `resource.delimiter` 属性重写它）

<a id="using-boot-maven-parent-pom"></a>

#### 13.2.1、继承 Starter Parent

要将项目配置继承 `spring-boot-starter-parent`，只需要按以下方式设置 `parent`：

```xml
<!-- 从 Spring Boot 继承默认配置 -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.0.RELEASE</version>
</parent>
```

**注意**

> 您只需要在此依赖上指定 Spring Boot 的版本号。如果您要导入其它 starter，则可以放心地省略版本号。

通过该设置，您还可以重写自己项目中的配置属性来覆盖个别依赖。例如，要升级到另一个 Spring Data 发行版本，您需要将以下内容添加到 `pom.xml` 文件中。

```xml
<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

**提示**

> 查看 [`spring-boot-dependencies` pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml) 以获取受支持的属性清单。

<a id="using-boot-maven-without-a-parent"></a>

#### 13.2.2、不使用父 POM

不是每个人都喜欢从 `spring-boot-starter-parent` 继承 POM。您可能需要使用自己公司标准的父 POM，或者您可能只是希望明确地声明所有 Maven 配置。

如果您不想使用 `spring-boot-starter-parent`，则仍然可以通过使用 `scope=import` 依赖来获得依赖管理（但不是插件管理）的好处：

```xml
<dependencyManagement>
		<dependencies>
		<dependency>
			<!-- 从 Spring Boot 导入依赖管理 -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.0.0.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
如上所述，上述示例设置不会让您使用属性来重写个别依赖。要达到相同的目的，需要在 `spring-boot-dependencies` 项**之前**在项目的 `dependencyManagement` 中添加一项。例如，要升级到另一个 Spring Data 发行版，您可以将以下元素添加到 `pom.xml`中：

```xml
<dependencyManagement>
	<dependencies>
		<!-- 重写 Spring Boot 提供的 Spring Data -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-releasetrain</artifactId>
			<version>Fowler-SR2</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.0.0.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

**注意**

> 以上示例中，我们指定了一个 **BOM**，但是任何的依赖类型都可以用这个方法来重写。

<a id="using-boot-maven-plugin"></a>

#### 13.2.3、使用 Spring Boot Maven 插件

Spring Boot 包括了一个 [Maven 插件](#build-tool-plugins-maven-plugin)，它可以将项目打包成一个可执行 jar。如果要使用它，请将插件添加到您的 `<plugins>` 中：

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

> 如果您使用了 Spring Boot starter 的父 pom，则只需要添加插件。除非您要修改父级中定义的设置，否则不需要进行配置。

<a id="using-boot-gradle"></a>

### 13.3、Gradle

要了解如何使用 Spring Boot 和 Gradle，请参阅 Spring Boot 的 Gradle 插件文档：

- 参考文档（[HTML](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/gradle-plugin/reference/html) 与 [PDF](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)）
- [API](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/gradle-plugin/api)

<a id="using-boot-ant"></a>

### 13.4、Ant

可以使用 Apache Ant+Ivy 构建 Spring Boot 项目。`spring-boot-antlib` AntLib 模块也可以帮助 Ant 创建可执行 jar 文件。

要声明依赖，可参考以下一个典型的 `ivy.xml` 文件内容：

```xml
<ivy-module version="2.0">
	<info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
	<configurations>
		<conf name="compile" description="everything needed to compile this module" />
		<conf name="runtime" extends="compile" description="everything needed to run this module" />
	</configurations>
	<dependencies>
		<dependency org="org.springframework.boot" name="spring-boot-starter"
			rev="${spring-boot.version}" conf="compile" />
	</dependencies>
</ivy-module>
```

一个典型的 `build.xml` 大概是这样：

```xml
<project
	xmlns:ivy="antlib:org.apache.ivy.ant"
	xmlns:spring-boot="antlib:org.springframework.boot.ant"
	name="myapp" default="build">

	<property name="spring-boot.version" value="2.0.0.RELEASE" />

	<target name="resolve" description="--> retrieve dependencies with ivy">
		<ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
	</target>

	<target name="classpaths" depends="resolve">
		<path id="compile.classpath">
			<fileset dir="lib/compile" includes="*.jar" />
		</path>
	</target>

	<target name="init" depends="classpaths">
		<mkdir dir="build/classes" />
	</target>

	<target name="compile" depends="init" description="compile">
		<javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
	</target>

	<target name="build" depends="compile">
		<spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
			<spring-boot:lib>
				<fileset dir="lib/runtime" />
			</spring-boot:lib>
		</spring-boot:exejar>
	</target>
</project>
```

**提示**

> 如果您不想使用 `spring-boot-antlib` 模块，请参阅第 84.10 节：[**使用 Ant 构建可执行归档文件**](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#howto-build-an-executable-archive-with-ant)，无需使用 `spring-boot-antlib`。

<a id="using-boot-starter"></a>

### 13.5、Starter

Starter 是一组惯例依赖描述资源，可以包含在应用中。从 starter 中，您可以获得所需的所有 Spring 和相关技术的一站式支持，无须通过示例代码和复制粘贴来获取依赖。比如，如果您要使用 Spring 和 JPA 进行数据库访问，那么只需要在项目中包含 `spring-boot-starter-data-jpa` 依赖项即可。

starter 包含了许多您需要用于使项目快速启动和运行，并且需要一组受支持的可传递依赖关系的依赖。

---

**名字的含义**

官方的所有 starter 都遵循类似的命名规则：`spring-boot-starter-*`，其中 `*` 是特定类型的应用。这个命名结构旨在帮助您找到 starter。许多 IDE 中 Maven 集成允许您按名称搜索依赖。例如，安装了 Eclipse 或者 STS 插件后，您可以简单地在 POM 编辑器中按下 `ctrl-space` 并输入 **spring-boot-starter** 来获取完整的列表。

正如[**创建自己的 starter**](#boot-features-custom-starter) 章节所述，第三方的 starter 命名不应该以 `spring-boot` 开头，因为它是官方 Spring Boot 构件所保留的规则。例如，有一个第三方 starter 项目叫做 `thirdpartyproject`，它通常会命名为 `thirdpartyproject-spring-boot-starter`。

---

Spring Boot 在 `org.springframework.boot` group 下提供了以下应用 starter：

**表 13.1、Spring Boot 应用类 Starter**

| 名称 | 描述 | POM |
|:---|:---|:---:|
| `spring-boot-starter` | 核心 starter，包含自动配置支持、日志和 YAML | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq` | 提供 JMS 消息支持，使用 Apache ActiveMQ | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp` | 提供 Spring AMQP 与 Rabbit MQ 支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop` | 提供 Spring AOP 与 AspectJ 面向切面编程支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |
| `spring-boot-starter-artemis` | 提供 JMS 消息服务支持，使用 Apache Artemis | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
| `spring-boot-starter-batch` | 提供 Spring Batch 支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |
| `spring-boot-starter-cache` | 提供 Spring Framework 缓存支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| `spring-boot-starter-cloud-connectors` | 使用 Spring Cloud Connectors 简单连接到类似 Cloud Foundry 和 Heroku 等云平台 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
| `spring-boot-starter-data-cassandra` | 提供对 Cassandra 分布式数据库和 Spring Data Cassandra 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
| `spring-boot-starter-data-cassandra-reactive` | 提供对 Cassandra 分布式数据库和 Spring Data Cassandra Reactive 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| `spring-boot-starter-data-couchbase` | 提供对 Couchbase 面向文档数据库和 Spring Data Couchbase 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
| `spring-boot-starter-data-couchbase-reactive` | 提供对 Couchbase 面向文档数据库和 Spring Data Couchbase Reactive 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
| `spring-boot-starter-data-elasticsearch` | 提供对 Elasticseach 搜索与分析引擎和 Spring Data Elasticsearch 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| `spring-boot-starter-data-jpa` | 提供 Spring Data JPA 与 Hibernate 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-data-ldap` | 提供对 Spring Data LDAP 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
| `spring-boot-starter-data-mongodb` | 提供对 MongoDB 面向文档数据库和 Spring Data MongoDB 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| `spring-boot-starter-data-mongodb-reactive` | 提供对 MongoDB 面向文档数据库和 Spring Data MongoDB Reactive 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) | 
| `spring-boot-starter-data-neo4j` | 提供对 Neo4j 图数据库与 SPring Data Neo4j 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
| `spring-boot-starter-data-redis` | 提供对 Redis 键值数据存储、Spring Data Redis 和 Lettuce 客户端的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| `spring-boot-starter-data-redis-reactive` | 提供对 Redis 键值数据存储、Spring Data Redis Reactive 和 Lettuce 客户端的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
| `spring-boot-starter-data-rest` | 提供使用 Spring Data REST 通过 REST 暴露 Spring Data 资源库的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
| `spring-boot-starter-data-solr` | 提供对 Apache Solr 搜索平台与 Spring Data Solr 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |
| `spring-boot-starter-freemarker` | 提供使用 Freemakrer 视图构建 MVC web 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
| `spring-boot-starter-groovy-templates` | 提供使用 Groovy 模板视图构建 MVC web 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
| `spring-boot-starter-hateoas` | 提供使用 Spring MVC 与Spring HATEOAS 构建基于超媒体的 RESTful web 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) | 
| `spring-boot-starter-integration` | 提供对 Spring Integration 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
| `spring-boot-starter-jdbc` | 提供 JDBC 与 Tomcat JDBC 连接池的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
| `spring-boot-starter-jersey` | 提供对使用 JAX-RS 与 Jersey 构建 RESTful web 应用的支持。[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-web) 的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |
| `spring-boot-starter-jooq` | 提供对使用 JOOQ 访问 SQL 数据库的支持。[`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-data-jpa) 或 [`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-jdbc) 的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |
| `spring-boot-starter-json` | 提供了读写 json 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |
| `spring-boot-starter-jta-atomikos` | 提供 Atomikos JTA 事务支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| `spring-boot-starter-jta-bitronix` | 提供 Bitronix JTA 事务支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |
| `spring-boot-starter-jta-narayana` | 提供 Narayana JTA 支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-narayana/pom.xml) |
| `spring-boot-starter-mail` | 提供使用　Java Mail 与 Spring Framework 的邮件发送支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| `spring-boot-starter-mustache` | 提供使用 Mustache 视图构建 web 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
| `spring-boot-starter-quartz` | Quartz 支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| `spring-boot-starter-security` | Spring Security 支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-test` | 提供包含了 JUnit、Hamcrest 与 Mockito 类库的 Spring Boot 单元测试支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf` | 提供使用 Thymeleaf 视图构建 MVC web 应用的支持 | [pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation` | 提供 Hibernate Validator 与 Java Bean Validation 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web` | 提供使用 Spring MVC 构建 web（包含 RESTful）应用的支持，使用 Tomcat 作为默认嵌入式容器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-web-services` | Spring Web Services 支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
| `spring-boot-starter-webflux` | 提供使用 Spring Framework 的 Reactive Web 支持构建 WebFlux 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
| `spring-boot-starter-websocket` | 提供使用 Spring Framework 的 WebSocket 支持构建 WebSocket 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |

除了应用 starter，以下 starter 可用于添加[生产就绪](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#production-ready)特性：

**表 13.2、Spring Boot 生产类 starter**

| 名称 | 描述 | POM |
|:---|:---|:---:|
| `spring-boot-starter-actuator` | Spring Boot 的 Actuator 支持，其提供了生产就绪功能，帮助您监控和管理应用 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) | 

最后，Spring Boot 还包含以下 starter，如果您想要排除或切换特定技术，可以使用以下 starter：

**表 13.3、Spring Boot 技术类 starter**

| 名称 | 描述 | POM |
|:---|:---|:---:|
| `spring-boot-starter-jetty` | 使用 Jetty 作为嵌入式 servlet 容器。可代替 [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |
| `spring-boot-starter-log4j2` | 使用 Log4j2 作为日志组件。可代替 [`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-logging) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |
| `spring-boot-starter-logging` | 使用 Logback 作为日志组件，此 starter 为默认的日志 starter | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
| `spring-boot-starter-reactor-netty` | 使用 Reactor Netty 作为内嵌响应式 HTTP 服务器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| `spring-boot-starter-tomcat` | 使用 Tomcat 作为嵌入式 servlet 容器，此为 [`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-web) 默认的 servlet 容器 starter | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |
| `spring-boot-starter-undertow` | 使用 Undertow 作为嵌入式 servlet 容器，可代替 [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |

**提示**

> 有关其它社区贡献的 starter 列表，请参阅 GitHub 上的 `spring-boot-starters` 模块中的 [README 文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)。

<a id="using-boot-structuring-your-code"></a>

## 14、组织代码

Spring Boot 不需要任何特定的代码布局，但是有一些最佳实践是很有用的。

<a id="using-boot-using-the-default-package"></a>

### 14.1、使用 default 包

当一个类没有 `package` 声明时，它就被认为是在 **default** 包中。通常不鼓励使用 **default 包**，应该避免使用。对于使用 `@ComponentScan`、`@EntityScan` 或者 `@SpringBootApplication` 注解的 Spring Boot 应用，这样可能会导致特殊问题发生，因为每一个 jar 中的每一个类将会被读取到。

**提示**

> 我们建议您使用 Java 推荐的包命名约定，并使用域名的反向形式命名（例如  `com.example.project`）。

<a id="using-boot-locating-the-main-class"></a>

### 14.2、定位主应用类

我们通常建议您将主应用类放在其它类之上的根包中， `@EnableAutoConfiguration` 注解通常放在主类上，它隐式定义了某些项目的 **包搜索**的基准起点。例如，如果您在编写一个 JPA 应用程序，则被 `@EnableAutoConfiguration` 注解的类所属的包将被用于搜索标记有 `@Entity` 注解的类。

使用根包还可以允许使用没有指定 `basePackage` 属性的 `@ComponentScan` 注解。如果您的主类在根包中，也可以使用 `@SpringBootApplication` 注解。

以下是一个经典的包结构：

```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`Application.java` 文件声明了 `main` 方法，附带了 `@Configuration` 注解。

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

<a id="using-boot-configuration-classes"></a>

## 15、配置类

Spring Boot 支持基于 Java 的配置。虽然可以在 `SpringApplication` 中使用 XML 配置源，但我们通常建议主配置源为 `@Configuration` 类。通常，一个很好的选择是将定义了 `main` 方法的类作为 `@Configuration`。

**提示**

> 许多 Spring 的 XML 配置示例已经在 Internet 上发布了。如果可能的话，您无论如何都应该尝试着使用等效的基于 Java 的配置方式，搜索 `Enable*` 注解可以帮到您不少忙。

<a id="using-boot-importing-configuration"></a>

### 15.1、导入额外的配置类

你不需要把所有的 `@Configuration` 放在一个类中。`@Import` 注解可用于导入其他配置类。或者，您可以使用 `@ComponentScan` 自动扫描所有 Spring 组件，包括 `@Configuration` 类。

<a id="using-boot-importing-xml-configuration"></a>

### 15.2、导入 XML 配置

如果您一定要使用基于 XML 的配置，我们建议您仍然使用 `@Configuration` 类。您可以使用 `@ImportResource` 注解来加载 XML 配置文件。

<a id="https://docs.spusing-boot-auto-configuration"></a>

## 16、自动配置

Spring Boot 自动配置尝试根据您添加的 jar 依赖自动配置 Spring 应用。例如，如果 classpath 下存在 HSQLDB，并且您没有手动配置任何数据库连接 bean，那么 Spring Boot 将自动配置一个内存数据库。

您需要通过将 `@EnableAutoConfiguration` 或者 `@SpringBootApplication` 注解添加到其中一个 `@Configuration` 类之上以启用自动配置。

**提示**

您应该仅添加一个 `@EnableAutoConfiguration` 注解。我们通常建议您将其添加到主 `@Configuration` 类中。

<a id="using-boot-replacing-auto-configuration"></a>

### 16.1、平滑替换自动配置

自动配置是非入侵的，您可以随时定义自己的配置来代替自动配置的特定部分。例如，如果您添加了自己的 `DataSource` bean，默认的嵌入式数据库支持将不会自动配置。

如果您需要了解当前正在应用的自动配置，以及为什么使用，请使用 `--debug` 开关启动应用。这样做可以为核心 logger 启用调试日志，并记录到控制台。

<a id="using-boot-disabling-specific-auto-configuration"></a>

### 16.2、禁用指定的自动配置类

如果您发现在正在使用不需要的自动配置类，可以通过使用 `@EnableAutoConfiguration` 的 `exclude` 属性来禁用它们。

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果类不在 classpath 下，您可以使用注解的 `excludeName` 属性并指定完全类名。最后，您还可以通过 `spring.autoconfigure.exclude` property 控制要排除的自动配置类列表。

**提示**

> 您可以同时使用注解和 property 定义排除项

<a id="using-boot-spring-beans-and-dependency-injection"></a>

## 17、Spring Bean 与依赖注入

您可以自由使用任何标准的 Spring Framework 技术来定义您的 bean 以及它们注入的依赖。我们发现使用 `@ComponentScan` 来寻找 bean 和结合 `@Autowired` 构造器注入可以很好地工作。

如果您按照上述的建议（将应用类放在根包中）来组织代码，则可以添加无参的 `@ComponentScan`。所有应用组件（`@Component`、`@Service`、`@Repository`、`@Controller` 等）将自动注册为 Spring Bean。

以下是一个 `@Service` Bean，其使用构造注入方式获取一个必需的 `RiskAssessor` bean。

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	@Autowired
	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```

如果 bean 中只有一个构造方法，您可以忽略掉 `@Autowired` 注解。

```java
@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```


**提示**

> 请注意，构造注入允许 `riskAssessor` 字段被修饰为 `final`，这表示以后它不能被更改。

<a id="using-boot-using-springbootapplication-annotation"></a>

## 18、使用 @SpringBootApplication 注解

很多 Spring Boot 开发者总是使用 `@Configuration`、`@EnableAutoConfiguration` 和 `@ComponentScan` 注解标记在主类上。由于 这些注解经常一起使用（特别是如果您遵循上述的[最佳实践](#using-boot-structuring-your-code)）。Spring Boot 提供了一个更方便的 `@SpringBootApplication` 注解可用来替代这个组合。

`@SpringBootApplication` 注解相当于使用 `@Configuration`、`@EnableAutoConfiguration` 和 `@ComponentScan` 及他们的默认属性：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 相当于使用 @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
 
}
```

**注意**

> `@SpringBootApplication` 还提供了别名来自定义 `@EnableAutoConfiguration` 和 `@ComponentScan` 的属性。

<a id="using-boot-running-your-application"></a>

## 19、运行您的应用

将应用程序打包成 `jar` 可执行文件并使用嵌入式 HTTP 服务器的最大有点之一就是可以按照您想使用的其它方式来运行应用。调试 Spring Boot 也是很简单，您不需要任何特殊的 IDE 插件或者扩展。

**注意**

本章节仅涵盖基于　jar　的打包方式，如果您选择将应用打包为　war　文件，则应该参考您的服务器和　IDE　文档。

<a id=""></a>

### 19.1、使用 IDE 运行

您可以使用 IDE 运行 Spring Boot应用，就像运行一个简单的 Java 应用程序一样，但是首先您需要导入项目，导入步骤取决于您的 IDE 和构建系统。大多数 IDE 可以直接导入 Maven 项目，例如 Eclipse 用户可以从 `File` 菜单中选择 `Import` ​ → `Existing Maven Projects`。

如果您无法将项目直接导入到 IDE 中，则可以使用构建插件生成 IDE 元数据（metadata）。Maven 包含了 [Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/) 和 [IDEA](https://maven.apache.org/plugins/maven-idea-plugin/) 的插件,Gradle 也为[各种 IDE](https://docs.gradle.org/4.2.1/userguide/userguide.html) 提供了插件。

**提示**

> 如果您不小心运行了两次 web 应用，您将看到一个 **Port already in use** （端口已经被使用）错误。STS 用户可以使用 `Relaunch` 按钮运行以确保现有的任何实例都已关闭，而不是使用 Run 按钮。

<a id="using-boot-running-as-a-packaged-application"></a>

### 19.2、作为打包应用运行

如果您使用 Spring Boot Maven 或者 Gradle 插件创建可执行 jar，可以使用 `java -jar` 命令运行应用。例如：

```bash
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

也可以在运行打包应用程序时开启远程调试支持。该功能允许您将调试器附加到打包的应用中。

```bash
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

<a id="using-boot-running-with-the-maven-plugin"></a>

### 19.3、使用 Maven 插件

Spring Boot Maven 插件包含一个可用于快速编译和运行应用程序的 `run` goal。应用程序以快速形式运行，就像在 IDE 中一样。以下示例展示了运行 Spring Boot 应用程序的典型 Maven 命令：

```bash
$ mvn spring-boot:run
```

您可能还想使用 `MAVEN_OPTS` 操作系统环境变量，如下例所示：

```bash
$ export MAVEN_OPTS=-Xmx1024m
```

<a id="using-boot-running-with-the-gradle-plugin"></a>

### 19.4、使用 Gradle 插件

Spring Boot Gradle 插件包含一个 `bootRun` 任务，可用于以快速形式运行应用程序。每当应用 `org.springframework.boot` 和 java 插件时都会添加 `bootRun` 任务：

```bash
$ gradle bootRun
```

您可能还想使用 `JAVA_OPTS` 操作系统环境变量：

```bash
$ export JAVA_OPTS=-Xmx1024m
```

<a id="using-boot-hot-swapping"></a>

### 19.5、热交换

由于 Spring Boot 应用程序只是普通的 Java 应用程序，因此 JVM 热插拔是可以开箱即用。JVM 热插拔在可替换字节码方面有所限制。想要更完整的解决方案，可以使用 [JRebel](https://zeroturnaround.com/software/jrebel/)。

`spring-boot-devtools` 模块包含了对快速重新启动应用程序的支持。有关详细信息，请参阅本章后面的[第 20 章：开发人员工具](#using-boot-devtools)部分以及[热插拔的 How-to 部分](#howto-hotswapping)。

**待续……**