# 虚函数内存分布

## 问题原因

安全函数整改添加析构函数，安全规范中要求原则上析构函数必须是虚函数，保证类被继承时资源可以被正确释放。 其中一个类（这里成为msgClass）较为特殊，其事实上包含一个消息头加一个消息体（此类数据结构使用struct更为合理），类结构如下：

```cpp
class MsgClass {
public:
    MsgClass()
    {
        ...
    }
private:
    MsgHead msgHead;
    MsgBody msgBody;
};
```

发送前构造消息时获取相应长度序列化缓存serializableBuff，并且采用指针偏移的方式直接将消息头、消息体放入缓存，长度采用消息头长度+消息体长度的方式进行定义（sizeof\(msgHead\)+sizeof\(msgBody\)）。接收消息时，将收到的指针直接强转为msgClass\* 类型指针。 在原有的类结构中这种强转是正确的，内存将会按照msgHead-&gt;msgBody的方式进行分布。但是添加了虚析构函数之后（关键是虚函数，若仅添加析构函数不会有此问题），类结构如下：

```cpp
class MsgClass
{
    public:
    MsgClass()
    {
        ...
    }


    virtual ~MsgClass()
    {
        ...
    }
private:
    MsgHead msgHead;
    MsgBody msgBody;
};
```

编译器会为类所占用的内存空间的头部添加虚函数指针，导致内存分布发生改变，使得强转之后解析内存空间出现错位导致错误。

## 虚函数造成的内存分布改变

编译器不同对于虚函数指针在内存中的位置可能不同，此处使用visual studio 2019所用默认编译器为例。 为了在编译结果中看到内存分布，我们需要在编译命令中加入相关命令： 右键点击相关源文件-&gt;选择属性-&gt;选择C/C++-&gt;选择 命令行-&gt;在其中选项中加入”/d1 reportAllClassLayout”（不包含双引号） 我们以上述类结构为例进行改造：

```cpp
struct MsgHead
{
    int a;
    int b;
};


struct MsgBody
{
    int c;
    int d;
};


class MsgClass
{
public:
    MsgClass(){}


private:
    MsgHead msgHead;
    MsgBody msgBody;
};


int main()
{
    MsgClass msgClass;
    return 0;
}
```

查看编译结果：

```bash
1....
2.**1>class MsgClass size(16):
3.1> +---
4.1> 0 | MsgHead msgHead
5.1> 8 | MsgBody msgBody
6.1> +---**
7....
```

我们可以看到此处的内存分布与结构体类似，根据声明的顺序摆放两个结构体位置。 而如果加入了析构函数呢？

```cpp
class MsgClass
{
public:
    MsgClass(){}
    ~MsgClass(){}
private:
    MsgHead msgHead;
    MsgBody msgBody;
};


int main()
{
    MsgClass msgClass;
    return 0;
}
```

查看编译结果：

```bash
1....
2.**1>class MsgClass size(16):
3.1> +---
4.1> 0 | MsgHead msgHead
5.1> 8 | MsgBody msgBody
6.1> +---**
7....
```

我们会发现结果一致，即内存分布一致，添加成员函数（包括析构函数）并不会改变内存分布。 而如果我们将析构函数定义为虚函数呢？

```cpp
class MsgClass
{
public:
    MsgClass(){}
    virtual ~MsgClass(){}
private:
    MsgHead msgHead;
    MsgBody msgBody;
};


int main()
{
    MsgClass msgClass;
    return 0;
}
```

查看编译结果：

```bash
1....
2.1>class MsgClass size(20):
3.1> +---
4.1> 0 | {vfptr}
5.1> 4 | MsgHead msgHead
6.1>12 | MsgBody msgBody
7.1> +---
8.1>MsgClass::$vftable@:
9.1> | &MsgClass_meta
10.1> | 0
11.1> 0 | &MsgClass::{dtor}
12.1>MsgClass::{dtor} this adjustor: 0
13.1>MsgClass::__delDtor this adjustor: 0
14.1>MsgClass::__vecDelDtor this adjustor: 0
15....
```

我们会发现内存布局、占用内存的大小都发生了改变，编译器会将虚函数指针放在内存的开头从而导致内存分布的改变。 不止如此，我们还能观察到随着虚函数的引入，内存中还会添加虚表，位置在类所占用内存之后，这部分内容会在随后章节中讨论。 事实上还有一个疑问，如果类中有多个虚函数，内存占用会再次改变吗？它的分布是什么？ 我们为这个类再添加一个虚函数：

```cpp
class MsgClass
{
public:
    MsgClass(){}
    virtual ~MsgClass(){}
    virtual void setMsgHead(){}
private:
    MsgHead msgHead;
    MsgBody msgBody;
};


int main()
{
    MsgClass msgClass;
    return 0;
}
```

查看编译结果：

```bash
1....
2.1>class MsgClass size(20):
3.1> +---
4.1> 0 | {vfptr}
5.1> 4 | MsgHead msgHead
6.1>12 | MsgBody msgBody
7.1> +---
8.1>MsgClass::$vftable@:
9.1> | &MsgClass_meta
10.1> | 0
11.1> 0 | &MsgClass::{dtor}
12.1> 1 | &MsgClass::setMsgHead
13.1>MsgClass::{dtor} this adjustor: 0
14.1>MsgClass::setMsgHead this adjustor: 0
15.1>MsgClass::__delDtor this adjustor: 0
16.1>MsgClass::__vecDelDtor this adjustor: 0
17....
```

我们发现类的内存占用和分布并没有改变，而虚表的内容发生了改变，加入了虚函数setMsgHead\(\)。这很好理解，虚函数指针指向的事实上是虚表，而虚函数的增加减少将会反映在虚表中，除非类中虚函数数量为0。

## 虚表

&lt;== ToBeContinued ==

## 总结

无论出于何种缘由，安全整改也好代码重构也好，修改前人代码总归是一项非常折磨人的工作，无论是需要考虑业务的一致性、正确性，还是由于种种原因需要对代码原有结构进行一定程度上的破坏与重构，都能够很好的磨炼心性，同时也能够领略到各式各样的神仙代码（手动狗头）。总之，针对老代码的修改，以下几点是需要注意的：

* 修改代码需要注意业务一致性，在一些场合中其优先级甚至高于保证业务的正确性，尤其是对于已经稳定工作一段时间的代码。例如添加异常场景判断并返回的代码，某些场景理应属于异常场景，但是由于代码已经按照这个逻辑稳定运行并且在其之上可能还承载了其他接口和功能，此时就需要将这样的场景从异常场景中剔除出去，留待未来修改。
* 修改类、结构体时需要额外注意，这里的修改包括且不限于增加、删除成员变量，添加虚函数等。这些变化不会造成编译错误，但可能导致业务逻辑出现问题。尤其需要注意消息类和结构体，由于其需要被大量传递的特性，在存量代码中可能有一些对其的裸指针操作，这些操作可以理解成在是在严格确认了操作对象内存构造的情况下进行的精密操作，因此任何对这样的操作对象进行修改都有可能造成系统连锁式的崩塌。
* 可能的情况下，尽量消灭所谓的数据类，也就是只有成员变量没有成员函数的类，转而使用结构体替代（个人看法）。由于C++的历史包袱，结构体被赋予了极强的功能，与类几乎没有区别，也就造成了一定程度的混乱，例如在结构体中写函数的做法就会让人困惑。因此个人的看法是如果只有成员变量，只是为了聚合数据，那么使用结构体足以。若未来需要进行功能扩展，将结构体扩展为类成本也是极低（只是需要注意上一条的问题），不需要在一开始就预留空间，否则就大概属于过度设计了。

