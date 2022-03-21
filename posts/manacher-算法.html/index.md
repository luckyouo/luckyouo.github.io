# Manacher 算法


<!--more-->

## 前言

检查字符串是否为回文序列，常用方法有 `中心扩展` 和 `Manacher` 。前者的时间复杂度为 `O(n^2)` ，后者算法的时间复杂度为 `O(n)` ，可以在线性时间的完成回文序列的判断

[剑指 Offer II 020. 回文子字符串的个数](https://leetcode-cn.com/problems/a7VOhD/)

## 算法思想

`Manacher` 算法是利用保存前字符串序列的最大回文半径来减少后续字符串回文与否的判定。算法过程如下：

- 先将初始字符串在每个字符间增加一个间隔符号，比如 `#` （也包括在字符串的开头和字符串的结尾），该步骤主要是为了将偶数回文序列和奇数回文序列合并在一起进行判断。同时在字符串的开头分别加上两个不同的特殊符号，比如开头增加 `$` ，结尾增加 `^` ，这两个字符充当哨兵作用

  ```markdown
  "abab" ---> "$#a#b#a#b$^"
  ```

- 创建上述字符串的半径数组，从左往右依次计算各序号的半径，同时记录当前的字符串回文的最大右边界 `rmax` 和对于的序号 `maxi`，计算过程如下：

  - 当该序号 `i` 小于等于保存的最大的右边界 `rmax` 时，则将当前序号的半径初始化为 `r = max(rmax - i + 1, 2 * maxi - i)` 
  - 反之，则直接初始化为 `1`

- 再使用 `中心扩展法` 判断当前半径是否为最大回文序列半径，若不是，则递增继续判断（在上一步骤中，若得到的`r + i` 小于最大右边界时，则 `r`即为当前序号的最大半径；若等于最大右边界时，则需要该步骤进行判断）

从上述步骤看，主要思路是通过记录当前序号的最大半径 `r` 和最大右边界 `rmax`，减少后续序号最大半径的搜索范围。（`中心扩展法` 默认都是从半径为 `1` 依次进行判断）。该算法思路与字符串KMP算法思想类似

## 代码

```java
class Solution {
    public int countSubstrings(String s) {
        int res = 0;
        StringBuilder sb = new StringBuilder();
        sb.append("$#");
        for(int i = 0; i < s.length(); i++){
            sb.append(s.charAt(i));
            sb.append("#");
        }
        sb.append('^');
        int[] r = new int[sb.length()];

        r[0] = 1;
        int max = r[0];
        int maxi = 0;
        for(int i = 1; i < r.length - 1; i++){
            r[i] = i < max ? Math.min(max - i + 1, r[2 * maxi - i]) : 1;
            while(sb.charAt(i - r[i]) == sb.charAt(i + r[i])){
                r[i]++;              
            }
            if(i + r[i] - 1 >= max){
                max = i + r[i] - 1;
                maxi = i;
            }
            res +=  r[i]  / 2;
        }       
        return res;
    }
}
```

## 参考链接

[Manacher 算法](https://segmentfault.com/a/1190000008484167)

[回文子字符串的个数](https://leetcode-cn.com/problems/a7VOhD/solution/hui-wen-zi-zi-fu-chuan-de-ge-shu-by-leet-ejfv/)


