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
#include <iosream>
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

在这个场景中，我们事实上需要的

