# Chapter II：链表篇

> 阅读本章内容需要以下前置知识：
>
> C++ 基础语法
>
> 指针与结构体指针

链表是一种十分基础而重要的「链式结构」容器。熟练的掌握链表是学习很多高级数据结构的基础。

在讨论链表之前，我们首先需要弄明白两个问题：

1. 链表是为了解决什么问题而被设计出来的。
2. 链表是如何解决这个问题的。

## 2.1 链表解决了什么问题

回到我们第一章中的例子：

我们已经使用了一个可变数组 $$vector$$  来维护 $$A$$ 班所有同学的名字，并通过 $$insert()$$ 和 $$erase()$$ 方法实现了成员的添加和删除。暂时我们没有发现什么不妥，可变数组 $$vector$$ 很好的满足的我们的需求。

但是随着 $$A$$ 班的不断壮大，成员不断的增多，我们慢慢发现，插入和删除的操作变得「越来越慢」。最后甚至慢到完全无法接受的程度。

这是由于如果我们要将数组中的下标 $$p$$ 位置的成员删除，仅仅删除数据是「不够的」。因为如果仅仅删除数据，则会在下标 $$p$$ 处留下一个「空位」。这不仅仅是对于宝贵的内存的浪费，而且会在数组的遍历过程中产生错误。所以我们除了要将下标 $$p$$ 处的数据删除之外，还要让从下标 $$p + 1$$ 处开始，直到数组末尾的所有元素，都「往前移动一位」，以填补 $$p$$ 位置的「空缺」。

可想而知，最坏情况下，如果我们要删除 $$A$$ 中的「第一个成员」，就需要「将之后的所有成员全部移动一次」。在数组成员很多的时候，这个开销是十分巨大，并且没有意义的。

插入的时候同理。如果我们需要在数组中插入一个成员，那么首先必须让插入位置之后的所有成员都先向后移动一个位置，以腾出空间给新成员。

所以在数组成员很多，并且插入/删除操作频繁的时候，$$vector$$ 已经不再能够满足我们的需求。我们需要一个新的容器来解决这个问题，而这个新的容器就是链表。

## 2.2 链表如何解决这个问题

在上一章中我们已经讨论过，数组就像一个储物柜，每个「格子」都编好了号码，并且格子之间的相对位置是「固定的」，这就是造成以上问题的核心原因。那么如果需要解决这个问题，我们就首先需要改变这些格子的「连接方式」。

我们考虑给每一个成员都添加一个新的「域」，这个「域」中保存一个额外的信息，即「当前成员的下一个成员的位置」。这样，集合中的每一个成员，都可以通过这个「域」直接找到在它之后的下一个成员。

然后选择一个格子，这个格子可以不放东西，但是它的「位置固定」，作为所有成员的「头部」。

这样，我们的「储物柜」就可以被改造成另一种结构：所有格子不再按照编号排列在一起，而是像一条项链，一个串着一个：

![Screen Shot 2021-11-25 at 12.18.14 PM](https://tva1.sinaimg.cn/large/008i3skNly1gwr9zbfueqj31i208yaao.jpg)

这种数据存储方式，就叫做链表。由于每个成员的「指针域」都单向的指向下一个成员，所以这种链表也称为「单向链表」，简称为「单链表」。

这个经过改造的「储物柜」，插入和删除成员将变得十分容易：

例如我想要删除 $$Bob$$，只需要将 $$Alice$$ 的「指针域」修改为直接指向 $$Celine$$ 即可。然后就可以释放掉 $$Bob$$ 使用的内存了。整个过程中不再需要操作不相干的成员。

![Screen Shot 2021-11-25 at 12.56.52 PM](https://tva1.sinaimg.cn/large/008i3skNly1gwrdojn39fj31i208ydgk.jpg)

而如果我想要在 $$Celine$$ 和 $$David$$ 之前插入一个新同学 $$Ford$$ ，只需要执行以下操作：

1. 将 $$Celine$$ 的「指针域」指向 $$Ford$$ ；
2. 将 $$Ford$$ 的「指针域」指向 $$David$$ ；

这样整个插入操作就完成了，同样的，这个过程中不需要再操作不相干的成员。

![Screen Shot 2021-11-25 at 2.26.24 PM](https://tva1.sinaimg.cn/large/008i3skNly1gwrdomfgvaj31i208ywf8.jpg)

## 2.3 使用结构体指针实现链表

在代码实现上，可以简单的使用结构体来定义链表的成员，其中每个成员都有一个「数据域」和一个「指针域」：

```cpp
#include <string>
struct Node {
    string name;  // 数据域，用于保存数据的原始值
    Node* next;   // 指针域，保存下一个成员的内存地址
    Node() {}     // 默认构造函数，用于构造 head 节点
    Node(string s) : name(s), next(NULL) {}
};
```

然后就可以构造一个 $$head$$ 节点，用来标记整个链表的「头部」：

```cpp
Node* head = new Node;
```

需要注意的是，$$head$$ 节点作为整个链表的头部，只起到一个标记作用，并非「链表中的第一个成员」。它没有数据域，初始时指针域也处于未初始化的状态。

### 2.3.1 向链表中添加成员

现在，我们可以准备向链表中加入新成员了。由于链表也是一种线性的数据结构，在添加新成员时，默认是添加在链表的尾部。所以在添加新成员之前，我们还需要一个辅助的指针 $$rear$$，指向当前链表中的「最后一个成员」，并且在初始状态下，$$rear$$ 和 $$head$$ 相同：

```cpp
Node* rear = head;
```

现在我们准备向链表中添加第一个成员 $$Alice$$ 。过程如下：

首先构造一个新节点 $$p$$，其「数据域」为字符串 $$Alice$$ ：

```cpp
Node* p = new Node("Alice");
```

然后，我们需要让 $$head$$ 的「指针域」指向这个新节点 $$p$$ 。注意，在这一步我们不要直接操作 $$head$$ 节点，而应该操作 $$rear$$ 指针：

```cpp
rear->next = p; // 将 p 设为 rear 的下一个节点
```

在操作之后，节点 $$p$$ 现在是链表中的最后一个节点，所以应该同步的更新 $$rear$$ 指针：

```cpp
rear = p; // 将 rear 更新为 p
```

这样，在加入第一个成员时，此时 $$rear$$ 就是 $$head$$，且它们都是指针，所以令 $$rear$$->$$next = p$$ 就是令 $$head$$->$$next = p$$ 。而在此之后，$$rear$$ 已经指向了 $$p$$，下一个新加入的成员就自然会连接到 $$p$$ 的后面。

现在可以尝试将原本保存在数组中的数据，通过循环加入到一个链表中：

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Node {
    string name;
    Node* next;
    Node() {}
    Node(string s) : name(s), next(NULL) {}
};

int main() {
    string a[5] = {"Alice", "Bob", "Celine", "David", "Elizabeth"};
    Node* head = new Node;
    Node* rear = head;
    Node *cur, *p;
    for (int i = 0; i < 5; i++) {
        p = new Node(a[i]);
        rear->next = p;
        rear = p;
    }
    cur = head;
    while (cur->next) {
        cout << cur->next->name << endl;
        cur = cur->next;
    }
    return 0;
}
```

编译并运行以上代码，可以在终端中看到输出如下：

```
ALice
Bob
Celine
David
Elizabeth
```

### 2.3.2 删除链表中的成员

现在我们尝试将 $$Bob$$ 从链表中删除。按照之前的设计，想要删除 $$Bob$$ 需要做两件事：

1. 将 $$Alice$$ 的指针域指向 $$Bob$$ 的下一个成员，即 $$Celine$$ ；
2. 将 $$Bob$$ 的内存释放，彻底删除 $$Bob$$ 。

那么首先需要找到链表中 $$Bob$$ 「之前」的那个成员，在这个例子里，就是 $$Alice$$ ：

```cpp
cur = head;
while (cur->next && cur->next->name != "Bob") {
    cur = cur->next;
}
```

当上面的 $$while$$ 循环退出时，$$cur$$ 就应该指向 $$Alice$$ 。

接下来就可以执行上面的两个步骤了，代码实现如下（此时的 $$cur$$ 指向 $$Alice$$）：

```cpp
Node* tmp = cur->next;       // 先将指向 Bob 的指针保存下来 
cur->next = cur->next->next; // 将 Alice 的指针域指向 Bob 的下一个人，即 Celine
delete(tmp);                 // 删除 Bob
```

为了方便起见，我们将打印整个链表的代码封装到一个函数 $$print()$$ 中，然后测试一下删除 $$Bob$$ 的效果：

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Node {
    string name;
    Node* next;
    Node() {}
    Node(string s) : name(s), next(NULL) {}

    void print() {
        Node* cur = this;
        while (cur->next) {
            cout << cur->next->name << endl;
            cur = cur->next;
        }
    }
};

int main() {
    string a[5] = {"Alice", "Bob", "Celine", "David", "Elizabeth"};
    // 初始化
    Node* head = new Node;
    Node* rear = head;
    Node *cur, *p;
    for (int i = 0; i < 5; i++) {
        p = new Node(a[i]);
        rear->next = p;
        rear = p;
    }
    // 找到 Bob 之前的成员
    cur = head;
    while (cur->next && cur->next->name != "Bob") {
        cur = cur->next;
    }
    // 删除 Bob
    Node* tmp = cur->next;
    cur->next = cur->next->next;
    delete(tmp);
    // 打印结果
    head->print();
    return 0;
}
```

编译并运行以上代码，可以在终端中看到输出如下：

```
Alice
Celine
David
Elizabeth
```

$$Bob$$ 已经被成功的删除了。

### 2.3.3 向链表中插入一个成员

现在我们尝试将新同学 $$Ford$$ 插入到 $$Celine$$ 和 $$David$$ 之间。首先，需要找到 $$Celine$$ 的位置：

```cpp
cur = head->next;
while (cur && cur->name != "Celine") {
    cur = cur->next;
}
```

当上面的 $$while$$ 循环退出时，$$cur$$ 就应该指向 $$Celine$$。

按照之前的设计，想要把 $$Ford$$ 插入到 $$Celine$$ 和 $$David$$ 之间，需要做两件事：

1. 将 $$Celine$$ 的指针域指向 $$Ford$$ ；
2. 将 $$Ford$$ 的指针域指向 $$David$$ ；

代码实现如下（此时的 $$cur$$ 指向 $$Celine$$）：

```cpp
p = new Node("Ford"); // 构造一个新节点
p->next = cur->next;  // 此时的 cur->next 指向 David
cur->next = p;        // 更新 cur->next 的值为新节点 p
```

之后再次打印整个链表，可以看到 $$Ford$$ 已经被插入到指定的位置上了。

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Node {
    string name;
    Node* next;
    Node() {}
    Node(string s) : name(s), next(NULL) {}

    void print() {
        Node* cur = this;
        while (cur->next) {
            cout << cur->next->name << endl;
            cur = cur->next;
        }
    }
};

int main() {
    string a[5] = {"Alice", "Bob", "Celine", "David", "Elizabeth"};
    Node* head = new Node;
    Node* rear = head;
    Node *cur, *p;
    for (int i = 0; i < 5; i++) {
        p = new Node(a[i]);
        rear->next = p;
        rear = p;
    }
    // 删除 Bob
    cur = head;
    while (cur->next && cur->next->name != "Bob") {
        cur = cur->next;
    }
    Node* tmp = cur->next;
    cur->next = cur->next->next;
    delete(tmp);
    // 把 Ford 插入到 Celine 和 David 之间
    cur = head->next;
    while (cur && cur->name != "Celine") {
        cur = cur->next;
    }
    p = new Node("Ford");
    p->next = cur->next;
    cur->next = p;
    // 打印结果
    head->print();
    return 0;
}
```

编译并运行以上代码，可以在终端中看到输出如下：

```
Alice
Celine
Ford
David
Elizabeth
```

> 1. 以上操作的代码实现并不唯一，但是操作的逻辑都是一样的。
> 2. 复杂的链式结构是非常容易写错的。希望读者能仔细琢磨示例代码中的细节，确保看懂了每一个环节，并且能够举一反三。

## 2.4 双向链表

对于单链表而言，其成员只包含一个「指针域」$$next$$，指向该成员的下一个成员。这就导致了单链表只能进行顺序遍历和查找，在很多时候，这并不是很方便。

所以我们可以在单链表的基础上进行一些「改进」，为其成员再添加一个「指针域」 $$prev$$，指向该成员的「前一个成员」。这样改进后的链表既支持顺序遍历和查找，也支持反向的遍历和查找。这样的链表称为「双向链表」。

现在，我们尝试将之前的单链表修改为一个双向链表。

比起单链表，双向链表的成员除了需要多一个指针域 $$prev$$ 之外，为了方便从后往前遍历，可以考虑在链表的尾部也添加一个空节点 $$tail$$ ，来标记链表的末端。

同 $$head$$ 节点类似，$$tail$$ 也是一个空节点，仅用来标记链表的结束，而并非「链表的最后一个元素」。

初始时，将 $$head$$ 和 $$tail$$ 直接相连：

```cpp
// ...
struct Node {
    string name;
    Node *prev, *next;
    Node() {}
    Node(string s) : name(s), prev(NULL), next(NULL) {}
};
// ...
Node* head = new Node, tail = new Node;
head->next = tail;
tail->prev = head;
```

由于现在我们有了一个「尾节点」$$tail$$，那么就不再需要 $$rear$$ 指针来标记链表的最后一个成员了。链表的最后一个成员总是可以通过 $$tail$$->$$prev$$ 访问到。

那么向链表中加入成员的过程如下，注意由于有两个指针域，连接两个成员时需要连两次：

```cpp
// ...
const string a[5] = {"Alice", "Bob", "Celine", "David", "Elizabeth"};
Node *p, *cur;
for (int i = 0; i < 5; i++) {
    p = new Node(a[i]);
    p->prev = tail->prev;
    p->next = tail;
    tail->prev->next = p;
    tail->prev = p;
}
```

从链表中删除成员的逻辑跟单链表是相同的，即先找到需要删除的成员，然后将它的前一个成员和后一个成员直接连起来。最后再释放掉该成员的内存。

现在来从链表中删除 $$Bob$$ ：

```cpp
// ...
cur = head;
while (cur->next && cur->next->name != "Bob") {
    cur = cur->next;
}
// cur 现在指向 Bob
Node *pre = cur->prev, *nex = cur->next;
pre->next = nex;
nex->prev = pre;
delete(cur);
```

插入的情况也类似。下面来将新成员 $$Ford$$ 插入到 $$Celine$$ 和 $$David$$ 之间：

```cpp
// ...
cur = head;
while (cur->next && cur->next->name != "David") {
    cur = cur->next;
}
// cur 现在指向 Celine，cur->next 指向 David
p = new Node("Ford");
p->prev = cur;
p->next = cur->next;
cur->next->prev = p;
cur->next = p;
```

在连接两个成员时需要特别注意「顺序」。在上面的代码中，$$cur$$->$$next$$ 必须最后被更新。因为在此之前我们需要指向 $$David$$ 的指针，而一旦 $$cur$$->$$next$$ 被更新为 $$p$$ ，我们就失去了 $$David$$ 的地址了。所以在更新 $$cur$$->$$next$$ 之前，必须将跟 $$David$$ 相关的连接全部完成。

比较好的一种避免犯错的方法是，使用两个临时变量将 $$Celine$$ 和 $$David$$ 的地址都保存下来。之后再进行连接的时候，就不需要在意连接的顺序了。

## 2.5 链表的特性

从前面的内容可以看出，链表这种数据结构的「优势」在于插入和删除操作速度很快（时间复杂度为常数），但是由于链表中的成员之间仅通过「指针域」相连，而不像数组那样每个成员都有一个确定的「位置」。对于数组而言，可以通过「下标」直接找到相应的成员，但是对于链表来说，只有 $$head$$（或 $$tail$$）节点的位置是确定的，想要找到链表中的某个成员，只能从 $$head$$（或 $$tail$$）节点开始，顺着「指针域」不断往下找。

所以在实际应用中，如果对于数据的操作以「插入/删除」为主，「查找/修改」很少，那么就应该使用链表而非数组；反之，如果操作以「查找/修改」为主，而「插入/删除」很少，那么这个时候就不应该使用链表。充分利用数据结构的特性，能够极大的提高程序的性能。

在下一章我们会开始讨论一种更为复杂的链式结构：树形结构。
