
# 布隆过滤器定义
本质上布隆过滤器是一种数据结构，比较巧妙的**概率型数据结构**（probabilistic data structure），特点是高效地插入和查询，可以用来告诉你 **“某样东西一定不存在或者可能存在”**。
相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的。

# 数据结构
布隆过滤器是由一个很长的二进制向量和一系列随机映射函数组成，每个key会通过一系列hash函数(假设是k个哈希函数)生成k个值，数值的大小在**0～m-1**之间，`m`是二进制的长度， 然后将二进制向量对应位置设置成1。通过这种方式可以确定，当一个key被hash完之后生成的数字对应位置的值不为1，则可以确定该值不存在。然而，即使对应位置的值都为1也不能确定该值一定存在，判断错误的概率称为误判率， 显然误判率和二进制数组的长度，hash函数的个数还有插入数据的个数相关。如果二进制向量的指都为1了，那么过滤器就没有用了。这里借鉴https://zhuanlan.zhihu.com/p/43263751 的几张图直观的看一下布隆过滤器的数据结构：
![](https://pic3.zhimg.com/80/v2-530c9d4478398718c15632b9aa025c36_1440w.jpg)
![](https://pic4.zhimg.com/80/v2-a0ee721daf43f29dd42b7d441b79d227_1440w.jpg)
![](https://pic3.zhimg.com/80/v2-c0c20d8e06308aae1578c16afdea3b6a_1440w.jpg)

值得注意的是，4 这个 bit 位由于两个值的哈希函数都返回了这个 bit 位，因此它被覆盖了。现在我们如果想查询 “dianping” 这个值是否存在，哈希函数返回了 1、5、8三个值，结果我们发现 5 这个 bit 位上的值为 0，说明没有任何一个值映射到这个 bit 位上，因此我们可以很确定地说 “dianping” 这个值不存在。而当我们需要查询 “baidu” 这个值是否存在的话，那么哈希函数必然会返回 1、4、7，然后我们检查发现这三个 bit 位上的值均为 1，那么我们可以说 “baidu” 存在了么？答案是不可以，只能是 “baidu” 这个值可能存在。

这是为什么呢？答案跟简单，因为随着增加的值越来越多，被置为 1 的 bit 位也会越来越多，这样某个值 “taobao” 即使没有被存储过，但是万一哈希函数返回的三个 bit 位都被其他值置位了 1 ，那么程序还是会判断 “taobao” 这个值存在。

# 支持删除吗
传统的布隆过滤器不支持删除，[Counting Bloom filter](https://cloud.tencent.com/developer/article/1136056) 这篇文章中提到了一种实现删除的功能，这种方式在删除前一定要确定元素存在，否则可能会出现误删。

# 如何选择m和k的大小
![](https://pic4.zhimg.com/80/v2-05d4a17ec47911d9ff0e72dc788d5573_1440w.jpg)
$$ m = -\frac{n*lnp}{(ln2)^2} $$
$$
k = \frac{m}{n}ln2
$$