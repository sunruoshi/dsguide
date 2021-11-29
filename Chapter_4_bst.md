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

1. $$max$$：树中值最大的节点。
2. $$min$$ ：树中值最小的节点。
3. 节点 $$x$$ 的「前驱」：树中值比节点 $$x$$ 小的最大节点。
4. 节点 $$x$$ 的「后继」：树中值比节点 $$x$$ 大的最小节点。

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

之前提到过，$$BST$$ 中，节点 $$x$$ 的前驱节点 $$prev(x)$$ 的定义是「比 $$x$$ 小的最大节点」，后继节点 $$next(x)$$ 的定义是「比 $$x$$ 大的最小节点」。