## 合并两个有序链表

- [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

```java
// 迭代
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(-1);
    ListNode pre = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            pre.next = l1;
            l1 = l1.next;
        } else {
            pre.next = l2;
            l2 = l2.next;
        }
        pre = pre.next;
    }
    pre.next = l1 == null ? l2 : l1;
    return dummy.next;
}

// 递归
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    else if (l2 == null) return l1;
    else if (l1.val < l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```

## 合并 K 个升序链表

- [合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

> 方法一：先把所有结点的值添加到辅助列表里，然后 Collections.sort() 排序，最后重造。
>
> 面试时不可取。

```java
public ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    List<Integer> helper = new ArrayList<>();
    for (ListNode listNode : lists) {
        while (listNode != null) {
            helper.add(listNode.val);
            listNode = listNode.next;
        }
    }
    Collections.sort(helper);
    ListNode dummy = new ListNode(-1);
    ListNode pre = dummy;
    for (Integer x : helper) {
        pre.next = new ListNode(x);
        pre = pre.next;
    }
    return dummy.next;
}
```

> 方法二：优先队列 => 小根堆

```java
/**
* 构建小根堆，然后把 lists 中每个链表的头结点先放入；
* 迭代进行：把小根堆堆顶加入结果集，并且把队顶元素的下一个结点加入小根堆中；
* 直到 小根堆 为空。
*/
public ListNode mergeKLists(ListNode[] lists) {
    // 小根堆：Queue<ListNode> queue = new PriorityQueue<>((o1, o2) -> o1.val - o2.val);
    // 大根堆：Queue<ListNode> queue = new PriorityQueue<>((o1, o2) -> o2.val - o1.val);
    Queue<ListNode> queue = new PriorityQueue<>(Comparator.comparingInt(o -> o.val));
    for (ListNode node : lists) {
        if (node != null)
            queue.offer(node);
    }
    // 构建返回结果伪头结点
    ListNode dummy = new ListNode(-1);
    ListNode pre = dummy;
    
    while (!queue.isEmpty()) {
        ListNode minNode = queue.poll();
        pre.next = minNode;
        pre = minNode;
        // 因为添加进堆的时候已经判断 node 不为空，所以这里不需要 minNode != null 的判断
        if (minNode.next != null)
            queue.offer(minNode.next);
    }
    return dummy.next;
}
```

> 方法三：分治法

```java

public ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    return merge(lists, 0, lists.length - 1)l
}

public ListNode merge(ListNode[] lists, int left, int right) {
    if (left == right) return lists[left];
    int mid = left + (right - left) / 2;
    ListNode l = merge(lists, left, mid);
    ListNode r = merge(lists, mid + 1, right);
    return mergeTwoLists(l, r);
}

public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    else if (l2 == null) return l1;
    else if (l1.val < l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```