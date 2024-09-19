
# Java SE 23 新增特性


作者：[Grey](https://github.com)


原文地址：


[博客园：Java SE 23 新增特性](https://github.com)


[CSDN：Java SE 23 新增特性](https://github.com)


## 源码


源仓库: [Github：java\_new\_features](https://github.com)


## Primitive Types in Patterns, instanceof, and switch (预览功能)


通过 instanceof 和 switch，我们可以检查对象是否属于特定类型，如果是，则将该对象绑定到该类型的变量，执行特定的程序路径，并在该程序路径中使用新变量。



```
public class PrimitiveTypesTest {
    void main() {
        test1("hello world");
        test2("hello world");
        test1(56);
        test2(56);
        test1(java.time.LocalDate.now());
        test2(java.time.LocalDate.now());
    }

    private static void test1(Object obj) {
        if (obj instanceof String s && s.length() >= 5) {
            System.out.println(s.toUpperCase());
        } else if (obj instanceof Integer i) {
            System.out.println(i * i);
        } else {
            System.out.println(obj);
        }
    }

    private static void test2(Object obj) {
        switch (obj) {
            case String s when s.length() >= 5 -> System.out.println(s.toUpperCase());
            case Integer i -> System.out.println(i * i);
            case null, default -> System.out.println(obj);
        }
    }
}

```

[JEP 455](https://github.com) 在 Java 23 中引入了两项变更：


* 可以在 switch 表达式和语句中使用所有基元类型，包括 long、float、double 和 boolean。
* 其次，我们还可以在模式匹配中使用所有基元类型，包括 instanceof 和 switch。


在这两种情况下，即通过 long、float、double 和布尔类型进行 switch 以及使用基元变量进行模式匹配时，与所有新的 switch 功能一样，switch 必须要涵盖所有可能的情况。



```
private static void test3(int x) {
    switch (x) {
        case 1, 2, 3 -> System.out.println("Low");
        case 4, 5, 6 -> System.out.println("Medium");
        case 7, 8, 9 -> System.out.println("High");
    }
}

```

## Module Import Declarations (模块导入声明，预览功能)


通过简洁地导入模块导出的所有包的功能来增强 Java 编程语言。这简化了模块库的重复使用，但不要求导入代码本身必须在模块中。这是一项预览语言功能。


自 Java 1\.0 起，`java.lang` 包中的所有类都会自动导入到每个 .java 文件中。这就是为什么我们无需导入语句就能使用 `Object`、`String`、`Integer`、`Exception`、`Thread` 等类的原因。


我们还可以导入完整的包。例如，导入 `java.util.*` 意味着我们不必单独导入 `List`、`Set`、`Map`、`ArrayList`、`HashSet` 和 `HashMap` 等类。


[JEP 467](https://github.com)现在允许我们导入完整的模块，更准确地说，是导入模块导出的包中的所有类。


例如，我们可以按如下方式导入完整的 `java.base` 模块，然后使用该模块中的类（例如 `List`、`Map`、`Collectors`、`Stream`），而无需进一步导入：



```
package git.snippets.jdk23;

import module java.base;

public class ModuleImportDeclarationsTest {
    void main() {
        System.out.println(groupByFirstLetter("a", "abc", "bcd", "ddd", "dddc", "dfc", "bc"));
    }

    public static Map<Character, List<String>> groupByFirstLetter(String... values) {
        return Stream.of(values).collect(Collectors.groupingBy(s -> Character.toUpperCase(s.charAt(0))));
    }
}

```

如果有两个同名的导入类，例如下面示例中的 Date，编译器就会出错：



```
import module java.base;
import module java.sql;

```

如果一个导入模块临时导入了另一个模块，那么我们也可以使用临时导入模块导出包中的所有类，而无需显式导入。


例如，java.sql 模块转义导入了 java.xml 模块：



```
module java.sql {
  . . .
  requires transitive java.xml;
  . . .
}

```

因此，在下面的示例中，我们不需要显式导入 SAXParserFactory 和 SAXParser，也不需要显式导入 java.xml 模块：



```
SAXParserFactory factory = SAXParserFactory.newInstance();
SAXParser saxParser = factory.newSAXParser();

```

## Flexible Constructor Bodies (二次预览)


在 JDK 23 之前，下述代码中，Child1的构造函数，只能先通过super构造父类，然后才能初始化子类的 b 这个变量。



```
public class FlexibleConstructorBodies {
    void main() {
        new Child1(1, 2);
    }
}


class Parent {
    private final int a;

    public Parent(int a) {
        this.a = a;
        printMe();
    }

    void printMe() {
        System.out.println("a = " + a);
    }
}

// JDK 23 之前
class Child1 extends Parent {
    private final int b;

    public Child1(int a, int b) {
        super(verifyParamsAndReturnA(a, b));
        this.b = b;
    }

    @Override
    void printMe() {
        super.printMe();
        System.out.println("b = " + b);
    }

    private static int verifyParamsAndReturnA(int a, int b) {
        if (a < 0 || b < 0) throw new IllegalArgumentException();
        return a;
    }
}

```

当我们执行



```
new Child1(1,2);

```

这段代码的时候，本来我们期待返回的是



```
a = 1
b = 2

```

但是由于父类在构造时候调用了`printMe()`，且这个调用是在 b 变量初始化之前调用的，所以导致程序执行的结果是



```
a = 1
b = 0

```

今后，在使用 super(...) 调用超级构造函数之前，以及在使用 this(...) 调用替代构造函数之前，我们可以执行任何不访问当前构造实例（即不访问其字段）的代码


此外，我们还可以初始化正在构造的实例的字段。详见[JEP 482](https://github.com):[westworld加速](https://tianchuang88.com)


在 JDK 23 上，上述代码可以调整为:



```
class Child2 extends Parent {
    private final int b;

    public Child2(int a, int b) {
        if (a < 0 || b < 0) throw new IllegalArgumentException();  // ⟵ Now allowed!
        this.b = b;                                                // ⟵ Now allowed!
        super(a);
    }

    @Override
    void printMe() {
        super.printMe();
        System.out.println("b = " + b);
    }
}

```

其中构造函数中，a和b的初始化和判断，都可以在super(...)函数调用之前，
执行



```
new Child2(1,2);

```

打印结果为预期结果



```
a = 1
b = 2

```

## Implicitly Declared Classes and Instance Main Methods (第三次预览)


最早出现在 JDK 21 中，见[Java SE 21 新增特性](https://github.com)


原来我们写一个main方法，需要



```
public class UnnamedClassesAndInstanceMainMethodsTest {

    public static void main(String[] args) {
        System.out.println("Hello World!");
    }

}


```

而且Java文件的名称需要和UnnamedClassesAndInstanceMainMethodsTest保持一致，到了JDK 23，上述代码可以简化成



```
void main() {
    System.out.println("hello world");
}

```

甚至连 public class ... 这段也不需要写。


## 更多


[Java SE 7及以后各版本新增特性，持续更新中...](https://github.com)


## 参考资料


[Java Language Changes for Java SE 23](https://github.com)


[JDK 23 Release Notes](https://github.com)


[JAVA 23 FEATURES(WITH EXAMPLES](https://github.com)


