## IoC、DI、AOP 的理解

[IoC、DI、AOP 的理解](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486938&idx=1&sn=c99ef0233f39a5ffc1b98c81e02dfcd4&chksm=cea24211f9d5cb07fa901183ba4d96187820713a72387788408040822ffb2ed575d28e953ce7&token=1666190828&lang=zh_CN#rd)

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

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个 Bean 的实例。
- 如果涉及到一些属性值利用 set 方法进行设置。
- 检查 `Aware `相关接口并进行相应地设置。
  - 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName() 方法，传入 Bean 的名字。
  - 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader() 方法，传入 ClassLoader 对象的实例。
  - 如果 Bean 实现了 BeanFactoryAware 接口，调用 setBeanFactory() 方法，传入BeanFactory对象的实例。
  - 与上面的类似，如果实现了其他 XxxAware 接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行postProcessBeforeInitialization() 方法。
- 如果 Bean 实现了 `InitializingBean` 接口，执行afterPropertiesSet()方法。
- 如果 Bean 在配置文件中的定义包含 `init-method` 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行postProcessAfterInitialization()方法。
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 destroy() 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

![Bean 生命周期](https://gitee.com/raymond-zhao/oss/raw/master/uPic/5496407.jpg)

> [Java Guide Spring Bean 的生命周期](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/SpringBean?id=%e4%ba%8c-bean%e7%9a%84%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f)

## Bean 的创建过程

![Bean 的创建过程](https://gitee.com/raymond-zhao/oss/raw/master/uPic/image-20201009133947220.png)

这里主要是讲的 `doCreateBean()` 方法的主要过程。

1. 如果是单例则需要首先清除缓存
2. 实例化 bean，将 BeanDefinition 转换为 BeanWrapper。
3. 应用 MergedBeanDefinitionProcessor，进行 Bean 合并后的处理。
4. 进行依赖处理(解决可能出现的循环依赖)
5. 进行属性填充，将所有属性填充至 bean 的实例中。
6. 检查循环依赖
7. 注册 DisposableBean
8. 完成创建并返回

## 循环依赖是如何解决的？

- 构造器注入构成的循环依赖**是无法解决的**，只能抛出异常。Spring 解决循环依赖依靠的是 Bean 的“**中间态**”这个概念，而这个中间态指的是*已经实例化*，但*还没初始化*的状态。而构造器的功能就是完成实例化，所以构造器的循环依赖无法解决。

- `prototype` field 属性注入循环依赖**无法解决**。

- field 属性注入 (setter) 循环依赖**可以解决**。

**三级缓存机制**

```java
// 从上至下 分表代表这“三级缓存”
// 一级缓存: 存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
// 二级缓存: 提前曝光的单例对象的缓存，存放原始的 bean 对象(尚未填充属性)，用于解决循环依赖
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
// 三级缓存: 单例对象工厂的缓存，存放 bean 工厂对象，用于解决循环依赖
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

1. 先从**一级缓存**`singletonObjects`中去获取，如果获取到 Bean 就直接返回。
2. 如果获取不到或者对象正在创建中，则从**二级缓存**`earlySingletonObjects`中获取，如果获取到就直接返回。
3. 如果前两级缓存都没成功返回，且允许 `singletonFactories(allowEarlyReference=true)`通过`getObject()`获取。就从**三级缓存**`singletonFactory.getObject()`获取。如果获取到了就从`singletonFactories`中移除，并且放进`earlySingletonObjects`。其实也就是从三级缓存移动到了二级缓存。是剪切、不是复制。

> 为什么要有三级缓存？两级不行吗？

参考：[一文告诉你Spring是如何利用"三级缓存"巧妙解决Bean的循环依赖问题的](https://cloud.tencent.com/developer/article/1497692)

## 哪些场景需要用事务？

- 五种事务隔离级别：`default`、Read Uncommitted、Read Committed、Repeatable Read、Serializable。

- 对数据库的更新操作需要开启事务。

- 如果一次执行多条查询语句，也可以开启事务。

## Spring 事务传播

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于**TransactionDefinition.PROPAGATION_REQUIRED**。

**实现**：基于 `ThreadLocal` ，因此不可跨线程，跨线程需要**透传**。

## Spring 中的事务具体怎么实现的？

1. 如果 Spring 没有指定事务隔离级别，则会采用数据库默认的事务隔离级别；当Spring 指定了事务隔离级别，则会在代码里将事务隔离级别修改为指定值；当数据库不支持 Spring 指定的隔离级别，效果则以数据库的为准（比如采用了MyISAM引擎）。
2. Spring 事务的底层**依赖于数据库的事务**，代码层面上利用 AOP 实现。MySQL 的事务有隔离级别的概念，只有 InnoDB 有事务，实现方式是利用 undo log 和 redo log。
3. 通过 AOP 实现声明式事务。

## Spring MVC 工作流程

![SpringMVC工作流程图](https://c-catnip.github.io/2019/08/19/%E5%90%8E%E7%AB%AF/SpringMVC/SpringMVC_%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B/SpringMVC%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

1. 用户发送请求到 DispatcherServlet
2. DispatcherServlet 收到请求后调用处理器映射器 HandlerMapping
3. 处理器映射器找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet
4. DispatcherServlet 调用 处理器适配器HandlerAdapter
5. 处理器适配器经过适配调用具体的处理器(Controller，也叫后端控制器)
6. Controller执行完成返回ModelAndView
7. HandlerAdapter将Controller执行结果ModelAndView返回给DispatcherServlet
8. DispatcherServlet将ModelAndView传给视图解析器ViewReslover
9. ViewReslover对ModelAndView进行解析后返回具体视图给DispatcherServlet
10. DispatcherServlet根据视图进行渲染，把模型数据填充至request域中
11. 最后DispatcherServlet把处理结果响应给用户

> - [SpringMVC工作流程](https://c-catnip.github.io/2019/08/19/%E5%90%8E%E7%AB%AF/SpringMVC/SpringMVC_%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B/)
> - [从SpringMvc源码分析其工作原理](https://juejin.im/post/6844903826000969742)

## Spring 中的设计模式

1. 工厂设计模式 : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
2. 代理设计模式 : Spring AOP 功能的实现。
3. 单例设计模式 : Spring 中的 Bean 默认都是单例的。
4. 模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
5. 包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
6. 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
7. 适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。

> 推荐阅读：[面试官:“谈谈Spring中都用到了那些设计模式?”](https://juejin.im/post/6844903849849962509)
>
