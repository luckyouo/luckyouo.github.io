# lambda 表达式


## 前言

**java 8** 增加的一个重大特性，就是 `lambda` 表达式了。`lambda` 作为 java 的语法糖，提供了更加简洁的语法书写，减少了使用匿名类带来的代码冗余性。 

在 **java 8** 之前，如果需要使用一个功能函数接口，必须通过类来实现，因为，在 java 当中，方法是依托于类存在的，这也导致 java 不能实现函数式编程。而在 **java 8** ，之后，引入了函数式接口、lambda 表达式和方法引用，在一定程度上，将函数上升为 “一等公民”。

## 基本使用

java 的 `lambda` 表达式必须结合函数式接口来使用。函数式接口是指该接口只有一个抽象方法（可以有多个方法，但必须有一个为实现，因为 java 8 接口中的方法可以使用默认实现），且该接口最好使用 `@FunctionalInterface` 注解标注。

lambda 表达式的基本书写形式如下：

```java
(T param1, T param2) -> {operator(param1, param2)}
```

`lambda` 表达式使用样例

```java
interface Description {
  String brief();
}

interface Body {
  String detailed(String head);
}

interface Multi {
  String twoArg(String head, Double d);
}

public class LambdaExpressions {

  static Body bod = h -> h + " No Parens!"; // [1]

  static Body bod2 = (h) -> h + " More details"; // [2]

  static Description desc = () -> "Short info"; // [3]

  static Multi mult = (h, n) -> h + n; // [4]

  static Description moreLines = () -> { // [5]
    System.out.println("moreLines()");
    return "from moreLines()";
  };

  public static void main(String[] args) {
    System.out.println(bod.detailed("Oh!"));
    System.out.println(bod2.detailed("Hi!"));
    System.out.println(desc.brief());
    System.out.println(mult.twoArg("Pi! ", 3.14159));
    System.out.println(moreLines.brief());
  }
}
```

**[937. 重新排列日志文件](https://leetcode-cn.com/problems/reorder-data-in-log-files/)** 解题代码使用 lambda 表达式

```java
class Solution {
    public String[] reorderLogFiles(String[] logs) {
        Arrays.sort(logs, (log1, log2)->{
			String[] split1 = log1.split(" ",2);
			String[] split2 = log2.split(" ",2);
			boolean isDigit1 = Character.isDigit(split1[1].charAt(0));
			boolean isDigit2 = Character.isDigit(split2[1].charAt(0));
			if (!isDigit1 && !isDigit2){
				int cmp = split1[1].compareTo(split2[1]);
				if (cmp !=0) return cmp;
				return split1[0].compareTo(split2[0]);
			}
			return isDigit1 ? (isDigit2 ? 0:1) : -1;
		});
		return logs;
    }
}
```

函数调用的 lambda 表达式

```java
class Solution {
    public String[] reorderLogFiles(String[] logs) {
        int length = logs.length;
        Pair[] arr = new Pair[length];
        for (int i = 0; i < length; i++) {
            arr[i] = new Pair(logs[i], i);
        }
        Arrays.sort(arr, (a, b) -> logCompare(a, b));
        String[] reordered = new String[length];
        for (int i = 0; i < length; i++) {
            reordered[i] = arr[i].log;
        }
        return reordered;
    }

    public int logCompare(Pair pair1, Pair pair2) {
        String log1 = pair1.log, log2 = pair2.log;
        int index1 = pair1.index, index2 = pair2.index;
        String[] split1 = log1.split(" ", 2);
        String[] split2 = log2.split(" ", 2);
        boolean isDigit1 = Character.isDigit(split1[1].charAt(0));
        boolean isDigit2 = Character.isDigit(split2[1].charAt(0));
        if (isDigit1 && isDigit2) {
            return index1 - index2;
        }
        if (!isDigit1 && !isDigit2) {
            int sc = split1[1].compareTo(split2[1]);
            if (sc != 0) {
                return sc;
            }
            return split1[0].compareTo(split2[0]);
        }
        return isDigit1 ? 1 : -1;
    }
}

class Pair {
    String log;
    int index;

    public Pair(String log, int index) {
        this.log = log;
        this.index = index;
    }
}
```

## 注意事项

1. 如果编译器可以根据接口类型推断出参数类型，则 `lambda` 表达式可以省略参数的类型。所以 `lambda` 表达式是运行时才确认的，故 lambda 表达式使用的数据必须是不可以变更的（`final` 类型），即是闭包的。

2. `lambda` 表达式如果只有一个参数时，可以省略参数的括号（不建议）。

3. 在 `lambda` 表达式中使用的关键字 `this` ，表示创建 `lambda` 表达式方法的 `this` 参数。
4.  `lambda` 表达式内如果存在分支返回值时，则另外的分支必须也存在返回值，否则不合法。
5. 尽量将 `lambda` 表达式看作为一个函数，而不是一个对象。
6. **如果只有一条语句， 则大括号可以省略，且不需要 return 关键字，编译器会自动返回表达式值。如果使用大括号，则必须使用 return 返回表达式的值。返回值也函数式接口返回类型相同** 
7. `lambda` 表达式后半段可以使用函数调用代替大括号内容

## 参考资料

[高阶函数和Java的Lambda](https://www.jianshu.com/p/7e831f29ee6e) 

[Java中的函数式编程（三）lambda表达式](https://juejin.cn/post/7021531239072923678)

[第十三章 函数式编程](https://wizardforcel.gitbooks.io/onjava8/content/book/13-Functional-Programming.html#Currying%E5%92%8CPartial-Evaluation) 

[937. 重新排列日志文件](https://leetcode-cn.com/problems/reorder-data-in-log-files/) 

