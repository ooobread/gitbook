# C++11 新特性理解

## 继承构造函数

对于基类的非默认构造函数，衍生类需要显式声明基类的构造函数，因此在基类有大量不同版本的构造函数，而衍生类完全采用基类构造函数的情况下，衍生类中需要写大量“透传”的构造函数，例子如下：

```cpp
struct A {
    A(int i) {}
    A(double d, int i) {}
    A(float f, int i, const char* c) {}
    // ...
};

struct B : A {
    B(int i) : A(i) {}
    B(double d, int i) {}
    B(float f, int i, const char* c) {}
};
```

使用using声明，我们可以使用基类的成员函数

```cpp
struct A {
    void f(int i) {}
};

struct B : A {
    using A::f;
    void f(double d) {}
};
```

这样，在衍生类B中，我们可以使用基类A的函数f，在传入整型数调用函数f时，使用的是A中的函数，传入浮点数调用函数f时，使用的是B中的函数。

同理，对于构造函数，我们也可以采用类似方法，直接使用A的构造函数初始化： 

```cpp
struct A {
    A(int i) {}
    A(double d, int i) {}
    A(float f, int i, const char* c) {}
    // ...
};

struct B: A {
    using A::A;
    //...
};
```

但是这种方法无法初始化衍生类B中的成员变量，因此我们还可以运用C++11的另一项新特性来初始化成员变量：

```cpp
struct B: A {
    using A::A;
    int d {0};
};
```

## 委派构造函数\*

## 移动语义

### 浅拷贝与深拷贝

当类中涉及指针时，我们通常需要重写拷贝构造函数，以防止出现编译器隐式调用拷贝构造函数，导致两个类中的指针成员变量指向同一块内存，导致重复释放：

```cpp
class HasPtrMem {
public:
    HasPtrMem : d(new int(0)) {}
    ~HasPtrMem { delete d; }
    
    int* d;
};

int main()
{
    HasPtrMem a;
    HasPtrMem b(a);
}
```

上面这段代码中，在使用对象a对对象b进行初始化时，隐式调用了拷贝构造函数，将对象a中的指针d直接赋值给对象b中的指针d，也就是通常所说的“浅拷贝”，这会导致两个指针指向同一块内存空间，导致内存的重复释放，而通常来说，我们通过显式实现拷贝构造函数来实现“深拷贝”，保证拷贝过程中为新的指针分配新的内存，防止重复释放：

```cpp
class HasPtrMem {
public:
    HasPtrMem : d(new int(0)) {}
    HasPtrMem(HasPtrMem &h) : d(new int(*(h.d))) {}
    ~HasPtrMem { delete d; }
    
    int* d;
};

int main()
{
    HasPtrMem a;
    HasPtrMem b(a);
}
```

### 移动语义

考虑如下例子：

```cpp
#include <iostream>
using namespace std;

class HasPtrMem {
public:
    HasPtrMem() : d(new int(0))
    {
        cout << "Construct: " << ++n_cstr << endl;
    }
    
    HasPtrMem(const HasPtrMem &h) : d(new int(*(h.d)))
    {
        cout << "Copy construct: " << ++n_cptr << endl;
    }
    
    ~HasPtrMem()
    {
        cout << "Destruct: " << ++n_dstr << endl;
    }
    
    int* d;
    static int n_cstr;
    static int n_dstr;
    static int n_cptr;
};

int HasPtrMem::n_cstr = 0;
int HasPtrMem::n_dstr = 0;
int HasPtrMem::n_cptr = 0;

HasPtrMem GetTemp()
{
    return HasPtrMem();
}

int main()
{
    HasPtrMem a = GetTemp();
}

// 未加-fno-elide-constructors编译选项结果：
// Construct: 1
// Destruct: 1

// 加入-fno-elide-constructors编译选项结果：
// Construct: 1
// Copy construct: 1
// Destruct: 1
// Copy constrcut: 2
// Destruct: 2
// Destruct: 3
```

\*tips: 若未加入-fno-elide-constrctors编译选项，C++标准允许编译器进行一种对代码中的赋值初始化进行的优化，即省略创建一个只为了初始化而存在的临时对象。而该编译选项将关闭这种优化。详细描述如下：

> **-fno-elide-constructors**
>
> The C++ standard allows an implementation to omit creating a temporary that is only used to initialize another object of the same type. Specifying this option disables that optimization, and forces G++ to call the copy constructor in all cases.

我们可以看到，在编译器不进行优化的情况下，会进行的操作如下：

1. 调用构造函数，为了创建临时对象。
2. 调用拷贝构造函数，利用创建的临时对象拷贝构造一个临时对象。
3. 临时对象摧毁，调用析构函数。
4. 调用拷贝构造函数，利用刚刚拷贝构造的临时对象再次拷贝构造一个对象a。
5. 调用两次析构函，依次摧毁两个对象。

通过上述过程，我们显然发现这段代码效率是极低的，这体现在为了拷贝构造目标对象而创建的临时对象，我们需要对临时对象进行构造和析构，而在构造函数和拷贝构造函数中存在内存申请时，这种额外开销带来的低效将会被放大，这事实上不被我们所需要。

在这个场景中，我们事实上需要的是让新的对象中的指针指向临时对象所申请的内存，同时保证临时对象不会对这块内存进行释放。这样的将临时变量中资源移作己用的构造函数被称为“移动构造函数”，而这样的行为被成为“移动语义”。参考如下代码：

```cpp
#include <iostream>
using namespace std;

class HasPtrMem {
public:
    HasPtrMem() : d(new int(0))
    {
        cout << "Construct: " << ++n_cstr << endl;
    }
    
    HasPtrMem(const HasPtrMem &h) : d(new int(*(h.d)))
    {
        cout << "Copy construct: " << ++n_cptr << endl;
    }
    
    HasPtrMem(HasPtrMem &&h) : d(h.d)
    {
        h.d = nullptr;
        cout << "Move construct: " << ++n_mvtr << endl; // 移动构造函数，将传入对象中的指针赋值给目标对象中的指针，是他们指向同一块内存，然后将临时对象中指针置空，防止其释放内存
    }

    ~HasPtrMem()
    {
        if (d != nullptr) {
            delete d;
        }

        cout << "Destruct: " << ++n_dstr << endl;
    }
    
    int* d;
    static int n_cstr;
    static int n_dstr;
    static int n_cptr;
    static int n_mvtr;
};

int HasPtrMem::n_cstr = 0;
int HasPtrMem::n_dstr = 0;
int HasPtrMem::n_cptr = 0;
int HasPtrMem::n_mvtr = 0;

HasPtrMem GetTemp()
{
    return HasPtrMem();
}

int main()
{
    HasPtrMem a = GetTemp();
}

// 运行结果：
// Construct: 1
// Move construct: 1
// Destruct: 1
// Destruct: 2
// Destruct: 3
```
我们可以看到使用移动构造函数之后，我们避免使用了内存的拷贝，仅仅通过一次移动构造函数就将内存所有权转交给另一个对象中的指针，避免了性能的浪费。这里使用了新的&&语法，即右值引用。

### 右值引用
C++中的值可以被分为左值和右值，简单来说，左值是一个具体的，可以被察觉到的，或者说有名字的值；而右值则通常指向一些临时的、匿名的值（比如说1+3所产生的临时变量值）或者字面值（比如2、'c'、true）。在C++11之前，我们只能使用T&语法来引用左值，这叫做左值引用；C++11引入了右值引用（T&&）。左值引用是具名变量的别名，而右值引用则是匿名变量的别名。

### std::move: 强制转化为右值
std::move 的功能为将一个左值强制转化为右值引用，继而我们可以通过右值引用使用该值，以用于移动语义。

事实上，这个函数近似于对左值进行一个类型转换：
```cpp
static_cast<T&&>(lvalue);
```
因此，这个函数不会对左值的声明周期产生影响。也就是说，在上述例子中，lvalue的声明周期依然会持续到作用域结束，而不是在转换之后立即结束。

下面是一个**错误**使用的例子：
```cpp
#include <iostream>
using namespace std;
class Moveable {
public:
    Moveable() : i(new int(3)) {}
    ~Moveable() {delete i;}
    Moveable(const Moveable &m) : i(new int(*(m.i))) {}
    Moveable(Moveable &&m) : i(i.m)
    {
        m.i = nullptr;
    }

    int* i;
};

int main()
{
    Moveable a;
    Moveable c(move(a)); // 调用了c的移动构造函数，a.i被置空
    cout << *(a.i) << endl; // a的生命周期还未结束，但a.i被置空，因此会出现运行时错误
}
```
通常来说，我们需要使用std::move来将一个确实生命周期即将结束的对象转换为右值，以便使用移动构造函数。如下例子：
```cpp
#include <iostream>
using namespace std;
class HugeMem {
public:
    HugeMem(int size) : sz(size > 0 ? size : 1)
    {
        c = new int[sz];
    }
    ~HugeMem() { delete[] c; }

    HugeMem(HugeMem&& hm) : sz(hm.sz), c(hm.c)
    {
        hm.c = nullptr;
    }

    int* c;
    int sz;
};

class Moveable {
public:
    Moveable() : i(new int(3)), h(1024) {}
    ~Moveable() { delete i; }
    Moveable(Moveable&& m) : i(m.i), h(move(m.h)) // 使用std::move将m.h转化为右值，以使用移动构造函数来初始化h，此处m虽然为右值，但m.h依然为左值，因此需要将其转为右值来调用h的移动构造函数。如果此处不使用std::move，则将直接调用h的拷贝构造函数
    {
        m.i = nullptr;
    }

    int* i;
    HugeMem h;
};

Moveable GetTemp()
{
    Moveable tmp = Moveable();
    return tmp;
}

int main()
{
    Moveable a(GetTemp());
    return 0;
}
```
拷贝构造/复制和移动构造/赋值函数必须同时提供，或者同时不提供，才能保证类同时具有拷贝和移动语义，只声明其中一种的话，类都仅能实现一种语义。

通常来说，只实现移动语义的情况也是存在的，通常来说为“资源型”的类型，例如智能指针、文件流等。

### 完美转发
完美转发(perfect forwarding)，指在函数模板中，完全依照模板的参数的类型，将参数传递给函数模板中调用的另一个函数。如下例子：
```cpp
template <typename T>
void IamForwarding(T t) { IrunCodeActually(t); }
```
在上述例子中，IamForwarding为一个转发函数模板，而真正执行的函数为IrunCodeActually。这里由于没有使用引用，因此多了一次拷贝开销。要解决这个问题，我们可以使用一个引用类型来消除这个开销，但是为了能够让改模板函数既可以接收左值引用，也可以接收右值引用，我们需要使用常量引用作为入参类型，而这就可能出现如下情况：
```cpp
void IrunCodeActually(int t) {}
template <typename T>
void IamForwarding(const T& t) { IrunCodeActually(t); } // IrunCodeActually入参为非常量左值引用类型，因此无法接收常亮左值引用类型t。
```
为了解决这个问题，C++11引入了“引用折叠”（reference collapsing）的语言规则，结合新的模板推到规则来完成完美转发。

考虑以下语句：
```cpp
typedef const int T;
typedef T& TR;
TR& v = 1; // 在C++98中会导致编译错误
```
上述例子中，`TR&`将被推导为`const int T &&`,由于C++98不支持右值引用，因此将会编译错误。

而结合C++11新的“引用折叠”规则，变量v将被推导为`const int T &`类型，考虑到常量左值引用为万能类型，因此该语句将编译成功。

实际上，“引用折叠”简单来说为以下推导：

`T &` + `&` = `T &`

`T &` + `&&` = `T &`

`T &&` + `&` = `T &`

`T &&` + `&&` = `T &&`

因此我们可以将模板函数写为如下形式，来达成我们的完美转发
```cpp
template <typename T>
void IamForwarding(T && t)
{
    IrunCodeActually(static_cast<T &&>(t)); // static_cast是为了右值引用准备的。在右值引用表达式中，t实际上是一个左值，为了继续使用右值传递，我们使用static_cast将其转换为右值引用（类似于std::move，或者后面提到的std::forward）。
}
```

这样，通过新的类型推导，我们既可以传入左值引用，也可以传入右值引用。

一个完美转发的例子：
```cpp
#include <iostream>
using namespace std;

void RunCode(int && m) { cout << "rvalue ref" << endl; }
void RunCode(int & m) { cout << "lvalue ref" << endl; }
void RunCode(const int && m) { cout << "const rvalue ref" << endl; }
void RunCode(const int & m) { cout << "cosnt lvalue ref" << endl; }

template <typename T>
void PerfectForward(T &&t) { RunCode(forward<T>(t)); }

int main()
{
    int a = 0;
    int b = 0;
    const int c = 1;
    const int d = 0;
    PerfectForward(a); // lvalue ref
    PerfectForward(move(b)); // rvalue ref
    PerfectForward(c); // const lvalue ref
    PerfectForward(move(d)); // const rvalue ref
}
```
