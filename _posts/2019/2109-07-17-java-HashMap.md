---
layout: post
title: HashMap原理分析
category: java
tags: [java]
keywords: java,HashMap,原理分析
---
## 前言
HashMap是开发中常用的键值对集合类，在使用中往往只是put，get，本文是研读了源码后对HashMap的原理分析，是本人的个人理解，仅供参考（本文原理为jdk8版本的HashMap）

## 一、HashMap简介
基于哈希表的 Map 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。
HashMap实际上是由数组+链表（1.8加入了红黑树提高检索性能）组成的，初始化默认一个长度16的数组，放入是根据规则计算放在哪个位置上

## 二、Hash原理简述
### put操作
当一个键值对key-value需放入HashMap中时，做以下操作：
1. 计算key的hash值，并将hash值的高16位保持不变，低16位与高16异或，得到一个新的hash数（此操作是为了减少hash碰撞）；
2. 将hash值和当前HashMap长度-1（位数从0开始故-1）取位与操作，得到存放的index；
3. 判断该位置是否已经有节点（即hash值是否碰撞），无，创建Node节点对象放入键值对key-value；有，判断hash值是否相同，相同，新Node覆盖老Node，不相同，新放入的Node链接在原来Node对象后面，构成一个链表对象放入该位置（也可能是红黑树对象）；**这个说简单点就是，该位置没有就放入Node节点，有且key相同就覆盖掉，有但key不同，就把原来的和这次放入的Node合并成一个链表放在改位置上**
4. 放入结束后，判断当前HashMap的容量是否大于临界值（当前长度 x 0.75），不大于，结束;大于，扩容（当前长度 x 2）并更重新计算各节点位置

### get操作
当由一个键key获取HashMap中的value时，做以下操作：
1. 前两步同put的1和2，根据key计算hash和位置；
2. 获取该位置上的对象，为null，返回null；不为null，判断该对象的hash是否和key的hash相同，相同返回该节点的value，不同，判断该对象是否为链表或红黑树Node，是，向后寻找，找到hash相同的节点返回，最后还是找不到，返回null

### remove操作
remove有多个重载方法，但最后处理是都是用的key来操作：
1. 前两步同put的1和2，根据key计算hash和位置；
2. 同get操作的，只不过将找到node位置赋值为空或在链表上移除或在树上移除；

## 三、Hash源码详述
### put操作
put方式实际内部调用的是putVal()方法实现的，故直接贴putVal()的源码：
```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
逻辑详解：
1. 先是判断，hashmap中数组对象是否为空，为空，初始化即创建一个长度为16的数组；
2. p = tab[i = (n - 1) & hash]) == null 得到对应的数组位置，并判断该位置上是否为空，为空直接将key-value以及hash值创建一个对象放入（即newNode(hash, key, value, null)）该位置；说明：创建的Node对象有链表功能，即对象中有next属性，可指向链路的另一Node
3. 该位置不为空，判断对象的hash值、key与新传入的是否相同，相同说明是同一个key，则将新的value替换覆盖老的value；
4. 该位置不为空，对象的hash值、key与新传入的不相同，判断该位置上对象类型是否为treeNode，是，将hash, key, value添加到treeNode中；
5. 该位置不为空，对象的hash值、key与新传入的不相同，判断该位置上对象类型也不是treeNode，则为链表对象，将新传入的key-value及hash值创建newNode(hash, key, value, null)，放在链表对象的末端，并判断链表长度是否超过7，若超过7，则将链表转换为红黑树TreeNode，以提高检索性能
5. 如果map中存在传入的key，即代码中的if (e != null) {.. }说明处理后map中数组中实际对象个数没有变，这将将e中的老值替换为新值，并将老值返回，ps：这里替换新老值，主要是为第3步中服务的，3中替换逻辑即是这里实现的，并且由于实际对象个数没变，故直接返回不执行下面的重置数组大小的逻辑；
6. 数组中添加对象后，会判断当前实际对象个数是否大于临界值（数组长度x0.75）,大于扩容数组大小（2倍）和扩容后对象位置调，小于，结束

### get操作和remove操作
类同put，get的代码逻辑为put的反向操作，remove的代码逻辑大致与get相同，只不过是将取出值改为移除