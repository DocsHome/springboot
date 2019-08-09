<div align="center">
    <img src="https://spring.io/img/homepage/icon-spring-boot.svg" alt="Spring Boot">
    <h1>Spring Boot 中文文档</h1>
</div> 

使用 Spring Boot，您只需极少的配置就能轻松构建出一个独立、基于 Spring 的生产级应用程序。

该项目为 Spring Boot 文档翻译项目，由本人个人发起，利用业余时间基于 [Spring Boot 2.1.5](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle) 的官方文档进行翻译。

> 由于翻译工作启动时是参照 1.5.x 文档进行翻译，现在转向 2.1.5 文档，前期的内容可能比较旧，之后会慢慢更新内容。翻译顺序不会按照官网文档目录顺序，优先翻译常用的技术内容。

## 阅读方式

[Github](https://github.com/DocsHome/springboot/blob/master/SUMMARY.md) | [Gitbook](https://www.gitbook.com/book/docshome/springboot)

## 安装

本项目使用 [GitBook](https://www.gitbook.com) 生成文档，如果您想在本地运行项目，请先安装 GitBook。

安装 GitBook 命令行工具（请确保您已经安装了 [Node.js](https://nodejs.org) 和 npm）。

```bash
npm install gitbook-cli -g
```

进入项目根目录，执行以下命令安装 gitbook 依赖：

```bash
gitbook install
```

启动本地服务器在线浏览

```bash
gitbook serve
```

生成 HTML，生成的文件位于 `_book` 目录下。

```
gitbook build
```

更多文档操作，请参照 GitBook 命令。

## 项目状态

> 由于是个人翻译，因此文档翻译的时限不定，根据个人业余时间调整。

| 章节 | 进度 |
| --- | --- |
| [一、Spring Boot 文档](pages/boot-documentation.md#boot-documentation) | 100% |
| [二、入门](pages/getting-started.md) | 100% |
| [三、使用 Spring Boot](pages/using-spring-boot.md) | 100% |
| [四、Spring Boot 特性](pages/spring-boot-features.md#boot-features) | 98% (缺少章节： **45、测试**)|
| [五、Spring Boot Actuator: 生产就绪功能](pages/production-ready.md#production-ready) | 100% |
| [六、部署 Spring Boot 应用程序](deployment.md) | 1% |
| 七、Spring Boot CLI | 0% |
| 八、构建工具插件 | 0% |
| 九、How-to 指南 | 0% |
| 十、附录 | 0% |

## License

![](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png)

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。
