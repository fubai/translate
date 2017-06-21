# Java中的字符串池

## Java中的字符串池也被称作：

* 字符串字面量池
* 字符串常量池
* 字符串intern池(字符串保留池？字符串拘留池？)

## Java里的字符串池是什么？它的好处是什么？

当不是用`new`创建字符串，且字符串池里已有该字符串时，那么不会新建一个字符串，而是直接用池里这个已经存在的字符串。

## 字符串池在内存哪里？

从Java 7开始，字符串池被放在了java堆内存里。

而在Java 7之前，字符串池是在方法区中。(方法区是JVM内存模型中的一块区域，这块区域在java7之前就是gc中的永久代)
```
    String s1 = "abc"; 
    String s3 = "abc";
```
上面两行代码执行后，只有一个字符串被创建到字符串池中。

`String s1 = "abc";`被执行时，池里还没有`"abc"`，所以这个字符串会被创建到字符串池里，s1是一个变量引用，指向它。

`String s3 = "abc";`被执行时，池里已有了`"abc"`，所以s3也会指向字符串池里的`"abc"`。

### JDK文档上的字符串池

在JDK 7里，被保留的(interned)字符串不再被分配到方法区中，而是和应用创建的其它对象一样，被分配到Java堆里。这个改变导致更多的数据被存到了堆里，也许这要调整堆的大小了。这个改变对大多数应用来说，在堆的用法上只会有相对较小的不同。但在加载很多类或重度使用String.intern()的大应用中，会有更显著的差异。

## 字符串池的示例图

![Diagram to demonstrate String pool in java](java_string_pool.png)

## 让我们一步步讨论下面5个语句执行时将会发生什么
```
    String s1 = "abc";
    String s2 = new String("abc");
    String s3 = "abc";
    String s4 = new String("abc");
    String s5 = new String("abc").intern();
```

### String s1 = "abc";

这时候字符串池里没有`"abc"`，所以JVM会在字符串池里创建它，s1是一个变量引用，指向它。

### String s2 = new String("abc");

这里用new创建字符串，new会强制JVM在堆里创建新的字符串，不在字符串池里。

### String s3 = "abc";

字符串池里已经有了`"abc"`，所以s3会直接引用池里的`"abc"`。

### String s4 = new String("abc");

这里用new创建字符串，new会强制JVM在堆里创建新的字符串，不在字符串池里。

### String s5 = new String("abc").intern();

这里用new创建字符串，但是调用了这个字符串对象的intern方法，所以s5也会指向字符串池里的`"abc"`。

## Java里的intern方法

该方法在java.lang.String类中，当在字符串对象a上调用intern时，如果字符串池里已经有了一个字符串b，通过[Object的equals方法](http://www.javamadesoeasy.com/2015/05/difference-between-equals-method-and.html)，能判断a和b相等，那么intern方法会返回b。否则，会添加一个字符串到字符串池里，并返回这个对象的引用。

### 例子

```
    String s1 = "abc";
    String s5 = new String("abc").intern();
```

上面第一句会在字符串池中创建`"abc"`。

第二句，当调用intern方法时，如果字符串池中已经有一个相等的字符串，就返回该字符串，否则，在字符串池中创建一个新的。在第一句执行后，字符串池中已经有了`"abc"`，所以s5会直接指向它。

所以，`s1 == s5`会返回`true`。

## 为什么Java里有字符串池

Java代码里广泛的使用字符串，由于字符串是不可变的，所以可以把字符串缓存在内存中，这样可以节省内存，提升性能。越少的在java堆里创建字符串，留给垃圾回收器的工作就越少。

## 在Java中，下面的操作会产生多少个字符串？

```
    String str = "abc";
    str = str + "def";
```

### 答案

`String str = "abc";`，这个语句执行时，JVM会在字符串池中创建一个字符串（池里的第一个字符串）。

现在，棘手部分来了：`str = str + "def";`。`+`在内部会使用StringBuffer串联多个字符串。

所以，内部的实际操作是：
```
String str = new StringBuilder(str).append("def").toString();
```

_"def"会是字符串池里的第二个字符串_

最后：`str = "abcdef"`

_"abcdef"会是字符串池里的第三个字符串_

[原文链接](http://www.javamadesoeasy.com/2015/05/string-pool-string-literal-pool-string.html)
