[TOC]

# C++相关

## 编译内存相关

### C++ 程序编译过程

编译过程分为四个过程：编译（编译预处理、编译、优化），汇编，链接。

+ **编译预处理**：处理以 # 开头的指令；

+ **编译、优化**：将源码 .cpp 文件翻译成 .s 汇编代码；

+ **汇编**：将汇编代码 .s 翻译成机器指令 .o 文件；

+ **链接**：汇编程序生成的目标文件，即 .o 文件，并不会立即执行，因为可能会出现：.cpp 文件中的函数引用了另一个 .cpp 文件中定义的符号或者调用了某个库文件中的函数。那链接的目的就是将这些文件对应的目标文件连接成一个整体，从而生成可执行的程序 .exe 文件。

![image-20210721160149344](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721160149344.png)

链接分为两种：

+ 静态链接：代码从其所在的静态链接库中拷贝到最终的可执行程序中，在该程序被执行时，这些代码会被装入到该进程的虚拟地址空间中。
+ 动态链接：代码被放到动态链接库或共享对象的某个目标文件中，链接程序只是在最终的可执行程序中记录了共享对象的名字等一些信息。在程序执行时，动态链接库的全部内容会被映射到运行时相应进行的虚拟地址的空间。

二者的优缺点：

+ 静态链接：浪费空间，每个可执行程序都会有目标文件的一个副本，这样如果目标文件进行了更新操作，就需要重新进行编译链接生成可执行程序（更新困难）；优点就是执行的时候运行速度快，因为可执行程序具备了程序运行的所有内容。
+ 动态链接：节省内存、更新方便，但是动态链接是在程序运行时，每次执行都需要链接，相比静态链接会有一定的性能损失。

### C++ 内存管理

C++ 内存分区：栈、堆、全局/静态存储区、常量存储区、代码区。

+ **栈**：存放函数的局部变量、函数参数、返回地址等，由编译器自动分配和释放。
+ **堆**：动态申请的内存空间，就是由 malloc 分配的内存块，由程序员控制它的分配和释放，如果程序执行结束还没有释放，操作系统会自动回收。
+ **全局区/静态存储区**（.bss 段和 .data 段）：存放全局变量和静态变量，程序运行结束操作系统自动释放，在 C 语言中，未初始化的放在 .bss 段中，初始化的放在 .data 段中，C++ 中不再区分了。
+ **常量存储区**（.data 段）：存放的是常量，不允许修改，程序运行结束自动释放。
+ **代码区**（.text 段）：存放代码，不允许修改，但可以执行。编译后的二进制文件存放在这里。
  说明：

+ 从操作系统的本身来讲，以上存储区在内存中的分布是如下形式(从低地址到高地址)：.text 段 --> .data 段 --> .bss 段 --> 堆 --> unused --> 栈 --> env

```cc
#include <iostream>
using namespace std;

/*
说明：C++ 中不再区分初始化和未初始化的全局变量、静态变量的存储区，如果非要区分下述程序标注在了括号中
*/

int g_var = 0; // g_var 在全局区（.data 段）
char *gp_var;  // gp_var 在全局区（.bss 段）

int main()
{
    int var;                    // var 在栈区
    char *p_var;                // p_var 在栈区
    char arr[] = "abc";         // arr 为数组变量，存储在栈区；"abc"为字符串常量，存储在常量区
    char *p_var1 = "123456";    // p_var1 在栈区；"123456"为字符串常量，存储在常量区
    static int s_var = 0;       // s_var 为静态变量，存在静态存储区（.data 段）
    p_var = (char *)malloc(10); // 分配得来的 10 个字节的区域在堆区
    free(p_var);
    return 0;
}

```

### 栈和堆的区别

+ 申请方式：栈是系统自动分配，堆是程序员主动申请。
+ 申请后系统响应：分配栈空间，如果剩余空间大于申请空间则分配成功，否则分配失败栈溢出；申请堆空间，堆在内存中呈现的方式类似于链表（记录空闲地址空间的链表），在链表上寻找第一个大于申请空间的节点分配给程序，将该节点从链表中删除，大多数系统中该块空间的首地址存放的是本次分配空间的大小，便于释放，将该块空间上的剩余空间再次连接在空闲链表上。
+ 栈在内存中是连续的一块空间（向低地址扩展）最大容量是系统预定好的，堆在内存中的空间（向高地址扩展）是不连续的。
+ 申请效率：栈是有系统自动分配，申请效率高，但程序员无法控制；堆是由程序员主动申请，效率低，使用起来方便但是容易产生碎片。
+ 存放的内容：栈中存放的是局部变量，函数的参数；堆中存放的内容由程序员控制。

### 变量的区别

**全局变量、局部变量、静态全局变量、静态局部变量的区别**
C++ 变量根据定义的位置的不同的生命周期，具有不同的作用域，作用域可分为 6 种：全局作用域，局部作用域，语句作用域，类作用域，命名空间作用域和文件作用域。

从作用域看：

+ **全局变量**：具有全局作用域。全局变量只需在一个源文件中定义，就可以作用于所有的源文件。当然，其他不包含全局变量定义的源文件需要用 extern 关键字再次声明这个全局变量。
+ **静态全局变量**：具有文件作用域。它与全局变量的区别在于如果程序包含多个文件的话，它作用于定义它的文件里，不能作用到其它文件里，即被 static 关键字修饰过的变量具有文件作用域。这样即使两个不同的源文件都定义了相同名字的静态全局变量，它们也是不同的变量。
+ **局部变量**：具有局部作用域。它是自动对象（auto），在程序运行期间不是一直存在，而是只在函数执行期间存在，函数的一次调用执行结束后，变量被撤销，其所占用的内存也被收回。
+ **静态局部变量**：具有局部作用域。它只被初始化一次，自从第一次被初始化直到程序运行结束都一直存在，它和全局变量的区别在于全局变量对所有的函数都是可见的，而静态局部变量只对定义自己的函数体始终可见。

从分配内存空间看：

+ 静态存储区：全局变量，静态局部变量，静态全局变量。
+ 栈：局部变量。

说明：

+ 静态变量和栈变量（存储在栈中的变量）、堆变量（存储在堆中的变量）的区别：静态变量会被放在程序的静态数据存储区（.data 段）中（静态变量会自动初始化），这样可以在下一次调用的时候还可以保持原来的赋值。而栈变量或堆变量不能保证在下一次调用的时候依然保持原来的值。
+ 静态变量和全局变量的区别：静态变量用 static 告知编译器，自己仅仅在变量的作用范围内可见。

### 全局变量定义在头文件中有什么问题？

如果在头文件中定义全局变量，当该头文件被多个文件 `include` 时，该头文件中的全局变量就会被定义多次，导致重复定义，因此不能再头文件中定义全局变量。

### 对象创建限制在堆或栈

#### 如何限制类的对象只能在堆上创建？如何限制对象只能在栈上创建？

说明：C++ 中的类的对象的建立分为两种：静态建立、动态建立。

+ 静态建立：由编译器为对象在栈空间上分配内存，直接调用类的构造函数创建对象。例如：A a;
+ 动态建立：使用 new 关键字在堆空间上创建对象，底层首先调用 operator new() 函数，在堆空间上寻找合适的内存并分配；然后，调用类的构造函数创建对象。例如：A *p = new A();
  

限制对象只能建立在堆上：

+ 最直观的思想：避免直接调用类的构造函数，因为对象静态建立时，会调用类的构造函数创建对象。但是直接将类的构造函数设为私有并不可行，因为当构造函数设置为私有后，不能在类的外部调用构造函数来构造对象，只能用 new 来建立对象。但是由于 new 创建对象时，底层也会调用类的构造函数，将构造函数设置为私有后，那就无法在类的外部使用 new 创建对象了。因此，这种方法不可行。

解决方法 1：

+ 将析构函数设置为私有。原因：静态对象建立在栈上，是由编译器分配和释放内存空间，编译器为对象分配内存空间时，会对类的非静态函数进行检查，即编译器会检查析构函数的访问性。当析构函数设为私有时，编译器创建的对象就无法通过访问析构函数来释放对象的内存空间，因此，编译器不会在栈上为对象分配内存。

```cc
class A
{
public:
    A() {}
    void destory()
    {
        delete this;
    }

private:
    ~A()
    {
    }
};
```

该方法存在的问题：

+ 用 new 创建的对象，通常会使用 delete 释放该对象的内存空间，但此时类的外部无法调用析构函数，因此类内必须定义一个 destory() 函数，用来释放 new 创建的对象。
+ 无法解决继承问题，因为如果这个类作为基类，析构函数要设置成 virtual，然后在派生类中重写该函数，来实现多态。但此时，析构函数是私有的，派生类中无法访问。

解决方法 2：

+ 构造函数设置为 protected，并提供一个 public 的静态函数来完成构造，而不是在类的外部使用 new 构造；将析构函数设置为 protected。原因：类似于单例模式，也保证了在派生类中能够访问析构函数。通过调用 create() 函数在堆上创建对象。

```cc
class A
{
protected:
    A() {}
    ~A() {}

public:
    static A *create()
    {
        return new A();
    }
    void destory()
    {
        delete this;
    }
};
```

限制对象只能建立在栈上：

+ 解决方法：将 operator new() 设置为私有。原因：当对象建立在堆上时，是采用 new 的方式进行建立，其底层会调用 operator new() 函数，因此只要对该函数加以限制，就能够防止对象建立在堆上。
  C++

```cc
class A
{
private:
    void *operator new(size_t t) {}    // 注意函数的第一个参数和返回值都是固定的
    void operator delete(void *ptr) {} // 重载了 new 就需要重载 delete
public:
    A() {}
    ~A() {}
};
```

### 什么是内存对齐？内存对齐的原则？为什么要进行内存对齐，有什么优点？

内存对齐：编译器将程序中的每个“数据单元”安排在字的整数倍的地址指向的内存之中
内存对齐的原则：

+ 结构体变量的首地址能够被其最宽基本类型成员大小与对齐基数中的较小者所整除；
+ 结构体每个成员相对于结构体首地址的偏移量 （offset） 都是该成员大小与对齐基数中的较小者的整数倍，如有需要编译器会在成员之间加上填充字节 （internal padding）；
+ 结构体的总大小为结构体最宽基本类型成员大小与对齐基数中的较小者的整数倍，如有需要编译器会在最末一个成员之后加上填充字节 （trailing padding）。

```cc
/*
说明：程序是在 64 位编译器下测试的
*/
#include <iostream>

using namespace std;

struct A
{
    short var; // 2 字节
    int var1;  // 8 字节 （内存对齐原则：填充 2 个字节） 2 (short) + 2 (填充) + 4 (int)= 8
    long var2; // 12 字节 8 + 4 (long) = 12
    char var3; // 16 字节 （内存对齐原则：填充 3 个字节）12 + 1 (char) + 3 (填充) = 16
    string s;  // 48 字节 16 + 32 (string) = 48
};

int main()
{
    short var;
    int var1;
    long var2;
    char var3;
    string s;
    A ex1;
    cout << sizeof(var) << endl;  // 2 short
    cout << sizeof(var1) << endl; // 4 int
    cout << sizeof(var2) << endl; // 4 long
    cout << sizeof(var3) << endl; // 1 char
    cout << sizeof(s) << endl;    // 32 string
    cout << sizeof(ex1) << endl;  // 48 struct
    return 0;
}
```

进行内存对齐的原因：（主要是硬件设备方面的问题）

+ 某些硬件设备只能存取对齐数据，存取非对齐的数据可能会引发异常；
+ 某些硬件设备不能保证在存取非对齐数据的时候的操作是原子操作；
+ 相比于存取对齐的数据，存取非对齐的数据需要花费更多的时间；
+ 某些处理器虽然支持非对齐数据的访问，但会引发对齐陷阱（alignment trap）；
+ 某些硬件设备只支持简单数据指令非对齐存取，不支持复杂数据指令的非对齐存取。

内存对齐的优点：

+ 便于在不同的平台之间进行移植，因为有些硬件平台不能够支持任意地址的数据访问，只能在某些地址处取某些特定的数据，否则会抛出异常；
+ 提高内存的访问效率，因为 CPU 在读取内存时，是一块一块的读取。

### 类的大小

类的大小是指类的实例化对象的大小，用 sizeof 对类型名操作时，结果是该类型的对象的大小。
计算原则：

+ 遵循结构体的对齐原则。
+ 与普通成员变量有关，与成员函数和静态成员无关。即普通成员函数，静态成员函数，静态数据成员，静态常量数据成员均对类的大小无影响。因为静态数据成员被类的对象共享，并不属于哪个具体的对象。
+ 虚函数对类的大小有影响，是因为虚函数表指针的影响。
+ 虚继承对类的大小有影响，是因为虚基表指针带来的影响。
+ 空类的大小是一个特殊情况，空类的大小为 1，当用 new 来创建一个空类的对象时，为了保证不同对象的地址不同，空类也占用存储空间。

```cc
/*
说明：程序是在 64 位编译器下测试的
*/
#include <iostream>

using namespace std;

class A
{
private:
    static int s_var; // 不影响类的大小
    const int c_var;  // 4 字节
    int var;          // 8 字节 4 + 4 (int) = 8
    char var1;        // 12 字节 8 + 1 (char) + 3 (填充) = 12
public:
    A(int temp) : c_var(temp) {} // 不影响类的大小
    ~A() {}                    // 不影响类的大小
};

class B
{
};
int main()
{
    A ex1(4);
    B ex2;
    cout << sizeof(ex1) << endl; // 12 字节
    cout << sizeof(ex2) << endl; // 1 字节
    return 0;
}

```

带有虚函数的情况：（注意：虚函数的个数并不影响所占内存的大小，因为类对象的内存中只保存了指向虚函数表的指针。）

```cc
/*
说明：程序是在 64 位编译器下测试的
*/
#include <iostream>

using namespace std;

class A
{
private:
    static int s_var; // 不影响类的大小
    const int c_var;  // 4 字节
    int var;          // 8 字节 4 + 4 (int) = 8
    char var1;        // 12 字节 8 + 1 (char) + 3 (填充) = 12
public:
    A(int temp) : c_var(temp) {} // 不影响类的大小
    ~A() {}                      // 不影响类的大小
    virtual void f() { cout << "A::f" << endl; }

    virtual void g() { cout << "A::g" << endl; }

    virtual void h() { cout << "A::h" << endl; } // 24 字节 12 + 4 (填充) + 8 (指向虚函数的指针) = 24
};

int main()
{
    A ex1(4);
    A *p;
    cout << sizeof(p) << endl;   // 8 字节 注意：指针所占的空间和指针指向的数据类型无关
    cout << sizeof(ex1) << endl; // 24 字节
    return 0;
}

```

### 什么是内存泄露

**内存泄漏：**由于疏忽或错误导致的程序未能释放已经不再使用的内存。
进一步解释：

+ 并非指内存从物理上消失，而是指程序在运行过程中，由于疏忽或错误而失去了对该内存的控制，从而造成了内存的浪费。
+ 常指堆内存泄漏，因为堆是动态分配的，而且是用户来控制的，如果使用不当，会产生内存泄漏。
+ 使用 malloc、calloc、realloc、new 等分配内存时，使用完后要调用相应的 free 或 delete 释放内存，否则这块内存就会造成内存泄漏。
+ 指针重新赋值

```cc
char *p = (char *)malloc(10);
char *p1 = (char *)malloc(10);
p = np;
```

开始时，指针 p 和 p 1 分别指向一块内存空间，但指针 p 被重新赋值，导致 p 初始时指向的那块内存空间无法找到，从而发生了内存泄漏。

#### 怎么防止内存泄漏？内存泄漏检测工具的原理？

**智能指针**
智能指针是 C++ 中已经对内存泄漏封装好了一个工具，可以直接拿来使用

智能指针是为了解决动态内存分配时带来的内存泄漏以及多次释放同一块内存空间而提出的。C++11 中封装在了 <memory> 头文件中。

C++11 中智能指针包括以下三种：

+ **共享指针（shared_ptr）**：资源可以被多个指针共享，使用计数机制表明资源被几个指针共享。通过 use_count() 查看资源的所有者的个数，可以通过 unique_ptr、weak_ptr 来构造，调用 release() 释放资源的所有权，计数减一，当计数减为 0 时，会自动释放内存空间，从而避免了内存泄漏。
+ **独占指针（unique_ptr）**：独享所有权的智能指针，资源只能被一个指针占有，该指针不能拷贝构造和赋值。但可以进行移动构造和移动赋值构造（调用 move() 函数），即一个 unique_ptr 对象赋值给另一个 unique_ptr 对象，可以通过该方法进行赋值。
+ 弱指针（weak_ptr）：指向 share_ptr 指向的对象，能够解决由shared_ptr带来的循环引用问题。

**智能指针的实现原理**： 计数原理。

```c++
#include <iostream>
#include <memory>

template <typename T>
class SmartPtr
{
private : 
	T *_ptr;
	size_t *_count;

public:
	SmartPtr(T *ptr = nullptr) : _ptr(ptr)
	{
		if (_ptr)
		{
			_count = new size_t(1);
		}
		else
		{
			_count = new size_t(0);
		}
	}

	~SmartPtr()
	{
		(*this->_count)--;
		if (*this->_count == 0)
		{
			delete this->_ptr;
			delete this->_count;
		}
	}

	SmartPtr(const SmartPtr &ptr) // 拷贝构造：计数 +1
	{
		if (this != &ptr)
		{
			this->_ptr = ptr._ptr;
			this->_count = ptr._count;
			(*this->_count)++;
		}
	}

	SmartPtr &operator=(const SmartPtr &ptr) // 赋值运算符重载 
	{
		if (this->_ptr == ptr._ptr)
		{
			return *this;
		}
		if (this->_ptr) // 将当前的 ptr 指向的原来的空间的计数 -1
		{
			(*this->_count)--;
			if (this->_count == 0)
			{
				delete this->_ptr;
				delete this->_count;
			}
		}
		this->_ptr = ptr._ptr;
		this->_count = ptr._count;
		(*this->_count)++; // 此时 ptr 指向了新赋值的空间，该空间的计数 +1
		return *this;
	}

	T &operator*()
	{
		assert(this->_ptr == nullptr);
		return *(this->_ptr);
	}

	T *operator->()
	{
		assert(this->_ptr == nullptr);
		return this->_ptr;
	}

	size_t use_count()
	{
		return *this->count;
	}
};

```

#### 一个 unique_ptr 怎么赋值给另一个 unique_ptr 对象？

借助 std::move() 可以实现将一个 unique_ptr 对象赋值给另一个 unique_ptr 对象，其目的是实现所有权的转移。

```cc
// A 作为一个类 
std::unique_ptr<A> ptr1(new A());
std::unique_ptr<A> ptr2 = std::move(ptr1);
```

#### 使用智能指针会出现什么问题？怎么解决？

**智能指针可能出现的问题：循环引用**
在如下例子中定义了两个类 Parent、Child，在两个类中分别定义另一个类的对象的共享指针，由于在程序结束后，两个指针相互指向对方的内存空间，导致内存无法释放。

```cc
#include <iostream>
#include <memory>

using namespace std;

class Child;
class Parent;

class Parent {
private:
    shared_ptr<Child> ChildPtr;
public:
    void setChild(shared_ptr<Child> child) {
        this->ChildPtr = child;
    }

    void doSomething() {
        if (this->ChildPtr.use_count()) {

        }
    }

    ~Parent() {
    }
};

class Child {
private:
    shared_ptr<Parent> ParentPtr;
public:
    void setPartent(shared_ptr<Parent> parent) {
        this->ParentPtr = parent;
    }
    void doSomething() {
        if (this->ParentPtr.use_count()) {

        }
    }
    ~Child() {
    }
};

int main() {
    weak_ptr<Parent> wpp;
    weak_ptr<Child> wpc;
    {
        shared_ptr<Parent> p(new Parent);
        shared_ptr<Child> c(new Child);
        p->setChild(c);
        c->setPartent(p);
        wpp = p;
        wpc = c;
        cout << p.use_count() << endl; // 2
        cout << c.use_count() << endl; // 2
    }
    cout << wpp.use_count() << endl;  // 1
    cout << wpc.use_count() << endl;  // 1
    return 0;
}
```

**循环引用的解决方法**： weak_ptr

循环引用：该被调用的析构函数没有被调用，从而出现了内存泄漏。

+ weak_ptr 对被 shared_ptr 管理的对象存在 **非拥有性（弱）引用**，在访问所引用的对象前必须先转化为 shared_ptr；
+ weak_ptr 用来打断 shared_ptr 所管理对象的循环引用问题，若这种环被孤立（没有指向环中的外部共享指针），shared_ptr 引用计数无法抵达 0，内存被泄露；令环中的指针之一为弱指针可以避免该情况；
+ weak_ptr 用来表达临时所有权的概念，当某个对象只有存在时才需要被访问，而且随时可能被他人删除，可以用 weak_ptr 跟踪该对象；需要获得所有权时将其转化为 shared_ptr，此时如果原来的 shared_ptr 被销毁，则该对象的生命期被延长至这个临时的 shared_ptr 同样被销毁。

```cc
#include <iostream>
#include <memory>

using namespace std;

class Child;
class Parent;

class Parent {
private:
    //shared_ptr<Child> ChildPtr;
    weak_ptr<Child> ChildPtr;
public:
    void setChild(shared_ptr<Child> child) {
        this->ChildPtr = child;
    }

    void doSomething() {
        //new shared_ptr
        if (this->ChildPtr.lock()) {

        }
    }

    ~Parent() {
    }
};

class Child {
private:
    shared_ptr<Parent> ParentPtr;
public:
    void setPartent(shared_ptr<Parent> parent) {
        this->ParentPtr = parent;
    }
    void doSomething() {
        if (this->ParentPtr.use_count()) {

        }
    }
    ~Child() {
    }
};

int main() {
    weak_ptr<Parent> wpp;
    weak_ptr<Child> wpc;
    {
        shared_ptr<Parent> p(new Parent);
        shared_ptr<Child> c(new Child);
        p->setChild(c);
        c->setPartent(p);
        wpp = p;
        wpc = c;
        cout << p.use_count() << endl; // 2
        cout << c.use_count() << endl; // 1
    }
    cout << wpp.use_count() << endl;  // 0
    cout << wpc.use_count() << endl;  // 0
    return 0;
}

```

## 语言特性

### C++ 11 新特性

说明：C++11 的新特性有很多，从面试的角度来讲，如果面试官问到该问题，常以该问题作为引子，对面试者提到的知识点进行深入展开提问。面试者尽可能的列举常用的并且熟悉的特性，尽可能的掌握相关原理，下文只是对相关知识点进行了简单的阐述，有关细节还需要结合相关知识点的相关问题。
下面对常用的做一下总结：

1. auto 类型推导
   auto 关键字：自动类型推导，编译器会在 编译期间 通过初始值推导出变量的类型，通过 auto 定义的变量必须有初始值。

auto 关键字基本的使用语法如下：

```cc
auto var = val1 + val2; // 根据 val1 和 val2 相加的结果推断出 var 的类型，
```

注意：编译器推导出来的类型和初始值的类型并不完全一样，编译器会适当地改变结果类型使其更符合初始化规则。

2. decltype 类型推导
   decltype 关键字：decltype 是“declare type”的缩写，译为“声明类型”。和 auto 的功能一样，都用来在编译时期进行自动类型推导。如果希望从表达式中推断出要定义的变量的类型，但是不想用该表达式的值初始化变量，这时就不能再用 auto。decltype 作用是选择并返回操作数的数据类型

区别：

```cc
auto var = val1 + val2; 
decltype(val1 + val2) var1 = 0; 
```

+ auto 根据 = 右边的初始值 val1 + val2 推导出变量的类型，并将该初始值赋值给变量 var；decltype 根据 val1 + val2 表达式推导出变量的类型，变量的初始值和与表达式的值无关。
+ auto 要求变量必须初始化，因为它是根据初始化的值推导出变量的类型，而 decltype 不要求，定义变量的时候可初始化也可以不初始化。

3. lambda 表达式
   lambda 表达式，又被称为 lambda 函数或者 lambda 匿名函数

lambda匿名函数的定义:

```cc
[capture list] (parameter list) -> return type
{
   function body;
};
```

其中：

+ capture list：捕获列表，指 lambda 所在函数中定义的局部变量的列表，通常为空。
+ return type、parameter list、function body：分别表示返回值类型、参数列表、函数体，和普通函数一样。

举例：

```cc
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    int arr[4] = {4, 2, 3, 1};
    //对 a 数组中的元素进行升序排序
    sort(arr, arr+4, [=](int x, int y) -> bool{ return x < y; } );
    for(int n : arr){
        cout << n << " ";
    }
    return 0;
}
```

4. 范围 for 语句

语法格式：

```cc
for (declaration : expression){
    statement
}
```

参数的含义：

expression：必须是一个序列，例如用花括号括起来的初始值列表、数组、vector ，string 等，这些类型的共同特点是拥有能返回迭代器的 beign、end 成员。
declaration：此处定义一个变量，序列中的每一个元素都能转化成该变量的类型，常用 auto 类型说明符。


5. 右值引用
   右值引用：绑定到右值的引用，用 && 来获得右值引用，右值引用只能绑定到要销毁的对象。为了和右值引用区分开，常规的引用称为左值引用。

举例：

```cc
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    int var = 42;
    int &l_var = var;
    int &&r_var = var; // error: cannot bind rvalue reference of type 'int&&' to lvalue of type 'int' 错误：不能将右值引用绑定到左值上

	int &&r_var2 = var + 40; // 正确：将 r_var2 绑定到求和结果上
	return 0;
}
```

6. 标准库 move() 函数
   move() 函数：通过该函数可获得绑定到左值上的右值引用，该函数包括在 utility 头文件中。该知识点会在后续的章节中做详细的说明。

7. 智能指针
   相关知识已在第一章中进行了详细的说明，这里不再重复。

8. delete 函数和 default 函数

+ delete 函数：= delete 表示该函数不能被调用。
+ default 函数：= default 表示编译器生成默认的函数，例如：生成默认的构造函数。

```cc
#include <iostream>
using namespace std;

class A
{
public:
	A() = default; // 表示使用默认的构造函数
	~A() = default;	// 表示使用默认的析构函数
	A(const A &) = delete; // 表示类的对象禁止拷贝构造
	A &operator=(const A &) = delete; // 表示类的对象禁止拷贝赋值
};
int main()
{
	A ex1;
	A ex2 = ex1; // error: use of deleted function 'A::A(const A&)'
	A ex3;
	ex3 = ex1; // error: use of deleted function 'A& A::operator=(const A&)'
	return 0;
}
```

## 面向对象

### 什么是面向对象？面向对象的三大特性

面向对象：对象是指具体的某一个事物，这些事物的抽象就是类，类中包含数据（成员变量）和动作（成员方法）。

面向对象的三大特性：

+ 封装：将具体的实现过程和数据封装成一个函数，只能通过接口进行访问，降低耦合性。
+ 继承：子类继承父类的特征和行为，子类有父类的非 private 方法或成员变量，子类可以对父类的方法进行重写，增强了类之间的耦合性，但是当父类中的成员变量、成员函数或者类本身被 final 关键字修饰时，修饰的类不能继承，修饰的成员不能重写或修改。
+ 多态：多态就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。

### 重载、重写、隐藏的区别

概念解释：

+ 重载：是指同一可访问区内被声明几个具有不同参数列（参数的类型、个数、顺序）的同名函数，根据参数列表确定调用哪个函数，重载不关心函数返回类型。

```cc
class A
{
public:
    void fun(int tmp);
    void fun(float tmp);        // 重载 参数类型不同（相对于上一个函数）
    void fun(int tmp, float tmp1); // 重载 参数个数不同（相对于上一个函数）
    void fun(float tmp, int tmp1); // 重载 参数顺序不同（相对于上一个函数）
    int fun(int tmp);            // error: 'int A::fun(int)' cannot be overloaded 错误：注意重载不关心函数返回类型
};
```

隐藏：是指派生类的函数屏蔽了与其同名的基类函数，主要只要同名函数，不管参数列表是否相同，基类函数都会被隐藏。

```cc
#include <iostream>
using namespace std;

class Base
{
public:
    void fun(int tmp, float tmp1) { cout << "Base::fun(int tmp, float tmp1)" << endl; }
};

class Derive : public Base
{
public:
    void fun(int tmp) { cout << "Derive::fun(int tmp)" << endl; } // 隐藏基类中的同名函数
};

int main()
{
    Derive ex;
    ex.fun(1);       // Derive::fun(int tmp)
    ex.fun(1, 0.01); // error: candidate expects 1 argument, 2 provided
    return 0;
}
```

说明：上述代码中` ex.fun(1, 0.01);` 出现错误，说明派生类中将基类的同名函数隐藏了。若是想调用基类中的同名函数，可以加上类型名指明` ex.Base::fun(1, 0.01);`，这样就可以调用基类中的同名函数。

+ 重写(覆盖)：是指派生类中存在重新定义的函数。函数名、参数列表、返回值类型都必须同基类中被重写的函数一致，只有函数体不同。派生类调用时会调用派生类的重写函数，不会调用被重写函数。重写的基类中被重写的函数必须有` virtual `修饰。

```cc
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun(int tmp) { cout << "Base::fun(int tmp) : " << tmp << endl; }
};

class Derived : public Base
{
public:
    virtual void fun(int tmp) { cout << "Derived::fun(int tmp) : " << tmp << endl; } // 重写基类中的 fun 函数
};
int main()
{
    Base *p = new Derived();
    p->fun(3); // Derived::fun(int) : 3
    return 0;
}
```

**重写和重载的区别：**

+ 范围区别：对于类中函数的重载或者重写而言，重载发生在同一个类的内部，重写发生在不同的类之间（子类和父类之间）。
+ 参数区别：重载的函数需要与原函数有相同的函数名、不同的参数列表，不关注函数的返回值类型；重写的函数的函数名、参数列表和返回值类型都需要和原函数相同，父类中被重写的函数需要有 virtual 修饰。
+ `virtual` 关键字：重写的函数基类中必须有 virtual关键字的修饰，重载的函数可以有 virtual 关键字的修饰也可以没有。

**隐藏和重写，重载的区别：**

+ 范围区别：隐藏与重载范围不同，隐藏发生在不同类中。
+ 参数区别：隐藏函数和被隐藏函数参数列表可以相同，也可以不同，但函数名一定相同；当参数不同时，无论基类中的函数是否被 `virtual` 修饰，基类函数都是被隐藏，而不是重写。

### 如何理解 C++ 是面向对象编程

该问题最好结合自己的项目经历进行展开解释，或举一些恰当的例子，同时对比下面向过程编程。

### 什么是多态？多态如何实现？

**多态**：多态就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。在基类的函数前加上 virtual 关键字，在派生类中重写该函数，运行时将会根据对象的实际类型来调用相应的函数。如果对象类型是派生类，就调用派生类的函数；如果对象类型是基类，就调用基类的函数。
**实现方法**：多态是通过虚函数实现的，虚函数的地址保存在虚函数表中，虚函数表的地址保存在含有虚函数的类的实例对象的内存空间中。

**实现过程**：

+ 在类中用 virtual 关键字声明的函数叫做虚函数；
+ 存在虚函数的类都有一个虚函数表，当创建一个该类的对象时，该对象有一个指向虚函数表的虚表指针（虚函数表和类对应的，虚表指针是和对象对应）；
+ 当基类指针指向派生类对象，基类指针调用虚函数时，基类指针指向派生类的虚表指针，由于该虚表指针指向派生类虚函数表，通过遍历虚表，寻找相应的虚函数。

举例：

```cc
#include <iostream>
using namespace std;

class Base
{
public:
	virtual void fun() { cout << "Base::fun()" << endl; }

	virtual void fun1() { cout << "Base::fun1()" << endl; }

	virtual void fun2() { cout << "Base::fun2()" << endl; }
};
class Derive : public Base
{
public:
	void fun() { cout << "Derive::fun()" << endl; }

	virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }

	virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
};
int main()
{
	Base *p = new Derive();
	p->fun(); // Derive::fun() 调用派生类中的虚函数
	return 0;
}
```

基类的虚函数表如下：

![image-20210721163831828](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721163831828.png)

派生类的对象虚函数表如下：

![image-20210721163844122](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721163844122.png)

简单解释：当基类的指针指向派生类的对象时，通过派生类的对象的虚表指针找到虚函数表（派生类的对象虚函数表），进而找到相应的虚函数 Derive::f() 进行调用。

## 关键字库函数

`strlen` 测量的是字符串的实际长度（其源代码如下），以 `\0` 结束。而 `sizeof` 测量的是字符数组的分配大小。

```cc
#include <iostream>
#include <cstring>

using namespace std;

int main()
{
    char arr[10] = "hello";
    cout << strlen(arr) << endl; // 5
    cout << sizeof(arr) << endl; // 10
    return 0;
}
```

若字符数组 arr 作为函数的形参，sizeof(arr) 中 arr 被当作字符指针来处理，strlen(arr) 中 arr 依然是字符数组，从下述程序的运行结果中就可以看出。

```cc
#include <iostream>
#include <cstring>

using namespace std;

void size_of(char arr[])
{
    cout << sizeof(arr) << endl; // warning: 'sizeof' on array function parameter 'arr' will return size of 'char*' .
    cout << strlen(arr) << endl; 
}

int main()
{
    char arr[20] = "hello";
    size_of(arr); 
    return 0;
}
/*
输出结果：
8
5
*/
```

+ strlen 本身是库函数，因此在程序运行过程中，计算长度；而 sizeof 在编译时，计算长度；

+ sizeof 的参数可以是类型，也可以是变量；strlen 的参数必须是 char* 类型的变量。

### lambda 表达式（匿名函数）的具体应用和使用场景

lambda 表达式的定义形式如下：

```cc
[capture list] (parameter list) -> reurn type
{
   function body
}
```

其中：

+ capture list：捕获列表，指 lambda 表达式所在函数中定义的局部变量的列表，通常为空，但如果函数体中用到了 lambda 表达式所在函数的局部变量，必须捕获该变量，即将此变量写在捕获列表中。捕获方式分为：引用捕获方式 [&]、值捕获方式 [=]。
+ return type、parameter list、function body：分别表示返回值类型、参数列表、函数体，和普通函数一样。

举例：
`lambda` 表达式常搭配排序算法使用。

```cc
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    vector<int> arr = {3, 4, 76, 12, 54, 90, 34};
    sort(arr.begin(), arr.end(), [](int a, int b) { return a > b; }); // 降序排序
    for (auto a : arr)
    {
        cout << a << " ";
    }
    return 0;
}
/*
运行结果：90 76 54 34 12 4 3
*/
```



### explicit 的作用（如何避免编译器进行隐式类型转换）

作用：用来声明类构造函数是显示调用的，而非隐式调用，可以阻止调用构造函数时进行隐式转换。只可用于修饰单参构造函数，因为无参构造函数和多参构造函数本身就是显示调用的，再加上 explicit 关键字也没有什么意义。

```cc
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    A(int tmp)
    {
        var = tmp;
    }
};
int main()
{
    A ex = 10; // 发生了隐式转换
    return 0;
}
```

上述代码中，A ex = 10; 在编译时，进行了隐式转换，将 10 转换成 A 类型的对象，然后将该对象赋值给 ex，等同于如下操作：

为了避免隐式转换，可用 explicit 关键字进行声明：

```cc
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    explicit A(int tmp)
    {
        var = tmp;
        cout << var << endl;
    }
};
int main()
{
    A ex(100);
    A ex1 = 10; // error: conversion from 'int' to non-scalar type 'A' requested
    return 0;
}
```

### static 的作用

作用：
static 定义静态变量，静态函数。

+ 保持变量内容持久：static 作用于局部变量，改变了局部变量的生存周期，使得该变量存在于定义后直到程序运行结束的这段时间。

```cc
#include <iostream>
using namespace std;

int fun(){
    static int var = 1; // var 只在第一次进入这个函数的时初始化
    var += 1;
    return var;
}
  
int main()
{
    for(int i = 0; i < 10; ++i)
    	cout << fun() << " "; // 2 3 4 5 6 7 8 9 10 11
    return 0;
}

```

+ 隐藏：static 作用于全局变量和函数，改变了全局变量和函数的作用域，使得全局变量和函数只能在定义它的文件中使用，在源文件中不具有全局可见性。（注：普通全局变量和函数具有全局可见性，即其他的源文件也可以使用。）

+ static 作用于类的成员变量和类的成员函数，使得类变量或者类成员函数和类有关，也就是说可以不定义类的对象就可以通过类访问这些静态成员。注意：类的静态成员函数中只能访问静态成员变量或者静态成员函数，不能将静态成员函数定义成虚函数。

```cc
#include<iostream>
using namespace std;

class A
{
private:
    int var;
    static int s_var; // 静态成员变量
public:
    void show()
    {
        cout << s_var++ << endl;
    }
    static void s_show()
    {
        cout << s_var << endl;
		// cout << var << endl; // error: invalid use of member 'A::a' in static member function. 静态成员函数不能调用非静态成员变量。无法使用 this.var
        // show();  // error: cannot call member function 'void A::show()' without object. 静态成员函数不能调用非静态成员函数。无法使用 this.show()
    }
};
int A::s_var = 1;  // 静态成员变量在类外进行初始化赋值，默认初始化为 0

int main()
{
    
    // cout << A::sa << endl;    // error: 'int A::sa' is private within this context
    A ex;
    ex.show();
    A::s_show();
}
```

### static 在类中使用的注意事项（定义、初始化和使用）

**static 静态成员变量：**

1. 静态成员变量是在类内进行声明，在类外进行定义和初始化，在类外进行定义和初始化的时候不要出现 static 关键字和private、public、protected 访问规则。
2. 静态成员变量相当于类域中的全局变量，被类的所有对象所共享，包括派生类的对象。
3. 静态成员变量可以作为成员函数的参数，而普通成员变量不可以。

```cc
#include <iostream>
using namespace std;

class A
{
public:
    static int s_var;
    int var;
    void fun1(int i = s_var); // 正确，静态成员变量可以作为成员函数的参数
    void fun2(int i = var);   //  error: invalid use of non-static data member 'A::var'
};
int main()
{
    return 0;
}
```

4. 静态数据成员的类型可以是所属类的类型，而普通数据成员的类型只能是该类类型的指针或引用。

```cc
#include <iostream>
using namespace std;

class A
{
public:
    static A s_var; // 正确，静态数据成员
    A var;          // error: field 'var' has incomplete type 'A'
    A *p;           // 正确，指针
    A &var1;        // 正确，引用
};

int main()
{
    return 0;
}
```

**static 静态成员函数：**

1. 静态成员函数不能调用非静态成员变量或者非静态成员函数，因为静态成员函数没有 this 指针。静态成员函数做为类作用域的全局函数。
2. 静态成员函数不能声明成虚函数（virtual）、const 函数和 volatile 函数。

### static 全局变量和普通全局变量的异同

**相同点：**

+ 存储方式：普通全局变量和 static 全局变量都是静态存储方式。

**不同点：**

+ 作用域：普通全局变量的作用域是整个源程序，当一个源程序由多个源文件组成时，普通全局变量在各个源文件中都是有效的；静态全局变量则限制了其作用域，即只在定义该变量的源文件内有效，在同一源程序的其它源文件中不能使用它。由于静态全局变量的作用域限于一个源文件内，只能为该源文件内的函数公用，因此可以避免在其他源文件中引起错误。
+ 初始化：静态全局变量只初始化一次，防止在其他文件中使用。

### const 作用及用法

**作用：**

+ const 修饰成员变量，定义成 const 常量，相较于宏常量，可进行类型检查，节省内存空间，提高了效率。
+ const 修饰函数参数，使得传递过来的函数参数的值不能改变。
+ const 修饰成员函数，使得成员函数不能修改任何类型的成员变量（mutable 修饰的变量除外），也不能调用非 const 成员函数，因为非 const 成员函数可能会修改成员变量

**在类中的用法：**

const 成员变量：

1. const 成员变量只能在类内声明、定义，在构造函数初始化列表中初始化。
2. const 成员变量只在某个对象的生存周期内是常量，对于整个类而言却是可变的，因为类可以创建多个对象，不同类的 const 成员变量的值是不同的。因此不能在类的声明中初始化 const 成员变量，类的对象还没有创建，编译器不知道他的值。

const 成员函数：

1. 不能修改成员变量的值，除非有 mutable 修饰；只能访问成员变量。
2. 不能调用非常量成员函数，以防修改成员变量的值。

```cc
#include <iostream>
using namespace std;

class A
{
public:
	int var;
	A(int tmp) : var(tmp) {}
	void c_fun(int tmp) const // const 成员函数
	{
		var = tmp; // error: assignment of member 'A::var' in read-only object. 在 const 成员函数中，不能修改任何类成员变量。		
		fun(tmp); // error: passing 'const A' as 'this' argument discards qualifiers. const 成员函数不能调用非 const 成员函数，因为非 const 成员函数可能会修改成员变量。
	}

	void fun(int tmp)
	{
		var = tmp;
	}
};

int main()
{
    return 0;
}
```

### define 和 const 的区别

区别：

+ 编译阶段：define 是在编译预处理阶段进行替换，const 是在编译阶段确定其值。
+ 安全性：define 定义的宏常量没有数据类型，只是进行简单的替换，不会进行类型安全的检查；const 定义的常量是有类型的，是要进行判断的，可以避免一些低级的错误。
+ 内存占用：define 定义的宏常量，在程序中使用多少次就会进行多少次替换，内存中有多个备份，占用的是代码段的空间；const 定义的常量占用静态存储区的空间，程序运行过程中只有一份。
+ 调试：define 定义的宏常量不能调试，因为在预编译阶段就已经进行替换了；const 定义的常量可以进行调试。

const 的优点：

+ 有数据类型，在定义式可进行安全性检查。
+ 可调式。
+ 占用较少的空间。

### define 和 typedef 的区别

+ 原理：#define 作为预处理指令，在编译预处理时进行替换操作，不作正确性检查，只有在编译已被展开的源程序时才会发现可能的错误并报错。typedef 是关键字，在编译时处理，有类型检查功能，用来给一个已经存在的类型一个别名，但不能在一个函数定义里面使用 typedef 。
+ 功能：typedef 用来定义类型的别名，方便使用。#define 不仅可以为类型取别名，还可以定义常量、变量、编译开关等。
+ 作用域：#define 没有作用域的限制，只要是之前预定义过的宏，在以后的程序中都可以使用，而 typedef 有自己的作用域。
+ 指针的操作：typedef 和 #define 在处理指针时不完全一样

```cc
#include <iostream>
#define INTPTR1 int *
typedef int * INTPTR2;

using namespace std;

int main()
{
    INTPTR1 p1, p2; // p1: int *; p2: int
    INTPTR2 p3, p4; // p3: int *; p4: int *

    int var = 1;
    const INTPTR1 p5 = &var; // 相当于 const int * p5; 常量指针，即不可以通过 p5 去修改 p5 指向的内容，但是 p5 可以指向其他内容。
    const INTPTR2 p6 = &var; // 相当于 int * const p6; 指针常量，不可使 p6 再指向其他内容。
    
    return 0;
}

```

### inline 作用及使用方法

**作用：**
inline 是一个关键字，可以用于定义内联函数。内联函数，像普通函数一样被调用，但是在调用时并不通过函数调用的机制而是直接在调用点处展开，这样可以大大减少由函数调用带来的开销，从而提高程序的运行效率。

**使用方法：**

1. 类内定义成员函数默认是内联函数
   在类内定义成员函数，可以不用在函数头部加 inline 关键字，因为编译器会自动将类内定义的函数（构造函数、析构函数、普通成员函数等）声明为内联函数，代码如下：

```cc
#include <iostream>
using namespace std;

class A{
public:
    int var;
    A(int tmp){ 
      var = tmp;
    }    
    void fun(){ 
        cout << var << endl;
    }
};

int main()
{    
    return 0;
}
```

2. 类外定义成员函数，若想定义为内联函数，需用关键字声明
   当在类内声明函数，在类外定义函数时，如果想将该函数定义为内联函数，则可以在类内声明时不加 inline 关键字，而在类外定义函数时加上 inline 关键字。

```cc
#include <iostream>
using namespace std;

class A{
public:
    int var;
    A(int tmp){ 
      var = tmp;
    }    
    void fun();
};

inline void A::fun(){
    cout << var << endl;
}

int main()
{    
    return 0;
}
```

另外，可以在声明函数和定义函数的同时加上 inline；也可以只在函数声明时加 inline，而定义函数时不加 inline。只要确保在调用该函数之前把 inline 的信息告知编译器即可。

#### inline 函数工作原理

+ 内联函数不是在调用时发生控制转移关系，而是在编译阶段将函数体嵌入到每一个调用该函数的语句块中，编译器会将程序中出现内联函数的调用表达式用内联函数的函数体来替换。
+ 普通函数是将程序执行转移到被调用函数所存放的内存地址，当函数执行完后，返回到执行此函数前的地方。转移操作需要保护现场，被调函数执行完后，再恢复现场，该过程需要较大的资源开销。

### 宏定义（define）和内联函数（inline）的区别

+ 内联函数是在编译时展开，而宏在编译预处理时展开；在编译的时候，内联函数直接被嵌入到目标代码中去，而宏只是一个简单的文本替换。
+ 内联函数是真正的函数，和普通函数调用的方法一样，在调用点处直接展开，避免了函数的参数压栈操作，减少了调用的开销。而宏定义编写较为复杂，常需要增加一些括号来避免歧义。
+ 宏定义只进行文本替换，不会对参数的类型、语句能否正常编译等进行检查。而内联函数是真正的函数，会对参数的类型、函数体内的语句编写是否正确等进行检查。
  

### new 和 malloc 如何判断是否申请到内存？

+ malloc ：成功申请到内存，返回指向该内存的指针；分配失败，返回 NULL 指针。
+ new ：内存分配成功，返回该对象类型的指针；分配失败，抛出 bad_alloc 异常。

### delete 实现原理？delete 和 delete[] 的区别？

delete 的实现原理：

+ 首先执行该对象所属类的析构函数；
+ 进而通过调用 operator delete 的标准库函数来释放所占的内存空间。

delete 和 delete [] 的区别：

+ delete 用来释放单个对象所占的空间，只会调用一次析构函数；
+ delete [] 用来释放数组空间，会对数组中的每个成员都调用一次析构函数。

### new 和 malloc 的区别，delete 和 free 的区别

在使用的时候` new`、`delete` 搭配使用，`malloc`、`free` 搭配使用。

+ `malloc`、`free `是库函数，而`new`、`delete` 是关键字。
  `new `申请空间时，无需指定分配空间的大小，编译器会根据类型自行计算；`malloc `在申请空间时，需要确定所申请空间的大小。
+ `new` 申请空间时，返回的类型是对象的指针类型，无需强制类型转换，是类型安全的操作符；`malloc `申请空间时，返回的是` void*` 类型，需要进行强制类型的转换，转换为对象类型的指针。
+ `new` 分配失败时，会抛出 `bad_alloc` 异常，`malloc` 分配失败时返回空指针。
+ 对于自定义的类型，`new` 首先调用 `operator new() `函数申请空间（底层通过 malloc 实现），然后调用构造函数进行初始化，最后返回自定义类型的指针；`delete `首先调用析构函数，然后调用 `operator delete() `释放空间（底层通过 free 实现）。`malloc`、`free` 无法进行自定义类型的对象的构造和析构。
+ `new` 操作符从自由存储区上为对象动态分配内存，而 `malloc` 函数从堆上动态分配内存。（自由存储区不等于堆）

### malloc 的原理？malloc 的底层实现？

`malloc `的原理:

+ 当开辟的空间小于 128K 时，调用 brk() 函数，通过移动 _enddata 来实现；
+ 当开辟空间大于 128 K 时，调用` mmap() `函数，通过在虚拟地址空间中开辟一块内存空间来实现。

malloc 的底层实现：

+ `brk() `函数实现原理：向高地址的方向移动指向数据段的高地址的指针` _enddata`。
+ `mmap` 内存映射原理：
  1. 进程启动映射过程，并在虚拟地址空间中为映射创建虚拟映射区域；
  2. 调用内核空间的系统调用函数 `mmap()`，实现文件物理地址和进程虚拟地址的一一映射关系；
  3. 进程发起对这片映射空间的访问，引发缺页异常，实现文件内容到物理内存（主存）的拷贝。

### C 和 C++ struct 的区别？

+ 在 C 语言中 `struct` 是用户自定义数据类型；在 C++ 中 `struct `是抽象数据类型，支持成员函数的定义。
+ C 语言中 `struct `没有访问权限的设置，是一些变量的集合体，不能定义成员函数；C++ 中` struct `可以和类一样，有访问权限，并可以定义成员函数。
+ C 语言中` struct` 定义的自定义数据类型，在定义该类型的变量时，需要加上 `struct `关键字，例如：`struct A var`;，定义 A 类型的变量；而 C++ 中，不用加该关键字，例如：`A var`;

### struct 和 union 的区别

说明：union 是联合体，struct 是结构体。

区别：

1. 联合体和结构体都是由若干个数据类型不同的数据成员组成。使用时，联合体只有一个有效的成员；而结构体所有的成员都有效。
2. 对联合体的不同成员赋值，将会对覆盖其他成员的值，而对于结构体的对不同成员赋值时，相互不影响。
3. 联合体的大小为其内部所有变量的最大值，按照最大类型的倍数进行分配大小；结构体分配内存的大小遵循内存对齐原则。

```cc
#include <iostream>
using namespace std;

typedef union
{
    char c[10];
    char cc1; // char 1 字节，按该类型的倍数分配大小
} u11;

typedef union
{
    char c[10];
    int i; // int 4 字节，按该类型的倍数分配大小
} u22;

typedef union
{
    char c[10];
    double d; // double 8 字节，按该类型的倍数分配大小
} u33;

typedef struct s1
{
    char c;   // 1 字节
    double d; // 1（char）+ 7（内存对齐）+ 8（double）= 16 字节
} s11;

typedef struct s2
{
    char c;   // 1 字节
    char cc;  // 1（char）+ 1（char）= 2 字节
    double d; // 2 + 6（内存对齐）+ 8（double）= 16 字节
} s22;

typedef struct s3
{
    char c;   // 1 字节
    double d; // 1（char）+ 7（内存对齐）+ 8（double）= 16 字节
    char cc;  // 16 + 1（char）+ 7（内存对齐）= 24 字节
} s33;

int main()
{
    cout << sizeof(u11) << endl; // 10
    cout << sizeof(u22) << endl; // 12
    cout << sizeof(u33) << endl; // 16
    cout << sizeof(s11) << endl; // 16
    cout << sizeof(s22) << endl; // 16
    cout << sizeof(s33) << endl; // 24

    cout << sizeof(int) << endl;    // 4
    cout << sizeof(double) << endl; // 8
    return 0;
}

```

### class 和 struct 的异同

+ struct 和 class 都可以自定义数据类型，也支持继承操作。
+ struct 中默认的访问级别是 public，默认的继承级别也是 public；class 中默认的访问级别是 private，默认的继承级别也是 private。
+ 当 class 继承 struct 或者 struct 继承 class 时，默认的继承级别取决于 class 或 struct 本身， class（private 继承），struct（public 继承），即取决于派生类的默认继承级别。

```cc
struct A{}；
class B : A{}; // private 继承 
struct C : B{}； // public 继承
```

举例：

```cc
#include<iostream>

using namespace std;

class A{
public:
    void funA(){
        cout << "class A" << endl;
    }
};

struct B: A{ // 由于 B 是 struct，A 的默认继承级别为 public
public:
    void funB(){
        cout << "class B" << endl;
    }
};

class C: B{ // 由于 C 是 class，B 的默认继承级别为 private，所以无法访问基类 B 中的 printB 函数

};

int main(){
    A ex1;
    ex1.funA(); // class A

    B ex2;
    ex2.funA(); // class A
    ex2.funB(); // class B

    C ex3;
    ex3.funB(); // error: 'B' is not an accessible base of 'C'.
    return 0;
}

```

### volatile 的作用？是否具有原子性，对编译器有什么影响？

`volatile` 的作用：当对象的值可能在程序的控制或检测之外被改变时，应该将该对象声明为 violatile，告知编译器不应对这样的对象进行优化。

`volatile`不具有原子性。

`volatile `对编译器的影响：使用该关键字后，编译器不会对相应的对象进行优化，即不会将变量从内存缓存到寄存器中，防止多个线程有可能使用内存中的变量，有可能使用寄存器中的变量，从而导致程序错误。

#### 什么情况下一定要用 volatile， 能否和 const 一起使用？

使用` volatile `关键字的场景：

+ 当多个线程都会用到某一变量，并且该变量的值有可能发生改变时，需要用 volatile 关键字对该变量进行修饰；
+ 中断服务程序中访问的变量或并行设备的硬件寄存器的变量，最好用 volatile 关键字修饰。

`volatile `关键字和` const` 关键字可以同时使用，某种类型可以既是` volatile` 又是` const` ，同时具有二者的属性。

### 返回函数中静态变量的地址会发生什么？

```cc
#include <iostream>
using namespace std;

int * fun(int tmp){
    static int var = 10;
    var *= tmp;
    return &var;
}

int main() {
    cout << *fun(5) << endl;
    return 0;
}

/*
运行结果：
50
*/
```

说明：上述代码中在函数 `fun `中定义了静态局部变量 `var`，使得离开该函数的作用域后，该变量不会销毁，返回到主函数中，该变量依然存在，从而使程序得到正确的运行结果。但是，该静态局部变量直到程序运行结束后才销毁，浪费内存空间。

### extern C 的作用？

当 C++ 程序 需要调用 C 语言编写的函数，C++ 使用链接指示，即 `extern "C"` 指出任意非 C++ 函数所用的语言。

举例：

```cc
// 可能出现在 C++ 头文件<cstring>中的链接指示
extern "C"{
    int strcmp(const char*, const char*);
}
```

### sizeof(1==1) 在 C 和 C++ 中分别是什么结果？

C 语言代码：

```cc
#include<stdio.h>

void main(){
    printf("%d\n", sizeof(1==1));
}

/*
运行结果：
4
*/
```

C++ 代码：

```cc
#include <iostream>
using namespace std;

int main() {
    cout << sizeof(1==1) << endl;
    return 0;
}

/*
1
*/
```

### memmove函数的底层原理？

```cc
void *memcpy(void *dst, const void *src, size_t size)
{
    char *psrc;
    char *pdst;

    if (NULL == dst || NULL == src)
    {
        return NULL;
    }

    if ((src < dst) && (char *)src + size > (char *)dst) // 出现地址重叠的情况，自后向前拷贝
    {
        psrc = (char *)src + size - 1;
        pdst = (char *)dst + size - 1;
        while (size--)
        {
            *pdst-- = *psrc--;
        }
    }
    else
    {
        psrc = (char *)src;
        pdst = (char *)dst;
        while (size--)
        {
            *pdst++ = *psrc++;
        }
    }

    return dst;
}
```

### strcpy 函数有什么缺陷？

strcpy 函数的缺陷：strcpy 函数不检查目的缓冲区的大小边界，而是将源字符串逐一的全部赋值给目的字符串地址起始的一块连续的内存空间，同时加上字符串终止符，会导致其他变量被覆盖。

```cc
#include <iostream>
#include <cstring>
using namespace std;

int main()
{
    int var = 0x11112222;
    char arr[10];
    cout << "Address : var " << &var << endl;
    cout << "Address : arr " << &arr << endl;
    strcpy(arr, "hello world!");
    cout << "var:" << hex << var << endl; // 将变量 var 以 16 进制输出
    cout << "arr:" << arr << endl;
    return 0;
}

/*
Address : var 0x23fe4c
Address : arr 0x23fe42
var:11002164
arr:hello world!
*/
```

说明：从上述代码中可以看出，变量 `var `的后六位被字符串 `"hello world!"` 的 `"d!\0"` 这三个字符改变，这三个字符对应的 ASCII 码的十六进制为：`\0`(0x00)，`!`(0x21)，`d`(0x64)。
原因：变量` arr` 只分配的 10 个内存空间，通过上述程序中的地址可以看出` arr` 和` var `在内存中是连续存放的，但是在调用` strcpy` 函数进行拷贝时，源字符串 `"hello world!"` 所占的内存空间为 13，因此在拷贝的过程中会占用` var` 的内存空间，导致 `var`的后六位被覆盖。

### auto 类型推导的原理

auto 类型推导的原理：
编译器根据初始值来推算变量的类型，要求用 auto 定义变量时必须有初始值。编译器推断出来的 auto 类型有时和初始值类型并不完全一样，编译器会适当改变结果类型使其更符合初始化规则。

## 类相关

### 虚函数

**虚函数**：被 `virtual` 关键字修饰的成员函数，就是虚函数。

```cc
#include <iostream>
using namespace std;

class A
{
public:
    virtual void v_fun() // 虚函数
    {
        cout << "A::v_fun()" << endl;
    }
};
class B : public A
{
public:
    void v_fun()
    {
        cout << "B::v_fun()" << endl;
    }
};
int main()
{
    A *p = new B();
    p->v_fun(); // B::v_fun()
    return 0;
}
```

**纯虚函数：**

+ 纯虚函数在类中声明时，加上 =0；
+ 含有纯虚函数的类称为抽象类（只要含有纯虚函数这个类就是抽象类），类中只有接口，没有具体的实现方法；
+ 继承纯虚函数的派生类，如果没有完全实现基类纯虚函数，依然是抽象类，不能实例化对象。

说明：

+ 抽象类对象不能作为函数的参数，不能创建对象，不能作为函数返回类型；
+ 可以声明抽象类指针，可以声明抽象类的引用；
+ 子类必须继承父类的纯虚函数，并全部实现后，才能创建子类的对象。

#### 虚函数和纯虚函数的区别？

+ 虚函数和纯虚函数可以出现在同一个类中，该类称为抽象基类。（含有纯虚函数的类称为抽象基类）
+ 使用方式不同：虚函数可以直接使用，纯虚函数必须在派生类中实现后才能使用；
+ 定义形式不同：虚函数在定义时在普通函数的基础上加上 virtual 关键字，纯虚函数定义时除了加上virtual 关键字还需要加上 =0;
+ 虚函数必须实现，否则编译器会报错；
+ 对于实现纯虚函数的派生类，该纯虚函数在派生类中被称为虚函数，虚函数和纯虚函数都可以在派生类中重写；
+ 析构函数最好定义为虚函数，特别是对于含有继承关系的类；析构函数可以定义为纯虚函数，此时，其所在的类为抽象基类，不能创建实例化对象。

### 虚函数的实现机制

**实现机制**：虚函数通过虚函数表来实现。虚函数的地址保存在虚函数表中，在类的对象所在的内存空间中，保存了指向虚函数表的指针（称为“虚表指针”），通过虚表指针可以找到类对应的虚函数表。虚函数表解决了基类和派生类的继承问题和类中成员函数的覆盖问题，当用基类的指针来操作一个派生类的时候，这张虚函数表就指明了实际应该调用的函数。

**虚函数表相关知识点：**

+ 虚函数表存放的内容：类的虚函数的地址。
+ 虚函数表建立的时间：编译阶段，即程序的编译过程中会将虚函数的地址放在虚函数表中。
+ 虚表指针保存的位置：虚表指针存放在对象的内存空间中最前面的位置，这是为了保证正确取到虚函数的偏移量。

+ 虚函数表和类绑定，虚表指针和对象绑定。即类的不同的对象的虚函数表是一样的，但是每个对象都有自己的虚表指针，来指向类的虚函数表。

实例

```cc
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void B_fun1() { cout << "Base::B_fun1()" << endl; }
    virtual void B_fun2() { cout << "Base::B_fun2()" << endl; }
    virtual void B_fun3() { cout << "Base::B_fun3()" << endl; }
};

class Derive : public Base
{
public:
    virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};
int main()
{
    Base *p = new Derive();
    p->B_fun1(); // Base::B_fun1()
    return 0;
}

```

基类和派生类的继承关系：

![image-20210721194713308](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194713308.png)

基类的虚函数表：

![image-20210721194726001](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194726001.png)

派生类的虚函数表：

![image-20210721194739297](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194739297.png)

主函数中基类的指针 `p` 指向了派生类的对象，当调用函数 `B_fun1()` 时，通过派生类的虚函数表找到该函数的地址，从而完成调用。

### 单继承和多继承的虚函数表结构

**编译器处理虚函数表：**

+ 编译器将虚函数表的指针放在类的实例对象的内存空间中，该对象调用该类的虚函数时，通过指针找到虚函数表，根据虚函数表中存放的虚函数的地址找到对应的虚函数。
+ 如果派生类没有重新定义基类的虚函数 A，则派生类的虚函数表中保存的是基类的虚函数 A 的地址，也就是说基类和派生类的虚函数 A 的地址是一样的。
+ 如果派生类重写了基类的某个虚函数 B，则派生的虚函数表中保存的是重写后的虚函数 B 的地址，也就是说虚函数 B 有两个版本，分别存放在基类和派生类的虚函数表中。
+ 如果派生类重新定义了新的虚函数 C，派生类的虚函数表保存新的虚函数 C 的地址。

1. 单继承无虚函数覆盖的情况：

```cc
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void B_fun1() { cout << "Base::B_fun1()" << endl; }
    virtual void B_fun2() { cout << "Base::B_fun2()" << endl; }
    virtual void B_fun3() { cout << "Base::B_fun3()" << endl; }
};

class Derive : public Base
{
public:
    virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};
int main()
{
    Base *p = new Derive();
    p->B_fun1(); // Base::B_fun1()
    return 0;
}
```

基类和派生类的继承关系：

![image-20210721194846671](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194846671.png)

基类的虚函数表：

![image-20210721194908798](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194908798.png)

派生类的虚函数表：

![image-20210721194920915](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194920915.png)

2. 单继承有虚函数覆盖的情况：

```cc
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun1() { cout << "Base::fun1()" << endl; }
    virtual void B_fun2() { cout << "Base::B_fun2()" << endl; }
    virtual void B_fun3() { cout << "Base::B_fun3()" << endl; }
};

class Derive : public Base
{
public:
    virtual void fun1() { cout << "Derive::fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};
int main()
{
    Base *p = new Derive();
    p->fun1(); // Derive::fun1()
    return 0;
}
```

派生类的虚函数表：

![image-20210721194947621](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721194947621.png)

3. 多继承无虚函数覆盖的情况：

```cc
#include <iostream>
using namespace std;

class Base1
{
public:
    virtual void B1_fun1() { cout << "Base1::B1_fun1()" << endl; }
    virtual void B1_fun2() { cout << "Base1::B1_fun2()" << endl; }
    virtual void B1_fun3() { cout << "Base1::B1_fun3()" << endl; }
};
class Base2
{
public:
    virtual void B2_fun1() { cout << "Base2::B2_fun1()" << endl; }
    virtual void B2_fun2() { cout << "Base2::B2_fun2()" << endl; }
    virtual void B2_fun3() { cout << "Base2::B2_fun3()" << endl; }
};
class Base3
{
public:
    virtual void B3_fun1() { cout << "Base3::B3_fun1()" << endl; }
    virtual void B3_fun2() { cout << "Base3::B3_fun2()" << endl; }
    virtual void B3_fun3() { cout << "Base3::B3_fun3()" << endl; }
};

class Derive : public Base1, public Base2, public Base3
{
public:
    virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};

int main(){
    Base1 *p = new Derive();
    p->B1_fun1(); // Base1::B1_fun1()
    return 0;
}
```

基类和派生类的关系：

![image-20210721195016430](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721195016430.png)

派生类的虚函数表：（基类的顺序和声明的顺序一致）

![image-20210721195029617](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721195029617.png)

4. 多继承有虚函数覆盖的情况：

```cc
#include <iostream>
using namespace std;

class Base1
{
public:
    virtual void fun1() { cout << "Base1::fun1()" << endl; }
    virtual void B1_fun2() { cout << "Base1::B1_fun2()" << endl; }
    virtual void B1_fun3() { cout << "Base1::B1_fun3()" << endl; }
};
class Base2
{
public:
    virtual void fun1() { cout << "Base2::fun1()" << endl; }
    virtual void B2_fun2() { cout << "Base2::B2_fun2()" << endl; }
    virtual void B2_fun3() { cout << "Base2::B2_fun3()" << endl; }
};
class Base3
{
public:
    virtual void fun1() { cout << "Base3::fun1()" << endl; }
    virtual void B3_fun2() { cout << "Base3::B3_fun2()" << endl; }
    virtual void B3_fun3() { cout << "Base3::B3_fun3()" << endl; }
};

class Derive : public Base1, public Base2, public Base3
{
public:
    virtual void fun1() { cout << "Derive::fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};

int main(){
    Base1 *p1 = new Derive();
    Base2 *p2 = new Derive();
    Base3 *p3 = new Derive();
    p1->fun1(); // Derive::fun1()
    p2->fun1(); // Derive::fun1()
    p3->fun1(); // Derive::fun1()
    return 0;
}
```

基类和派生类的关系：

![image-20210721195100454](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721195100454.png)

派生类的虚函数表：

![image-20210721195110137](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721195110137.png)

### 如何禁止构造函数的使用？

为类的构造函数增加 `= delete` 修饰符，可以达到虽然声明了构造函数但禁止使用的目的。

```cc
#include <iostream>

using namespace std;

class A {
public:
    int var1, var2;
    A(){
        var1 = 10;
        var2 = 20;
    }
    A(int tmp1, int tmp2) = delete;
};

int main()
{
    A ex1;    
    A ex2(12,13); // error: use of deleted function 'A::A(int, int)'
    return 0;
}
```

### 什么是类的默认构造函数？

默认构造函数：未提供任何实参，来控制默认初始化过程的构造函数称为默认构造函数。

```cc
#include <iostream>

using namespace std;

class A
{
public:
    A(){ // 类的默认构造函数
        var = 10;
        c = 'q';
    }
    int var;
    char c;
};

int main()
{
    A ex;
    cout << ex.c << endl << ex.var << endl;
    return 0;
}
/*
运行结果：
q
10
*/
```

### 构造函数、析构函数是否需要定义成虚函数？为什么？

构造函数一般不定义为虚函数，原因：

+ 从存储空间的角度考虑：构造函数是在实例化对象的时候进行调用，如果此时将构造函数定义成虚函数，需要通过访问该对象所在的内存空间才能进行虚函数的调用（因为需要通过指向虚函数表的指针调用虚函数表，虽然虚函数表在编译时就有了，但是没有虚函数的指针，虚函数的指针只有在创建了对象才有），但是此时该对象还未创建，便无法进行虚函数的调用。所以构造函数不能定义成虚函数。
+ 从使用的角度考虑：虚函数是基类的指针指向派生类的对象时，通过该指针实现对派生类的虚函数的调用，构造函数是在创建对象时自动调用的。
+ 从实现上考虑：虚函数表是在创建对象之后才有的，因此不能定义成虚函数。
+ 从类型上考虑：在创建对象时需要明确其类型

析构函数一般定义成虚函数，原因：

析构函数定义成虚函数是为了防止内存泄漏，因为当基类的指针或者引用指向或绑定到派生类的对象时，如果未将基类的析构函数定义成虚函数，会调用基类的析构函数，那么只能将基类的成员所占的空间释放掉，派生类中特有的就会无法释放内存空间导致内存泄漏。

### 如何避免拷贝？

最直观的想法是：将类的拷贝构造函数和赋值构造函数声明为私有 private，但对于类的成员函数和友元函数依然可以调用，达不到完全禁止类的对象被拷贝的目的，而且程序会出现错误，因为未对函数进行定义。

解决方法：声明一个基类，具体做法如下。

+ 定义一个基类，将其中的拷贝构造函数和赋值构造函数声明为私有 private
+ 派生类以私有 private 的方式继承基类

```cc
class Uncopyable
{
public:
    Uncopyable() {}
    ~Uncopyable() {}

private:
    Uncopyable(const Uncopyable &);            // 拷贝构造函数
    Uncopyable &operator=(const Uncopyable &); // 赋值构造函数
};
class A : private Uncopyable // 注意继承方式
{ 
};
```

简单解释：

+ 能够保证，在派生类 A 的成员函数和友元函数中无法进行拷贝操作，因为无法调用基类 Uncopyable 的拷贝构造函数或赋值构造函数。同样，在类的外部也无法进行拷贝操作。

### 如何减少构造函数开销？

在构造函数中使用类初始化列表，会减少调用默认的构造函数产生的开销，具体原因可以参考本章“为什么用成员初始化列表会快些？”这个问题。

```
class A
{
private:
    int val;
public:
    A()
    {
        cout << "A()" << endl;
    }
    A(int tmp)
    {
        val = tmp;
        cout << "A(int " << val << ")" << endl;
    }
};
class Test1
{
private:
    A ex;

public:
    Test1() : ex(1) // 成员列表初始化方式
    {
    }
};
```

### 多重继承时会出现什么状况？如何解决？

多重继承（多继承）：是指从多个直接基类中产生派生类。

多重继承容易出现的问题：命名冲突和数据冗余问题。

举例:

```cc
#include <iostream>
using namespace std;

// 间接基类
class Base1
{
public:
    int var1;
};

// 直接基类
class Base2 : public Base1
{
public:
    int var2;
};

// 直接基类
class Base3 : public Base1
{
public:
    int var3;
};

// 派生类
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { var1 = tmp; } // error: reference to 'var1' is ambiguous. 命名冲突
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}

```

上述程序的继承关系如下：（菱形继承）

![image-20210721195507124](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721195507124.png)

**上述代码中存的问题：**

对于派生类 `Derive` 上述代码中存在直接继承关系和间接继承关系。

+ 直接继承：Base2 、Base3
+ 间接继承：Base1

对于派生类中继承的的成员变量 var1 ，从继承关系来看，实际上保存了两份，一份是来自基类 Base2，一份来自基类 Base3。因此，出现了命名冲突。

**解决方法 1： 声明出现冲突的成员变量来源于哪个类**

```cc
#include <iostream>
using namespace std;

// 间接基类
class Base1
{
public:
    int var1;
};

// 直接基类
class Base2 : public Base1
{
public:
    int var2;
};

// 直接基类
class Base3 : public Base1
{
public:
    int var3;
};

// 派生类 
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { Base2::var1 = tmp; } // 这里声明成员变量来源于类 Base2，当然也可以声明来源于类 Base3
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}
```

**解决方法 2： 虚继承**

使用虚继承的目的：保证存在命名冲突的成员变量在派生类中只保留一份，即使间接基类中的成员在派生类中只保留一份。在菱形继承关系中，间接基类称为虚基类，直接基类和间接基类之间的继承关系称为虚继承。

实现方式：在继承方式前面加上 virtual 关键字。

```
#include <iostream>
using namespace std;

// 间接基类，即虚基类
class Base1
{
public:
    int var1;
};

// 直接基类 
class Base2 : virtual public Base1 // 虚继承
{
public:
    int var2;
};

// 直接基类 
class Base3 : virtual public Base1 // 虚继承
{
public:
    int var3;
};

// 派生类
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { var1 = tmp; } 
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}
```

类之间的继承关系：

![image-20210721195631104](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210721195631104.png)

### 空类占多少字节？C++ 编译器会给一个空类自动生成哪些函数

**空类声明时编译器不会生成任何成员函数：**

对于空类，声明编译器不会生成任何的成员函数，只会生成 1 个字节的占位符。

**空类定义时编译器会生成 6 个成员函数：**

当空类 A 定义对象时，sizeof(A) 仍是为 1，但编译器会生成 6 个成员函数：缺省的构造函数、拷贝构造函数、析构函数、赋值运算符、两个取址运算符。

```cc
#include <iostream>
using namespace std;
/*
class A
{}; 该空类的等价写法如下：
*/
class A
{
public:
    A(){};                                       // 缺省构造函数
    A(const A &tmp){};                           // 拷贝构造函数
    ~A(){};                                      // 析构函数
    A &operator=(const A &tmp){};                // 赋值运算符
    A *operator&() { return this; };             // 取址运算符
    const A *operator&() const { return this; }; // 取址运算符（const 版本）
};

int main()
{
    A *p = new A(); 
    cout << "sizeof(A):" << sizeof(A) << endl; // sizeof(A):1
    delete p;       
    return 0;
}
```

### 为什么拷贝构造函数必须为引用？

原因：避免拷贝构造函数无限制的递归，最终导致栈溢出。

### C++ 类对象的初始化顺序

构造函数调用顺序：

+ 按照派生类继承基类的顺序，即派生列表中声明的顺序，依次调用基类的构造函数；
+ 按照派生类中成员变量的声名顺序，依次调用派生类中成员变量所属类的构造函数；

+ 执行派生类自身的构造函数。

综上可以得出，类对象的初始化顺序：基类构造函数–>派生类成员变量的构造函数–>自身构造函数

注：

- 基类构造函数的调用顺序与派生类的派生列表中的顺序有关；
- 成员变量的初始化顺序与声明顺序有关；
- 析构顺序和构造顺序相反。

```cc
#include <iostream>
using namespace std;

class A
{
public:
    A() { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
};

class B
{
public:
    B() { cout << "B()" << endl; }
    ~B() { cout << "~B()" << endl; }
};

class Test : public A, public B // 派生列表
{
public:
    Test() { cout << "Test()" << endl; }
    ~Test() { cout << "~Test()" << endl; }

private:
    B ex1;
    A ex2;
};

int main()
{
    Test ex;
    return 0;
}
/*
运行结果：
A()
B()
B()
A()
Test()
~Test()
~A()
~B()
~B()
~A()
*/
```

程序运行结果分析：

1. 首先调用基类 A 和 B 的构造函数，按照派生列表 public A, public B 的顺序构造；
2. 然后调用派生类 Test 的成员变量 ex1 和 ex2 的构造函数，按照派生类中成员变量声明的顺序构造；
3. 最后调用派生类的构造函数；
4. 接下来调用析构函数，和构造函数调用的顺序相反。

### 如何禁止一个类被实例化？

方法一：

- 在类中定义一个纯虚函数，使该类成为抽象基类，因为不能创建抽象基类的实例化对象；

方法二：

- 将类的构造函数声明为私有 `private`

### 为什么用成员初始化列表会快一些？

**说明：**数据类型可分为内置类型和用户自定义类型（类类型），对于用户自定义类型，利用成员初始化列表效率高。

**原因：**用户自定义类型如果使用类初始化列表，直接调用该成员变量对应的构造函数即完成初始化；如果在构造函数中初始化，因为 C++ 规定，对象的成员变量的初始化动作发生在进入构造函数本体之前，那么在执行构造函数的函数体之前首先调用默认的构造函数为成员变量设初值，在进入函数体之后，调用该成员变量对应的构造函数。因此，使用列表初始化会减少调用默认的构造函数的过程，效率高。

```cc
#include <iostream>
using namespace std;
class A
{
private:
    int val;
public:
    A()
    {
        cout << "A()" << endl;
    }
    A(int tmp)
    {
        val = tmp;
        cout << "A(int " << val << ")" << endl;
    }
};

class Test1
{
private:
    A ex;

public:
    Test1() : ex(1) // 成员列表初始化方式
    {
    }
};

class Test2
{
private:
    A ex;

public:
    Test2() // 函数体中赋值的方式
    {
        ex = A(2);
    }
};
int main()
{
    Test1 ex1;
    cout << endl;
    Test2 ex2;
    return 0;
}
/*
运行结果：
A(int 1)

A()
A(int 2)
*/

```

说明：
从程序运行结果可以看出，使用成员列表初始化的方式会省去调用默认的构造函数的过程。

### 实例化一个对象需要哪几个阶段

1. 分配空间
   创建类对象首先要为该对象分配内存空间。不同的对象，为其分配空间的时机未必相同。全局对象、静态对象、分配在栈区域内的对象，在编译阶段进行内存分配；存储在堆空间的对象，是在运行阶段进行内存分配。
2. 初始化
   首先明确一点：初始化不同于赋值。初始化发生在赋值之前，初始化随对象的创建而进行，而赋值是在对象创建好后，为其赋上相应的值。这一点可以联想下上一个问题中提到：初始化列表先于构造函数体内的代码执行，初始化列表执行的是数据成员的初始化过程，这个可以从成员对象的构造函数被调用看的出来。
3. 赋值
   对象初始化完成后，可以对其进行赋值。对于一个类的对象，其成员变量的赋值过程发生在类的构造函数的函数体中。当执行完该函数体，也就意味着类对象的实例化过程完成了。（总结：构造函数实现了对象的初始化和赋值两个过程，对象的初始化是通过初始化列表来完成，而对象的赋值则才是通过构造函数的函数体来实现。）

注：对于拥有虚函数的类的对象，还需要给虚表指针赋值。

+ 没有继承关系的类，分配完内存后，首先给虚表指针赋值，然后再列表初始化以及执行构造函数的函数体，即上述中的初始化和赋值操作。
+ 有继承关系的类，分配内存之后，首先进行基类的构造过程，然后给该派生类的虚表指针赋值，最后再列表初始化以及执行构造函数的函数体，即上述中的初始化和赋值操作。

### 友元函数的作用及使用场景

作用：友元提供了不同类的成员函数之间、类的成员函数与一般函数之间进行数据共享的机制。通过友元，一个不同函数或另一个类中的成员函数可以访问类中的私有成员和保护成员。

使用场景：

1. 普通函数定义为友元函数，使普通函数能够访问类的私有成员。

```cc
#include <iostream>

using namespace std;

class A
{
    friend ostream &operator<<(ostream &_cout, const A &tmp); // 声明为类的友元函数

public:
    A(int tmp) : var(tmp)
    {
    }

private:
    int var;
};

ostream &operator<<(ostream &_cout, const A &tmp)
{
    _cout << tmp.var;
    return _cout;
}

int main()
{
    A ex(4);
    cout << ex << endl; // 4
    return 0;
}
```

2. 友元类：类之间共享数据。

```cc
#include <iostream>

using namespace std;

class A
{
    friend class B;

public:
    A() : var(10){}
    A(int tmp) : var(tmp) {}
    void fun()
    {
        cout << "fun():" << var << endl;
    }

private:
    int var;
};

class B
{
public:
    B() {}
    void fun()
    {
        cout << "fun():" << ex.var << endl; // 访问类 A 中的私有成员
    }

private:
    A ex;
};

int main()
{
    B ex;
    ex.fun(); // fun():10
    return 0;
}

```

### 深拷贝和浅拷贝的区别

如果一个类拥有资源，该类的对象进行复制时，如果资源重新分配，就是深拷贝，否则就是浅拷贝。

+ 深拷贝：该对象和原对象占用不同的内存空间，既拷贝存储在栈空间中的内容，又拷贝存储在堆空间中的内容。
+ 浅拷贝：该对象和原对象占用同一块内存空间，仅拷贝类中位于栈空间中的内容

当类的成员变量中有指针变量时，最好使用深拷贝。因为当两个对象指向同一块内存空间，如果使用浅拷贝，当其中一个对象的删除后，该块内存空间就会被释放，另外一个对象指向的就是垃圾内存。

**浅拷贝实例**

```cc
#include <iostream>

using namespace std;

class Test
{
private:
	int *p;

public:
	Test(int tmp)
	{
		this->p = new int(tmp);
		cout << "Test(int tmp)" << endl;
	}
	~Test()
	{
		if (p != NULL)
		{
			delete p;
		}
		cout << "~Test()" << endl;
	}
};

int main()
{
	Test ex1(10);	
	Test ex2 = ex1; 
	return 0;
}
/*
运行结果：
Test(int tmp)
~Test()
*/
```

说明：上述代码中，类对象 ex1、ex2 实际上是指向同一块内存空间，对象析构时，ex2 先将内存释放了一次，之后 析构对象 ex1 时又将这块已经被释放过的内存再释放一次。对同一块内存空间释放了两次，会导致程序崩溃。

**深拷贝实例：**

```cc
#include <iostream>

using namespace std;

class Test
{
private:
	int *p;

public:
	Test(int tmp)
	{
		p = new int(tmp);
		cout << "Test(int tmp)" << endl;
	}
	~Test()
	{
		if (p != NULL)
		{
			delete p;
		}
		cout << "~Test()" << endl;
	}
	Test(const Test &tmp) // 定义拷贝构造函数
	{
		p = new int(*tmp.p);
		cout << "Test(const Test &tmp)" << endl;
	}

};
```

### 编译时多态和运行时多态的区别

编译时多态：在程序编译过程中出现，发生在模板和函数重载中（泛型编程）。
运行时多态：在程序运行过程中出现，发生在继承体系中，是指通过基类的指针或引用访问派生类中的虚函数。

编译时多态和运行时多态的区别：

+ 时期不同：编译时多态发生在程序编译过程中，运行时多态发生在程序的运行过程中；
+ 实现方式不同：编译时多态运用泛型编程来实现，运行时多态借助虚函数来实现。

### 实现一个类成员函数，要求不允许修改类的成员变量？

如果想达到一个类的成员函数不能修改类的成员变量，只需用 `const` 关键字来修饰该函数即可。

```cc
#include <iostream>

using namespace std;

class A
{
public:
    int var1, var2;
    A()
    {
        var1 = 10;
        var2 = 20;
    }
    void fun() const // 不能在 const 修饰的成员函数中修改成员变量的值，除非该成员变量用 mutable 修饰
    {
        var1 = 100; // error: assignment of member 'A::var1' in read-only object
    }
};

int main()
{
    A ex1;
    return 0;
}
```

### 如何让类不能被继承？

解决方法一：借助 `final` 关键字，用该关键字修饰的类不能被继承。

```cc
#include <iostream>

using namespace std;

class Base final
{
};

class Derive: public Base{ // error: cannot derive from 'final' base 'Base' in derived type 'Derive'

};

int main()
{
    Derive ex;
    return 0;
}
```

解决方法二：借助友元、虚继承和私有构造函数来实现

```cc
#include <iostream>
using namespace std;

template <typename T>
class Base{
    friend T;
private:
    Base(){
        cout << "base" << endl;
    }
    ~Base(){}
};

class B:virtual public Base<B>{   //一定注意 必须是虚继承
public:
    B(){
        cout << "B" << endl;
    }
};

class C:public B{
public:
    C(){}     // error: 'Base<T>::Base() [with T = B]' is private within this context
};


int main(){
    B b;  
    return 0;
}c
```

说明：在上述代码中 B 类是不能被继承的类。
具体原因：

虽然 Base 类构造函数和析构函数被声明为私有 private，在 B 类中，由于 B 是 Base 的友元，因此可以访问 Base 类构造函数，从而正常创建 B 类的对象；
B 类继承 Base 类采用虚继承的方式，创建 C 类的对象时，C 类的构造函数要负责 Base 类的构造，但是 Base 类的构造函数私有化了，C 类没有权限访问。因此，无法创建 C 类的对象， B 类是不能被继承的类。
注意：在继承体系中，友元关系不能被继承，虽然 C 类继承了 B 类，B 类是 Base 类的友元，但是 C 类和 Base 类没有友元关系。

## 语言特性相关

### 左值和右值的区别？左值引用和右值引用的区别，如何将左值转换成右值？

左值：指表达式结束后依然存在的持久对象。

右值：表达式结束就不再存在的临时对象。

左值和右值的区别：左值持久，右值短暂

右值引用和左值引用的区别：

+ 左值引用不能绑定到要转换的表达式、字面常量或返回右值的表达式。右值引用恰好相反，可以绑定到这类表达式，但不能绑定到一个左值上。
+ 右值引用必须绑定到右值的引用，通过 && 获得。右值引用只能绑定到一个将要销毁的对象上，因此可以自由地移动其资源。

std::move 可以将一个左值强制转化为右值，继而可以通过右值引用使用该值，以用于移动语义。

```cc
#include <iostream>
using namespace std;

void fun1(int& tmp) 
{ 
  cout << "fun1(int& tmp):" << tmp << endl; 
} 

void fun2(int&& tmp) 
{ 
  cout << "fun2(int&& tmp)" << tmp << endl; 
} 

int main() 
{ 
  int var = 11; 
  fun1(12); // error: cannot bind non-const lvalue reference of type 'int&' to an rvalue of type 'int'
  fun1(var);
  fun2(1); 
}
```

### std::move() 函数的实现原理

`std::move()` 函数原型：

```cc
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
	return static_cast<typename remove_reference<T>::type &&>(t);
}
```

说明：引用折叠原理

+ 右值传递给上述函数的形参 T&& 依然是右值，即 T&& && 相当于 T&&。
+ 左值传递给上述函数的形参 T&& 依然是左值，即 T&& & 相当于 T&。

小结：通过引用折叠原理可以知道，`move()` 函数的形参既可以是左值也可以是右值。

`remove_reference` 具体实现：

```cc
//原始的，最通用的版本
template <typename T> struct remove_reference{
    typedef T type;  //定义 T 的类型别名为 type
};
 
//部分版本特例化，将用于左值引用和右值引用
template <class T> struct remove_reference<T&> //左值引用
{ typedef T type; }
 
template <class T> struct remove_reference<T&&> //右值引用
{ typedef T type; }   
  
//举例如下,下列定义的a、b、c三个变量都是int类型
int i;
remove_refrence<decltype(42)>::type a;             //使用原版本，
remove_refrence<decltype(i)>::type  b;             //左值引用特例版本
remove_refrence<decltype(std::move(i))>::type  b;  //右值引用特例版本 

```

举例：

```cc
nt var = 10; 

转化过程：
1. std::move(var) => std::move(int&& &) => 折叠后 std::move(int&)

2. 此时：T 的类型为 int&，typename remove_reference<T>::type 为 int，这里使用 remove_reference 的左值引用的特例化版本

3. 通过 static_cast 将 int& 强制转换为 int&&

整个std::move被实例化如下
string&& move(int& t) 
{
    return static_cast<int&&>(t); 
}
```

总结：
std::move() 实现原理：

+ 利用引用折叠原理将右值经过 T&& 传递类型保持不变还是右值，而左值经过 T&& 变为普通的左值引用，以保证模板可以传递任意实参，且保持类型不变；
+ 然后通过 remove_refrence 移除引用，得到具体的类型 T；
+ 最后通过 static_cast<> 进行强制类型转换，返回 T&& 右值引用。

### 什么是指针？指针的大小及用法？

**指针：** 指向另外一种类型的复合类型。

**指针的大小：** 在 64 位计算机中，指针占 8 个字节空间。

### 什么是野指针和悬空指针？

悬空指针：

若指针指向一块内存空间，当这块内存空间被释放后，该指针依然指向这块内存空间，此时，称该指针为“悬空指针”。
举例：

```cc
void *p = malloc(size);
free(p); 
// 此时，p 指向的内存空间已释放， p 就是悬空指针。
```

野指针：

“野指针”是指不确定其指向的指针，未初始化的指针为“野指针”

```cc
void *p; 
// 此时 p 是“野指针”。
```

### C++ 11 nullptr 比 NULL 优势

NULL：预处理变量，是一个宏，它的值是 0，定义在头文件 <cstdlib> 中，即 #define NULL 0。
nullptr：C++ 11 中的关键字，是一种特殊类型的字面值，可以被转换成任意其他类型。
nullptr 的优势：

+ 有类型，类型是 typdef decltype(nullptr) nullptr_t;，使用 nullptr 提高代码的健壮性。
+ 函数重载：因为 NULL 本质上是 0，在函数调用过程中，若出现函数重载并且传递的实参是 NULL，可能会出现，不知和哪一个函数匹配的情况；但是传递实参 nullptr 就不会出现这种情况。

### 指针和引用的区别？

+ 指针所指向的内存空间在程序运行过程中可以改变，而引用所绑定的对象一旦绑定就不能改变。（是否可变）
+ 指针本身在内存中占有内存空间，引用相当于变量的别名，在内存中不占内存空间。（是否占内存）
+ 指针可以为空，但是引用必须绑定对象。（是否可为空）
+ 指针可以有多级，但是引用只能一级。（是否能为多级）

### 常量指针和指针常量的区别

**常量指针：**
常量指针本质上是个指针，只不过这个指针指向的对象是常量。
特点：const 的位置在指针声明运算符 * 的左侧。只要 const 位于 * 的左侧，无论它在类型名的左边或右边，都表示指向常量的指针。（可以这样理解，* 左侧表示指针指向的对象，该对象为常量，那么该指针为常量指针。）

```cc
const int * p;
int const * p;
```

注意 1：指针指向的对象不能通过这个指针来修改，也就是说常量指针可以被赋值为变量的地址，之所以叫做常量指针，是限制了通过这个指针修改变量的值。

注意 2：虽然常量指针指向的对象不能变化，可是因为常量指针本身是一个变量，因此，可以被重新赋值。

**指针常量**：
指针常量的本质上是个常量，只不过这个常量的值是一个指针。
特点：const 位于指针声明操作符右侧，表明该对象本身是一个常量，* 左侧表示该指针指向的类型，即以 * 为分界线，其左侧表示指针指向的类型，右侧表示指针本身的性质。

```cc
const int var;
int * const c_p = &var; 
```

注意 1：指针常量的值是指针，这个值因为是常量，所以指针本身不能改变。

注意 2：指针的内容可以改变。

### 函数指针和指针函数的区别

**指针函数：**
指针函数本质是一个函数，只不过该函数的返回值是一个指针。相对于普通函数而言，只是返回值是指针。

```cc
#include <iostream>
using namespace std;

struct Type
{
  int var1;
  int var2;
};

Type * fun(int tmp1, int tmp2){
    Type * t = new Type();
    t->var1 = tmp1;
    t->var2 = tmp2;
    return t;
}

int main()
{
    Type *p = fun(5, 6);
    return 0;
}
```

**函数指针：**
函数指针本质是一个指针变量，只不过这个指针指向一个函数。函数指针即指向函数的指针。

```cc
#include <iostream>
using namespace std;
int fun1(int tmp1, int tmp2)
{
  return tmp1 * tmp2;
}
int fun2(int tmp1, int tmp2)
{
  return tmp1 / tmp2;
}

int main()
{
  int (*fun)(int x, int y); 
  fun = fun1;
  cout << fun(15, 5) << endl; 
  fun = fun2;
  cout << fun(15, 5) << endl; 
  return 0;
}
/*
运行结果：
75
3
*/
```

**函数指针和指针函数的区别：**

+ 本质不同
  指针函数本质是一个函数，其返回值为指针。
  函数指针本质是一个指针变量，其指向一个函数。

### 强制类型转换有哪几种？

+  static_cast：用于数据的强制类型转换，强制将一种数据类型转换为另一种数据类型。
  1. 用于基本数据类型的转换。
  2. 用于类层次之间的基类和派生类之间 指针或者引用 的转换（不要求必须包含虚函数，但必须是有相互联系的类），进行上行转换（派生类的指针或引用转换成基类表示）是安全的；进行下行转换（基类的指针或引用转换成派生类表示）由于没有动态类型检查，所以是不安全的，最好用 dynamic_cast 进行下行转换。
  3. 可以将空指针转化成目标类型的空指针。
  4. 可以将任何类型的表达式转化成 void 类型

+ const_cast：强制去掉常量属性，不能用于去掉变量的常量性，只能用于去除指针或引用的常量性，将常量指针转化为非常量指针或者将常量引用转化为非常量引用（注意：表达式的类型和要转化的类型是相同的）。

+ reinterpret_cast：改变指针或引用的类型、将指针或引用转换为一个足够长度的整型、将整型转化为指针或引用类型。

+ dynamic_cast：
  1. 其他三种都是编译时完成的，动态类型转换是在程序运行时处理的，运行时会进行类型检查。
  2. 只能用于带有虚函数的基类或派生类的指针或者引用对象的转换，转换成功返回指向类型的指针或引用，转换失败返回 NULL；不能用于基本数据类型的转换。
  3. 在向上进行转换时，即派生类类的指针转换成基类类的指针和 static_cast 效果是一样的，（注意：这里只是改变了指针的类型，指针指向的对象的类型并未发生改变）

```CC
#include <iostream>
#include <cstring>

using namespace std;

class Base
{
};

class Derive : public Base
{
};

int main()
{
    Base *p1 = new Derive();
    Derive *p2 = new Derive();

    //向上类型转换
    p1 = dynamic_cast<Base *>(p2);
    if (p1 == NULL)
    {
        cout << "NULL" << endl;
    }
    else
    {
        cout << "NOT NULL" << endl; //输出
    }

    return 0;
}
```

​			4. 在下行转换时，基类的指针类型转化为派生类类的指针类型，只有当要转换的指针指向的对象类型和转化以后的对象类型相同时，才会转化成功。

```cc
#include <iostream>
#include <cstring>

using namespace std;

class Base
{
public:
    virtual void fun()
    {
        cout << "Base::fun()" << endl;
    }
};

class Derive : public Base
{
public:
    virtual void fun()
    {
        cout << "Derive::fun()" << endl;
    }
};

int main()
{
    Base *p1 = new Derive();
    Base *p2 = new Base();
    Derive *p3 = new Derive();

    //转换成功
    p3 = dynamic_cast<Derive *>(p1);
    if (p3 == NULL)
    {
        cout << "NULL" << endl;
    }
    else
    {
        cout << "NOT NULL" << endl; // 输出
    }

    //转换失败
    p3 = dynamic_cast<Derive *>(p2);
    if (p3 == NULL)
    {
        cout << "NULL" << endl; // 输出
    }
    else
    {
        cout << "NOT NULL" << endl;
    }

    return 0;
}
```

### 参数传递时，值传递、引用传递、指针传递的区别？

参数传递的三种方式：

+ 值传递：形参是实参的拷贝，函数对形参的所有操作不会影响实参。
+ 指针传递：本质上是值传递，只不过拷贝的是指针的值，拷贝之后，实参和形参是不同的指针，通过指针可以间接的访问指针所指向的对象，从而可以修改它所指对象的值。
+ 引用传递：当形参是引用类型时，我们说它对应的实参被引用传递。

```cc
#include <iostream>
using namespace std;

void fun1(int tmp){ // 值传递
    cout << &tmp << endl;
}

void fun2(int * tmp){ // 指针传递
    cout << tmp << endl;
}

void fun3(int &tmp){ // 引用传递
    cout << &tmp << endl;
}

int main()
{
    int var = 5;
    cout << "var 在主函数中的地址：" << &var << endl;

    cout << "var 值传递时的地址：";
    fun1(var);

    cout << "var 指针传递时的地址：";
    fun2(&var);

    cout << "var 引用传递时的地址：";
    fun3(var);
    return 0;
}

/*
运行结果：
var 在主函数中的地址：0x23fe4c
var 值传递时的地址：0x23fe20
var 指针传递时的地址：0x23fe4c
var 引用传递时的地址：0x23fe4c
*/
```

说明：从上述代码的运行结果可以看出，只有在值传递时，形参和实参的地址不一样，在函数体内操作的不是变量本身。引用传递和指针传递，在函数体内操作的是变量本身。

### 什么是模板？如何实现？

模板：创建类或者函数的蓝图或者公式，分为函数模板和类模板。
实现方式：模板定义以关键字 template 开始，后跟一个模板参数列表。

+ 模板参数列表不能为空；
+ 模板类型参数前必须使用关键字 class 或者 typename，在模板参数列表中这两个关键字含义相同，可互换使用。

```cc
template <typename T, typename U, ...>
```

函数模板：通过定义一个函数模板，可以避免为每一种类型定义一个新函数。

+ 对于函数模板而言，模板类型参数可以用来指定返回类型或函数的参数类型，以及在函数体内用于变量声明或类型转换。
+ 函数模板实例化：当调用一个模板时，编译器用函数实参来推断模板实参，从而使用实参的类型来确定绑定到模板参数的类型

```cc
#include<iostream>

using namespace std;

template <typename T>
T add_fun(const T & tmp1, const T & tmp2){
    return tmp1 + tmp2;
}

int main(){
    int var1, var2;
    cin >> var1 >> var2;
    cout << add_fun(var1, var2);

    double var3, var4;
    cin >> var3 >> var4;
    cout << add_fun(var3, var4);
    return 0;
}
```

类模板：类似函数模板，类模板以关键字 template 开始，后跟模板参数列表。但是，编译器不能为类模板推断模板参数类型，需要在使用该类模板时，在模板名后面的尖括号中指明类型。

```cc
#include <iostream>

using namespace std;

template <typename T>
class Complex
{
public:
    //构造函数
    Complex(T a, T b)
    {
        this->a = a;
        this->b = b;
    }

    //运算符重载
    Complex<T> operator+(Complex &c)
    {
        Complex<T> tmp(this->a + c.a, this->b + c.b);
        cout << tmp.a << " " << tmp.b << endl;
        return tmp;
    }

private:
    T a;
    T b;
};

int main()
{
    Complex<int> a(10, 20);
    Complex<int> b(20, 30);
    Complex<int> c = a + b;

    return 0;
}
```

### 函数模板和类模板的区别？

+ 实例化方式不同：函数模板实例化由编译程序在处理函数调用时自动完成，类模板实例化需要在程序中显式指定。
+ 实例化的结果不同：函数模板实例化后是一个函数，类模板实例化后是一个类。
+ 默认参数：类模板在模板参数列表中可以有默认参数。
+ 特化：函数模板只能全特化；而类模板可以全特化，也可以偏特化。
+ 调用方式不同：函数模板可以隐式调用，也可以显式调用；类模板只能显式调用。

### 什么是模板特化？为什么特化？

模板特化的原因：模板并非对任何模板实参都合适、都能实例化，某些情况下，通用模板的定义对特定类型不合适，可能会编译失败，或者得不到正确的结果。因此，当不希望使用模板版本时，可以定义类或者函数模板的一个特例化版本。

模板特化：模板参数在某种特定类型下的具体实现。分为函数模板特化和类模板特化

+ 函数模板特化：将函数模板中的全部类型进行特例化，称为函数模板特化。
+ 类模板特化：将类模板中的部分或全部类型进行特例化，称为类模板特化。

特化分为全特化和偏特化：

+ 全特化：模板中的模板参数全部特例化。
+ 偏特化：模板中的模板参数只确定了一部分，剩余部分需要在编译器编译时确定。

说明：要区分下函数重载与函数模板特化
定义函数模板的特化版本，本质上是接管了编译器的工作，为原函数模板定义了一个特殊实例，而不是函数重载，函数模板特化并不影响函数匹配。

实例：

```cc
#include <iostream>
#include <cstring>

using namespace std;
//函数模板
template <class T>
bool compare(T t1, T t2)
{
    cout << "通用版本：";
    return t1 == t2;
}

template <> //函数模板特化
bool compare(char *t1, char *t2)
{
    cout << "特化版本：";
    return strcmp(t1, t2) == 0;
}

int main(int argc, char *argv[])
{
    char arr1[] = "hello";
    char arr2[] = "abc";
    cout << compare(123, 123) << endl;
    cout << compare(arr1, arr2) << endl;

    return 0;
}
/*
运行结果：
通用版本：1
特化版本：0
*/
```

### include " " 和 <> 的区别

include<文件名> 和 #include"文件名" 的区别:

+ 查找文件的位置：include<文件名> 在标准库头文件所在的目录中查找，如果没有，再到当前源文件所在目录下查找；#include"文件名" 在当前源文件所在目录中进行查找，如果没有；再到系统目录中查找。
+ 使用习惯：对于标准库中的头文件常用 include<文件名>，对于自己定义的头文件，常用 #include"文件名"

### switch 的 case 里为何不能定义变量

switch 下面的这个花括号表示一块作用域，而不是每一个 case 表示一块作用域。如果在某一 case 中定义了变量，其作用域在这块花括号内，按理说在另一个 case 内可以使用该变量，但是在实际使用时，每一个 case 之间互不影响，是相对封闭的，参考如下实例。

### 迭代器的作用？

作用：在无需知道容器底层原理的情况下，遍历容器中的元素。

### 泛型编程如何实现？

泛型编程实现的基础：模板。模板是创建类或者函数的蓝图或者说公式，当时用一个 vector 这样的泛型，或者 find 这样的泛型函数时，编译时会转化为特定的类或者函数。

泛型编程涉及到的知识点较广，例如：容器、迭代器、算法等都是泛型编程的实现实例。面试者可选择自己掌握比较扎实的一方面进行展开。

+ 容器：涉及到 STL 中的容器，例如：vector、list、map 等，可选其中熟悉底层原理的容器进行展开讲解。
+ 迭代器：在无需知道容器底层原理的情况下，遍历容器中的元素。
+ 模板：可参考本章节中的模板相关问题。

## 设计模式

### 单例模式

**单例模式：**保证类的实例化对象仅有一个，并且提供一个访问他的全局访问点。

**应用场景：**

+ 表示文件系统的类，一个操作系统一定是只有一个文件系统，因此文件系统的类的实例有且仅有一个。
+ 打印机打印程序的实例，一台计算机可以连接好几台打印机，但是计算机上的打印程序只有一个，就可以通过单例模式来避免两个打印作业同时输出到打印机。

**实现方式：**

单例模式可以通过全局或者静态变量的形式实现，这样比较简单，但是这样会影响封装性，难以保证别的代码不会对全局变量造成影响。

+ **默认的构造函数、拷贝构造函数、赋值构造函数声明为私有的**，这样禁止在类的外部创建该对象；
+ 全局访问点也要定义成 **静态类型的成员函数**，没有参数，返回该类的指针类型。因为使用实例化对象的时候是通过类直接调用该函数，并不是先创建一个该类的对象，通过对象调用。


不安全的实现方式：

原因：考虑当两个线程同时调用 getInstance 方法，并且同时检测到 instance 是 NULL，两个线程会同时实例化对象，不符合单例模式的要求。

```cc
class Singleton{
private:
    static Singleton * instance;
    Singleton(){}
    Singleton(const Singleton& tmp){}
    Singleton& operator=(const Singleton& tmp){}
public:
    static Singleton* getInstance(){
        if(instance == NULL){
            instance = new Singleton();
        }
        return instance;
    }
};
Singleton* Singleton::instance = NULL;

```

分类：

懒汉模式：直到第一次用到类的实例时才去实例化，上面是懒汉实现。
饿汉模式：类定义的时候就实例化。
线程安全的懒汉模式实现：
方法：加锁
存在的问题：每次判断实例对象是否为空，都要被锁定，如果是多线程的话，就会造成大量线程阻塞。

```cc
class Singleton{
private:
    static pthread_mutex_t mutex;
    static Singleton * instance;
    Singleton(){
        pthread_mutex_init(&mutex, NULL); 
    }
    Singleton(const Singleton& tmp){}
    Singleton& operator=(const Singleton& tmp){}
public:
    static Singleton* getInstance(){
        pthread_mutex_lock(&mutex);
        if(instance == NULL){            
            instance = new Singleton();            
        }
        pthread_mutex_unlock(&mutex);
        return instance;
    }
};
Singleton* Singleton::instance = NULL;
pthread_mutex_t Singleton::mutex;

```

方法：**内部静态变量**，在全局访问点 `getInstance` 中定义静态实例。

```cc
class Singleton{
private:
    static pthread_mutex_t mutex;
    Singleton(){
        pthread_mutex_init(&mutex, NULL);
    }
    Singleton(const Singleton& temp){}
    Singleton& operator=(const Singleton& temp){}
public:
    static Singleton* getInstance(){ 
        static Singleton instance;
        return &instance;
    }
};
pthread_mutex_t Singleton::mutex; 
```

饿汉模式的实现：
饿汉模式本身就是线程安全的不用加锁。

```cc
class Singleton{
private:
    static Singleton* instance;
    Singleton(const Singleton& temp){}
    Singleton& operator=(const Singleton& temp){}
protected:
	 Singleton(){} 
public:
    static Singleton* getInstance(){ 
        return instance;    
    }
};
Singleton* Singleton::instance = new Singleton();

```

### 工厂模式

https://zhuanlan.zhihu.com/p/83535678