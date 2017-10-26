---
title: Sonar代码质量分析使用
date: 2017-10-17 18:47:09
tags:
---
## Sonar概述
Sonar是一个用于代码质量管理的开放平台。通过插件机制，Sonar 可以集成不同的测试工具，代码分析工具，以及持续集成工具。

与持续集成工具（例如 Hudson/Jenkins 等）不同，Sonar 并不是简单地把不同的代码检查工具结果（例如 FindBugs，PMD 等）直接显示在 Web 页面上，而是通过不同的插件对这些结果进行再加工处理，通过量化的方式度量代码质量的变化，从而可以方便地对不同规模和种类的工程进行代码质量管理。

![ffmpeg](http://wx3.sinaimg.cn/mw690/78d85414ly1fknfudniaej21kw0uan49.jpg)
<!-- more -->

在对其他工具的支持方面，Sonar 不仅提供了对 IDE 的支持，可以在 Eclipse 和 IntelliJ IDEA 这些工具里联机查看结果；同时 Sonar 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用 Sonar。
此外，Sonar的插件还可以对 Java 以外的其他编程语言提供支持，对国际化以及报告文档化也有良好的支持。

## Sonar安装
本文主要介绍 Sonar 的使用方法，直接到[Sonar官网](https://www.sonarqube.org)下载最近的发型包即可，本文使用的为最新的版本为6.5(推荐使用最新版)，其源代码可以参考[github地址](https://github.com/SonarSource/sonarqube)。

下载zip包后，直接解压，然后根据应用服务器环境启动 bin 目录下的脚本即可


```
bin/linux-x86-64/sonar.sh -h          // 显示所有命令
bin/linux-x86-64/sonar.sh start       // 启动，默认为9000端口
```

然后在浏览器中访问 http://localhost:9000 即可, 初始化用户名和密码为: admin/admin

## Sonar数据库配置
Sonar 默认使用的是 Derby 数据库，但这个数据库一般用于评估版本或者测试用途。商用及对数据库要求较高时，建议使用其他数据库。Sonar 可以支持大多数主流关系型数据库（例如 Microsoft SQL Server, MySQL, Oracle, PostgreSQL 等，本文以 MySQL 为例说明如何更改 Sonar 的数据库设置:

```bash,monokai
mysql> CREATE USER sonar IDENTIFIED BY 'sonar';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar' WITH GRANT OPTION;
```
配置好数据库权限后，修改 sonar.properties 文件配置如下(数据库用户名密码为：sonar/sonar)
![sonar_config](img/sonar_manual/sonar_manual_1.png)
配置后重新启动sonar即可，此次因为需要创建数据库，重启较慢，重启成功后会在数据库中生成sonar相关的表。

## 使用Sonar进行代码质量管理
由于本人主要使用 Java 作为开发工具，主要介绍对 Java 代码代码质量管理，sonar默认是不需要登录权限认证就可以上传代码监测报告的，在生产环境中需要打开用户权限，在[配置]->[通用配置]->[权限]中打开即可，如下图所示
![sonar_auth](img/sonar_manual/sonar_manual_2.png)
### Maven集成Sonar
Maven 插件会自动把所需数据（如单元测试结果、静态检测结果等）上传到 Sonar 服务器上，需要说明的是，关于 Sonar 的配置并不在每个工程的 pom.xml 文件里，而是在 Maven 的配置文件 settings.xml 文件里，涉及到以下 maven 配置项目:

| 配置项 | 作用 | 默认值 |
|--------|---------|-------|
| sonar.host.url | sonar服务器地址文件 | http://127.0.0.1:9000|
| sonar.login | sonar用户名 | 用户或者token(如果利用token则不用密码，推荐这种方式登陆) |
| sonar.password | sonar密码 | admin |
#### sonar生成登陆token
为了强化安全，避免直接暴露出分析用户的密码，使用用户令牌来代替用户登陆,如下图
![sonar_auth](img/sonar_manual/sonar_manual_3.png)

#### Maven配置文件修改
具体配置如下:

```bash
<profile>
        <id>sonar</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <sonar.host.url>http://localhost:9000</sonar.host.url>
            <sonar.login>93a87b9d138cd836b65c2c52fc5578fc71270707</sonar.login>
        </properties>
    </profile>
```
编译命令如下

```bash
mvn clean install
mvn sonar:sonar
```
将 Soanr 所需要的数据上传到 Sonar 服务器上之后，Sonar 安装的插件会对这些数据进行分析和处理，并以各种方式显示给用户，从而使用户方便地对代码质量的监测和管理。
