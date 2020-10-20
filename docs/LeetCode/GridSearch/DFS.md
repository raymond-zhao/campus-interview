## 329. 矩阵中的最长递增路径

- [329. 矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)
- 方法：记忆化深度优先搜索

```java
class Solution {
    
    private int[][] directions = {{-1, 0}, {1, 0}, {0, 1}, {0, -1}};
    private int rows, cols;
    
    public int longestIncreasingPath(int[][] matrix) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0)
            return 0;
        rows = matrix.length;
        cols = matrix[0].length;
        
        int[][] memo = new int[rows][cols];
        
        int res = 0;
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                int cur = dfs(matrix, i, j, memo);
                res = Math.max(res, cur);
            }
        }
        return res;
    }
    
    private int dfs(int[][] matrix, int row, int col, int[][] memo) {
        if (memo[row][col] != 0)
            return memo[row][col];
        ++memo[row][col];
        for (int[] direction : directions) {
            int newRow = row + direction[0], newCol = col + direction[1];
            if (inArea(matrix, newRow, newCol) && matrix[newRow][newCol] > matrix[row][col])
                memo[row][col] = Math.max(memo[row][col], dfs(matrix, newRow, newCol, memo) + 1);
        }
        return memo[row][col];
    }
    
    private boolean inArea(int[][] matrix, int i, int j) {
        return 0 <= i && i < matrix.length && 0 <= j && j < matrix[0].length;
    }
}
```

