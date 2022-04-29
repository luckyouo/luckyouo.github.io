# 自动机


前言

**确定有限状态自动机**或**确定有限自动机**（英语：deterministic finite automaton, DFA）是一个能实现状态转移的[自动机](https://zh.wikipedia.org/wiki/自动机)。对于一个给定的属于该自动机的状态和一个属于该自动机字母表的字符，它都能根据事先给定的转移函数转移到下一个状态（这个状态可以是先前那个状态)。

需要注意的是，自动机只是一个 **数学模型**，而 **不是算法**，也 **不是数据结构**，自动机的结构就是一张有向图。实现同一个自动机的方法有很多种，可能会有不一样的时空复杂度。

常用的自动机有 **字典树**，**KMP 自动机**，**后缀自动机**，**回文自动机** 和 **序列自动机** 等等

## 序列自动机

[字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/) 题目中要求将字符串序列转换为整数，但由于边界条件多，如果使用 `if-else` 完成的话，则代码过于复杂，因此可以通自动机实现

```markdown
请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。

函数 myAtoi(string s) 的算法如下：

读入字符串并丢弃无用的前导空格
检查下一个字符（假设还未到字符末尾）为正还是负号，读取该字符（如果有）。 确定最终结果是负数还是正数。 如果两者都不存在，则假定结果为正。
读入下一个字符，直到到达下一个非数字字符或到达输入的结尾。字符串的其余部分将被忽略。
将前面步骤读入的这些数字转换为整数（即，"123" -> 123， "0032" -> 32）。如果没有读入数字，则整数为 0 。必要时更改符号（从步骤 2 开始）。
如果整数数超过 32 位有符号整数范围 [−2_31,  2^31 − 1] ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 −2_31 的整数应该被固定为 −2_31 ，大于 2^31 − 1 的整数应该被固定为 2^31 − 1 。
返回整数作为最终结果。
注意：

本题中的空白字符只包括空格字符 ' ' 。
除前导空格或数字后的其余字符串外，请勿忽略 任何其他字符

```

题目状态图如下：

![image-20220429102733473](/images/dfm.png)

### 代码

```java
class Solution {
    public int myAtoi(String str) {
        Automaton automaton = new Automaton();
        int length = str.length();
        for (int i = 0; i < length; ++i) {
            automaton.get(str.charAt(i));
        }
        return (int) (automaton.sign * automaton.ans);
    }
}

class Automaton {
    public int sign = 1;
    public long ans = 0;
    private String state = "start";
    private Map<String, String[]> table = new HashMap<String, String[]>() {{
        put("start", new String[]{"start", "signed", "in_number", "end"});
        put("signed", new String[]{"end", "end", "in_number", "end"});
        put("in_number", new String[]{"end", "end", "in_number", "end"});
        put("end", new String[]{"end", "end", "end", "end"});
    }};

    public void get(char c) {
        state = table.get(state)[get_col(c)];
        if ("in_number".equals(state)) {
            ans = ans * 10 + c - '0';
            ans = sign == 1 ? Math.min(ans, (long) Integer.MAX_VALUE) : Math.min(ans, -(long) Integer.MIN_VALUE);
        } else if ("signed".equals(state)) {
            sign = c == '+' ? 1 : -1;
        }
    }

    private int get_col(char c) {
        if (c == ' ') {
            return 0;
        }
        if (c == '+' || c == '-') {
            return 1;
        }
        if (Character.isDigit(c)) {
            return 2;
        }
        return 3;
    }
}

```

## 参考资料

[自动机](https://oi-wiki.org/string/automaton/)

[字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/solution/zi-fu-chuan-zhuan-huan-zheng-shu-atoi-by-leetcode-/)

[Basics of Automata Theory](https://cs.stanford.edu/people/eroberts/courses/soco/projects/2004-05/automata-theory/basics.html)

