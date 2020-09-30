## 类加载过程

![类加载过程](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7t2gw8xxj30ha05saax.jpg)

1. 类加载主要分为三个过程，分别是加载、连接、初始化，其中连接过程又分为验证、准备、解析三个步骤。
2. **加载过程**，在这个过程虚拟机需要完成三件事情
   1. 根据类的全限定名获取定义此类的二进制字节流
   2. 将该二进制字节流所代表的静态存储结构转化为方法区中的运行时数据结构
   3. 在内存中生成一个代表该类的java.lang.Class对象，作为方法区中这个类的各种数据的访问入口。
3. **验证阶段**：目的是确保Class文件的字节流中包含的信息合法，又分为
   1. **文件格式验证**：验证字节流是否符合Class文件格式的规范，比如魔数、主次版本号、常量池等。
   2. **元数据验证**：对字节码描述的信息进行语义分析，对类的元数据信息进行语义校验。是否有父类、是否继承了不该继承的类，诸如此类。
   3. **字节码验证**：通过数据流分析和控制流分析，确定程序语义是否合法，是否符合逻辑。
   4. **符号引用验证**：目的是确保解析行为能够正常进行。
4. **准备阶段**：为类中定义的变量(即静态变量，被`static`修饰的变量)分配内存并设置类变量初始值。
5. **解析阶段**：将常量池内的符号引用替换为直接引用的过程
   1. 类或接口的解析
   2. 字段解析
   3. 方法解析
   4. 接口方法解析
6. **初始化阶段**：此前的阶段都由JVM来主导，初始化阶段开始JVM才真正执行类中编写的Java程序代码，将主导权交给应用程序。
   1. 初始化阶段，会根据程序员通过程序编码制定的主观计划去初始化类变量和其他资源。
   2. 其实就是执行类构造器`<clinit>()`方法的过程，这个方法是Javac编译器的生成物。

> **符号引用**：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
>
> **直接引用**：直接引用是可以直接指向目标的指针、相对偏移量、或者是一个能间接定位到目标的句柄。

## 类加载的时机

- 遇到 `new`、`getstatic`、`putstatic` 或 `invokestatic` 这四条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化阶段。能够生成这四条指令的典型Java代码场景有：
  - 使用 `new` 关键字实例化对象时
  - 读取或设置一个类型的静态字段时（被final修饰、已在编译期把结果放入常量池的静态字段除外）。
  - 调用一个类型的静态方法时。
- 使用 `java.lang.reflect` 包的方法对类进行反射调用时，如果类没有进行过初始化，则需要先触发其初始化。
- 当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
- 当使用 JDK 7 新加入的动态语言支持时，如果一个 `java.lang.invoke.MethodHandle` 实例最后的解析结果为 `REF_getStatic`、`REF_putStatic`、`REF_invokeStatic`、`REF_newInvokeSpecial` 四种类型的方法句柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化。
- 当一个接口中定义了 JDK 8 新加入的默认方法（被default关键字修饰的接口方法）时，如果这个接口的某个实现类发生了初始化，那该接口要在实现类之前被初始化。

## 双亲委派模型

类加载器有三层，分别是：

- 启动类加载器（Bootstrap Class Loader）：负责加载存放在`<JAVA_HOME>/lib`目录，或者被`-Xbootclasspath`所指定的路径中存放的，而且能够被JVM识别的类库加载到虚拟机的内存中。
- 扩展类加载器（Extension Class Loader）：负责加载`<JAVA_HOME>/lib/ext`目录中，或者被`java.ext.dirs`系统变量所指定的路径中所有的类库。
- 应用程序类加载器（Application Class Loader）：负责加载用户类路径（ClassPath）上所有的类库，如果应用程序中没有自定义过类加载器，这就是默认的类加载器。

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求(它的搜索范围中没有找到所需的类)时，子加载器才会尝试自己去完成加载。

> 双亲委派模型的好处：
>
> 1. 保证每个类只被加载一次，避免类的重复加载。
> 2. 防止核心 API 被篡改，比如重写 `rt.jar` 下的同名 Java 类，不会被加载运行(自定义一个String类，里面写个main方法，连 run 的启动都不显示了，同包下所有使用到 String 类的地方也全部会报错)。

## 如何自己实现类加载器？

1. 继承`java.lang.ClassLoader`类
2. 如果想打破双亲委派模型的话，重写`loadClass()`类
3. 如果不想打破双亲委派模型的话，重写`findClass()`类

## Tomcat 是如何违背双亲委派模型的

Tomcat 作为 Web 容器：

1. 需要支持**文件的热修改**；
2. 相同 Web 容器中不同应用程序相互隔离(不同应用程序可能会依赖相同的第三方类库的不同版本)；
3. Web 容器也有自己依赖的类库，不能与应用程序的类库混淆；
4. 需要**隔离性**。

![Tomcat 类加载模型](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7taxqg87j30cu0i774f.jpg)

在 Tomcat 的类加载机制中：

- 最上层三个类加载器仍然是系统类加载器相同；
- Tomcat 自定义了 Common 类加载器（已加载的 class 可被 **Tomcat Server本身** 及**各个 WebApp **访问）；
- 自定义了 Catalina ClassLoader（类对 WebApp 不可见）、SharedClassLoader （WebApp 共享，；类对所有 WebApp 可见，对 Catelina 不可见）作为 CommonClassLoader 的两个子加载器，它们之间相互隔离。
- 在 SharedClassLoader 下定义了子类加载器 WebAppClassLoader（WebApp 私有，加载路径中的类只对当前 WebApp 可见，可以使用 SharedXX 加载的类），再之下是  JasperLoader（仅限于 JSP 编译出的类文件，目的是实现热更新。当JSP被修改时，替换掉当前的 JasperLoader 实例，建立新的JSP类加载器实现热更新）。

> 如果 Common ClassLoader 想加载 WebApp ClassLoader 中的类该怎么办？
>
> 使用线程上下文类加载器，让父类加载器请求父类加载器去完成类加载动作。
