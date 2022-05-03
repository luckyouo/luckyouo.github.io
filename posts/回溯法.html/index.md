# 回溯法


## 前言

回溯法是一种选优搜索法，按选优条件向前搜索，以达到目标，但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法，而满足回溯条件的某个状态的点称为“回溯点”。

回溯法简单来说就是按照深度优先的顺序，穷举所有可能性的算法，但是回溯算法比暴力穷举法更高明的地方就是回溯算法可以随时判断当前状态是否符合问题的条件，所以根据这类问题，我们有一些优化剪枝策略以及启发式搜索策略。

## 算法实现

算法基本步骤如下：

1. 确定终止条件。回溯法的本质就是递归，所以需要确定递归的终止条件，防止死循环。
2. 循环回溯。循环回溯是回溯法的主要部分，该部分进行迭代->回溯，以此实现穷举，在迭代过程中可以根据需求，对部分路径进行剪枝。

[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/) 代码实现

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        ArrayList<String> ans = new ArrayList<>();
        if(digits.length() == 0){
            return ans;
        }
        ArrayList<String> list = new ArrayList<>();
        list.add("abc");
        list.add("def");
        list.add("ghi");
        list.add("jkl");
        list.add("mno");
        list.add("pqrs");
        list.add("tuv");
        list.add("wxyz");

        int size = list.size();
        
        StringBuilder sb = new StringBuilder();
        backtrack(digits, list, ans, 0, sb);
        return ans;
    }

    private void backtrack(String digits, ArrayList<String> list, ArrayList<String> ans, int i, StringBuilder sb) {
        // 终止条件
        if (i == digits.length()) {
            ans.add(sb.toString());
            return;
        }else{
            int idx = Integer.parseInt(String.valueOf(digits.charAt(i)));
            String s = list.get(idx - 2);
            int len = s.length();
            // 循环迭代
            for (int j = 0; j < len; j++) {
                sb.append(s.charAt(j));
                // 迭代
                backtrack(digits, list, ans, i + 1, sb);
                // 回溯
                sb.deleteCharAt(sb.length() - 1);
            }
        }
    }
}
```

[39. 组合总和](https://leetcode-cn.com/problems/combination-sum/) 代码实现

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> ans = new ArrayList<>();
        int min = candidates[0];

        List<Integer> list = new ArrayList<>();
        backtrack(ans, list, candidates, target, 0);
        return ans;
    }

    public void backtrack(List<List<Integer>> ans, List<Integer> list, int[] candidates, int target, int idx){
        if(idx == candidates.length){
            return;
        }

        if(target == 0){
            ans.add(new ArrayList<>(list));
            return;
        }
		
        // 查找单元素集合
        backtrack(ans, list, candidates, target, idx + 1);
        // 查找多元素集合
        if(target - candidates[idx] >= 0){
            list.add(candidates[idx]);
            backtrack(ans, list, candidates, target - candidates[idx], idx);
            list.remove(list.size() - 1);
        }
    }
}
```

## 回溯法 + 去重

在某些算法的要求下，需要对结果进行去重，简单粗暴的方法就是通过 HashSet 去重，但是时间复杂度高，因为结果需要对比。也可以通过循环，对某些条件进行剪枝。

[剑指 Offer II 082. 含有重复元素集合的组合](https://leetcode-cn.com/problems/4sjJUc/) 回溯法 + 循环剪枝去重

```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> ans = new ArrayList<>();
        backtrack(candidates, ans, new ArrayList<Integer>(), target, 0);
        return ans;
    }

    public void backtrack(int[] candidates, List<List<Integer>> ans, List<Integer> list, int target, int idx) {
        if (target == 0) {
            ans.add(new ArrayList<>(list));
            return;
        }


        for (int i = idx; i < candidates.length; i++) {
            if (candidates[i] > target) {
                return;
            }

            if (i > idx && candidates[i] == candidates[i - 1]) {
                continue;
            }


            list.add(candidates[i]);
            backtrack(candidates, ans, list, target - candidates[i], i + 1);
            list.remove(list.size() - 1);
        }
    }
}
```

## 参考资料

[五大基本算法之回溯算法 backtracking](https://houbb.github.io/2020/01/23/data-struct-learn-07-base-backtracking)

[小白带你学--回溯算法](https://www.jianshu.com/p/dd3c3f3e84c0)

[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

[39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

[剑指 Offer II 082. 含有重复元素集合的组合](https://leetcode-cn.com/problems/4sjJUc/) 

