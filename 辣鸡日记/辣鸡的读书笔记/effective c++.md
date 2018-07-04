# 让自己习惯C++
## 6/28
1. 类的构造函数问题
```
class Widget {
public:
    Widget();   default构造
    Widget(const Widget &rhs);  copy构造
    Widget& operator=(const Widget& rhs);   copy assignment构造
    ...
}
Widget w1;        default构造   
Widget w2(w1);    copy构造  
w2 = w1;          copy assignment构造   
```
但是  Widget w3 = w2;  却是调用的copy构造函数；     
因为<font color=#0099ff >**如果一个新对象被定义**</font>(如w3)，一定会有个构造函数被调用，不可能调用赋值操作。如果没有新对象被定义，(例如前述的 w2 = w1)，就不会有构造函数被调用，那么当然就是赋值操作被调用。    

如果函数调用类是传值而不是传引用，那么会先使用copy构造出一个临时变量。

2. 尽量以const enum inline 替换#define宏    
取一个const的地址是合法的，但是取一个enume的地址不合法，取一个#define的地址也不合法，这样可以实现不让别人活得一个pointer或reference指向你的某个整数常量的约束。  
3. const std::vector<int>::iterator iter=vec.begin()    iter指向的内容可以变，但是这个迭代器不能指向其他位置，例如不能进行iter++操作；  
然而  const_iterator和上面正好相反，它限定迭代器指向的内容不可改变，而迭代器自身可以移动改变
4. 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。详见P24  
<font color=#0099ff >const </font>
# 构造/析构/赋值运算
## 6/30
1. 将成员函数声明为private且故意不实现它们，可以避免编译器默认合成函数从而避免别人拷贝访问它们，甚至member函数和friend函数都不能调用它们。  
或者是声明一个base class(专门问了阻止copying动作而设计的，思想和上面的一样)，然后让有要求的类去继承它，
2. 析构函数绝对不要突出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们(不传播)或者结束程序。 
如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。
3. 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class(比起当前执行构造函数和析构函数的那层)
4. 令赋值(assignment)操作符返回一个reference to **this*
## 7/3
1. 在operator=中处理自我赋值问题，让其具“自我赋值安全性”和“异常安全性”  
>详解C++右值引用
[https://blog.csdn.net/renwotao2009/article/details/46335859](https://blog.csdn.net/renwotao2009/article/details/46335859)
2. 
- copying函数应该确保复制“对象内的所有成员变量”以及“所有base class”成分    
- 不要尝试以某个copying函数实现另一个copying函数。应该将共同机制放进第三个函数中，并由两个copying函数共同使用。
# 资源管理
3. 不要轻易创造使用“未加工指针”，为防止资源泄露，请使用RAII对象，他们在构造函数中获得资源并在西沟函数中释放资源。
两个常用的RAII classes分别是tr1：：shared_ptr和auto_ptr，前者通常比较简单，详见c++primer；若选择auto_ptr，复制动作会使它(被复制者)指向null。
>RAII：Resource Acquisition Is Initialization,资源取得时机便是初始化时机。
4. tr1：：shared_ptr润徐指定所谓的“删除器”(deleter)，那是一个函数或者函数对象(function object)，当引用次数为0时便被调用(此机能并不存在于auto_ptr——它总是将其指针删除)。删除器对tr1：：shared_ptr构造函数而言是可有可无的第二参数。
5. tr1：：shared_ptr和auto_ptr都提供一个get成员函数，用来执行显式转换，也就是它会返回智能指针内部的院士指针(的复件)。它们也重载了指针取值操作符(operator->和operator*)，它们允许隐式转换至底部原始指针。
6. 如果