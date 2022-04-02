# 博弈论


## 前言

博弈论考虑游戏中的个体的预测行为和实际行为，并研究它们的优化策略。表面上不同的相互作用可能表现出相似的激励结构（incentive structure），所以它们是同一个游戏的特例。 

## 博弈论基本解法

由于博弈论都是在考虑最优解的情况下进行的步骤，所以只需要知道在各个步骤的的必败必胜必和情况，并以此进行规划就可以知道最后的胜负状况。其中最重要的是，是确定必胜/必败/必和的条件。

### 案例一

[异或数列](http://lx.lanqiao.cn/problem.page?gpid=T2897) ： 

**Alice 和 Bob 正在玩一个异或数列的游戏。初始时，Alice 和 Bob 分别有一个整数 a 和 b，有一个给定的长度为 n 的公共数列 $X_1, X_2, · · · , X_n$Alice 和 Bob 轮流操作，Alice 先手，每步可以在以下两种选项中选一种：**

- **选项 1：从数列中选一个 Xi 给 Alice 的数异或上，或者说令 a 变为 a ⊕ $X_i$。（其中 ⊕ 表示按位异或）**

- **选项 2：从数列中选一个 Xi 给 Bob 的数异或上，或者说令 b 变为 b ⊕ $X_i$。**

**每个数 Xi 都只能用一次，当所有 $X_i$ 均被使用后（n 轮后）游戏结束。游戏结束时，拥有的数比较大的一方获胜，如果双方数值相同，即为平手。现在双方都足够聪明，都采用最优策略，请问谁能获胜。（$n < 2^{20}$）**

首先确认游戏的必胜/必败/必和情况。其中必和情况为：`A=B` ，所以 `A^B = 0` ，所以只需要对各个数字异或判断是否为 `0` 即可。剩下比较异或结果的数字大小，只需要看最高位即可，故只需要统计一下各个位 `1` 的个数。由题目可知，各个数字都可以由 20 位表示。

- 必和：$X_0$  ⊕  $X_i$ ⊕ $X_n$ = 0 	i = 1, 2, ..., n -1

- 必胜/必败：最高位，`1` 的个数位偶数，则下一位判断。奇数时
  - `1` 的个数为 `1` 时，则先手获胜
  - 大于 `1` 时，总数字个数位偶数时，先手必败
  - 大于 `1` 时，总数字个数位奇数时，先手必胜

```java
import java.util.Scanner;

public class Main {
	public static void main(String[] args) {
		Scanner in = new Scanner(System.in);
		int T = in.nextInt();
		int[] ans = new int[T];
		long sum = 1;
		for(int i = 0; i < T; i++) {
			int n = in.nextInt();
			int[] count = new int[20];
			sum = 0;
			for(int j = 0; j < n; j++) {
				long num = in.nextLong();
				int temp = 1;
				int k = 0;
				sum = sum ^ num;
				for(; k < 20; k++) {
					if((num & temp) == temp) {
						count[k]++;
						
					}
					temp <<= 1;
				}			
			}
			if(sum == 0) {
				ans[i] = 0;
//				System.out.println(0);
				continue;
			}
			int k = 19;
			for(; k >= 0; k--) {
				if(count[k] == 1) {
					ans[i] = 1;
//					System.out.println(1);
					break;
				}else if(count[k] % 2 == 1 && n % 2 == 0) {
					ans[i] = -1;
//					System.out.println(-1);
					break;
				}else if(count[k] % 2 == 1 && n % 2 == 1) {
					ans[i] = 1;
//					System.out.println(1);
					break;
				}
			}
			
		}
		
		for(int i = 0; i < T; i++) {
			System.out.println(ans[i]);
		}
		
	}
}
```



### 案例二

著名的 [Nim游戏](https://zh.wikipedia.org/wiki/%E5%B0%BC%E5%A7%86%E6%B8%B8%E6%88%8F) ：

**有 n 堆石子，每堆各有 $a_i$  颗石子。Alice 和 Bob 轮流从非空的石子堆取走至少一颗石子。Alice 先取，取光所有石子的一方获胜。当双方采取最优策略时，谁会获胜？**

同样，首先确认该游戏必胜/必败/必和情况。有如下结论成立

$$a_1 XOR a_2 XOR ... XOR  a_n != 0   ---> 必胜态 $$

$$a_1 XOR a_2 XOR ... XOR  a_n == 0    ---> 必败态 $$

其中 XOR 表示异或。



## 参考

[博弈论](https://zh.wikipedia.org/wiki/%E5%8D%9A%E5%BC%88%E8%AE%BA)

[挑战程序设计竞赛](https://book.douban.com/subject/24749842/)

[蓝桥杯2021年第十二届省赛真题-异或数列](https://blog.csdn.net/m0_49924838/article/details/122796494)

