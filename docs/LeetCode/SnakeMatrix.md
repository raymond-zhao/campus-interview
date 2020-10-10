## 起源：螺旋矩阵

[螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/)

```java
public int[] spiralOrder(int[][] matrix) {
    if (matrix == null || matrix.length == 0) return new int[0];
    int top = 0, bottom = matrix.length - 1, left = 0, right = matrix[0].length - 1;
    int x = 0;
    int[] res = new int[(bottom + 1) * (right + 1)];
    while (true) {
        // 从左向右
        for (int i = left; i <= right; i++) res[x++] = matrix[top][i];
        if (++top > bottom) break;
        // 自上而下
        for (int i = top; i <= bottom; i++) res[x++] = matrix[i][right];
        if (--right < left) break;
        // 从右向左
        for (int i = right; i >= left; i--) res[x++] = matrix[bottom][i];
        if (--bottom < top) break;
        // 自下而上
        for (int i = bottom; i >= top; i--) res[x++] = matrix[i][left];
        if (++left > right) break;
    }
    return res;
}
```

## Main 函数

```java
public static void main(String[] args) {
    int n = 6;
    int[][] clockwise = clockwise(n);
    System.out.println("-----------顺时针-----------");
    for (int[] row : clockwise) {
        for (int val : row) {
            System.out.print(val + "\t");
        }
        System.out.println();
    }

    int[][] anticlockwise = anticlockwise(n);
    System.out.println("-----------逆时针-----------");
    for (int[] row : anticlockwise) {
        for (int val : row) {
            System.out.print(val + "\t");
        }
        System.out.println();
    }

    int[][] matrix = generateMatrix(n);
    System.out.println("-----------顺逆时针-----------");
    for (int[] row : matrix) {
        for (int val : row) {
            System.out.print(val + "\t");
        }
        System.out.println();
    }
}
```

## 顺时针打印矩阵

```java
/**
* 顺时针打印蛇形矩阵
* @param n
* @return
-----------顺时针-----------
1	2	3	4	5	6	
20	21	22	23	24	7	
19	32	33	34	25	8	
18	31	36	35	26	9	
17	30	29	28	27	10	
16	15	14	13	12	11
*/
private static int[][] clockwise(int n) {
    int top = 0, bottom = n - 1, left = 0, right = n - 1;
    int[][] matrix = new int[n][n];
    int counter = 1, limit = n * n;
    while (counter <= limit) {
        // 从左到右
        for (int i = left; i <= right; i++) matrix[top][i] = counter++;
        ++top;

        // 从上到下
        for (int i = top; i <= bottom; i++) matrix[i][right] = counter++;
        --right;

        // 从右向左
        for (int i = right; i >= left; i--) matrix[bottom][i] = counter++;
        --bottom;

        // 自下而上
        for (int i = bottom; i >= top; i--) matrix[i][left] = counter++;
        ++left;
    }
    return matrix;
}
```

## 逆时针打印矩阵

```java
/**
* 逆时针打印蛇形矩阵
* @param n
* @return
-----------逆时针-----------
1	20	19	18	17	16	
2	21	32	31	30	15	
3	22	33	36	29	14	
4	23	34	35	28	13	
5	24	25	26	27	12	
6	7	8	9	10	11
*/
private static int[][] anticlockwise(int n) {
    int top = 0, bottom = n - 1, left = 0, right = n - 1;
    int[][] matrix = new int[n][n];
    int counter = 1;
    while (true) {
        // 从上到下
        for (int i = top; i <= bottom; i++) matrix[i][left] = counter++;
        if (++left > right) break;

        // 从左到右
        for (int i = left; i <= right; i++) matrix[bottom][i] = counter++;
        if (--bottom < top) break;

        // 自下而上
        for (int i = bottom; i >= top; i--) matrix[i][right] = counter++;
        if (--right < left) break;

        // 从右向左
        for (int i = right; i >= left; i--) matrix[top][i] = counter++;
        if (++top > bottom) break;
    }
    return matrix;
}
```

## 顺逆时针交替打印矩阵

> 网易互娱笔试题

```java
/**
* 1 ~ N^2 之间的数
*
* @param n 维度
* @return 奇数层 --bottom < top
* 偶数层 --right > left

-----------顺逆时针-----------
1	2	3	4	5	6	
20	21	32	31	30	7	
19	22	33	34	29	8	
18	23	36	35	28	9	
17	24	25	26	27	10	
16	15	14	13	12	11
*/
private static int[][] generateMatrix(int n) {
    boolean clockwise = true;
    int top = 0, bottom = n - 1, left = 0, right = n - 1;
    int[][] matrix = new int[n][n];
    int counter = 1;
    while (counter <= n * n) {
        if (clockwise) {
            for (int i = left; i <= right; i++) matrix[top][i] = counter++;
            ++top;
            // 从上到下
            for (int i = top; i <= bottom; i++) matrix[i][right] = counter++;
            --right;
            // 从右向左
            for (int i = right; i >= left; i--) matrix[bottom][i] = counter++;
            --bottom;
            // 自下而上
            for (int i = bottom; i >= top; i--) matrix[i][left] = counter++;
            ++left;
        } else {
            // 从上到下
            for (int i = top; i <= bottom; i++) matrix[i][left] = counter++;
            ++left;
            // 从左到右
            for (int i = left; i <= right; i++) matrix[bottom][i] = counter++;
            --bottom;
            // 自下而上
            for (int i = bottom; i >= top; i--) matrix[i][right] = counter++;
            --right;
            // 从右向左
            for (int i = right; i >= left; i--) matrix[top][i] = counter++;
            ++top;
        }
        clockwise = !clockwise;
    }
    return matrix;
}
```

