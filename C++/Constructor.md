# 构造函数

拷贝构造函数，入参为const 本类型&。
在使用同类型的左值初始化另一对象时，将调用拷贝构造函数。
在使用同类型的右值初始化另一对象时，在没有移动构造函数的情况下将调用拷贝构造函数。
如果未显式定义拷贝构造函数，编译器会提供一个默认拷贝构造函数，进行成员之间的拷贝。

移动构造函数（C++），入参为本类型的右值引用，即本类型&&。
在使用同类型的右值初始化另一对象时，编译器会优先调用移动构造函数，在没有移动构造函数的情况下，将调用拷贝构造函数。

如果显式声明了移动构造函数，默认拷贝构造函数将被定义为deleted，因此在这种情况下，如果没有显式定义拷贝构造函数，编译器将报错。

C++标准说明：
> If the class definition does not explicitly declare a copy constructor, one is declared implicitly. **If the class definition declares a move constructor or move assignment operator, the implicitly declared copy constructor is defined as deleted**; otherwise, it is defined as defaulted. The latter case is deprecated if the class has a user-declared copy assignment operator or a user-declared destructor. Thus, for the class definition
> ```cpp
> struct X {
>    X(const X&, int);
> };
> ```
> a copy constructor is implicitly-declared. If the user-declared constructor is later defined as
> ```cpp
> X::X(const X& x, int i =0) { /* ... */ }
> ```
> then any use of X’s copy constructor is ill-formed because of the ambiguity; no diagnostic is required.
