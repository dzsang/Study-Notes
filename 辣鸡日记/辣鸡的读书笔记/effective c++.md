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
两个常用的RAII classes分别是tr1::shared_ptr和auto_ptr，前者通常比较简单，详见c++primer；若选择auto_ptr，复制动作会使它(被复制者)指向null。
>RAII：Resource Acquisition Is Initialization,资源取得时机便是初始化时机。
4. tr1::shared_ptr润徐指定所谓的“删除器”(deleter)，那是一个函数或者函数对象(function object)，当引用次数为0时便被调用(此机能并不存在于auto_ptr——它总是将其指针删除)。删除器对tr1::shared_ptr构造函数而言是可有可无的第二参数。
5. tr1::shared_ptr和auto_ptr都提供一个get成员函数，用来执行显式转换，也就是它会返回智能指针内部的院士指针(的复件)。它们也重载了指针取值操作符(operator->和operator*)，它们允许隐式转换至底部原始指针。
6. 如果在new表达式中使用[]，必须在相应的delete表达式中也使用[]。如果在new表达式中不使用[]，一定不要再相应的delete表达式中使用[]。
7. 以独立语句将newed对象存储于(置入)智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄露。
# 设计与声明
## 7/4
1. 
- 好的接口很容易被使用，不容易被误用。你应该在你的所有接口中努力达成这些性质
- “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容
- “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象之，以及消除客户的资源管理责任
- tr1::shared_ptr支持定制删除器，可防范DLL问题，可被用来自动解除互斥锁(mutexes)
2. 只要函数不会改变传入参数的值，传引用总是比传值要好，简单快速且不会有slicing(对象切割)问题(派生类对象传值给基类参数，会损失派生类对象的特性而表现为一个基类对象)。    
但是对于内置类型以及STL的迭代器和函数对象，传值往往比较适当。
3. 必须返回对象时，别妄想返回其引用。
4. 切忌将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允许约束条件获得保证，并提供class作者以充分的实现弹性。   
protected并不比public更具封装性。
5. 如果你需要为某个函数的所有参数(包括被this指针所致的那个隐喻参数)进行类型转换，那么这个函数必须是个non-member。       
(尽量避免friend函数)
6. 如果你觉得swap的缺省实现版的效率不足(那几乎总是意味着你的class或template使用了某种pimpl手法)，试着做以下事情：
- 提供一个public swap成员函数，让它高效地置换你的类型的两个对象值，并且这个函数绝不该抛出异常
- 在你的class或template所在的命名空间内提供一个non-member swap，并令它调用上述swap成员函数
- 如果你正在编写一个class(而非class template)，为你的class特化std::swap，并令他调用你的swap成员函数。
最后，如果你调用swap，请确定包含一个using声明，以便让std::swap在你的函数内曝光可见吗，然后不加任何namespace修饰符，赤裸裸地调用swap
# 7/6 
1. 通过default构造函数构造一个对象然后对它赋值比直接在构造时指定初值效率差。
2. 尽量延后变量定义式的出现，这样做可增加程序的清晰度并改善程序效率。
3. 
- const_cast:将对象的常量特性去除
- dynamic_cast:主要用来执行“安全向下转型”，也就是用来决定某对象是否归属继承体系中的某个类型。他是唯一无法由旧式语法执行的动作，也是唯一可能耗费重大运行成本的转型动作
- reinterpret-cast:意图执行低级转型，时机动作(及结果)可能取决于编译器，这就表示他不可移植。例如将一个pointer to int转型成一个int。这一类转型在低级代码以外很少见。
- static_cast:用来强迫隐式转换，例如将con-const对象转为const对象，或将int转为double等等。他也可以用来执行上述多种转换的反向，例如将void*指针转换成typed指针，将基类指针转换成派生类指针。但它无法将const转为non-const。
4. 单一对象(例如一个类型为derived的对象)可能拥有一个以上的地址例如“以base* 指向它时的地址”和以derived* 指向它时的地址)
5. 使用dynamic_cast通常是因为你想在一个你认定为derived class对象身上执行derived vlass操作函数，但你的手上却只有一个“指向base”的pointer或者reference，你只能靠它们来处理对象
6. 
- 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_cast。如果有个设计需要转型动作，试着发展无需转型的替代设计。
- 如果转型是必要的，试着将它隐藏在某个函数背后。客户随后可以调用该函数，而不需将转型方进他们自己的代码内。
- 宁可使用C++-style转型，不要使用旧式转型
7. 避免返回handles(包括reference、指针、迭代器)指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”(hangling handles)的可能性降至最低。
8. Inline函数通常一定被置于头文件内，Inlining在大多数c++程序中是编译期行为，为了将一个“函数调用”替换为“被调用函数的本体”，编译器必须知道那个函数长什么样子。    
Templates通常也被置于头文件内，应为它一旦被使用，编译器为了将它具现化，需要知道它长什么样子。    
编译器可能拒接将函数本体inlining或者是无法inlining。    
大部分调试器面对inline函数都束手无策。
9. 将大多数inlining限制在小型、被频繁调用的函数上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。     
不要因为function template出现在头文件，就将它们声明为inline。
10. 将文件的编译依存关系将至最低    
- (尽量使用前置声明而不是引用头文件)
- 如果使用对象引用或对象指针可以完成任务， 就不要使用objects
- 如果能够，尽量以class声明式替换class定义式
- 为声明式和定义式提供不同的头文件
# 继承与面向对象设计
## 7/8
1. 确定你的public继承塑模出is-a关系     
“public继承”以为is-a。适用于base classes身上的没意见事情一定也适用于derived classes身上，因为每一个derived classes对象也都是一个base classes对象。
2. 避免遮掩继承而来的名称
derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此。为了让被遮掩的名称重见天日，可使用using声明式或转交函数。
3. 区分接口继承和实现继承
- 接口继承和实现继承不同。在public继承之下，derived classes总是继承base class的接口
- pure virtual函数只具体指定接口继承
- 简朴的(非纯)impure virtual函数具体指定接口继承及缺省实现继承
- non-virtual函数具体指定接口继承以及强制性实现继承
4. 考虑virtual函数以外的其他选择
<font color=#00ffff>条款35不太明白<font>
5. 绝不重新定义继承而来的non-virtual函数
6. 绝不重新定义继承而来的缺省参数值
静态绑定又名前期绑定，动态绑定又名后期绑定      
静态绑定下派生类的虚函数并不从其基类继承缺省参数值，但若以指针或引用调用次函数，可以不指定参数值，因为动态绑定下这个函数会从其base继承缺省参数值
7. 通过复合塑模出has-a或“根据某物实现出”
- 如果classes之间的继承关系是private，编译器不会自动将一个derived class对象
- 由private base class继承而来的所有成员，在derived class中都会变成private属性，纵使它们在base class中原本是protected或public属性
8. 明智而审慎地使用private继承
- 独立(非附属)对象的大小一定不为零
- 空白基类最优化(EBO)，条件是在单一继承而非多重继承下
- private继承以为is-implemented-in-terms-of(根据某物实现出).它通常比复合(composition)的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计师合理的。
9. 明智而审慎地使用多重继承
### C++中虚拟继承的概念
为了解决从不同途径继承来的同名的数据成员在内存中有不同的拷贝造成数据不一致问题，将共同基类设置为虚基类。这时从不同的路径继承过来的同名数据成员在内存中就只有一个拷贝，同一个函数名也只有一个映射。这样不仅就解决了二义性问题，也节省了内存，避免了数据不一致的问题。    
class 派生类名：virtual 继承方式  基类名    
virtual是关键字，声明该基类为派生类的虚基类。   
在多继承情况下，虚基类关键字的作用范围和继承方式关键字相同，只对紧跟其后的基类起作用。声明了虚基类之后，虚基类在进一步派生过程中始终和派生类一起，维护同一个基类子对象的拷贝        

**◇执行顺序**   

首先执行虚基类的构造函数，多个虚基类的构造函数按照被继承的顺序构造；
执行基类的构造函数，多个基类的构造函数按照被继承的顺序构造；
执行成员对象的构造函数，多个成员对象的构造函数按照申明的顺序构造；
执行派生类自己的构造函数；
析构以与构造相反的顺序执行；

<font color = red>mark<font>

从虚基类直接或间接派生的派生类中的构造函数的成员初始化列表中都要列出对虚基类构造函数的调用。但只有用于建立对象的最派生类的构造函数调用虚基类的构造函数，而该派生类的所有基类中列出的对虚基类的构造函数的调用在执行中被忽略，从而保证对虚基类子对象只初始化一次。    
在一个成员初始化列表中同时出现对虚基类和非虚基类构造函数的调用时，虚基类的构造函数先于非虚基类的构造函数执行。

- 使用虚继承的那些classes所产生的对象往往比使用非虚继承的兄弟们体积大，访问虚基类的成员变量时，也比访问非虚基类的成员变量速度慢。
- virtual base的初始化责任是由继承体系中的最低层class负责，这暗示着
    (1)classes若派生自virtual bases而需要初始化，必须认知其virtual bases——不论那些bases距离多远
    (2)当一个新的derived class加入继承体系中，它必须承担起virtual bases（不论直接或间接）的初始化责任
# 模板与泛型编程
7/9
1. 了解隐式接口和编译器多态
- 声明template参数时，前缀关键字class和typename可互换
- 任何时候当你想要在template中指涉UI和嵌套从属名称，就必须在紧临它的前一个位置放上关键词typename。
- 不得在base class lists或member initialization list(成员初值列)内以它作为base class修饰符
2. 学习处理模板化基类内的名称
- class定义式最前头的“template<>”语法象征这既不是template也不是标准class，而是个特化版的A template，在template实参是B时被使用。这是所谓的模板全特化
```
template<>
class A<B> {
public:
    ...
}
```
3. 将与参数无关的代码抽离templates
>这一章难度有点大，以后再看一遍

# 定制new和delete
