---
mathjax: true
title: 面向对象程序设计
date: 2017-12-25 20:40:55
tags: C++
categories: 编程
---

### OOP：概述
面向对象程序设计的核心思想是：数据抽象，继承和动态绑定。

* 通过数据抽象，将类的接口和实现进行分离
* 通过继承，可以定义相似的类型并对其相似关系建模
* 使用动态绑定，可以在一定程度上忽略相似类型的区别。

#### 派生类构造
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

#### 虚函数
在C++ 中，基类必须区分两种函数：

* 一种是基类希望派生类进行重写（override）的函数。
* 一种是基类希望派生类直接继承而不要改变的函数。

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
#### 动态绑定

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

#### 抽象基类
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

### 一个应用
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
