---
title: LeetCode 链表专题
date: 2021-03-29 00:23:02
tags:
- 算法
- LeetCode
---

<!-- more -->

## 链表有环
### [LeetCode 141：判断链表是否有环](https://leetcode-cn.com/problems/linked-list-cycle/)
#### 题目
给定一个链表，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 true 。 否则，返回 false 。

进阶：

你能用 O(1)（即，常量）内存解决此问题吗？

示例 1：

```

输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```
示例 2：

```
输入：head = [1,2], pos = 0
输出：true
解释：链表中有一个环，其尾部连接到第一个节点。
```
示例 3：
```
输入：head = [1], pos = -1
输出：false
解释：链表中没有环。
```

提示：
- 链表中节点的数目范围是 [0, 104]
- -105 <= Node.val <= 105
- pos 为 -1 或者链表中的一个 有效索引 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/linked-list-cycle
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 个人解答：
```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */

/**
 * @param {ListNode} head
 * @return {boolean}
 */
var hasCycle = function(head) {
    if (!head) return false;
    let slow = head, fast = head.next;
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (fast === slow) {
            return true;
        }
    }
    return false;
};
```

### [LeetCode 142：链表寻相遇点](https://leetcode-cn.com/problems/linked-list-cycle-ii)
#### 题目
给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。

说明：不允许修改给定的链表。

进阶：

你是否可以使用 O(1) 空间解决此题？

示例 1：

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210318082318.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```
示例 2：

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210318082414.png)

```
输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
```
示例 3：

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210318082443.png)

```
输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```
提示：

- 链表中节点的数目范围在范围 `[0, 104]` 内
- `-105 <= Node.val <= 105`
- pos 的值为 -1 或者链表中的一个有效索引


来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/linked-list-cycle-ii
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 个人解答：
```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */

/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var detectCycle = function(head) {
    if (!head || !head.next) return null;
    let slow = head.next, fast = head.next.next;
    while(fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow === fast) {
            // 有环
            slow = head;
            while(slow !== fast) {
                slow = slow.next;
                fast = fast.next;
            }
            return slow;
        }
    }
    return null;
};
```


### [LeetCode 202：快乐数](https://leetcode-cn.com/problems/happy-number/)
#### 题目
编写一个算法来判断一个数 n 是不是快乐数。

「快乐数」定义为：

对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。
如果 可以变为  1，那么这个数就是快乐数。
如果 n 是快乐数就返回 true ；不是，则返回 false 。

示例 1：
```
输入：19
输出：true
解释：
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

示例 2：
```
输入：n = 2
输出：false
```

提示：
```
1 <= n <= 231 - 1
```
来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/happy-number
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 个人解答
```
 /**
 * @param {number} n
 * @return {boolean}
 */
var isHappy = function(n) {
    var squaresSum = function(n) {
        var sum = 0;
        do {
             var num = n % 10;
             sum = sum + num ** 2;
             var n = Math.floor(n / 10)
        } while (n > 0)
        return sum;
    }

    var slow = squaresSum(n), fast = squaresSum(squaresSum(n));

    while(fast !== 1) {
        slow = squaresSum(slow);
        fast = squaresSum(squaresSum(fast))
        if (fast === slow) {
            return false;
        }
    }
    return true;
};
```

## 链表的反转
### [LeetCode 206：反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)
#### 题目
反转一个单链表。

示例:
```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

进阶:
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？



来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/reverse-linked-list
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 个人解答
```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function (head) {
    if (!head) return null;
    var prevNode = null, curNode = head, nextNode = head.next;

    while (nextNode) {
        curNode.next = prevNode;
        prevNode = curNode;
        curNode = nextNode;
        nextNode = nextNode.next;
    }
    curNode.next = prevNode;
    return curNode;
};
```
递归做法
```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function (head) {
    function reverse(prevNode, curNode) {
        if (!curNode) return prevNode;
        const listReversed = reverse(curNode, curNode.next)
        curNode.next = prevNode;
        return listReversed;
    }

    return reverse(null, head);
};
```
或者
```js
var reverseList = function (head) {
    if (!head || !head.next) return head;

    var listReversed = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    
    return listReversed;
};
```
### [LeetCode 92：翻转区间的链表](https://leetcode-cn.com/problems/reverse-linked-list-ii)
#### 题目
给你单链表的头指针 head 和两个整数 left 和 right ，其中 left <= right 。请你反转从位置 left 到位置 right 的链表节点，返回 反转后的链表 。

示例 1：

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210320175606.png)

```
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```

示例 2：
```
输入：head = [5], left = 1, right = 1
输出：[5]
```

提示：
- 链表中节点数目为 n
- `1 <= n <= 500`
- `-500 <= Node.val <= 500`
- `1 <= left <= right <= n`

进阶： 你可以使用一趟扫描完成反转吗？

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/reverse-linked-list-ii
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 个人解答
```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} left
 * @param {number} right
 * @return {ListNode}
 */
var reverseBetween = function (head, left, right) {
    const dummy = new ListNode(0, head);

    let prev = dummy;

    for (let i = 0; i < left - 1; i++) {
        prev = prev.next;
    }

    let reverseTail = prev.next;
    for (let i = left; i < right; i++) {
        let reverseHead = reverseTail.next;
        reverseTail.next = reverseHead.next;
        reverseHead.next = prev.next;
        prev.next = reverseHead;
    }

    return dummy.next;
};
```

### [LeetCode 25：k个一组的翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

#### 题目
给你一个链表，每 k 个节点一组进行翻转，请你返回翻转后的链表。

k 是一个正整数，它的值小于或等于链表的长度。

如果节点总数不是 k 的整数倍，那么请将最后剩余的节点保持原有顺序。

进阶：
- 你可以设计一个只使用常数额外空间的算法来解决此问题吗？
- 你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。
 

示例 1：

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210329001844.png)

```
输入：head = [1,2,3,4,5], k = 2
输出：[2,1,4,3,5]
```

示例 2：

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210329001934.png)

```
输入：head = [1,2,3,4,5], k = 3
输出：[3,2,1,4,5]
```
示例 3：
```
输入：head = [1,2,3,4,5], k = 1
输出：[1,2,3,4,5]
```
示例 4：
```
输入：head = [1], k = 1
输出：[1]
```
提示：
- 列表中节点的数量在范围 sz 内
- 1 <= sz <= 5000
- 0 <= Node.val <= 1000
- 1 <= k <= sz

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/reverse-nodes-in-k-group
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

#### 个人解答
解法1：使用堆栈，
- 依次将 node 推入堆栈，当到达 k 个的时候，将堆栈的依次连接实现反转
- 然后再连接堆栈的首尾元素
- 将 k 个反转后，清空堆栈
- 一直遍历到链表结束即可

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} k
 * @return {ListNode}
 */
var reverseKGroup = function (head, k) {
    let stack = [];
    const dummy = new ListNode(null, head);
    let prevNode = new ListNode(null, head);
    let p = head;
    let firstReversed = true;

    // 遍历链表，依次 push 到 stack，当到达 k 的时候拿出来
    while (p) {
        stack.push(p);
        p = p.next;
        if (stack.length === k) {
            // 到数量到达 k 的时候，依次取出来连接在一块
            // 第一次 k 反转的最后一个就是链表的第一个node了
            if (firstReversed) {
                firstReversed = false;
                dummy.next = stack[k - 1];
            }
            prevNode.next = stack[k - 1];
            stack[0].next = stack[k - 1].next;
            prevNode = stack[0];
            for (let i = 0; i < k - 1; i++) {
                stack[i + 1].next = stack[i]
            }
            stack = [];
        }
    }

    return dummy.next;
};
```

### [LeetCode 61：旋转链表]()

### [LeetCode 24：两两交换链表中的节点]()

## 链表节点的删除

### [LeetCode 19：删除链表倒数的第 N 个节点]()

### [LeetCode 83：删除排序链表中的重复节点]()

### [LeetCode 82：删除排序排序链表中的重复节点2]()

## 其他
翻转链表的两种实现
- 正常实现
- 递归实现

什么时候使用虚拟头：  
若头地址有可能发生改变的时候一般使用虚拟头地址
