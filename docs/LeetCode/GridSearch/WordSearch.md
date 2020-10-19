## 79. 单词搜索

- [79. 单词搜索](https://leetcode-cn.com/problems/word-search/)

```java
public boolean exist(char[][] board, String word) {
    if (board == null || board.length == 0 || board[0].length == 0 || word == null || word.length() == 0)
        return false;
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[0].length; j++) {
            if (board[i][j] == word.charAt(0)) {
                if (dfs(board, i, j, 0, word.toCharArray())) {
                    return true;
                }
            }
        }
    }
    return false;
}

private boolean dfs(char[][] board, int i, int j, int idx, char[] words) {
    if (!inArea(board, i, j) || board[i][j] != words[idx])
        return false;

    if (idx == words.length - 1)
        return true;

    char c = board[i][j];
    board[i][j] = '/';
    boolean res = dfs(board, i + 1, j, idx + 1, words) || dfs(board, i - 1, j, idx + 1, words)
        || dfs(board, i, j + 1, idx + 1, words) || dfs(board, i, j - 1, idx + 1, words);
    board[i][j] = c;

    return res;
}

private boolean inArea(char[][] grid, int i, int j) {
    return 0 <= i && i < grid.length && 0 <= j && j < grid[0].length;
}
```

