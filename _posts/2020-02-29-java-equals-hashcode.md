---
layout: post
category: life
title: 灵魂拷问：equals和hashCode是远房亲戚吗？
tagline: by 沉默王二
tags: 
  - java
---

在逛 programcreek 的时候，我发现了一些专注细节但价值连城的主题。比如说：Java 的 `equals()` 和 `hashCode()` 是远房亲戚吗？像这类灵魂拷问的主题，非常值得深入地研究一下。

<!--more-->



另外，我想要告诉大家的是，研究的过程非常的有趣，就好像在迷宫里探宝一样，起初有些不知所措，但经过一番用心的摸索后，不但会找到宝藏，还会有一种茅塞顿开的感觉，非常棒。

![](http://www.itwanger.com/assets/images/2020/02/java-equals-hascode-01.png)


对于绝大多数的初级程序员或者说不重视“内功”的老鸟来说，往往停留在“知其然不知其所以然”的层面上——会用，但要说底层的原理，可就只能挠挠头双手一摊一张问号脸了。

很长一段时间内，我，沉默王二也一直处于这种层面上。但我决定改变了，因为“内功”就好像是在打地基，只有把地基打好了，才能盖起经得住考验的高楼大厦。借此机会，我就和大家一起，对“equals() 和 hashCode()”进行一次深入地研究。

`equals()` 和 `hashCode()` 是 Java 的超级祖先类 Object 定义的两个重要的方法：

```java
public boolean equals(Object obj)
public int hashCode()
```

讲道理，单从方法的定义上来看，`equals()` 和 `hashCode()` 这两个方法之间没有任何亲戚关系，远房都够不上资格。但往深处扒拉，它们之间还真的是有千丝万缕的关系。到底是什么关系呢？如果大家伙比较感兴趣的话，就请随我来，打怪进阶喽！

### 01、equals()

为了勾起大家的好奇心，我特意编写了下面这个示例。

```java
public class Cmower {
    private String name;

    public Cmower(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null) return false;
        if (!(obj instanceof Cmower))
            return false;
        if (obj == this)
            return true;
        return this.name.equals(((Cmower) obj).name);
    }

    public static void main(String[] args) {
        Cmower a1 = new Cmower("沉默王二");
        Cmower a2 = new Cmower("沉默王三");

        Map<Cmower, Integer> m = new HashMap<Cmower, Integer>();
        m.put(a1, 18);
        m.put(a2, 28);
        System.out.println(m.get(new Cmower("沉默王二")));
    }
}
```

我们定义了一个 Cmower 类，它有一个字段 name；然后我们重写了 `equals()` 方法：

1）如果指定对象为 null，返回 false；

2）如果指定对象的类型不是 Cmower 类，返回 false；

3）如果指定对象“==”当前对象，返回 true；

4）如果指定对象的 name 和当前对象的 name 相等，返回 true；意味着只要 name 相等，两个对象就是 equals 的。

在 main 方法中，我们创建了两个 Cmower 类型的对象，name 分别为“沉默王二”和“沉默王三”，并将它们作为键放入了 HashMap 当中；按理说，只要键的 name 为“沉默王二”，程序就应该能够获取我们之前放入的 Cmower 对象。但结果却“出人意料”：

```
null
```

可明明 HashMap 中放入了“沉默王二”啊，debug 也可以证明这一点。

![](http://www.itwanger.com/assets/images/2020/02/java-equals-hascode-02.png)


那究竟是哪里出了错呢？

### 02、hashCode()

开门见山地说吧，问题出在 `hashCode()` 身上，Cmower 类没有重写该方法。借此机会交代一下 `equals()` 和 `hashCode()` 这两个方法之间的关系吧：

1）如果两个对象需要相等（equals），那么它们必须有着相同的哈希码（hashCode）；

2）但如果两个对象有着相同的哈希码，它们却不一定相等。

这就好像在说：你和你对象要想在一起天长地久，就必须彼此相爱；但即便你和你对象彼此相爱，但却不一定真的能在一起。

（扎心了，老铁）

HashMap 之所以能够更快地通过键获取对应的值，是因为它的键位上使用了哈希码。当我们需要从 HashMap 中获取一个值的时候，会先把键转成一个哈希码，判断值所在的位置；然后在使用“==”操作符或者 `equals()` 方法比较键位是否相等，从而取出键位上的值。

可以查看一下 HashMap 类的 `get()` 方法的源码。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

在上例中，之所以没有从 HashMap 中取出 name 为“沉默王二”的 Cmower 对象，就是因为 put 的时候和 get 的时候两个对象的哈希码不同的原因造成的。

明白了原因之后，我们就可以对 Cmower 类进行改造，来看重写后的 `hashCode()` 吧。

```java
@Override
public int hashCode() {
    return this.name.hashCode();
}
```

重写后的 `hashCode()` 方法体非常简单：返回 name 字段的哈希码。这样的话，put 和 get 用到的哈希码就是相同的，因为“沉默王二”的哈希码是 867758096。再次运行程序，你就会发现输出结果不再是 null 而是 18 了。



### 03、小结

1）`equals()` 的作用是用来判断两个对象是否相等。

2）`hashCode()` 的作用是获取对象的哈希码；哈希码一般是一个整数，用来确定对象在哈希表（比如 HashMap）中的索引位置。

拿 HashMap 来说，它本质上是通过数组实现的。当我们要获取某个“值”时，实际上是要获取数组中的某个位置的元素。而数组的位置，就是通过“键”来获取的；更进一步说，是通过“键”对应的[哈希码](http://www.itwanger.com/java/2019/11/08/java-hashmap.html)计算得到的。

3）如果两个对象需要相等（equals），那么它们必须有着相同的哈希码（hashCode）；

4）但如果两个对象有着相同的哈希码，它们却不一定相等。

来对照一下官方给出的关于 `equals()` 和 `hashCode()` 的解释：

*   If two objects are equal according to the `equals(Object)` method, then calling the `hashCode` method on each of the two objects must produce the same integer result.
*   It is *not* required that if two objects are unequal according to the `equals(java.lang.Object)` method, then calling the `hashCode` method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

可能有读者会问：“一定要同时重写 `equals()` 和 `hashCode()` 吗？”

回答当然是否定的。如果对象作为键在哈希表中，那么两个方法都要重写，因为 put 和 get 的时候需要用到哈希码和 `equals()` 方法；

如果对象不在哈希表中，仅用来判断是否相等，那么重写 `equals()` 就行了。

嘿嘿😜，这下搞清楚 `equals()` 和 `hashCode()` 之间的关系了吧！

### 04、鸣谢


好了，读者朋友们，以上就是本文的全部内容了。能看到这里的都是最优秀的程序员，升职加薪就是你了👍。**原创不易，如果觉得有点用的话，请不要吝啬你手中点赞的权力——因为这将是我写作的最强动力**。

![](http://www.itwanger.com/assets/images/cmower_4.png)














