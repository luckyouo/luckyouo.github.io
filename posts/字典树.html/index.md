# 字典树


## 前言

`字典树` ，是一种**空间换时间**的数据结构，又称 `Trie` 树、`前缀树` ，是一种树形结构(字典树是一种数据结构)，用于统计、排序、和保存大量字符串

![trie](/images/trie/trie1.png)

## 算法思想

按照题目所给的 `字典序` 要求，生成相应的 `字典树` 结构。后续查找通过公共前缀来减少查找时间，降低查找时间复杂度

`字典树` 的三个重要性质：

- 根节点不包含字符，除了根节点每个节点都只包含一个字符。root节点不含字符这样做的目的是为了能够包括所有字符串。
- 从根节点到某一个节点，路过字符串起来就是该节点对应的字符串。（也可以直接在该节点保存字符连接后的结果）
- 每个节点的子节点字符不同，也就是找到对应单词、字符是唯一的。

## 在优化

`字典树` 的空间复杂度为 `O(N)` ，可以根据 `前序` 结果推导出 `后序` 结果，可以不创建 `字典树` ，直接使用 `前序` 进行迭代，相应的空间复杂度可降低至 `O(1)` 。例如 [440. 字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/) 

## 代码

`字典树` 的实现

```java
public class TrieNode {
	int count;
	int prefix;
	TrieNode[] nextNode=new TrieNode[26];
	public TrieNode(){
		count=0;
		prefix=0;
	}
}

//插入一个新单词
public static void insert(TrieNode root,String str){
		if(root==null||str.length()==0){
			return;
		}
		char[] c=str.toCharArray();
		for(int i=0;i<str.length();i++){
			//如果该分支不存在，创建一个新节点
			if(root.nextNode[c[i]-'a']==null){
				root.nextNode[c[i]-'a']=new TrieNode();
			}
			root=root.nextNode[c[i]-'a'];
			root.prefix++;//注意，应该加在后面
			}
		
		//以该节点结尾的单词数+1
		root.count++;
}

 //查找该单词是否存在，如果存在返回数量，不存在返回-1
public static int search(TrieNode root,String str){
	if(root==null||str.length()==0){
		return -1;
	}
    char[] c=str.toCharArray();
    for(int i=0;i<str.length();i++){
        //如果该分支不存在，表名该单词不存在
        if(root.nextNode[c[i]-'a']==null){
            return -1;
        }
        //如果存在，则继续向下遍历
        root=root.nextNode[c[i]-'a'];	
    }
		
    //如果count==0,也说明该单词不存在
    if(root.count==0){
        return -1;
    }
    return root.count;
}

//查询以str为前缀的单词数量
public static int searchPrefix(TrieNode root,String str){
    if(root==null||str.length()==0){
        return -1;
    }
    char[] c=str.toCharArray();
    for(int i=0;i<str.length();i++){
        //如果该分支不存在，表名该单词不存在
        if(root.nextNode[c[i]-'a']==null){
            return -1;
        }
        //如果存在，则继续向下遍历
        root=root.nextNode[c[i]-'a'];	
    }
    return root.prefix;
}

```

 [440. 字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/)  代码实现

```java
class Solution {
    public int findKthNumber(int n, int k) {
        int pos = 1;
        k--;
        int step;
        while(k > 0){
            step = getStep(pos, n);
            if(step > k){
                pos *= 10;
                k--;           
            }else{
                k -= step;
                pos++;
            }
        }
        return pos;
        
    }

    public int getStep(int pos, int n){
        long first = pos;
        long last = pos;
        int step = 0;
        while(first <= n){
            step += Math.min(last, n) - first + 1;
            first = first * 10;
            last = last * 10 + 9;
        }

        return step;
    }
}
```



## 参考资料

[字典树 (Trie)](https://oi-wiki.org/string/trie/)

[数据结构与算法：字典树（前缀树）](https://zhuanlan.zhihu.com/p/28891541)

[一文搞懂字典树](https://segmentfault.com/a/1190000040801084)


