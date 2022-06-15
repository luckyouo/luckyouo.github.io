# 二分查找


## 前言

二分查找一般是用来在有序序列中查找某个范围的数据，通过不断缩小范围来求解，但也可以搭配其他算法来求解其他问题。

## 二分搜索问题

这是学习二分查找时，解决的最基本的问题。求解有序数组小于等于 target 的最大值

```java
public int binarySearch(int[] nums, int left, int right, int taget){
    while(left < right){
        int mid = (left + right) / 2;
        if(nums[mid] >= target){
            right = mid;
        }else{
            left = mid+1;
        }
    }
    return left;
}
```

上述算法是常用的二分左边界搜索，二分根据循环条件不同，可以有中间随机搜索、左边界搜索和右边界搜索

#### 中间随机搜索

该搜索的循环条件为 $left<=right$，若未查询到目标值，返回 $-1$，查询到目标值，则直接返回。由于查询是查询到目标值直接返回，所以对于含有**重复数据**时，则按照二分顺序返回对应所以，可能为左端、右端或者中间任意一索引，所以，该方法一般只适用于数组中不含有重复数据。

注意：

1. 左右边界必须都是闭区间，
2. 每次左右区间变化都是 $left+1$ 或者 $right-1$ 

**Java Arrays 内部实现的 binarySearch 采用的就是该搜索方式**

```java
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}
```

#### 左边界搜索

该搜索的常见的循环条件为 $left < right$，在循环的过程中，即使已经遍历到了目标值时，也仍继续循环，直到左右边界指向同一元素。当数组有重复值时，为了寻找最左端的边界，当 $arr[mid] <= target$，应该将 $right=mid$，使左右边界尽量往左靠，而每次的判断边界情况应该为 $[left, right)$，直到 $arr[left] == arr[right]$ 退出循环，即二者同时指向目标值。

```java
int left_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0;
    int right = nums.length; // 注意
    
    while (left < right) { // 注意
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            right = mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; // 注意
        }
    }
    return left;
}

int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定左侧边界
            right = mid - 1;
        }
    }
    // 最后要检查 left 越界的情况
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}

```

#### 右边界搜索

和左边界类似，每次更改边界时，尽量往右靠。

返回值需要 $-1$， 这是因为为了向右靠时，即使等于目标值，left 也进行 $+1$ 操作，所以，$left$ 和 $right$ 最终结果是指向目标的下一个值。

```java
int right_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;
    
    while (left < right) {
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            left = mid + 1; // 注意
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    return left - 1; // 注意
}

int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定右侧边界
            left = mid + 1;
        }
    }
    // 最后要检查 right 越界的情况
    if (right < 0 || nums[right] != target)
        return -1;
    return right;
}
```

## 其他问题

### k 值的问题

#### [668. 乘法表中第k小的数](https://leetcode.cn/problems/kth-smallest-number-in-multiplication-table/) 

题目中，需要求解乘法表中第 k 大的值。由于乘法表的数据范围过大，直接使用数组存储的化，会导致内存溢出，而计算一个数大于等于乘法表的数字的数量则相对简单，因此可以通过二分查找进行搜索。

乘法表中的某一行数据小于等于该数个个数为：$min(\frac{x}{i}, n)$ ，其中 $i$ 表示行数，且第 $i$ 行都是 $i$ 的倍数。

有由于当 $i < \frac{x}{n}$ 时，$\frac{x}{i} > n$ ，故可以得到如下计算公式

$$\lfloor\frac{x}{n}\rfloor*n + \sum_{\frac{x}{n} + 1}^{m}\lfloor\frac{x}{i}\rfloor$$ 

```java
class Solution {
    public int findKthNumber(int m, int n, int k) {
        int left = 1, right = m * n;
        while (left < right) {
            int x = left + (right - left) / 2;
            int count = x / n * n;
            for (int i = x / n + 1; i <= m; ++i) {
                count += x / i;
            }
            if (count >= k) {
                right = x;
            } else {
                left = x + 1;
            }
        }
        return left;
    }
}
```

#### [719. 找出第 K 小的数对距离](https://leetcode.cn/problems/find-k-th-smallest-pair-distance/)

```markdown
数对 (a,b) 由整数 a 和 b 组成，其数对距离定义为 a 和 b 的绝对差值。

给你一个整数数组 nums 和一个整数 k ，数对由 nums[i] 和 nums[j] 组成且满足 0 <= i < j < nums.length 。返回 所有数对距离中 第 k 小的数对距离。
```

和上题类似

```java
class Solution {
    public int smallestDistancePair(int[] nums, int k) {
        Arrays.sort(nums);
        int n = nums.length, left = 0, right = nums[n - 1] - nums[0];
        while (left <= right) {
            int mid = (left + right) / 2;
            int cnt = 0;
            for (int i = 0, j = 0; j < n; j++) {
                while (nums[j] - nums[i] > mid) {
                    i++;
                }
                cnt += j - i;
            }
            if (cnt >= k) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
}
```

### 可行性问题

[Cable master](http://poj.org/problem?id=1064) 

```markdown
有 N 条绳子，他们的长度分为 L_i。如果从中切割出 K 条长度相同的绳子的话，则 K 条绳子的每条最长能多长。
```

该问题看似与二分查找无关。但从最后的结果来看，若长度越长，则必然导致 K 条数量不够，因此可以通过二分查找来判断长度 $l$ 是否可以满足获取 K 条长度相同的绳子。

```java
public void solve(int[] lengths, int k){
    int max = lengths[0];
    for(int len: lengths){
        max = Math.max(max, len);
    }
    
    int left = 0;
    int right = max;
    while(left < right){
        int mid = (left + right) / 2;
        int count = 0;
        for(int len : lengths){
            count += len / mid;
        }
        if(count >= k){
            left = mid;
        }else{
            right= mid + 1;
        }
    }
    return left;
}
```

### [1760. 袋子里最少数目的球](https://leetcode.cn/problems/minimum-limit-of-balls-in-a-bag/)

```markdown
给你一个整数数组 nums ，其中 nums[i] 表示第 i 个袋子里球的数目。同时给你一个整数 maxOperations 。

你可以进行如下操作至多 maxOperations 次：

选择任意一个袋子，并将袋子里的球分到 2 个新的袋子中，每个袋子里都有 正整数 个球。
比方说，一个袋子里有 5 个球，你可以把它们分到两个新袋子里，分别有 1 个和 4 个球，或者分别有 2 个和 3 个球。
你的开销是单个袋子里球数目的 最大值 ，你想要 最小化 开销。

请你返回进行上述操作后的最小开销。
```

**该问题可以看作是可行性问题的推广**，该题相当于求 maxoPenrations 操作后可能的最小值，由于解的范围固定了，即$[1, max]$ ，且解越大，则需要的操作次数越少，必然导致操作减少，所以和上题一样，只使用二分判断解的可行性即可

```java
class Solution {
    public int minimumSize(int[] nums, int maxOperations) {
        int n = nums.length;
        int max = 0;
        for (int i = 0; i < n; i++) {
            max = Math.max(max, nums[i]);
        }

        int left = 1;
        int right = max;
        while (left < right){

            int mid = left + right >> 1;
            long cnt = 0;
            for (int i = 0; i < n; i++) {
                cnt += (nums[i] - 1) / mid;
            }
			
            // 判断解的可行性
            if(cnt > maxOperations){
                left = mid + 1;
            }else{
                right = mid;
            }
        }
        return left;
    }
}
```

## 总结

以上问题可以使用二分查找来解决问题，通常都是其解都是 ”单向“ 的，且解的范围初始就固定了。这个单向指的是解的结果影响的方向。比如对于 k 值问题，若解增大时，则必然导致小于等于该数的数量非严格递增。而对于可行性问题来说，若解的结果越大，则获取的绳子的数量必然是非严格递减。

## 参考资料

[668. 乘法表中第k小的数](https://leetcode.cn/problems/kth-smallest-number-in-multiplication-table/) 

[Cable master](http://poj.org/problem?id=1064) 

[二分查找详解](https://github.com/labuladong/fucking-algorithm/blob/master/%E7%AE%97%E6%B3%95%E6%80%9D%E7%BB%B4%E7%B3%BB%E5%88%97/%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE%E8%AF%A6%E8%A7%A3.md)

