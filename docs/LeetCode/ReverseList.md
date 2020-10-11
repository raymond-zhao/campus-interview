## 反转链表

- [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

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

// recursive
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return null;
    ListNode p = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return p;
}
```

## 反转链表 II

- [反转链表 II - 反转 m~n 之间的链表](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

```java
public ListNode reverseBetween(ListNode head, int m, int n) {
	// 先用两个指针定位 m~n 之间的链表，同时保存 m-1, n+1 位置处的结点
    
    // 然后调用上面的反转链表的方法。
}
```