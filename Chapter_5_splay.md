# Chapter V：伸展树篇

> 阅读本章内容需要掌握以下知识：
>
> $$BST$$ 的基本原理。
>
> $$BST$$ 中常用方法的代码实现。

在第四章中我们提到，由于 $$BST$$ 的高度 $$h$$ 可能会随着节点的插入和删除变得越来越大，最坏情况下会让 $$BST$$ 退化为一个链表。这会严重影响 $$BST$$ 的性能。

为了解决这个问题，我们需要让 $$BST$$ 在节点的插入和删除过程中能够动态的调节树高，使得内部节点的排列尽量的接近完全二叉树，使得 $$BST$$ 上各种操作的时间复杂度可以稳定在 $$O(log_2^N)$$ 。这样的 $$BST$$ 叫做「平衡树」。

常见的平衡树有 $$AVL$$，$$RBT$$，$$SBT$$ 等。平衡树是一种接近完美的数据结构，在各个层面都有许多重要的应用。如 C++ 标准模板库中的有序数据结构 $$set$$ 和 $$map$$ ，都是使用红黑树（$$RBT$$）来实现内部有序。

美中不足的是，许多平衡树实现自平衡的原理的和代码实现都比较复杂，并且都需要额外的「域」来实现自平衡。而这些额外的「域」除了维持树的平衡之外并没有别的作用。

而在本章中我们要讨论的另外一种 $$BST$$，它的所有操作的均摊时间复杂度同样为 $$O(log_2^N)$$ ，且代码实现相比平衡树来说非常简洁，而且不需要用于维持平衡的额外的「域」。这就是「伸展树」$$Splay\ Tree$$ 。

伸展树是由 $$Daniel\ Sleater$$ 和 $$Robert\ Tarjan$$ 在 $$1985$$ 年发明的一种重要的数据结构。它对于 $$BST$$ 的优化基于一个非常朴素的思想：

> 将操作频率高的节点尽量的移动到离树根更近的位置。

而这是通过一种名为「伸展」（$$splay$$）的操作来实现的。在讨论「伸展」操作之前，我们需要先了解 $$BST$$ 上的一种重要的操作：$$BST$$ 的旋转。

## 5.1 二叉搜索树的旋转

$$BST$$ 的旋转可谓是高级数据结构中最神奇，也最有魅力的操作，它是 $$BST$$ 实现自平衡的基础。$$BST$$ 的旋转有两种方式：「右旋」和「左旋」：

![bst_rotate](https://tva1.sinaimg.cn/large/008i3skNgy1gwza1m8bu1j30ez08f3ys.jpg)

不论是右旋操作还是左旋操作，都保证了旋转之后仍然是一棵 $$BST$$ 。而 $$splay$$ 的「伸展」操作，就是通过不断的「旋转」，将节点 $$x$$ 移动到树根。

从上图中可以看出，右旋（$$Right\ Rotate$$）操作实际上是做了这么两件事：

- 将 $$x$$ 的右子树变成 $$y$$ 的左子树（$$y$$ 是 $$x$$ 的父节点，且 $$x$$ 为 $$y$$ 的左儿子）。
- 将 $$y$$ 变成 $$x$$ 的右儿子。

左旋操作同理（此时 $$x$$ 是 $$y$$ 的父节点，且 $$y$$ 为 $$x$$ 的右儿子）：

- 将 $$y$$ 的左子树变成 $$x$$ 的右子树。
- 将 $$x$$ 变成 $$y$$ 的左儿子。

为了便于实现旋转操作，我们可以这样定义 $$Splay$$ 的节点：

```cpp
struct SplayNode {
    int val;
    SplayNode *L, *R, *F;
    SplayNode(int x) : val(x), L(NULL), R(NULL), F(NULL) {}
};
```

其中除了 $$L$$ 和 $$R$$ 之外，我们添加了一个额外的指针域 $$F$$ ，指向节点的父节点。

现在我们就可以实现 $$rightRotate()$$ 和 $$leftRotate()$$ 操作。

### 5.1.1 伸展树的右旋操作

根据之前的分析，要实现 $$rightRotate()$$ 操作，我们需要完成两件事：

- 将需要旋转的节点 $$cur$$ 的右子树变成 $$y$$ 的左子树（$$y$$ 是 $$cur$$ 的父节点，且 $$cur$$ 为 $$y$$ 的左儿子）。
- 将 $$y$$ 变成 $$cur$$ 的右儿子。

一种代码实现如下：

```cpp
// ...
void rightRotate(SplayNode* &cur) {
    SplayNode* y = cur->F; // 设 y 为当前节点的父节点
    cur->F = y->F; // 将当前节点的父节点更新为 y 的父节点
    if (cur->F) {
        if (y == cur->F->L) cur->F->L = cur; // 如果 y 是左儿子，就将当前节点变成左儿子
        else cur->F->R = cur; // 如果 y 是右儿子，就将当前节点变成右儿子
    }
    if (cur->R) cur->R->F = y; // 将当前节点的右子树变成 y 的左子树
    y->L = cur->R;
    y->F = cur; // 将 y 变成当前节点的右儿子
    cur->R = y;
}
```

在编码时需要注意：由于增加了一个指针域 $$F$$ ，在连接两个节点时需要进行双向绑定。

在伸展树中，将 $$x$$ 的右旋操作记为 $$Zig(x)$$ 。

### 5.1.2 伸展树的左旋操作

对于左旋操作 $$leftRorate()$$ 同理，我们同样需要完成两件事：

- 将需要旋转的节点 $$cur$$ 的左子树变成 $$y$$ 的右子树。
- 将 $$y$$ 变成 $$cur$$ 的左儿子。

注意，此时 $$y$$ 是 $$cur$$ 的父节点，且 $$cur$$ 为 $$y$$ 的右儿子。

一种代码实现如下：

```cpp
// ... 
void leftRotate(SplayNode* &cur) {
    SplayNode* y = cur->F;
    cur->F = y->F;
    if (cur->F) {
        if (y == cur->F->L) cur->F->L = cur;
        else cur->F->R = cur;
    }
    if (cur->L) cur->L->F = y;
    y->R = cur->L;
    y->F = cur;
    cur->L = y;
}
```

这里的操作和右旋的情况是镜像的。

在伸展树中，将 $$x$$ 的左旋操作记为 $$Zag(x)$$ 。

完成了旋转操作之后，我们接着讨论伸展树的「伸展」。

## 5.2 伸展树的伸展操作

之前已经提到过：

> 「伸展」操作就是要将节点 $$x$$ 通过一系列的旋转移动到树根。

下面通过几个简单的示意图来分析一下这个过程。

### 5.2.1 情况一：x 的父节点 y 已经是树根

首先讨论最简单的情况：$$x$$ 的父节点 $$y$$ 已经是树根。

在这种情况下，$$x$$ 只需要再进行一次旋转就可以到达树根：

- 如果 $$x$$ 是左儿子，那么就进行一次「右旋」。
- 如果 $$x$$ 是右儿子，那么就进行一次「左旋」。

![splay_condition_1](https://tva1.sinaimg.cn/large/008i3skNgy1gwzbketuxfj30ez08zt8z.jpg)

上图为 $$x$$ 是左儿子的情况。$$x$$ 是右儿子的情况同理，只需要进行一次 $$Zag(x)$$ 即可。

### 5.2.2 情况二：x 的父节点 y 不是树根

情况二稍微复杂一些。如果 $$y$$ 还不是树根，那么我们就将 $$y$$ 的父节点记为 $$z$$ 。

接下来我们就需要将 $$x$$ 旋转到 $$z$$ 的位置。

操作方法根据 $$x$$ 和 $$y$$ 的位置不同要再分为 $$4$$ 种可能的情况：

1. **$$x$$ 和 $$y$$ 都是左儿子**

如果 $$x$$ 和 $$y$$ 都是左儿子，我们可以通过以下方法将 $$x$$ 移动到 $$z$$ 的位置：

![splay_condition_2_zig_zig](https://tva1.sinaimg.cn/large/008i3skNgy1gwzbw0cmhyj30sb0b30tj.jpg)

即先进行一次 $$Zig(y)$$ ，再进行一次 $$Zig(x)$$ 。我们可以将整个操作记为 $$Zig-Zig$$ 。

2. **$$x$$ 和 $$y$$ 都是右儿子**

如果 $$x$$ 和 $$y$$ 都是右儿子，我们可以通过以下方法将 $$x$$ 移动到 $$z$$ 的位置：

![splay_condition_2_zag_zag](https://tva1.sinaimg.cn/large/008i3skNgy1gwzcu6rcm3j30r90b3t9i.jpg)

即先进行一次 $$Zag(y)$$ ，再进行一次 $$Zag(x)$$ 。我们可以将整个操作记为 $$Zag-Zag$$ 。

3. **$$x$$ 是左儿子且 $$y$$ 是右儿子**

如果 $$x$$ 是左儿子且 $$y$$ 是右儿子，我们可以通过以下方法将 $$x$$ 移动到 $$z$$ 的位置：

![splay_condition_2_zig_zag](https://tva1.sinaimg.cn/large/008i3skNgy1gwzdfifmtvj30qn0b3q3q.jpg)

即先进行一次 $$Zig(x)$$ ，再进行一次 $$Zag(x)$$ 。我们可以将整个操作记为 $$Zig-Zag$$ 。

4. **$$x$$ 是右儿子且 $$y$$ 是左儿子**

如果 $$x$$ 是右儿子且 $$y$$ 是左儿子，我们可以通过以下方法将 $$x$$ 移动到 $$z$$ 的位置：

![splay_condition_2_zag_zig](https://tva1.sinaimg.cn/large/008i3skNgy1gwzdsu20yej30pg0b3dgm.jpg)

即先进行一次 $$Zag(x)$$ ，再进行一次 $$Zig(x)$$ 。我们可以将整个操作记为 $$Zag-Zig$$ 。

这样，我们就可以根据不同的情况选择不同的旋转方式，不断旋转直到 $$x$$ 成为树根为止。

接下来就可以根据以上内容，实现伸展树的 $$splay()$$ 方法：

```cpp
// ...
void splay(SplayNode* x, SplayNode* &root) {
    while (x->F) { // 循环直到 x 到达根节点 root
        SplayNode* y = x->F;
        SplayNode* z = y->F;
        if (!z) { // 如果 z 为空，即 y 就是根节点 root
            if (x == y->L) rightRotate(x); // 如果 x 是左儿子就右旋
            else leftRotate(x); // 如果 x 是右儿子就左旋
        } else { // 如果 y 还不是根节点 root
            if (x == y->L && y == z->L) { // x 和 y 都是左儿子
                rightRotate(y);
                rightRotate(x);
            } else if (x == y->L && y == z->R) { // x 是左儿子，y 是右儿子
                rightRotate(x);
                leftRotate(x);
            } else if (x == y->R && y == z->R) { // x 是右儿子，y 是左儿子
                leftRotate(y);
                leftRotate(x);
            } else { // x 和 y 都是右儿子
                leftRotate(x);
                rightRotate(x);
            }
        }
    }
    root = x; // 将根节点 root 更新为 x
}
```

如果你理解了上面的内容，就会发现 $$splay()$$ 操作的代码实现是何其的简洁优美，且非常符合直觉。需要注意的是，在将 $$x$$ 移动到根节点之后，要更新根节点 $$root$$ 的值为 $$x$$ 。

$$splay()$$ 操作是伸展树最核心的操作。其余的操作跟普通的 $$BST$$ 是相同的。

下面我们来看看伸展树的实际应用。

## 5.3 伸展树的实际操作

在本章的开头我们讲到，伸展树的核心思想是：

> 将操作频率高的节点尽量的移动到离树根更近的位置。

所以伸展树的核心用法是：

> 每次操作一个节点之后（哪怕只是查询），都手动将该节点 $$splay$$ 到根部。

如果这样进行操作，自然而然的，节点被操作的次数越多，就会离根节点 $$root$$ 越近。而离根节点越近，下一次操作的时间开销就越低。

为了更直观的理解伸展树的特性，下面我们通过一道算法题来作为例子：

---

[Luogu-P2234-HNOI2002-营业额统计](https://www.luogu.com.cn/problem/P2234)

**题目描述**

$$Tiger$$ 最近被公司升任为营业部经理，他上任后接受公司交给的第一项任务便是统计并分析公司成立以来的营业情况。

$$Tiger$$ 拿出了公司的账本，账本上记录了公司成立以来每天的营业额。分析营业情况是一项相当复杂的工作。由于节假日，大减价或者是其他情况的时候，营业额会出现一定的波动，当然一定的波动是能够接受的，但是在某些时候营业额突变得很高或是很低，这就证明公司此时的经营状况出现了问题。经济管理学上定义了一种最小波动值来衡量这种情况：当最小波动值越大时，就说明营业情况越不稳定。

而分析整个公司的从成立到现在营业情况是否稳定，只需要把每一天的最小波动值加起来就可以了。你的任务就是编写一个程序帮助 $$Tiger$$ 来计算这一个值。

我们定义，一天的最小波动值 = $$min\{|$$该天以前某一天的营业额$$-$$该天营业额$$|\}$$

特别地，第一天的最小波动值为第一天的营业额。

**输入格式**

第一行为正整数 $$n$$ ($$n\le32767$$)，表示该公司从成立一直到现在的天数。

接下来的 $$n$$ 行每行有一个整数 $$a_i$$ ($$|a_i|\le10^6$$)，表示第 $$i$$ 天公司的营业额，可能存在负数。

**输出格式**

输出一个正整数，即每一天最小波动值的和，保证结果小于 $$2^{31}$$ 。

**输入输出样例**

**输入**

```
6
5
1
2
5
4
6
```

**输出**

```
12
```

---

**解题思路**

题意非常简洁，朴素解法是：计算出每一天的最小波动值，再累加求和。由于每次计算第 $$i$$ 天的最小波动值，都需要遍历区间 $$[1, i-1]$$ ，时间复杂度为 $$O(N^2)$$ ，参考数据范围，是无法 $$AC$$ 本题的。

根据最小波动值的定义：

> 最小波动值 $$res(i) = min\{|$$该天以前某一天的营业额$$-$$该天营业额$$|\}$$

如果设 $$x_i$$ 为第 $$i$$ 天的营业额，$$pred(x_i)$$ 为区间 $$[1, i-1]$$ 中 $$x_i$$ 的「前驱」，$$succ(x_i)$$ 为区间 $$[1, i-1]$$ 中 $$x_i$$ 的「后继」，那么显然有：

> $$res(i)=min(|x_i-pred(x_i)|,\ |x_i-succ(x_i)|)$$ 

所以问题转化为怎样高效的求得 $$pred(x_i)$$ 和 $$succ(x_i)$$ 。这正好可以用 $$BST$$ 解决。

所以考虑将每一天的营业额都保存在一棵 $$Splay$$ 中。每次读入一个新的 $$x_i$$ 之后，先在 $$Splay$$ 中找到 $$pred(x_i)$$ 和 $$succ(x_i)$$ ，求出最小波动值并累加到答案，然后再将 $$x_i$$ 加入 $$Splay$$ 中。

这样在数据读入完毕之后就可以直接得到答案。每次求前驱和后继的时间为 $$O(log_2^N)$$ ，总时间复杂度为 $$O(Nlog_2^N)$$ 。可以 $$AC$$ 本题。

伸展树的定义和所需方法的代码实现如下：

```cpp
// 定义伸展树节点
struct SplayNode {
    int val;
    SplayNode *L, *R, *F;
    SplayNode(int x) : val(x), L(NULL), R(NULL), F(NULL) {}
};
// 右旋操作
void rightRotate(SplayNode* &cur) {
    SplayNode* y = cur->F;
    cur->F = y->F;
    if (cur->F) {
        if (y == cur->F->L) cur->F->L = cur;
        else cur->F->R = cur;
    }
    if (cur->R) cur->R->F = y;
    y->L = cur->R;
    y->F = cur;
    cur->R = y;
}
// 左旋操作
void leftRotate(SplayNode* &cur) {
    SplayNode* y = cur->F;
    cur->F = y->F;
    if (cur->F) {
        if (y == cur->F->L) cur->F->L = cur;
        else cur->F->R = cur;
    }
    if (cur->L) cur->L->F = y;
    y->R = cur->L;
    y->F = cur;
    cur->L = y;
}
// 伸展操作
void splay(SplayNode* x, SplayNode* &root) {
    while (x->F) {
        SplayNode* y = x->F;
        SplayNode* z = y->F;
        if (!z) {
            if (x == y->L) rightRotate(x);
            else leftRotate(x);
        } else {
            if (x == y->L && y == z->L) {
                rightRotate(y);
                rightRotate(x);
            } else if (x == y->L && y == z->R) {
                rightRotate(x);
                leftRotate(x);
            } else if (x == y->R && y == z->R) {
                leftRotate(y);
                leftRotate(x);
            } else {
                leftRotate(x);
                rightRotate(x);
            }
        }
    }
    root = x;
}
// 插入操作
SplayNode* insert(SplayNode* &cur, int x) {
    if (x <= cur->val) {
        if (!cur->L) {
            cur->L = new SplayNode(x);
            cur->L->F = cur;
            return cur->L;
        } else return insert(cur->L, x);
    } else {
        if (!cur->R) {
            cur->R = new SplayNode(x);
            cur->R->F = cur;
            return cur->R;
        } else return insert(cur->R, x);
    }
}
// 寻找 x 的前驱
SplayNode* findPred(SplayNode* cur, int x) {
    if (x >= cur->val) {
        if (!cur->R) return cur;
        SplayNode* res = findPred(cur->R, x);
        if (!res) return cur;
        return res;
    } else {
        if (!cur->L) return NULL;
        return findPred(cur->L, x);
    }
}
// 寻找 x 的后继
SplayNode* findSucc(SplayNode* cur, int x) {
    if (x <= cur->val) {
        if (!cur->L) return cur;
        SplayNode* res = findSucc(cur->L, x);
        if (!res) return cur;
        return res;
    } else {
        if (!cur->R) return NULL;
        return findSucc(cur->R, x);
    }
}
```

主函数部分如下：

```cpp
int main() {
    ios::sync_with_stdio(0);
    int n, k;
    cin >> n >> k;
    SplayNode* root = new SplayNode(k); // 将第一天的营业额作为根节点
    int ans = k; // 第一天的最小波动值就是营业额，直接加入答案
    for (int i = 2; i <= n; i++) {
        cin >> k;
        SplayNode* pred = findPred(root, k); // 求 k 的前驱
        SplayNode* succ = findSucc(root, k); // 求 k 的后继
        if (!pred) ans += succ->val - k; // 计算最小波动值并累加到答案
        else if (!succ) ans += k - pred->val;
        else ans += min(k - pred->val, succ->val - k);
        SplayNode* x = insert(root, k); // 将 k 插入 Splay 中
        splay(x, root); // 每次插入一个新节点时，都将该节点 splay 到根部
    }
    cout << ans;
}
```

很遗憾的是，如果只是这样，只能得到 $$90$$ 分，有一个测试点会 $$TLE$$ ：

![Screen Shot 2021-12-02 at 2.45.24 PM](https://tva1.sinaimg.cn/large/008i3skNgy1gwzhk63wukj317g0o6tb1.jpg)

这是由于伸展树并非是时刻保持平衡的，换句话说，它在最坏情况下依然可能会退化成一条链。在算法竞赛中，很有可能存在这样刻意构造的数据。

那么如何解决呢？对于伸展树来说，如果 $$splay()$$ 操作执行的次数越多，整棵树的形态就会越平衡。在以上代码中，只在插入节点时进行伸展操作时不够的。每次执行完 $$findPred()$$ 和 $$findSucc()$$ 操作之后，都要再执行一次 $$splay()$$ 操作：

```cpp
// ...
SplayNode* pred = findPred(root, k);
if (pred) splay(pred, root); // 每次都将找到的前驱节点 splay 到根部
SplayNode* succ = findSucc(root, k);
if (succ) splay(succ, root); // 每次都将找到的后继节点 splay 到根部
// ...
```

这样修改之后，就可以 $$AC$$ 本题：

![Screen Shot 2021-12-02 at 2.52.03 PM](https://tva1.sinaimg.cn/large/008i3skNgy1gwzhr7vfh9j317g0o6jtk.jpg)

可以看到，之前 $$TLE$$ 的测试点现在只用了 $$10ms$$ 就跑完了。可见 $$splay()$$ 操作对于伸展树的性能影响之大。

## 5.4 伸展树的性能分析

由伸展树的设计理念可以直到，伸展树并非时刻保持平衡的，它并不是一棵平衡树。它的特点在于：

- 操作频繁的节点距离根节点很近，其时间复杂度接近一个常数。
- 对于不常操作的节点，其时间复杂度较高，最坏情况下会达到 $$O(N)$$ 。

可以证明，在大量随机数据下，伸展树操作的均摊时间复杂度为 $$O(log_2^N)$$ ，跟平衡树相当。但是对于操作频繁的数据来说，其性能异常的优秀。再加上其书写简单，所以伸展树不论是在算法竞赛中，还是在实际生产中都非常受欢迎。

从上面的例子可以很明显的看出，尽量多的进行 $$splay()$$ 操作，可以很大程度上避免最坏情况的出现。

## 5.5 伸展树的一些其他方法

因为伸展树的特殊性质，在伸展树中可以很简单的实现一些特殊的方法：如将一棵伸展树在节点 $$x$$ 处分断为两棵伸展树的 $$split()$$ 方法，已经将两棵伸展树 $$x$$ 和 $$y$$ 合并为一棵伸展树的 $$join()$$ 方法。在此基础之上，伸展树可以非常简单的实现删除节点 的 $$deleteNode()$$ 方法。

关于这一部分的内容，请参考 附录二 。