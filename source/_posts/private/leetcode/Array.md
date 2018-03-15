---
title: Array
date: 2018-2-16 21:12
toc: true
---
# Array

## _01_RotateArray

> 时间：2017/12/28、2018/01/30、2018/03/12

> 题意：给出一个数组，将其向右移动k位，并返回

```java
public void rotate(int[] nums, int k) {
        if (nums == null) {
            throw new NullPointerException("数组不能为空");
        }
        int length = nums.length;
        k %= length;
        if (k == 0) {
            return;
        }
        reverse(nums, 0, length - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, length - 1);
    }

    public static void reverse(int[] nums, int left, int right) {
        while (left < right) {
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;
            left++;
            right--;
        }
    }
```

1. 方法一：先整体翻转，再将两部分各自翻转即可

   ![](https://ws1.sinaimg.cn/large/8747d788ly1fnzjxj2f8sj20ug09dglo.jpg)

2. 自己计算每次要移动的下标，总共移动了`nums.length`次之后，就成功了

   ```java
   /**
    * 方法2：自己计算每次要移动的下标，总共移动了nums.length次之后，就成功了
    * 这是一种纯操作的方式
    *
    * @param nums
    * @param k
    */
   public void rotate2(int[] nums, int k) {
       if (nums == null) {
           throw new NullPointerException("数组不能为空");
       }
       k = k % nums.length;
       if (k == 0) {
           return;
       }
       int count = 0;
       for (int start = 0; count < nums.length; start++) {
           int currentIndex = start;
           int curVal = nums[start];
           do {
               int nextIndex = (currentIndex + k) % nums.length;
               int nextVal = nums[nextIndex];
               nums[nextIndex] = curVal;
               curVal = nextVal;
               currentIndex = nextIndex;
               count++;
           } while (currentIndex != start); // 有可能形成环,有环的情况下要跳出，不然会死循环
       }
   }
   ```


---



## _02_Contains_Duplicate

> 时间：2018/01/31、2018/03/12

> 题意：给出一个数组，判断是否有重复元素
1. 快排，循环判断


```java
public boolean containsDuplicate2(int[] nums) {
    Arrays.sort(nums);
    for (int ind = 1; ind < nums.length; ind++) {
        if (nums[ind] == nums[ind - 1]) {
            return true;
        }
    }
    return false;
}
```

1. 使用Set
   1. contains判断

   ```java
   public static boolean containsDuplicate3(int[] nums) {
       final Set<Integer> distinct = new HashSet<>();
       for (int num : nums) {
           if (distinct.contains(num)) {
               return true;
           }
           distinct.add(num);
       }
       return false;
   }
   ```

   2. 或者根据add()返回值判断；true：没添加过

   ```java
   public static boolean containsDuplicate4(int[] nums) {
       Set<Integer> set = new HashSet<>();
       for (int i : nums) {
           if (!set.add(i)) {
               return true;
           }
       }
       return false;
   }
   ```


---



## _03_Find_Peak_Element

> 时间：2018/01/31、2018/03/13

> 题意：给定一个左右相邻不相等的数组，找出一个局部最大值。PS：num[-1]=num[n]=-∞

```java
class Solution {
    public int findPeakElement(int[] nums) {
       return binarySearch(nums, 0, nums.length - 1);
    }

    private int binarySearch(int[] nums, int left, int right) {
        if (left == right) {
            return left;
        }
        // 每次筛选都会去掉部分下标
        int mid = (left + right) / 2;
        if (nums[mid] > nums[mid + 1]) { // 不存在'='的
            return binarySearch(nums, left, mid); // 右边的没了
        }
        return binarySearch(nums, mid + 1, right); // 左边的没了
        // 最后一次应该剩下2个，比如下标3跟4，mid=3；一比较就能得到3，3或者4,4，下轮递归就结束了
    }
}
```

* 二分法；
  * why：复杂度低，$log_2N$，根据mid跟mid+1可以排除一半 
* 每次筛选的区间，[left ~ mid] 或者 [mid+1 ~ right]
  ![](https://ws1.sinaimg.cn/large/8747d788ly1fnzvbs27oqj212d0b7aaa.jpg)
* 结束的条件：left == right
* 注意：
  * mid使用(left + right)/2时
  * 要比较的是mid + 1,不能跟mid - 1比较；因为如果剩下两个元素时，mid就不是正确的(不在比较范围内)


---



## _04_Maximum_Subarray 

> 时间：2018/02/09、2018/03/13

> 题意：求给出数组最长连续子序列之和

> 分析：注意要连续

```java
	public int maxSubArray(int[] nums) {
        int length = nums.length;
        int dp = nums[0];
        int max = dp;
        for (int i = 1; i < length; i++) {
            int temp = nums[i] + (dp > 0 ? dp : 0); // 因为要连续，所以nums[i]时一定要加的
            dp = temp;
            max = Math.max(temp, max);
        }
        return max;
    }
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpbb7eiholj21ff0jhjse.jpg)

---

## _05_KthLargestElementinanArray

> 时间：2018/02/14、2018/03/13

> 题意：在数组中找到第k大的元素

> 分析：几种解法： 
>
> 1、O(N lg N) running time + O(1) memory 直接快排，选出nums[k]
>
> 2、O(N lg K) running time + O(K) memory 使用优先队列
>
> 3、快速选择算法(这里选择使用该算法)
>
> ![图解](https://upload.wikimedia.org/wikipedia/commons/0/04/Selecting_quickselect_frames.gif)

```java
    public int findKthLargest(int[] nums, int k) {
        return partition(nums, 0, nums.length - 1, nums.length - k);
    }

    int partition(int[] a, int left, int right, int k) {
        int p = (left + right) / 2; // 选择pivot
        int curIndex = left;

        // 把小于pivot的元素从左堆到右
        swap(a, p, right); // 先弄到最后
        for (int i = left; i < right; i++) {
            if (a[i] < a[right]) {
                swap(a, i, curIndex++);
            }
        }
        swap(a, curIndex, right); // 记得还原

        if (curIndex == k) {
            return a[k];
        } else if (curIndex < k) { // k在右边，继续筛选
            return partition(a, curIndex + 1, right, k);
        } else { // k在左边，继续筛选
            return partition(a, left, curIndex - 1, k);
        }
    }

    private void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
```

复杂度：

| 最坏时间复杂度 | О(*n*2) e.g.已排好序的(降序)，找最后一个，n + (n - 1)+ (n - 2) + ... + 1 |
| ------- | ---------------------------------------- |
| 最优时间复杂度 | О(*n*)                                   |
| 平均时间复杂度 | O(*n*)                                   |
| 空间复杂度   | O(*1*)                                   |

---

## _06_FindAllDuplicatesinanArray

> 时间：2018/02/14、2018/03/13
>

> 题意：给出一个数组: 1 ≤ a[i] ≤ n (n = size of array), some elements appear twice and others appear once. // 有些出现2次，有些出现1次;
>
> Find all the elements that appear twice in this array. // 找出出现2次的数字

> 分析：因为1 ≤ a[i] ≤ n，可以用数组解决（如果a[i]是int就难搞了）；将每次出现的a[a[i]]变相反数，那么第二次判断的时候，就能知道出现第二次了

```java
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < nums.length; i++) {
            int index = Math.abs(nums[i]) - 1;
            if (nums[index] < 0) { // < 0证明是第二次出现
                res.add(Math.abs(nums[i]));
            }
            nums[index] = -nums[index]; // 把对应的数字变为相反数，用来判断是否出现第二次
        }
        return res;
    }
```

注意：这里是直接操作nums，如果不想改变nums的话，再复制一个吧。

---

## _07_MaxIncreasingSubsequence

> 时间：2018/02/14、2018/03/13

> 给出一个未排序的数组，求出最长增长子序列(不连续)长度
>
> Given `[10, 9, 2, 5, 3, 7, 101, 18]`,
> The longest increasing subsequence is `[2, 3, 7, 101]`, therefore the length is `4`

`tails` is an array storing the smallest tail of all increasing subsequences with length `i+1` in `tails[i]`.
For example, say we have `nums = [4,5,6,3]`, then all the available increasing subsequences are:

```
len = 1   :      [4], [5], [6], [3]   => tails[0] = 3
len = 2   :      [4, 5], [5, 6]       => tails[1] = 5
len = 3   :      [4, 5, 6]            => tails[2] = 6
```

We can easily prove that tails is a increasing array. Therefore it is possible to do a binary search in tails array to find the one needs update.

Each time we only do one of the two:

```
(1) if x is larger than all tails, append it, increase the size by 1
(2) if tails[i-1] < x <= tails[i], update tails[i]
```

Doing so will maintain the tails invariant. The the final answer is just the size.

**Java**

```java
public int lengthOfLIS(int[] nums) {
    int[] tails = new int[nums.length];
    int size = 0;
    for (int x : nums) {
        int i = 0, j = size;
        while (i != j) {
            int m = (i + j) / 2;
            if (tails[m] < x) // 以大于为判断标准，符合才往右移动
                i = m + 1;
            else
                j = m; // m = (i + j) / 2 这种写法j跟m是不相等的，在i==j之前，j都会大于m，所以不会死循环;如果是==的话，不会增长size
        }
        tails[i] = x; // x会在tails[]中找到合适的位置插入
        if (i == size)
            ++size; // size未赋值，找到了长度能加1就写进去（上面的tails[i]相当于tails[size]）
    }
    return size;
}
```

图例：

![](https://ws1.sinaimg.cn/large/8747d788gy1fpbij5d0ymj21dh0xjtb2.jpg)

debug流程：

1. nums：![](https://ws1.sinaimg.cn/large/8747d788gy1foh7sc8jvmj20mf01fa9y.jpg)
2. tails：
   1. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7tbmp1mj20k101ja9x.jpg)
   2. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7tp16sfj20k7019dfp.jpg)
   3. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7twmybej20kv01h3ye.jpg)
   4. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7u5k5c0j20la01gjra.jpg)
   5. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7uh0nqmj20ll01gjra.jpg)
   6. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7x6quibj20ld01e746.jpg)
   7. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7uvj4ehj20ko01g746.jpg)
   8. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7v9yq0hj20k001g746.jpg)
   9. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7vfo67bj20ke01eq2u.jpg)
   10. ![](https://ws1.sinaimg.cn/large/8747d788gy1foh7vmwko0j20k101fgli.jpg)

* 方法2

```java
public int lengthOfLIS2(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int result = 1;
    int[] dp = new int[nums.length];
    for (int i = 0; i < dp.length; i++) {
        dp[i] = 1;
    }
    // 有可能：[1,2,11,12,13,3,4,5,6,7]；没办法一步得到最长，只好都遍历计算；当然上面第一种算法挺屌的
    for (int i = 0; i < nums.length; i++) {
        int maxl = 1;
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                maxl = Math.max(maxl, dp[j] + 1);
            }
        }
        dp[i] = maxl;
        result = Math.max(maxl, result);
    }
    return result;
}
```

---

## _08_RotateImage

> 时间：2018/02/15、2018/03/14

> 给出一个二维数组，将其顺时针翻转90度

> ```
> 对于90度的翻转有很多方法，一步或多步都可以解，我们先来看一种直接的方法，对于当前位置，计算旋转后的新位置，
> 然后再计算下一个新位置，第四个位置又变成当前位置了，所以这个方法每次循环换四个数字，如下所示：
> 1  2  3                 7  2  1                  7  4  1
> 4  5  6        -->      4  5  6        -->       8  5  2
> 7  8  9                 9  8  3　　        　　　 9  6  3
> 即从最外层开始循环，一直到最内层
> ```

```java
public void rotate(int[][] matrix) {
    if (matrix == null) {
        return;
    }
    int n = matrix.length;
    for (int i = 0; i < n / 2; i++) {
        for (int j = i; j < n - 1 - i; j++) { // 这里注意：不能加'='号
            int temp = matrix[i][j];
            matrix[i][j] = matrix[n - 1 - j][i];
            matrix[n - 1 - j][i] = matrix[n - 1 - i][n - 1 - j];
            matrix[n - 1 - i][n - 1 - j] = matrix[j][n - 1 - i];
            matrix[j][n - 1 - i] = temp;
        }
    }
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788ly1fpc5eqhvr5j212b0rsdh3.jpg)

---

## _09_ShuffleanArray

> 时间：2018/02/15、2018/03/15

> 题意：给出一个int数组，设计算法让其打乱（等概率）；[题目](https://leetcode.com/problems/shuffle-an-array/#/description)

> 分析：Fisher–Yates shuffle算法；[证明](https://www.kancloud.cn/kancloud/data-structure-and-algorithm-notes/72928)；
>
> 类似的的问题：随机抽样

```java
public class _09_ShuffleanArray {
    int[] nums;
    int[] cards;

    public _09_ShuffleanArray(int[] nums) {
        this.nums = nums;
        cards = new int[nums.length];
        System.arraycopy(nums, 0, cards, 0, nums.length);
    }

    /**
     * Resets the array to its original configuration and return it.
     */
    public int[] reset() {
        return nums;
    }

    /**
     * Returns cards random shuffling of the array.
     */
    public int[] shuffle() {
        if (cards == null || cards.length == 0)
            return new int[0];

        Random random = new Random();
        for (int i = 1; i < cards.length; i++) {
            int j = random.nextInt(i + 1);
            int temp = cards[j];
            cards[j] = cards[i];
            cards[i] = temp;
        }
        return cards;
    }
}
```

```java
/*
 * shuffle cards
 */
public static void shuffleCard(int[] cards) {
  if (cards == null || cards.length == 0) return;

  Random rand = new Random();
  for (int i = 0; i < cards.length; i++) {
    int k = rand.nextInt(i + 1); // 0~i (inclusive)
    int temp = cards[i];
    cards[i] = cards[k];
    cards[k] = temp;
  }
}
```

---

## _10_FindMinimuminRotatedSortedArray

> 时间：2018/02/15、2018/03/15

> 题目：给出一个旋转之后的数组（有序），找出其最小值

> 分析：
>
> 1、最直接的想法，遍历一边，时间：$O(n)$；
>
> 2、二分；$O(log_2n)$

```java
public int findMin(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int indexL = 0;
    int indexR = nums.length - 1;
    int result = 0; // 坑1：处理已经排好序的了
    while (nums[indexL] >= nums[indexR]) {
        if (indexR - indexL == 1) {
            result = indexR;
            break;
        }
        int indexMid = (indexL + indexR) / 2;
        // 坑2：处理三个值一样大小的情况：
        // indexL、indexMid、indexR指向的三个数字相等， 则只能顺序查找
        if (nums[indexL] == nums[indexR] && nums[indexL] == nums[indexMid]) {
            return minInOrder(nums, indexL, indexR);
        }

        if (nums[indexMid] >= nums[indexL]) {
            indexL = indexMid;
        } else {
            indexR = indexMid;
        }
    }
    return nums[result];
}

private int minInOrder(int[] nums, int indexL, int indexR) {
    int result = nums[indexL];
    for (int i = indexL + 1; i <= indexR; i++) {
        if (result > nums[i]) {
            result = nums[i];
        }
    }
    return result;
}
```

注意：

while结束条件：
```java
if (indexR - indexL == 1) {
  result = indexR;
  break;
}
```

**L跟R相差1！！！**

如何做到？以下判断条件：**都不是+1 / -1，保证mid永远在上面或者永远在下面**

```Java
    if (nums[indexMid] >= nums[indexL]) {
        indexL = indexMid;
    } else {
        indexR = indexMid;
    }
```


 图解：



![](https://ws1.sinaimg.cn/large/8747d788gy1fohk6nvz1dj20ib0fodhk.jpg)

3个值相等的情况（不能判断mid是在左边，还是在右边）：
![](https://ws1.sinaimg.cn/large/8747d788gy1fohk7a9i95j20vp05r75l.jpg)

---

## _11_SearchinRotatedSortedArray

> 时间：2018/02/16、2018/03/15

> 题意：给出一个"旋转"过的有序数组，找出target，有则返回其下标，否则返回-1

> 分析：二分法

```java
public int search(int[] nums, int target) {
    if (nums == null || nums.length == 0) {
        return -1;
    }
    int l = 0;
    int r = nums.length - 1;
    while (l <= r) { // 要用<=，例子：nums{1},如果用!=，会直接返回-1
        int m = l + (r - l) / 2;
        // 把m的过滤掉先，下面就不用判断了，直接m+1/m-1
        if (target == nums[m]) {
            return m;
        }
        if (nums[m] >= nums[l]) {
            // m不能用=，上面已经判断过了
            if (target >= nums[l] && target < nums[m]) {
                r = m - 1;
            } else {
                l = m + 1;
            }
        } else {
            // m不能用=，上面已经判断过了
            if (target > nums[m] && target <= nums[r]) {
                l = m + 1;
            } else {
                r = m - 1;
            }

        }
    }
    return -1;
}
```

图解：

![](https://ws1.sinaimg.cn/large/8747d788gy1foil6klha1j21fw0knwfi.jpg)







