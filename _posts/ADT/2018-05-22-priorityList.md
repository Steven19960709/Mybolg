---
layout: post
title: 优先级队列（1）
date: 2018-05-22
tags: [ADT]
---

## 优先级队列

优先级队列，不再是按照先入先出，或者先进后出，而是按照优先级来进行出队。

每个队列的对象都有一个优先级，不是寻秩访问，寻值访问，寻关键码访问，而是寻优先级访问，call-by-priority。

需要实现的功能接口如下：

<img src="http://os310ujuc.bkt.clouddn.com/pri.png">

### 基本实现

考虑成本和效率。

对于向量，将所有元素组织成一个向量，通过向量的insert即可插入。

<img src="http://os310ujuc.bkt.clouddn.com/pri1.png">

但是复杂度高，insert的时候需要重新遍历整个向量。

如果对于有序向量，也是不可行的，因为维护成本过高，例如查找的时候需要二分logn。

列表同样都是不合适，插入操作需要O(n)。

最后自然考虑使用BBST，三个接口只需O(logn)时间。但是这个对于成本这一项太大了。BBST始终维持所由元素的全序关系，我们只需要维护偏序关系即可。

#### 完全二叉树

<img src="http://os310ujuc.bkt.clouddn.com/pri3.png">

complete Binary tree平衡因子处处非负，是AVL树的特例。

对于结构性，向量等价与完全二叉树，物理上可以直接借助向量实现。完全二叉树的节点和向量中的节点都是一一对应的。这样可以得到向量中父子节点的关系。

例如，以节点11为例，它的父节点可以表示为（11 - 1）/2，对于6的左孩子（1 + 6 * 2）为13.因为这种结构借助了完全二叉树，那么称之为完全二叉堆。

通过继承来作为完全二叉堆（完全二叉树+向量）

<img src="http://os310ujuc.bkt.clouddn.com/pri4.png">

### 堆序性

在完全二叉堆中简历顺序性。数值上，只要 0 < i，必然满足:

    H[i] <= H[Parent(i)]

所以H[0]是全局最大元素。

### 插入和上滤

为插入词条e，只需将e作为末元素接入向量。等价于在完全二叉树末端接入一个节点。可以让结构性自然保持，堆序性却未必可以延续，需要考虑其与父亲节点的关系，是否需要互换位置,不断重复知道e满足堆序性（e每次上升一层）

<img src="http://os310ujuc.bkt.clouddn.com/pri5.png">

#### 实现

<img src="http://os310ujuc.bkt.clouddn.com/pri6.png">

首先将带插入的词条插入到向量之中，秩为n-1,调用percolateUp算法，实行上滤。

#### 效率

没经过一步迭代，节点都会上升一层，所有迭代累计不过O(logn)。但是每次交换swap，可能会含有3次logn的复制，这个时候可以进行一次备份，当确定需要交换位置时，在进行交换，即上滤。

### 删除

<img src="http://os310ujuc.bkt.clouddn.com/pri7.png">

内容读取操作只限于特定的节点（获取最大元或删除最大元），得益于堆序性，首元素就是最大元。摘取向量首元素，代之以末元素，结构性可以保持，但是堆序性不一定。堆顶节点可能会与孩子节点违反堆序性。这个时候，需要进行交换。将e节点与它的较大节点进行交换。没经过一次交换，都会下降一层。这个过程称之为下滤。

#### 实例

<img src="http://os310ujuc.bkt.clouddn.com/pri8.png">

<img src="http://os310ujuc.bkt.clouddn.com/pri9.png">

实现：首先取出对顶元素备份，以备最终返回，然后调用percolteDown对堆顶实施下滤。在一个规模为n的完全二叉树，对秩为i的元素实行下滤，在这个过程中，每次都是找出节点i以及它最多两个孩子中找出最大者j，只要i不是j，这意味着至少有一个孩子比i大，堆序性不满足，这个时候需要将i和j互换，退出循环有两种情况，第一种就是i已经大于任何一个孩子，另一种就是i持续下滤到最底层，这个条件使用ProperParent宏隐式给出的。当while循环退出，返回下滤终点。

与上滤过程相比，上滤每个节点都是与唯一的父亲进行一次比较，但是下滤过程每一步比较都是两次比较，一个节点和它的两个孩子。

### 批量建堆

蛮力方法，可以对每个节点进行一次上滤插入节点。

<img src="http://os310ujuc.bkt.clouddn.com/pri10.png">

这个算法，对于最坏情况，自上而下，自左而右的上滤。最坏情况，节点需要上升到根节点，所需成本线性正比与其深度，即便只考虑地测，有n/2个叶节点，深度均为logn，累计耗时nlogn，所以效率低下。

这样长的时间，本足以全排序，所以能够更快。

#### 自下而上的下滤

<img src="http://os310ujuc.bkt.clouddn.com/pri11.png">

也称弗洛伊德算法。自下而上，自右而左，对于每个节点只需要做一次下滤即可。当根节点的下滤完成，总体节点就构成一个完全二叉堆。

实际的计算成本，来自于节点的下滤。每个内部节点所需的调整时间，正比于其高度而非深度。复杂度为O(n)

造成这种差别，区别在于是对高度求和还是深度求和。因为，对于完全二叉堆，底层的节点占大部分，高层越少。如果以深度做指标，则多耗时间，高层多时间，底层少时间。

### 堆排序

在选择排序中，将数据分为两部分，未排序和已排序两部分，每次从未排序中取出排序到已排序。复杂度为n^2.

针对于以前的线性结构，我们可以利用堆得结构作为序列的结构。然后通过建堆来作为初始化，然后每次取出最大元delMax，复杂度为logn，所以整体复杂度可以上升到nlogn。

<img src="http://os310ujuc.bkt.clouddn.com/pri13.png">

对于空间复杂度，最后可以实现就地排序O(1)复杂度。

<img src="http://os310ujuc.bkt.clouddn.com/pri14.png">

每步迭代为2步，下滤，和交换知道Heap变空。

<img src="http://os310ujuc.bkt.clouddn.com/pri15.png">

初始化调用完全二叉堆的建堆算法，此后进入反复迭代，完全二叉堆当前所在的区间是由lo和high界定的，lo是首元素，hi是尾部哨兵，每一步迭代中，都只需取出首元素位置处的最大元，并将它与末元素交换，而新的对顶元素的下滤功能，也是有delMax在底层完成的。以下是实例：

<img src="http://os310ujuc.bkt.clouddn.com/pri15.png">
<img src="http://os310ujuc.bkt.clouddn.com/pri16.png">
<img src="http://os310ujuc.bkt.clouddn.com/pri17.png">
