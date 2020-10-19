## 二分搜索置顶

- [liweiwei1419 老哥的汇总讲解](https://leetcode-cn.com/problems/search-insert-position/solution/te-bie-hao-yong-de-er-fen-cha-fa-fa-mo-ban-python-/)

## 二分可用的数组分类

- 正序数组：[1, 2, 3, 4, 5, 6]，中规中矩的非递减数组。
- 旋转数组：[5, 6, 1, 2, 3, 4]，[5, 6]、[1, 2, 3, 4] 是两个有序数组。[5, 6] 右移循环到了数组前面。
- 山脉数组：[1, 2, 3, 4, 6, 5]，中间高，两边低。

## 二分搜索问题分类

- 在一个数组里查找一个数，简称为「**二分下标**」；
- 查找一个有范围的整数，简称为「**二分答案**」，这部分题目还挺常见；
- 通过一个变量与另一个变量的相关关系，进而确定目标变量的值，统称为「复杂问题」，通常这一类问题的判别函数需要遍历整个数组。

## 二分循环分类

> 第一类：while (left <= right)

- 这种写法表示**在循环体内部直接查找元素**，在循环的过程：
  - 先看位于中间位置的元素，如果等于目标元素则直接返回；
  - 否则，根据中间位置的值与目标值 `target` 进行对比，决定下一步是往左边还是右边继续查找。
- `left <= right`，表示此时区间中还剩下一个元素，需要继续寻找（因为我们要做的是***在循环体内部直接查找元素***）；
- 退出循环时的条件为 `left = right + 1` 或者 `right = left - 1`，此时构不成区间，所以是空区间，区间内不存在任何元素；

> 在此引用一下 **liweiwei1419** 老哥的图片，下图同。图速度有点快，可以在某个时刻截图，定格那个瞬间。

![img](https://pic.leetcode-cn.com/1598340841-leuSyu-file_1598340837977)

> 第二类：while (left < right)

- 这种写法表示**在循环体内部排除元素**；
- 退出循环时，`left == right`，区间 $[left, right]$ 只剩下 $1$ 个元素，这个元素***有可能***就是我们要寻找的目标元素 `target`。

![img](https://pic.leetcode-cn.com/1598340841-tpXMfu-file_1598340837988)

## 二分搜索中间值的计算方式

```java
@Test
void testLimit() {
    int left = 1, right = Integer.MAX_VALUE;
    System.out.println(left + right);
    int mid1 = (left + right) / 2;
    int mid2 = (left + right) >> 1;
    int mid3 = left + (right - left) / 2;
    int mid4 = (left + right) >>> 1;
    System.out.println("mid1: " + mid1);
    System.out.println("mid2: " + mid2);
    System.out.println("mid3: " + mid3);
    System.out.println("mid4: " + mid4);
}
```

- 第一种：`int mid = (left + right) / 2;`
  - 这种方式的缺点是可能会产生上溢，比如 `left = 1; right = Integer.MAX_VALUE;` 此时 `left + right = -2147483648`；
  -  `mid = -1073741824`，因为 `Integer` 为有符号整数，最高位为符号位，最大值为 $2^{31} - 1$，当超过最大值之后继续**进位**，结果将会由正变负。抛出 `IndexOutOfBounds` 异常。
- 第二种：`int mid2 = (left + right) >> 1;`
  - `>>` 符号为**有符号右移**，效果跟第一种差不多。
- 第三种：`int mid3 = left + (right - left) / 2;`，
  - 把等式右边通分，可得到 `int mid3 = (left + right) / 2;`；
  - 可以看出，在数学上，与第一种情况相同，但是在 Java 中计算结果却是截然不同。
  - 聪明的我们可以推测出，在判断**完全平方数**中，在判断 `mid * mid == target` 时，为了防止溢出，写成 `mid = target / mid` 是一种比较明智的选择。
- 第四种：`int mid4 = (left + right) >>> 1;`
  - `>>>` 是**无符号右移**运算符，借鉴自 JDK 中 `Arrays.binarySearch(..)`；中的实现；
  - 既然是无符号右移，右移时高位补 $0$，依旧是正数，所以正负号便不再是主要问题了。

## 704. 二分查找

- [704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

给定一个 $n$ 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target`  ，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 `-1`。

<!-- tabs:start -->

#### ** 思路 1 **

```java
public int search(int[] nums, int target) {
    if (nums == null || nums.length == 0) return -1;
    int left = 0, right = nums.length - 1; // right 取值，可以取到。
    while (left <= right) { // while 条件 [left, right]
        int mid = (left + right) >>> 1;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1; // right 更新
        }
    }
    return -1;
}
```


#### ** 思路 2 **

```java
public int search(int[] nums, int target) {
    if (nums == null || nums.length == 0) return -1;
    // right 取值，取不到 nums.length，在数组中会越界
    int left = 0, right = nums.length;
    while (left < right) { // while 条件 [left, right)
        int mid = (left + right) >>> 1;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid; // right 更新
        }
    }
    return -1;
}
```

<!-- tabs:end -->

## 35. 搜索插入位置

- [35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

假设数组中无重复元素。

<!-- tabs:start -->
#### ** 思路 1 **

```java
// 利用思路1：时间 100%，空间 99.73%
public int searchInsert(int[] nums, int target) {
    if (nums == null || nums.length == 0) return -1;
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return left;
}
```

#### ** 思路 2 **

```java
// 利用思路1：双百
public int searchInsert(int[] nums, int target) {
    if (nums == null || nums.length == 0) return -1;
    int left = 0, right = nums.length;
    while (left < right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] < target) {
            left = mid + 1; // 下一轮搜索的区间是 [mid + 1, right]
        } else {
            right = mid; // 下一轮搜索的区间是 [left, mid]
        }
    }
    return left;
}
```
<!-- tabs:end -->

## 34. 在排序数组中查找元素的第一个和最后一个位置

- [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

给定一个按照升序排列的整数数组 $nums$，和一个目标值 $target$。找出给定目标值在数组中的开始位置和结束位置。

你的算法时间复杂度必须是 $O(log n)$ 级别。

如果数组中不存在目标值，返回 `[-1, -1]`。

```java
// 100%; 99.88%
public int[] searchRange(int[] nums, int target) {
    if (nums == null || nums.length == 0)
        return new int[]{-1, -1};
    // 1. 搜索左边界
    int leftIdx = getLeftIdx(nums, target);
    // 1.1 如果左边界为 -1，说明不存在，提前返回。
    if (leftIdx == -1)
        return new int[]{-1, -1};
    // 2. 搜索右边界
    int rightIdx = getRightIdx(nums, target);
    // 2.1 只要能走到这里，说明上面至少找到了左边界，
    // 即使 target 元素只出现了一次，此时 rightIdx = leftIdx，也是有结果的。
    return new int[]{leftIdx, rightIdx};
}

private int getLeftIdx(int[] nums, int target) {
    int left = 0, right = nums.length - 1; // [0, len - 1] 范围内
    while (left < right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] < target) { // 搜索左边界，继续往左找。
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    if (nums[left] == target)
        return left;
    return -1;
}

private int getRightIdx(int[] nums, int target) {
    int left = 0, right = nums.length - 1; // [0, len - 1] 范围内
    while (left < right) {
        int mid = (left + right + 1) >>> 1; // 上取整
        if (nums[mid] > target) { // 搜索右边界，继续往右找。
            right = mid - 1;
        } else {
            left = mid; // 出现这个，就要注意上取整。
        }
    }
    return right;
}
```

## 33. 搜索旋转排序数组

- [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

有一个升序排列的整数数组 `nums` ，和一个整数 `target` 。

假设按照升序排序的数组在预先未知的某个点上进行了旋转。（例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` ）。在数组中搜索 `target` ，如果数组中存在这个目标值，则返回它的索引，否则返回 `-1` 。

- 方法：先确定一个单调递增的区间，然后在其中搜索。

```java
public int search(int[] nums, int target) {
	if (nums == null || nums.length == 0)
        return -1;
    int left = 0, right = nums.length - 1; // 闭区间：[left, right]
    while (left <= right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] == target)
            return mid;
        // 根据 nums[mid] 与 nums[left] 的关系判断 mid 是在左段还是右段 
        // [5, 6, 0, 1, 2, 3, 4]
        if (nums[mid] < nums[right]) {
            // 如果中间值小于最右边值，说明从 [mid, right] 有序。
            if (nums[mid] < target && target <= nums[right]) { // 注意点，(mid, right]
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        } else {
            // 如果中间值大于最右边值，说明 [left, mid] 有序。
            if (nums[left] <= target && target < nums[mid]) { // 注意点[left, mid)
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
    }
    return -1;
}
```

## 81. 搜索旋转排序数组 II

- [81. 搜索旋转排序数组 II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)
- 在上一题的基础上，增加了数组中**元素可重复**这个条件。

```java
public boolean search(int[] nums, int target) {
    if (nums == null || nums.length == 0)
        return false;
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] == target)
            return true;
        // 形如：[1, 0, 1, 1, 1]，去掉一个重复的干扰项。
        if (nums[left] == nums[mid]) {
            ++left;
            continue;
        }

        if (nums[mid] > nums[left]) {
            // 形如：[2, 3, 4, 5, 6, 7, 1]，左半部分 [left, mid] 有序。
            if (nums[left] <= target && target < nums[mid]) { // 局部有序，正经二分。
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } else {
            // 形如：[6, 7, 1, 2, 3, 4, 5]，有半部分 [mid, right] 有序。
            if (nums[mid] < target && target <= nums[right]) { // 局部有序，正经二分。
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    return false;
}
```

## 153. 寻找旋转排序数组中的最小值

- [153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

假设按照升序排序的数组在预先未知的某个点上进行了旋转。( 例如，数组 `[0,1,2,4,5,6,7]` 可能变为 `[4,5,6,7,0,1,2]` )。找出其中最小的元素。可以假设数组中不存在重复元素。

<!-- tabs:start -->

### **思路一**

```java
// 双百
public int findMin(int[] nums) {
    if (nums == null || nums.length == 0)
        throw new IllegalArgumentException("数组为空，不存在重复元素。");
	int res = nums[0];
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] < nums[right]) { // [mid, right] 有序，最小值在最左边。
            res = Math.min(nums[mid], res);
            right = mid - 1;
        } else { // [left, mid] 有序，最小值在最左边。
            res = Math.min(res, nums[left]);
            left = mid + 1;
        }
    }
    return res;
}
```

### **思路二**

```java
public int findMin(int[] nums) {
    if (nums == null || nums.length == 0)
        throw new IllegalArgumentException("数组为空，不存在重复元素。");
    int left = 0, right = nums.length - 1;
    while (left < right) { // 变化 1
        int mid = (left + right) >>> 1;
        if (nums[mid] < nums[right]) { // [mid, right] 有序，最小值在最左边。
            right = mid;  // 变化 2
        } else { // [left, mid] 有序，最小值在最左边。
            left = mid + 1;
        }
    }
    return nums[left]; // 变化 3
}
```

<!-- tabs:end -->

从本题中我们还应该学到的一点时，要在一个数组中查找某一个数 `x`，如果 `x` 不存在，那返回值应该是什么？

- 此时，我们不可以返回 `Integer.MIN_VALUE ~ Integer.MAX_VALUE` 之间的数，因为它确实是个整数，但也确实可能不是我们所寻找的 `target`。

- 所以，这个时候选择抛异常，应该是个比较不错的选择。

## 154. 寻找旋转排序数组中的最小值 II

- [154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)
- 在上一题的基础上，增加了 **元素可重复** 这个条件。

```java
public int findMin(int[] nums) {
    if (nums == null || nums.length == 0)
        throw new IllegalArgumentException("数组为空，不存在重复元素。");
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] < nums[right]) { // [mid, right] 有序
            right = mid;
        } else if (nums[mid] > nums[right]) {
            left = mid + 1;
        } else { // nums[mid] == nums[right]
            --right;
        }
    }
    // 退出 while 循环时 left == right，
    // 此时 [left, right] 包含且仅包含 1 个元素，
    // 而这个元素可能就是我们要找的答案。
    // 在本题中，一定是我们要找的答案。
    return nums[left];
}
```



---

> 二分答案系列

## 374. 猜数字大小

- [374. 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)

```java
/** 
 * Forward declaration of guess API.
 * @param  num   your guess
 * @return 	     -1 if num is lower than the guess number
 *			      1 if num is higher than the guess number
 *               otherwise return 0
 * int guess(int num);
 */
public int guessNumber(int n) {
    int left = 1, right = n;
    while (left <= right) {
        int mid = (left + right) >>> 1;
        if (guess(mid) == 0) {
            return mid;
        } else if (guess(mid) == 1) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return 0;
}
```

