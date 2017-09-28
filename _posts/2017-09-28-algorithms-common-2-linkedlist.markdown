---
layout: post
title:  常见算法基础题思路简析(二)-链表篇
date:   2017-09-28 21:41:00 +08:00
category: 算法和数据结构
tags: 算法 数据结构
comments: true
---

* content
{:toc}


本文对和 **链表** 有关的常见算法基础题思路分类进行分析和总结，并以 Java 为例，适当指出需要注意的编程细节






- 相关题目和代码在 GitHub: [https://github.com/brianway/algorithms-learning](https://github.com/brianway/algorithms-learning)
- 题目见 `com.brianway.learning.algorithms.lectures.linkedlist`包


**记号约定：链表未特殊说明则未单链表**

## 判断一个单链表是否有环

题目见 `ChkLoop`

思路：

1. 判断环：使用快慢两个指针遍历链表，快指针一次移动两个节点，慢指针一次移动一个节点。终止条件为遍历到链表结尾或者快指针和慢指针相遇
2. 找入环第一个节点：假如快慢指针相遇，则此时将慢指针指向头节点，然后均以每次一个节点的速度同步移动快慢指针，再次相遇点即为入环点

注意：

- 快指针移动时需要判断 next 和 next.next 均不为空才行
- 第二步的证明可以参考
    - [http://blog.csdn.net/wuzhekai1985/article/details/6725263](http://blog.csdn.net/wuzhekai1985/article/details/6725263)
    - [http://blog.sina.com.cn/s/blog_6a0e04380101a9o2.html](http://blog.csdn.net/wuzhekai1985/article/details/6725263)

## 判断两无环单链表是否相交

题目见 `CheckIntersect`

思路：

1. 分别遍历两个链表，得到长度 l1 和 l2
2. 将较长的链表先遍历 `|l1-l2|` 个节点，再对两个链表同步遍历
3. 若在遍历完前有相同节点，则相交，否则不相交

注意：

- 用新的变量指向链表，不要影响原链表结构


## 判断两有环单链表是否相交

题目见 `ChkIntersection`

思路：

1. 先判断两链表是否均有环，有一个无环则不相交
2. 若入环节点一样，则相交
3. 若入环节点不一样，则从一个入环点往后遍历，若能和另一个链表的入环点相遇则相交，若遍历回自己的入环点还没相遇，则不相交

注意：

- 第 3 步里从一个入环点往后遍历，注意起始条件

## 判断两单链表是否相交

题目见 `ChkIntersection2`

思路：为前面几题的整合

1. 先判断两链表是否均有环
2. 若均无环，按照“判断两无环单链表是否相交”来解
3. 若均有环，按照“判断两有环单链表是否相交”来解
4. 若一个有环，一个无环，则不相交


## 删除特定值的节点

题目见 `ClearValue`

思路：

1. 找到第一个不为特定值的节点作为新的头节点
2. 使用双指针 pre 和 current，current 用于依次遍历，pre 用于跳过为特定值的节点，只依次经过满足条件的其余节点

注意：

- 一定要循环跳过头节点里为特定值的节点，第二步必须以非特定值节点或者空节点开始
- pre 和 current 的初始化以及移动赋值细节要留意


## 打印两个升序链表公共部分

题目见 `Common`

思路：

新建一个数组用于容纳公共部分，同时遍历两个链表，根据当前的节点值分情况进行处理：

1. 一个链表当前节点值大于另一个链表当前节点值，则较小值的链表节点向后移动一步
2. 两个链表当前值相等，将公共部分存入数组，两个链表均向后移动一步

## 复制复杂链表

题目见 `CopyList`

思路：

1. 在每个节点的后面插入该节点的直接拷贝
2. 对每个 copy 节点，更新其 random 指针为 random 指向节点的下一个节点(即指向其对应 copy 节点)
3. 将 copy 节点从中分离出来即可

注意：

- 该题的难点主要在于每个节点的随机指针不是简单的直接拷贝就行了（那样会指向原链表中的节点），而需要在复制的链表里指向对应的复制节点
- 无论是插入拷贝还是分离节点，都要注意空指针判断和更新指针操作


## 链表partition

题目见 `Divide`

思路：

1. 使用两个链表分别收集大于阈值和小于等于阈值的节点
2. 每次取出当前节点，归入其中一个链表，原链表向后移动一个
2. 将两个链表首尾相连即可

注意：

- 两个链表的首尾节点都要记录
- 大于/小于等于的划分要全覆盖
- 取出的节点记得“断尾”，即  next 置为 null

## 有序环链表中插入值

题目见 `InsertValue`

思路：

1. 以一个节点(head)开始，用两个指针指向前后相邻的节点依次向后遍历，满足 `pre.val <= val < after.val` 则插入并停止遍历
2. 若遍历完(`after == head`)，说明还没插入，此时应插入在 head 前
3. 若 val 比 head.val 小，则更新头节点

注意：

- 注意空链表时该节点自己组环
- 插入节点时判断条件，前后的开闭要注意，`[pre,after)` 还是 `(pre,after]`
- 注意特殊情况，此时不满足 `pre.val<=val<after.val`（例如环全等，插入值是最小/最大等），这时需要插入在头节点之前及更新头节点


## 链表K逆序

题目见 `KInverse`

思路：

1. 用 kHead 指向每次 k 个节点的头节点（初始值为头节点）
2. 遍每计数到 k 时，逆序这部分并返回逆序后的头节点
    - 若新的头节点为空，则初始化新的头节点
    - 否则，将该返回的头节点接在上次逆序完的尾节点 lastTail 后
3. 更新 lastTail 指向该次逆序完的尾节点(即逆序前的 kHead)
4. 更新 kHead 指向下一个节点继续重复上述步骤直到遍历结束

逆序部分：

1. 使用 pre, current, next 分别指向当前节点及其前后节点
2. 每次先用 pre 保存 newHead，用 next 保存 current.next；再更新 newhead 为 current，更新 newHead.next 指向 pre；最后移动 current 指向 next 即可
3. 进一步优化，其实只需要 pre 和 current 两个指针就够了

```java
//普通反转整个链表
public ListNode reverse(ListNode head) {
    ListNode newHead = null;
    ListNode current = head;
    ListNode pre = null;

    while(current != null){
        pre = newHead;
        newHead = current;
        current = current.next;
        newHead.next = pre;
    }
    return newHead;
}
```

注意：

- 逆序前需要用 next 指针保留好下一个节点，这样才能正确更新 kHead
- 注意循环中计数和清零的位置
- 逆序部分，需要把之前的头的 next 置为原末尾的 next，衔接起来


## 检查链表是否回文

题目见 `Palindrome`

思路：

1. 通过快慢指针得到链表中点
2. 将后一半链表逆序
3. 分别从头尾向中间遍历链表依次比较值，不等则停止，看能否在中点相遇
4. 将逆序链表还原


注意：

- 注意逆序需要根据链表总节点数的分单双讨论
    - 单数(`quick.next = null`)则 slow 指向正中间的节点，所以设置 `mid = slow`
    - 双数时 slow 指向前一半的最后一个节点，所以设置 `mid = slow.next`
- 双数时从两头遍历前需要将 slow.next 置为 null，否则两边遍历终止条件不同


## 删除单节点

题目见 `Remove`

思路：

1. 若该节点为空或者为最后一个节点，返回 false
2. 否则将下一个节点的 val 和 next 赋值给当前节点，达到“删除”的效果
