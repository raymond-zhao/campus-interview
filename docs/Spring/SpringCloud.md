## Feign 工作原理

> 本部分总结自：[Feign原理(图解)](https://www.cnblogs.com/crazymakercircle/p/11965726.html) 与 Feign 源码。

Feign 远程调用，核心就是通过一系列的封装和处理，将以 Java 注解的方式定义的远程调用 API 接口，最终转换成 HTTP 的请求形式，然后将 HTTP 的请求的响应结果，解码成 Java Bean，返回给调用者。

**在微服务启动时：**

1. 扫描所有加上 @FeignClient 注解的远程接口，创建本地 JDK Proxy 代理实例，生成 RequestTemplate，并且注册到 IoC 容器中；
2. 进入 FeignInvocationHandler 调用处理器这个类，根据其成员变量 dispatch 完成从 Method 到 MethodHandler 的映射，然后调用 MethodHandler 的 invoke(...) 方法；
3. 进入 MethodHandler 类，通过其内部 Client 对象，完成实际的 URL 请求、获取响应结果，根据选用的 Client 对象的类型进行解析（主要还是负载均衡 Client ），最后返回给调用者。
4. **应用了模板方法、代理模式等设计模式。**

在这个过程中，各组件的作用如下：

- **动态代理 JDK Proxy：**
  - Proxy 代理实例，实现了一个加 @FeignClient 注解的远程调用**接口**；
  - Proxy 代理实例，能在内部进行 HTTP 请求的封装，以及发送 HTTP 请求；
  - Proxy 代理实例，能处理远程 HTTP 请求的响应，并且完成结果的解码，然后返回给调用者。
- **默认的调用处理器 FeignInvocationHandler：** 自定义静态内部类，和 JDK Proxy 中的 InvocationHandler 没有关系。
  - 为了创建 Feign 的远程接口的代理实现类，Feign 提供了自己的一个默认的调用处理器；
  - 调用处理器可以进行替换，如果 Feign 与 Hystrix 结合使用，则会替换成 HystrixInvocationHandler 调用处理器类。
  - 成员变量 `Map<Method, MethodHandler> dispatch;` 保存着远程接口方法 Method 到 MethodHandler 方法处理器的映射。
  - `invoke(...)` 方法：
    - 根据 Java 反射的方法实例，在 dispatch 映射对象中，找到对应的 MethodHandler 方法处理器；
    - 调用 MethodHandler 方法处理器的 invoke(...) 方法，完成实际的 HTTP 请求和结果的处理。
- **方法处理器 MethodHandler**：
  - `invoke(...)`方法：完成实际远程请求，返回解码后的远程响应。
  - 实现类：`SynchronousMethodHandler `提供了基本的远程 URL 的同步请求处理，`invoke(…)` 方法，调用了自己的 `executeAndDecode(…)` 请求执行和结果解码方法。该方法的工作步骤：
    - 通过 `RequestTemplate` 请求模板实例，生成远程 URL 请求实例 request；
    - 调用内部的 Feign 客户端 Client 成员的 `execute(request, this.options)` 方法执行请求，并且接收响应；
    - 对 Response 响应进行解码。
- **客户端组件 feign.Client：**负责端到端的执行 URL 请求。发送 Request 请求到服务器，并接收 Response 响应后进行解码。
  - **Client.Default**：默认的 feign.Client 客户端实现类，内部使用 HttpURLConnnection 完成 URL 请求处理；
    - 使用简单的 HTTP 连接池技术，复用能力与性能都比较低、
  - **ApacheHttpClient**：内部使用 Apache httpclient 开源组件完成 URL 请求处理的 feign.Client 客户端实现类；
    - 带有连接池的功能，具备优秀的 HTTP 连接的复用能力。
  - **OkHttpClient**：内部使用 OkHttp3 开源组件完成 URL 请求处理的 feign.Client 客户端实现类；
    - 支持 SPDY协议（Google 开发的基于 TCP 的传输层协议，用以最小化网络延迟，提升网络速度，优化用户的网络使用体验。）
  - **LoadBalancerFeignClient**：内部使用 Ribbon 负载均衡技术完成 URL 请求处理的 feign.Client 客户端实现类。使用了 delegate 包装代理模式：
    - Ribbon 计算出合适的 server 之后，由内部包装 delegate 代理 Client 完成到 Server 的 HTTP 请求；
    - 所封装的 delegate 客户端代理实例的类型，可以是 Client.Default 默认客户端，也可以是 ApacheHttpClient 客户端类或 OkHttpClient 高性能客户端类，还可以其他的定制的 feign.Client 客户端实现类型。

![Feign远程调用的基本流程](https://gitee.com/raymond-zhao/oss/raw/master/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70.png)

## Spring Session 工作原理

> [Spring Session 工作原理详情](https://github.com/raymond-zhao/cat-mall/wiki/Spring-Session)

![Session工作原理](https://tva1.sinaimg.cn/large/007S8ZIlly1ghsxy2b1vxj30ly0alt9n.jpg)

### 分布式 Session 出现的问题

- 同一个服务，复制多份，Session 不同步；
- 不同服务，Session 不共享。

### 负载均衡后 Session 共享解决方案

- Web-Server 间进行 Session 复制
  - 优点：Web-Server(Tomcat) 原生支持，只需要修改配置文件即可。
  - 缺点：
    - Session 同步需要数据传输，占用网络带宽，降低了服务器群的业务处理能力；
    - 任意一台 Web-Server 保存的数据都是所有 Web-Server 的 Session 总和，收到内存限制无法水平扩展更多的 Web-Server；
    - 大量分布式集群情况下，由于所有 Web-Server 都保存全量数据，所以此方案不可取。
- Session 存储在客户端 Cookie 中
  - 优点：服务器不需存储 Session，用户保存自己的 Session 信息到 Cookie 中，节省服务端资源。
  - 缺点：**全是缺点，这只是思路！**
    - 每次 HTTP 请求，携带用户在 Cookie 中的完整信息，占用网络带宽；
    - Session 数据放在 Cookie 中，而浏览器一般都对 Cookie 长度做限制（4 K），不能保存大量信息；
    - Session 放在 Cookie 中，存在**被泄漏、被篡改、被窃取**的风险。
- Nginx 负载均衡之 IP_HASH
  - 优点：
    - 只需要修改 Nginx 配置，不需要修改应用代码；
    - 负载均衡，只要 Hash 属性的值分布式均匀的，多台 Web-Server 的负载是均衡的；
    - 可以支持 Web-Server 水平扩展
  - 缺点：
    -  Session 还是存储在 Web-Server 中，所以 Web-Server 重启可能导致部分 Session 丢失，影响业务，如部分用户需要重新登录；
    - 如果 Web-Server 水平扩展，rehash 后 Session 重新分布，也会有一部分用户路由不到正确的 Session。
    - 但是以上两点entity也不是很大，因为 Session 本来也是有有效期的。
- 后端统一存储（Redis、DB 等）
  - 优点：
    - 没有安全隐患；
    - 可以水平扩展，数据库/缓存水平切分即可；
    - Web-Server 重启或者扩容都不会有 Session 丢失。
  - 不足：
    - 增加了一次网络调用，并且需要修改应用代码；

> Spring Session 可以解决上面几种方案中的缺点。

### Spring Session 核心原理

- 第一次使用 Session，浏览器将会保存相应信息 JSESSIONID 这个 Cookie；
- 以后浏览器访问网站时将会带上这个网站的 Cookie；
- 在子域时进行作用域放大，同时 JSON 序列化对象存储到 Redis；
- **关于子域放大为什么会生效**：[RFC 6265](http://tools.ietf.org/html/rfc6265)，**忽略任何前导点**，所以可以在子域和顶级域上使用 Cookie，而 Session 是利用 Cookie 实现的。

> **一言以蔽之**：Spring 实现了一个实现了 javax.servlet 包下 Filter 的 Bean，结合 Redis 作为存储容器，利用 **装饰器模式** 将原生的 request 与 response 对象进行包装，包装后的对象应用在后面的整个 doFilter() 执行链，以后 Session 不用再从原生的 HttpServletRequest 中获取，而是从 Redis 中获取。

**源码分析**

1. `@EnableRedisHttpSession`注解导入了`RedisHttpSessionConfiguration`配置类；
2. `RedisHttpSessionConfiguration`给容器中添加了名为 `RedisIndexedSessionRepository`的组件，这个类封装了 Redis 对 Session 的大量操作；
3. `RedisHttpSessionConfiguration` 继承的 `SpringHttpSessionConfiguration` 中有一个类型为`SessionRepositoryFilter` 的`Bean`，这个`Bean`最上层的接口就是`javax.servlet`下的`Filter`；

- `SessionRepositoryFilter`继承了抽象类`OncePerRequestFilter`，这个类下有一个名为 `doFilterInternal` 的抽象方法，`SessionRepositoryFilter`重写了这个方法，作为`Session`存储过滤器，每个请求都必须经过`filter`，它在
  - 创建的时候，自动从容器中获取`SessionRepository`；
  - 原始的`request`被包装成`SessionRepositoryRequestWrapper`；
  - 原始的`response`被包装成`SessionRepositoryResponseWrapper`；
  - 包装后的对象应用到了后面的整个 doFilter() 执行链；
  - 以后再获取`Session`，使用`wrappedRequest.getSession()` 从 `SessionRepository`中获取(从 `Redis` 中获取)，而不用从 `HttpServletRequest.getSession()` 获取；
  - 使用了装饰者`Decorator`模式。
- 使用`Spring Session`时，`Redis`中存储的`Session`有默认时间，但是如果页面有活动的话也会自动续期。

## Nacos 工作原理

