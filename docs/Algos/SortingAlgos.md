## 出处

> [十大排序算法总结](https://raymond-zhao.top/2020/03/19/2020-03-19-Algo-Sorting/)

## 十大排序算法总结

| 分类 |   算法名称   |     平均      |     最好     |    最坏     | 空间复杂度 | 稳定性 |
| :--: | :----------: | :-----------: | :----------: | :---------: | :--------: | :----: |
| 插入 |   直接插入   |   $O(n^2)$    |    $O(n)$    |  $O(n^2)$   |   $O(1)$   |  $Y$   |
| 插入 |   折半插入   |   $O(n^2)$    | $O(n\lg n)$  |  $O(n^2)$   |   $O(1)$   |  $Y$   |
| 插入 |   希尔排序   |  $O(n\lg n)$  | $O(n^{1.3})$ |  $O(n^2)$   |   $O(1)$   |  $N$   |
| 交换 |   冒泡排序   |   $O(n^2)$    |    $O(n)$    |  $O(n^2)$   |   $O(1)$   |  $Y$   |
| 交换 |   快速排序   |  $O(n\lg n)$  | $O(n\lg n)$  |  $O(n^2)$   | $O(\lg n)$ |  $N$   |
| 选择 | 简单选择排序 |   $O(n^2)$    |   $O(n^2)$   |  $O(n^2)$   |   $O(1)$   |  $N$   |
| 选择 |    堆排序    |  $O(n\lg n)$  |    $O(n)$    | $O(n\lg n)$ |   $O(1)$   |  $N$   |
| 归并 |   二路归并   |  $O(n\lg n)$  | $O(n\lg n)$  | $O(n\lg n)$ |   $O(n)$   |  $Y$   |
| 线性 |   基数排序   |  $O(d(n+r))$  | $O(d(n+r))$  | $O(d(n+r))$ |  $O(r+n)$  |  $Y$   |
| 线性 |   计数排序   | $\Theta(n+k)$ |   $O(n+k)$   |  $O(n+k)$   |   $O(k)$   |  $Y$   |
| 线性 |    桶排序    |   $O(n+k)$    |    $O(n)$    |  $O(n^2)$   |  $O(n*k)$  |  $Y$   |

- 平均而言，快速排序最佳，最坏情况下不如堆排序与归并排序。
- 直接插入排序，折半插入排序，简单选择排序，冒泡排序等时间复杂度为 $O(n^2)$ 的排序不适用与 $n$ 较大的情况。
- 一趟排序至少能保证一个关键字到达最终位置的排序：交换类(冒泡排序，快速排序)，选择类排序(简单选择排序，堆排序)。
- 关键字比较次数和原始序列无关的排序：简单选择排序，折半插入排序，归并排序，基数排序。
- 排序趟数和原始序列有关的排序：交换类排序(冒泡排序，快速排序)。
- 排序趟数与原始序列无关的排序：直接插入排序，折半插入排序，希尔排序，堆排序，二路归并排序，简单选择排序，基数排序。
- 借助于***比较***进行排序的算法，最坏时至少为 $O(n\lg n)$ 。
- 直接插入和折半插入的区别仅在于查找插入位置的方式不同。
- 元素的移动次数与原始序列无关的排序：基数排序。
- 不稳定排序：快速排序，希尔排序，简单选择排序，堆排序。
- 时间复杂度为 $O(n\lg n)$ 的排序：快速排序，希尔排序，二路归并排序，堆排序。
- 计数排序不是比较排序，因此不受 $\Omega(n\lg n)$ 的限制。

## 插入排序

```java
public int[] insertionSort(int[] array) {
    if (array.length == 0) return array;
    for (int i = 1; i < array.length; i++) {
        // 先保存要插入的值，然后寻找它需要插入的位置
        int key = array[i];
        // 将array[j] 插入到已经排好序的 array[0..i-1]中
        // 当前值的前一个值
        int j = i - 1;
        while (j >= 0 && array[j] > key) {
            // 本次循环的作用是找到属于key的位置
            // 如果key前面的值大于key的话，则交换二者的位置
            array[j + 1] = array[j];
            j--;
        }
        // 最后将保存的key放到属于它的位置上
        array[j + 1] = key;
    }
    return array;
}
```

## 折半插入

```java
public static int[] binaryInsertionSort(int[] array) {
    if (array.length == 0) return array;
    for (int i = 1; i < array.length; i++) {
        // 先保存要插入的值
        int key = array[i];
        int left = 0, right = i - 1;
        // 然后寻找它需要插入的位置
        while (left <= right) {
            int middle = (left + right) / 2;
            if (array[middle] < key) {
                left = middle + 1;
            } else {
                right = middle - 1;
            }
        }
        // 将 left 后边的数组整体后移一位
        for (int j = i; j > left; j--) {
            array[j] = array[j - 1];
        }

        array[left] = key;
    }
    return array;
}
```

## 希尔排序

```java
// 步长 n / 2
public static int[] shellSort(int[] array) {
    int length = array.length;
    if (length == 0) return array;
    int temp;
    for (int step = length / 2; step >= 1; step /= 2) {
        // 步长每次取为分组长度的 1/2
        for (int i = step; i < length; i++) {
            // 组内排序
            temp = array[i];
            int j = i - step;
            while (j >= 0 && array[j] > temp) {
                array[j + step] = array[j];
                j -= step;
            }
            array[j + step] = temp;
        }
    }
    return array;
}
```

## 冒泡排序

```java
public int[] bubbleSort(int[] array) {
    int temp;
    for (int i = 0; i < array.length - 1; i++) {
        // 从第一个到倒数第二个元素
        for (int j = 0; j < array.length - 1 - i; j++) {
            // 每轮冒泡结束后至少有一个元素到达最终位置，所以比较的元素个数依次为
            // n - 1、n - 2、n - 3 ...
            if (array[j] > array[j + 1]) {
                // 每次取得两个数之间较大的数
                temp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = temp;
            }
        }
    }
    return array;
}
```

## 快速排序

```java
public int[] quickSort(int[] array, int p, int r) {
    if (p < r) {
        int q = partition(array, p, r);
        quickSort(array, p, q - 1);
        quickSort(array, q + 1, r);
    }
    return array;
}

// v1
private int partition(int[] array, int p, int r) {
    int x = array[r];
    int i = p - 1;
    for (int j = p; j < r; j++) {
        if (array[j] <= x) {
            i++;
            swap(array, i, j);
        }
    }
    swap(array, i + 1, r);
    return i + 1;
}

private void swap(int[] a, int i, int j) {
    int temp = a[i];
    a[i] = a[j];
    a[j] = temp;
}

/**
* v2
* 左边的都小于等于 pivot，右边的都大于pivot
* @param arr 待排序数组
* @param l 数组左边界
* @param r 数组右边界
* @return 枢轴索引
*/
private int partition(int[] arr, int l, int r) {
    int x = arr[l]; //  选左
    int i = l, j = r;
    while (i < j) {
        while (arr[j] > x && i < j) --j;
        while (arr[i] <= x && i < j) ++i;
        swap(arr, i, j);
    }
    swap(arr, l, i);
    return i;
}

/**
* v3
* 左边的都小于等于 pivot，右边的都大于pivot
* @param arr 待排序数组
* @param l 数组左边界
* @param r 数组右边界
* @return 枢轴索引
*/
private int partition3(int[] arr, int l, int r) {
    int x = arr[r]; // 选右
    int i = l, j = r;
    while (i < j) {
        while (arr[i] < x && i < j) ++i;
        while (arr[j] >= x && i < j) --j;
        swap(arr, i, j);
    }
    swap(arr, i, r);
    return i;
}
```

## 简单选择排序

```java
public int[] selectionSort(int[] arr) {
    if (arr == null || arr.length == 0) return new int[0];
    int min, minIdx; // 保存最小值及其索引
    for (int i = 0; i < arr.length; i++) {
        min = arr[i]; // 保存最小值
        minIdx = i; // 保存最小值的索引
        for (int j = i + 1; j < arr.length; j++) { // 遍历当前位置之后的所有元素
            if (arr[j] < min) {
                min = arr[j]; // 更新最小值
                minIdx = j; // 更新最小值的索引
            }
        }
        // 等到找到最小值以后，将当前位置的值与最小值进行交换
        if (minIdx != i) {
            int tmp = arr[i];
            arr[i] = arr[minIdx];
            arr[minIdx] = tmp;
        }
    }
    return arr;
}
```

## 堆排序

```java
public class HeapSort {
    private int[] arr;
    public HeapSort(int[] arr) {
        this.arr = arr;
    }

    /**
     * 堆排序的主要入口方法，共两步。
     */
    public void sort() {
        /*
         *  第一步：将数组堆化
         *  beginIndex = 第一个非叶子节点。
         *  从第一个非叶子节点开始即可。无需从最后一个叶子节点开始。
         *  叶子节点可以看作已符合堆要求的节点，根节点就是它自己且自己以下值为最大。
         */
        int len = arr.length - 1;
        int beginIndex = (arr.length >> 1)- 1;
        for (int i = beginIndex; i >= 0; i--)
            maxHeapify(i, len);
        /*
         * 第二步：对堆化数据排序
         * 每次都是移出最顶层的根节点A[0]，与最尾部节点位置调换，同时遍历长度 - 1。
         * 然后从新整理被换到根节点的末尾元素，使其符合堆的特性。
         * 直至未排序的堆长度为 0。
         */
        for (int i = len; i > 0; i--) {
            swap(0, i);
            maxHeapify(0, i - 1);
        }
    }

    private void swap(int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    /**
     * 调整索引为 index 处的数据，使其符合堆的特性。
     *
     * @param index 需要堆化处理的数据的索引
     * @param len 未排序的堆（数组）的长度
     */
    private void maxHeapify(int index, int len) {
        int li = (index << 1) + 1; // 左子节点索引
        int ri = li + 1;           // 右子节点索引
        int cMax = li;             // 子节点值最大索引，默认左子节点。
        if (li > len) return;      // 左子节点索引超出计算范围，直接返回。
        if (ri <= len && arr[ri] > arr[li]) // 先判断左右子节点，哪个较大。
            cMax = ri;
        if (arr[cMax] > arr[index]) {
            swap(cMax, index);      // 如果父节点被子节点调换，
            maxHeapify(cMax, len);  // 则需要继续判断换下后的父节点是否符合堆的特性。
        }
    }
}
```

## 归并排序

```java
private static int[] merge(int[] array, int low, int mid, int high) {
    int[] temp = new int[high - low + 1];
    int i = low; // 左指针
    int j = mid + 1; // 右指针
    int k = 0;
    // 把较小的数先移到新数组中
    while (i <= mid && j <= high) {
        if (array[i] < array[j]) {
            temp[k++] = array[i++];
        } else {
            temp[k++] = array[j++];
        }
    }
    // 把左边剩余的数移入数组
    while (i <= mid) {
        temp[k++] = array[i++];
    }
    // 把右边边剩余的数移入数组
    while (j <= high) {
        temp[k++] = array[j++];
    }
    // 把新数组中的数覆盖nums数组
    for (int k2 = 0; k2 < temp.length; k2++) {
        array[k2 + low] = temp[k2];
    }
    return array;
}

public static int[] mergeSort(int[] array, int low, int high) {
    int mid = (low + high) / 2;
    if (low < high) {
        mergeSort(array, low, mid);
        mergeSort(array, mid + 1, high);
        merge(array, low, mid, high);
    }
    return array;
}
```

