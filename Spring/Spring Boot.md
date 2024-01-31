# Spring Boot

## 外部化配置

### 外部配置的生效顺序

这里只关注配置文件 applicaition.properties 的覆盖顺序：

- 在jar中打包的Application properties（application.properties 和 YAML）。
- 在jar中打包的 特定的 Profile application properties（application-{profile}.properties 和 YAML）。
- 在打包的jar之外的Application properties性（application.properties和YAML）。
- 在打包的jar之外的特定的 Profile application properties（ application-{profile}.properties 和YAML）。
  
对于同级别的配置文件 properties 的优先级高于 yaml。

## 测试

大多数开发者使用 `spring-boot-starter-test` “Starter”，它同时导入Spring Boot测试模块以及JUnit Jupiter、AssertJ、Hamcrest和其他一些有用的库。

### Spring MVC 测试的自动配置

要测试Spring MVC控制器是否按预期工作，可以使用 `@WebMvcTest` 注解。`@WebMvcTest` 自动配置Spring MVC基础设施，并将扫描的Bean限制在 `@Controller`、`@ControllerAdvice`、`@JsonComponent`、`Converter`、`GenericConverter`、`Filter`、`HandlerInterceptor`、`WebMvcConfigurer`、`WebMvcRegistrations` 以及 `HandlerMethodArgumentResolver`。当使用 `@WebMvcTest` 注解时，常规的 `@Component` 和 `@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties` 可以用来包括 `@ConfigurationProperties` Bean。

通常情况下，`@WebMvcTest` 仅限于一个controller，并与 `@MockBean` 结合使用，为需要的合作者提供mock实现。

`@WebMvcTest` 也自动配置了 MockMvc。 Mock MVC提供了一种强大的方式来快速测试MVC controller，而不需要启动一个完整的HTTP服务器。

[MockMVC 单元测试具体实现](../test/单元测试.md#对-driving-的接口进行单元测试)
