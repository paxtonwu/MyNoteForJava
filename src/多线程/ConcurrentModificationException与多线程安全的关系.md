# ConcurrentModificationException与多线程安全的关系



项目中开发用到ArrayList时，产生一些问题。 记录一些自己的理解，加深印象，当然网上也有其它的源码分析的资料

---------------

## 1.问题: 单线程下ArrayList获取跌代器iterator后，能不能再修改ArrayList的长度，如调用add、remove?

## 结论:

单线程下ArrayList获取跌代器iterator后，是不能再修改ArrayList的长度的，如调用add、remove的。否则 会抛出运行时异常 ConcurrentModificationException, 从字面意义上理解就是 Concurrent:并发，Modification:修改， 即 并发修改异常。 但我们这是在单线程环境下，哪儿来的并发呢？  这里，我认为 java设计之初的理念上， Concurrent 不能仅仅理解为多线程，理解为并发是合理，并发即同时发生，在遍历的同时又发生了增删等操作。

单线程下创建跌代器： Iterator iterator = ArrayList.iterator(); 这个 Iterator 实质是 ArrayList 的 非静态内部类，在创建之初 便赋值为 ArrayList的 长度， 在之后 的iterator 的每一步操作时，都会检验 记录下的数组长度与 这个iterator所代表的ArrayList的长度 是否相等， 若相等则正常执行；若不相等，则会抛出ConcurrentModificationException；  

![](C:\Users\Admin\Pictures\面试笔记截图\Iterator_ArrayList.png)

为什么会有这样的设计呢？

设计者认为 一个跌代器作为 ArrayList的 操作者，那它 应该能通过 跌代 来 操作完整的ArrayList 数据，当外界ArrayList发生改变，而又无法通知到 Iterator时，这时将会引发很多不可确定性，给语言的使用者、使用目的 带来困扰。 因此设计者仅仅是通过抛出一个 运行时异常，来 禁止开发者这样调用。这样至少保证了程序的功能没有问题。

------------

## 2.问题: 单线程下在iterator循环中增加、删除元素，如何操作？

## 结论:

单线程下在iterator循环中删除,添加元素，必须调用 iterator.remove、add，而不能调用 ArrayList.remove、add；否则iterator将抛出ConcurrentModificationException.

-----------------

## 3.问题: 多线程下使用ArrayList.iterator时也会抛出 ConcurrentModificationException异常吗？

## 结论:

会。 例如，线程1 里 创建一个 ArrayList的 iterator，它正在遍历时， 线程2 往这个ArrayList里add、remove等操作，导致修改了这个ArrayList的长度， 那么线程1的 iterator 在执行时也会 判定长度发生变化，而抛出ConcurrentModificationException.

--------------

## 4.问题: ConcurrentModificationException和多线程安全是什么关系？ 是否解决了多线程安全后就不会抛出 ConcurrentModificationException异常?

## 结论:

ConcurrentModificationException 和多线程安全并没有直接的关系。

a、多线程安全 指的是 多线程在操作同一个对象时，不会互相干扰； 并不是说 某个类存在多线程安全问题，就会抛出ConcurrentModificationException异常。 

b、即使解决了多线程安全问题，也还是有可能抛出ConcurrentModificationException； 即使解决了 ConcurrentModificationException异常，也可能存在多线程安全问题； 这两者并不等价；

c、多线程安全问题并不会抛出运行时异常而终止运行。它只会在很小的概率下出现，但这种小概率在大量用户、多次运行时就一定会出现 ，而一旦出现这种问题，将对业务系统造成重大损失。

d、至于如何解决多线程安全问题就不在这篇文章的讲解范围了，

# More

详情请见:

 [多线程.md - link.lnk](C:\Users\Admin\Desktop\我的面试笔记\note_links\多线程.md - link.lnk) 