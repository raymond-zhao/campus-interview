## IoC、DI、AOP 的理解

## Bean 作用域

Spring 框架支持以下五个作用域，如果你使用 web-aware ApplicationContext 时，其中三个是可用的。

| 作用域         | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| singleton      | 该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)。 |
| prototype      | 该作用域将单一 bean 的定义限制在任意数量的对象实例。         |
| request        | 该作用域将 bean 的定义限制为 HTTP 请求。只在 web-aware Spring ApplicationContext 的上下文中有效。 |
| session        | 该作用域将 bean 的定义限制为 HTTP 会话。 只在web-aware Spring ApplicationContext的上下文中有效。 |
| global-session | 该作用域将 bean 的定义限制为全局 HTTP 会话。只在 web-aware Spring ApplicationContext 的上下文中有效。 |

## Bean 生命周期

- [Java Guide Spring Bean 的生命周期](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/SpringBean?id=%e4%ba%8c-bean%e7%9a%84%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f)

## 循环依赖

## Spring 事务

## Spring Boot 自动配置原理

## Spring MVC 工作流程

## AOP 的底层实现

## 字节码增强技术



## 