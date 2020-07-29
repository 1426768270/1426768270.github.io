---
title: leetcode-链表
date: 2020-7-21 21:56:26
categories: 
- leetcode
tag:
- java
---

#  1.归并两个有序的链表

<!--more-->

21. Merge Two Sorted Lists

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    if (l1.val < l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```

# 2.从有序链表中删除重复节点

[83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

```java
 public ListNode deleteDuplicates(ListNode head) {
        ListNode current = head;
        while (current != null && current.next != null){
            if (current.next.val == current.val){
                current.next = current.next.next;
            }else {
                current = current.next;
            }
        }
        return head;
    }
```

# 3.删除链表的倒数第 n 个节点

[19. 删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

首先设立**预先指针 pre**，预先指针是一个小技巧，在第2题中进行了讲解

设预先指针 **pre 的下一个节点指向 head**，设前指针为 start，后指针为 end，二者都等于 pre

**start 先向前移动n步**

**之后 start 和 end 共同向前移动，此时二者的距离为 n**，**当 start 到尾部时，end 的位置恰好为倒数第 n 个节点**

因为要删除该节点，所以要移动到该节点的前一个才能删除，所以**循环结束条件为 start.next != null**

**删除后返回 pre.next，为什么不直接返回 head 呢，因为 head 有可能是被删掉的点**

时间复杂度：O(n)

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
         ListNode pre = new ListNode(0);
        pre.next=head;
        ListNode start=pre;
        ListNode end=pre;
        while(n!=0){
            start=start.next;
            n--;
        }
        while(start.next!=null){
            start=start.next;
            end=end.next;
        }
        end.next=end.next.next;
        return pre.next;
    }
```



# 4 两两交换链表中的节点

24(https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

```java
 public ListNode swapPairs(ListNode head) {
        ListNode per = new ListNode(0);
        per.next=head;

        ListNode tmp=per;
        //正序交换
        while(tmp.next!=null&&tmp.next.next!=null){

            ListNode start=tmp.next;
            ListNode end = tmp.next.next;

            tmp.next=end;
            start.next=end.next;
            end.next=start;

            tmp=tmp.next.next;
        }
        return per.next;
    }
```

# 5.相交链表

160(https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

编写一个程序，找到两个单链表相交的起始节点.

如果两个链表相交，那么相交点之后的长度是相同的

设 A 的长度为 a + c，B 的长度为 b + c，其中 c 为尾部公共部分长度，可知 a + c + b = b + c + a。

**当访问 A 链表的指针访问到链表尾部时，令它从链表 B 的头部开始访问链表 B；同样地，当访问 B 链表的指针访问到链表尾部时，令它从链表 A 的头部开始访问链表 A。这样就能控制访问 A 和 B 两个链表的指针能同时访问到交点。**

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode l1 = headA;
        ListNode l2 = headB;
        while (l1 != l2){
            l1 = (l1 == null) ? headB : l1.next;
            l2 = (l2 == null) ? headA : l2.next;
        }
        return l1;
    }
```

# 6.链表反转

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

反转一个单链表。

头插法

```java
public ListNode reverseList(ListNode head) {
        ListNode newhead = new ListNode(0);
        while (head!=null){
            ListNode next = head.next;
            head.next = newhead.next;
            newhead.next = head;
            head = next;
        }
        return newhead.next;
    }
```

递归

```java
public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null){
            return head;
        }
        ListNode next = head.next;
        ListNode newhead = reverseList(next);
        next.next = head;
        head.next = null;
        return newhead;
    }
```

# 7.链表求和

445 给你两个 **非空** 链表来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。

将数组压入栈中，在顺序取出

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    //用栈实现
    Stack<Integer> s1 = new Stack<Integer>();
    Stack<Integer> s2 = new Stack<Integer>();

    ListNode current1 = l1;
    ListNode current2 = l2;
    //数据进栈
    while (current1 != null){
        s1.push(current1.val);
        current1 = current1.next;
    }
    while (current2 != null){
        s2.push(current2.val);
        current2 = current2.next;
    }
    ListNode head = null;
    int c = 0 ;//进位
    while (!s1.isEmpty() || !s2.isEmpty() || c!=0){
        int sum = (s1.isEmpty()? 0: s1.pop())+(s2.isEmpty()? 0 : s2.pop())+c;
        ListNode node = new ListNode(sum % 10);
        c = sum / 10;
        node.next = head;
        head = node;
    }
    return head;
}
```