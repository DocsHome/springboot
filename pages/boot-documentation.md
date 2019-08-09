**作者**

- Phillip Webb
- Dave Syer
- Josh Long
- Stéphane Nicoll
- Rob Winch
- Andy Wilkinson
- Marcel Overdijk
- Christian Dupuis
- Sébastien Deleuze
- Michael Simons
- Vedran Pavić
- Jay Bryant
- Madhura Bhave

**译者**

- [Oopsguy](https://github.com/oopsguy)

---

**2.1.5.RELEASE**（前半部分为 1.5.9.RELEASE 的内容，之后会更新）

Copyright © 2012-2018

---

在不对副本收取任何费用且每个副本都包含版权声明的情况下，您可以将本文档的副本分发给他人，无论是印刷形式还是电子发行形式。

<a id="boot-documentation"></a>
# I、Spring Boot 文档

本节将简要介绍 Spring Boot 参考文档，可以看作是本文档内容的概述。您可以以线性方式阅读此参考指南，如果您不感兴趣，可以跳过该部分。

<a id="boot-documentation-about"></a>
## 1、关于文档

Spring Boot 参考指南提供了 [html](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/html)、[pdf](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/pdf/spring-boot-reference.pdf) 和 [epub](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/epub/spring-boot-reference.epub) 格式的文档。最新的副本可在[docs.spring.io/spring-boot/docs/current/reference](https://docs.spring.io/spring-boot/docs/current/reference) 获取。

在不对副本收取任何费用且每个副本都包含版权声明的情况下，您可以将本文档的副本分发给他人，无论是印刷形式还是电子发行形式。

<a id="boot-documentation-getting-help"></a>

## 2、获取帮助

如果您在使用 Spring Boot 时遇到了麻烦，可参考以下指南。

- 尝试 [How-to](howto.md)  —— 最常见问题的解决方案都在这里。
- 学习 Spring 基础  ——  Spring Boot 是建立在多个 Spring 项目之上, 请查看 [spring.io](https://spring.io/) 网站以获取更多参考文档。如果您是刚刚开始使用 Spring, 请尝试其中一个[指南](https://spring.io/guides)。
- 提问问题 —— 我们时刻关注着 [stackoverflow.com](https://stackoverflow.com/) 上有关 `spring-boot` 标签相关的问题。
- 在 [github.com/spring-projects/spring-boot/issues](https://github.com/spring-projects/spring-boot/issues) 报告 Spring Boot 的 bug。

> Spring Boot 是全部开源的，包括文档！如果您发现文档中存在错误了，或者您想改进它们，请[参与我们](https://github.com/spring-projects/spring-boot/tree/v2.1.1.RELEASE)。

<a id="boot-documentation-first-steps"></a>
## 3、起步

如果您是刚开始使用 Spring Boot，或者对 Spring 大体有个印象, [您可以从这里开始学习](page/getting-started.md)!

- **从零开始**： [概述](getting-started.md#getting-started-introducing-spring-boot) | [要求](getting-started.md#getting-started-system-requirements) | [安装](getting-started.md#getting-started-installing-spring-boot)
- **教程**： [第 1 部分](getting-started.md) | [第 2 部分](getting-started.md#getting-started-first-application-code)
- **运行您的例子**： 第 1 部分 | 第 2 部分

<a id="_working_with_spring_boot"></a>
## 4、使用 Spring Boot

准备开始使用 Spring Boot 了? [立即上手](using-spring-boot.md)。

- **构建系统**：Maven | Gradle | Ant | Starter
- **最佳实践**：代码结构 | @Configuration | @EnableAutoConfiguration | Bean 与依赖注入
- **运行代码**：IDE | 打包 | Maven | Gradle
- **打包应用**：生产环境下的 jar
- **Spring Boot CLI**：使用 CLI

<a id="_learning_about_spring_boot_features"></a>
## 5、了解 Spring Boot 新特性

需要更多关于 Spring Boot 核心特性？[Spring 特性](boot-features.md)!

- **核心特性**：SpringApplication | 外部配置 | Profile | 日志
- **Web 应用程序**：MVC | 嵌入式容器
- **使用数据**：SQL | NO-SQL
- **消息传递**：概述 | JMS
- **测试**：概述 | Boot Applications | 实用工具
- **延伸**：Auto-configuration | @Conditions

<a id="_moving_to_production"></a>
## 6、生产环境

您如果准备将 Spring Boot 应用推送至生产环境，或许会对以下内容感兴趣。

- **管理端点**：概述 | 自定义
- **连接方式**：HTTP | JMX | SSH
- **监控**：度量 | 审计 | 追踪 | 流程

<a id="_advanced_topics"></a>

## 7、高级内容

最后，我们为高级用户提供了几个主题。

- **部署 Spring Boot 应用**：云部署 | OS 服务
- **构建工具插件**：Maven | Gradle
- **附录**：Application Properties | Auto-configuration 类 | 可执行 Jar