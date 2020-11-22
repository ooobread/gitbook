  
# 智能指针
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