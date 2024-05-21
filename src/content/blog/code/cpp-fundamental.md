---
author: Chongzhuo Yang
pubDatetime: 2017-12-23T03:30:26.816Z
# featured: false
tags:
  - code
mathjax: true
title: C++ 基础
slug: "cpp-fundamental"
description: C++ 中基础关键字和特性。
---

## new delete

在C++ 中可以使用关键词 new 和 delete 进行动态内存的管理。

```cpp
// 几种方法都可以
int *p1 = new int(3);
int *p2 = new int[2];
int *p3 = new int;
int *points = new Point[4];
```

```cpp
// 对于对象数组来说
delete points // 代表用来释放内存，且只用来释放points指向的内存。
delete[] points // 逐一调用数组中每个对象的析构函数。

// 但对于内置类型来说，下面两个效果是一样的。
delete p2
delete[] p2
```

## 引用

### 引用作为函数形参

联想到 c 语言的swap 函数

<!--more-->

```c
void swap(int a, int b) {
    int t = a;
    a = b;
    b = t;
}

void main() {
    int a = 2, b = 3;
    swap(a, b);
}

```

这个函数是不会真正交换 main 函数中的变量 a, b的值的。如果要实现交换值就要使用指针。

```c
void swap_x(int *a, int *b) {
    int t = *a;
    *a = *b;
    *b = a;
}

void main() {
    int a = 2, b = 3;
    swap(&a, &b);
}
```

但是也发现了仅仅实现这么简单的功能也要使用指针，太麻烦了。
在C++ 中利用引用可以解决这个问题。

```cpp
void swap(int &a, int &b) {
    int t = a;
    a = b;
    b = t;
}

int main() {
    int a = 2, b = 3;
    swap(a, b);
}
```

这就是引用的最简单的应用。

#### 引用作为函数返回值

当函数返回值是引用时，函数也就能够作为左值。
具体看代码：

```cpp
const int maxn = 1000;
int a[maxn]; // 为了简化代码直接使用了全局变量

int& at(int i) {
    return a[i];
}

// 一个错误的示例
int at_wrong(int i) {
    return a[i];
}

void test() {
    at(0) = 1;
    at(1) = 6;
    at(2) = 4;
    at(3) = 8;
    at(4) = 8;

    // 下面一行代码会报错。因为 int 是不能作为左值的。
    at_wrong(5) = 1;
}
```

引用作为函数返回值这个 很重要。比如：自定义一个数据结构的时候，
需要提供接口，用来修改数据。比如 标准库中的 `vector`

下面举一个自定义的 类来说明这个特性的重要性。

```cpp
const unsigned maxn = 10000;

template<class T>
class my_vector {
    T a[maxn];
    unsigned cur = 0;
public:
    void add(T item) {
        a[cur++] = item;
    }

    T& operator [] (int i) {
        return a[i];
    }
};

int main() {
    my_vector<int> a;
    a.add(1);
    a.add(6);
    a.add(4);
    a.add(8);
    a.add(8);
    a.add(3);

    a[6] = 1;
}

```

重载了 `[]` 运算符。提供和数组类似的数据访问方式。

## const 修饰符

### 对于指针的 const

对于指针的 const 有一些特殊的地方。下面做一些讨论。
注意：对于引用，并没有这种不同，两种定义的都是，底层 const。

```cpp
int a = 2;
int b = 3;

const int *p1 = &a;
// 底层（low-level） const
// 不允许指针改变变量的值
// *p1 = 2;
p1 = &b; // 允许

int * const p2 = &a;
// 顶层（top-level） const
// 不允许改变指针的值
// p2 = &b;
*p2 = 3; // 允许

const int * const p3 = &a;
// 都不允许
// p3 = &b;
// *p3 = 3;
```

### 对结构体（类）的引用

```cpp
struct Point {
    int x, y;
};

// 加上const 修饰符保护数据
void test(const Point &point) {
    point.x = 3; // 错误
}

// 重点：这种情况下会发生重载。
// 具体见下一节
void test(Point &point) {
    point.x = 3; // 正确
}
```

## 函数特性

### 内联函数

利用内联函数，相对于正常的函数来说，一个优点就是执行效率会有所提升。
将指定的函数体插入并取代每一处调用该函数的地方。这样会减少调用函数（我认为是 压栈出栈所消耗的时间）的时间吧。

注意：对递归函数的内联扩展可能引起部分编译器的无穷编译。

```cpp
int max(int x, int y) {
    return x > y ? x : y;
}

// 使用内联函数的 max 函数
inline max_x(int x, int y) {
    return x > y ? x : y;
}
```

### 函数重载

重载的条件：同名函数的形式参数（指参数的个数、类型或者顺序）不同。

利用一个函数名去执行不同的功能。

```cpp
// add 能够加 两个或者三个数。
// 形式参数个数不同
void add(int a, int b, int c);
void add(int a, int b);
```

请注意下面一种情况：

下面程序表明 `const Point &` 这个类型和 `Point &` 是不同的。

```cpp
struct Point {
    int x, y;
};

// 发生函数重载。const 关键字修饰
void show(const Point &point) {
    cout << "const: " << point.x << "," << point.y << endl;
}

// 调用后将 x 坐标变为 0
void show(Point &point) {
    cout << "not const: " << point.x << "," << point.y << endl;
    point.x = 0;
}

int main() {
    Point point1 = {1, 2};
    const Point point2 = {2, 3};
    show(point1);
    show(point1);
    show(point2);
}
```

程序运行的结果是。

```
not const: 1,2
not const: 0,2
const: 2,3
```

### 函数参数默认值

使用函数默认值，调用的时候就可以简化传参，有时候也避免了再写一个函数去重载。

```cpp
// 计算两个数，a ope b。ope 默认为 +
void compute(int a, int b, char ope = '+');

void compute(int a, int b, char ope) {
    ...
}
```

注意：函数默认值应该在声明的时候写上去。

具有默认值的应该在函数形参列表的最右边。

```cpp
void add(int a = 0, int b); // 这样子是错的
```

## 数据的抽象

类的基本思想是数据抽象和封装。

> 数据抽象是一种依赖于 接口和实现 分离的编程技术。

接口包括用户所能执行的操作；类的实现则包括类的数据成员，负责接口实现的函数体以及定义类所需的各种私有函数。

> 封装实现了类的接口与实现的分离。

封装后的类隐藏了它的实现细节，也就是无法通过接口去访问实现部分。

在C++ 中抽象数据类型(abstract data type) 的定义可以用 class 和 struct 来定义。但是他们两个区别仅仅在于 默认的数据访问权限。（struct 默认为 public, class 默认为 private）。

<!--more-->

## 成员函数

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

## 构造函数

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

## 隐式类类型转换

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

## 聚合类

聚合类的定义：

- 所有成员都是 public
- 没有定义任何构造函数
- 没有类内初始值
- 没有基类，也没有 virtual 函数。

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

## 友元函数

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

## 静态成员

可以在成员前面加上 `static` 关键字，来声明静态成员。

要注意几点：

- 静态成员函数只能访问静态成员。
- 静态成员的定义一般放在类定义外面。

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

## OOP：概述

面向对象程序设计的核心思想是：数据抽象，继承和动态绑定。

- 通过数据抽象，将类的接口和实现进行分离
- 通过继承，可以定义相似的类型并对其相似关系建模
- 使用动态绑定，可以在一定程度上忽略相似类型的区别。

### 派生类构造

注意：派生类构造函数只能初始化它的直接基类。而且必须要在初始化列表中调用基类的构造函数。

```cpp
struct Base {
    int i;
    Base(int i) : i(i) {}
};

struct Derived1 : public Base {
    int ii;
    Derived1(int i, int ii) :
        Base(i), ii(ii) {
    }
};

struct Derived2 : public Derived1 {
    int iii;
    Derived2(int i, int ii, int iii) :
        Derived1(i, ii), iii(iii) {
    }
};

int main() {
    Derived2 obj(1, 2, 3);
}
```

<!--more-->

### 虚函数

在C++ 中，基类必须区分两种函数：

- 一种是基类希望派生类进行重写（override）的函数。
- 一种是基类希望派生类直接继承而不要改变的函数。

对于希望派生类进行重写的函数，在函数定义前面加上 `virtual` 关键字。
如果基类把函数声明为，虚函数，那么该函数在派生类中`隐式` 的也是虚函数。

下面举例说明一下。

```cpp
#include <iostream>
using namespace std;
struct A {
    void test() {
        cout << "HelloA" << endl;
    }
};

struct B : public A {
    void test() {
        cout << "HelloB" << endl;
    }
};

struct C : public B {
    void test() {
        cout << "HelloC" << endl;
    }
};

int main() {
    A a;
    B b;
    C c;
    A *pb = &b;
    A *pc = &c;
    pb->test();
    pc->test();
}
```

输出结果是：

```
HelloB
HelloC
```

如果把`virtual` 关键字去下来，输出的结果就是

```
HelloA
HelloA
```

### 动态绑定

> 只有指针或者引用 能实现运行时多态。（重点）

可以将基类的指针或引用绑定到派生类对象上。例如：在上一节例子中，可以用 A& 指向一个 B对象。

```cpp
#include <iostream>
using namespace std;

struct Base {
    int v;
    virtual void print() {
        cout << "Base" << endl;
    }
};

struct Derived : public Base {
    int vv;
    void print() {
        cout << "Derived" << endl;
    }
};

void test(Base &p) {
    p.print();
}

// 对象不能实现运行时多态
void test_wrong(Base p) {
    p.print();
}

int main() {
    Derived obj;
    test(obj);
    test_wrong(obj);
}
```

输出结果是：

```
Derived
Base
```

静态类型是变量声明时的类型（比如 p 的静态类型是 Base）。动态类型表示的是`内存`中对象的类型。（运行时才能知道）
p的动态类型根据 test 传入的参数不同，动态类型也会改变。

注意：基类指针或者引用的静态类型可能与其动态类型不同。如果既不是指针也不是引用那么动态类型永远与静态类型一致。(上述例子中的test_wrong 函数输出的内容)

### 抽象基类

在声明纯虚函数的时候在后面 书写 `=0` 就可以把虚函数定义为 纯虚。
就可以在类定义的时候，不定义这个虚函数。（让派生类去定义）。

含有纯虚函数的类是抽象基类。（`不能` 实例化抽象基类）

```cpp
#include <iostream>
using namespace std;

struct Base {
    int i;
    virtual void pure_virtual_fun() = 0;
};

struct Derived : public Base {
    void pure_virtual_fun() {
        cout << "HelloWorld" << endl;
    }
};

int main() {
    // 不能定义抽象基类
    // Base b; // 错误

    Derived obj;
    Base *p = &obj;
    p->pure_virtual_fun();
}
```

## 一个应用

下面是一个简单工厂模式。是面向对象编程中的一种重要设计模式。
有兴趣的可以把这个程序改成抽象工厂模式。

```cpp
#include <iostream>
using namespace std;

class Book {
protected:
    string name;
    string isbn;
public:
    virtual void show() = 0;
};

class AdventureBook : public Book {
public:
    void show() {
        cout << "this is a adventure book." << endl;
    }
};

class ScienceBook : public Book {
public:
    void show() {
        cout << "this is a science book." << endl;
    }
};

enum BookType {
    ADVENTURE,
    SCIENCE
};

class Factory {
public:
    Book * create(BookType type) {
        switch (type) {
            case BookType::ADVENTURE :
                return new AdventureBook();
            case BookType::SCIENCE :
                return new ScienceBook();
            default:
                return nullptr;
        }
    }
};

int main() {
    Factory factory;
    Book *book1 = factory.create(BookType::ADVENTURE);
    Book *book2 = factory.create(BookType::SCIENCE);

    book1->show();
    book2->show();
}

```

[1] Lippman S B. C++ Primer[M]. Pearson Education India, 2005.

[2] [关于RVO和std move的一篇文章](https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/RVO_V_S_std_move?lang=zh)

[3] [new_and_delete](http://www.cnblogs.com/hazir/p/new_and_delete.html)
