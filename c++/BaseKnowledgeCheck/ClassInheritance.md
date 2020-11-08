# 类继承

## 构造函数/析构函数
* 创建派生类对象时，先调用基类构造函数，再调用派生类构造函数。
* 销毁派生类对象时，先调用派生类析构函数，再调用基类析构函数。
* 派生类的构造函数总是调用基类的构造函数，可以使用列表初始化指定调用的基类构造函数，若未指定，则调用基类默认构造函数。

## 基类/派生类指针/引用
* 基类指针可以在不进行显式类型转换的情况下指向派生类对象，基类**引用**可以在不进行显式类型转换的情况下引用派生类对象。
* 基类指针/引用只能调用基类方法/成员变量，即便其实际类型为派生类（即基类指针指向派生类对象，或者基类引用引用派生类对象）。

## 公有继承
* 有两种方法实现多态公有继承，即重新定义基类的方法和虚函数。
* 在继承中，虚函数的意义在于，程序将根据引用或指针指向的对象类型来选择调用的函数；而仅仅进行函数的重写，程序将仅根据类型来选择调用的函数，一个简单的例子：
```cpp
#include <iostream>
using namespace std;

class Base {
public:
    Base() {}
    virtual ~Base() {}

    void FuncNonVirtual()
    {
        cout << "Base FuncNonVirtual" << endl;
    }

    virtual FuncVirtual()
    {
        cout << "Base FuncVirtual" << endl;
    }
};

class Derived : public Base {
public:
    Derived() {}
    virtual ~Derived() {}

    void FuncNonvirtual()
    {
        cout << "Derived FuncNonVirtual" << endl;
    }

    virtual FuncVirtual()
    {
        cout << "Derived FuncVirtual" << endl;
    }
};

int main()
{
    Base* pBase = new Derived();
    pBase->FuncNonVirtual(); // Base FuncNonVirtual
    pBase->FuncVirtual(); // Derived FuncVirtual
}
```

* 赋值运算符不能被继承

## 私有继承
* 表示一种has-a关系，通常被用于表示“包含”关系的两个类之间。
* 在私有继承中，基类的公有成员和保护成员都将成员派生类的私有成员。
* 当我们使用私有继承继承几个基类时，我们将可以使用基类的组件，实现了“包含”的意涵。我们当然可以通过在类中直接声明对象来使用组件，事实上这是实现“包含”更推荐的方式。但是使用私有继承会更加灵活，它的好处包括但不限于：
  * 可以使用基类的保护成员
  * 可以重新定义虚函数