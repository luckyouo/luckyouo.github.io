# 双指针


## 前言

双指针主要用来在序列中进行遍历，其中常用方式有 `快慢指针` 、 `对撞指针` 和 `滑动窗口` 。可以从将暴力求解的时间复杂度 `O(n^2)` 降低至 `O(n)` 

## 算法思想

### 快慢指针

`快慢指针` 是指两个指针的速度移动不一致。比如一个循环内一个指针移动一位，另一个移动两位，这样在循环结束时，两个指针的移动的距离之比则是固定倍数。

`快慢指针` 一般用来求序列的中间位置、序列是否成环（进一步也可也求成环的初始位置），例题参考如下

[剑指 Offer II 022. 链表中环的入口节点](https://leetcode-cn.com/problems/c32eOV/)

[剑指 Offer II 027. 回文链表](https://leetcode-cn.com/problems/aMhZSa/)

### 对撞指针

`对撞指针` 是指两个指针初始在序列的不同端，并且不断的向中心进行寻找

`对撞指针` 一般用来求解序列两数之 ”和“（广义上指两元素之间关系） 是否满足某一条件，通常该序列一般经过特殊处理过，比如升序排列，也可以对字符串进行翻转。例题参考如下

[344. 反转字符串](https://leetcode-cn.com/problems/reverse-string/)

[ 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/) 体积问题转换为左右边界问题，而左右边界问题可以通过对撞指针在时间复杂度为 `O(n)` 完成。

### 滑动窗口

`滑动窗口` 两个指针分别为窗口的左端和右端，并按一定的条件进行有移动。

`滑动窗口` 按照题目的要求，可以使用固定宽度和动态宽度。该方法一般用来求解`连续`序列问题

## 代码

`快慢指针` 代码参考如下

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        ListNode p = head;

        while(fast != null){
            fast = fast.next;
            if(fast == null){
                return null;
            }
            // System.out.println(fast.val);
            fast = fast.next;
            if(fast == null){
                return null;
            }
            slow = slow.next;
            if(fast == slow){
                break;
            }
        }

        while(p != slow){
            p = p.next;
            slow = slow.next;
        }
        return p;      
    }
}
```

对撞指针代码参考如下：

```java
class Solution {
    public void reverseString(char[] s) {
        int i=0;
        int j=s.length-1;
        while(i<j){
            char tmp = s[i];
            s[i] = s[j];
            s[j] =  tmp;
            i++;
            j--; 
        }

    }
}
```

滑动窗口参考代码如下：

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int n = nums.length;
        if (n == 0) {
            return 0;
        }
        int ans = Integer.MAX_VALUE;
        int start = 0, end = 0;
        int sum = 0;
        while (end < n) {
            sum += nums[end];
            while (sum >= s) {
                ans = Math.min(ans, end - start + 1);
                sum -= nums[start];
                start++;
            }
            end++;
        }
        return ans == Integer.MAX_VALUE ? 0 : ans;
    }
}
```

## 参考资料

[和大于等于 target 的最短子数组](https://leetcode-cn.com/problems/2VG8Kg/solution/he-da-yu-deng-yu-target-de-zui-duan-zi-s-ixef/)

[算法一招鲜——双指针问题](https://zhuanlan.zhihu.com/p/71643340)


