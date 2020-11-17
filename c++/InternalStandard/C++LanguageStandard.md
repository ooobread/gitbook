# C++通用语言规范

* C++11后推荐使用using来定义类型：
  ```cpp
  using u4 = char_t;
  using u8 = short_t;
  ```
* 成员变量命名的三种风格，这三种风格仅适用于class的成员，struct/union的成员仍然与局部变量命名风格保持一致：
  ```cpp
  class Foo {
  private:
    std::string fileName;
    std::string m_fileName;
    std::string fileName_;
  };
  ```
* 建议行宽不超过120个字符
* 