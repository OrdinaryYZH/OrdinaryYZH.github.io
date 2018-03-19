### _01_MinStack

> 时间：2018/03/19

> 题意：实现栈，实现以下方法
>
> - push(x) -- Push element x onto stack.
> - pop() -- Removes the element on top of the stack.
> - top() -- Get the top element.
> - getMin() -- Retrieve the minimum element in the stack.

> 分析：
>
> 注：getMin(1)用$O(1)$实现
>
> 使用辅助栈min，贪心，push后，如果x比辅助栈顶小，则加到辅助栈

```java
private Stack<Integer> stack;

/**
 * 辅助栈是一个记录当前栈最小值的栈，是一个降序序列
 */
private Stack<Integer> min;

/**
 * initialize your data structure here.
 */

public _01_MinStack() {
    stack = new Stack<>();
    min = new Stack<>();
}

public void push(int x) {
    stack.push(x);
    // isEmpty是首次加入时，只有当小于才加入，所以是一个升序栈
    if (min.isEmpty() || x <= min.peek()) {
        min.push(x);
    }
}

public void pop() {
    int p = stack.pop();
    // 只有最小值pop，才会更新辅助栈的栈顶
    if (min.peek() == p) {
        min.pop();
    }

}

public int top() {
    return stack.peek();
}

public int getMin() {
    return min.peek();
}
```

图例：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpiijkcppzj21f90wkmzk.jpg)

---

### _02_QueuewithMinimum

> 时间：2018/03/19

> 题意：实现队列，队列有min()方法

> 分析：
>
> 如果插入了一个最小值到队列中，那么出队到这个最小值之前的min都不会变
>
>  如果后面有插入了比最小值大的值，那么最小值出队后;
>
> 使用辅助栈mins

```java
Queue<Long> queue = new LinkedList<>();
LinkedList<Long> mins = new LinkedList<>();

private void pop() {
    long last = queue.poll();
    if (mins.peek().compareTo(last) == 0) mins.poll();
}

private Long min() {
    return mins.peek();
}

private void push(long i) {
    queue.add(i);
    if (mins.isEmpty() || mins.peekLast().compareTo(i) <= 0)
        mins.add(i);
    else {
        while (!mins.isEmpty() && mins.peekLast().compareTo(i) > 0) {
            mins.removeLast();
        }
        mins.add(i);
    }
}
```



---

### _03_ImplementStackUsingQueues

> 时间：2018/03/19

> 题意：使用队列实现栈

> 分析：略

```java
public class MyStack {
    Queue<Integer> q;
    Queue<Integer> temp;

    /**
     * Initialize your data structure here.
     */
    public MyStack() {
        q = new LinkedList<>();
        temp = new LinkedList<>();
    }

    /**
     * Push element x onto stack.
     */
    public void push(int x) {
        if (q.isEmpty()) {
            q.add(x);
        } else {
            while (!q.isEmpty()) {
                // copy all to helper
                temp.add(q.poll());
            }
            q.add(x); // add element
            while (!temp.isEmpty()) {
                // copy back all to helper
                q.add(temp.poll());
            }
        }

    }

    /**
     * Removes the element on top of the stack and returns that element.
     */
    public int pop() {
        return q.poll();
    }

    /**
     * Get the top element.
     */
    public int top() {
        return q.peek();
    }

    /**
     * Returns whether the stack is empty.
     */
    public boolean empty() {
        return q.isEmpty();
    }
}
```

---

### _04_ImplementQueueusingStacks

> 时间：2018/03/19

> 题意：实现队列，队列有getMin()方法

> 分析：略

```java
public class _04_ImplementQueueusingStacks {
    Stack<Integer> stack;
    Stack<Integer> helper;

    /**
     * Initialize your data structure here.
     */
    public _04_ImplementQueueusingStacks() {
        stack = new Stack<>();
        helper = new Stack<>();
    }

    /**
     * Push element x to the back of queue.
     */
    public void push(int x) {
        stack.push(x);
    }

    /**
     * Removes the element from in front of queue and returns that element.
     */
    public int pop() {
        if (helper.isEmpty()) {

            while (!stack.isEmpty()) {
                helper.push(stack.pop());
            }
        }

        return helper.pop();
    }

    /**
     * Get the front element.
     */
    public int peek() {
        if (helper.isEmpty()) {

            while (!stack.isEmpty()) {
                helper.push(stack.pop());
            }
        }
        return helper.peek();
    }

    /**
     * Returns whether the queue is empty.
     */
    public boolean empty() {
        return helper.isEmpty() && stack.isEmpty();
    }
}
```