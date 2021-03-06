# Chapter III：二叉堆篇

> 阅读本章内容需要以下前置知识：
>
> C++ 基础语法
>
> 二叉树的基本概念
>
> 时空复杂度的基本概念

## 3.1 有序的数据

在计算机中，大部分的原始数据都是无序的，但是很多时候因为某些需求，我们需要让数据变得有序。

沿用上一章中的例子：

$$A$$ 班的学生们越来越多了，为了方便管理，很多时候需要将他们「排序」，例如在上课点名时，我们希望把所有学生的名字按照字典序排序，然后按照字典序从小到大的顺序来点人回答问题。

$$A$$ 班同学们的原始信息被保存在一个数组中：

```cpp
string names[5] = {"Bob", "David", "Celine", "Alice", "Elizabeth"};
```

当然，我们可以对 $$A$$ 班的所有成员按名字的字典序从小到大排序一次。但是一旦有新的成员加入，或者有老的成员离开，那么所有成员的顺序就又乱了。那么在下次点名时，就又需要重新排序一次。在 $$A$$ 班的成员数量很多的时候，这显得太低效了。

所以我们想要这样的一种数据结构来维护 $$A$$ 班成员的信息：

1. 我们希望这个数据结构能够在成员有增减的时候「自动」的让所有成员保持有序，并且相应的时间开销不能太高；
2. 由于在点名时，一次只会点到一个同学，所以实际上我们不需要让整个序列都有序，只需要保证序列中的第一个学生名字一直都是剩下的所有学生中字典序最小的即可。

事实上，这种数据结构叫做「优先队列」，而优先队列大多数是通过一种数据结构「堆」来实现的。

## 3.2 什么是堆

「堆」（$$Heap$$）是一种数据结构，它可以被视为一棵「完全二叉树」。

这棵完全二叉树具有这样一种「性质」：

> 除了根节点以外，**所有节点的值都不得超过（或小于）其父节点的值**。

这样就可以推出：**堆中最大/最小的元素存放在根节点（称为堆顶）中**。

如果堆顶的元素是堆中「最大」的，那么这个堆就叫做「大顶堆」，或者「最大堆」。

如果堆顶的元素是堆中「最小」的，那么这个堆就叫做「小顶堆」，或者「最小堆」。

通过完全二叉树的性质实现的堆，称为「二叉堆」。

### 3.2.1 堆的容器

由于堆本身可以看作一棵「完全二叉树」，那么堆自然也满足完全二叉树的一个重要性质：

> 一棵完全二叉树，如果按照层序遍历的顺序给树中的所有节点编号为 $$1$$ 到 $$N$$，那么树中所有节点的编号满足以下性质：
>
> 如果一个节点的编号为 $$k$$，那么它的左儿子节点（如果存在）的编号为 $$2k$$，右儿子（如果存在）编号为 $$2k + 1$$ ，父节点（如果存在）的编号为 $$k / 2$$ 。

利用这个重要性质，我们可以直接把完全二叉树的所有节点按编号存储在一个数组中，节点的编号对应数组的下标，如下图所示：

![堆的存储示意图](https://tva1.sinaimg.cn/large/008i3skNly1gwut7k38czj30mj0boaag.jpg)

这种存储方式也叫做「堆式存储」，它具有一个很好的优势，即不再需要通过节点的「指针域」来找到儿子节点,而可以直接通过编号访问。

所以我们可以直接使用一个数组来做为堆的容器。其中堆顶的编号为 $$1$$:

```cpp
const int MAXN = 100;  // 堆的最大容量
string heap[MAXN + 1]; // 使用数组 heap 作为堆的容器
int heapSize = 0;      // heapSize 表示堆的大小，初始值为 0
```

### 3.2.2 堆的方法

堆有两个重要的方法需要实现：

1. 往堆中加入一个元素，即 $$push$$ 方法。
2. 取出堆顶的元素，即 $$pop$$ 方法。

需要注意的是，我们要始终保持堆顶的元素在进行 $$push$$ 和 $$pop$$ 操作之后仍然是堆中最大/小的。

由于我们需要让名字的字典序最小的同学先出队，所以这里需要一个最小堆。

接下来我们来实现一个最小堆的 $$push$$ 和 $$pop$$ 方法。

## 3.3 最小堆的代码实现

### 3.3.1 最小堆的 $$push$$ 方法

最小堆的 $$push$$ 方法实现原理如下：

1. 从「堆尾」加入一个新成员，并把这个新成员设为当前节点 $$cur$$ 。
2. 比较当前节点和它的「父节点」$$fa$$ 的大小：
   * 如果当前节点「小于」父节点，则「交换他们的值」。
   * 如果当前节点「大于等于」父节点，则退出。
3. 将 $$cur$$ 更新为 $$fa$$，重复操作 $$2$$，直到 $$cur$$ 就是根节点（即堆顶，编号为 $$1$$）为止。

下面给出一种代码实现：

```cpp
// ...
void push(string x) {
    heap[++heapSize] = x;  // 将新节点放到堆尾
    int cur = heapSize;    // 设这个节点为当前节点
    while (cur > 1) {      // 循环直到当前节点为根节点（堆顶）
        int fa = cur >> 1; // fa 为当前节点的父节点
        if (heap[cur] >= heap[fa]) break;
        swap(heap[cur], heap[fa]);
        cur = fa;
    }
}
```

使用 $$push$$ 方法将所有的成员都加入堆中，我们就建好了一个小顶堆。

```cpp
// ...
for (int i = 0; i < 5; i++) {
    push(names[i]);
}
```

但是我们现在还无法验证堆顶的学生是否是所有学生中名字的字典序最小的。为了从堆中取出元素，我们需要继续实现 $$pop$$ 方法。

### 3.3.2 最小堆的 $$pop$$ 方法

最小堆的 $$pop$$ 方法实现原理如下：

1. 取出堆顶成员的「值」。
2. 把堆的最后一个节点放到根的位置上，「把根覆盖掉」，并把堆的大小减一。
3. 把根节点设为 「当前的父节点」 $$fa$$ ：
   * 如果 $$fa$$ 没有儿子，则直接退出。
   * 否则，将 $$fa$$ 的两个（或一个）儿子中「值最小」的那个设为「当前的子节点」 $$child$$ 。
   * 比较 $$fa$$ 与 $$child$$ 的值，如果 $$fa$$ 的值「小于等于」 $$child$$ ，则直接退出。
   * 否则交换这两个节点的值，把 $$fa$$ 更新为 $$child$$ ，再执行一次步骤 $$3$$ 。

下面给出一种代码实现：

```cpp
// ...
string pop() {
    string res = heap[1];
    heap[1] = heap[heapSize--];
    int fa = 1;
    while ((fa << 1) <= heapSize) {
        int child = fa << 1;
        if (child < heapSize && heap[child + 1] < heap[child]) child++;
        if (heap[fa] <= heap[child]) break;
        swap(heap[fa], heap[child]);
        fa = child;
    }
    return res;
}
```

$$pop$$ 方法会返回堆顶的成员 ，并且保证新的堆顶成员依然是堆中剩余成员中名字的字典序最小的。

现在，我们可以实现通过使用一个最小堆，来将 $$A$$ 班的学生按名字的字典序从小到大输出了：

```cpp
// ...
while (heapSize) {
    string x = pop();
    cout << "Name: " << x << endl;
}
```

运行之后可以看到输出如下：

```
Name: Alice
Name: Bob
Name: Celine
Name: David
Name: Elizabeth
```

## 3.4 二叉堆的性能及应用

由于二叉堆可以看作一棵完全二叉树，所以不论是 $$push$$ 还是 $$pop$$ 方法，将堆中的最大/小元素移动到堆顶的时间复杂度都不会超过 $$O(log_2^n)$$ 。如果总操作数为 $$n$$ ，那么完成所有操作的总时间不会超过 $$O(nlog_2^n)$$ 。

这是一个非常优秀的时间复杂度，也就是说，二叉堆拥有非常出色的时间性能。

在使用「堆式存储」的情况下，二叉堆空间复杂度为 $$O(n)$$ ，即只需要存储原始数据的空间，这同样非常理想。所以二叉堆被广泛应用在其他算法和数据结构中，例如「堆排序」和实现「优先队列」。

但是美中不足的是，二叉堆在某些场景下并不是太好用。例如，如果你想要知道所有学生中名字的字典序「第二小」的人是谁，你是没有办法直接从堆中得到的：你必须先取出堆顶的成员，然后等待堆把第二小的成员「送」到堆顶，但很多时候你只是想要查询一下，并不想要取出元素。最坏情况下，如果你想要知道一个最小堆中的最大成员，你甚至需要把堆清空，这并不是很方便。

所以，堆并不是一个理想的用于长期存储数据的容器，但是堆的这种「尾部插入，头部取出」的行为跟另一种十分重要的数据结构「队列」不谋而合。再加上堆的有序性，所以堆是最适合用来实现「优先队列」的数据结构。

那么有没有一种更理想的数据结构，可以在保持内部数据有序的前提下，能够更加方便且高效的实现数据的增删查改呢？

这种数据结构就是「二叉搜索树」。
