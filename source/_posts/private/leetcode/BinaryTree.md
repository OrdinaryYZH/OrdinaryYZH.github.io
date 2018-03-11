

# BinaryTree

## 总结

1. 二叉树递归遍历的情况

   1. 叶子节点（~~肯定首先被遍历~~递归可能根据条件不符合而终止：e.g.：_08_SymmetricTree）
   2. 只有左节点
   3. 只有右节点
   4. 有两个子节点

   ![](https://ws1.sinaimg.cn/large/8747d788gy1forz8xp259j20q20d3mxr.jpg)

2. 用队列可以模拟深搜

---



### _01_BinaryTreeLevelOrderTraversal

> 时间：2018/02/23

> 题意：给出一颗二叉树，输出每一层的节点

> 分析：BFS + 队列辅助

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) {
        return result; // 这里应该返回[]，而不是null
    }

    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);

    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            list.add(node.val);
            if (node.left != null) {
                queue.add(node.left);
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }
        result.add(list);
    }
    return result;
}
```

---

### _02_SumofLeftLeaves

> 时间：2018/02/23

> 题意：给出一个二叉树，求其"左叶子节点"之和
>
> ```
>     3
>    / \
>   9  20
>     /  \
>    15   7
>
> There are two left leaves in the binary tree, with values 9 and 15 respectively. Return 24.
> ```

> 分析：
>
> 1、如何判断左叶子节点：`root.left.left == null && root.left.right == null`
>
> 2、遍历判断即可（可用多种方法遍历）

```java
public class _02_SumofLeftLeaves {

    /**
     * 解法一，深搜(递归)
     *
     * @param root
     * @return
     */
    public int sumOfLeftLeaves(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int sum = 0;
        if (root.left != null) {
            // 判断左叶子节点
            if (root.left.left == null && root.left.right == null) {
                sum += root.left.val;
            } else {
                sum += sumOfLeftLeaves(root.left);
            }
        }
        if (root.right != null) {
            sum += sumOfLeftLeaves(root.right);
        }
        return sum;
    }

    /**
     * 错误解法，左叶子节点判断错误
     * root不知道是左节点还是右节点
     *
     * @param root
     * @return
     */
    public int sumOfLeftLeaves4(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int sum = 0;
        if (root.left == null && root.right == null) {
            return root.val;
        }
        if (root.left != null) {
            sum += sumOfLeftLeaves(root.left);
        }
        if (root.right != null) {
            sum += sumOfLeftLeaves(root.right);
        }
        return sum;
    }

    /**
     * 解法二：BFS + 队列辅助
     *
     * @param root
     * @return
     */
    public int sumOfLeftLeaves2(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int sum = 0;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (node.left != null) {
                if (node.left.left == null && node.left.right == null) {
                    sum += node.left.val;
                } else {
                    queue.add(node.left);
                }
            }
            if (node.right != null) {
                if (node.right.left != null || node.right.right != null) {
                    queue.add(node.right);
                }
            }
        }
        return sum;
    }

    /**
     * 解法3：用栈；
     * 用栈的话，遍历方式跟递归差不多，但这里会先遍历右节点，再遍历左节点
     *
     * @param root
     * @return
     */
    public int sumOfLeftLeaves3(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int ans = 0;
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);

        while (!stack.empty()) {
            TreeNode node = stack.pop();
            if (node.left != null) {
                if (node.left.left == null && node.left.right == null) {
                    ans += node.left.val;
                } else {
                    stack.push(node.left);
                }
            }
            if (node.right != null) {
                if (node.right.left != null || node.right.right != null) {
                    stack.push(node.right);
                }
            }
        }
        return ans;
    }
}
```

---

### _03_InvertBinaryTree

> 时间：2018/02/24

> 题意：翻转二叉树

```java
/**
 * 深搜
 *
 * @param root
 * @return
 */
public TreeNode invertTree(TreeNode root) {
    if (root == null) {
        return root;
    }
    TreeNode left = invertTree(root.left);
    TreeNode right = invertTree(root.right);
    root.left = right;
    root.right = left;
    // 这个return啥也没干，就返回参数root
    return root;
}
```

```java
/**
 * 广搜
 *
 * @param root
 * @return
 */
public TreeNode invertTree2(TreeNode root) {
    if (root == null) {
        return null;
    }

    Queue<TreeNode> queue = new LinkedList<TreeNode>();
    queue.add(root);
    while (!queue.isEmpty()) {
        TreeNode node = queue.poll();
        swap(node);
        if (node.left != null) {
            queue.add(node.left);
        }
        if (node.right != null) {
            queue.add(node.right);
        }
    }
    return root;
}
```

一样的题目：[二叉树的镜像](https://www.nowcoder.com/practice/564f4c26aa584921bc75623e48ca3011?tpId=13&tqId=11171&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

注意该方法没有返回值

```java
public class Solution {
    public void Mirror(TreeNode root) {
        if (root == null)
            return;
        Mirror(root.left);
        Mirror(root.right);
        TreeNode tempNode = root.left;
        root.left = root.right;
        root.right = tempNode;
    }
}
```

---

### _04_BSTIterator

> 时间：2018/02/24

> 题意：给出一个二叉搜索树，设计一个迭代器，包含以下两个方法：
>
> 1、hasNext()：是否存在下一个节点
>
>  2、next()：输出下一个节点的val（由小到大输出）

> 分析：
>
> ```
> 二叉搜索树：
> * 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
> * 若任意节点的右子树不空，则右子树上所有节点的值均大于它的根节点的值；
> * 任意节点的左、右子树也分别为二叉查找树；
> ```
> 明显的中序遍历

```java
public class _04_BSTIterator {
    Deque<TreeNode> stack;
    TreeNode cur;

    public _04_BSTIterator(TreeNode root) {
        stack = new ArrayDeque<>();
        cur = root;
    }

    /**
     * @return whether we have a next smallest number
     */
    public boolean hasNext() {
        return !stack.isEmpty() || cur != null;
    }

    /**
     * @return the next smallest number
     */
    public int next() {
        while (cur != null) {
            stack.offerLast(cur);
            cur = cur.left;
        }
        TreeNode temp = stack.pollLast();
        cur = temp.right;
        return temp.val;
    }
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788gy1forogsektyj20mx0j4jsp.jpg)

---

### _05_PostOrderTraversalTree

> 时间：2018/02/24

> 题意：返回后序遍历(【左右根】)的节点val数组

> 分析：这里有四种写法，最简单是递归写法

```java
public class _05_PostOrderTraversalTree {

    /**
     * 简单的递归写法
     *
     * @param root
     * @return
     */
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) {
            return result;
        }

        if (root.left != null) {
            result.addAll(postorderTraversal(root.left));
        }
        if (root.right != null) {
            result.addAll(postorderTraversal(root.right));
        }
        result.add(root.val);
        return result;
    }

    /**
     * 非递归写法:
     * 用栈和集合辅助
     *
     * @param root
     * @return
     */
    public List<Integer> postorderTraversal2(TreeNode root) {
        List<Integer> result = new LinkedList<>();
        if (root == null) {
            return result;
        }

        Deque<TreeNode> stack = new ArrayDeque<>();
        Set<TreeNode> set = new HashSet<>();
        stack.offer(root);
        // 叶子节点时，peek.left == peek.right == null
        set.add(null);

        while (!stack.isEmpty()) {
            TreeNode peek = stack.peekLast();
            // 当左节点和右节点都在set中，当前节点就要遍历了
            if (set.contains(peek.left) && set.contains(peek.right)) {
                // 这里才在栈中去掉
                result.add(stack.pollLast().val);
                set.add(peek);
            } else {
                if (peek.right != null) {
                    stack.offerLast(peek.right);
                }
                if (peek.left != null) {
                    stack.offerLast(peek.left);
                }
            }
        }
        return result;
    }

    /**
     * 非递归的另一种写法
     * https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/73027
     *
     * @param root
     * @return
     */
    public List<Integer> postorderTraversal3(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) {
            return result;
        }

        Stack<TreeNode> s = new Stack<>();
        s.push(root);
        TreeNode prev = null;
        while (!s.empty()) {
            TreeNode curr = s.peek();
            boolean noChild = false;
            // 1、叶子节点
            if (curr.left == null && curr.right == null) {
                noChild = true;
            }
            boolean childVisited = false;
            // 2、当前节点的子节点已经访问了；
            // 它有两个节点或者一个节点，有以下情况：
            // ---- 1、只有左节点，则left == prev
            // ---- 2、有两个节点，或者只有右节点，则right == prev
            // 所以这里使用 ||
            if (prev != null && (curr.left == prev || curr.right == prev)) {
                childVisited = true;
            }

            // traverse
            if (noChild || childVisited) {
                result.add(curr.val);
                s.pop();
                prev = curr;
            } else {
                if (curr.right != null) {
                    s.push(curr.right);
                }
                if (curr.left != null) {
                    s.push(curr.left);
                }
            }

        }

        return result;
    }

    /**
     * 解法3，因为要返回的是【左右根】，我们可以用发现先序遍历的时候可以做到【根右左】，所以再把它reverse一下就得到答案了
     *
     * @param root
     * @return
     */
    public List<Integer> postorderTraversal44(TreeNode root) {
        List<Integer> result = new LinkedList<>();
        if (root == null) {
            return result;
        }

        Deque<TreeNode> stack = new ArrayDeque<>();
        stack.offerLast(root);

        while (!stack.isEmpty()) {
            TreeNode treeNode = stack.pollLast();
            result.add(treeNode.val);

            if (treeNode.left != null) {
                stack.offerLast(treeNode.left);
            }
            if (treeNode.right != null) {
                stack.offerLast(treeNode.right);
            }
        }
        Collections.reverse(result);
        return result;
    }

}

```

---

### _06_BinaryTreePreorderTraversal

> 时间：2018/02/24

> 题意：给出一个二叉树，返回先序遍历（【根左右】）数组

> 分析：
>
> 1、递归
>
> 2、非递归

```java
public class _06_BinaryTreePreorderTraversal {

    /**
     * 递归写法
     *
     * @param root
     * @return
     */
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) {
            return result;
        }

        result.add(root.val);
        if (root.left != null) {
            result.addAll(preorderTraversal(root.left));
        }
        if (root.right != null) {
            result.addAll(preorderTraversal(root.right));
        }
        return result;
    }

    /**
     * 非递归写法
     *
     * @param root
     * @return
     */
    public List<Integer> preorderTraversal2(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) {
            return result;
        }

        Deque<TreeNode> stack = new ArrayDeque<>();
        stack.offerLast(root);

        while (!stack.isEmpty()) {
            TreeNode treeNode = stack.pollLast();
            result.add(treeNode.val);
            if (treeNode.right != null) {
                stack.offerLast(treeNode.right);
            }
            if (treeNode.left != null) {
                stack.offerLast(treeNode.left);
            }
        }
        return result;
    }
}
```

---

### _07_FlattenBinaryTreetoLinkedList

> 时间：2018/02/25

> 题意：Given
>
> ```
>          1
>         / \
>        2   5
>       / \   \
>      3   4   6
>
> ```
>
> The flattened tree should look like:
>
> ```
>    1
>     \
>      2
>       \
>        3
>         \
>          4
>           \
>            5
>             \
>              6
> ```

> 分析：
>
> 1、递归，先左后右
>
> 2、递归，先右后左（推荐）
>
> 3、非递归

```java
public class _07_FlattenBinaryTreetoLinkedList {

    /**
     * 解法1：
     * 思路是先利用DFS的思路找到最左子节点，
     * 然后回到其父节点，把其父节点和右子节点断开，将原左子结点连上父节点的右子节点上，
     * 然后再把原右子节点连到新右子节点的右子节点上，然后再回到上一父节点做相同操作
     *
     * @param root
     */
    public void flatten(TreeNode root) {
        if (root == null) {
            return;
        }
        // 先左后右，那么下面就要while(root.right != null)
        flatten(root.left);
        flatten(root.right);

        TreeNode temp = root.right;
        root.right = root.left;
        root.left = null;
        while (root.right != null) {
            root = root.right;
        }
        root.right = temp;
    }

    /**
     * 辅助变量
     */
    private TreeNode prev = null;

    /**
     * 解法2，递归，先右后左 + 变量辅助
     *
     * @param root
     */
    public void flatten2(TreeNode root) {
        if (root == null) {
            return;
        }
        // 先右后左
        flatten2(root.right);
        flatten2(root.left);
        // prev可能是left，也有可能是right
        root.right = prev;
        root.left = null;
        prev = root;
    }

    /**
     * 非递归：不断的将左子树放到根与右子树之间
     * 参考：http://www.cnblogs.com/grandyang/p/4293853.html
     *
     * @param root
     */
    public void flatten3(TreeNode root) {
        if (root == null) {
            return;
        }

        TreeNode cur = root;
        while (cur != null) {
            // 不断的将左子树放到根与右子树之间
            if (cur.left != null) {
                TreeNode temp = cur.left;
                while (temp.right != null) {
                    temp = temp.right;
                }
                temp.right = cur.right;
                cur.right = cur.left;
                cur.left = null;
            }
            cur = cur.right;
        }
    }

}
```

---

### _08_SymmetricTree

> 时间：2018/02/25

> 题意：判断二叉树是否是对称
>
> For example, this binary tree `[1,2,2,3,4,4,3]` is symmetric:
>
> ```
>     1
>    / \
>   2   2
>  / \ / \
> 3  4 4  3
>
> ```
>
> But the following `[1,2,2,null,3,null,3]` is not:
>
> ```
>     1
>    / \
>   2   2
>    \   \
>    3    3
> ```

> 分析：
>
> 1、递归；向两边递归，使用两个参数
>
> 2、非递归：BFS + 回文判断
>
> 3、非递归：使用两个队列模拟递归

```java
public class _08_SymmetricTree {

    /**
     * 解法一：递归
     *
     * @param root
     * @return
     */
    public boolean isSymmetric(TreeNode root) {
        if (root == null) {
            return true;
        }
        return check(root.left, root.right);
    }

    private boolean check(TreeNode left, TreeNode right) {
        if (left == null && right == null) {
            return true;
        }
        if (left == null || right == null) {
            return false;
        }
        boolean con1 = left.val == right.val ? true : false;
        boolean con2 = check(left.left, right.right);
        boolean con3 = check(left.right, right.left);
        return con1 & con2 & con3;
    }

    /**
     * 非递归：BFS + 回文判断
     * 注意：
     * ArrayDeque不能添加null；
     * list.add(treeNode.left.val)会 nullpointexception
     *
     * @param root
     * @return
     */
    public boolean isSymmetric2(TreeNode root) {
        if (root == null) {
            return true;
        }
        Deque<TreeNode> queue = new ArrayDeque<>();
        List<Integer> list = new ArrayList<>();
        List<Integer> list2;
        queue.offer(root);
        while (!queue.isEmpty()) {
            list.clear();
            int length = queue.size();
            for (int i = 0; i < length; i++) {
                TreeNode treeNode = queue.pollFirst();
                if (treeNode.left != null) {
                    queue.offer(treeNode.left);
                }
                if (treeNode.right != null) {
                    queue.offer(treeNode.right);
                }

                if (treeNode.left == null) {
                    list.add(0);
                } else {
                    list.add(treeNode.left.val);
                }
                if (treeNode.right == null) {
                    list.add(0);
                } else {
                    list.add(treeNode.right.val);
                }
            }
//            Integer[] integers = Arrays.<Integer, Integer>copyOf(list.toArray(new Integer[list.size()]), list.size(), Integer[].class);
            list2 = new ArrayList<>(Collections.nCopies(list.size(), 0));
            Collections.copy(list2, list);
            Collections.reverse(list2);
            boolean flag = Objects.deepEquals(list, list2);
            if (flag == false) {
                return false;
            }
        }
        return true;
    }

    /**
     * 非递归，用两个队列辅助
     *
     * @param root
     * @return
     */
    public boolean isSymmetric3(TreeNode root) {
        if (root == null) {
            return true;
        }
        Queue<TreeNode> queue1 = new LinkedList<>();
        Queue<TreeNode> queue2 = new LinkedList<>();
        queue1.offer(root.left);
        queue2.offer(root.right);
        while (!queue1.isEmpty() && !queue2.isEmpty()) {
            TreeNode left = queue1.poll();
            TreeNode right = queue2.poll();
            // 1、两边都为null，不用处理
            // 2、一边为空，一边不为空，就不对称
            if ((left == null && right != null) ||
                    (left != null && right == null)) {
                return false;
            }
            // 3、两边都有值的情况
            if (left != null && right != null) {
                if (left.val != right.val) {
                    return false;
                }
                queue1.offer(left.left);
                queue1.offer(left.right);
                queue2.offer(right.right);
                queue2.offer(right.left);
            }
        }
        return true;
    }


    public static void main(String[] args) {
//        String str = "[1,2,2,3,4,4,3]";
        String str = "[1,2,2,null,3,null,3]";
        TreeNode node = TreeNode.mkTree(str);
        _08_SymmetricTree symmetricTree = new _08_SymmetricTree();
        boolean flag = symmetricTree.isSymmetric3(node);
        System.out.println(flag);
    }
}
```

---

### _09_BinaryTreeInorderTraversal

> 时间：2018/02/26

> 题意：返回二叉树中序遍历节点数组，不包括null
>
> Given binary tree `[1,null,2,3]`,
>
> ```
>    1
>     \
>      2
>     /
>    3
>
> ```
>
> return `[1,3,2]`.

> 分析：
>
> 1、非递归，用栈辅助
>
> 2、递归

```java
/**
 * 非递归，用栈辅助
 *
 * @param root
 * @return
 */
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> list = new ArrayList<>();

    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;

    while (cur != null || !stack.empty()) {
        while (cur != null) {
            stack.add(cur);
            cur = cur.left;
        }
        cur = stack.pop();
        list.add(cur.val);
        cur = cur.right;
    }

    return list;
}
```

```java
/**
 * 递归
 *
 * @param root
 * @return
 */
public List<Integer> inorderTraversal2(TreeNode root) {
    List<Integer> list = new ArrayList<>();
    inOrder(root, list);
    return list;
}


void inOrder(TreeNode x, List<Integer> list) {
    if (x == null) {
        return;
    }
    inOrder(x.left, list);
    list.add(x.val);
    inOrder(x.right, list);
}
```

---

### _10_SameTree

> 时间：2018/02/26

> 题意：判断两棵二叉树是否一样

> 分析：跟判断二叉树对称差不多，递归判断



```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) {
        return true;
    }
    if (p == null || q == null) {
        return false;
    }
    if (p.val != q.val) {
        return false;
    }
    boolean check1 = isSameTree(p.left, q.left);
    boolean check2 = isSameTree(p.right, q.right);
    return check1 && check2;

}
```

别人写的：

```java
public boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null) {
        return q == null;
    }
    if (q == null) {
        return p == null;
    }
    if (p.val != q.val) {
        return false;
    }
    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
}
```



---

## 树的深度

### _11_MaximumDepthofBinaryTree

> 时间：2018/02/26

> 题意：求二叉树的深度

> 分析：
>
> 1、不用额外变量可否？
>
> 2、不信定义方法可否？
>
> 可以的

```java
public int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```



---

### _12_BalancedBinaryTree

> 时间：2018/02/26

> 题意：判断二叉树是否是平衡二叉树（它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树）
>
> **Example 1:**
>
> Given the following tree `[3,9,20,null,null,15,7]`:
>
> ```
>     3
>    / \
>   9  20
>     /  \
>    15   7
> ```
>
> Return true.
> **Example 2:**
>
> Given the following tree `[1,2,2,3,3,null,null,4,4]`:
>
> ```
>        1
>       / \
>      2   2
>     / \
>    3   3
>   / \
>  4   4
>
> ```
>
> Return false.

>  分析：递归实现；给出的方法没返回值，需要重新定义新的方法

```java
public boolean isBalanced(TreeNode root) {
    if (root == null) {
        return true;
    }
    return maxDepth(root) != -1;
}

private int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    // 如果左边/右边不是平衡树，就可以判断整棵树都不是
    int leftsize = maxDepth(root.left);
    if (leftsize == -1) {
        return -1;
    }

    int rightsize = maxDepth(root.right);
    if (rightsize == -1) {
        return -1;
    }

    if (Math.abs(rightsize - leftsize) > 1) {
        return -1;
    }
    return 1 + Math.max(leftsize, rightsize);
}
```



---

### _13_MinimumDepthofBinaryTree

> 时间：2018/02/26

> 题意：求二叉树最浅深度(跟叶子节点最接近的距离)

> 分析：
>
> 注意是叶子节点；
>
> 不能直接取Min(left, right)，node的left/right可能为null

```java
public int minDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int left = minDepth(root.left);
    int right = minDepth(root.right);
    // 左或右是空的情况
    if (left == 0 || right == 0) {
        return 1 + left + right;
    }
    // 左跟右都不空
    return 1 + Math.min(left, right);
}
```



---

## 二叉搜索树



### _14_ConvertSortedListtoBinarySearchTree

> 时间：2018/02/27

> 题意：给出一个升序的链表，返回一个高度平衡的搜索树
>
> ```
> Given the sorted linked list: [-10,-3,0,5,9],
>
> One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:
>
>       0
>      / \
>    -3   9
>    /   /
>  -10  5
> ```

> 分析：这题需要将一个排好序的链表转成一个平衡二叉树，我们知道，对于一个二叉树来说，左子树一定小于根节点，而右子树大于根节点。所以我们需要找到链表的中间节点，这个就是根节点，链表的左半部分就是左子树，而右半部分则是右子树，我们继续递归处理相应的左右部分，就能够构造出对应的二叉树了。
>
> 这题的难点在于如何找到链表的中间节点，我们可以通过fast，slow指针来解决，fast每次走两步，slow每次走一步，fast走到结尾，那么slow就是中间节点了。

```java
public TreeNode sortedListToBST(ListNode head) {
    if (head == null) {
        return null;
    }
    return build(head, null);
}

/**
 * 注：最后的元素不包含在内（end可能为null，可能不为null）
 * 实际长度可理解为end - start，比如：1, 2, 3，那么3在以下代码中可以算是null（fast != end && fast.next != end）
 * @param start
 * @param end
 * @return
 */
private TreeNode build(ListNode start, ListNode end) {
    if (start == end) {
        return null;
    }

    ListNode slow = start;
    ListNode fast = start.next; // 当只有2个节点的时候，slow应该是第一个
    while (fast != end && fast.next != end) {
        slow = slow.next;
        fast = fast.next.next;
    }
    TreeNode treeNode = new TreeNode(slow.val);
    treeNode.left = build(start, slow);
    treeNode.right = build(slow.next, end);
    return treeNode;
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788gy1fouv8leqbcj20wt0nl75e.jpg)

---

### _15_ValidateBinarySearchTree

> 时间：2018/02/27

> 题意：给出一个二叉树，判断是否是搜索树

> 分析：
>
> ▲注意：root的left子树要都满足比root小的条件，right同理
>
> 坑：给出的数据可能有以下情况：[-2147483648,null,2147483647]
>
> ![](https://ws1.sinaimg.cn/large/8747d788ly1fov1ugm37ij20ml0dnmxu.jpg)
>
> 新建的方法要用long参数



```java
public boolean isValidBST(TreeNode root) {
    if (root == null) {
        return true;
    }
    return helper(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean helper(TreeNode root, long lower, long upper) {
    if (root == null) {
        return true;
    }
    if (root.val <= lower || root.val >= upper) {
        return false;
    }
    if (helper(root.left, lower, root.val) == false) {
        return false;
    }
    if (helper(root.right, root.val, upper) == false) {
        return false;
    }
    return true;
}
```

自己写的麻烦一点的写法，left，right也写在了方法中：

```java
public boolean isValidBST4(TreeNode root) {
    if (root == null) {
        return true;
    }
    return check(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean check(TreeNode root, long low, long high) {
    if (root.left == null && root.right == null) {
        return true;
    }
    if (root.left != null && !(low < root.left.val && root.left.val < root.val)) {
        return false;
    }
    if (root.right != null && !(root.val < root.right.val && root.right.val < high)) {
        return false;
    }
    boolean check = true;
    if (root.left != null) {
        check = check(root.left, low, root.val);
    }
    boolean check2 = true;
    if (root.right != null) {
        check2 = check(root.right, root.val, high);
    }
    return check & check2;
}
```

错误写法，只比较了左右节点：

```java
public boolean isValidBST3(TreeNode root) {
    if (root == null) {
        return true;
    }
    if (root.left == null && root.right == null) {
        return true;
    }
    if (root.left != null && root.left.val >= root.val) {
        return false;
    }
    if (root.right != null && root.right.val <= root.val) {
        return false;
    }
    return isValidBST3(root.left) & isValidBST3(root.left);
}
```



---

### _16_ConvertSortedArraytoBinarySearchTree

> 时间：2018/02/27

> 题意：给出一个升序数组，构造出BST（二叉搜索树）
>
> ```
> Given the sorted array: [-10,-3,0,5,9],
>
> One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:
>
>       0
>      / \
>    -3   9
>    /   /
>  -10  5
> ```

> 分析：跟14一样，这里更容易操作一点

```java
public TreeNode sortedArrayToBST(int[] nums) {
    if (nums == null || nums.length == 0) {
        return null;
    }

    return buildBST(nums, 0, nums.length - 1);
}

private TreeNode buildBST(int[] nums, int l, int r) {
    // 注意这里
    if (l > r) {
        return null;
    }
    int m = (l + r) / 2;
    TreeNode treeNode = new TreeNode(nums[m]);
    treeNode.left = buildBST(nums, l, m - 1);
    treeNode.right = buildBST(nums, m + 1, r);
    return treeNode;
}
```



---

### _17_KthSmallestElementinaBST

> 时间：2018/02/28

> 题意：求搜索二叉树的第k个节点值

> 分析：
>
> 1、递归，要用全局变量辅助
>
> 2、非递归

```java
int ct;

public int kthSmallest(TreeNode root, int k) {
    if (root == null) {
        return -1;
    }
    int res = kthSmallest(root.left, k);
    if (ct == k) {
        return res;
    }
    ct++;
    if (ct == k) {
        return root.val;
    }
    return kthSmallest(root.right, k);
}
```

```java
public int kthSmallest3(TreeNode root, int k) {
    int cnt = k;
    Deque<TreeNode> stack = new ArrayDeque<>();

    while (root != null) {
        stack.offer(root);
        root = root.left;
    }

    while (!stack.isEmpty()) {
        TreeNode last = stack.pollLast();
        cnt--;
        if (cnt == 0) {
            return last.val;
        }
        last = last.right;
        while (last != null) {
            stack.offer(last);
            last = last.left;
        }
    }
    return -1;
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788ly1fovz216a08j20r20f9dgq.jpg)

---

### _18_ZigZagOrderLevelTraversalBST

> 时间：2018/02/28

> 题意：Given binary tree `[3,9,20,null,null,15,7]`,
>
> ```
>     3
>    / \
>   9  20
>     /  \
>    15   7
>
> ```
>
> return its zigzag level order traversal as:
>
> ```
> [
>   [3],
>   [20,9],
>   [15,7]
> ]
> ```

> 分析：明显是BFS即可，这道题要求的是蛇形遍历，我们可以发现奇数行的遍历仍然可以按照广度优先遍历的方式实现，而对于偶数行，只要翻转一下就好了

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> list = new ArrayList<>();
    if (root == null) {
        return list;
    }

    boolean odd = true;
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode poll = queue.poll();
            level.add(poll.val);
            if (poll.left != null) {
                queue.offer(poll.left);
            }
            if (poll.right != null) {
                queue.offer(poll.right);
            }
        }
        if (odd) {
            list.add(level);
        } else {
            Collections.reverse(level);
            list.add(level);
        }
        odd = !odd;
    }
    return list;
}
```



---

### _19_DeleteNodeinaBST

> 时间：2018/02/28

> 题意：BST中删除指定节点

> 分析：
>
> 删除时，为了保持高度，一般用以下两种做法：
>
> 1、把左子树的最大节点放到删除节点处
>
> 2、把右子树的最小节点放到删除节点处
>
> 这里用第二种

新的方法，2018/02/28

注：相当于删除两次，第一次：key；第二次：右子树最小节点；

而第二次删除时，右子树最小节点的左节点一定是null，所以会在

```java
if (root.left == null) {
    return root.right;
}
```
返回`root.right`，可能为null，也可能不为null；

```java
public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) {
        return null;
    }
    
    if (root.val > key) {
        root.left = deleteNode(root.left, key);
    } else if (root.val < key) {
        root.right = deleteNode(root.right, key);
    } else {
        // 注：不返回当前root，就相当于当前root删除
        // 左右其中为空，返回另外一个
        if (root.left == null) {
            return root.right;
        }
        if (root.right == null) {
            return root.left;
        }
    
        // 左右都不空，返回右子树最小节点
        TreeNode rightSmallest = root.right;
        while (rightSmallest.left != null) {
            rightSmallest = rightSmallest.left;
        }
        // 以下顺序不能调转
        // 因为如果先rightSmallest.left = root.left; 那么递归root.right时，最小节点的left就不为空，会出现期望外的结果
        rightSmallest.right = deleteNode(root.right, rightSmallest.val);
        rightSmallest.left = root.left;
        return rightSmallest;
    }
    return root;
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788ly1fow6j60pb8j21g60i5tad.jpg)

---

以下方法只是找到右子树最小节点，然后赋值，如果treeNode有多个值，就不推荐这么写

```java
public TreeNode deleteNode(TreeNode root, int key) {

    if (root == null) {
        return null;
    }
    
    if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = deleteNode(root.right, key);
    } else {
        if (root.left != null && root.right != null) {
            root.val = findMin(root.right).val; // 找出右节点最小的填到要删除的节点处
            root.right = deleteNode(root.right, root.val); // 删除右子树最小的节点
        } else {
            return root.left != null ? root.left : root.right;
        }
    }
    return root;
}

private TreeNode findMin(TreeNode node) {
    while (node.left != null) {
        node = node.left;
    }
    return node;
}
```



---

### _20_LowestCommonAncestorBST

> 时间：2018/03/01

> 题意：求BST的LCA（最小公共祖先）

> 分析：
>
> 因为是BST，所以每一次判断，能够去掉另一半的可能
>
> 1、递归
>
> 2、非递归
>
> 但是：以下算法都是有BUG的，如果p跟q不在BST上，讲道理了应该是返回null的，但是却返回了不准确的值；
>
> 要想正确的话，可以用28的解法，左右子树都要判断

```java
/**
 * 非递归
 * 注意：这里用乘法判断是因为，p跟q的大小不确定
 *
 * @param root
 * @param p
 * @param q
 * @return
 */
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while ((root.val - p.val) * (root.val - q.val) > 0) {
        root = p.val < root.val ? root.left : root.right;
    }
    return root;
}
```



```java
/**
 * 递归，一行搞定
 *
 * @param root
 * @param p
 * @param q
 * @return
 */
public TreeNode lowestCommonAncestor3(TreeNode root, TreeNode p, TreeNode q) {
    return (root.val - p.val) * (root.val - q.val) < 1 ? root :
            lowestCommonAncestor3(p.val < root.val ? root.left : root.right, p, q);
}
```



---

### _22_BinaryTreeRightSideView

> 时间：2018/03/01

> 题意：求二叉树的右侧节点列表

> 分析：可将问题转化为其每一层的最右节点列表
>
> 解法1：BFS
>
> 解法2：递归，需要辅助的变量帮助判断（可定义全局变量、也可定义新方法中包含该辅助变量）

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) {
        return result;
    }

    Deque<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        Integer val = null;
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.pollFirst();
            if (i == size - 1) {
                val = node.val;
            }
            if (node.left != null) {
                queue.offerLast(node.left);
            }
            if (node.right != null) {
                queue.offerLast(node.right);
            }
        }
        result.add(val);
    }
    return result;
}
```

```java
/**
 * 解法2：先遍历右节点
 *
 * @param root
 * @return
 */
public List<Integer> rightSideView2(TreeNode root) {
    if (root == null) {
        return new LinkedList<>();
    }
    List<Integer> lst = new LinkedList<>();
    helper(root, 0, lst);
    return lst;
}

/**
 * 这种递归写法，靠右的都会先被遍历到
 *
 * @param root    当前节点
 * @param current 第几层
 * @param lst     结果集
 */
private void helper(TreeNode root, int current, List<Integer> lst) {
    if (root == null) {
        return;
    }
    // 判断能否添加节点，current相当于第几层
    if (lst.size() <= current) {
        lst.add(root.val);
    }
    helper(root.right, current + 1, lst);
    helper(root.left, current + 1, lst);
}
```

注意这种情况：

![](https://ws1.sinaimg.cn/large/8747d788ly1fox689yvd2j20o30eggm6.jpg)

返回的是[1, 3, 4]

---

### _23_FindModeInBST

> 时间：2018/03/01

> 题意：在一棵存在重复的搜索树中，找出出现次数最多的节点

> 分析：
>
> 方法一：利用搜索树特性，用中序遍历；需要遍历两次，第一次确定出现的最多次数，第二次根据第一次遍历知道的最大值添加答案
>
> 方法二：使用hashmap

```java
int maxCount = 0;
int curCount = 0;
int modeCount;
int[] modes;
int curVal;

public int[] findMode(TreeNode root) {
    inorder(root); // 第一次遍历只是确定最多有多少个
    modes = new int[modeCount];
    modeCount = 0;
    curCount = 0;
    inorder(root); // 第二次遍历才构造数组
    return modes;
}

private void inorder(TreeNode root) {
    if (root == null) {
        return;
    }
    inorder(root.left);
    handleValue(root.val);
    inorder(root.right);
}

/**
 * 处理方法
 * @param value
 */
private void handleValue(int value) {
    if (value != curVal) {
        curVal = value;
        curCount = 0;
    }
    curCount++;
    if (curCount > maxCount) {
        maxCount = curCount;
        modeCount = 1;
    } else if (curCount == maxCount) {
        // 第一次遍历时，modes为null
        if (modes != null) {
            modes[modeCount] = value;
        }
        modeCount++;
    }
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788ly1foxbhtgpsdj215m0lkabw.jpg)

方法二：

```java
int max = 1;

/**
 * 解法二，借助map
 *
 * @param root
 * @return
 */
public int[] findMode(TreeNode root) {
    if (root == null) {
        return new int[0];
    }
    Map<Integer, Integer> map = new HashMap<>();
    find(root, map);
    int[] ints = map.entrySet().stream()
            .filter(integerIntegerEntry -> integerIntegerEntry.getValue() == max)
            .map(integerIntegerEntry1 -> integerIntegerEntry1.getKey())
            .collect(Collectors.toList())
            .stream()
            .mapToInt(Integer::intValue)
            .toArray();
    /*List<Integer> list = new ArrayList<>();
    for (Integer integer : map.keySet()) {
        if (map.get(integer) == max) {
            list.add(integer);
        }
    }
    int[] a = new int[list.size()];
    int i = 0;
    for (Integer integer : list) {
        a[i++] = integer;
    }*/

    return ints;
}

private void find(TreeNode root, Map<Integer, Integer> map) {
    if (root == null) {
        return;
    }
    if (map.containsKey(root.val)) {
        int val = map.get(root.val) + 1;
        map.put(root.val, val);
        max = Math.max(max, val);
    } else {
        map.put(root.val, 1);
    }
    find(root.left, map);
    find(root.right, map);
}
```





---

### _24_MostFrequentSubtreeSum

> 时间：2018/03/09

> 题意：给出一个二叉树，求子树的和中，出现最多的和次数最多的是哪些

> 分析：
>
> 推理条件：n各节点，对应n颗子树；
>
> 后序遍历 + hashMap
>
> 跟_23_FindModeinBST差不多的做法

```java
int max = 1;

public int[] findFrequentTreeSum(TreeNode root) {
    if (root == null) {
        return new int[0];
    }
    Map<Integer, Integer> map = new HashMap<>();
    getSum(root, map);

    int[] result = map.entrySet().stream()
            .filter(integerIntegerEntry -> integerIntegerEntry.getValue().equals(max))
            .map(integerIntegerEntry -> integerIntegerEntry.getKey())
            .collect(Collectors.toList())
            .stream()
            .mapToInt(Integer::intValue)
            .toArray();
    return result;
}

private int getSum(TreeNode root, Map<Integer, Integer> map) {
    if (root == null) {
        return 0;
    }
    int leftSum = getSum(root.left, map);
    int rightSum = getSum(root.right, map);
    int sum = leftSum + rightSum + root.val;
    int val = map.getOrDefault(sum, 0) + 1;
    map.put(sum, val);
    max = Math.max(max, val);
    return sum;
}
```



---

### _25_FindLargestElementinEachRow

> 时间：2018/03/11

> 题意：给出一个二叉树，返回每一行的最大节点val

> 分析：
>
> 解法一：非递归，BFS
>
> 解法二：递归，递归方法添加dept形参判断是第几层

```java
public List<Integer> largestValues(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) {
        return result;
    }
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int length = queue.size();
        int max = Integer.MIN_VALUE;
        for (int i = 0; i < length; i++) {
            TreeNode node = queue.poll();
            if (node.val > max) {
                max = node.val;
            }
            if (node.left != null) {
                queue.add(node.left);
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }
        result.add(max);
    }
    return result;
}
```



```java
public List<Integer> largestValues2(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    helper(root, res, 0);
    return res;
}

private void helper(TreeNode root, List<Integer> res, int depth) {
    if (root == null) {
        return;
    }
    // 每层第一个的条件，0层->size=0,1层->size=1
    if (depth == res.size()) {
        res.add(root.val);
        // 每层其他节点的条件
    } else if (root.val > res.get(depth)) {
        res.set(depth, root.val);
    }
    helper(root.left, res, depth + 1);
    helper(root.right, res, depth + 1);
}
```





---

### _26_SerializeAndDeserializeBST

> 时间：2018/03/11

> 题意：给出一颗搜索树，设计序列化，反序列化方法

> 分析：
>
> 注意点：搜索树
>
> 序列化可以用先序遍历
>
> 反序列化时，第一个是root，while到比root大的就是右节点，递归即可

```java
public class _26_SerializeAndDeserializeBST {
    private static final String SEP = ",";
    private static final String NULL = "null";

    // 先序遍历（非递归）
    public String serialize(TreeNode root) {
        if (root == null) {
            return NULL;
        }
        Deque<TreeNode> stack = new ArrayDeque<>();
        stack.offer(root);
        StringBuilder sb = new StringBuilder();
        while (!stack.isEmpty()) {
            TreeNode node = stack.pollLast();
            sb.append(node.val).append(SEP);
            if (node.right != null) {
                stack.offerLast(node.right);
            }
            if (node.left != null) {
                stack.offerLast(node.left);
            }
        }
        return sb.toString();
    }


    // 反序列
    public TreeNode deserialize(String data) {
        if (data.equals(NULL)) {
            return null;
        }
        Deque<Integer> queue = new ArrayDeque<>();
        String[] split = data.split(SEP);
        for (String s : split) {
            queue.offer(Integer.parseInt(s));
        }
        return getNode(queue);
    }

    // some notes:
    //   5
    //  3 6
    // 2   7
    private TreeNode getNode(Deque<Integer> q) {  //q: 5,3,2,6,7
        if (q.isEmpty()) return null; // 注意F
        Integer root = q.poll();
        TreeNode node = new TreeNode(root);  //root (5)
        Deque<Integer> leftQue = new ArrayDeque<>();
        while (!q.isEmpty() && q.peek() <= root) {
            leftQue.offer(q.poll());
        }
        // smallerQueue : 3,2   storing elements smaller than 5 (root)
        node.left = getNode(leftQue);
        // q: 6,7   storing elements bigger than 5 (root)
        node.right = getNode(q);
        return node;
    }
}
```





---

### _27_SerializeAndDeserializeBT

> 时间：2018/03/11

> 题意：设计二叉树的序列化跟反序列化方法

> 分析：
>
> 解法一：
>
> 序列化：先序遍历（递归/非递归都可以）
>
> 反序列化：知道根节点，递归
>
> 解法二：
>
> 序列化：BFS
>
> 反序列化：for +i,j组合left、right

```java
/**
 * 使用前序遍历将null也存起来，不存起来的话无法根据这样的前序遍历数组反序列化成功
 * 而且使用前序遍历的另一个原因是，可以马上知道根节点
 *
 * @param root
 * @return
 */
// Encodes a tree to a single string.
public String serialize(TreeNode root) {
    if (root == null) {
        return "";
    }
    StringBuilder sb = new StringBuilder();
    serialHelper(root, sb);
    return sb.substring(0, sb.length() - 1);
}

private void serialHelper(TreeNode root, StringBuilder sb) {
    if (root == null) {
        sb.append("#,");
    } else {
        sb.append(root.val).append(",");
        serialHelper(root.left, sb);
        serialHelper(root.right, sb);
    }
}

// Decodes your encoded data to tree.
public TreeNode deserialize(String data) {
    if (data == null || data.length() == 0) {
        return null;
    }

    java.util.StringTokenizer st = new java.util.StringTokenizer(data, ",");
    return deserialHelper(st);
}

private TreeNode deserialHelper(java.util.StringTokenizer st) {
    if (!st.hasMoreTokens()) {
        return null;
    }
    String val = st.nextToken();
    if (val.equals("#")) {
        return null;
    }
    TreeNode root = new TreeNode(Integer.parseInt(val));
    root.left = deserialHelper(st);
    root.right = deserialHelper(st);
    return root;
}
```

```java
/**
 * 解法二：BFS，把null也保存起来
 */
public String serialize2(TreeNode root) {
    if (root == null) {
        return "";
    }
    List<Integer> list = new ArrayList<>();
    Deque<TreeNode> que = new LinkedList<>(); // ArrayDeque不能添加null，所以用linkedList
    que.offer(root);
    while (!que.isEmpty()) {
        int length = que.size();
        for (int i = 0; i < length; i++) {
            TreeNode node = que.pollFirst();
            list.add(node == null ? null : node.val);
            if (node != null) {
                que.offer(node.left);
                que.offer(node.right);
            }
        }
    }
    return list.stream().map(String::valueOf).collect(Collectors.joining(","));
}

// Decodes your encoded data to tree.

/**
 * First we make TreeNode array and set set created TreeNode or null values to T array.
 * then we set left and right children of non-null values of T array.
 */
public TreeNode deserialize2(String data) {
    if (data.equals("")) {
        return null;
    }
    String[] a = data.split(",");
    TreeNode[] t = new TreeNode[a.length];
    for (int i = 0; i < a.length; i++) {
        t[i] = getNode(a[i]);
    }
    int j = 1;
    for (int i = 0; i < a.length && j < a.length - 2; i++) {
        if (t[i] != null) {
            t[i].left = t[j++];
            t[i].right = t[j++];
        }
    }

    return t[0];
}

TreeNode getNode(String s) {
    if (s.equals("null")) {
        return null;
    }
    return new TreeNode(Integer.parseInt(s));
}
```

---

### _28_LowestCommonAncestorofaBinaryTree

> 时间：2018/03/01

> 题意：求二叉树LCA

> 分析：我们仍然可以用递归来解决，递归寻找两个带查询LCA的节点p和q，当找到后，返回给它们的父亲。如果某个节点的左右子树分别包括这两个节点，那么这个节点必然是所求的解，返回该节点。否则，返回左或者右子树（哪个包含p或者q的就返回哪个）。
>
> [参考](https://www.hrwhisper.me/algorithm-lowest-common-ancestor-of-a-binary-tree)

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) {
        return root;
    }
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) {
        return root;
    }
    return left == null ? right : left;
}
```

