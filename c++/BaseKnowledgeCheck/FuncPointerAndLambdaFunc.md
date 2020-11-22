# 函数指针与lambda匿名函数

## 函数指针
* 基础用法：`double (*pf)(int);`，代表一个入参类型为`int`，返回类型为`double`的函数指针声明。与常见的函数声明方式`double func(int);`对比，我们可以发现在函数指针声明中，我们只是把函数名`func`用`*pf`替代，也就是说，`pf`即函数指针。需要注意的是，声明时需要用括号将`*`和`pf`放在一起，否则这个声明（`double *pf(int);`）的含义将变为：声明一个返回类型为`double`型指针，函数名为`pf`，入参类型为`int`的函数。
* 声明函数指针之后，我们可以将函数的地址赋值给它：
  ```cpp
  double func(int);
  double (*pf)(int);
  pf = func;
  ```
* 函数指针调用的逻辑也很简单：与声明的逻辑一致，即将`*pf`等价为函数名`func`，调用方式如下：
  ```cpp
  double func(int);
  double (*pf)(int);
  pf = func;
  double a = func(0);
  double b = (*pf)(0);
  double c = pf(0); // C++也支持使用这种方式调用函数指针所指向的函数，但上面的方法可以更加明确地强调正在使用函数指针。
  ```
* 声明函数指针数组：
  ```cpp
  double (*pf1)(int);
  double (*pf2)(int);
  double (*pf3)(int);
  double (*pf[3])(int) = {pf1, pf2, pf3};
  ```
* 使用typedef简化语法：
  ```cpp
  typedef double (*PF_TYPE)(int);
  PF_TYPE pf;
  double a = pf(0);
  double b = (*pf)(0);
  ```

## lambda函数
* 基础用法：`[](int x){return x % 3 == 0;};`，其中的`[]`即为被省略的函数名，这也是为什么lambda函数也被称为匿名函数的原因。需要注意的是，这里没有返回值，在只有一个return语句的lambda函数中，返回值由自动推断得出。而在其他情况下，需要使用返回类型后置语法指定返回值：`[](double x)->double{int y = x; return x = y;};`。
* 可以在中括号中捕获作用域中的变量，参考如下代码：
  ```cpp
  #include <iostream>
  using namespace std;

  int main()
  {
      int a = 1;
      int b = 2;

      auto func1 = [a, b](){return a+b;};
      auto func2 = [&a, b](){return a+b;};
      auto func3 = [a, &b](){return a+b;};
      auto func4 = [&](){return a+b;};
      auto func5 = [=](){return a+b;};

      a++;

      cout << func1() << endl; // 3
      cout << func2() << endl; // 4
      cout << func3() << endl; // 3
      cout << func4() << endl; // 4
      cout << func5() << endl; // 3
  }
  ```
