# C++

## C++内存管理常考察点

### 1、 C++的构造函数，复制构造函数，和析构函数。什么是深复制和浅复制，构造函数和析构函数哪个能写成虚函数，为什么

#### 1.1 复制构造函数

```cpp
    Complex::Complex(const Complex & c){}
```

   * **复制构造函数的参数必须是引用**
  因为调用拷贝构造函数的时候是实参向形参传值，如果传进来的不是引用，那么就是值传递，那么就会在函数里又重新创建一个对象，而重新创建又是通过调用拷贝构造函数，所以如果不是引用的话，就会**一直调用**下去。
* **复制构造函数的参数需要加上 const**
  加上 const，防止对引用类型参数值的意外修改。

####    1.2 深拷贝与浅拷贝

  + **浅拷贝**
    又称值拷贝，将源对象的值拷贝到目标对象中去，本质上来说源对象和目标对象**共用一份实体**，只是所引用的**变量名不同**，**地址其实还是相同的。**

    有时候会导致析构函数企图**释放一块已经被释放的内存区域**，程序将会崩溃。

  + **深拷贝**
    拷贝的时候先开辟出和源对象大小一样的空间，然后将源对象里的内容拷贝到目标对象中去，这样**两个指针就指向了不同的内存位置**，并且里面的内容是一样的。

#### 1.3 构造函数和析构函数哪个能写成虚函数，为什么

  + **构造函数为什么不能是虚函数**
    存储空间角度：虚函数对应一个 vtable 虚函数表，vtable 存储于**对象的内存空间**，若构造函数是虚的，则需要通过 vtable 来调用，若对象还未实例化，即内存空间还没有，无法找到 vtable。

    使用角度：虚函数主要用于在信息不全的情况下，能使重载的函数得到对应的调用。构造函数本身就是要初始化实例，那使用虚函数就**没有实际意义**。

    实际含义：在调用构造函数时还不能确定对象的真实类型（因为子类会调父类的构造函数），而且构造函数的作用是提供初始化，在对象生命期只执行一次，不是对象的动态行为，也没有太大的必要成为虚函数。

  + **析构函数为什么需要是虚函数**
    用派生类类型指针绑定派生类实例，析构的时候，不管基类析构函数是不是虚函数，都会正常析构。

    用**基类类型指针绑定派生类实例**，析构的时候，如果基类析构函数不是虚函数，则只会析构基类，不会析构派生类对象，从而造成**内存泄漏**。

### 2、C++数组，链表，二叉树的内存排列是什么样的，结构体占多大内存如何计算，类占用多大空间如何计算，空类的空间是多少，为什么

####    2.1 结构体占多大内存如何计算

  + **内存对齐**
    结构体(struct)的 sizeof 值，并不是简单的将其中各元素所占字节相加，而是要考虑到存储空间的字节对齐问题。

    1、结构体成员变量的首地址偏移量能够被其对齐字节数大小所整除。

    2、结构体每个成员相对结构体首地址的偏移都是成员大小的整数倍 ，如不满足，对前一个成员填充字节以满足。

    3、结构体的总大小为结构体对最大成员大小的整数倍 ，如不满足，最后填充字节以满足。

    4、如果是结构体内嵌套了结构体的情况，嵌套的结构体对齐到自己成员最大对齐数的整数倍，结构体的整体大小就是所有最大对齐数（含嵌套结构体的对齐数）的整数倍。

    因此，在使用结构体的时候，占用空间小的元素应集中在一起声明可以节省内存。

  + **静态成员（static）**
    静态成员存放在全局数据段，不影响结构体实际的大小。

  + **虚函数**
    虚函数表就占用 4 个字节，当结构体中含有虚函数时，无论是否是继承来的，无论数量，那么都会产生一个 4 字节的指针指向虚函数表。

####    2.2 类的占用空间和空类的大小

  **空类大小为 1字节**，因为 C++标准规定空类也可以实例化，每个实例在内存中都有一个独一无二的地址，为了达到这个目的，编译器往往会给一个空类隐含的加一个字节，这样空类在实例化后在内存得到了独一无二的地址。

  + 2.2.1 类的大小为**非静态成员数据**的类型大小之和，静态成员存放在全局数据段，不影响类实例化的大小。
  + 2.2.2 类本身的一些特性占用的大小，虚函数表就占用 4 个字节，当类中含有虚函数时，无论是否是继承来的，无论数量，那么都会产生一个 4 字节的指针指向虚函数表。
  + 2.2.3 为优化存取，会产生字节对齐问题，具体机制与结构体相同。
  + 2.2.4 类的成员函数不会占用空间。

### 3、虚函数和虚表的原理是什么（重点）

* 虚函数的地址存放于虚函数表之中。运行期多态就是通过虚函数和虚函数表实现的。

  类的对象内部会有指向类内部的虚表地址的指针，通过这个指针调用虚函数。

  虚函数的调用会被编译器转换为对虚函数表的访问：

```cpp
ptr->f(); //ptr代表this指针，f是虚函数
*(ptr->vptr[1])(ptr);
```

* 上述代码中，ptr 代表一个 this 指针，ptr 指向的 vptr 是类内部的虚表指针。这个虚表指针会被放在类的最前方（VS2017）。

  1 就是虚函数指针（vtpr[ ]）在虚函数表中的索引值。在这个索引值表示的虚表的槽中存放的就是 f()的地址。

  虚表指针的名字也会被编译器更改，所以在多继承的情况下，类的内部可能存在多个虚表指针。通过不同的名字被编译器标识。

  虚函数表中可能还存在其他的内容，如用于 RTTI 的 type_info 类型，或者直接将虚基类的指针存放在虚表中。
 
####    3.1 单继承

  + 这种情况下，派生类中仅有一个虚函数表。这个虚函数表和基类的虚函数表不是一个表（无论派生类有没有重写基类的虚函数）。
  + 3.1.1 如果派生类没有重写基类的虚函数的话，基类和派生类的虚函数表指向的函数地址都是相同的。
<div align=center>
    <img src=./image/C++知识点/1637655538685.png>
</div>

* 3.1.2 如果类 C 中重写了 A 类中的函数，那么就会覆盖 A 类的虚函数，重写一部分就会覆盖一部分，重写全部就会覆盖全部。
<div align=center>
    <img src=./image/C++知识点/1637655626658.png>
</div>

* 3.1.3 如果 C 中重新写了一些别的虚函数，那么这些虚函数将排在父类的后面
<div align=center>
    <img src=./image/C++知识点/1637655691961.png>
</div>

####    3.2 多继承

  + 多继承情况下，派生类中有多个虚函数表，虚函数的排列方式和继承的顺序一致。派生类重写函数将会覆盖所有虚函数表的同名内容。
  + **派生类自定义新的虚函数将会在第一个类的虚函数表的后面进行扩充。**

### 4、内存泄漏出现的时机和原因，如何避免

#### 4.1 在类的构造函数和析构函数中没有匹配的调用 new 和 delete 函数

  两种情况下会出现这种内存泄露：一是在堆里创建了对象占用了内存，但是没有显示地释放对象占用的内存；二是在类的构造函数中动态的分配了内存，但是在析构函数中没有释放内存或者没有正确的释放内存
####    4.2 没有正确地清除嵌套的对象指针
####    4.3 在释放对象数组时在 delete 中没有使用方括号
####    4.4 指向对象的指针数组不等同于对象数组

  **对象数组**是指：数组中存放的是对象，只需要 delete []p，即可调用对象数组中的每个对象的析构函数释放空间

  **指向对象的指针数组**是指：数组中存放的是指向对象的指针，不仅要释放每个对象的空间，还要释放每个指针的空间，delete []p 只是释放了每个指针，但是并没有释放对象的空间，正确的做法，是通过一个循环，将每个对象释放了，然后再把指针释放了。
 
####    4.5 浅拷贝导致的重复释放同一个内存区域问题
####    4.6 没有将基类的析构函数定义为虚函数

### 5、指针的工作原理，函数的传值和传址

#### 5.1 传值
  实际是把实际参数的值复制给形式参数，相当于copy。那么对形式参数的修改，不会影响实际参数的值。
 
#### 5.2 传址
  实际是传值的一种特殊方式，只是他传递的是**地址**，不是普通的值，那么传地址以后，实际参数和形式参数都指向同一个对象，因此对形式参数的修改会影响到实际参数。

#### 5.3 传引用参数
  在C++中，建议使用引用来代替C语言中的传址方法。引用可以大致理解为一个对象的别名。

```cpp
     int n = 0;
     int &r = n;//r绑定了n（即r是n的另一个名字）

     //该函数接受一个int对象的引用，然后将对象的值置为0
     void reset(int &i)
     {
        i = 0;
     }
```

### 6、new和delete使用解释一下，和malloc和free的区别

####    6.1 new与delete的使用

  + **6.1.1**  **new用法**

    - **6.1.1.1 开辟单变量地址空间**
      - 使用new运算符时必须已知数据类型，new运算符会向C++自由存储区申请足够的存储空间，具体是在堆上还是静态存储区上要看具体实现。
      - 如果申请成功，就返回该内存块的首地址，如果申请不成功，则会抛出std::bad_alloc异常或者返回空指针。
      - new运算符返回的是一个指向所分配类型变量（对象）的指针。对所创建的变量或对象，都是通过该指针来间接操作的，而动态创建的对象本身没有标识符名。

      一般使用格式：

      - **格式1：指针变量名=new 类型标识符;**
      `int *a = new int;//即将一个int类型的地址赋值给整型指针a`
      `int *a = new int();//对于非内置类型，c++11中的情况为：若构造函数没有参数，则()可以省略，因此new Type和new Type()是完全一样的`
      - **格式2：指针变量名=new 类型标识符(初始值);**
      `int *a = new int(5);//作用同上,但是同时将整数赋值为5`
      - **格式3：指针变量名=new 类型标识符[内存单元个数];**
      `int *a＝new int[5];//a指向该数组的首地址`
      `int a[] = new int[5];//等价`
      可以使用`int a[] = new int[5](0);`或者`int a[] = new int[5]{0,1,2,3,4};`来进行初始化
      `delete [] a;//注意堆上数组的delete操作`
      - 格式1和格式2都是申请分配某一数据类型所占字节数的内存空间。
        格式2在内存分配成功后，同时将一初值存放到该内存单元中。
        格式3可同时分配若干个内存单元，相当于在堆上动态形成一个数组。
      - 如果失败的话：
        ```cpp
            int *p1 = new int;//new会抛出std::bad_alloc异常
            int *p2 = new (nothrow) int;//返回空指针
        ```

    - **6.1.1.2 开辟数组空间**
        对于数组进行动态分配的格式为：
        **指针变量名=new 类型名[下标表达式];**
        ```cpp
            int *a = new int[100];//开辟一个大小为100的整型数组空间
            int **b = new int[5][6];//二维数组
            int *c = new int[0];//动态分配一个空数组是合法的，但是无法解引用（即使用*运算符）
            int *d = new int[get_size()];//分配大小可以通过别的方法确定
            //也可以使用别名
            typedef int arrT[42];//arrT表示42个int的数组类型
            int *e = new arrT;
        ```
        **delete [] 指向该数组的指针变量名;**
        如果delete语句中少了方括号，编译器会认为需要回收的该是指向数组第一个元素，会产生内存泄露的问题（只回收了第一个元素所占空间），加了方括号后就转化为指向数组的指针，回收整个数组。

  + **6.1.2**  **delete用法**

    - **6.1.2.1 删除单变量地址空间**
        ```cpp
            int *a = new int;
            delete a;
        ```

    - **6.1.2.2 删除数组空间**
        ```cpp
            int *a = new int[5];
            delete []a;
        ```

    - **6.1.2.3 需要注意的地方**
        - new了对象之后请**不要忘记使用delete操作释放对象**，不然会发生内存泄漏。
        - 请不要使用已经被释放掉的对象，这个问题**有时**可以通过在释放对象之后及时地把指针置空`p = nullptr;`的方法解决。
        置空指针的方法**只对当前被置空的指针有效**，如果还有其他指针也指向这个指针之前指向的内存区域的话，还是会出现问题，这只能提供**有限的保护**。
        - 请不要对着**同一块内存**进行**多次释放操作**。
        - 坚持**只使用智能指针**，可以避免上述三个问题。<br/>
        动态分配的变量或对象的生命期，我们也称堆空间为自由空间，但必须记住释放该对象所占堆空间只能释放一次，在函数内使用new创建对象请不要忘记释放它，一旦到了函数外局部变量就会失效，往往会出错。
        ```cpp
            Foo* factory(T arg)
            {
                return new Foo(arg);//调用者负责释放此内存
            }
            void use_factory(T arg)
            {
                Foo *p = factory(arg);
                //使用p但是不delete它
                //一定要在调用者内使用delete p;释放内存
            }
            //p离开了作用域，但是它指向的内存没有被释放！！！
        ```

####    6.2 new/delete和malloc/free区别

malloc/free为C的标准库函数，函数原型为：
    
```c
    void* malloc(size_t size)//参数代表字节个数
    void free(void* pointer)//参数代表内存地址
```

malloc/free使用方法

```c
    int* p1=(int*)malloc(sizeof(int));//开辟一个空间
    if(p1==NULL)
    {
        exit(1);
    }
    free(p1);

    int*p2=(int*)malloc(sizeof(int)*4);//开辟多个空间
    if(p2==NULL)
    {
        exit(1);
    }
    free(p2);
```

   * malloc开辟空间类型大小需手动计算，new是由编译器自己计算。
   * malloc返回类型为void*,必须强制类型转换对应类型指针，new则直接返回对应类型指针。
   * malloc开辟内存时返回内存地址要检查判空，因为若它可能开辟失败会返回NULL；new则不用判断，因为内存分配失败时，它会抛出异常std::bad_alloc,可以使用异常机制。
   * 无论释放几个空间大小，free()的参数只传递指针，而new生成的多个对象在释放时需要使用delete[]。
   * malloc/free函数只是开辟内存空间和释放内存空间，new/delete则不仅会开辟或释放内存空间，还会调用构造函数和析构函数进行初始化和清理。当对内置类型使用new[]的时候，会多开辟四个字节，用于存放对象的个数，在返回地址则也会向后偏移4个字节，而在delete[]时则会查看内存上对象个数，从而根据个数count确定调用几次析构函数，从而完全清理所有对象占用内存。
   * new/delete底层是基于malloc/free来实现的，而malloc/free不能基于new/delete实现。
   * 由于new/delete是C++的操作符，它调用operator new() / operator delete()，可以被重载，在标准库里它有8个重载版本，而malloc/free不可以重载。
   * 对于malloc分配内存后，若在使用过程中内存分配不够或太多，这时可以使用realloc函数对其进行扩充或缩小，但是new分配好的内存不能这样被直观简单的改变。
   * 对于new/delete若内存分配失败，用户可以指定处理函数或重新制定分配器（new_handler(可以在此处进行扩展)），malloc/free用户是不可以处理的。
   * malloc是在堆上分配内存的，但new其实不能说是在堆上，C++中，对new申请内存位置有一个抽象概念，称为自由存储区，new申请的内存空间可以在堆上，也可以在静态存储区上，这主要取决于operator new()实现细节，取决与它在哪里为对象分配空间。

### 7、C++内存区域如何划分（栈，堆那些）

**在C++中，内存分为5个区：栈、堆、自由存储区、全局/静态存储区和常量存储区。**

<div align=center>
    <img src=./image/C++知识点/20190710165757308.png>
</div>

*   栈：就是那些由编译器在需要的时候分配，在不需要的时候自动清楚的变量的存储区，里面的变量通常是局部变量、函数参数等。

*   堆： 操作系统层面的术语。就是那些由malloc等分配的内存块，用free来结束自己的生命的。

*   自由存储区：C++层面上的术语，就是那些由new分配的内存块，他们的释放编译器不去管，由我们的应用程序去控制，一般一个new就要对应一个delete。如果程序员没有释放掉，那么在程序结束后，操作系统会自动回收。new的申请会调用malloc，**自由存储区和堆类似，但不等价**。存储对象可以在不立即初始化的情况下分配内存，并且可以在不立即释放内存的情况下销毁它们。

*   全局/静态存储区：全局变量和静态变量被分配到同一块内存中，在以前的C语言中，全局变量又分为初始化的和未初始化的，在C++里面没有这个区分了，他们共同占用同一块内存区。

*   常量存储区：这是一块比较特殊的存储区，他们里面存放的是常量，不允许修改。

## C++11新特性

### 1、说说常见的新特性你知道哪些

  https://zhuanlan.zhihu.com/p/139515439

####    auto & decltype

  https://zhuanlan.zhihu.com/p/137662774

C++11引入了auto和decltype关键字，使用他们可以在编译期就推导出变量或者表达式的类型。

-   **auto**

      让编译器在编译器就推导出变量的类型，可以通过=右边的类型推导出变量的类型。
      
```cpp
   auto a = 10; // 10是int型，可以自动推导出a是int
```

- **decltype**

相对于auto用于推导变量类型，而decltype则用于推导表达式类型，这里只用于编译器分析表达式的类型，表达式实际不会进行运算。

```cpp
   cont int &i = 1;
   int a = 2;
   decltype(i) b = 2; // b是const int&
```

####    右值引用

  https://zhuanlan.zhihu.com/p/137662465

####    列表初始化

    https://zhuanlan.zhihu.com/p/137851769

####    std::function & std::bind & lambda表达式

  https://zhuanlan.zhihu.com/p/137884434

####    模版改进

  https://zhuanlan.zhihu.com/p/137851516

####    并发与多线程

  https://zhuanlan.zhihu.com/p/137914574

####    智能指针

  https://zhuanlan.zhihu.com/p/137958974

####    基于范围的for循环

```cpp
    vector<int> vec;

    for (auto iter = vec.begin(); iter != vec.end(); iter++) { // before c++11
        cout << *iter << endl;
    }

    for (int i : vec) { // c++11基于范围的for循环
        cout << "i" << endl;
    }
```

####    委托构造函数

  委托构造函数允许在同一个类中一个构造函数调用另外一个构造函数，可以在变量初始化时简化操作

```cpp
  //不使用委托构造函数
  struct A {
    A(){}
    A(int a) { a_ = a; }

    A(int a, int b) { // 好麻烦
        a_ = a;
        b_ = b;
    }

    A(int a, int b, int c) { // 好麻烦
        a_ = a;
        b_ = b;
        c_ = c;
    }

    int a_;
    int b_;
    int c_;
  };

  //使用委托构造函数
  struct A {
    A(){}
    A(int a) { a_ = a; }

    A(int a, int b) : A(a) { b_ = b; }

    A(int a, int b, int c) : A(a, b) { c_ = c; }

    int a_;
    int b_;
    int c_;
  };
```

####    继承构造函数

  继承构造函数可以让派生类直接使用基类的构造函数，如果有一个派生类，我们希望派生类采用和基类一样的构造方式，可以直接使用基类的构造函数，而不是再重新写一遍构造函数

```cpp
  //不使用继承构造函数
  struct Base {
    Base() {}
    Base(int a) { a_ = a; }

    Base(int a, int b) : Base(a) { b_ = b; }

    Base(int a, int b, int c) : Base(a, b) { c_ = c; }

    int a_;
    int b_;
    int c_;
  };

  struct Derived : Base {
    Derived() {}
    Derived(int a) : Base(a) {} // 好麻烦
    Derived(int a, int b) : Base(a, b) {} // 好麻烦
    Derived(int a, int b, int c) : Base(a, b, c) {} // 好麻烦
  };

  //使用继承构造函数
  struct Base {
    Base() {}
    Base(int a) { a_ = a; }

    Base(int a, int b) : Base(a) { b_ = b; }

    Base(int a, int b, int c) : Base(a, b) { c_ = c; }

    int a_;
    int b_;
    int c_;
  };

  struct Derived : Base {
    using Base::Base;
  };
```

####    nullptr

  nullptr是c++11用来表示空指针新引入的常量值，在c++中如果表示空指针语义时建议使用nullptr而不要使用NULL，因为NULL本质上是个int型的0，其实不是个指针

```cpp
    void func(void *ptr) {
      cout << "func ptr" << endl;
    }

    void func(int i) {
      cout << "func i" << endl;
    }

    int main() {
      func(NULL); // 编译失败，会产生二义性
      func(nullptr); // 输出func ptr
      return 0;
    }
```

####    final & override

  c++11关于继承新增了两个关键字，final用于修饰一个类，表示禁止该类进一步派生和虚函数的进一步重载。

  override用于修饰派生类中的成员函数，标明该函数重写了基类函数，如果一个函数声明了override但父类却没有这个虚函数，编译报错，使用override关键字可以避免开发者在重写基类函数时无意产生的错误。

```cpp
  struct Base {
    virtual void func() {
        cout << "base" << endl;
    }
  };

  struct Derived : public Base{
      void func() override { // 确保func被重写
          cout << "derived" << endl;
      }

      void fu() override { // error，基类没有fu()，不可以被重写
      }
  };
```

```cpp
  struct Base final {
    virtual void func() {
        cout << "base" << endl;
    }
  };

  struct Derived : public Base{ // 编译失败，final修饰的类不可以被继承
      void func() override {
          cout << "derived" << endl;
      }
  };
```

####    default

  c++11引入default特性，多数时候用于声明构造函数为默认构造函数，如果类中有了自定义的构造函数，编译器就不会隐式生成默认构造函数

```cpp
    struct A {
      int a;
      A(int i) { a = i; }
    };

    int main() {
      A a; // 编译出错
      return 0;
    }
```

  上面代码编译出错，因为没有匹配的构造函数，因为编译器没有生成默认构造函数，而通过default，程序员只需在函数声明后加上``` = default;```，就可将该函数声明为 defaulted 函数，编译器将为显式声明的 defaulted 函数自动生成函数体

```cpp
    struct A {
      A() = default;
      int a;
      A(int i) { a = i; }
    };

    int main() {
      A a;
      return 0;
    }
```

####    delete

  c++中，如果开发人员没有定义特殊成员函数，那么编译器在需要特殊成员函数时候会隐式自动生成一个默认的特殊成员函数，例如拷贝构造函数或者拷贝赋值操作符，如下代码：

```cpp
    struct A {
        A() = default;
        int a;
        A(int i) { a = i; }
    };

    int main() {
        A a1;
        A a2 = a1;  // 正确，调用编译器隐式生成的默认拷贝构造函数
        A a3;
        a3 = a1;  // 正确，调用编译器隐式生成的默认拷贝赋值操作符
    }
```

  而我们有时候想禁止对象的拷贝与赋值，可以使用delete修饰，如下：

```cpp
    struct A {
        A() = default;
        A(const A&) = delete;
        A& operator=(const A&) = delete;
        int a;
        A(int i) { a = i; }
    };

    int main() {
        A a1;
        A a2 = a1;  // 错误，拷贝构造函数被禁用
        A a3;
        a3 = a1;  // 错误，拷贝赋值操作符被禁用
    }
```

  delele函数在c++11中很常用，std::unique_ptr就是通过delete修饰来禁止对象的拷贝的。

####    explicit

  explicit专用于修饰构造函数，表示只能显式构造，不可以被隐式转换，根据代码看explicit的作用：

  不用explicit

```cpp
    struct A {
        A(int value) { // 没有explicit关键字
            cout << "value" << endl;
        }
    };

    int main() {
        A a = 1; // 可以隐式转换
        return 0;
    }
```

  使用explicit

```cpp
    struct A {
        explicit A(int value) {
            cout << "value" << endl;
        }
    };

    int main() {
        A a = 1; // error，不可以隐式转换
        A aa(2); // ok
        return 0;
    }
```

####    constexpr

  constexpr是c++11新引入的关键字，用于编译时的常量和常量函数，这里直接介绍constexpr和const的区别：

  两者都代表可读，const只表示read only的语义，只保证了运行时不可以被修改，但它修饰的仍然有可能是个动态变量，而constexpr修饰的才是真正的常量，它会在编译期间就会被计算出来，整个运行过程中都不可以被改变，constexpr可以用于修饰函数，这个函数的返回值会尽可能在编译期间被计算出来当作一个常量，但是如果编译期间此函数不能被计算出来，那它就会当作一个普通函数被处理。

```cpp
    #include<iostream>
    using namespace std;

    constexpr int func(int i) {
        return i + 1;
    }

    int main() {
        int i = 2;
        func(i);// 普通函数
        func(2);// 编译期间就会被计算出来
    }
```

####    enum class

  不带作用域的枚举代码
  ```cpp
    enum AColor {
        kRed,
        kGreen,
        kBlue
    };

    enum BColor {
        kWhite,
        kBlack,
        kYellow
    };

    int main() {
        if (kRed == kWhite) {
            cout << "red == white" << endl;
        }
        return 0;
    }
  ```

  如上代码，不带作用域的枚举类型可以自动转换成整形，且不同的枚举可以相互比较，代码中的红色居然可以和白色比较，这都是潜在的难以调试的bug，而这种完全可以通过有作用域的枚举来规避。

  ```cpp
  enum class AColor {
      kRed,
      kGreen,
      kBlue
  };

  enum class BColor {
      kWhite,
      kBlack,
      kYellow
  };

  int main() {
      if (AColor::kRed == BColor::kWhite) { // 编译失败
          cout << "red == white" << endl;
      }
      return 0;
  }
  ```

  使用带有作用域的枚举类型后，对不同的枚举进行比较会导致编译失败，消除潜在bug。

  同时带作用域的枚举类型可以选择底层类型，默认是int，可以改成char等别的类型。

  ```cpp
  enum class AColor : char {
      kRed,
      kGreen,
      kBlue
  };
  ```

  我们平时编程过程中使用枚举，一定要使用有作用域的枚举取代传统的枚举。

####    非受限联合体

  c++11之前union中数据成员的类型不允许有非POD类型，而这个限制在c++11被取消，允许数据成员类型有非POD类型，看代码：

  ```cpp
  struct A {
      int a;
      int *b;
  };

  union U {
      A a; // 非POD类型 c++11之前不可以这样定义联合体
      int b;
  };
  ```

####    sizeof

  c++11中sizeof可以用的类的数据成员上

  ```cpp
    struct A {
        int data[10];
        int a;
    };

    int main() {
        cout << "size " << sizeof(A::data) << endl;
        return 0;
    }
  ```

  想知道类中数据成员的大小在c++11中是不是方便了许多，而不需要定义一个对象，在计算对象的成员大小。

####    assertion

  ```cpp
    static_assert(true/false, message);
  ```

  c++11引入static_assert声明，用于在编译期间检查，如果第一个参数值为false，则打印message，编译失败。

####    自定义字面量

  https://blog.csdn.net/K346K346/article/details/85322227

####    内存对齐

  https://zhuanlan.zhihu.com/p/139520591

####    thread_local

  c++11引入thread_local，用thread_local修饰的变量具有thread周期，每一个线程都拥有并只拥有一个该变量的独立实例，一般用于需要保证线程安全的函数中

####    基础数值类型

  c++11新增了几种数据类型：long long、char16_t、char32_t等

####    随机数功能

  c++11关于随机数功能则较之前丰富了很多，典型的可以选择概率分布类型，先看如下代码

  ```cpp
  #include <time.h>

  #include <iostream>
  #include <random>

  using namespace std;

  int main() {
      std::default_random_engine random(time(nullptr));

      std::uniform_int_distribution<int> int_dis(0, 100); // 整数均匀分布
      std::uniform_real_distribution<float> real_dis(0.0, 1.0); // 浮点数均匀分布

      for (int i = 0; i < 10; ++i) {
          cout << int_dis(random) << ' ';
      }
      cout << endl;

      for (int i = 0; i < 10; ++i) {
          cout << real_dis(random) << ' ';
      }
      cout << endl;

      return 0;
  }
  ```
  代码中举例的是整数均匀分布和浮点数均匀分布，c++11提供的概率分布类型还有好多，例如伯努利分布、正态分布等。

####    正则表达式

  c++11引入了regex库更好的支持正则表达式

  ```cpp
  #include <iostream>
  #include <iterator>
  #include <regex>
  #include <string>

  int main() {
      std::string s = "I know, I'll use2 regular expressions.";
      // 忽略大小写
      std::regex self_regex("REGULAR EXPRESSIONS", std::regex_constants::icase);
      if (std::regex_search(s, self_regex)) {
          std::cout << "Text contains the phrase 'regular expressions'\n";
      }

      std::regex word_regex("(\\w+)");  // 匹配字母数字等字符
      auto words_begin = std::sregex_iterator(s.begin(), s.end(), word_regex);
      auto words_end = std::sregex_iterator();

      std::cout << "Found " << std::distance(words_begin, words_end) << " words\n";

      const int N = 6;
      std::cout << "Words longer than " << N << " characters:\n";
      for (std::sregex_iterator i = words_begin; i != words_end; ++i) {
          std::smatch match = *i;
          std::string match_str = match.str();
          if (match_str.size() > N) {
              std::cout << "  " << match_str << '\n';
          }
      }

      std::regex long_word_regex("(\\w{7,})");
      // 超过7个字符的单词用[]包围
      std::string new_s = std::regex_replace(s, long_word_regex, "[$&]");
      std::cout << new_s << '\n';
  }
  ```

####    chrono

  c++11关于时间引入了chrono库，源于boost，功能强大，chrono主要有三个点：

  + duration
  + time_point
  + clocks

####    新增数据结构

  + std::forward_list：单向链表，只可以前进，在特定场景下使用，相比于std::list节省了内存，提高了性能。

  ```cpp
    std::forward_list<int> fl = {1, 2, 3, 4, 5};
    for (const auto &elem : fl) {
        cout << elem;
    }
  ```
  + std::unordered_set：基于hash表实现的set，内部不会排序，使用方法和set类似
  + std::unordered_map：基于hash表实现的map，内部不会排序，使用方法和set类似
  + std::array：数组，在越界访问时抛出异常，**建议使用std::array替代普通的数组**
  + std::tuple：元组类型，类似pair，但比pair扩展性好

  ```cpp
    typedef std::tuple<int, double, int, double> Mytuple;
    Mytuple t(0, 1, 2, 3);
    std::cout << "0 " << std::get<0>(t);
    std::cout << "1 " << std::get<1>(t);
    std::cout << "2 " << std::get<2>(t);
    std::cout << "3 " << std::get<3>(t);
  ```

####    新增算法

  + all_of：检测表达式是否对范围[first, last)中所有元素都返回true，如果都满足，则返回true。

  ```cpp
    std::vector<int> v(10, 2);
    if (std::all_of(v.cbegin(), v.cend(), [](int i) { return i % 2 == 0; }))
    {
      std::cout << "All numbers are even\n";
    }
  ```

  + any_of：检测表达式是否对范围[first, last)中至少一个元素返回true，如果满足，则返回true，否则返回false，用法和上面一样
  + none_of：检测表达式是否对范围[first, last)中所有元素都不返回true，如果都不满足，则返回true，否则返回false，用法和上面一样
  + find_if_not：找到第一个不符合要求的元素迭代器，和find_if相反
  + copy_if：复制满足条件的元素
  + itoa：对容器内的元素按序递增
  ```cpp
  std::vector<int> l(10);
  std::iota(l.begin(), l.end(), 19); // 19为初始值
  for (auto n : l) std::cout << n << ' ';
  // 19 20 21 22 23 24 25 26 27 28
  minmax_element：返回容器内最大元素和最小元素位置
  int main() {
      std::vector<int> v = {3, 9, 1, 4, 2, 5, 9};

      auto result = std::minmax_element(v.begin(), v.end());
      std::cout << "min element at: " << *(result.first) << '\n';
      std::cout << "max element at: " << *(result.second) << '\n';
      return 0;
  }
  // min element at: 1
  // max element at: 9
  ```
  + is_sorted、is_sorted_until：返回容器内元素是否已经排好序。

### 2、C++11 智能指针用过吗，有哪些，他们的区别和各自的优缺点

  C++ 中有四种智能指针：auto_ptr、unique_ptr、shared_ptr、weak_ptr。

  其中后三个是 C++11 支持，第一个已经被 C++11 弃用且被 unique_prt 代替，不推荐使用。

  https://zhuanlan.zhihu.com/p/137958974

  https://zhuanlan.zhihu.com/p/150555165

### 3、C++11 auto关键字知道吗，如果全部都用auto声明变量行不行

  全部都用肯定不行

  + 滥用auto可能会降低代码可读性

  + auto的类型转换可能得不到你想要的类型

  + auto有时也可能会引入额外的性能开销。比如一个函数返回一个重量级的类的引用。``` MyBigDataType& func(); auto value = func();```

  + auto是会移除表达式类型reference属性的，那么此时以上最后一行的行为就是拷贝构造一MyBigData实例，相信这不是此函数的实现者希望的。

  总结一下，可以或者应当使用auto的情况：

  + 一：某个变量是局域的，且其类型能在auto声明的上下三五行以内被肉眼轻易看见。除非类型名称太长，否则还是显式声明可读性强。

  + 二：迭代子。必须用auto啊。

  + 三：C++11的for each中，容器变量的类型被显式声明于这一for循环之前五行以内。看情况。多重循环时每一层都用auto有可能会让代码变得很难读懂。可以在for的后面加注释标明元素类型。

以下是一些非常不推荐用auto的情况：

  + 一：模板类。如果嫌长可以用typedef。

  + 二：基本类型，如int long char等。

  + 三：使用std:pair和std:tuple中的元素时。

一切都是为了可读性。很多时候人们容易忘记两点∶

不是每个人都会使用IDE的。比如vim和emacs的用户就懒得切换到IDE中修改代码。

不是什么情况下IDE都能用的。比如在gerrit和github上做code review时就只能看到文本。

### 4、C++11 Lambda表达式会用吗

  https://zhuanlan.zhihu.com/p/150554945

### 5、C++11 override关键字必须吗

  https://zhuanlan.zhihu.com/p/258383836

### 6、C++11 右值引用说一下，可能会实际运用（重点！！！）

  https://zhuanlan.zhihu.com/p/335994370
  https://zhuanlan.zhihu.com/p/137662465
  https://zhuanlan.zhihu.com/p/99524127
  https://www.zhihu.com/question/28039779/answer/2218787422

## STL

### 1、STL的六大部件和联系

<div align=center>
    <img src=./image/C++知识点/STL六大部件.jpg>
</div>

* 1.容器：存放我们要操作的数据，可以是数字、对象等。
* 2.配置器：容器需要占用内存，容器占用的内存由分配器分配。
* 3.算法：被独立出来的模板函数，用来操作容器，包块常见的排序算法、查找算法等。
* 4.迭代器：算法既然要操作容器中的数据，需要有工具访问容器中数据，那就是迭代器，是一种泛化的指针。
* 5.容器适配器：一些容器底层和数据操作具有一定的相似，所以一些容器使用其他容器作为底层数据结构，将其他容器的函数转换为自己的函数。
* 6.仿函数：实际上是类中的operator()小括号运算符重载函数，存在类似函数的行为。

### 2、STL容器知道哪些，都解释一下，原理是什么，实现一下链表（接下来就可能考察算法题了）

  https://blog.csdn.net/weixin_41588502/article/details/87978490?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164318417416780274166829%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164318417416780274166829&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-87978490.first_rank_v2_pc_rank_v29&utm_term=STL%E5%AE%B9%E5%99%A8&spm=1018.2226.3001.4187

* 顺序容器
  + array
  + vector

    扩充方法：两倍扩充，先寻找内存中能存放两倍于之前容量（capacity）的空间，申请空间之后把原本的数据移动到新申请的空间中。
    排序方法：快速排序

  + list
  
    排序方法：归并排序

  + forward_list
  + deque

* 关联容器（红黑树）
  + set/multiset
  + map/multimap

* 无序容器（哈希表 链地址法）
  + unordered_set/unordered_multiset
  + unordered_map/unordered_multimap

* 容器适配器
  + stack(deque)
  + queue(deque)
  + priority_queue(deque)


### 3、map，multmap，set，multset的原理（接下来会考察二叉树相关的题，我后面会介绍）

  红黑树：https://www.cnblogs.com/skywang12345/p/3245399.html
  https://www.jianshu.com/p/e136ec79235c
  https://www.jianshu.com/p/e136ec79235c
  https://www.bilibili.com/video/BV1Tb4y197Fe?from=search&seid=17714196166085518916&spm_id_from=333.337.0.0
  https://www.bilibili.com/video/BV18y4y1m721?from=search&seid=17714196166085518916&spm_id_from=333.337.0.0

### 4、hashtable的作用与原理

<div align=center>
    <img src=./image/C++知识点/哈希表.png>
</div>

  https://zhuanlan.zhihu.com/p/95156642

### 5、操作符重载问题

  https://blog.csdn.net/ArtAndLife/article/details/120043623
  https://www.runoob.com/cplusplus/cpp-overloading.html

### 6、前置++与后置++的区别，操作符重载的角度分析源码实现

  https://blog.csdn.net/happyjacob/article/details/104233792

### 7、函数模板（泛化和特化的问题),什么是全特化和偏特化

  https://www.it610.com/article/1274988962074607616.html
  https://www.jianshu.com/p/add4f01a0de1
  https://www.jb51.net/article/179800.htm

  * 类模板
    + 全特化
    + 偏特化
      - 模版参数数量
      - 模版参数范围
  * 函数模版
    + 全特化
    + 不能偏特化

### 8、STL中常见的算法用过哪些，原理是什么（随机数，查找等）

  https://zhuanlan.zhihu.com/p/381607411