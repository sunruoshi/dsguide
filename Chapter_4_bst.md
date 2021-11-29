# Chapter VI：二叉搜索树篇

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
3. 值 $$x$$ 的「前驱」：树中比 $$x$$ 小的最大值。
4. 值 $$x$$ 的「后继」：树中比 $$x$$ 大的最小值。

这对于解决我们上一章中提到的那个问题非常方便：如果想要找到树中「第二大」的节点，那么就只需要找到 $$max$$ 的「前驱」就好了。

下面我们来讨论如何构造一棵 $$BST$$ ，并将数据存放到 $$BST$$ 中。

## 4.2 二叉搜索树的构建

对于所有二叉树来说，树的构建过程就是不断向树中插入新节点的过程，$$BST$$ 也不例外。那么首先我们需要使用一个结构体来定义 $$BST$$ 中的节点。

对于二叉树的节点来说，除了数据域之外，都至少需要两个指针域，分别指向节点的左儿子和右儿子。沿用之前的例子，我们想将 $$A$$ 班的学生信息保存在一棵 $$BST$$ 中：

```cpp
#include <string>

const string names[5] = {"Alice", "Bob", "Celine", "David", "Elizabeth"};
const int ages[5] = {13, 15, 11, 16, 10};

struct Node {
    string name; // 数据域一：学生的名字
    int age;     // 数据域二：学生的年龄
    Node *L, *R; // 指针域，分别指向左儿子和右儿子，初始值都为 NULL
    Node(string s, int d) : name(s), age(d), L(NULL), R(NULL) {}
};
```

然后可以将第一个学生作为根节点 $$root$$ ：

```cpp
// ...
Node* root = new Node(names[0], ages[0]);
```

那么现在我们就需要一个 $$insert()$$ 方法，可以将后续的成员都一一插入到 $$BST$$ 中。

### 4.2.1 往 $$BST$$ 中插入节点

要向二叉树中插入节点，那么就要先找到一个可以插入新节点的「位置」，即一个指针域为 $$NULL$$ 的地方。

跟普通的二叉树不同，$$BST$$ 必须把比当前节点值小的节点插入到左子树，比当前节点值大的节点插入到右子树，那么根据这个原则，很容易实现一个 $$insert()$$ 方法：

```cpp
// ...
Node* insert(Node* &cur, string name, int age) {
    if (cur == NULL) {
        cur = new Node(name, age);
        return cur;
    }
    if (age <= cur->age) return insert(cur->L, name, age);
    else return insert(cur->R, name, age);
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
    cout << "Age: " << cur->age << ' ' << "Name: " << cur->name << endl;
    inOrder(cur->R);
}
// ...
for (int i = 1; i < 5; i++) {
    insert(root, names[i], ages[i]);
}
inOrder(root);
```

运行以上代码可以看到以下输出：

```
Age: 10 Name: Elizabeth
Age: 11 Name: Celine
Age: 13 Name: Alice
Age: 15 Name: Bob
Age: 16 Name: David
```

所有学生已经按照年龄升序排列了！

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
cout << "Max Age: " << Max->age << ' ' << "Name: " << Max->name << endl;
cout << "Min Age: " << Min->age << ' ' << "Name: " << Min->name << endl;
```

可以看到输出如下：

```
Max Age: 16 Name: David
Min Age: 10 Name: Elizabeth
```

很容易就找到了年龄最大和最小的同学。

### 4.2.3 找前驱/后继

之前提到过，$$BST$$ 中，$$x$$ 的前驱 $$prev(x)$$ 的定义是「比 $$x$$ 小的最大值」，后继 $$next(x)$$ 的定义是「比 $$x$$ 大的最小值」。

如果想要找到 $$prev(x)$$ ，过程如下：

从根节点 $$root$$ 开始往下找：

- 如果 $$x$$ 大于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在右子树，则返回 $$cur$$ ；
  - 否则则向右子树递归，返回一个结果 $$res$$ ；
    - 如果 $$res == NULL$$ ，则还是返回 $$cur$$ ；
    - 否则返回 $$res$$ 。
- 如果 $$x$$ 小于等于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在左子树，则返回 $$NULL$$ ；
  - 否则，则向左子树递归求解。

```cpp
// ...
Node* findPrev(Node* cur, int x) {
    if (x > cur->age) {
        if (cur->R == NULL) return cur;
        Node* res = findPrev(cur->R, x);
        if (res == NULL) return cur;
        return res;
    } else {
        if (cur->L == NULL) return NULL;
        else return findPrev(cur->L, x);
    }
}
```

下面测试一下，找到 $$A$$ 班学生中年龄「第二大」的。

首先找到所有学生中最大的年龄 $$maxAge$$ ：

```cpp
// ...
int maxAge = findMax(root)->age;
```

年龄「第二大」的，显然就是 $$maxAge$$ 的前驱：

```cpp
Node* pre = findPrev(root, maxAge);
cout << "Second greatest Age: " << pre->age << ' ' << "Name: " << pre->name << endl;
```

运行之后可以看到输出如下：

```
Second greatest Age: 15 Name: Bob
```

年龄第二大的人是 $$Bob$$ ，年龄为 $$15$$ 。

查找 $$next(x)$$ 的原理是类似的，过程如下：

从根节点 $$root$$ 开始往下找：

- 如果 $$x$$ 小于等于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在左子树，则返回 $$cur$$ ；
  - 否则则向左子树递归，返回一个结果 $$res$$ ；
    - 如果 $$res == NULL$$ ，则还是返回 $$cur$$ ；
    - 否则返回 $$res$$ 。
- 如果 $$x$$ 大于当前节点 $$cur$$ 的数据域，那么则需要判断：
  - 如果 $$cur$$ 不存在右子树，则返回 $$NULL$$ ；
  - 否则，则向右子树递归求解。

代码实现如下：

```cpp
// ...
Node* findNext(Node* cur, int x) {
    if (x <= cur->age) {
        if (cur->L == NULL) return cur;
        Node* res = findPrev(cur->L, x);
        if (res == NULL) return cur;
        return res;
    } else {
        if (cur->R == NULL) return NULL;
        else return findPrev(cur->R, x);
    }
}
```

同样，我们来尝试找出 $$A$$ 班中年龄「第二小」的。

首先找到所有学生中的最小年龄 $$minAge$$ ：

```cpp
// ...
int minAge = findMin(root)->age;
```

年龄「第二小」的，显然就是 $$minAge$$ 的后继：

```cpp
Node* nex = findNext(root, minAge);
cout << "Second least Age: " << nex->age << ' ' << "Name: " << nex->name << endl;
```

运行之后可以看到输出如下：

```
Second least Age: 13 Name: Alice
```

年龄第二小的人是 $$Alice$$ ，年龄为 $$13$$ 。

### 4.2.4 删除 $$BST$$ 中的节点

$$BST$$ 删除节点的操作就有些复杂了。因为和插入的时候一样，我们要保证 $$BST$$ 在删除了一个节点之后「仍然是一棵 $$BST$$」。

为了保证这一点，在 $$BST$$ 中删除节点 $$x$$ 通常有两种方法：

1. 找到 $$x$$ 的「前驱节点」$$prev(x)$$ ，用 $$prev(x)$$ 覆盖掉 $$x$$ ，然后删除 $$prev(x)$$ ；
2. 找到 $$x$$ 的「后继节点」$$next(x)$$ ，用 $$next(x)$$ 覆盖掉 $$x$$ ，然后删除 $$next(x)$$ ；

以上操作可能需要「递归」进行。如果递归到了一个「叶子节点」，那么就可以直接删除这个节点，因为删除「叶子节点」不会对其他节点造成影响了。

这两种做法都可以保证删除操作之后仍然是一棵 $$BST$$ 。

一种不太好但是可行的代码实现如下：

```cpp
// ...
void deleteNode(Node* &cur, Node* x) {
    if (cur == NULL) return;             // 如果节点不存在直接返回
    if (cur == x) {                      // 如果找到了节点 x
        if (cur->L == NULL && cur->R == NULL) {
            cur = NULL;                  // 如果是叶子节点就直接删除
        } else if (cur->L != NULL) {     // 如果有左子树
            Node* pre = findMax(cur->L); // 找到左子树中最大的节点，即前驱
            cur->name = pre->name;       // 用 pre 覆盖掉 cur
            cur->age = pre->age;
            deleteNode(cur->L, pre);     // 递归的在左子树中继续删除 pre
        } else if (cur->R != NULL) {     // 如果有右子树
            Node* nex = findMin(cur->R); // 找到右子树中的最小节点，即后继
            cur->name = nex->name;       // 用 nex 覆盖掉 cur
            cur->age = nex->age;
            deleteNode(cur->R, nex);     // 递归的在右子树中继续删除 nex
        }
    } else if (x->age <= cur->age) {
        deleteNode(cur->L, x);
    } else if (x->age > cur->age) {
        deleteNode(cur->R, x);
    }
}
```

当然，要删除节点 $$x$$ ，首先要找到节点 $$x$$ 。所以我们可以定义一个在 $$BST$$ 中查找节点的 $$findNode()$$ 方法。

对于当前的例子，我们希望通过姓名来查找相应的同学。由于 $$BST$$ 是按照「年龄」来构建，所以我们并不能根据姓名来判断该往左子树查找还是右子树查找，在这种情况下，就对左右子树都进行查找：

```cpp
// ...
Node* findNode(Node* cur, string s) {
    if (cur == NULL) return NULL;
    if (cur->name == s) return cur;
	Node* res = findNode(cur->L, s);
    if (res == NULL) return findNode(cur->R, s);
    else return res;
}
```

现在我们尝试删除掉 $$Bob$$ ，之后再中序遍历 $$BST$$ ：

```cpp
// ...
Node* x = findNode(root, "Bob");
deleteNode(root, x);
inOrder(root);
```

可以看到输出如下：

```
Age: 10 Name: Elizabeth
Age: 11 Name: Celine
Age: 13 Name: Alice
Age: 16 Name: David
```

$$Bob$$ 被成功的删除了，而 $$BST$$ 的性质依然得以保留。

以上代码主要是用来体现删除操作的思路，实际使用中，以上代码可以通过很多手段优化。

例如可以在找到 $$x$$ 的后继 $$next(x)$$ 之后，不进行递归，而是通过这样的手段直接删除该后继：

>  假设后继 $$next(x)$$ 的父节点是 $$F$$ ，则显然 $$next(x)$$ 是 $$F$$ 的左儿子。那么由于 $$next(x)$$ 「一定没有左子树」，便可以直接把 $$next(x)$$ 的右子树代替 $$next(x)$$ 成为 $$F$$ 的左子树，这样就删去了节点 $$next(x)$$ 。前驱同理。

为了方便操作，这需要在 $$BST$$ 的节点定义中增加一个指针域，指向父节点的地址。

但是有一点需要注意，总是优先删除前驱（或后继）容易导致 $$BST$$ 的左右子树高度极度不平衡，使得 $$BST$$ 退化成一条链，这样会大大降低 $$BST$$ 的性能。在之后的内容中我们会继续讨论这个问题。

## 4.3 二叉搜索树的性能

