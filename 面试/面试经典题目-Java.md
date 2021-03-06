# HashMap与ConcurrentHashMap

### HashMap 

> HashMap底层数据结构 

JDK7：数组+链表 

JDK8: 数组+链表+红黑树（看过源码的同学应该知道JDK8中即使用了单向链表，也使用了双向链表，双向链表主要是为了链表操作方便，应该在插入，扩容，链表转红黑树，红黑树转链表的过程中都要操作链表）

> JDK8中的HashMap为什么要使用红黑树？ 

当元素个数小于一个阈值时，链表整体的插入查询效率要高于红黑树，当元素个数大于此阈值时，链表整体的插入查询效率要低于红黑树。**此阈值在HashMap中为8** 

> JDK8中的HashMap什么时候将链表转化为红黑树？ 

这个题很容易答错，一部分答案就是：当链表中的元素个数大于8时就会把链表转化为红黑树。但是其实还有另外一个限制：当发现链表中的元素个数大于8之后，还会判断一下当前数组的长度，如果数组长度小于64时，此时并不会转化为红黑树，而是进行扩容。**只有当链表中的元素个数大于8，并且数组的长度大于等于64时才会将链表转为红黑树。** 

上面那样条件扩容的原因是，如果数组长度还比较小，就先利用扩容来缩小链表的长度。 

> JDK8中HashMap的put方法的实现过程？ 

1. 根据key生成hashcode 
2. 判断当前HashMap对象中的数组是否为空，如果为空则初始化该数组 
3. 根据逻辑与运算，算出hashcode基于当前数组对应的数组下标i 
4. 判断数组的第i个位置的元素（tab[i]）是否为空 
   - 如果为空，则将key，value封装为Node对象赋值给tab[i] 
   - 如果不为空： 
     - 如果put方法传入进来的key等于tab[i].key，那么证明存在相同的key 
     - 如果不等于tab[i].key，则： 
       - 如果tab[i]的类型是TreeNode，则表示数组的第i位置上是一颗红黑树，那么将key和 value插入到红黑树中，并且在插入之前会判断在红黑树中是否存在相同的key
       -  如果tab[i]的类型不是TreeNode，则表示数组的第i位置上是一个链表，那么遍历链表寻找是否存在相同的key，并且在遍历的过程中会对链表中的结点数进行计数，当遍历到最后一个结点时，会将key,value封装为Node插入、 
       -  到链表的尾部，同时判断在插入新结点之前的链表结点个数是不是大于等于8（并且数组长度大于等于64），如果是，则将链表改为红黑树。
     - 如果上述步骤中发现存在相同的key，则根据onlyIfAbsent标记来判断是否需要更新value值，然后返回oldValue
5. modCount++ 
6. HashMap的元素个数size加1 
7. 如果size大于扩容的阈值，则进行扩容 

> JDK8中HashMap的get方法的实现过程 

1. 根据key生成hashcode 
2. 如果数组为空，则直接返回空 
3. 如果数组不为空，则利用hashcode和（数组长度-1）通过逻辑与操作算出key所对应的数组下标i 
4. 如果数组的第i个位置上没有元素，则直接返回空 
5. 如果数组的第1个位上的元素的key等于get方法所传进来的key，则返回该元素，并获取该元素的value 
6. 如果不等于则判断该元素还有没有下一个元素，如果没有，返回空 
7. 如果有则判断该元素的类型是链表结点还是红黑树结点
   - 如果是链表则遍历链表
   - 如果是红黑树则遍历红黑树
8. 找到即返回元素，没找到的则返回空 

> JDK7中HashMap插入key-value为什么使用头插法

**至于为什么会采用头插法，据说是考虑到热点数据的原因，即最近插入的元素也很可能最近会被使用到。所以为了缩短链表查找元素的时间，所以每次都会将新插入的元素放到表头。**

这里再稍微拓展下，大家都知道数组查找元素快，而插入或删除元素慢；而链表恰恰相反，查找元素慢，插入或删除快。这是因为两个的数据结构不同而导致的。
数组因为有下标的存在，可以直接根据下标定位到相应元素。而在插入元素或删除元素时，却需要移动该元素后面所有的元素，所以开销会比较大。
而链表没有下标的存在，想要查找元素只能从头结点顺着往下找，若链表非常长且目标元素恰巧在链表尾部，花费的时间相对而言也不短了。同时链表有前继节点与后继节点的存在，当需要插入或删除元素时，只需要修改两个节点的前继节点与后继节点的指向就行了，这也是为什么链表新增或删除元素要比数组快的原因。

> JDK7与JDK8中HashMap的不同点 

1. JDK8中使用了红黑树 
2. JDK7中链表的插入使用的头插法（扩容转移元素的时候也是使用的头插法，头插法速度更快，无需遍历链表，但是在多线程扩容的情况下使用头插法会出现循环链表的问题，导致CPU飙升），JDK8中链表使用的尾插法（JDK8中反正要去计算链表当前结点的个数，反正要遍历链表的，所以直接使用尾插法） 
3. JDK7的Hash算法比JDK8中的更复杂，Hash算法越复杂，生成的hashcode则更散列，那么hashmap 中的元素则更散列，更散列则hashmap的查询性能更好，JDK7中没有红黑树，所以只能优化Hash算法使得元素更散列，而JDK8中增加了红黑树，查询性能得到了保障，所以可以简化一下Hash算法，毕竟Hash算法越复杂就越消耗CPU 
4. 扩容的过程中JDK7中有可能会重新对key进行哈希（重新Hash跟哈希种子有关系），而JDK8中没有这部分逻辑，Java7扩容时，遍历每个节点，并重新hash获得当前数组的位置并添加到链表中；Java8进一步做了优化，将元素的hash和旧数组的大小（大小为2次幂）做与运算，为0则表示数组位置不变，不为0则表示需要移位，新位置为原先位置+旧数组的小大（新数组大小为旧数组翻倍），并将当前链表拆分为两个链表，一个链表放到原先位置，另一个链表放到新位置，效率比Java7高。
5. JDK8中扩容的条件和JDK7中不一样，除开判断size是否大于阈值之外，JDK7中还判断了tab[i]是否为空，不为空的时候才会进行扩容，而JDK8中则没有该条件了 
6. JDK8中还多了一个API：putIfAbsent(key,value) 
7. JDK7和JDK8扩容过程中转移元素的逻辑不一样，JDK7是每次转移一个元素，JDK8是先算出来当前位置上哪些元素在新数组的低位上，哪些在新数组的高位上（拆分为两条链表），然后再一次性转移 


### ConcurrentHashMap 

> JDK7中的ConcurrentHashMap是怎么保证并发安全的？ 

主要利用Unsafe操作+ReentrantLock+分段思想。 

主要使用了Unsafe操作中的： 

1. compareAndSwapObject：通过cas的方式修改对象的属性 
2. putOrderedObject：并发安全的给数组的某个位置赋值 
3. getObjectVolatile：并发安全的获取数组某个位置的元素 

分段思想是为了提高ConcurrentHashMap的并发量，分段数越多则支持的最高并发量越大，程序员可以 通过concurrencyLevel参数来指定并发量。ConcurrentHashMap的内部类Segment就是用来表示某一个段的。 

每个Segment就是一个小型的HashMap的，当调用ConcurrentHashMap的put方法是，最终会调用到 Segment的put方法，而Segment类继承了ReentrantLock，所以Segment自带可重入锁，当调用到 Segment的put方法时，会先利用可重入锁加锁，加锁成功后再将待插入的key,value插入到小型HashMap 中，插入完成后解锁。 

> JDK7中的ConcurrentHashMap的底层原理 

ConcurrentHashMap底层是由两层嵌套数组来实现的： 

1. ConcurrentHashMap对象中有一个属性segments，类型为Segment[]; 
2. Segment对象中有一个属性table，类型为HashEntry[]; 

当调用ConcurrentHashMap的put方法时，先根据key计算出对应的Segment[]的数组下标j，确定好当前 key,value应该插入到哪个Segment对象中，如果segments[ j ]为空，则先创建一个Segment对象，然后利用（CAS）自旋锁的方式在 j 位置引用生成的 Segment对象。 

然后调用Segment对象的put方法。 

Segment对象的put方法会先加锁，然后也根据key计算出对应的HashEntry[]的数组下标 i，然后将 key,value封装为HashEntry对象放入该位置，此过程和JDK7的HashMap的put方法一样，然后解锁。 在加锁的过程中逻辑比较复杂，先通过自旋加锁，如果超过一定次数就会直接阻塞等等加锁。

> JDK8中的ConcurrentHashMap是怎么保证并发安全的？ 

主要利用Unsafe操作+synchronized关键字。 

Unsafe操作的使用仍然和JDK7中的类似，主要负责并发安全的修改对象的属性或数组某个位置的值。 

synchronized主要负责在需要操作某个位置时进行加锁（该位置不为空），比如向某个位置的链表进行插入结点，向某个位置的红黑树插入结点。 

JDK8中其实仍然有分段锁的思想，只不过JDK7中段数是可以控制的，而JDK8中是数组的每一个位置都有一把锁。 

当向ConcurrentHashMap中put一个key,value时， 

1. 首先根据key计算对应的数组下标 i，如果该位置没有元素，则通过自旋的方法去向该位置赋值。 
2. 如果该位置有元素，则synchronized会加锁 
3. 加锁成功之后，再判断该元素的类型
   - 如果是链表节点则进行添加节点到链表中 
   - 如果是红黑树则添加节点到红黑树 
4. 添加成功后，判断是否需要进行树化 
5. addCount，这个方法的意思是ConcurrentHashMap的元素个数加1，但是这个操作也是需要并发安全的，并且元素个数加1成功后，会继续判断是否要进行扩容，如果需要，则会进行扩容，所以这个方法很重要。 
6. 同时一个线程在put时如果发现当前ConcurrentHashMap正在进行扩容则会去帮助扩容。 

> JDK7和JDK8中的ConcurrentHashMap的不同点 

这两个的不同点太多了...，既包括了HashMap中的不同点，也有其他不同点，比如： 

1. JDK8中没有分段锁了，而是使用synchronized来进行控制 
2. JDK8中的扩容性能更高，支持多线程同时扩容，实际上JDK7中也支持多线程扩容，因为JDK7中的扩容是针对独立的一个Segment的，所以也可能多线程扩容，但是性能没有JDK8高，因为JDK8中对于任意一个线程都可以去帮助扩容 
3. JDK8中的元素个数统计的实现也不一样了，JDK8中增加了CounterCell来帮助计数，而JDK7中没 有，JDK7中是put的时候每个Segment内部计数，统计的时候是遍历每个Segment对象加锁统计（当然有一点点小小的优化措施，看视频吧..）。
4. JDK8中进行扩容是（整个老的数组）从后往前面分成一个个固定步长（通过cpu核数进行计算有关）区域进行扩容，如果有线程要在正在进行扩容的区域进行put操作，则会停止put操作进行帮助扩容，扩容完一个区域后再往前一个区域进行扩容（可能是另一个线程，也可能是本线程），扩容完后才进行 put 操作，而JDK7进行扩容是在需要进行扩容的Segment下的数组进行扩容，Segments数组长度不变
5. JDK8因为有红黑树，所以在扩容时如果移动到新位置的红黑树如果结点数小于等于6，则转为链表，JDK8和JDK7进行转移数据时，操作是差不多的，JDK7是找到最后要移动到新位置一样的一段链表，然后移动到计算出来的新位置，然后将剩下的链表逐个遍历移动到新位置，而JDK8是找到最后要移动到新位置一样的一段链表（即使是红黑树，其实里面隐藏着一个双向链表，方便操作），然后将剩下的链表进行遍历，分成两个链表，分别放到计算出来的两个新位置（其中一个链表链接着前面找到的最后一段相同位置的链表）。
6. JDK8节点插入采用尾插法，JDK7采用头插法



# 一文搞明白位运算、补码、反码、原码

在平时看各种框架的源码的过程中，经常会看到一些位移运算，所以作为一个Java开发者是一定掌握位移运算的。

### 正数位移运算

Java中有三个位移运算：

- `<<：左移`
- `>>：右移`
- `>>>：无符号右移`
我们直接看一下Demo:

```java
System.out.println(2 << 1);     // 4
System.out.println(2 >> 1);     // 1
System.out.println(2 >>> 1);    // 1
System.out.println(-2 << 1);    // -4
System.out.println(-2 >> 1);    // -1
System.out.println(-2 >>> 1);   // 2147483647
```

乍一眼看到上面Demo的打印结果，你应该是懵逼的，接下来我来解释一下这个结果到底是如何运算出来的。

上面的Demo中有“2”和“-2”，这是两个十进制数，并且是int类型的(java中占四个字节)，位运算是基于二进制bit来的，所以我们需要**将十进制转换为二进制之后再进行运算**：

- `2 << 1`：十进制“2”转换成二进制为“00000000 00000000 00000000 00000010”，再将二进制左移一位，高位丢弃，低位补0，所以结果为“00000000 00000000 00000000 00000100”，换算成十进制则为“4”
- `2 >> 1`：十进制“2”转换成二进制为“00000000 00000000 00000000 00000010”，再将二进制右移一位，低位丢弃，高位补0，所以结果为“00000000 00000000 00000000 00000001”，换算成十进制则为“1”
对于这两种情况非常好理解，那什么是**无符号右移**，以及负数是怎么运算的呢？
我们先来看`-2 << 1`与`-2 >> 1`，这两个负数的左移与右移操作其实和正数类似，都是先将十进制数转换成二进制数，再将二进制数进行移动，所以现在的关键是负数如何用二进制数进行表示。

### 原码、反码、补码

接下来我们主要介绍十进制数用二进制表示的不同方法，所以为了简洁，我们用一个字节，也就是8个bit来表示二进制数。

#### 原码
| 十进制 | 原码      |
| ------ | --------- |
| 2      | 0000 0010 |
| -2     | 1000 0010 |


原码其实是最容易理解的，只不过需要利用二进制中的第一位来表示符号位，0表示正数，1表示负数，所以可以看到，一个数字用二进制原码表示的话，取值范围是`-111 1111 ~ +111 1111`，换成十进制就是`-127 ~ 127`。

#### 反码

在数学中我们有加减乘除，而对于计算机来说最好只有加法，这样计算机会更加简单高效，我们知道在数学中`5-3=2`，其实可以转换成`5+(-3)=2`，这就表示减法可以用加法表示，而乘法是加法的累积，除法是减法的累积，所以在计算机中只要有加法就够了。
一个数字用原码表示是容易理解的，但是需要单独的一个bit来表示符号位。并且在进行加法时，计算机需要先识别某个二进制原码是正数还是负数，识别出来之后再进行相应的运算。这样效率不高，能不能让计算机在进行运算时不用去管符号位，也就是说让符号位也参与运算，这就要用到反码。

| 十进制 | 原码      | 反码      |
| ------ | --------- | --------- |
| 2      | 0000 0010 | 0000 0010 |
| -2     | 1000 0010 | 1111 1101 |

正数的反码和原码一样，负数的反码就是在原码的基础上符号位保持不变，其他位取反。
那么我们来看一下，用反码直接运算会是什么情况，我们以`5-3`举例。
`5 - 3` 等于 `5 + (-3)`

| 十进制 | 原码      | 反码      |
| ------ | --------- | --------- |
| 5      | 0000 0101 | 0000 0101 |
| -3     | 1000 0011 | 1111 1100 |


```java
5-3
= 5+(-3)
= 0000 0101(反码) + 1111 1100(反码) 
= 0000 0001(反码)
= 0000 0001(原码) 
= 1
```

这不对呀?!! 5-3=1?，为什么差了1？
我们来看一个特殊的运算：

```java
1-1
= 1+(-1)
= 0000 0001(反码) + 1111 1110(反码)
= 1111 1111(反码)
= 1000 0000(原码)
= -0
```

我们来看一个特殊的运算：

```java
0+0
= 0000 0000(反码) + 0000 0000(反码)
= 0000 0000(反码)
= 0000 0000(原码)
= 0
```

我们可以看到1000 0000表示-0，0000 0000表示0，虽然-0和0是一样的，但是在用原码和反码表示时是不同的，我们可以理解为在用一个字节表示数字取值范围时，这些数字中多了一个-0，所以导致我们在用反码直接运算时符号位可以直接参加运算，但是结果会不对。

#### 补码

为了解决反码的问题就出现了补码。

| 十进制 | 原码      | 反码      | 补码      |
| ------ | --------- | --------- | --------- |
| 2      | 0000 0010 | 0000 0010 | 0000 0010 |
| -2     | 1000 0010 | 1111 1101 | 1111 1110 |


正数的补码和原码、反码一样，负数的补码就是反码+1。

| 十进制 | 原码      | 反码      | 补码      |
| ------ | --------- | --------- | --------- |
| 5      | 0000 0101 | 0000 0101 | 0000 0101 |
| -3     | 1000 0011 | 1111 1100 | 1111 1101 |


```java
5-3
= 5+(-3)
= 0000 0101(补码) + 1111 1101(补码)
= 0000 0010(补码)
= 0000 0010(原码) 
= 2
```

5-3=2！！正确。
再来看特殊的：

```java
1-1
= 1+(-1)
= 0000 0001(补码) + 1111 1111(补码)
= 0000 0000(补码)
= 0000 0000(原码)
= 0
```

1-1=0！！正确
再来看一个特殊的运算：

```java
0+0
= 0000 0000(补码) + 0000 0000(补码)
= 0000 0000(补码)
= 0000 0000(原码)
= 0
```

0+0=0！！也正确。
所以，我们可以看到补码解决了反码的问题。
所以对于数字，我们可以使用补码的形式来进行二进制表示。

### 负数位移运算

我们再来看`-2 << 1`与`-2 >> 1`。
-2用原码表示为`10000000 00000000 00000000 00000010`
-2用反码表示为`11111111 11111111 11111111 11111101`
-2用补码表示为`11111111 11111111 11111111 11111110`
`-2 << 1`，表示-2的补码左移一位后为`11111111 11111111 11111111 11111100`，该补码对应的反码为

```java
  11111111 11111111 11111111 11111100 
- 00000000 00000000 00000000 00000001
= 11111111 11111111 11111111 11111011
```

该反码对应的原码为：符号位不变，其他位取反，为`10000000 00000000 00000000 00000100`，表示-4。
所以`-2 << 1 = -4`。
同理`-2 >> 1`是一样的计算方法，这里就不演示了。

### 无符号右移

上面在进行左移和右移时，我有一点没讲到，就是在对补码进行移动时，符号位是固定不动的，而无符号右移是指在进行移动时，**符号位也会跟着一起移动**。
比如`-2 >>> 1`。
-2用原码表示为`10000000 00000000 00000000 00000010`
-2用反码表示为`11111111 11111111 11111111 11111101`
-2用补码表示为`11111111 11111111 11111111 11111110`
-2的补码右移1位为：`01111111 11111111 11111111 11111111`
右移后的补码对应的反码、原码为：`01111111 11111111 11111111 11111111` （因为现在的符号位为0，表示正数，正数的原、反、补码都相同）
所以，对应的十进制为2147483647。
也就是`-2 >>> 1 = 2147483647`

### 总结

文章写的可能比较乱，希望大家能看懂，能有所收获。这里总结一下，我们可以发现：

2 << 1 = 4 = 2*2

2 << 2 = 8 = 2*2 *2

2 << n = 2*2^n^

m << n = m * 2^n^

右移则相反，所以大家以后在源码中再看到位运算时，可以参考上面的公式。



# Integer.highestOneBit(int i)方法的作用与底层实现

在Integer类中有这么一个方法，你可以给它传入一个数字，它将返回最大的小于等于这个数字的一个2的幂次方数。这个方法就是highestOneBit(int i)。

比如下面的Demo，注意方法的输入与返回值：
```java
System.out.println(Integer.highestOneBit(15));  // 输出8
System.out.println(Integer.highestOneBit(16));  // 输出16
System.out.println(Integer.highestOneBit(17));  // 输出16
```


这个方法的实现代码量也是非常少的：
```java
public static int highestOneBit(int i) {
	// HD, Figure 3-1
	i |= (i >>  1);
	i |= (i >>  2);
	i |= (i >>  4);
	i |= (i >>  8);
	i |= (i >> 16);
	return i - (i >>> 1);
}
```

接下来，我们就来详细分析一下这块代码的逻辑。

首先，对于这个方法的功能：**给定一个数字，找到小于或等于这个数字的一个2的幂次方数。**

如果我们要自己来实现的话，我们需要知道：**怎么判断一个数字是2的幂次方数。**

说真的，我一下想不到什么好方法来判断，唯一能想到的就是一个数字如果把它转换成二进制表示的话，它会有一个规律：**如果一个数字是2的幂次方数，那么它对应的二进制表示仅有一个bit位上是1，其他bit位全为0。**
比如：
十进制6，二进制表示为：0000 0110
十进制8，二进制表示为：0000 1000
十进制9，二进制表示为：0000 1001
所以，我们可以利用一个数字的二进制表示来判断这个数字是不是2的幂次方数。关键代码怎么实现呢？去遍历每个bit位？可以，但是不好，那怎么办？我们还是回头仔细看看Integer是如何实现的吧？

```java
public static int highestOneBit(int i) {
	// HD, Figure 3-1
	i |= (i >>  1);
	i |= (i >>  2);
	i |= (i >>  4);
	i |= (i >>  8);
	i |= (i >> 16);
	return i - (i >>> 1);
}
```

我们发现这段代码中没有任何的遍历，只有位运算与一个减法，也就是说它的实现思路和我们自己的实现思路完全不一样，它的思路就是：**给定一个数字，通过一系列的运算，得到一个小于或等于该数字的一个2的幂次方数。**

也就是：如果给定一个数字18，通过运算后，要得到16。
18用二进制表示为：      0001 0010
想要得到的结果(16)是：0001 0000

那么这个运算的过程无非就是**将18对应的二进制数中除最高位的1之外的其他bit位都清零，则拿到了我们想要的结果。**

那怎么通过位运算来实现这个过程呢？

我们拿18对应的二进制数`0001 0010`来举个例子就行了：
先将`0001 0010`右移1位，得到`0000 1001`，
再与自身进行或运算：得到`0001 1011`。

再将`0001 1011`右移2位，得到`0000 0110`，
再与自身进行或运算：得到`0001 1111`。

再将`0001 1111`右移4位，得到`0000 0001`，
再与自身进行或运算：得到`0001 1111`。

再将`0001 1111`右移8位，得到`0000 0000`，
再与自身进行或运算：得到`0001 1111`。

再将`0001 1111`右移16位，得到`0000 0000`，
再与自身进行或运算：得到`0001 1111`。

再将`0001 1111`无符号右移1位，得到`0000 1111`。

> 关于无符号右移，可以看我之前写的文章。

最后用`0001 1111  - 0000 1111 = 0001 0000`
震惊！得到了我们想要的结果。

其实这个过程可以抽象成这样：
现在有一个二进制数据，`0001****`，我们不关心低位的取值情况，我们对其进行右移并且进行或运算。

先将`0001****`右移1位，得到`00001***`，
再与自身进行或运算：得到`00011***`。

再将`00011***`右移2位，得到`0000011*`，
再与自身进行或运算：得到`0001111*`。

再将`0001111*`右移4位，得到`00000001`，
再与自身进行或运算：得到`00011111`。

后面不用再推算了，到这里我们其实可以发现一个规律：
**右移与或运算的目的就是想让某个数字的低位都变为1，再用该结果 减去 该结果右移一位后的结果，则相当于清零了原数字的低位。即得到了我们想要的结果。**



# String.intern()使用总结

## First Blood

先看下面的代码：

```java
String s = new String("1");
String s1 = s.intern();
System.out.println(s == s1);
```

```
打印结果为：
false
```

对于`new String("1")`，会生成两个对象，一个是String类型对象，它将存储在**Java Heap**中，另一个是字符串常量对象"1"，它将存储在**字符串常量池**中。
`s.intern()`方法首先会去字符串常量池中查找是否存在字符串常量对象"1"，如果存在则返回该对象的地址，如果不存在则在字符串常量池中生成为一个"1"字符串常量对象，并返回该对象的地址。

如下图：![img](https://gitee.com/xudongyin/img/raw/master/img/20200822094507.png)变量`s`指向的是Stirng类型对象，变量`s1`指向的是"1"字符串常量对象，所以`s == s1`结果为false。

## Double kill

在上面的基础上我们再定义一个s2如下：

```java
String s = new String("1");
String s1 = s.intern();
String s2 = "1";
System.out.println(s == s1);
System.out.println(s1 == s2); // true
```

`s1 == s2`为true，表示变量s2是直接指向的字符串常量，如下图：![img](https://gitee.com/xudongyin/img/raw/master/img/20200822094531.png)



## Triple kill

在上面的基础上我们再定义一个t如下：

```java
String s = new String("1");
String t = new String("1");
String s1 = s.intern();
String s2 = "1";
System.out.println(s == s1);
System.out.println(s1 == s2);
System.out.println(s == t);   // false
System.out.println(s.intern() == t.intern());   // true
```

`s == t`为false，这个很明显,变量s和变量t指向的是不同的两个String类型的对象。
`s.intern() == t.intern()`为true，因为intern方法返回的是字符串常量池中的同一个"1"对象，所以为true。

![img](https://gitee.com/xudongyin/img/raw/master/img/20200822094557.png)

## Ultra kill

在上面的基础上我们再定义一个x和s3如下：

```java
String s = new String("1");
String t = new String("1");
String x = new String("1") + new String("1");
String s1 = s.intern();
String s2 = "1";
String s3 = "11";
System.out.println(s == s1);
System.out.println(s1 == s2);
System.out.println(s == t);
System.out.println(s.intern() == t.intern());
System.out.println(x == s3);  // fasle
System.out.println(x.intern() == s3.intern());  // true
```

变量x为两个String类型的对象相加，因为`x != s3`，所以x肯定不是指向的字符串常量，实际上x就是一个String类型的对象，调用`x.intern()`方法将返回"11"对应的字符串常量，所以`x.intern() == s3.intern()`为true。

## Rampage

将上面的代码简化并添加几个变量如下：

```java
String x = new String("1") + new String("1");
String x1 = new String("1") + "1";
String x2 = "1" + "1";
String s3 = "11";

System.out.println(x == s3);  // false
System.out.println(x1 == s3);  // false
System.out.println(x2 == s3); // true
```

`x == s3`为false表示x指向String类型对象，s3指向字符串常量;
`x1 == s3`为false表示x1指向String类型对象，s3指向字符串常量;
`x2 == s3`为true表示x2指向字符串常量，s3指向字符串常量;

所以我们可以看到`new String("1") + "1"`返回的String类型的对象。

## 总结

现在我们知道intern方法就是将字符串保存到常量池中，在保存字符串到常量池的过程中会先查看常量池中是否已经存在相等的字符串，如果存在则直接使用该字符串。
所以我们在写业务代码的时候，应该尽量使用字符串常量池中的字符串，比如使用`String s = "1";`比使用`new String("1");`更节省内存。我们也可以使用`String s  = 一个String类型的对象.intern();`方法来间接的使用字符串常量，这种做法通常用在你接收到一个String类型的对象而又想节省内存的情况下，当然你完全可以String s  = 一个String类型的对象;但是这么用可能会因为变量s的引用而影响String类型对象的垃圾回收。所以我们可以使用intern方法进行优化，但是需要注意的是`intern`能节省内存，但是会影响运行速度，因为该方法需要去常量池中查询是否存在某字符串。

参考：[https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)



# Java集合类

### 接口：Collection

Collection是基本的集合接口，一个Collection代表一组Object，即Collection的元素(Elements)。一些Collection允许相同的元素而另一些不行。一些能排序而另一些不行。Java SDK不提供直接继承自Collection的类，Java SDK提供的类都是继承自Collection的“子接口”如List和Set。

所有实现Collection接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的Collection，有一个Collection参数的构造函数用于创建一个新的Collection，这个新的Collection与传入的Collection有相同的元素。后一个构造函数允许用户复制一个Collection。

主要的一个接口方法：boolean add(Ojbect c)

虽然返回的是boolean，但不是表示添加成功与否，这个返回值表示的意义是add()执行后，集合的内容是否改变了(就是元素的数量、位置等有无变化)。类似的addAll，remove，removeAll，remainAll也是一样的。



### 用Iterator模式实现遍历集合

Collection有一个重要的方法：iterator()，返回一个Iterator(迭代器)，用于遍历集合的所有元素。Iterator模式可以把访问逻辑从不同的集合类中抽象出来，从而避免向客户端暴露集合的内部结构。典型的用法如下：

~~~java
Iterator it = collection.iterator(); // 获得一个迭代器
while(it.hasNext()) {
	Object obj = it.next(); // 得到下一个元素
}
~~~

不需要维护遍历集合的“指针”，所有的内部状态都由Iterator来维护，而这个Iterator由集合类通过工厂方法生成。

每一种集合类返回的Iterator具体类型可能不同，但它们都实现了Iterator接口，因此，我们不需要关心到底是哪种Iterator，它只需要获得这个Iterator接口即可，这就是接口的好处，面向对象的威力。

要确保遍历过程顺利完成，必须保证遍历过程中不更改集合的内容(Iterator的remove()方法除外)，所以，确保遍历可靠的原则是：只在一个线程中使用这个集合，或者在多线程中对遍历代码进行同步。

由Collection接口派生的两个接口是List和Set。

***\*List接口\****

List是有序的Collection，使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引(元素在List中的位置，类似于数组下标)来访问List中的元素，这类似于Java的数组。和下面要提到的Set不同，List允许有相同的元素。

除了具有Collection接口必备的iterator()方法外，List还提供一个listIterator()方法，返回一个ListIterator接口，和标准的Iterator接口相比，ListIterator多了一些add()之类的方法，允许添加，删除，设定元素，还能向前或向后遍历。

实现List接口的常用类有LinkedList，ArrayList，Vector和Stack。

***\*LinkedList类\****

LinkedList实现了List接口，允许null元素。此外LinkedList提供额外的get，remove，insert方法在LinkedList的首部或尾部。这些操作使LinkedList可被用作堆栈(stack)，队列(queue)或双向队列(deque)。

注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：

List list = Collections.synchronizedList(new LinkedList(…));

***\*ArrayList类\****

ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。

size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。

每个ArrayList实例都有一个容量(Capacity)，即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长算法并没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。

和LinkedList一样，ArrayList也是非同步的(unsynchronized)。

***\*Vector类\****

Vector非常类似ArrayList，但是Vector是同步的。由Vector创建的Iterator，虽然和ArrayList创建的Iterator是同一接口，但是，因为Vector是同步的，当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态(例如，添加或删除了一些元素)，这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。

***\*Stack 类\****

Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

***\*Set接口\****

Set是一种不包含重复的元素的Collection，即任意的两个元素e1和e2都有e1.equals(e2)=false，Set多有一个null元素。

很明显，Set的构造函数有一个约束条件，传入的Collection参数不能包含重复的元素。

请注意：必须小心操作可变对象(Mutable Object)。如果一个Set中的可变元素改变了自身状态导致Object.equals(Object)=true将导致一些问题。

***\*Map接口\****

请注意，Map没有继承Collection接口，Map提供key到value的映射。一个Map中不能包含相同的key，每个key只能映射一个value。Map接口提供3种集合的视图，Map的内容可以被当作一组key集合，一组value集合，或者一组key-value映射。

***\*Hashtable类\****

Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空(non-null)的对象都可作为key或者value。

添加数据使用put(key, value)，取出数据使用get(key)，这两个基本操作的时间开销为常数。

Hashtable通过initial capacity和load factor两个参数调整性能。通常缺省的load factor 0.75较好地实现了时间和空间的均衡。增大load factor可以节省空间但相应的查找时间将增大，这会影响像get和put这样的操作。

使用Hashtable的简单示例如下，将1，2，3放到Hashtable中，他们的key分别是”one”，”two”，”three”：

Hashtable numbers = new Hashtable();

numbers.put(“one”, new Integer(1));

numbers.put(“two”, new Integer(2));

numbers.put(“three”, new Integer(3));

要取出一个数，比如2，用相应的key：

Integer n = (Integer)numbers.get(“two”);

System.out.println(“two = ” + n);

由于作为key的对象将通过计算其散列函数来确定与之对应的value的位置，因此任何作为key的对象都必须实现hashCode和equals方法。hashCode和equals方法继承自根类Object，如果你用自定义的类当作key的话，要相当小心，按照散列函数的定义，如果两个对象相同，即obj1.equals(obj2)=true，则它们的hashCode必须相同，但如果两个对象不同，则它们的hashCode不一定不同，如果两个不同对象的hashCode相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的hashCode()方法，能加快哈希表的操作。

如果相同的对象有不同的hashCode，对哈希表的操作会出现意想不到的结果(期待的get方法返回null)，要避免这种问题，只需要牢记一条：要同时复写equals方法和hashCode方法，而不要只写其中一个。

Hashtable是同步的。

***\*HashMap类\****

HashMap和Hashtable类似，不同之处在于HashMap是非同步的，并且允许null，即null value和null key。，但是将HashMap视为Collection时(values()方法可返回Collection)，其迭代器操作时间开销和HashMap的容量成比例。因此，如果迭代操作的性能相当重要的话，不要将HashMap的初始化容量设得过高，或者load factor过低。

***\*WeakHashMap类\****

WeakHashMap是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。

***\*总结\****

如果涉及到堆栈，队列等操作，应该考虑用List，对于需要快速插入，删除元素，应该使用LinkedList，如果需要快速随机访问元素，应该使用ArrayList。

如果程序在单线程环境中，或者访问仅仅在一个线程中进行，考虑非同步的类，其效率较高，如果多个线程可能同时操作一个类，应该使用同步的类。

要特别注意对哈希表的操作，作为key的对象要正确复写equals和hashCode方法。

尽量返回接口而非实际的类型，如返回List而非ArrayList，这样如果以后需要将ArrayList换成LinkedList时，客户端代码不用改变。这就是针对抽象编程。