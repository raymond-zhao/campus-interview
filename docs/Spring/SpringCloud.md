## Feign 工作原理

- 首先通过 @EnableFeignCleints 注解开启 FeignCleint；
- 根据 Feign 的规则实现接口，并加 @FeignCleint 注解；
- 程序启动后，会进行包扫描，扫描所有的 @FeignCleint 的注解的类，并将这些信息注入到 IoC 容器中；
- 当接口的方法被调用，通过 JDK 的代理，来生成具体的 RequesTemplate；
- RequesTemplate 再生成 Request；
- Request 交给 Client 去处理，其中 Client 可以是 HttpUrlConnection、HttpClient 也可以是 Okhttp；
- 最后 Client 被封装到 LoadBalanceClient 类，这个类结合 Ribbon 做到了负载均衡。

## Spring Session 工作原理

## Nacos 工作原理