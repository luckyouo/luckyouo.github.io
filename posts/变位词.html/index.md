# 变位词


## 前言

`变位词` 是指字符串更改字符顺序后相等的字符串（即字符串内各个字符出现的次数相等）。`变位词` 匹配就是将字符串数组中所有相同的变位词找出。

[剑指 Offer II 033. 变位词组](https://leetcode-cn.com/problems/sfvd7V/)

## 算法思想

- `暴力求解` ：将两个字符串的各个字符查询统计并比较，这种算法的时间复杂度为 `O(m*n)` ，且难适用于多个字符串同时进行匹配分组
- `计数拼接` ： 将各个字符统计并计数，最后将各个字符与对应的次数进行拼接起来，比如 `leetcode`，计算得到的字符为 `c1d1e3l1o1t1`，接着将结果作为哈希 `Key` ， 该字符的索引作为 `Value` ，使用哈希匹配分组
- `素数求积` ： 将各个字符映射为一个素数，同时求得字符串的累积，并将结果作为哈希的 `Key` ，字符串的索引作为 `Value` ，使用哈希匹配分组。该思路需要考虑 `大数` 问题，若基本数据会溢出的话，可以使用 `mod` 求余，但这也需要考虑哈希冲突

## 代码

1. 计数拼接代码，参考自 [字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            int[] counts = new int[26];
            int length = str.length();
            for (int i = 0; i < length; i++) {
                counts[str.charAt(i) - 'a']++;
            }
            // 将每个出现次数大于 0 的字母和出现次数按顺序拼接成字符串，作为哈希表的键
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < 26; i++) {
                if (counts[i] != 0) {
                    sb.append((char) ('a' + i));
                    sb.append(counts[i]);
                }
            }
            String key = sb.toString();
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

2. 素数求积。根据题目所给要求，判断不会溢出，故没有取余

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        List<List<String>> res = new ArrayList<>();
        Map<Long, Integer> map = new HashMap<>();

        int[] primeNum = {2,3,5,7,11,13,17,19,23,29,31,37,41,43,47,
                            53,59,61,67,71,73,79,83,89,97,101};
        long product = 1;
        String temp;
        int index;
        for(int i = 0; i < strs.length; i++){
            temp = strs[i];
            product = 1;
            for(int j = 0; j < temp.length(); j++){
                product *= primeNum[temp.charAt(j) - 'a'];      
            }
            if(map.containsKey(product)){
                index = map.get(product);
                res.get(index).add(temp);
            }else{
                List<String> list = new ArrayList<>();
                list.add(temp);
                res.add(list);
                map.put(product, res.size()-1);
            }
        }
        return res;
    }
}
```



## 参考资料

[变位词问题](https://songlee24.github.io/2014/04/08/brother-word/)

