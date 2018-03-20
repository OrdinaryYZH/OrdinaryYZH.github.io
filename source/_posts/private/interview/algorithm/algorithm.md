#### 1.二分查找

> 总结：
>
> 1、注意return的时机，return就证明已经确定了
>
> 2、注意while(l < r)，还是while(l <= r)
>
> 3、注意三种情况逐一判断即可，大于、小于和等于

##### 1.1 常规

```java
public static int binary_search(int[] arr, int start, int end, int khey) {
    int mid;
    while (start <= end) {
        mid = (start + end) / 2;
        if (arr[mid] < khey)
            start = mid + 1;
        else if (arr[mid] > khey)
            end = mid - 1;
        else
            return mid;
    }
    return -1;
}
```

##### 1.2 变种

> 存在重复元素的情况，以下列出一些变种的情况

1. key的最小下标 

    ```java
    public static int searchFirstEqual(int[] arr, int key) {
       int length = arr.length;
       int left = 0, right = length - 1;
       while (left < right)//根据两指针的意义，如果key存在于数组，left==right相等时已经得到解
       {
           int mid = (left + right) / 2;
           if (arr[mid] > key) {
               //一定在mid为止的左边，并且不包含当前位置
               right = mid - 1;
           } else if (arr[mid] < key) {
               //一定在mid位置的右边，并且不包括当前mid位置
               left = mid + 1;
           } else {
               // 只能确定<=mid（因为求最小下标，如果最大下标，就是>=mid）
               right = mid;
           }
       }
       if (arr[left] == key) {
           return left;
       }
       return -1;
    }
    ```


2. 找出最后一个与key相等的元素的位置

   ```java
   public static int searchLastEqual(int[] arr, int key) {
       int length = arr.length;
       int left = 0, right = length - 1;
       while (left < right - 1) {
           int mid = (left + right) / 2;
           if (arr[mid] > key) {//key一定在mid位置的左边，并且不包括当前mid位置
               right = mid - 1;
           } else if (arr[mid] < key) {//key一定在mid位置的右边，相等时答案有可能是当前mid位置
               left = mid + 1;
           } else {
               left = mid;
           }
       }
       if (arr[left] <= key && arr[right] == key) {
           return right;
       }
       if (arr[left] == key && arr[right] > key) {
           return left;
       }
       return -1;
   }
   ```

3. 查找第一个>=Key的元素的位置

   ```java
   public static int searchFirstEqualOrLarger(int[] arr, int key) {
       int length = arr.length;
       int left = 0, right = length - 1;
       while (left < right) {
           int mid = (left + right) / 2;
           if (arr[mid] > key) {
               right = mid - 1;
           } else if (arr[mid] < key) {
               left = mid + 1;
           } else {
               right = mid;
           }
       }
       return left;
   }
   ```

4. 查找第一个>key的元素的位置

   ```java
   public static int searchFirstLarger(int[] arr, int key) {
       int length = arr.length;
       int left = 0, right = length - 1;
       while (left <= right) {
           int mid = (left + right) / 2;
           // 这里可能mid-1减多了，可能mid-1就是key的下标，但是后面left+1会加回来，而且循环条件是left <= right
           if (arr[mid] > key) {
               right = mid - 1;
           } else if (arr[mid] <= key) {
               left = mid + 1;
           }
       }
       if (left >= arr.length) {
           return -1;
       }
       if (arr[left] > key) {
           return left;
       }
       return -1;
   }
   ```

5. 查找最后一个<=key的元素的位置

   ```java
   public static int searchLastEqualOrSmaller(int[] arr, int key) {
       int length = arr.length;
       int left = 0, right = length - 1;
       while (left <= right) {
           int m = (left + right) / 2;
           if (arr[m] > key) {
               right = m - 1;
           } else if (arr[m] < key) {
           // 这里有可能加多了，+1之后，可能arr[left]>=key;
           // 但是加多了之后，right会减回来，如果不存在相等情况，那么最后right会在left的左边，最后返回right即系答案
           left = m + 1;
           } else {
               left = m + 1;
           }
       }
       return right;
   }
   ```

6. 查找最后一个小于key的元素的位置

   ```java
   public static int searchLastSmaller(int[] arr, int key) {
       int length = arr.length;
       int left = 0, right = length - 1;
       while (left <= right) {
           int mid = (left + right) / 2;
           if (arr[mid] >= key) {
               right = mid - 1;
           } else if (arr[mid] < key) {
               left = mid + 1;
           }
       }
       return right;
   }
   ```

