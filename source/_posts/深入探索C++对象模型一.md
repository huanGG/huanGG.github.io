---
title: 深入探索C++对象模型一
tags:
  - 读书笔记
  - C++
permalink: In-depth-exploration-of-the-C-++-object-model-1
translate_title: indepth-exploration-of-the-c-object-model
date: 2017-07-31 10:00:00
---

# 什么是C++对象模型
* 语言中直接支持面向对象程序设计的部分
* 对于各种支持的底层实现机制（其具体实现依赖于编译器的策略）     

书的内容侧重于第二点。    
<!--more-->
# chapter 1:    关于对象     
## 与 C 相比，加上封装后的成本    
类的封装没有增加任何成本（内存对齐在此被忽略）。C++ 在布局及存取时间上主要的额外负担是由  virtual 引起的，包括
* virtual functions 机制，用以执行动态绑定（runtime binding），即保存 vtable 和通过 vtable 找到函数地址；
* virtual base class 用以实现“多次出现在继承体系中的基类,有一个单一而被共享的实例”，通过指针来找到基类的成员。    

此外，还有多重继承下的额外负担，发生在“一个派生类和其第二或后继的基类”的转换。

## C++ 对象模型
![](https://blog-assert-1253725031.cos.ap-shanghai.myqcloud.com/c++_object_model.png)    
Nonstatic data members 被配置于每一个 class object 之内；    
Static data members 则被存放在所有的 class object 之外；    
Static 和 Nonstatic function members 也被放在所有的 class object 之外。    
通过两个步骤支持 virtual functions：
* 每个 class 产生出一堆指向 virtual functions 的指针，放在 vtbl (virtual table)中。
* 每一个 class object 被安插一个指针，指向相关的 vtbl。通常这个指针被称为 vptr。    

特别地，一个 vtbl 对应 一个 class ， 一个 vptr 对应 一个 class object。
RTTI（runtime type identification): 每一个 class 相关联的 type_info 对象的指针通常也保存在 vtbl 的第一个 slot 中。    
## 引入继承后的对象模型成本：
* 如果是普通的继承，父对象被直接包含在子对象里面，这样对父对象的存取也是直接进行的，没有额外的成本；
* 如果是虚拟继承，则父对象会由一个指针被指出来，这样的话对父对象的存取就添加了一层间接性，必须经由一个指针来访问，添加了一次间接的额外成本。缺点是因为间接性导致的空间和存取时间上的额外负担，优点是派生类的大小不会因为基类而改变(在 VC 下测试时会改变)。     
C++优先判断一个语句为声明：当语言无法区分一个语句是声明还是表达式时（比如 int (*pq)() ），就需用用一个超越语言范围的规则 —— C++优先判断为声明。    

## struct 和 class 关键字的意义：
* 它们之间在语言层面（除了模板声明时）并无本质的区别，更多的是概念和编程思想上的区别。
*  struct 用来表现那些只有数据的集合体 POD（Plain OI' Data）、而 class 则希望表达的是 ADT（abstract data type）的思想；
* 由于这 2 个关键字在本质是无区别，所以 class 并没有必须要引入，但是引入它的确非常令人满意，因为这个语言所引入的不止是这个关键字，还有它所支持的封装和继承的哲学；
* 可以这样想象：保留 struct 一方面为了与 C 兼容，另一方面方便 C 程序员迁徙到 C++的用途了。

## C++只保证处于同一个 access section 的数据，一定会以声明的次序出现在内存布局当中。
C++标准只提供了这一点点的保证。对于跨section的，依赖于编译器的实现。

## 与 C 兼容的内存布局： 组合，而非继承，才是把 C 和 C++ 结合在一起的唯一可行的方法。
只有使用组合时，才能够保证与 C 拥有相同的内存布局，使用继承时的内存布局是不受 C++ Standard 所保证的（很多编译器也可行，但是标准未定义！）。

## C++支持三种形式的编程风格（或称典范 paradigm）：
* 面向过程的风格：就像 C 一样，一条语句接一条语句的执行或者函数跳转；
* 基于对象的风格（object-based）（或称 ADT）： 仅仅使用了 class 的封装，很多人都是在用基于对象的风格却误以为自己在使用面向对象的风格；
* 面向对象的风格（object-oriented）， 使用了 class 的封装和多态的编程思维（多态才是真正的面向对象的特征）。
* 纯粹以一种 paradigm 写程序，有助于整体行为的良好稳固。

## 一个对象的内存布局大小（通常由 3 部分组成）：
* 其 nonstatic data member 的总和大小；
* 任何由于内存对齐所需要的填补上去的空间；
* 加上了为了支持 virtual 机制而引起的额外负担。    
 这也印证了前面的一个结论：C++中的额外成本通常都是由于 virtual 机制所引起的。    

## 指针的类型：
* 对于内存来说，不同类型的指针并没有什么不同。它们都是占用一个 word 的大小（ps:在不同的机器上占用大小不一样，最好使用 sizeof ），包含一个数字，这个数字代表内存中的一个地址；
* 感觉上，指针的类型是编译器的概念，对于硬件来说，并没有什么指针类型的概念；
* 转型操作也只是一种编译器的指令，它改变的是编译器对被指内存的解释方式而已！   

## 多态只能由“指针” 或 “引用” 来实现，根本原因在于：
* 指针和引用（通常以指针来实现）的大小是固定的（一个 word），而对象的大小却是可变的。其类的指针和引用可以指向（或引用）子类，但是基类的对象永远也只能是基类，没有变化则不可能引发多态。
* 一个指针或引用绝不会引发任何与类型有关的内存委托操作 ，在指针类型转换时会受到的改变的只有它们所指向内存的解释方式而已。（例如指针绝不会引发 slice，因为它们大小相同）
* 在初始化、 assignment 等操作时，编译器会保证对象的 vptrs 得到正确的设置。这是编译器的职责，它必须做到。一般都是通过在各种操作中插入编译器的代码来实现的。   

# chapter 2: 构造函数语意学
> 单一参数的构造函数有时会被隐式转换。C++提供了关键字 explicit，可以阻止单一参数的构造函数被当作一个 conversion 运算符，或者说声明为 explicit 的构造函数不能在隐式转换中使用。    

##  Default Constructor 的构造操作
### C++中对于默认构造函数的解释
默认的构造函数会在需要的时候被编译器产生出来。这里非常重要的一点是：谁需要？是程序的需要还是编译器的需要？如果是程序的需要，那是程序员的责任；只有在是编译器的需要时，默认构造函数才会被编译器产生出来，而且被产生出来的默认构造函数只会执行编译器所需要的行动，而且这个产生操作只有在默认构造函数真正被调用时才会进行合成。    
例如：成员变量的初始化为 0 操作，这个操作就是程序的需要，而不是编译器的需要。

### 区分 trivial 和 notrivial： 
* 只有编译器需要的时候，合成操作才是 nontrivial 的，这样的函数才会被真正的合成出来；
* 如果编译器不需要，而程序员又没有提供，这时的默认构造函数就是 trivial 的。虽然它在概念上存在，但是编译器实际上根本不会去合成出来，因为他不做任何有意义的事情，所以当然可以忽略它不去合成。trivial 的函数只存在于概念上，实际上不存在这个函数。   

### 变量的初始化
只有全局变量和静态变量才会保证初始化。Golbal objects 的内存保证会在程序激活的时候被清 0；Local objects 配置于程序的堆栈中，Heap objects 配置于自由空间中，都不一定会被清为 0,它们的内容将是内存上次被使用后的痕迹！

### 类声明头文件可以被许多源文件所包含，如何避免合成默认构造函数、拷贝构造函数、析构函数、赋值拷贝操作符（4 大成员函数）时不引起函数的重定义？    
解决方法是以 inline 的方式完成，一个 inline 函数有静态链接，不会被文件以外者看到。如果函数太复杂不适合 inline，就会合成一个 explicit noninline static 实体（static 函数独立于编译单元）。

### 带有默认构造函数类的成员对象、带有默认构造函数的基类    
* 如果 class A 内含一个或以上的 member objects，那么 A 的 constructor 必须调用每一个 member class 的默认构造函数。    
* 如果 class A 派生自一个带有默认构造函数的基类，则 A 的  default constructor 会被视为 nontrivial，因此需要被合成出来。    
具体方法是： 编译器会扩张 constructors ，在其中安插代码使得在 user code 被调用之前先调用 member objects 的默认构造函数（当然如果需要调用基类的默认构造函数，则放在基类的默认构造函数调用之后：基类构造函数 -> 成员构造函数 ->user code）。
C++要求以“member objects 在 class 中的声明次序”来调用各个 construtors。这就是声明的次序决定了初始化次序（构造函数初始化列表一直要求以声明顺序来初始化）的根本原因！

### 带有 virtual functions 的类、带有 virtual base class 的类    
* 带有 virtual functions 的类的默认构造函数是 notrivial 的，需要编译器安插额外的成员 vptr 并在构造函数中正确的设置好 vptr，这是编译器的重要职责之一。   
* 带有 virtual base class 的类的默认构造函数同样也是 notrivial，编译器需要正确设置相关的信息以使得这些虚基类的信息能够在执行时准备妥当，这些设置取决于编译实现虚基类的手法。    

上述 4 种情况会使得编译器真正的为 class 生成 nontrivial 的默认构造函数。这个 nontrivial 的默认构造函数只满足编译器的需要（调用 member objects 或 base class 的默认构造函数、初始化 virtual function 或 virutal base class 机制）。其它情况时，类在概念上拥有默认构造函数，但是实际上根本不会被产生出来（前面的区分 trivial 和 notrivial）。

### C++新手常见的 2 个误区：
 * ERROR: 如果 class 没有定义 default constructor 就会被合成一个。    
首先定义了其它的 constructor 就不会合成默认构造函数，再次即使没有定义任何构造函数也不一定会合成 default constructor，可能仅仅是概念上有，但实际上不合成出来。
 * ERROR: 编译器合成出来的默认构造函数会明确设定每一个 data member 的默认值。    
明显不会，区分了 Global objects, Stack objects, Heap objects 就非常明白了只有在 Global 上的 objects 会被清 0，其它的情况都不会保证被清 0。

## 拷贝构造函数的构造操作
Copy constructors 和默认构造函数一样，只有在必须的时候才会被产生出来，对于有些类而言，拷贝构造函数仅仅需要按位拷贝就可以。满足 bitwise copy semantics 的拷贝构造函数是 trivial 的，就不会真正被合成出来（与默认构造函数一样，只有 nontrivial 的拷贝构造函数才会被真正合成出来）。
对多数类按位拷贝就够了，什么时候一个 class 不展现出 bitwise copy semantics 呢？分为 4 中情况，前 2 种很明显，后 2 种是由于编译器必须保证正确设置虚机制而引起的。
   * 当 class 内含一个 member object 而后者声明了（也可能由于 nontrivial 语意从而编译器真正合成出来的）一个 copy constructor 时；
   * 当 class 继承自一个存在有 copy constructor 的 base class（同样也可能是合成）时；
   * 当 class 声明了一个或多个 virtual functions 时；（vf 影响了位语意，进而影响效率）
   * 当 class 派生自一个继承串链，其中一个或多个 virtual base classes 时。    

## 程序转化语意学
### 显式的初始化操作
对于下面的三个定义，都明显地以 x0 来初始化其 class object:    
{% codeblock  lang:cpp %}
void foo_bar()
{
    X x1(x0);
    X x2 = x0;
    X x3 = X(x0);
}
{% endcodeblock %}    
必要的程序转化有两个阶段：   
    1. 重写每一个定义，其中的初始化操作会被剥除；
    2. class 的 copy constructor 调用操作会被安插进去。    

在明确的双阶段转化后，foo_bar()可能看起来像这样：    
{% codeblock  lang:cpp %}
int main()
{
    X x1;   //定义被重写，初始化操作被剥除
    X x2;
    X x3;
    
    //编译器安插 X copy construction 的调用操作
    x1.X::X(x0);
    x2.X::X(x0);
    x3.X::X(x0);
    //避免了临时变量的产生，提升了效率
}
{% endcodeblock %}   

### 参数的初始化   
采用“拷贝建构”（copy construct）的方式把参数直接建构在其应该的位置上。   
### 返回值的初始化     
NVR（named return value） 优化：编译器会把返回值作为一个参数传到函数内，比如:X foo() {…}会被自动更改（也可以自己手动做这个优化）为：void foo(X &__result) { … }。    

### 不要随意提供 copy constructor
对于满足 bitwise copy semantics 的类来说，编译器自动生成的拷贝构造函数自动地使用了位拷贝（这是效率最高的），如果你自己随意提供 copy constructor 就会压抑掉编译器的这个行为，画蛇添足还影响了效率。 

## 成员们的初始化队伍
### 成员初始化列表：   
编译器会一一操作初始化列表，把其中的初始化操作以 member 声明的次序在 constructor 内安插初始化操作，并且在任何 explicit user code 之前。member 声明的次序与初始化列表中的排列次序不一致” ，可能会导致一些不明显的 Bug。    
据说 GCC 已经可以强制要求使用声明次序来进行初始化以避免了这个陷阱。    
理解了初始化列表中的实际执行顺序中以“ member 声明的次序”来决定的，就可以理解一些很微妙的错误了。比如：
{% codeblock  lang:cpp %}
A(int val) :  j(66),i(j) {… }
int i,  j;
{% endcodeblock %}   
i 依赖于 j，由于 j 声明在 i 之后，就产生了使用未初始化成员的错误。

# chapter 3: Data语意学
空类大小并不为0，它有隐藏的 1 byte 大小，是被编译器安插进去的一个 char。以确保一个 class 的两个 objects 在内存中有独一无二的位置。     
类的大小受以下三方面影响：    
* 语言本身造成的额外负担。当语言支持 virtual base classes 时，就会导致一些额外负担。
* 编译器对于特殊情况所提供的优化处理。视编译器而定，不同编译器会采取不同的策略。机器的位数（32还是64）对类的大小也会有影响。
* Alignment 的限制，即内存对齐。在 32 位计算机上，alignment 通常为 4 bytes。

## Data Member的绑定
对 member functions 本身的分析会直到整个 class 的声明都出现了才开始。所以 class 的 member functions 可以引用声明在后面的成员，C 语言就做不到。    
和上面对比，需要十分注意的一点是：class 中的 typedef 并不具备这个性质。因此，类中的 typedef 的影响会受到函数与 typedef 的先后顺序的影响。
{% codeblock  lang:cpp %}
typedef int length;
class Point3d{
public:
 void f1(length l){ cout << l << endl; }
 typedef string length;
 void f2(length l){ cout << l << endl; }
};
{% endcodeblock %}   
这样 f1 绑定的 length 类型是 int；而 f2 绑定的 length 类型才是 string。所以，对于 typedef 需要防御性的程序风格：始终把 nested type 声明（即 typedef）放在class 起始处！
## Data Member的布局
C++ 只规定在同一个 access section 中，较晚出现的 member 有较高的地址。传统上，vptr 被安放在所有被明确声明的 member 的最后，不过也有些编译器把 vptr 放在最前面（据说 MSVC++ 就是把 vptr 放在最前面，而 GCC 是把 vptr 放在最后面）。
## Data Member 的存取
由一个对象或由一个指针来存取一个 static  member ，效率是完全一样的，都会被编译器转化为 Class:: 的形式。    
而由一个对象存取一个  nonstatic member 会比指针直观上更快捷一些，但在没有虚拟继承的情况下，其存取效率应该是相同的，因为每一个 nonstatic data member 的偏移位置在编译期即可确定。    
而在有虚拟继承的情况下，我们不能确定指针的 class type，也就无法在编译期确定 member 真正的 offset，所以必须延迟至执行期，经由一个额外的间接引导，才能够解决。而如果使用对象，其类型是确定的，编译器可以在编译期确定其位置。    
     
经由一个函数调用的结果来存取静态成员，C++ 标准要求编译器必须对这个函数进行求值，虽然这个求值的结果并无用处。    
```
foo().static_member = 100;    
```
foo() 返回一个类型为 X 的对象，含有一个 static_member，foo() 其实可以不用求值而直接访问这个静态成员，但是 C++标准保证了 foo() 会被求值，可能的代码扩展为：    
```
     (void) foo();
    X::static_member = 100;    
```
对一个 nonstatic data member 进行存取操作，编译器会进行如下扩展：
 ```   
    origin.y_ = 10;       扩展为      &origin + (&Point3d::y_ - 1)    
 ```
注意其中的-1 操作：指向 data member 的指针，其 offset 值总是被加上 1。这样可以使编译系统区分出“一个指向 data member 的指针，用以指向 class 的第一个 member”和“一个指向 data member 的指针，但是没有指向任何 member”两种情况（成员指针也需要有个表示NULL 的方式，0 相当于用来表示 NULL 了，其它的就都要加上 1 了）。     
PS：目前来说，现在编译器都已经不需要再减1，类的起始地址即是类的第一个成员变量的地址。
{% codeblock  lang:cpp %}
    //还是之前的 C1 类，现在去取其成员变量的地址
    //cout << &c1::a_ << endl;  //这样打印指针是不对的，在最新的编译器上，这行和下行打印的都是1
	//cout << &c1::b_ << endl; 
	printf("%p\n", &c1::a_) ;   //这里输出的是 00000000，即该成员变量在类中的偏移
	printf("%p\n", &c1::b_) ;   //这里输出的是 00000004
{% endcodeblock %}   

## 继承与 Data Member    
派生类的成员和基类成员的排列次序并未在 C++ Standard 中强制指定；理论上编译器可以自由安排， 但是对于大部分的编译器实现来说，都是把基类成员放在前面，但是 virtual base class 除外。（一般而言，任何一条规则一旦碰到 virtual base class 就没辙了）。    
 一般而言，具体继承（相对于虚拟继承）并不会增加时间和空间上的负担，C++中的额外成本大多来自于 virtual 机制。    
 C++ Standard 保证：“出现在派生类中的 base class subobject 有其完整原样性”。    

### 支持多态所带来的 4 个负担：
   * 导入 virtual table 用来存放每一个 virtual functions 的地址。这个 Table 的元素数目一般而言是被声明的 virtual functions 数目再加上一个或两个 slots（用以支持 RTTI）；
   * 在每一个 class object 中安插一个 vptr 指向相应的 vtable；
   * 在 constructor 中安插代码以正确设置 vptr；
   * 在 destructor 中安插代码以正确设置 vptr；

### 单一继承并且没有 virtual function 
{% codeblock  lang:cpp %} 
class C
{
    private:
    int val;
    char c1;
    char c2;
    char c3;
}
{% endcodeblock %}  
在一台32位的机器中，类 C 的大小的 8 bytes，其中 val 占用 4 bytes，c1、c2 和 c3 各占用 1 byte，对齐需要 1 byte。
{% codeblock  lang:cpp %}
class C1
{
private:
	int a;
	char b;
};
class C2 :public C1
{
private:
	char c;
};
class C3 :public C2
{
private:
	char d;
};
{% endcodeblock %}     
而在分裂为三层结构后，C1、C2、C3的大小分别为 8、12、16。为什么要采取这样的边界调整呢？     
现在假设在没有上述边界调整的情况下，有如下声明：
```
    C2* p2;
    C1* p11,p12;
```
若让 p11 指向 C2 对象
 ```   p11 = p2; ```   
然后再执行
 ```   *p12=*p11; ```    
原本的语意是将 p11 指向的 C2 对象的 C1 subject 赋值给 p12 指向的对象的 C1 subject部分。然而现在 C2 中的 c 与 C1 对象“捆绑”在一起，那么 p12 对象的 c 位置将会被覆盖掉，产生预期外的影响。    

### 单一继承并含有虚拟函数时的内存布局（书上说考虑一般把 vptr 放在尾部的设计）
单一继承时 vptr 被放在第一个子类的末尾，产生这样布局的根本原因在于“基类的完整性必须在子类中得以保存”。这个名叫 vptr\_Point2d 的 vptr 可以这样理解，由 Point2d 而引发的 vptr，在 Point2d 的对象中，这个 vptr\_Point2d 所指向的是与 Point2d 所对应的 point2d\_vtable，而在 Point3d的对象中，这个 vptr\_Point2d 所指向的却是与 Point3d 所对应的 point3d_vtable。
![](https://blog-assert-1253725031.cos.ap-shanghai.myqcloud.com/SingleInheritanceWithVirtualFunction.png)
下面是一些测试：
{% codeblock  lang:cpp %} 
class A
{
public:
	char x;
	virtual void func();
};

class B :public virtual A
{
	virtual void foo();
public:
    int a;
}

class C:public A
{
   virtual void bar();
}
class D:public A
{
}
int main()
{
    printf("%d\n", sizeof(A)); //8
	printf("%p\n", &A::x);     //00000004  说明 vptr 放在类的首部
	printf("%d\n", sizeof(B));     //20 ，基类中一个 vptr，一个 char型变量 x，占用 8 个 bytes；子类中一个 vptr，一个 int 型变量 a，一个指向基类的指针，占用 12 字节
	printf("%p\n", &B::x);         //00000004   仍然是其在基类中的偏移，说明子类中有一个指向基类的指针
    printf("%p\n", &B::a);         //00000004 
    printf("%d\n", sizeof(C)); //8
	printf("%p\n", &C::x);     //00000004
    printf("%d\n", sizeof(D)); //8
	printf("%p\n", &D::x);     //00000004
}
{% endcodeblock %}  

### 多重继承时的内存布局（多重继承时的主要问题在于派生类与非第 1 基类之间的转换）：
![](https://blog-assert-1253725031.cos.ap-shanghai.myqcloud.com/multipleInheritance.png)
在多重继承的派生体系中，将派生类的地址转换为第一个（最左）基类是成本与单继承是相同的，只需要改换地址的解释方式而已；而对于转换为非第 1 基类的情况，则需要对地址进行一定的调整操作才行。    
 C++ Standard 并未明确 base classes 的特定排列次序，但是目前的编译器都是按照声明的次序来安放他们的。（有一个优化：如果第 1 基类没有 vtable 而后继基类有，则可能把它们调个位置）。多重继承中，可能有多个 vptr 指针，视其继承体系而定：派生类中 vptr 的数目最等于所有基类的 vptr 数目的总和。

### 虚拟继承：
虚拟继承把一个类切割为 2 部分：一个不变局部和一个共享局部。     
这个共享局部必须通过编译器安插的一些指针指向 virtual base classobject 来间接的存取，这样才能够实现共享。对于这个安插指针来实现共享的技术，有两种主流的做法。     
一种做法是直接使用一个指针指向虚基类：    
![](https://blog-assert-1253725031.cos.ap-shanghai.myqcloud.com/VirtualInheritance1.png)

另一种做法是在 vtable 中放置 virtual base class 的 offset：
![](https://blog-assert-1253725031.cos.ap-shanghai.myqcloud.com/VirtualInheritance2.png)

这种使用偏移地址的方式好处在于： vptr 是已经存在的成本，而 vtable 是 class 的所有 objects 所共享的成本。对于每一个 class object 没有引入任何的额外成本，仅仅在 vtable 多存储了一个 slot 布局，而前一种方式却对每一个 object 又再引入了一个指针成本。这两种方式都是把虚基类放在内存中模型中的最后面，然后借由一层间接性（指针或 offset）来访问。    

其实虚拟继承并不难理解，_就是把普通的继承中的直接访问添加了一层间接访问布局_，但具体实现依赖编译器的策略。    
普通封装不会带来任何执行期的成本，编译器可以轻松优化掉普通封装带来的任何成本。但是一旦涉及到虚拟继承，效率就会大幅降低，在有 n 层的虚拟继承体系中，普通的访问就要经
过 n 次间接，普通访问的成本就变为了 n 倍。    
再次强调，C++中的额外成本基本都是由于 virtual 机制引起的，所以尽量避免使用虚继承。如果一定要用，也尽量使用一个抽象的 virtual base class（类似于 Java 中的接口） ，没有任何 data members。

### 补充：重载、覆盖和隐藏
既然说到了继承的问题，那么不妨讨论一下经常提到的重载(overload)，覆盖(override)和隐藏(hide)，这是 Java 初学者在学习 C++ 的时候非常困惑的：
#### 重载的条件 (与 Java 类似)
* 相同的范围； 
* 函数名字相同； 
* 参数不同； 
* virtual 关键字可有可无。 
#### “覆盖”是指派生类函数覆盖基类函数，是实现多态的关键，条件是：
* 函数名字相同； 
* 参数和返回值相同； 
* 基类函数必须有virtual 关键字。 
#### “隐藏”是指派生类的函数屏蔽了与其同名的基类函数，条件是：
* 派生类的函数与基类的函数同名，参数不同，不论有无virtual关键字，基类的函数将被隐藏 ；
* 派生类的函数与基类的函数同名，参数相同，基类函数没有virtual 关键字，基类的函数被隐藏。

{% codeblock  lang:cpp %} 
class A
{
public:
	void print()
	{
		cout << "this is A" << endl;
	}
	virtual void print2()
	{
		cout << "this is A" << endl;
	}
};

class B :public A
{
public:
	void print()
	{
		cout << "this is B" << endl; //隐藏，无法实现多态
	}
	virtual void print2()
	{
		cout << "this is B" << endl;
	}
};

int main()
{
	A* a=new A;
	A* b=new B;
	a->print();     //this is A
	b->print();     //this is A，所以多态必须通过虚函数来实现
	a->print2();    //this is A
	b->print2();    this is  B
	return 0;
}
{% endcodeblock %}  
