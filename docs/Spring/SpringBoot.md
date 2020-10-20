## Spring Boot 自动装配原理

1. `spring-boot-dependencies`: 作为父工程，存放了 Spring Boot 的核心依赖。我们在写或者引入一些Spring Boot依赖的时候，不需要指定版本，因为 Spring Boot的父依赖已经帮我们维护了一套版本。
2. `@SpingBootApplication` 注解是一个复合注解，在它的实现中又包含`@SpringBootConfiguration`，`@EnableAutoConfiguration`两个注解。
3. 其中`@SpringBootConfiguration`中又包含`@Configuration`，所以可以理解为`@Configuration=@SpringBootConfiguration`。
4. `@EnableAutoConfiguration`注解，显然就是实现自动配置的注解，其中比较重要的注解是`@AutoConfigurationPackage`，`@Import(AutoConfigurationImportSelector.class)`
   1. @Import(AutoConfigurationImportSelector.class)中的`AutoConfigurationImportSelector`这个类中名为`getCandidateConfigurations()`可以获取所有的候选配置。
   2. 在获取候选配置时会去查找`spring.factories`，里面包含了大量的Spring已经写好的配置类。
   3. 通过 `@ConditionalOnClass` 注解进行判断条件是否成立(只要导入相应的stater，条件就能成立)，如果条件成立则加载配置类，否则不加载该配置类。
   4. 在获取配置类的时候，在上层方法中循环组装为 Properties 供我们使用。

Spring Boot 所有自动配置类都是在启动的时候进行扫描并加载，通过 spring.factories 可以找到自动配置类的路径，但是不是所有存在于 spring.factories 中的配置都进行加载，而是通过 @ConditionalOnClass 注解进行判断条件是否成立（只要导入相应的stater，条件就能成立），如果条件成立则加载配置类，否则不加载该配置类。

- [Spring Boot 自动装配原理初探](https://juejin.im/post/6844904009577267207)

## Spring Boot Starter

## Starter Jar 包与以往的 Jar 包有何不同？

