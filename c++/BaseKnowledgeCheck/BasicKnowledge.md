# BasicKnowledge

## 整数数据类型
* short至少16位；int至少与short一样长；long至少32位，且至少与int一样长；long long至少64位，且至少与long一样长。
* bool类型中，true可以被转换为1，false可以被转换为0

## 浮点数
* 两种书写方法：
  * 标准小数点法：12.34，0.00023
  * E表示法：3.45E6，即3.45乘以10的6次方的结果。
* C++的3种浮点类型：`float`, `double`, `long double`是按照它们可以表示的有效数位和允许的指数最小范围来描述的。
* 对有效数位的要求：`float`至少32位；`double`至少64位，且不少于`float`；`long double`至少和`double`一样多。
* 指定浮点类型：
  * 默认情况下，12.34、3.45E6这样的浮点常量都属于`double`。
  * 使用`1.234f`格式来存储`float`类型浮点数。
  * 使用`1.234l`格式来存储`long double`类型浮点数。

## 数据类型转换
* 列表初始化不允许收窄转换，例如浮点数转为整型数。
* 通常来说`long`或者`long long`可能可以被转换为`int`，这取决于类型转换的目标变量能否容纳被转换的变量类型，例如`int a = (long)123`是合法的，但在**列表初始化**中，由于被转换的是变量，因此编译器无法确定转换的目标变量是否可以容纳，因此不合法。

## 表达式中的类型转换
* `bool`, `char`, `unsigned char`, `signed char`, `short`将被转换为`int`，这被称为**整型提升**。
* 如果`short`变量的长度小于`int`，则`unsigned short`将被转换为`int`，如果`short`变量的长度等于`int`，则`unsigned short`将被转换为`unsigned int`来防止数据损失。
* 当不同类型数据进行算数运算时，较小的类型将被转换为较大的类型，例如`9.0/5`，5将被转换为`double`再进行计算。
* C++通过校验表来确定表达式中数据类型的转换，C++11的校验表如下（按照执行优先级排列）：
  1. 若有一个操作数为`long double`，则将另一个操作数转换为`long double`；
  2. 若有一个操作数为`double`，则将另一个操作数转换为`double`；
  3. 若有一个操作数为`float`，则将另一个操作数转换为`float`；
  4. 若操作数均为整型，则先运用整型提升（`char`和`short`先被转换为`int`）；
  5. 若两个操作数均为有符号或无符号，则将低级别类型操作数转换为高级别类型；
  6. 若一个操作数为有符号，一个操作数为无符号，且无符号操作数类型级别高于有符号操作数，则将有符号操作数类型转换为无符号操作数所属的类型；
  7. 若有符号类型可以表示无符号类型的所有可能取值，则将无符号操作数类型转换为有符号操作数所属类型；
  8. 将两个操作数均转换为有符号类型无符号版本。

## 传参中的类型转换
* C++传参中的类型转换由函数原型控制，若没有函数原型控制，则遵循整型提升。其中为了与C兼容，在没有函数原型控制的情况下，`float`将被优先转换为`double`。

## 数组
* 数组初始化规则：
  * 只有在定义数组时才能初始化，`int arr[3] = {1, 2, 3}`
  * 如果只对数组一部分初始化，则编译器将其他元素设置为0。
  * 如果采用类似`int arr[] = {1, 2, 3}`，编译器将自动计算元素数量。例如前面的例子中元素数量为3。
  * C++11中可以使用列表初始化来初始化数组，其特点如下：
    * 初始化数组时可省略等号：`int arr[3] {1, 2, 3}`
    * 当大括号内没有东西时，将所有元素置为0：`int arr[3] {}`
    * 使用列表初始化来初始化数组时，禁止收窄转换。

## 联合体
* 其中所有变量共用起始内存

## 命名空间
* 命名空间中声明的名称的链接性为外部的（除非他引用了常量）。
* 命名空间是开放的，可以把名称添加到已有的命名空间中；也可以将声明与实现写在同名的命名空间中。正是因为这一点，多个文件中的命名空间不会发生冲突。

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

  ## 智能指针
  * `shared_ptr`，当同一块内存空间需要被多个指针同时指向时，选择这种类型。示例如下：
    ```cpp
    #include <memory>
    #include <iostream>
    using namespace std;

    int main()
    {
      shared_ptr<int> sp1(new int(22));
      shared_ptr<int> sp2 = sp1;
      cout << *sp1 << endl; // 22
      cout << *sp2 << endl; // 22
      sp1.reset();
      cout << *sp2 << endl; // 22
    }
    ```
  * `unique_ptr`，当同一块内存空间仅需要被一个指针指向时，选择这种类型。示例如下：
    ```cpp
    #include <memory>
    #include <iostream>
    using namespace std;

    int main()
    {
      unique_ptr<int> up1(new int(11));
      unique_ptr<int> up2 = up1; // 编译错误，unique_ptr无法复制
      cout << *up1 << endl; // 11，重载操作符*，可以用类似于普通指针的方式使用
      unique_ptr<int> up3 = move(up1); // 将up1转换为右值，来转移所有权，但转移之后，up1则失去对象内存所有权
      cout << *up3 << endl; // 11
      cout << *up1 << endl; // 运行时错误
      up3.reset(); // 显示释放内存
      up1.reset(); // 不会导致运行时错误
      cout << *up3 << endl; // 运行时错误
    }
    ```
  * `weak_ptr`，可以指向shared_ptr指针指向的对象内存，但不拥有该内存，也就不影响shared_ptr的引用计数。使用weak_ptr成员lock，则可以返回其指向内存的一个shared_ptr对象，且在所指向对象内存已经无效时，返回空指针。示例如下：
    ```cpp
    #include <memory>
    #include <iostream>
    using namespace std;

    void Check(weak_ptr<int>& wp)
    {
      shared_ptr<int> sp = wp.lock(); // 返回一个shared_ptr<int>
      if (sp != nullptr) {
        cout << "still" << *sp << endl;
      } else {
        cout << "pointer is invalid." << endl;
      }
    }

    int main()
    {
      shared_ptr<int> sp1(new int(22));
      shared_ptr<int> sp2 = sp1;
      weak_ptr<int> wp = sp1; // 指针wp指向shared_ptr<int> sp1所指对象
      cout << *sp1 << endl; // 22
      cout << *sp2 << endl; // 22
      Check(wp); // still 22
      sp1.reset();
      cout << *sp2 << endl; // 22
      Check(wp); // still 22
      sp2.reset();
      Check(wp); // pointer is invalid
    }
    ```