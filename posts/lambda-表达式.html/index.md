# lambda 表达式


## 前言

**java 8** 增加的一个重大特性，就是 `lambda` 表达式了。`lambda` 作为 java 的语法糖，提供了更加简洁的语法书写，减少了使用匿名类带来的代码冗余性。 

在 **java 8** 之前，如果需要使用一个功能函数接口，必须通过类来实现，因为，在 java 当中，方法是依托于类存在的，这也导致 java 不能实现函数式编程。而在 **java 8** ，之后，引入了函数式接口、lambda 表达式和方法引用，在一定程度上，将函数上升为 “一等公民”。

## 基本使用

java 的 `lambda` 表达式必须结合函数式接口来使用。函数式接口是指该接口只有一个抽象方法（可以有多个方法，但必须有一个为实现，因为 java 8 接口中的方法可以使用默认实现），且该接口最好使用 `@FunctionalInterface` 注解标注。

lambda 表达式的基本书写形式如下：

```java
(T param1, T param2) -> operator(param1, param2)
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

## 注意事项

1. 如果编译器可以根据接口类型推断出参数类型，则 `lambda` 表达式可以省略参数的类型。所以 `lambda` 表达式是运行时才确认的，故 lambda 表达式使用的数据必须是不可以变更的（`final` 类型），即是闭包的。

2. `lambda` 表达式如果只有一个参数时，可以省略参数的括号（不建议）。

3. 在 `lambda` 表达式中使用的关键字 `this` ，表示创建 `lambda` 表达式方法的 `this` 参数。
4.  `lambda` 表达式内如果存在分支返回值时，则另外的分支必须也存在返回值，否则不合法。
5. 尽量将 `lambda` 表达式看作为一个函数，而不是一个对象。

## 参考资料

[高阶函数和Java的Lambda](https://www.jianshu.com/p/7e831f29ee6e) 

[Java中的函数式编程（三）lambda表达式](https://juejin.cn/post/7021531239072923678)

[第十三章 函数式编程](https://wizardforcel.gitbooks.io/onjava8/content/book/13-Functional-Programming.html#Currying%E5%92%8CPartial-Evaluation) 

