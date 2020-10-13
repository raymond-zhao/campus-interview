## 不大于 n 的所有质数

- [埃拉托斯特尼筛法](https://zh.wikipedia.org/wiki/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E6%B3%95)

```java
// O(nloglog n)
public static List<Integer> countPrime(int n) {
    List<Integer> res = new ArrayList<>();
    boolean[] flags = new boolean[n];
    for (int i = 2; i <= n; i++) {
        if (!flags[i - 1]) {
            res.add(i);
            for (int j = i * i ; j <= n; j += i) {
                flags[j - 1] = true;
            }
        }
    }
    return res;
}
```

