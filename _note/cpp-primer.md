# C++ Primer 笔记

## 基础知识

引用不是对象，不存在引用的引用。

```cpp
int *p, a;   // p 是 int* 。 a 是 int
int *&r = p; // 从右向左
```

### const 限定符

默认情况下，const 对象仅在文件内有效。在多个文件共享 const 对象，必须在变量定义时添加 extern 关键字。

#### 对 const 对象的引用、指针

```cpp
const int ci = 1024;
const int &r1 = ci;
int vi = 1024;
const int &r2 = vi; // 允许将 const int& 绑定到普通 int 对象上
const int pi = 1024;
```

绑定的特殊情况：

```cpp
int main() {
    int i = 42;
    int &r1 = i;
    const int &r2 = i;
    r1 = 0;
    cout << r2 << endl;
    double dval = 3.14;
    const int &cri = dval; // 正确，cri 绑定临时量
    // int &ri = dval; // 错误
    dval = 10.0;
    cout << cri << endl;
}
```

上面程序运行结果是 0，3。

#### const 指针

```cpp
int errNumb = 0;
// 同样从右向左读
int *const curErr = &errNumb;   // const 指针
int const * curErr1 = &errNumb; // 指向 const 对象的指针
const int * curErr2 = &errNumb; // 指向 const 对象的指针
```

#### 顶层 const、底层 const

顶层 const 是对于对象来说的。
底层 const 是对于指针、引用来说的。
指针类型既可以是顶层 const 也可以是底层 const。

```cpp
int i = 0;
int *const p1 = &i; // 不能改变 p1，顶层 const。
const int ci = 42;  // 不能改变 ci，顶层 const。
const *p2 = &i;     // 允许改变 p2，底层 const。
const int &r = ci;  // 引用的 const 都是底层 const。
int &const a = i;   // 错误
```

#### constexpr 变量

```cpp
constexpr int mf = 10;
constexpr int sz = size(); // size() 为一个 constexpr 函数时才正确。
```

constexpr 与指针

```cpp
const int *p = nullptr;      // p 是一个指向整数常量的指针。
constexpr int *q = nullptr;  // q 是一个指向整数的常量指针。

constexpr int i = 10;
int j = 0;

constexpr const int *cp1 = &i; // cp1 为常量指针，指向整数常量 i
constexpr int *cp2 = &j;       // cp2 为常量指针，指向整数 j
```

### 处理类型

#### 别名

```cpp
// 两个都是 int* 类型
using int1p_t = int*;
typedef int *int2p_t;

typedef char *pstring;
// 下面两者不相同
const pstring cstr = 0;
const char *cstr = 0;
```

#### auto

auto 一般会忽略顶层 const

```cpp
int i = 0;
const int ci = i, &cr = ci;
auto b = ci;  // 整数
auto c = cr;  // 整数（cr 是 ci 别名，ci 本身是顶层 const）
auto d = &i;  // 整数指针
auto e = &ci; // 指向整数常量的指针，对常量对象取地址是一种底层 const

const auto f = ci; // 明确指出顶层 const

auto &g = ci; // g 是整数常量引用，顶层 const 保留

// m 是对整型常量的引用（ci 的顶层 const 的作用）， p1 是指向整型常量的指针(&ci 的底层 const 的作用)
auto &m = ci, *p1 = &ci; // 正确
auto &n = i, *p2 = &ci;  // 错误
// TODO p62 和往常一样，如果我们给初始值绑定一个引用，则此时的常量就不是顶层常量了。
```

#### decltype

如果不想用该表达式的值初始化变量。可以用 decltype 得到该表达式的类型。

> 引用从来都作为其所指对象的同义词出现，只有在 decltype 处是一个例外。

```cpp
int i = 42, *p = &i, r = i;

decltype(r + 0) b; // b 为 int
decltype(*p) c;    // 错误，*p 为引用，没有初始化

// i 理解为变量，(i) 理解为左值表达式
decltype((i)) d;   // 错误，d 为 int&
decltype(i) e;     // e 为 int
```

decltype((variable)) 结果永远是引用。

### 细节

#### 类型转换

```cpp
double i = 0.1;
static_cast<double>(3) / i;
void *p = &i;
double* dp = static_cast<double*>(p);

// 改变底层 const，常常用于重载。
const int *pc;
int *p = const_cast<int*>(pc);
// 底层次的重新解释
int *ip;
// 相当于 char *cp = (char*) ip;
char *cp = reinterpret_cast<int*>(ip);
```

#### 字符串

头文件里面不要使用 `using`。

```cpp
using std::cin;
using std::cout;
using std::cout, std::endl; // C++17 支持
```

```cpp
string s1 = "Hello";  // 拷贝初始化
string s2("World");   // 直接初始化
string s3(10, '!');   // 直接初始化
```

#### 数组

```cpp
int *ptrs[10];            // 含有10个指针的数组
int &refs[10] = /* ? */ ; // 不存在引用数组
// 由内而外
int (*Parray)[10] = &arr; // 指向数组的指针
int (&arrRef)[10] = arr;  // 引用数组的引用
int *(&arry)[10] = ptrs;  // 引用包含指针的数组
```

数组与指针

```cpp
string nums[] = {"one", "two", "three"};
// 两个等价
string *p = nums;
string *p = &nums[0];

int ia[] = {0, 1, 2, 3, 4};
auto ia2(ia); // ia2 是一个整型指针，相当于 auto ia2(&ia[0])

decltype(ia) ia3 = {0, 1, 2, 3, 4}; // ia3 为数组

// 可以用 begin、end 获取头元素指针、尾元素下一个位置的指针。
auto pbeg = begin(ia);
auto pend = end(ia);
```

#### switch 语句

switch 语句的内部变量定义

```cpp
switch (c) {
    case true:
        string s;   // 错误，隐式初始化
        int a = 10; // 错误，显式初始化
        int j;      // 正确，未初始化
    case false:
        {
            string sa = "Hello"; // 正确，在代码块
        }
}
```

### 函数


initializer_list 形参

```cpp
void fun (initializer_list<list> l) {
    // ...
}
```

#### 返回数组指针的函数

```cpp
// 使用别名
using arr_t = int[10];
arr_t* fun(int i);

// 直接定义
int (*fun(int i))[10];

// 尾置类型
auto fun(int i) -> int (*)[10];

// 使用 decltype
int arr[10];
decltype(arr) *fun(int i);
```

#### 函数重载

```cpp
// 只有返回值不一样，不能重载！
Record lookup(Phone *);
book lookup(Phone *);
```

顶层 const 不影响

```cpp
Record lookup(Phone);
Record lookup(const Phone);  // 重复声明

Record lookup(Phone*);
Record lookup(Phone *const); // 重复声明
```

底层 const

```cpp
Record lookup(Phone&);
Record lookup(const Phone&);  // 新函数

Record lookup(Phone *);
Record lookup(const Phone*);  // 新函数
```

const_cast 与重载

```cpp
const string& shorterString(const string& s1, const string& s2) {
    return s1.size() <= s2.size() ? s1 : s2;
}

string& shorterString(string& s1, string& s2) {
    auto &r shorterString(const_cast<const string&>(s1), 
                          const_cast<const string&>(s2));
    return const_cast<string&>(r);
}
```

> 在 C++ 语言中，名字查找发生在类型检查之前。

```cpp
void print(const string&);
void print(const double);

int main() {
    void print(int);
    print("3.14"); // 错误
    print(3.14);   // 调用 print(int)
    print(3);      // 调用 print(int)
}
```

默认参数

```cpp
void fun(int a, int b, int c = 3);

void fun(int a, int b=3, int);

void fun(int a, int b, int c) {
    cout << a << endl;
    cout << b << endl;
    cout << c << endl;
}
```

默认实参初始化

```cpp
int ex_v = 0;

void test(int a, int b=ex_v) {
    cout << a << endl;
    cout << b << endl;
}

int main() {
    test(1); // 输出 1, 0
    ex_v = 3;
    test(1); // 输出 1, 3
}
```

> constexpr 函数不一定返回常量表达式

```cpp
// 如果 cnt 是常量表达式，scale(cnt) 也是常量表达式。
constexpr size_t scale(size_t cnt) {
    return 42 * cnt;
}

int arr[scale(2)]; // 正确
int i = 2;
int arr[scale(i)]; // 错误
```

constexpr 函数和 inline 函数可以在程序内多次定义。

> 把 inline 函数和 constexpr 函数放在头文件中。

#### 函数指针

```cpp
bool length_compare(const string&, const string&);

bool (*pf)(const string&, const string&);

// 下面两个等价
pf = length_compare;
pf = &length_compare;

auto b1 = pf("ba", "a");
auto b2 = (*pf)("ba", "a");
auto b3 = length_compare("ba", "a");
```

函数作为形式参数

```cpp
// 两者等价
// 函数类型
void use_bigger(const string &s1, const string &s2, 
                bool pf(const string&, const string&));
// 指针类型
void use_bigger(const string &s1, const string &s2, 
                bool (*pf)(const string&, const string&));

```

返回指向函数的指针

```cpp
// 下面两个等价
typedef bool Func(const string&, const string&);
typedef decltype(length_compare) Func;

// 下面两个等价
typedef bool (*FuncP)(const string&, const string&);
typedef decltype(length_compare) *FuncP;

using F = int(int*, int)
using PF = int (*)(int*, int)

PF f1(int); // 正确，f1 返回指向函数的指针
F f1(int);  // 错误
F *f1(int); // 正确

// f1 有形参列表，说明 f1 是一个函数
int (*f1(int))(int*, int);

// 尾置返回类型
auto f1(int) -> int (*)(int *, int);
```

## 泛型算法

### lambda 表达式

```cpp
int sz = 5;
auto wc = find_if(words.begin(), words.end(),
                 [sz](const string& s) {
                    return s.size() >= sz;
                 });
```

> 捕获列表只用于局部非 static 变量，lambda 可以直接使用局部 static 变量和所在函数声明之外的变量。

值捕获：捕获的值必须是可以拷贝的，并且是在 lambda 表达式创建过程时进行拷贝的，而不是在调用时。

引用捕获：捕获的值必须是可以拷贝的，并且是在 lambda 表达式创建过程时进行拷贝的，而不是在调用时。

函数返回值是 lambda 表达式的时候不应该使用引用捕获。

```cpp

void fun() {
    size_t v = 42;
    auto f = [&v] {return v;};
    // auto f = [v] {return v;};
    v = 0;
    // 当使用值捕获时 k = 42
    // 当使用引用捕获时 k = 0
    auto k = f();
}
```

隐式捕获

在捕获列表中写 = 表示值捕获，& 表示引用捕获。也可以使用混合捕获，这时候 = 和 & 必须在第一个。

```cpp
// os 为隐式捕获，引用捕获
// c  为显示捕获，值捕获
for_each(words.begin(), words.end(),
        [&, c] (const string &s) {os << s << c;});
// os 为显式捕获，引用捕获
// c  为隐式捕获，值捕获
for_each(words.begin(), words.end(),
        [=, &os] (const string &s) {os << s << c;});
```

#### 可变 lambda

和引用捕获有很大不同。可变指的是在 lambda 函数体内能改变。

```cpp
void fun() {
    size_t v = 42;
    auto f = [v] () mutable { return ++v;};
    auto k = f();
    // 输出 43 和 42
    // 如果使用引用捕获，结果都是 43
    cout << k << endl;
    cout << v << endl;

    const size_t vc = 42;
    auto fc = [&vc] () { return ++vc;}; // 错误
    auto kc = fc();
}
```

#### 指定返回类型


多个 return 语句的时候需要手动指定返回类型。

```cpp
// 错误，编译器不能推断返回类型
transform(vi.begin(), vi.end(), vi.begin(), [](int i) {
    if (i < 0) return -i;
    else return i;
});

// 正确
transform(vi.begin(), vi.end(), vi.begin(), [](int i) -> int {
    if (i < 0) return -i;
    else return i;
});
```

### 参数绑定

```cpp
auto wc = find_if(words.begin(), words.end(),
                 [sz](const string& s) {
                    return s.size() >= sz;
                 });
```

如果想要传入 sz 的值，可以多加一个参数，但是这样就不能作为 find_if 的参数了。

```cpp
bool check_size(const string &s, string::size_type sz) {
    return s.size() >= sz;
}
```

需要向 check_size 传递一个参数，需要用到标准 bind 库。

一般形式为：

```cpp
auto new_callable = bind(callable, arg_list);
```

```cpp
using std::placeholders::_1;

// 绑定 sz = 6
auto check6 = bind(check_size, _1, 6);
string s = "Hello";
bool b1 = check6(s);

// 使用 bind 的 find_if 函数
auto wc = find_if(words.begin(), words.end(), bind(check_size, _1, 6));
```

#### 通过 bind 重新安排参数顺序

```cpp
auto g = bind(f, a, b, _2, c, _1);
```

调用 `g(X, Y)` 相当于 `f(a, b, Y, c, X)`。

#### 绑定引用参数

ref 返回一个对象，包含给定引用，这个对象是可以拷贝的，cref 生成一个包含 const 引用的类。

```cpp
ostream &print(ostream &os, const string &s, char c) {
    return os << s << c;
}

// 错误，os 无法拷贝
for_each(words.begin(), words.end(), bind(print, os, _1, ' '));

// 使用标准库 ref 函数
for_each(words.begin(), words.end(), bind(print, ref(os), _1, ' '));
```

### 迭代器

#### 插入迭代器

有三种，back_inserter、front_inserter 和 inserter。

```cpp
vector<int> a;
auto it = back_inserter(a);
// 可以不加 *
it = 12;
it = 11;
// *it = 12;
// *it = 11;
for_each(a.begin(), a.end(), [](int v) {cout << v << " ";});

list<int> lst = { 1, 2, 3, 4};
list<int> lst2, lst3, lst4;
// lst2 为 1, 2, 3, 4
std::copy(lst.begin(), lst.end(), back_inserter(lst2));
// lst3 为 1, 2, 3, 4
// inserter 中 lst3.begin() 只表明初始插入位置
std::copy(lst.begin(), lst.end(), inserter(lst3, lst3.begin()));
// lst4 为 4, 3, 2, 1
std::copy(lst.begin(), lst.end(), std::front_inserter(lst4));
```

#### iostream 迭代器

istream_iterator

```cpp
istream_iterator<int> in(cin); // 从 cin 读取 int
istream_iterator<int> eof;     // 默认初始化迭代器

// 循环添加元素
vector<int> vec;
while (in != eof)
    vec.push_back(*in++);

// 构造函数
vector<int> vec(in, eof);

// 使用累加函数
accumulate(in, eof, 0);
```

ostream_iterator

```cpp
ostream_iterator<int> out(cout, " ");

for (auto e : vec) {
    // 两个语句等价
    // *out out++ out-- 都返回 out，不对 out 造成任何影响 
    // *out++ = e;
    out = e;
}

// 直接调用 copy 来打印 vec 的元素
copy(vec.begin(), vec.end(), out);
```

反向迭代器

```cpp
const string line = "FIRST,MIDDLE,LAST";
auto rcomma = find(line.crbegin(), line.crend(), ',');

/*
        comma            base    cend
          |               |       |
F I R S T , M I D D L E , L A S T 
                        |       |
                     rcomma  rcbegin
*/

// 输出 TSAL
cout << string(line.crbegin(), rcomma) << endl;
// 输出 LAST
cout << string(rcomma.base(), line.cend()) << endl;
```

## 动态内存

### shared_ptr

传入函数实参，作为函数返回值时都会增加智能指针的引用计数。

```cpp
auto p = make_shared<int>(42);
auto q(p); // 此时引用计数会加一
```


```cpp
// 创建对象，并使用 obj 进行初始化
auto p = new auto(obj);
// 创建 const 对象
const int *p = new const int(1024);

int *p = new int;           // 若分配失败抛出 std::bad_alloc
int *p = new (nothrow) int; // 若分配失败返回空指针
```

使用自己的释放操作：

```cpp
void end_connect(connection *p) { disconnect(*p); }

connection c = connect(&d);
shared_prt<connection> p(&c, end_connection);

// 直接使用 lambda 表达式
shared_prt<connection> p(&c, [](connnection *p) {
    disconnect(*p);
});
```

### unique_ptr

只能有一个 unique_ptr 对象指向给定对象。不支持普通的赋值和拷贝。

在容器内保存指针、管理动态数组。

```cpp
unique_ptr<string> p1(new string("Hello"));

// unique_ptr<string> p2(p1); 错误
unique_ptr<string> p2(p1.release());

// 执行特殊的拷贝
unique_ptr<int> clone(int p) {
    return unique_ptr<int>(new int(p));
}

// 管理删除器的方式与 shared_ptr 不同
unique_ptr<connection, decltype(end_connection)*> p (&c, end_connection);

// 只有 unique_ptr 可以这样
unique_ptr<int[]> up(new int[10]);
// shared_ptr 可以这样使用
shared_ptr<int> sp(new int[10], [](int *p) {delete[] p});
```

### week_ptr

解决循环引用。如果 B 中使用 shared_ptr 函数调用完之后，调用完之后，pa pb 的引用计数都是 1，并不会调用析构函数。

```cpp
struct A;
struct B;

struct A {
    shared_ptr<B> p_b;
    ~A() {
        cout << "~A" << endl;
    }
};

struct B {
    weak_ptr<A> p_a;
    // shared_ptr<A> p_a;
    ~B() {
        cout << "~B" << endl;
    }
};

void test_weakptr() {
    auto pa = make_shared<A>();
    auto pb = make_shared<B>();

    pa->p_b = pb;
    pb->p_a = pa;
}

int main() {
    test_weakptr();
    cout << "END" << endl;
}
```

### allocator 类

提供的内存是原始的、未构造的，避免无用的构造。

```cpp
allocator<string> alloc;

auto const p = alloc.allocate(n);
auto q = p; // q 指向最后构造的元素之后的位置

alloc.construct(q++);
alloc.construct(q++, 10, 'c');
alloc.construct(q++, 10, 'c');

// 正确
cout << *p << endl;
// 错误
cout << *q << endl;

// 销毁
while (q != p) {
    alloc.destroy(--q);
}

alloc.deallocate(p, n);
```


```cpp
auto p = alloc.allocate(vi.size() * 2);
// 将 vi 的内容拷贝到 p 指向的为初始化内存区域
auto q = uninitialized_copy(vi.begin(), vi.end(), p);
uninitialized_fill(q, vi.size(), 42);
```

## 拷贝控制

### default 和 delete

default 可以用于编译器可以自动为类合成的函数（拷贝控制成员或默认构造函数）。

delete 可以用于任何函数。并且只能出现在声明处。

如果一个类有数据成员不能默认构造、拷贝、赋值或销毁，那么相应的成员函数会被定义成删除的。

### 三/五原则

> 析构函数并不直接销毁成员。

如果一个类需要自定义析构函数，那么几乎可以肯定它需要自定义拷贝构造函数和拷贝赋值运算符。

如果一个类需要拷贝构造函数，那么它也需要一个拷贝赋值运算符。反之亦然。

如果一个类需要移动构造函数，那么它也必须定义拷贝构造函数。移动赋值和拷贝赋值运算也是如此。

> 五个拷贝控制成员应该看作是一个整体。

### 交换操作

```cpp
class HasPtr {
    friend void swap(HasPtr&, HasPtr&);
    int i;
    string *ps;
    // ..
};

void swap(HasPtr& lhs, HasPtr& rhs) {
    using std::swap;
    swap(lhs.ps, rhs.ps);
    swap(lhs.i, rhs.i);
}
```

#### copy & swap

> 使用拷贝和交换的赋值运算符自动就是**异常安全**的，且能正确处理**自赋值**。

```cpp
// copy & swap
HasPtr& HasPtr::operator= (HasPtr rhs) {
    swap(*this, rhs);
    return *this;
}

HasPtr& HasPtr::operator= (const HasPtr &rhs) {
    // 先释放会导致自赋值异常。
    delete this.ps;
    this.ps = new string(*rhs.ps)
    return *this;
}

HasPtr& HasPtr::operator= (const HasPtr &rhs) {
    if (this == &rhs) return *this;
    else delete this.ps;
    // 如果申请错误，此时 ps 已经删除。
    this.ps = new string(*rhs.ps)
    return *this;
}

HasPtr& HasPtr::operator= (const HasPtr &rhs) {
    // 可以删除这句话也能正确执行，但是效率会低一点。
    if (this == &rhs) return *this;

    // 重排顺序
    auto pre_ps = ps;
    this.ps = new string(*rhs.ps)
    delete pre_ps;
    return *this;
}
```

### 对象移动

#### 右值引用

> 可以将 const 引用绑定到右值上。

右值引用是必须绑定到右值的引用，只能绑定将要销毁的对象。


不能将右值引用直接绑定到一个左值上，但是可以用 std::move 实现。

```cpp
int &&rr1 = 42;  // 正确
int &&rr2 = rr1; // 错误，rr1 是左值
int &&rr3 = std::move(rr1); // 希望编译器按照右值处理这个左值
```

#### 移动构造函数和移动赋值运算符

移动操作通常不会抛出异常，可以对函数使用 noexcept（声明和定义都需要标记，在初始化列表之前）。
不抛出异常的移动构造函数和移动赋值运算符必须标记为 noexcept。

```cpp
class Foo {
public:
    Foo() = default;
    Foo(const Foo&); // 拷贝构造函数
    Foo(const Foo&&) noexcept;
};

Foo x;
Foo y(x);            // 调用拷贝构造函数，x 为左值
Foo z(std::move(x)); // 调用移动构造函数，std::move(x) 为右值
```

如果一个类**没有自定义**任何拷贝控制成员，且它的所有数据成员都是能够移动构造或移动赋值时，编译器才会为它合成移动构造函数或移动赋值运算符。

移动操作**不会隐式**定义为删除函数，如果显示要求编译器生成，并且有成员无法移动时，会定义成删除函数。

如果 Foo 不定义移动构造函数：

```cpp
Foo y(x);            // 调用拷贝构造函数，x 为左值
Foo z(std::move(x)); // 调用拷贝构造函数，std::move(x) 为右值
```

uninitialized_copy 中可以使用 make_move_iterator 将普通迭代器转化成移动迭代器来进行批量移动。

#### 拷贝并交换赋值运算符和移动操作

这里赋值运算符既是移动赋值运算符也是拷贝赋值运算符，主要区别是实参传递的过程。

```cpp
class HasPtr {
public:
    HasPtr(HasPtr &&p) noexcept : ps(p.ps), i(p.id) {p.ps = 0;}
    HasPtr& operator=(HasPtr rhs) {
        swap(*this, rhs);
        return *this;
    }
};

hp1 = hp;            // 拷贝赋值运算
hp2 = std::move(hp); // 移动赋值运算
```

#### 右值引用和成员函数

引用限定符 & 和 &&

```cpp
struct RRef {
    vector<int> a = {3, 1, 2};
    RRef sorted() const & {
        RRef b(*this);
        sort(b.a.begin(), b.a.end());
        cout << "l ref" << endl;
        return b;
    }

    RRef sorted() && {
        sort(a.begin(), a.end());
        cout << "r ref" << endl;
        return *this;
    }
};

RRef a;
auto b = a.sorted();
a.sorted().sorted();
```

重载相关

```cpp
struct Foo {
    Foo sorted() &&;
    Foo sorted() const; // 错误，必须加上引用限定符

    using Comp = bool(const int&, const int&);
    // 两个都没有引用限定符
    Foo sorted(Comp *);
    Foo sorted(Comp *) const;
};
```

## 面向对象程序设计

### 类

#### 可变成员变量

```cpp
class Foo {
    mutable size_t access_num;
    void some_member() const {
       access_num ++;
    }
};
```

#### 类的声明

```cpp
class Foo; // 前向声明

class Foo {
    class Foo1 {
        int c;
    };
    int a;
    Foo1* f1(Foo1);
};

// 返回值需要加 Foo::
Foo::Foo1* Foo::f1(Foo1 foo1) {
    return nullptr;
}
```

类型名要特殊处理

```cpp
class Account {
public:
    Money balance() { // 使用外层作用域的 Money
        return bal;
    }
    void set(Money bal) { // 使用外层作用域的 Money
        this->bal = bal;
    }
private:
    typedef double Money; // 错误，不能重新定义 Money
    Money bal;
};

int main() {
    Account a;
    a.set(1.2);
    cout << a.balance() << endl;
}
```

#### 隐式转换

```cpp
class Foo {
    Foo() = default;
    Foo(const string &str);
    Foo(istream &in);
    void f(Foo);
};

Foo foo;

foo.f("Hello");         // 错误，需要进行两次
foo.f(string("Hello")); // 正确
foo.f(cin);             // 正确
```

避免隐式转换

```cpp
class Foo {
    Foo() = default;
    explicit Foo(const string &str);
    explicit Foo(istream &in);
    void f(Foo);
};

// 错误 explicit 只能出现在类内声明处
explicit Foo::Foo(const string &str) {
}


foo.f(string("Hello"));       // 错误
foo.f(cin);                   // 错误
// static_cast 可以使用 explicit 的构造函数
foo.f(static_cast<Foo>(cin)); // 正确
```

#### 静态成员

```cpp
class Bar {
public:
    static string s;
    // 可以使用静态成员作为默认实参。
    void foo(string str=s);
private:
    Bar * bar;       // 正确，指针成员可以是不完全类型。
    static Bar bar1; // 正确，静态成员可以是不完全类型。
    // Bar bar1;        // 错误，数据成员必须是完全类型。
};

string Bar::s = "YES";

void Bar::foo(string str) {
    cout << "Bar static s: " << str << endl;
}

int main() {
    Bar bar;
    Bar::s = "NO";
    bar.foo();
    Bar::s = "YES";
    bar.foo();
}
```

### 继承与虚函数

#### 公有、私有和受保护继承

```cpp
// private B 表示 B 中的 public 在 D 中的属性
// struct 的默认继承是公有继承
// class  的默认继承是私有继承
struct D final : private B {
    // ...
    // 可以利用 using 使得 D 对象可以使用 B::f3。
    // using B::f3;
}

void D::f2() const {
    // 派生类允许向基类的转换
    B *bp = new D();
    // 智能指针不允许这样操作
    // shared_ptr<B> bp = make_shared<D>();
    bp->f3();
}

int main() {
    // 错误，私有继承不允许向基类转换
    // 如果是 public，下面两种方式都是正确的。
    B *bp1 = new D();
    bp1->f3();
    shared_ptr<B> bp2 = make_shared<D>();
    bp2->f3();
}
```

#### override 和 final

描述函数时，出现在形参列表（包括 const 和 &）和后置返回值之后。

```cpp
struct B {
    virtual void f1(int);
    virtual void f2() const;
};

// 可以在类名后添加 final 表示不允许继承
struct D final : public B {
    // 添加 override 可以有利于检查错误
    void f1(int) override;
    // final 不允许再重写
    void f2() const final;
    // 纯虚函数
    virtual void f3() = 0;
};
```

#### 虚函数表

1. 当类有虚函数时，类的**第一个**元素会变为一个指向「包含函数指针的数组」的指针 `void** __vfptr`。
2. 当继承一个有虚函数的类时，自己新定义的虚函数会在基类虚函数表的后面进行添加。
3. 重写基类虚函数，虚函数表的相应表项会变为派生类的函数指针。
4. 派生类指针转化成基类指针的后，两者并不一定相等，可能会做相应的偏移。

派生类的虚函数的必须和基类的形式参数相同，返回值也需要相同但是有个例外，函数返回值是类本身的指针或引用。

回避虚函数机制：

```cpp
B *b = new D();
// 调用 B::f1，在编译时完成解析
auto v = base_p->B::f1();
// 调用 D::f1
auto v = base_p->f1();
```

#### 拷贝控制与继承

虚析构函数的间接影响：如果一个类**定义了析构函数**，即使它通过 =default 的形式使用了合成的版本，编译器也**不会**为这个类合成**移动操作**。

```cpp
class D : public Base {
public:
    // 显示调用移动/拷贝构造函数
    // 拷贝基类成员
    D(const D& d) : Base(d) {
        // ...
    }

    // 移动基类成员
    D(D&& d) : Base(std::move(d)) {
        // ...
    }

    // 不会自动调用 Base::operator=，需要显示调用
    D& D::operator=(const D &rhs) {
        Base::operator=(rhs);
        return *this;
    }

    // Base::~Base 被自动调用执行
    ~D() {
        // 清理派生类成员的操作
        // ...
    }
};
```

在构造函数或析构函数里面调用虚函数时，会执行与其构造或析构类型所对应的虚函数版本。

```cpp
struct BaseVF {
    virtual void fun() {
        cout << "Base VF" << endl;
    }
    // 如果不加 virtual 调用 ~DerivedVF 依然会调用 ~BaseVF
    virtual ~BaseVF() {
        fun();
    }
};

struct DerivedVF : BaseVF {
    void fun() {
        cout << "Derived VF" << endl;
    }
    ~DerivedVF() {
        fun();
    }
};

void test_derive_virtual_fun() {
    BaseVF* p = new DerivedVF();
    p->fun();
    delete p;
}
```

#### 构造函数与继承

这里并不是真正在派生类中定义函数，而是直接调用基类的构造函数，虽然名字不一样。

```cpp
struct BaseCF {
    int acc;
    BaseCF(int a) : acc(a) {
    }
protected:
    BaseCF(string s) {
        cout << s << endl;
    }
private:
    BaseCF(const int *i) {
        cout << *i << endl;
    }
};

struct DerivedCF : BaseCF {
    int ratio;
    using BaseCF::BaseCF;
    void fun() {
        // 正确
        auto p1 = new DerivedCF("A");
        int i = 2;
        // 错误
        auto p2 = new DerivedCF(&i);
    }
};
```