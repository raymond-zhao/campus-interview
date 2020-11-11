## 树结点定义

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    public TreeNode() {}
    public TreeNode(int val) {
        this.val = val;
    }
}
```

## 前序遍历非递归

```java
public void preorderTraversal(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode node = root;
    while (node != null || !stack.isEmpty()) {
    	while (node != null) {
            System.out.println(node.val + " ");
            stack.push(node);
            node = node.left;
        }
        if (!stack.isEmpty()) {
            node = stack.pop();
            node = node.right;
        }
    }
}
```

## 中序遍历非递归

```java
public void inorderTraversal(TreeNode root) {
	Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode node = root;
    while (node != null || !stack.isEmpty()) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
        if (!stack.isEmpty()) {
            node = stack.pop();
            System.out.println(node.val + " ");
            node = node.right;
        }
    }
}
```

## 后序遍历非递归

```java
public static void postorderTraversal(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode node = root, lastVisit = root;
    while (node != null || !stack.isEmpty()) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
        node = stack.peek(); // 查看当前栈顶元素
        if (node.right == null || node.right == lastVisit) {
            // 如果右子树已空或者已被访问过，满足了 左-右-根 中的左右，访问根。
            System.out.println(node.val + " ");
            stack.pop();
            lastVisit = node;
            node = null;
        } else {
            node = node.right;
        }
    }
}
```

## 层次遍历

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null)
        return res;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int sz = queue.size();
        List<Integer> curLevel = new ArrayList<>();
        for (int i = 0; i < sz; i++) {
            TreeNode node = queue.poll();
            assert node != null;
            curLevel.add(node.val);
            if (node.left != null)
                queue.offer(node.left);mkdir
            if (node.right != null)
                queue.offer(node.right);
        }
        res.add(curLevel);
    }
    return res;
}
```

## 数组存储的完全二叉树的层次遍历

```java
/**
* Amazon 笔试题
* 数组存储的完全二叉树的层次遍历，输出从 A ~ B 层间的结点。
* @param tree 数组表示的完全二叉树
* @param levelA 起始层
* @param levelB 结束层
* @return 搜索结果
*/
private static List<List<Integer>> bfs(int[] tree, int levelA, int levelB) {
    List<List<Integer>> res = new ArrayList<>();
    if (tree == null || tree.length == 0 || levelA > levelB) return res;
    Queue<Integer> queue = new LinkedList<>();
    queue.offer(0); // 存的是索引
    while (!queue.isEmpty()) {
        int sz = queue.size();
        List<Integer> curLevel = new ArrayList<>();
        for (int i = 0; i < sz; i++) {
            int rootIdx = queue.poll();
            curLevel.add(tree[rootIdx]);
            int leftIdx = 2 * rootIdx + 1; // 左子树索引
            int rightIdx = 2 * rootIdx + 2; // 右子树索引
            if (leftIdx <= tree.length - 1 && tree[leftIdx] != -1)
                queue.offer(leftIdx);
            if (rightIdx <= tree.length && tree[rightIdx] != -1)
                queue.offer(rightIdx);
        }
        res.add(curLevel);
    }
    List<List<Integer>> ans = new ArrayList<>();
    for (int i = levelA; i < levelB + 1; i++)
        ans.add(res.get(i - 1));
    return ans;
}
```

## Z 字型遍历

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        LinkedList<Integer> curLevel = new LinkedList<>();
        int sz = queue.size();
        for (int i = 0; i < sz; i++) {
            TreeNode node = queue.poll();
            if ((res.size() & 1) == 0) {
                // 结果集中包含偶数个元素，说明下一层应该是奇数层。
                curLevel.addLast(node.val);
            } else {
                // 结果集中包含奇数个元素，说明下一层应该是偶数层。
                curLevel.addFirst(node.val);
            }
            if (node.left != null)
                queue.offer(node.left);
            if (node.right != null)
                queue.offer(node.right);
        }
        res.add(curLevel);

    }
    return res;
}
```

## 垂序遍历

## N 叉树定义

```java
public class Node {
    
    public int val;
    
    public List<Node> children;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, List<Node> _children) {
        val = _val;
        children = _children;
    }
}
```

## N 叉树的前序遍历

<!-- tabs:start -->

#### **递归法**

```java
public class Solution {
    
	private List<Integer> res = new ArrayList<>();
    
    public List<Integer> preorder(Node root) {
        dfs(root);
        return res;
    }
    
    private void dfs(Node node) {
        if (node == null)
            return;
        res.add(node.val);
        for (Node child : node.children) {
            dfs(child);
        }
    }
}
```

#### **迭代法**

```java
public class Solution {
    public List<Integer> preorder(Node root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) return res;
        Deque<Node> stack = new ArrayDeque<>();
        stack.push(root);
        while (!stack.isEmpty()) {
            Node node = stack.pop();
            res.add(node.val);
            Collections.reverse(node.children);
            for (Node child : node.children) {
                stack.push(child);
            }
        }
        return res;
    }
}
```

<!-- tabs:end -->

## N 叉树的中序遍历



## N 叉树的后序遍历

## N 叉树的层序遍历