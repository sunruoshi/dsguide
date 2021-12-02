# Extra II：伸展树中的常用方法

> 阅读本章内容之前请先阅读第五章。

伸展树除了旋转和伸展操作 $$splay()$$ 之外，其余方法跟普通 $$BST$$ 并无区别：

```cpp
struct SplayNode {
    int val;
    SplayNode *L, *R, *F;
    SplayNode(int x) : val(x), L(NULL), R(NULL), F(NULL) {}
};

void rightRotate(SplayNode* &cur) {
    // ...
}

void leftRotate(SplayNode* &cur) {
    // ...
}

void splay(SplayNode* x, SplayNode* &root) {
    // ...
}
// 插入新节点
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
// 查找节点
SplayNode* findNode(int x, SplayNode* cur) {
    if (x == cur->val) return cur;
    if (x < cur->val) {
        if (!cur->L) return NULL;
        return findNode(x, cur->L);
    } else {
        if (!cur->R) return NULL;
        return findNode(x, cur->R);
    }
}
// 查找最大值
SplayNode* findMax(SplayNode* cur) {
    if (!cur->R) return cur;
    return findMax(cur->R);
}
// 查找最小值
SplayNode* findMin(SplayNode* cur) {
    if (!cur->L) return cur;
    return findMin(cur->L);
}
// 查找前驱
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
// 查找后继
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

比较特别的是，由于伸展树的特性，我们可以很简单的实现将一棵伸展树在节点 $$x$$ 处分断为两棵伸展树的 $$split()$$ 方法，以及将两棵伸展树 $$x$$ 和 $$y$$ 合并为一棵伸展树的 $$join()$$ 方法。

其中 $$split()$$ 方法的代码实现如下：

```cpp
void split(int k, SplayNode* &root, SplayNode* &x, SplayNode* &y) {
    SplayNode* p = findNode(k, root);
    splay(p, root);
    x = p->L;
    y = p->R;
}
```

$$split()$$ 方法可以将伸展树 $$root$$ 从节点 $$k$$ 处（$$k$$ 为节点的值域）一分为二变成两棵伸展树 $$x$$ 和 $$y$$ 。

其实现逻辑非常简单：

1. 先找到数据域为 $$k$$ 的节点 $$p$$ 。
2. 将 $$p$$ 节点通过 $$splay()$$ 方法移动到根节点。
3. 最后直接将 $$p$$ 的左右子树分别分给 $$x$$ 和 $$y$$ 即可。

而 $$join()$$ 方法的代码实现如下：

```cpp
SplayNode* join(SplayNode* x, SplayNode* y) {
    SplayNode* p = findMax(x);
    splay(p, x);
    p->R = y;
    y->F = p;
    return p;
}
```

$$join()$$ 方法可以将两棵伸展树 $$x$$ 和 $$y$$ 合并为一棵伸展树，并返回根节点 $$p$$ 。

其实现逻辑同样非常简单：

1. 找到 $$x$$ 中的最大节点 $$p$$ 。
2. 将 $$p$$ 节点 $$splay$$ 到根节点。此时，$$x$$ 中所有的其他节点都位于 $$p$$ 的左子树。
3. 将 $$y$$ 变为 $$p$$ 的右子树。
4. 最终 $$p$$ 就是合并后的伸展树的根节点。

注意，**这需要保证 $$y$$ 中所有节点的数据域均大于 $$x$$ 中的所有节点**。

通过这个 $$join()$$ 方法，我们可以非常简单的实现伸展树中删除节点的 $$deleteNode()$$ 方法：

```cpp
SplayNode* deleteNode(SplayNode* x, SplayNode* &root) {
    splay(x, root);
    SplayNode *a = x->L, *b = x->R;
    delete x;
    if (a) a->F = NULL;
    if (b) b->F = NULL;
    if (!a) return b;
    if (!b) return a;
    return join(a, b);
}
```

其实现逻辑如下：

1. 先把待删除的 $$x$$ 节点 $$splay()$$ 到根部。
2. 用两个临时变量 $$a$$ 和 $$b$$ 分别保存 $$x$$ 的左右子树。
3. 然后直接删除 $$x$$ 。
4. 接下来直接进行一个判断：
   - 如果 $$a$$ 为空树，就直接返回 $$b$$ ；
   - 如果 $$b$$ 为空树，就直接返回 $$a$$ ；
   - 如果都不空，就返回 $$join(a, b)$$ 。

注意，**最后还需要将 $$deleteNode()$$ 的返回值作为新的 $$root$$ 节点**。

这种节点的删除方法非常的巧妙，充分的利用了伸展树的性质。