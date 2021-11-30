# Chapter IV：二叉搜索树篇

> 阅读本章内容需要以下前置知识：
>
> C++ 基础语法
>
> 指针与结构体指针
>
> 二叉树的基本概念
>
> 二叉树的中序遍历

## 4.1 什么是二叉搜索树

二叉搜索树（$$Binary$$ $$Search$$ $$Tree$$，$$BST$$）是一种「内部有序」的数据结构，它是一种特殊的二叉树：

>  左子树上所有节点的数据域均「小于或等于」根节点的数据域，右子树上所有节点的数据域均「大于」根节点的数据域。

满足以上性质的二叉树就是一棵 $$BST$$ ，如下图所示：

![bst_sample_1](https://tva1.sinaimg.cn/large/008i3skNly1gwvtbgu4z0j30jz08zglw.jpg)

这样乍一看似乎看不出来哪里有序。但是对于 $$BST$$ 的每一棵子树来说，都满足：

> 左儿子的值 <= 根节点的值 < 右儿子的值

所以如果我们想升序的打印出 $$BST$$ 中所有节点的值，只需要「中序遍历」$$BST$$ 即可。

不仅如此，根据 $$BST$$ 的性质，我们可以很容易找到树中的这样几个值：

1. $$max$$：树中的最大值。
2. $$min$$ ：树中的最小值。
3. 值 $$x$$ 的前驱（$$Predecessor$$）：树中比 $$x$$ 小的最大值。
4. 值 $$x$$ 的后继（$$Successor$$）：树中比 $$x$$ 大的最小值。

这对于解决我们上一章中提到的那个问题非常方便：如果想要找到所有学生中名字的字典序「第二大」的，那么就只需要找到名字字典序最大的学生的「前驱」就好了。

下面我们来讨论如何构造一棵 $$BST$$ ，并将数据存放到 $$BST$$ 中。

## 4.2 二叉搜索树的构建

对于所有二叉树来说，树的构建过程就是不断向树中插入新节点的过程，$$BST$$ 也不例外。那么首先我们需要使用一个结构体来定义 $$BST$$ 中的节点。

对于二叉树的节点来说，除了数据域之外，都至少需要两个指针域，分别指向节点的左儿子和右儿子。沿用之前的例子，我们想将 $$A$$ 班的学生信息保存在一棵 $$BST$$ 中：

```cpp
#include <string>

const string names[10] = {
    "Harris",
    "Ford",
    "Celine",
    "Jack",
    "Elizabeth",
    "Bob",
    "Gary",
    "Alice",
    "Isabelle",
    "David",
};

struct Node {
    string name; // 数据域：学生的名字
    Node *L, *R; // 指针域，分别指向左儿子和右儿子，初始值都为 NULL
    Node(string s) : name(s), L(NULL), R(NULL) {}
};
```

然后可以将第一个学生作为根节点 $$root$$ ：

```cpp
// ...
Node* root = new Node(names[0]);
```

那么现在我们就需要一个 $$insert()$$ 方法，可以将后续的成员都一一插入到 $$BST$$ 中。

### 4.2.1 往二叉搜索树中插入节点

要向二叉树中插入节点，那么就要先找到一个可以插入新节点的「位置」，即一个指针域为 $$NULL$$ 的地方。

跟普通的二叉树不同，$$BST$$ 必须把比当前节点值小的节点插入到左子树，比当前节点值大的节点插入到右子树，那么根据这个原则，很容易实现一个 $$insert()$$ 方法：

```cpp
// ...
Node* insert(Node* &cur, string name) {
    if (cur == NULL) {
        cur = new Node(name);
        return cur;
    }
    if (name <= cur->name) return insert(cur->L, name);
    else return insert(cur->R, name);
}
```

$$insert()$$ 方法的逻辑非常符合直觉：

- 如果找到了一个空位（$$cur == NULL$$），那么就在这里构造一个新节点并返回。
- 如果待插入的节点的 $$age$$ 值「小于等于」当前节点 $$cur$$ ，就递归的往「左子树」插入；
- 如果待插入的节点的 $$age$$ 值「大于」当前节点 $$cur$$ ，就递归的往「右子树」插入；

需要注意的是：由于 $$insert()$$ 方法会改变节点的指针域，所以对于形参 $$cur$$ 必须使用「引用」。

现在我们可以使用 $$insert()$$ 方法将剩余学生也插入到 $$BST$$ 中，并中序遍历这棵 $$BST$$ 来查看效果：

```cpp
// ...
void inOrder(Node* cur) {
    if (cur == NULL) return;
    inOrder(cur->L);
    cout << "Name: " << cur->name << endl;
    inOrder(cur->R);
}
// ...
for (int i = 1; i < 10; i++) {
    insert(root, names[i]);
}
inOrder(root);
```

运行以上代码可以看到以下输出：

```
Name: Alice
Name: Bob
Name: Celine
Name: David
Name: Elizabeth
Name: Ford
Name: Gary
Name: Harris
Name: Isabelle
Name: Jack
```

所有学生已经按照名字的字典序升序排列了！

下面我们来一一实现之前提到的四种操作。

### 4.2.2 找最大/最小值

根据 $$BST$$ 的定义，$$BST$$ 中的最大值就是「最右边」的节点，最小值就是「最左边」的节点。

那么就可以很简单的写出 $$findMax()$$ 和 $$findMin()$$ 方法：

```cpp
// ...
Node* findMax(Node* cur) {
    return cur->R != NULL ? findMax(cur->R) : cur;
}

Node* findMin(Node* cur) {
    return cur->L != NULL ? findMin(cur->L) : cur;
}
```

这两个方法的逻辑同样符合直觉：只要右/左子树不空，就一直往右/左子树查找。

测试一下效果：

```cpp
// ...
Node* Max = findMax(root);
Node* Min = findMin(root);
cout << "Max: " << Max->name << endl;
cout << "Min: " << Min->name << endl;
```

可以看到输出如下：

```
Max: Jack
Min: Alice
```

很容易就找到了名字的字典序最大和最小的同学。

### 4.2.3 找前驱/后继

之前提到过，$$BST$$ 中，$$x$$ 的前驱 $$pred(x)$$ 的定义是「比 $$x$$ 小的最大值」，后继 $$succ(x)$$ 的定义是「比 $$x$$ 大的最小值」。

也可以这样理解：

- $$pred(x)$$ 是 $$BST$$ 的中序遍历序列中 $$x$$ 的上一个值。
- $$succ(x)$$ 是 $$BST$$ 的中序遍历序列中 $$x$$ 的下一个值。

从 $$BST$$ 的性质可知：$$pred(x)$$ 就是 $$x$$ 的左子树中最大的节点，即「左子树中最右边的节点」，$$succ(x)$$ 就是 $$x$$ 的右子树中最小的节点，即「右子树中最左边的节点」。

通过这种思想，可以很简单的实现 $$findPred()$$ 和 $$findSucc()$$ 方法：

```cpp
Node* findPred(Node* x) { // 找 x 的左子树中最右边的节点
    x = x->L;
    while (x->R != NULL) x = x->R;
    return x;
}

Node* findSucc(Node* x) { // 找 x 的右子树中最左边的节点
    x = x->R;
    while (x->L != NULL) x = x->L;
    return x;
}
```

由于这里的 $$x$$ 是一个结构体指针，即节点本身，所以在此之前需要先使用 $$findNode()$$ 方法找到 $$x$$ 的位置。但是有些时候我们想直接查找「值」。

下面给出一种直接查找「值」的实现（省去查找节点的过程）：

如果想要找到 $$pred(x)$$ ，过程如下：

从根节点 $$root$$ 开始往下找：

- 如果 $$x$$ 大于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在右子树，则返回 $$cur$$ ；
  - 否则则向右子树递归，返回一个结果 $$res$$ ；
    - 如果 $$res == NULL$$ ，则还是返回 $$cur$$ ；
    - 否则返回 $$res$$ 。
- 如果 $$x$$ 小于等于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在左子树，则返回 $$NULL$$ ；
  - 否则，则向左子树递归求解。

一种代码实现如下：

```cpp
// ...
Node* findPred(Node* cur, string x) {
    if (x > cur->name) {
        if (cur->R == NULL) return cur;
        Node* res = findPred(cur->R, x);
        if (res == NULL) return cur;
        return res;
    } else {
        if (cur->L == NULL) return NULL;
        else return findPred(cur->L, x);
    }
}
```

查找 $$succ(x)$$ 的原理是类似的，过程如下：

从根节点 $$root$$ 开始往下找：

- 如果 $$x$$ 小于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在左子树，则返回 $$cur$$ ；
  - 否则则向左子树递归，返回一个结果 $$res$$ ；
    - 如果 $$res == NULL$$ ，则还是返回 $$cur$$ ；
    - 否则返回 $$res$$ 。
- 如果 $$x$$ 大于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在右子树，则返回 $$NULL$$ ；
  - 否则，则向右子树递归求解。

一种代码实现如下：

```cpp
// ...
Node* findSucc(Node* cur, string x) {
    if (x < cur->name) {
        if (cur->L == NULL) return cur;
        Node* res = findSucc(cur->L, x);
        if (res == NULL) return cur;
        return res;
    } else {
        if (cur->R == NULL) return NULL;
        else return findSucc(cur->R, x);
    }
}
```

以上两种实现方式可以根据不同的应用场景灵活的选用。

下面测试一下，找到 $$A$$ 班学生中，名字的字典序「第二大」的。

首先找到所有学生中字典序最大的名字 $$maxName$$ ：

```cpp
// ...
string maxName = findMax(root)->name;
```

字典序「第二大」的，显然就是 $$maxName$$ 的前驱：

```cpp
Node* pred = findPred(root, maxName);
cout << "Second greatest name: " << pred->name << endl;
```

运行之后可以看到输出如下：

```
Second greatest name: Isabelle
```

名字的字典序第二大的人是 $$Isabelle$$ 。

同样，我们来尝试找出 $$A$$ 班中名字的字典序「第二小」的。

首先找到所有学生中的字典序最小的名字 $$minName$$ ：

```cpp
// ...
string minName = findMin(root)->name;
```

名字的字典序「第二小」的，显然就是 $$minName$$ 的后继：

```cpp
Node* succ = findSucc(root, minName);
cout << "Second least name: " << succ->name << endl;
```

运行之后可以看到输出如下：

```
Second least name: Bob
```

名字的字典序第二小的人是 $$Bob$$ 。

### 4.2.4 删除二叉搜索树中的节点

$$BST$$ 删除节点的操作就有些复杂了。因为和插入的时候一样，我们要保证 $$BST$$ 在删除了一个节点之后「仍然是一棵 $$BST$$」。

为了保证这一点，在 $$BST$$ 中删除节点 $$x$$ 通常有两种方法：

1. 找到 $$x$$ 的「前驱节点」$$pred(x)$$ ，用 $$pred(x)$$ 覆盖掉 $$x$$ ，然后删除 $$pred(x)$$ ；
2. 找到 $$x$$ 的「后继节点」$$succ(x)$$ ，用 $$succ(x)$$ 覆盖掉 $$x$$ ，然后删除 $$succ(x)$$ ；

以上操作可能需要「递归」进行。如果递归到了一个「叶子节点」，那么就可以直接删除这个节点，因为删除「叶子节点」不会对其他节点造成影响了。

这两种做法都可以保证删除操作之后仍然是一棵 $$BST$$ 。

一种可行的代码实现如下：

```cpp
// ...
void deleteNode(Node* &cur, string x) {
    if (cur == NULL) return;                    // 如果节点不存在直接返回
    if (x == cur->name) {                       // 如果找到了节点 x
        if (cur->L == NULL && cur->R == NULL) {
            cur = NULL;                         // 如果是叶子节点就直接删除
        } else if (cur->L != NULL) {            // 如果有左子树
            string pred = findMax(cur->L)->name; // 找到左子树中最大的节点，即前驱
            cur->name = pred;                    // 用 pre 覆盖掉 cur
            deleteNode(cur->L, pred);            // 递归的在左子树中继续删除 pre
        } else if (cur->R != NULL) {            // 如果有右子树
            string succ = findMin(cur->R)->name; // 找到右子树中的最小节点，即后继
            cur->name = succ;                    // 用 nex 覆盖掉 cur
            deleteNode(cur->R, succ);            // 递归的在右子树中继续删除 nex
        }
    } else if (x < cur->name) {
        deleteNode(cur->L, x);
    } else if (x > cur->name) {
        deleteNode(cur->R, x);
    }
}
```

现在我们尝试删除掉 $$Bob$$ ，之后再中序遍历 $$BST$$ ：

```cpp
// ...
deleteNode(root, "Bob");
inOrder(root);
```

可以看到输出如下：

```
Name: Alice
Name: Celine
Name: David
Name: Elizabeth
Name: Ford
Name: Gary
Name: Harris
Name: Isabelle
Name: Jack
```

$$Bob$$ 被成功的删除了，而 $$BST$$ 的性质依然得以保留。

### 4.2.5 二叉搜索树中的其他域

有时候根据不同的需求，我们还可以给 $$BST$$ 添加一些额外的「域」，例如指向父节点的指针域 $$F$$ ，维护子树大小数据域的 $$size$$ 等。关于这一部分的相关内容以及代码实现，请参考 [附录一](Extra\_1\.md) 。

## 4.3 二叉搜索树的性能分析

从以上的操作中可以看出，$$BST$$ 中的所有操作的时间开销都只跟 $$BST$$ 的「高度」$$h$$ 有关，时间复杂度都不会超过 $$O(h)$$ 。如果 $$BST$$ 中的节点总个数为 $$N$$ ，那么最理想的情况下（一棵完全二叉树），有 $$h\approx\log_2^N$$ ，即如果 $$BST$$ 是一棵完全二叉树，那么可以认为所有操作的时间复杂度都不会超过 $$O(log_2^N)$$ 。

所以使用 $$BST$$ 来维护 $$A$$ 班学生的信息是十分理想的，理由如下：

1. 内部有序，并且可以方便的修改任何一个成员，而不像二叉堆，只能支持访问头部元素。
2. 高效。理想情况下，不论是查询，修改还是插入和删除的时间复杂度都是 $$O(log_2^N)$$ 级别。

但是很多时候，随着节点的不断插入和删除，$$BST$$ 会渐渐的变得不那么「平衡」，即 $$h$$ 会增大，这就会导致 $$BST$$ 的性能严重下降。最坏情况下，$$BST$$ 会退化成一个链表，这会让某些操作的时间复杂度上升到 $$O(N)$$ 级别。

那么接下来，我们自然就想要 $$BST$$ 能够动态的调整自己的高度，使得内部节点的排列尽量的接近完全二叉树。

这种能够自行调整高度的二叉搜索树，叫做「平衡树」。