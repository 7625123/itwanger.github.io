---
layout: post
category: life
title: 如何比较2个数组相等以及如何检出不匹配项
tagline: by 沉默王二
tags: 
  - java
---

当我空闲的时候，我会密切地观察读者群里的一举一动，不放过他们的任何一个问题，帮助他们成长虽然不是我应尽的义务，但我的确喜欢和他们交流的感觉，毕竟能帮一个是一个。也许我的一个小小的举动，就能够他们跨越成长过程中的一大步——请给我一次骄傲的机会嘛。

<!--more-->



有一次，我在“石器时代”群里发现了 1 个有意思的提问：“如何比较 2 个数组相等以及如何检出不匹配项？”严格意义上讲，这是 2 个问题，其一是比较数组是否相等，其二是如果数组不相等，是哪几个元素导致的。

### 01、比较数组是否相等

可以通过 `Arrays.equals()` 方法来比较 2 个数组是否相等，数组可以是基本数据类型，也可以是引用数据类型，以及泛型。我们就先拿字符串来作为示例吧。

```java
String [] str1 = {"沉","默", "王","二"};
String [] str2 = {"沉","默", "王","二"};
String [] str3 = {"沉","默", "王","三"};
```

现在，就让我们来比较一下：str1 是否与 str2 相等，str1 是否与 str3 相等。（尽管不用代码比较你都能猜出答案，但还是请假装配合一下）

```java
String[] str1 = {"沉", "默", "王", "二"};
String[] str2 = {"沉", "默", "王", "二"};
String[] str3 = {"沉", "默", "王", "三"};

System.out.println(Arrays.equals(str1, str2));
System.out.println(Arrays.equals(str1, str3));
```

程序输出的结果如下所示：

```
true
false
```

不错不错，和我们的预期完全相符。另外，我们还可以通过以下方法来判断 2 个数组中指定的范围是否相等：

```java
boolean equals(Object[] a, int aFromIndex, int aToIndex,
                         Object[] b, int bFromIndex, int bToIndex)
```

来比较一下 str1 和 str3 中前 3 个元素是否相等：

```java
System.out.println(Arrays.equals(str1, 0, 3, str3, 0, 3));
```

程序输出的结果如下所示：

```
true
```

现在，让我们来自定义一个类 Writer，它有两个字段：int 类型的 age，和 String 类型的 name，并重写了 `equals()` 和 `hashCode()` 方法。

```java
public class Writer {
    private int age;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Writer writer = (Writer) o;
        return age == writer.age &&
                Objects.equals(name, writer.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(age, name);
    }

    public Writer(int age, String name) {
        this.age = age;
        this.name = name;
    }

    // getter/setter
}
```

来创建 3 个 Writer 类型的数组：

```java
Writer [] writer1 = {new Writer(18,"沉默王二"),new Writer(16,"沉默王三")};
Writer [] writer2 = {new Writer(18,"沉默王二"),new Writer(16,"沉默王三")};
Writer [] writer3 = {new Writer(18,"沉默王一"),new Writer(16,"沉默王三")};
```

我们来比较一下：writer1 是否与 writer2 相等，writer1 是否与 writer3 相等。

```java
System.out.println(Arrays.equals(writer1,writer2));
System.out.println(Arrays.equals(writer1,writer3));
```

程序输出的结果如下所示：

```
true
false
```

答案完全符合预期，因为我们重写了 `equals()` 方法——如果 age 相等，name 相等，那就表明 2 个 Writer 对象相等。假如我们没有重写该方法，此时就可以借助 Comparator 比较器。

```java
Comparator<Writer> byAge = Comparator.comparing(Writer::getAge);
Comparator<Writer> byName = Comparator.comparing(Writer::getName);
```

byAge 是通过 Writer 的 age 比较的，byName 是通过 Writer 的 name 比较的。然后我们来通过比较器 byAge 和 byName 比较一下 writer1 和 writer3。

```java
System.out.println(Arrays.equals(writer1, writer3, byAge));
System.out.println(Arrays.equals(writer1, writer3, byName));
```

程序输出的结果如下所示：

```
true
false
```

答案完全符合预期，writer1 数组中的 age（18 和 16）和 writer3 数组中的 age（18 和 16）完全相同；writer1 数组中的 name（沉默王二和沉默王三）和 writer3 数组中的 name（沉默王一和沉默王三）不完全相同。

### 02、检出不匹配项

可以通过 `Arrays.mismatch()` 方法检出 2 个数组中哪几个元素不相等。如果 2 个数组完全相等，该方法返回 -1；否则的话，返回第一个不匹配项目的下标。

先来看看 str1 和 str2 是否有不相等的元素吧。

```java
System.out.println(Arrays.mismatch(str1, str2));
```

程序输出的结果如下所示：

```
-1
```

和我们预期的结果一致，因为 str1 和 str2 没有不匹配项。那再来看看 str1 和 str3 吧。

```java
System.out.println(Arrays.mismatch(str1, str3));
```

程序输出的结果如下所示：

```
3
```

的确是从下标为 3 的元素开始不匹配的，因为 str1 中下标为 3 的元素为“二”，str3 中下标为 3 的元素为“三”。

`Arrays.mismatch()` 方法同样适用于自定义类型 Writer。

```java
System.out.println(Arrays.mismatch(writer1,writer2));
System.out.println(Arrays.mismatch(writer1,writer3));
```

程序输出的结果如下所示：

```
-1
0
```

和我们预期的结果一致，因为 writer1 和 writer2 没有不匹配项，writer1 和 writer3 不相等的元素是从第 1 开始的，下标为 0。

也可以通过 Comparator 来检出不相等的元素：

```java
System.out.println(Arrays.mismatch(writer1, writer3, byAge));
System.out.println(Arrays.mismatch(writer1, writer3, byName));
```

程序输出的结果如下所示：

```
-1
0
```

原因我就不再解释了，因为我认为你已经完全掌握了 `Arrays.equals()` 方法和 `Arrays.mismatch()` 方法。希望你能亲自动手试一试哦，源码我已经上传到了[码云](https://gitee.com/qing_gee/JavaPoint/tree/master)。

### 03、鸣谢

好了，亲爱的读者朋友，以上就是本文的全部内容了，是不是又学到了新知识？如果你的答案是肯定的，请为本文点个赞👍。你这个小小的举动不仅能让更多的朋友看到这篇文章，还能顺便给我注满写作的动力，太感谢你了。

如果你觉得我的文章对你有点用，请微信搜索沉默王二，回复关键字「面试」便可免费获取一份我为你精心准备的一线大厂面试资料，等你哟。














