# IoC 的前世今生

## Matrin Fowler 的文章

### 问题的出现与思考

如何在面向接口编程中，使应用能够自动选择/注入/实例化所依赖接口的实现类？如 `Collection` 实现类众多，如何在不了解具体实现细节的前提下保持使用者与被使用者之间的正确适配？如果定义了 `List`，具体实现是应该选择 `ArrayList` 还是 `LinkedList` 还是 `AutoPopulatingList` 还是 `ManagedList` 还是其他实现？

- 核心问题：如何在程序设计中，将通过接口定义的插件组合成应用程序？

- 解决手段：无一例外地全是**控制反转**

- 争论焦点：只谈控制反转，不谈**反转了什么**没有实际意义。

- 反转的内容：如何定位插件的具体实现，如何消除应用程序对插件的具体实现的依赖，如何在运行期动态**加载/注入**新的具体实现？
- 反转的实现：
    - Service Locator：应用程序代码直接向服务定位器发送一个消息，明确要求服务的实现；
    - Dependency Injection：应用程序代码不发出显式的请求，服务的实现自然会出现在应用程序的代码中，**此谓”控制反转“**。
- 一切的关键：**配置的服务与使用应该分开**，也是一个基本的设计原则（分离接口与实现）。

Spring 并不是首先实现 IoC 的先驱者，实际上，在早期由应用程序向 UI 界面的转变也是控制反转，**控制权由应用程序转移到 UI 框架**。

### 依赖注入

Martin 在与多位 IoC 爱好者讨论后，决定将**控制反转**这个模式称为**依赖注入（Dependency Injection）**。

> 依赖注入模式的基本思想：用一个单独的对象/装配器来获得 SomeInterfaceOrPlugin 的**一个合适的实现**，然后将这个实例赋值给 SomeInterfaceOrPlugin 的一个字段。

依赖注入的三种方式：

- 构造器注入（Constructor Injection）
- 设置注入（Setter Injection）
- 接口注入（Interface Injection）

选择**构造器注入**还是**设值注入**可以折射出面向对象编程中的更一般性问题：**应该在何时何地填充对象属性？**

构造器初始化的好处：

- 在构造阶段就创建出完整的、合法的、有血有肉的对象；
- 可以隐藏任何不可变的字段/属性；
- 通过仅允许构造器实例化，可以强制开发者区分必要的属性。

构造器初始化的坏处：

- 如果参数太多，构造器就会显得凌乱不堪。
- 但是，如果参数太多，也恰恰说明了这个对象的设计可能并不合理，因为这表示一个对象承担了太多的职责，违反了**单一职责原则**，需要进行拆分设计。
- 构造器之间只能通过参数的格式与类型加以区分，增加了使用时的理解与选择成本。

## 对 IoC/DI 的理解

> 词源：目前来看，首先提出 IoC 的并不是 Martin Fowler，而是 Johnson 与 Foote 在 1988 年发表的论文《[Designing Reusable Classes](http://www.laputan.org/drc/drc.html)》，之后在社区内流传，后来被 GoF 写进《设计模式》。

在软件工程中，IoC 是被广泛应用于面向对象编程中的一种设计模式，有时也被称为 DI（依赖注入），DI 是实现 IoC 的一种方式。

在面向对象编程（`OOP`）中，通常是一个类中使用了许多其他对象作为其成员变量，来完成相应的功能，而为了使用某个对象的功能，首先要**初始化/设置**这个对象，通常需要使用 `Setter`执行 `setItem(Item item)` 或使用 `Constructor` 进行 `new Object(Item item)` 操作。

> 使用多参构造器暗示了这个对象做了不止一件事，违反了**单一指责原则**。

而在使用 `Item` 之前，首先还需要完成 `Item` 的初始化，同样的，在初始化 `Item` 时，可能 `Item` 也有很多其他**依赖（Dependencies）**需要初始化。

因此为了执行一个 `setItem(Item item)` 动作，可能要手动执行很多其他依赖的设置，为了解决这种复杂的**依赖注入与控制**，在 Java 中可以把这项工作**外包**给 `Spring`，以后使用某个对象时，可以对 `Spring` 说：“Hey spring，我现在需要一个 `Item` 的对象，我不关心你怎么构建，不关心你什么时候构建，不关心你花了多久构建，总之给我一个 `Item` 对象就好。”

简而言之，由 **大头兵** 向 **M** 转变（🐶 保命），只管需要时**索取**，不再手动**贡献**，不再关心具体过程，而只关心产出与结果。

- [Inversion of Control Containers and the Dependency Injection pattern - Martin Fowler](https://martinfowler.com/articles/injection.html)
- [InversionOfControl - Martin Fowler](https://martinfowler.com/bliki/InversionOfControl.html)

# Spring IoC

在 Spring 中，接口 `ApplicationContext` 代表 IoC 容器，负责对象的**初始化、配置、组装**，生产出的对象被称为 `Bean`，并管理其生命周期。

- [The IoC container - Spring Framework Docs](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)
- [Intro to Inversion of Control and Dependency Injection with Spring - Baeldung](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

## IoC 容器的核心工作流程

Spring IoC（控制反转）容器的核心工作流程如下：

1. 加载配置文件：Spring IoC 容器通过读取配置文件（通常是 XML 文件）来了解如何创建和配置各种 Bean。
2. 实例化 Bean：Spring IoC 容器根据配置文件中的定义创建 Bean 的实例。
3. 注入依赖：Spring IoC 容器会自动为 Bean 注入依赖关系，这意味着它会查找并注入所需的其他 Bean，以满足 Bean 的依赖关系。
4. 处理 BeanPostProcessor：Spring IoC 容器会处理 BeanPostProcessor 接口的实现类，这些类可以在 Bean 实例化和依赖注入之后进行一些自定义的初始化操作。
5. 装配 Bean：Spring IoC 容器会将各个 Bean 装配到相应的位置，这通常涉及将 Bean 放入合适的作用域中（如单例或原型），并将它们暴露给其他 Bean。
6. Bean 的使用：Spring IoC 容器可以将 Bean 注入到其他对象中，或者通过应用程序代码显式地请求访问 Bean。

总的来说，Spring IoC 容器的核心工作流程是将 Bean 的创建、装配和管理从应用程序代码中解耦，从而使得应用程序代码更加简洁、可读性更强，并且易于维护和扩展。

## 源码中的 IoC 刷新过程

1. 添加同步锁
2. 为刷新准备上下文：设置启动时间与激活状态，执行属性资源的初始化。
    1. 设置为激活状态
    2. 开启日志
    3. 初始化上下文环境中的所有预占熟悉资源，由子类决定具体需要初始化的内容。
    4. 验证上下文环境中所有被标记为“必须”核“可解析”的熟悉
    5. 初始化用于预刷新的应用监听器
    6. 初始化应用事件容器，待后续多播器可用时，发布此集合中的事件。
3. 告知各子类刷新内部 Bean 工厂
4. 准备当前上下文所需要的 Bean 工厂
    1. 告诉内部 Bean 工厂使用所属上下文的类加载器
    2. 设置 Bean 表达式解析器
    3. 添加属性编辑器注册机
    4. 配置 Bean 工厂的上下文回调函数（Aware 处理器等），添加 Bean 后置处理器等。
    5. 忽略若干个 `Aware` 接口（`EnvironmentAware, ResourceLoaderAware, MessageSourceAware`, `ApplicationContextAware` 等等）
    6. 注册一些可解析的依赖（`BeanFactory`, `ResourceLoader`, `ApplicationContext`, `ApplicationEventPublisher`）
    7. 注册用于检测内部 Bean 的早期后置处理器作为 ApplicationListeners
    8. 如果检测到**加载时织入器（``LoadTimeWeaver`）**，则准备织入，但是需要为类型匹配的上下文设置临时的类加载器。
    9. 以单例模式注册 `EnvironmentBean`
    10. 以单例模式注册 `SystemPropertiesBean`
    11. 以单例模式注册 `SystemEnvironmentBean`
    12. 以单例模式注册 `ApplicationStartupBean`
5. 执行当前上下文子类中的 BeanFactory 的后置处理
    1. 在完成标准模式的应用上下文初始化之后，修改应用上下文中的 BeanFactory。
    2. 初始的资源定义会被加载，但是不会执行后置处理器，也不会注册派生的 Bean，更重要的是，也没有初始化任何 Bean。
    3. 这个模板方法允许注册特殊的 Bean 后置处理器到 `AbstractApplicationContext` 的子类之中
6. 调用上下文中注册为 Bean 的工厂处理器：初始化并调用所有注册为 `BeanFactoryPostProcessor` 的 Bean，如果显式制定了顺序，则按照顺序执行。
    1. 调用 BeanFactoryPostProcessors
    2. 检测“加载时织入器”并准备织入，比如被 `@Bean` 注解标记并被 `ConfigurationClassPostProcessor` 注册的方法。

7. 注册用于拦截 Bean 创建的处理器
    1. 初始化并注册所有的 `BeanFactoryPostProcessor`，如果显式指定了顺序，则按照顺序执行。

8. 初始化当前上下文中的消息资源
9. 初始化当前上下文中的应用事件广播器，如果未定义，则使用 `SimpleApplicationEventMulticaster`。
10. 特定的上下文子类重写模板方法，初始化其他具有特殊用途的 Bean。
11. 检查应用监听器，并注册到多播器，最后发布早期的应用事件。
12. 完成当前上下文中 `BeanFactory` 的初始化，初始化所有剩余的非懒加载的单例。
     1. 初始化当前上下文的转换服务（`ConversionService`）
     2. 如果之前没有注册过 `BeanFactoryPostProcessor`，注册一个默认的**内置解析器**，用于解析注解的属性值。
     3. 初始化 `LoadTimeWeaverAware` 类型的 Bean，以便可以尽早地注册它们的转换器。
     4. 清空之前设置的临时类加载器，不再使用临时类加载器进行类型匹配。
     5. 缓存所有定义 Bean 的元数据，短时间内不会再有更改。
     6. 初始化所有剩余的非懒加载的实例

13. 完成上下文刷新，发布事件。
     1. 清空上下文级别的资源缓存（如 ASM 扫描时产生的元数据）
     2. 初始化当前上下文中的生命周期处理器
     3. 将完成刷新的通知给到生命周期处理器
     4. 发布最终的事件

14. 如果执行步骤 2~11 过程中产生异常
     1. 销毁已经创建完成的单例，避免资源紊乱。
     2. 取消刷新动作，重置“激活状态”。
     3. 上抛异常
15. 最后，重置 Spring 内核中的公共缓存，因为我们很可能再也不会用到这些元数据区创建单例了。

源码位于 [org.springframework.context.support.AbstractApplicationContext#refresh](https://github.com/spring-projects/spring-framework/blob/main/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java)

```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// 同步锁
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
			// 为刷新准备上下文
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			// 告诉各子类刷新内部 Bean 工厂
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			// 准备当前上下文中需要使用的 Bean 工厂
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 执行 IoC 容器 BeanFactory 的后置处理
				postProcessBeanFactory(beanFactory);

				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
				// 调用上下文中注册为 Bean 的工厂处理器
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				// 注册拦截 Bean 创建的 Bean 处理器
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();

				// Initialize message source for this context.
				// 初始化当前上下文中的消息资源
				initMessageSource();

				// Initialize event multicaster for this context.
				// 初始化当前上下文中的事件广播器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				// 在特定的上下文子类中初始化其他的具有特殊用途的 Bean
				onRefresh();

				// Check for listener beans and register them.
				// 检查并注册监听器 Bean
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				// 初始化所有的非懒加载的单例 Bean
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				// 完成上下文刷新，发布事件。
				finishRefresh();

			} catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				// 销毁已经创建完成的单例，避免资源紊乱。
				destroyBeans();

				// Reset 'active' flag.
				// 取消刷新，重置"激活状态"。
				cancelRefresh(ex);

				// Propagate exception to caller.
				// 异常上抛
				throw ex;
			} finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				// 重置 Spring 内核中的公共缓存，因为我们可能再也不会用到这些元数据去创建单例了。
				resetCommonCaches();
				contextRefresh.end();
			}
		}
	}
```

## BeanFactory 和 FactoryBean 的区别是什么?

Spring BeanFactory 和 FactoryBean 是两个不同的概念，虽然它们的名称很相似，但它们的作用和使用场景是不同的。

BeanFactory 是 Spring IoC 容器的最基本接口，是所有 Spring IoC 容器的基础，提供了一种用于实例化、配置和管理 Spring Bean 的机制。它是 Spring 框架的核心，负责管理 Spring Bean 的生命周期、依赖关系以及各种配置细节。

FactoryBean 是一个特殊的 Bean，它的主要作用是创建和管理其他 Bean。内部封装了实例化复杂 Bean 的逻辑，可以在 Bean 实例化之前进行一些自定义的处理操作，例如条件化创建 Bean 实例、延迟创建 Bean 实例等。FactoryBean 接口定义了两个方法：`getObject()` 和 `getObjectType()`，分别用于获取被管理 Bean 的实例和类型。

可以说，FactoryBean 是一种特殊的 Bean，它的主要作用是用于创建和管理其他 Bean，而 BeanFactory 则是 Spring IoC 容器的核心接口，它负责管理所有的 Spring Bean，包括 FactoryBean 和普通 Bean。

总的来说，BeanFactory 和 FactoryBean 在 Spring 中有不同的作用和使用场景，BeanFactory 是 Spring IoC 容器的基础接口，负责管理所有的 Bean，而 FactoryBean 则是一种特殊的 Bean，用于封装复杂的 Bean 实例化逻辑。

## 依赖注入的方式

> `@Autowire`，`@Resource`，`@Inject`。

```java
// 声明式编程（declarative programming）
@Autowired
SomeClass someClass;
// 命令式编程（imperative programming）
SomeClass someClass = new SomeClass();
```

相同点：每一个都可以通过 **Field Injection** 与 **Setter Injection** 注入依赖。

不同点：

- `javax.annotation.Resource` 与 ` javax.inject.Inject` 属于 Java 扩展包，`org.springframework.beans.factory.annotation.Autowired` 属于 Spring。
- `@Resource`：byName, byType, byQualifier
- `@Inject`：byType, byQualifier, byName
- `@Autowired`：byType, byQualifier, byName
- 使用场景：主要取决于应用设计者的系统设计思想
  - 如果应用行为基于**接口与抽象类**实现，而且这些行为会被全局性地应用，推荐使用 `@Inject` 或 `@Autowired`，在应用升级或补丁修复时使用 `match-by-type` 的影响较小。
  - 如果应用拥有非常复杂的行为，每个行为基于不同的**接口与抽象类**，并且**接口与抽象类**拥有较多的具体实现，推荐使用 `@Resource`，优先选择 `match-by-name`。
  - 如果使用场景仅支持 `Jakarta EE` 而不支持 Spring，则只能使用`@Resource` 或 `@Inject`。
  - 如果使用场景仅支持 `Spring`，则只能使用 `@Autowired`。

参考资料：[Wiring in Spring: @Autowired, @Resource and @Inject](https://www.baeldung.com/spring-annotations-resource-inject-autowire)

### 什么是控制反转？

简而言之，IoC 是一种设计模式，它让应用本身在涉及应用运行所需的依赖的创建与生命周期中扮演着主要角色。在传统的 OOP 开发模式中，当应用需要某个特定的功能时，通常是创建一个实现了这个功能的类的实例，然后再调用这个实例执行具体功能。然而在 IoC 中，应用并不直接创建所需依赖的实例，而是在需要依赖的时候通过第三方工具注入相关依赖。
IoC 可以被类比为餐馆中，顾客下单与厨师根据顾客订单准备食物之间的关系，食客自己并不下厨，而只需表达诉求，厨师会根据这个诉求提供食客想要的食物。

### IoC 的常见实现方式

- Dependency Injection
- Service Locator
- Strategy Pattern
- Factory Pattern

#### 依赖注入（DI）

依赖注入是实现 IoC 的方式之一，具体是指由第三方工具将将应用所需的依赖注入，而非由应用直接创建依赖。依赖注入可以通过三种方式实现：

- 构造器注入：通过类的构造器将依赖注入；
- Setter 注入：通过类的 `setXxx(...)` 方法注入；
- 接口注入：通过接口向接口的实现类注入。

#### 服务定位器（SL）

服务定位器是实现 IoC 的另一种方式，它为应用所需的依赖提供了一个中心化的注册表，服务定位器允许应用通过注册表寻找依赖来代替应用手动创建依赖。

### 控制反转（IoC）的好处

- 低耦合（Loose Coupling）：IoC 倡导应用组件之间的低耦合，减少组件之间的依赖关系，使得代码更易修改和维护。
- 模块化设计（Modular Design）：IoC 倡导模块化设计，应用的每个组件都是自给自足的、可独立测试的。
- 易测试（Testability）：因为可以方便地模拟和插入依赖，所以应用变得更易测试。
- 可复用（Reusability）：IoC 倡导组件的复用，因为组件可以被方便地交换与重置。

### 实现控制反转的步骤

1. 确定应用所需要的依赖；
2. 选择一个 DI 框架或自主实现 DI 容器；
3. 定义应用组件及其依赖；
4. 通过选用的 DI 容器注入组件所需的依赖。

# Spring Bean

## 什么是 Spring Bean？

## Bean 作用域

Spring 框架支持以下五个作用域，如果你使用 web-aware ApplicationContext 时，其中三个是可用的。

| 作用域         | 描述                                                                                                  |
| :------------- | :---------------------------------------------------------------------------------------------------- |
| singleton      | 该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)。                          |
| prototype      | 该作用域将单一 bean 的定义限制在任意数量的对象实例。                                                  |
| request        | 该作用域将 bean 的定义限制为 HTTP 请求。只在 web-aware Spring ApplicationContext 的上下文中有效。     |
| session        | 该作用域将 bean 的定义限制为 HTTP 会话。 只在 web-aware Spring ApplicationContext 的上下文中有效。    |
| global-session | 该作用域将 bean 的定义限制为全局 HTTP 会话。只在 web-aware Spring ApplicationContext 的上下文中有效。 |

## Bean 生命周期

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个 Bean 的实例。
- 如果涉及到一些属性值利用 set 方法进行设置。
- 检查 `Aware `相关接口并进行相应地设置。
  - 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName() 方法，传入 Bean 的名字。
  - 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader() 方法，传入 ClassLoader 对象的实例。
  - 如果 Bean 实现了 BeanFactoryAware 接口，调用 setBeanFactory() 方法，传入 BeanFactory 对象的实例。
  - 与上面的类似，如果实现了其他 XxxAware 接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行 postProcessBeforeInitialization() 方法。
- 如果 Bean 实现了 `InitializingBean` 接口，执行 afterPropertiesSet()方法。
- 如果 Bean 在配置文件中的定义包含 `init-method` 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行 postProcessAfterInitialization()方法。
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 destroy() 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

![Bean 生命周期](https://p.ipic.vip/lxd0i8.jpg)

## Bean 的创建过程

![Bean 的创建过程](https://p.ipic.vip/x22729.png)

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

参考：[一文告诉你 Spring 是如何利用"三级缓存"巧妙解决 Bean 的循环依赖问题的](https://cloud.tencent.com/developer/article/1497692)

# Spring 事务

## Spring 事务隔离级别

> 与此类似的有 `java.sql.Connection` 的事务隔离级别 `TRANSACTION_XXX`。

- `ISOLATION_DEFAULT = -1`: 使用数据库所提供的事务隔离级别；
- `ISOLATION_READ_UNCOMMITTED = 1`：可能会产生脏读、不可重复读、幻读；
- `ISOLATION_READ_COMMITTED = 2`：可以解决脏读，但可能会产生不可重复读与幻读；
- `ISOLATION_REPEATABLE_READ = 4`：可以解决脏读与不可重复读，但可能会产生幻读；
- `ISOLATION_SERIALIZABLE = 8`：可以解决脏读、幻读、不可重复读。

## Spring 事务传播

> 以下 7 种事务传播类型均定义在 `org.springframework.transaction.TransactionDefinition` 接口中，类型为 `int` 而非 `enum`，在这个 `interface` 中，还包含了对于事务隔离级别的定义。

**支持当前事务的情况：**

- **`PROPAGATION_REQUIRED = 0`：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建新事务。
- **`PROPAGATION_SUPPORTS = 1`：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **`PROPAGATION_MANDATORY = 2`：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

**不支持当前事务的情况：**

- **`PROPAGATION_REQUIRES_NEW = 3`：** 如果当前存在事务，则把当前事务挂起，创建一个新的事务。
- **`PROPAGATION_NOT_SUPPORTED = 4`：** 不支持当前事务，以非事务方式运行。
- **`PROPAGATION_NEVER = 5`：** 不支持当前事务，以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **`PROPAGATION_NESTED = 6`：** 如果当前存在事务，则以内嵌事务的形式运行；如果当前没有事务，则该取值等价于 **PROPAGATION_REQUIRED**。

**实现**：基于 `ThreadLocal` ，因此不可跨线程，跨线程需要**透传**。

**会新建事务的隔离级别**：`REQUIRED, REQUIRES_NEW, NESTED`

## Spring 事务失效的场景

### 事务不生效

1. 当 Spring 开启了仅允许 public 方法，但添加事务注解的方法为非 public 时；
2. 当 `@Transactional` 注解修饰的方法同时也被 `final` 关键字修饰时，因为 Spring 基于 AOP 使用动态代理实现事务，如果 Spring 无法进行对方法进行横切，则无法添加事务相关逻辑；
3. 未标注 `@Transactional` 注解的方法 A 调用标注了 `@Transactional` 注解的 方法 B 时，因为虽然 Spring 为 B 添加了事务功能，但具有事务功能的是 B 的代理对象 `B’`，而方法 A 调用 B 时实际上是通过 `this` 调用的 B 的原始方法，即未被 Spring 添加事务功能的原始方法 B；
   1. 解决方法 1：将方法 B 单独抽离到一个新的被 Spring 托管的 Bean ServiceB，在 A 中注入 ServiceB 后，使用 serviceB.B() 调用方法 B；
   2. 解决方法 2：在方法 A 所属的类 ServiceA 中注入自身，然后通过 serviceA.B() 调用方法 B；
   3. 解决方法 3：通将 `AopContext.currentProxy` 强转成 ServiceA，然后再调用方法 B。
4. 使用 `@Transactional` 注解方法未被 Spring 所管理；
5. Spring 的事务是基于数据库连接实现的，如果方法 A 与方法 B 分属于不同的线程并且获取了不同的数据库连接而属于不同的数据库事务，则事务也是失效的；
6. 所要操作的表所属的存储引擎不支持事务时；
7. Spring 的事务配置错误，未添加注解或未指定数据源；

### 事务不回滚

- 设置了错误的事务传播属性，如 `propagation = Propagation.NEVER`；
- 手动捕获了业务执行过程中抛出的异常，导致 Spring 没有感知；
- 抛异常了，但抛出的是 `RuntimeException/Error` 之外的异常/错误类型；
- 自定义了回滚异常，但业务代码中抛出的异常并不是自定义异常的类型，《阿里巴巴开发手册》建议指定该属性，但属性指定为 `Exception、Throwable`；

### 事务回滚多了

- 内嵌事务中没有手动捕获异常，导致异常上抛，导致外层事务也回滚。

## 大事务问题

可能导致死锁、锁等待、回滚时间长、接口超时、并发情况下数据库连接池被占满、数据库主从延迟。

## Spring 中的事务具体怎么实现的？

1. 如果 Spring 没有指定事务隔离级别，则会采用数据库默认的事务隔离级别；当 Spring 指定了事务隔离级别，则会在代码里将事务隔离级别修改为指定值；当数据库不支持 Spring 指定的隔离级别时，效果则以数据库的为准（比如采用了 MyISAM 引擎）。
2. Spring 事务的底层**依赖于数据库的事务**，代码层面上利用 AOP 实现。MySQL 的事务有隔离级别的概念，只有 InnoDB 有事务，实现方式是利用 undo log 和 redo log。
3. 通过 AOP 实现声明式事务
4. 通过注解的事务方式为**声明式事务**，但是 Spring 也通过 `TransactionTemplate` 提供了**编程式事务**。

## Spring MVC 工作流程

![request lifecycle](https://terasolunaorg.github.io/guideline/1.0.1.RELEASE/en/_images/RequestLifecycle.png)

1. 用户发送请求到 DispatcherServlet
2. DispatcherServlet 收到请求后调用处理器映射器 HandlerMapping
3. 处理器映射器找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给 DispatcherServlet
4. DispatcherServlet 调用 处理器适配器 HandlerAdapter
5. 处理器适配器经过适配调用具体的处理器(Controller，也叫后端控制器)
6. Controller 执行完成返回 ModelAndView
7. HandlerAdapter 将 Controller 执行结果 ModelAndView 返回给 DispatcherServlet
8. DispatcherServlet 将 ModelAndView 传给视图解析器 ViewReslover
9. ViewReslover 对 ModelAndView 进行解析后返回具体视图给 DispatcherServlet
10. DispatcherServlet 根据视图进行渲染，把模型数据填充至 request 域中
11. 最后 DispatcherServlet 把处理结果响应给用户

> - [从 SpringMvc 源码分析其工作原理](https://juejin.im/post/6844903826000969742)

# Spring 中的设计模式

## 答案 1

1. 工厂设计模式 : Spring 使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
2. 代理设计模式 : Spring AOP 功能的实现
3. 单例设计模式 : Spring 中的 Bean 默认都是单例的
4. 模板方法模式 : Spring 中 JdbcTemplate、HibernateTemplate 等以 Template 结尾的对数据库操作的类，使用到了模板模式。
5. 包装器设计模式 : 项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库，包装器模式让可以根据客户的需求能够动态切换不同的数据源。
6. 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用
7. 适配器模式: Spring AOP 的增强或通知（Advice）使用到了适配器模式、Spring MVC 中也是用到了适配器模式适配 Controller。

> 推荐阅读：[面试官:“谈谈 Spring 中都用到了那些设计模式?”](https://juejin.im/post/6844903849849962509)

## 答案 2：ChatGPT

在 Spring 框架中，有许多设计模式被广泛应用，以下是一些常见的设计模式：

1. 依赖注入（Dependency Injection）：Spring 框架中最核心的特性之一。它通过控制反转（Inversion of Control）实现，利用工厂模式创建对象，并通过依赖注入的方式将对象注入到调用方。这样，应用程序的组件可以完全脱离具体的实现，实现了松耦合。
2. AOP（面向切面编程）：Spring 框架通过 AOP 实现对横切关注点的处理，如事务、安全、缓存、日志等。AOP 主要利用代理模式实现，可以将横切关注点抽象出来形成切面（Aspect），通过将切面织入到目标对象中，来实现横切关注点的处理。
3. 工厂模式（Factory Pattern）：Spring 中有很多工厂类用于创建和管理对象，如 BeanFactory 和 ApplicationContext。这些工厂类都是使用工厂模式来实现的。
4. 单例模式（Singleton Pattern）：Spring 容器中的对象默认都是单例的，可以避免创建大量的重复对象，提高应用程序的性能。
5. 观察者模式（Observer Pattern）：Spring 框架中的事件驱动模型就是使用观察者模式实现的，事件源发布事件后，监听该事件的监听器就会接收到通知并进行相应的处理。
6. 模板方法模式（Template Method Pattern）：Spring 中的 JdbcTemplate 就是使用模板方法模式实现的，提供了一些标准的 SQL 操作方法，具体的 SQL 操作则由子类来实现。
7. 代理模式（Proxy Pattern）：Spring 中的 AOP 就是使用代理模式实现的，通过代理对象来实现对目标对象的增强处理。

总之，Spring 框架的设计思想是基于众多经典的设计模式，并将其结合在一起，形成了一个强大的框架，可以大大提高应用程序的开发效率和可维护性。

## Spring 是如何把注解标注的对象加入 IoC 容器的？
