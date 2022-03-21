# Java 基础


<!--more-->

## 前言

主要总结一些 `java` 的基本使用

持续总结中 `ing`

## 内容

### 字符串拼接

字符串的拼接优先使用StringBuilder和StringBuffer类，同时在使用拼接的过程中，尽量使用一个一个的字符串进行 `append` ，而不是将某些字符串拼接后再 `append` ，例如

```java
int n = 100;
StringBuilder sb = new StringBuilder();
for(int i = 0; i < n; i++){
    # sb.append(i + "#");
    sb.append(i);
    sb.append("#");
}
```

不要将 `i + "#"` 拼接好后在 `append` ，字符串拼接后会产生的新的字符串从而产生更多的开销




