## 反转整个链表

- [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

<!-- tabs:start -->
# **迭代**
```java
public ListNode reverseList(ListNode head) {
    if (head == null) return null;
    ListNode pre = null, cur = head, tmpNext;
    while (cur != null) {
        tmpNext = cur.next;
        cur.next = pre;
        pre = cur;
        cur = tmpNext;
    }
    return pre;
}
```

# **递归**
```java
// src = [1->2->3->4->5->6->null]
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    // 在 reverseList 之后变成
    // 		head 		    last
    // 		  |	 			 |
    // 		  ∨	 			 v
    // src = [1->2<-3<-4<-5<-6]
    // 			 |
    //			 ∨
    //			null
    ListNode last = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return last;
}
```
<!-- tabs:end -->
## 反转链表的前 K 个结点

> [『LeetCode』 上无此题，在上一题的基础上更改，为下一题做准备。](https://leetcode-cn.com/problems/reverse-linked-list-ii/solution/bu-bu-chai-jie-ru-he-di-gui-di-fan-zhuan-lian-biao/)

```java
ListNode successor = null;

// src = [1->2->3->4->5->6->null]， k = 3
public ListNode reverseN(ListNode head, int k) {
    // 递归终止条件
    if (k == 1) {
        successor = head.next; // 记录第 k+1 个结点
        return head;
    }
	// head last successor
    // |	 |   |
    // ∨	 v  /
    // 1->2<-3 4->5->6->null，successor=4
    //    |
    //    ∨
    //	 null
    ListNode last = reverseN(head.next, n - 1); // last=3，head=1
    head.next.next = head;
    // 让反转之后的 head 结点和后面的结点连接起来
    head.next = successor;
    return last;
}
```

**与上一题的区别：**

- 递归终止条件变为 `k==1`，当只剩一个元素时反转自身，同时记录**后继结点 successor**；
- 把 `head.next = null` 变为 `head.next = successor`，因为在上题中反转整个链表，原来的 `head` 结点变为链表中的最后一个结点，所以需要将后继结点置为 `null`，而在本题中并不一定反转整个链表。

## 反转链表 m~n 之间的结点

- [反转链表 II - 反转 m~n 之间的链表](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

<!-- tabs:start -->

# ** 迭代 **
```java
public ListNode reverseBetween(ListNode head, int m, int n) {
    if (head == null || head.next == null) return head;
    ListNode pre = null, cur = head;
    while (m-- > 1) {
        pre = cur;
        cur = cur.next;
        n--;
    }
    ListNode con = pre, tail = cur, tmp;
    while (n-- > 0) {
        tmp = cur.next;
        cur.next = pre;
        pre = cur;
        cur = tmp;
    }
    // [3, 5], 1, 1 => 如果不判空会 NPE，因为 pre 初始化为 null，
    // 如果只反转第一个结点，pre 不向后移动将一直是 null。
    if (con != null) con.next = pre;
    else head = pre;

    tail.next = cur;
    return head;
}
```

# ** 递归 **
```java
public ListNode reverseBetween(ListNode head, int m, int n) {
	if (m == 1)
        return reverseN(head, n); // 这个函数的函数体在上一题
    // 前进到反转的起点触发 终止条件
    head.next = reverseBetween(head.next, m - 1, n - 1);
    return head;
}
```
<!-- tabs:end -->

## K 个一组反转链表

- [K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(-1);
    dummy.next = head;
    ListNode pre = dummy, end = dummy;
    while (end != null || end.next != null) {
        for (int i = 0; i < k && end != null; i++) end = end.next;
        if (end == null) break;
        ListNode start = pre.next, tmpNext = end.next;
        end.next = null;

        pre.next = reverse(start); // 与第一题逻辑相同
        // 反转之后 start 已变成了链表最后一个结点，所以接起来。
        start.next = tmpNext;

        pre = start;
        end = start;
    }
    return dummy.next;
}

public ListNode reverse(ListNode head) {
    if (head == null) return null;
    ListNode pre = null, cur = head, tmpNext;
    while (cur != null) {
        tmpNext = cur.next;
        cur.next = pre;
        pre = cur;
        cur = tmpNext;
    }
    return pre;
}
```

