---
layout: post
category: java
title: 如何优雅地打印一个Java对象？
tagline: by 沉默王二
tag: java
---

我们来探讨一下如何优雅地打印一个Java对象

<!--more-->






你好呀，我是沉默王二，一个和黄家驹一样身高，和刘德华一样颜值的程序员。虽然已经写了十多年的 Java 代码，但仍然觉得自己是个菜鸟（请允许我惭愧一下）。

在一个月黑风高的夜晚，我思前想后，觉得再也不能这么蹉跎下去了。于是痛下决心，准备通过输出的方式倒逼输入，以此来修炼自己的内功，从而进阶成为一名真正意义上的大神。与此同时，希望这些文章能够帮助到更多的读者，让大家在学习的路上不再寂寞、空虚和冷。

![](http://www.itwanger.com/assets/images/2020/02/print-object-01.png)

为了更好的输入，我选择 Stack Overflow 作为战斗的第一线，毕竟很多前辈都在强烈推荐。本篇文章，我们来探讨一下如何优雅地打印一个Java对象。

真没想到，这个问题的访问量像阿尔泰山一样高，访问量足足有 29+ 万次，这不得了啊！说明有很多很多的程序员被这个问题困扰过。


来回顾一下提问者的问题吧。

提问者定义了这样一个类：

```java
public class Cmower {
    private String name;

    public Cmower(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

然后创建了一个该类的对象，并尝试打印它：

```java
Cmower cmower = new Cmower("沉默王二");
System.out.println(cmower);
```

但是输出的结果并不是他想要的：

```
com.cmower.java_demo.stackoverflow.printObject.Cmower@355da254
```

除此之外，他在打印数组的时候也出现了相似的问题：

```java
Cmower [] cmowers = {new Cmower("沉默王二"), new Cmower("沉默王三")};
System.out.println(cmowers);
```

输出结果为：

```
[Lcom.cmower.java_demo.stackoverflow.printObject.Cmower;@4dc63996
```

`Cmower@355da254` 和 `[LCmower;@4dc63996` 这样的输出结果代表着什么意思呢？怎么样才能把 Cmower 类的 name 打印出来呢？以及如何打印一个对象的列表（数组或者集合）呢？

如果大家也被这样的问题困扰过，或者正在被困扰，就请随[我](https://mp.weixin.qq.com/s/feoOINGSyivBO8Z1gaQVOA)来，咱们肩并肩手拉手一起梳理一下这个问题，并找出最佳答案。Duang、Duang、Duang，打怪进阶喽！

### 01、究竟发生了什么？

所有的 Java 对象都默认附带了一个 `toString()` 的方法，当我们尝试打印这个对象的时候，该方法就会被调用。

```java
System.out.println(object);  // 调用 object.toString()
```

`toString()` 方法由 Object 类（所有 Java 对象的超类）定义，该方法会返回一个看起来晦涩难懂的字符串：

1）Class 名，由包名和类名组成，比如 `com.Cmower`；
2）@ 连接符；
3）十六进制的哈希码。

来看一下该方法的源码：

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```



数组和普通的 Java 对象类似，只有一点点不同——追踪 Class 类的 `getName()` 方法就可以印证这一点。

>If this class object represents a class of arrays, then the internal form of the name consists of the name of the element type preceded by one or more '[' characters representing the depth of the array nesting.

大致的意思就是，如果是一个数组的话，Class 名的前面会有一个或者多个英文中括号“[”，表示数组的维度（一维数组为一个“[”，二维数组为两个“[”），然后再紧跟一个元素的类型首字母。

![](http://www.itwanger.com/assets/images/2020/02/print-object-02.png)

这就是为什么对象数组的前缀是“[L”的原因。是不是有一种恍然大悟的感觉？

![](http://www.itwanger.com/assets/images/2020/02/print-object-03.png)

### 02、自定义输出

如果想在打印的时候输出自己预期的结果，就必须在自定义类中重写 `toString()` 方法，来看例子。

```java
public class Cmower {
    private String name;
    // 省略构造方法和 getter/setter

    @Override
    public String toString() {
        return name;
    }
}
```

当我们再次打印 Cmower 对象时，输出结果就不再是 `com.Cmower@355da254` 了。

```
沉默王二
```

但是这样的结果并不会令我们满意，它有些突兀，没法表示对象的类型。更优雅的做法是这样的：

```java
public class Cmower {
    private String name;
    // 省略构造方法和 getter/setter

    @Override
    public String toString() {
        return getClass().getSimpleName() + "[name=" + name + "]";
    }
}
```

再次打印 Cmower 对象，输出结果为：

```
Cmower[name=沉默王二]
```

这样的形式不仅看起来美观，还能够在调试的时候给出有用的信息。但是，有时候我们不想重写 `toString()` 方法（想保留原有的打印格式 `ClassType@123121`），又想打印该对象的信息，那么最好定义一个新的方法，比如说 `toMyString()` 方法。

### 03、自动化输出

IDE（Eclipse 或者 Intellj IDEA） 通常会提供一种针对类的字段的输出格式，用来覆盖 `toString()` 方法。

```java
@Override
public String toString() {
    return "Cmower{" +
            "name='" + name + '\'' +
            '}';
}
```

另外，一些开源的第三方类库也会提供这样的功能，比如说：

1）Apache Commons Lang 的 ToStringBuilder。

使用方法：

```java
@Override
public String toString() {
    return ToStringBuilder.reflectionToString(this);
}
```

输出结果：

```
com.cmower.printObject.Cmower@355da254[name=沉默王二]
```

2）Google Guava 的 MoreObjects

使用方法：

```java
@Override
public String toString() {
    return MoreObjects.toStringHelper(this)
            .add("name", getName())
            .toString();
}
```

输出结果：

```
Cmower{name=沉默王二}
```


3）Lombok 的 `@toString` 注解（IDE 需要先安装 Lombok 的插件）

使用方法：

```java
@ToString
public class Cmower {

    private String name;

    // 省略构造方法和 getter/setter
}
```

只需要一个 `@toString` 注解，不需要覆盖 `toString()` 方法。

输出结果：

```
Cmower(name=沉默王二)
```

### 04、打印对象列表（数组或者集合）

上述内容已经把打印单个对象的事情唠明白了，are you ok？接下来，我们来说道说道打印对象列表的事儿。

1）数组

`Arrays.toString()` 可以将任意类型的数组转成字符串，包括基本类型数组和引用类型数组。代码示例如下。

```java
Cmower[] cmowers = {new Cmower("沉默王二"), new Cmower("沉默王三")};
System.out.println(Arrays.toString(cmowers));
```

输出结果：

```
[Cmower{name='沉默王二'}, Cmower{name='沉默王三'}]
```

2）集合

对于集合来说，可以直接打印就能输出我们预期的结果。代码示例如下。


```java
List<Cmower> list = new ArrayList<>();
list.add(new Cmower("沉默王二"));
list.add(new Cmower("沉默王三"));
System.out.println(list);
```

输出结果：

```
[Cmower{name='沉默王二'}, Cmower{name='沉默王三'}]
```



### 05、鸣谢

好了，我亲爱的读者朋友，以上就是本文的全部内容了。能在疫情期间坚持看技术文，二哥必须要伸出大拇指为你点个赞👍。

原创不易，如果觉得有点用的话，请不要吝啬你手中**点赞**的权力——因为这将是我写作的最强动力。


![](http://www.itwanger.com/assets/images/cmower_3.png)