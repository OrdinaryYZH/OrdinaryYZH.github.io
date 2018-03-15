## Linked List

### _01_LinkedList

> 时间：2018/02/21、2018/03/15

> 题意：实现单向链表

```java
public class _01_LinkedList<E> {
    int size;
    private Node head;
    private Node tail;

    public void add(E element) {
        // 如果链表是空的时候，头跟尾是一样的
        if (head == null) {
            head = tail = new Node(element);
        } else {
            tail.next = new Node(element);
            tail = tail.next;
        }
        size++;
    }

    public E get(int index) {
        if (index >= size || index < 0) {
            throw new NoSuchElementException();
        }

        Node node = head;
        for (int i = 0; i < index; i++) {
            node = node.next;
        }
        return node.element;
    }

    public boolean delete(E element) {
        Node temp = new Node(null, head);
        Node dummy = temp;
        // 因为是delete方法，所以需要使用next来判断，不能使用temp.element作判断，如果用temp的话，就不能删除了(因为删除是操作被删除的上一个节点的next)
        while (temp.next != null) {
            if (temp.next.element.equals(element)) {
                // 处理尾
                if (temp.next == tail) {
                    tail = temp;
                }
                temp.next = temp.next.next;
                // 处理头（优雅的处理head的element等于element的情况，包括只有一个元素的情况）
                head = dummy.next;
                size--;
                return true;
            }
            temp = temp.next;
        }
        return false;
    }

    private class Node {
        E element;
        Node next;

        public Node(E e) {
            this.element = e;
        }

        public Node(E e, Node next) {
            this.element = e;
            this.next = next;
        }
    }
}
```



---

### _02_DoublyLinkedList

> 时间： 2018/02/21、2018/03/15

> 题意：实现双向链表

```java
public class _02_DoublyLinkedList<E> {
    private int size;
    private Node head;
    private Node tail;

    public void add(E e) {
        if (e == null) throw new AssertionError();

        if (head == null) {
            head = tail = new Node(e);
        } else {
            tail.next = new Node(e, tail, null);
            tail = tail.next;
        }
        size++;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public int getSize() {
        return size;
    }

    public E get(int index) {
        if (index >= size) {
            throw new NoSuchElementException();
        }
        Node temp = head;
        for (int i = 0; i < index; i++) {
            temp = temp.next;
        }
        return temp.elem;
    }

    public boolean delete(E elem) {
        Node temp = head;
        while (temp != null && !temp.elem.equals(elem)) {
            temp = temp.next;
        }
        // no node with given element is found
        if (temp == null) return false;

        size--;

        // 处理只有一个节点
        if (size == 0) {
            head = tail = null;
        }

        // 处理删除的是头结点
        if (temp == head) {
            temp.next.pre = null;
            head = head.next;
            return true;
        }

        // 处理删除的是尾节点
        if (temp == tail) {
            temp.pre.next = null;
            tail = temp.pre;
            return true;
        }

        // change links
        temp.pre.next = temp.next;
        temp.next.pre = temp.pre;

        return true;
    }

    private class Node {
        E elem;
        Node pre;
        Node next;

        public Node(E elem) {
            this.elem = elem;
        }

        public Node(E elem, Node pre, Node next) {
            this.elem = elem;
            this.pre = pre;
            this.next = next;
        }
    }
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788gy1fooj12z1ifj21hb0kf40e.jpg)

---

### _04_PalindromeLinkedList

> 时间： 2018/02/22、2018/03/15

> 题意：给一个单向链表，判断是否是回文，最好是O(n) time and O(1) space

> 分析：
>
> 解法一：先得到链表长度，再利用递归的特性：栈中存head的next，全局存rightNode，从中间向两边判断

```java
public class _04_PalindromeLinkedList {

    ListNode rightStart = null;

    public boolean isPalindrome(ListNode head) {
        if (head == null || head.next == null) {
            return true;
        }

        ListNode temp = head;
        rightStart = head;
        int size = 0;
        while (temp != null) {
            size++;
            temp = temp.next;
        }

        int start = (size - 1) / 2 + 1; // size：元素个数，要得到中间下标：(size - 1)/2，要再得到中间靠右，所以+1
        for (int i = 0; i < start; i++) {
            rightStart = rightStart.next;
        }

        return checkTwoHalves(head, size / 2);

    }

    private boolean checkTwoHalves(ListNode x, int size) {
          if (size == 1) {
              return x.val == rightStart.val;
          }
          boolean b = checkTwoHalves(x.next, size - 1);
          rightStart = rightStart.next;
          return b && x.val == rightStart.val;
      }

}
```

解法二：使用快速指针遍历，同时慢指针遍历的时候把方向改变了；然后再从中间到两边对比

```java
public boolean isPalindrome2(ListNode head) {
    if (head == null || head.next == null) {
        return true;
    }

    ListNode slow = head;
    ListNode fast = head;
    ListNode pre = null;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        ListNode next = slow.next;
        slow.next = pre;
        pre = slow;
        slow = next;
    }

    if (fast != null) { // odd
        fast = slow.next;
        slow = pre;
    } else { // even
        fast = slow;
        slow = pre;
    }

    while (fast != null) {
        if (fast.val != slow.val) {
            return false;
        }
        fast = fast.next;
        slow = slow.next;
    }
    return true;
}
```

图例：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpds0qvkebj21760rujtr.jpg)

---

### _05_ReverseLinkedList

> 时间：2018/02/22、2018/03/15

> 题意：翻转单向链表

> 分析：递归和非递归两种方式

```java
/**
 * 非递归写法（修改顺序从左到右）
 *
 * @param head
 * @return
 */
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }

    ListNode pre = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = pre;
        pre = curr;
        curr = next;
    }
    return pre;
}
```

```java
/**
 * 递归写法:得到倒数第二个元素后直接操作即可，返回最后一个元素为链表头（修改顺序从右到左）
 *
 * @param head
 * @return
 */
public ListNode reverseList2(ListNode head) {
    // case1: empty list
    if (head == null) {
        return head;
    }
    // case2: only one element list
    if (head.next == null) {
        return head;
    }
    // case3: reverse from the rest after head
    ListNode newHead = reverseList2(head.next);
    // reverse between head and head->next
    head.next.next = head;
    // unlink list from the rest
    head.next = null;

    return newHead;
}
```

图例：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpds0dpzggj211m0ci0th.jpg)

---

### _06_IntersectionofTwoLinkedLists

> 时间：2018/02/22、2018/03/15

> 题意：给出两个单向链表，他们有可能后面部分完全重合，找出重合的节点
>
> ```
> * A:          a1 → a2
> * ****************** ↘
> * ****************** c1 → c2 → c3 → null
> * ****************** ↗
> * B:     b1 → b2 → b3
> ```

> 分析：
>
> 方法1、简单暴力...
>
> 1、先要知道两个链表的长度
>
> 2、以短的链表为首，长的先"跳到"跟短的一样长先
>
> 3、一一对比
>
> 方法2、一种不需要知道链表长度的做法，
>
> 用两个指针分别指向两个头，当遍历完一个链表时，跳到另一个，当长的一个遍历完跳到另一个时，短的一个也就刚好可以跟长的"靠右对齐"
>
> 达到跟解法1一样的效果，接着一个个对比即可

```java
/**
 * 解法1：简单暴力...
 * 1、先要知道两个链表的长度
 * 2、以短的链表为首，长的先"跳到"跟短的一样长先
 * 3、一一对比
 *
 * @param headA
 * @param headB
 * @return
 */
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {

    if (headA == null || headB == null) {
        return null;
    }

    int sizeA = getLinkedListSize(headA);
    int sizeB = getLinkedListSize(headB);
    int dif = Math.abs(sizeA - sizeB);

    if (sizeA > sizeB) {
        while (dif > 0) {
            headA = headA.next;
            dif--;
        }
    } else {
        while (dif > 0) {
            headB = headB.next;
            dif--;
        }
    }

    while (headA != null) {
        if (headA.val == headB.val) {
            return headA;
        }
        headA = headA.next;
        headB = headB.next;
    }
    return null;
}

private int getLinkedListSize(ListNode listNode) {
    int size = 0;
    while (listNode != null) {
        size++;
        listNode = listNode.next;
    }
    return size;
}
```

---

解法2分析：

* 会死循环？
  不会，a = a.next，直接赋值，没有条件判断a.next ！= null ，所以a会指到null；
  当两个链表不相交时，a跟b都会为null，循环结束，返回null

```java
/**
 * 解法2：
 * 一种不需要知道链表长度的做法，
 * 用两个指针分别指向两个头，当遍历完一个链表时，跳到另一个，当长的一个遍历完跳到另一个时，短的一个也就刚好可以跟长的"靠右对齐"
 * 达到跟解法1一样的效果，接着一个个对比即可
 *
 * @param headA
 * @param headB
 * @return
 */
public ListNode getIntersectionNode2(ListNode headA, ListNode headB) {
    ListNode a = headA;
    ListNode b = headB;

    // 能够处理无交点的情况
    while (a != b) {
        a = a == null ? headB : a.next;
        b = b == null ? headA : b.next;
    }
    return a;
}
```

---

### _07_LinkedListCycle

> 时间：2018/02/22、2018/03/15

> 题意：判断单向链表是否有环

> 分析：对于带环链表的检测，效率较高且易于实现的一种方式为使用快慢指针。快指针每次走两步，慢指针每次走一步，如果快慢指针相遇(快慢指针所指内存为同一区域)则有环，否则快指针会一直走到`NULL`为止退出循环，返回`false`.
>
> 快指针走到`NULL`退出循环即可确定此链表一定无环这个很好理解。那么带环的链表快慢指针一定会相遇吗？先来看看下图。
>
> ![Linked List Cycle](https://box.kancloud.cn/2015-10-24_562b1f4a7a4ad.png)
>
> 在有环的情况下，最终快慢指针一定都走在环内，加入第`i`次遍历时快指针还需要`k`步才能追上慢指针，由于快指针比慢指针每次多走一步。那么每遍历一次快慢指针间的间距都会减少1，直至最终相遇。故快慢指针相遇一定能确定该链表有环。

```java
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) {
        return false;
    }

    ListNode slow = head;
    ListNode fast = head.next;

    while (fast != null && fast.next != null) {
        if (slow.val == fast.val) {
            return true;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    return false;
}
```

---

### _08_RemoveNthNodeFromEndOfList

> 时间：2018/02/22、2018/03/15

> 题意：删除单向链表的倒数第n个节点

> 分析：
>
> 1、非递归
>
> 2、递归

```java
/**
 * 解法1：
 * 1、得到所有元素个数
 * 2、计算要删除的正数元素是第几个
 * 3、删除
 * 注意：使用dummy
 *
 * @param head
 * @param n
 * @return
 */
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode temp = new ListNode(0);
    temp.next = head;
    ListNode dummy = temp;

    int size = 0;
    while (temp.next != null) {
        size++;
        temp = temp.next;
    }
    temp = dummy;

    int i = size - n;
    while (i > 0) {
        temp = temp.next;
        i--;
    }
    temp.next = temp.next.next;
    return dummy.next;
}
```

---

```java
/**
 * 递归用到的字段
 */
private int n;

/**
 * 解法2：递归
 * 解法1更好理解
 *
 * @param head
 * @param n
 * @return
 */
public ListNode removeNthFromEnd2(ListNode head, int n) {
    if (head == null || head.next == null) {
        return null;
    }

    this.n = n;
    remove(head);
    return this.n == 0 ? head.next : head;
}

private void remove(ListNode x) {
    if (x == null) {
        return;
    }
    remove(x.next);
    if (n == 0) {
        x.next = x.next.next; // 找到时不return，如果是head被删除的话，n=0，否则n<0 || >0；当n==size时，即系要删除head时，这里是不会被执行到的
    }
    n--;
}
```

图例：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpdrzplmryj212c0j4jsr.jpg)

---

### _09_SortList

> 时间：2018/02/22、2018/03/15

> 题意：合并两个已排序的单向链表

> 分析：
>
> 1、递归
>
> 2、非递归

```java
/**
 * 归并排序
 */
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }

    ListNode slow = head;
    ListNode fast = head.next;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    ListNode start = slow.next;
    slow.next = null;

    ListNode leftNode = sortList(head);
    ListNode rightNode = sortList(start);
    return mergeTwoLists(leftNode, rightNode);
}

/**
 * 递归写法
 *
 * @param l1
 * @param l2
 * @return
 */
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null) {
        return l2;
    } else if (l2 == null) {
        return l1;
    }

    if (l1.val <= l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```

---

```java
/**
 * 非递归写法
 *
 * @param l1
 * @param l2
 * @return
 */
public ListNode mergeTwoLists2(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode head = dummy;
    while (l1 != null || l2 != null) {
        if (l1 == null) {
            dummy.next = l2;
            l2 = l2.next;
        } else if (l2 == null) {
            dummy.next = l1;
            l1 = l1.next;
        } else if (l1.val <= l2.val) {
            dummy.next = l1;
            l1 = l1.next;
        } else {
            dummy.next = l2;
            l2 = l2.next;
        }
        dummy = dummy.next;
    }
    return head.next;
}
```

---

### _10_LinkedListCycle2

> 时间：2018/02/22、2018/03/15

> 题意：给出一个单向链表，如果有环，返回环的开始节点，没有环的话：返回null

> 分析1：
>
> ![Linked List Cycle II](http://7b1hdq.com1.z0.glb.clouddn.com/blog/pic/259/circle.png)
>
> ```
> 1). 使用快慢指针法，若链表中有环，可以得到两指针的交点M
>
> 2). 记链表的头节点为H，环的起点为E
>
> 2.1) L1为H到E的距离
> 2.2) L2为从E出发，首次到达M时的路程
> 2.3) C为环的周长
> 2.4) n为快慢指针首次相遇时，快指针在环中绕行的次数
>
> 根据L1,L2和C的定义，我们可以得到：
>
> 慢指针行进的距离为L1 + L2
>
> 快指针行进的距离为L1 + L2 + n * C
>
> 由于快慢指针行进的距离有2倍关系，因此：
>
> 2 * (L1+L2) = L1 + L2 + n * C => L1 + L2 = n * C => L1 = (n - 1)* C + (C - L2)
>
> 可以推出H到E的距离 = 从M出发绕环到达E时的路程
>
> 因此，当快慢指针在环中相遇时，我们再令一个慢指针从头节点出发
>
> 接下来当两个慢指针相遇时，即为E所在的位置
> ```

```java
public ListNode detectCycle(ListNode head) {
    if (head == null || head.next == null) {
        return null;
    }

    ListNode slow = head.next;
    ListNode fast = head.next.next;

    while (fast != null && fast.next != null) {
        if (slow == fast) {
            break;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    if (fast == null || fast.next == null) {
        return null;
    }

    ListNode temp = head;
    while (slow != temp) {
        slow = slow.next;
        temp = temp.next;
    }
    return slow;
}
```

---

> 分析2：不同之处是找相交点即可( 参考：_06_IntersectionofTwoLinkedLists)，* 不用通过推算得出结论，slow从head开始；如果是解法1的话，slow需要从head.next开始才行

```java
/**
 * 解法2，不同之处是找相交点即可( 参考：_06_IntersectionofTwoLinkedLists)，
 * 不用通过推算得出结论，slow从head开始；如果是解法1的话，slow需要从head.next开始才行
 *
 * @param head
 * @return
 */
public ListNode detectCycle2(ListNode head) {
    if (head == null) {
        return null;
    }
    // 这样也可以
//        ListNode slow = head.next;
//        ListNode fast = head.next.next;
    ListNode slow = head;
    ListNode fast = head.next;
    while (fast != null && fast.next != null) {
        if (fast == slow) {
            break;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    ListNode temp = slow.next;
    slow.next = null;// break the link
    ListNode link = findLink(head, temp);
    slow.next = temp; // link the break
    return link;
}

ListNode findLink(ListNode x, ListNode y) {
    if (x == null || y == null) {
        return null;
    }

    ListNode xhead = x;
    ListNode yhead = y;

    while (xhead != yhead) {
        xhead = xhead == null ? y : xhead.next;
        yhead = yhead == null ? x : yhead.next;
    }
    return xhead;
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpdrzeclp5j20tk0t3407.jpg)

![](https://ws1.sinaimg.cn/large/8747d788gy1foparhsee6j20wg0m6wft.jpg)

---

### _11_MergekSortedLists

> 时间：2018/02/22

> 题意：合并k个已排序的链表

> 参考：https://algorithm.yuanbin.me/zh-hans/linked_list/merge_k_sorted_lists.html
>
> 需要明白时间复杂度度的计算；
>
> 这里使用二分法优化

```java
public class _11_MergekSortedLists {

    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) {
            return null;
        }
        if (lists.length == 1) {
            return lists[0];
        }

        return sort(lists, 0, lists.length - 1);
    }

    ListNode sort(ListNode[] lists, int left, int right) {
        if (left >= right) {
            return lists[left];
        }
        int mid = (left + right) / 2;
        ListNode leftNode = sort(lists, left, mid);
        ListNode rightNode = sort(lists, mid + 1, right);
        return mergeTwoLists(leftNode, rightNode);
    }

    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        }

        if (l1.val <= l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }

    /**
     * 非递归归写法
     *
     * @param left
     * @param right
     * @return
     */
    public ListNode merge(ListNode left, ListNode right) {
        ListNode x = new ListNode(0); // 非递归的方式需要帮助节点，在x的基础上修改next，而不是修改left.next/right.next
        ListNode xx = x;
        while (left != null || right != null) {
            if (left == null) {
                x.next = right;
                break;
            } else if (right == null) {
                x.next = left;
                break;
            } else {
                if (left.val < right.val) {
                    x.next = left;
                    left = left.next;
                } else {
                    x.next = right;
                    right = right.next;
                }
            }
            x = x.next;
        }
        return xx.next;
    }

}
```