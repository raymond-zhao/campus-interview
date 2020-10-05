1. 面向对象三大特性



2. [抽象类与接口的区别](https://www.zhihu.com/question/20149818)

* 接口是对动作的抽象，而抽象类是对根源的抽象。

* 抽象类和接口都不能被直接实例化，如果二者要实例化，就涉及到多态。如果抽象类要实例化，那么抽象类定义的变量必须指向一个子类对象，这个子类继承了这个抽象类并实现了这个抽象类的所有抽象方法。如果接口要实例化，那么这个接口定义的变量要指向一个子类对象，这个子类必须实现了这个接口所有的方法。
* 抽象类要被子类继承，接口要被子类实现。
* 接口里面只能对方法进行声明，抽象类既可以对方法进行声明也可以对方法进行实现。
* 抽象类里面的抽象方法必须全部被子类实现，如果子类不能全部实现，那么子类必须也是抽象类。接口里面的方法也必须全部被子类实现，如果子类不能实现那么子类必须是抽象类。
* 接口里面的方法只能声明，不能有具体的实现。这说明接口是设计的结果，抽象类是重构的结果。
* 抽象类里面可以没有抽象方法。
* 如果一个类里面有抽象方法，那么这个类一定是抽象类。
* 抽象类中的方法都要被实现，所以抽象方法不能是静态的static，也不能是私有的private。
* 接口（类）可以继承接口，甚至可以继承多个接口。但是类只能继承一个类。
* 抽象级别（从高到低）：接口>抽象类>实现类。
* 抽象类主要是用来抽象类别，接口主要是用来抽象方法功能。当你关注事物的本质的时候，请用抽象类；当你关注一种操作的时候，用接口。
* 抽象类的功能应该要远多于接口，但是定义抽象类的代价较高。因为高级语言一个类只能继承一个父类，即你在设计这个类的时候必须要抽象出所有这个类的子类所具有的共同属性和方法；即你在设计这个类的时候必须要抽象出所有这个类的子类所具有的共同属性和方法；因此每个接口你只需要将特定的动作方法抽象到这个接口即可。
* 也就是说，接口的设计具有更大的可扩展性，而抽象类的设计必须十分谨慎。

3. equals() 与 hashCode()

> - hashCode() 返回的并不是实际的内存地址，而是与内存地址相关联的计算结果值。如果是实际内存地址的话，JVM 中的频繁 GC 和内存移动将会导致对象 hashCode 的改变。
> - 从源码中可以看到 hashCode() 产生于 [ObjectSynchronizer::FastHashCode](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp)，它具体的实现在 [synchronizer.cpp](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/runtime/synchronizer.cpp) 中的 get_next_hash。

很多时候我们使用 equals() 需要比较的是**对象的值，而不是地址**，如果两个对象内存地址相同，那两个对象也一定相同。所以 equals() 重写的 equals() 方法中首先判断的还是**地址**(如 String 类)，如果地址不同才会去判断**值是否相等**。

> 为什么重写 equals() 就要重写 hashCode()？

因为如果不这样做的话，就会违反 hashCode() 的通用约定，从而导致该类无法结合所有基于散列的集合一起正常工作，这类集合包括 HashMap 和 HashSet。如果用对象作为 Key 的话，可能会返回 null。

这里的**通用约定**，从 Object 类的 hashCode() 方法的注释可以了解，主要包括以下几个方面：

- 在应用程序的执行期间，只要对象的 equals() 方法的比较操作所用到的信息没有被修改，那么对同一个对象的多次调用，hashCode() 方法都必须始终返回同一个值。
- **如果两个对象调用 equals() 方法比较是相等的，那么调用这两个对象中的 hashCode() 方法必须产生同样的整数结果。**
- 如果两个对象根据 equals() 方法比较是不相等的，那么调用者两个对象中的 hashCode() 方法，则不一定要求 hashCode 方法必须产生不同的结果。但是给不相等的对象产生不同的整数散列值，是有可能提高散列表（hash table）的性能。

String 类的 equals() 与 hashCode()

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true; // 首先比较地址
    }
    // 如果地址不同才去比较值
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}

public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
		// s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

- 参考：[对Java中HashCode方法的深入思考](https://juejin.im/post/6844903910839156743)


## 关键字作用域

| 作用域,可见性 | 当前类 | 同一package | 子类 | 其他package |
| ------------- | ------ | ----------- | ---- | ----------- |
| `private`     | ✅      |             |      |             |
| `default`     | ✅      | ✅           |      |             |
| `protected`   | ✅      | ✅           | ✅    |             |
| `public`      | ✅      | ✅           | ✅    | ✅           |

