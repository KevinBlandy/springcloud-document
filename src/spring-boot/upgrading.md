# 升级SpringBoot应用

项目[wiki](https://github.com/spring-projects/spring-boot/wiki)上提供了如何从早期版本的Spring Boot升级的说明。按照[发行说明](https://github.com/spring-projects/spring-boot/wiki#release-notes)部分的链接，找到你要升级的版本。

升级说明总是在发行说明中的第一项。如果你落后一个以上的版本，请确保你也查看你跳过的版本的发行说明。

## 1. 从1.x升级

如果你要从Spring Boot的`1.x`版本升级，请查看项目wiki上的["迁移指南"](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)，它提供了详细的升级说明。还可以查看["发行说明"](https://github.com/spring-projects/spring-boot/wiki)，了解每个版本的 "新的和值得注意的"功能列表。

## 2. 升级到新的功能版本

在升级到新的功能版本时，一些属性可能已经被重新命名或删除。Spring Boot提供了一种方法，可以在启动时分析你的应用程序的环境并打印诊断结果，但也可以在运行时为你临时迁移属性。要启用该功能，请在你的项目中添加以下依赖关系。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-properties-migrator</artifactId>
    <scope>runtime</scope>
</dependency>
```

> 在环境中添加得较晚的属性，例如使用@PropertySource时，将不会被考虑在内。

> 一旦你完成了迁移，请确保将这个模块从你的项目的依赖关系中删除。

## 3. 升级Spring Boot CLI

要升级现有的CLI安装，使用适当的软件包管理器命令（例如，`brew upgrade`）。如果你手动安装了CLI，请遵循[标准说明](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.installing.cli.manual-installation)，记得更新你的`PATH`环境变量，删除任何旧的引用。
