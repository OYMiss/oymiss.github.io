---
mathjax: true
title: 类与对象
date: 2017-12-24 20:40:26
tags: C++
categories: 编程
---

### 数据的抽象
类的基本思想是数据抽象和封装。

> 数据抽象是一种依赖于 接口和实现 分离的编程技术。

接口包括用户所能执行的操作；类的实现则包括类的数据成员，负责接口实现的函数体以及定义类所需的各种私有函数。

> 封装实现了类的接口与实现的分离。

封装后的类隐藏了它的实现细节，也就是无法通过接口去访问实现部分。

在C++ 中抽象数据类型(abstract data type) 的定义可以用 class 和 struct 来定义。但是他们两个区别仅仅在于 默认的数据访问权限。（struct 默认为 public, class 默认为 private）。

<!--more-->

### 成员函数

先定义一个 Book 类

```cpp
struct Book {
    std::string name;
    std::string book_no;

    std::string isbn();
}

std::string Book::isbn() {
    return book_no;
};

```

其中 isbn 成员函数是在类的体内`声明`，在类的体外`定义`。Book 类本身是一个作用于，所以定义 isbn 的时候，要加上 `Book::` 表示是在 类作用域中。

成员函数通过一个名为 `this` 的额外的隐式参数来访问调用它的那个对象。

```cpp
// 类似于这样
std::string isbn(Book* this);
```

```cpp
class Book {
    std::string name;
    std::string book_no;
public:
    std::string isbn() const {
        return book_no;
    }
};
```

这里的 isbn 在类的体内声明并且定义，并且加上了一个 `const` 修饰符。`const` 修改了隐式 this 指针的类型。所以成员函数中不能修改数据成员。

这个`const` 也很重要，比如在使用标准库中的 `set`时应该这么定义，

```cpp
struct Node {
    int v;
    // 重载 < 运算是必须的。因为 set 内部实现是红黑树，要用到 <
    // 第二个 const 也是必须的。
    bool operator < (const Node &rhs) const {
        return v < rhs.v;
    }
};

// 创建一个 set 类型的 数据。
set<Node> data;
```

### 构造函数

注意：C++编译器`只有`在用户没有定义`任何`构造函数的时候，才默认生成一个无参的构造函数。

```cpp
class Data {
    int id;
public:
    // C++ 11 中表示使用默认的构造函数。
    Data() = default;

    // 初始值列表
    Data(int id) : id(id * id) {
        cout << "new data" << endl;
    }

    // 拷贝构造函数
    Date(const Data &data) {
        this->id = data.id;
    }
};
```

下面主要对拷贝构造函数进行讨论：

```cpp
// 当调用 test1 时，形参的初始化需要调用拷贝构造函数
void test1(Data d);

// 调用 test2 的过程
// 1. 先传递引用给 d。     --- 不调用拷贝构造函数
// 2. 将 d 的值拷贝给 tmp。--- 调用拷贝构造函数
// 3. 将 tmp 作为返回值返回过去。--- 拷贝构造函数?
// 3. 对于第三步（tmp很特殊） C++ 11 做了优化（RVO）不会进行拷贝。
// Return Value Optimization (RVO)

Data test2(const Data &d) {
    Data tmp = d;
    return tmp;
}

// 毫无疑问 test3 在第三步也是会拷贝的。
// 有兴趣的可以去找关于RVO的资料，详细看看。
Data test3(const Data &d) {
    Data *tmp = new Data(d);
    return *tmp;
}
```

### 隐式类类型转换

```cpp
#include <iostream>
using namespace std;

struct Data {
    string val;
    Data(string v) : val(v) {}

    Data (istream &in) {
        in >> val;
    }
};

void test(Data d) {
    cout << d.val << endl;
}

int main() {
    // 下面 注释掉的是正常的调用构造函数去实例化对象。
    // 大家都知道就不多说。
    // Data d1(cin);
    // Data d2("12");
    // test(d1); test(d2);

    // 这里传入的参数是 istream
    // 而不是 test 的形式参数 Data
    test(cin);

    // 这里传入的参数是 string
    test((string)"abcd");
    // 下面的会报错
    // 因为 要隐式 "abcd" -> string -> Data
    // 要使用两种隐式转换才能转换。所以不行
    // test("abcd");
}
```

有时候并不需要这种隐式转换，这时候应该在类的构造函数前面加上 `explicit` 修饰符。（只对一个参数的构造函数有效）

### 聚合类

聚合类的定义：
* 所有成员都是 public
* 没有定义任何构造函数
* 没有类内初始值
* 没有基类，也没有 virtual 函数。

比如下面这个：

```cpp
// 这是一个非常常用的邻接表Edge 的结构体
struct Edge {
    int v, next;
    double d;
};

void test() {
    // 可以这样子进行初始化。
    // 初始值的顺序应该和声明的顺序一致。
    Edge e = {1, 3, 2.2};
}
```

### 友元函数
类可以允许其他类或者函数访问他的非公有数据成员，可以在这个类的体内 加上一个用 `friend` 修饰的类或者函数的声明。

```cpp
class Test;

class Num {
    // 添加有 friend 的声明。
    friend class Test;
    friend int getNum(Num &num);
    // 友元函数可以在类的内部定义。
    // 但是外部也要有函数的声明。
    /* friend int getNum(Num &num) {
        return num.val;
    }; */

    int val;
public:
    Num(int val) : val(val) {}
};

// 友元类定义
class Test {
public:
    int getNum(Num &num) {
        return num.val;
    }
};

// 也可以在外部
// 注意：友元函数getNum 并不属于Num类。
int getNum(Num &num) {
    return num.val;
};

#include <iostream>

using namespace std;

int main() {
    Num num(1);
    Test t;
    cout << t.getNum(num) << endl;
    cout << getNum(num) << endl;
}
```

### 静态成员
可以在成员前面加上 `static` 关键字，来声明静态成员。

要注意几点：

* 静态成员函数只能访问静态成员。
* 静态成员的定义一般放在类定义外面。

用来记录类被实例化几次的示例已经很古老而且没有新意。所以这里举出另一个关于静态成员的应用：

单例设计模式。（私有化构造函数，使得类实例化出的对象只有一个）

```cpp
#include <iostream>
using namespace std;

struct Node {
private:
    Node(int n) : val(n) {}

public:
    int val;
    // 声明的时候不进行初始化
    // static Node * one = nullptr; // 错误
    static Node * one;
    static Node * instance() {
        if (one == nullptr) {
            one = new Node(0);
        }
        return one;
    }
};

// 一般要在全局这里进行定义，初始化。
Node* Node::one = nullptr;

int main() {
    Node *p1 = Node::instance();
    Node *p2 = Node::instance();
    Node *p3 = Node::instance();
    cout << (p1 == p2) << endl;
    cout << (p2 == p3) << endl;
    // 结果 p1, p2 ,p3 都相等
}
```

[1] Lippman S B. C++ Primer[M]. Pearson Education India, 2005.

[2] [关于RVO和std move的一篇文章](https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/RVO_V_S_std_move?lang=zh)