## 相交链表

- [相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;
    ListNode romeo = headA, juliet = headB;
    while (romeo != juliet) {
        romeo = romeo == null ? headB : romeo.next;
        juliet = juliet == null ? headA : juliet.next;
    }
    return romeo;
}
```

> 最浪漫的一道题：听闻远方有你，动身跋涉千里。我吹过你吹过的风，这算不算相拥？我走过你走过的路，这算不算相逢？

## 环形链表

- [环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

```java
public boolean hasCycle(ListNode head) {
    if (head == null) return false;
    ListNode fast = head, slow = head;
    while (true) {
        if (fast == null || fast.next == null)
            return false;
        fast = fast.next.next;
        slow = slow.next;
        if (fast == slow)
            return true;
    }
}
```

## 环形链表 - 2

- [环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

```java
public ListNode detectCycle(ListNode head) {
    if (head == null) return head;
    ListNode fast = head, slow = head;
    while (true) {
        if (fast == null || fast.next == null)
            return null;
        fast = fast.next.next;
        slow = slow.next;
        if (fast == slow)
            break;
    }
    fast = head;
    while (fast != slow) {
        fast = fast.next;
        slow = slow.next;
    }
    return fast;
}
```

