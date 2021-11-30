# Extra I：$$BST$$ 的一些有用的域

## Ex-1.1 子树大小域 $$size$$

用来维护子树大小的 $$size$$ 域是在各种树形结构中都会经常使用到的。它可以帮我们高效的解决一些问题。

子树的大小即子树中包含节点的个数（包含根节点），所以在二叉树中，叶子节点的 $$size=1$$ ，而根节点 $$root$$ 的 $$size=N$$ ，即树中的节点总数。

$$size$$ 域在 $$BST$$ 中最经典的一个应用就是求解节点 $$x$$ 的「排名」$$rank(x)$$ 。

沿用第四章中的例子：

$$A$$ 班所有学生的名字已经保存在了一棵 $$BST$$ 中，即 $$A$$ 班的花名册现在已经按照名字的字典序升序建好了。现在我们想要知道 $$David$$ 是花名册中的第几个人。

这个问题实际上就是要求解 $$David$$ 在 $$BST$$ 中的排名。

首先，我们先给节点增加一个 $$size$$ 域：

```cpp
struct Node {
    string name;
    int size;
    Node *L, *R;
    Node(string s) : name(s), size(1), L(NULL), R(NULL) {}
};
```

初始时所有节点的 $$size$$ 值都是 $$1$$ 。

由于节点的插入和删除都会改变子树的大小，所以相应的，我们需要修改 $$insert()$$ 和 $$deleteNode()$$ 方法，在节点的数量变化时动态的维护节点的 $$size$$ 域。

插入的情况很简单：

在寻找插入位置的过程中，每访问到一个非空节点，都把该节点的 $$size$$ 值加 $$1$$ 就可以了：

```cpp
Node* insert(Node* &cur, string name) {
    if (cur == NULL) {
        cur = new Node(name);
        return cur;
    }
    cur->size++;
    if (name <= cur->name) return insert(cur->L, name);
    else return insert(cur->R, name);
}
```

删除的情况也类似，找到删除的位置之后，要把路径上所有节点的 $$size$$ 值都减 $$1$$ 。

```cpp
void deleteNode(Node* &cur, string x) {
    if (cur == NULL) return;
    cur->size--;
    if (x == cur->name) {
        if (cur->L == NULL && cur->R == NULL) {
            cur = NULL;
        } else if (cur->L != NULL) {
            string pred = findMax(cur->L)->name;
            cur->name = pred;
            deleteNode(cur->L, pred);
        } else if (cur->R != NULL) {
            string succ = findMin(cur->R)->name;
            cur->name = succ;
            deleteNode(cur->R, succ);
        }
    } else if (x < cur->name) {
        deleteNode(cur->L, x);
    } else if (x > cur->name) {
        deleteNode(cur->R, x);
    }
}
```

现在考虑如何求解 $$rank(x)$$ 。

很容易想到：

> $$rank(x)$$ 就是 $$BST$$ 中数据域小于 $$x$$ 的节点的个数再加 $$1$$ 。

为了方便描述，我们用 $$Val(x)$$ 代表节点 $$x$$ 的数据域：

从根节点 $$root$$ 开始：

- 如果 $$Val(x)<Val(root)$$ ，那么说明 $$x$$ 位于左子树，那么整个右子树加上根节点都大于 $$Val(x)$$ 。所以不需要把他们的大小累加到 $$rank(x)$$ 中，继续往左子树递归即可。

- 如果 $$Val(x)>Val(root)$$ ，那么说明 $$x$$ 位于右子树，那么整个左子树加上根节点都小于 $$Val(x)$$ 。所以这个时候需要将整个左子树的大小，加上根节点，都累加到 $$rank(x)$$ 中，并且继续向右子树递归。
- 如果 $$Val(x)==Val(root)$$ ，即找到了 $$x$$ ，就再加上 $$x$$ 自己左子树的大小再加 $$1$$ 。因为 $$x$$ 自己的左子树中的所有节点都一定小于 $$Val(x)$$ 。
- 如果递归到了叶子节点，就直接返回 $$1$$ 。

一种代码实现如下：

```cpp
int getRank(Node* cur, string x) {
    if (!cur->L && !cur->R) return 1;
    if (cur->L && x < cur->name) return getRank(cur->L, x);
    if (cur->R && x > cur->name) return getRank(cur->R, x) + cur->L->size + 1;
    return cur->L->size + 1;
}
```

如果使用第四章的例子来测试，可以打印出所有学生的排名如下：

```
Harris Rank: 8
Ford Rank: 6
Celine Rank: 3
Jack Rank: 10
Elizabeth Rank: 5
Bob Rank: 2
Gary Rank: 7
Alice Rank: 1
Isabelle Rank: 9
David Rank: 4
```

除了求解排名之外，$$size$$ 域还有很多重要的应用。例如一种特殊的平衡树 $$SBT$$ ，就是通过节点的 $$size$$ 域来实现自平衡的。

## Ex-1.2 父节点域 $$F$$

有些时候，我们可以给 $$BST$$ 添加一个额外的指向「父节点」的指针域 $$F$$ 。这可以帮助我们简单的实现一些特殊的方法。

首先，给节点添加一个指针域 $$F$$ ：

```cpp
struct Node {
    string name;
    Node *F, *L, *R;
    Node(string s) : name(s), F(NULL), L(NULL), R(NULL) {}
};
```

由于现在新增了一个指针域，所以 $$insert()$$ 方法也要做相应的修改：

```cpp
Node* insert(Node* &cur, string name) {
    if (name <= cur->name) {
        if (cur->L == NULL) {
            cur->L = new Node(name);
            cur->L->F = cur;
            return cur->L;
        } else return insert(cur->L, name);
    } else {
        if (cur->R == NULL) {
            cur->R = new Node(name);
            cur->R->F = cur;
            return cur->R;
        } else return insert(cur->R, name);
    }
}
```



