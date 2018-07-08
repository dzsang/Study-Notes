# 第一章 STL概论与版本简介
1. STL提供六大组件，彼此可以组合套用：
    1.1 容器:各种数据结构，如vector、list、deque、set、map用来存放数据
    1.2 算法:各种常用算法如sort，search，copy，erase...
    1.3 迭代器
    1.4 仿函式(functors):行为类似函式，可作为算法的某种策略。从实作的角度看，仿函式是一种重载了operator()的class或class template。
    1.5 配接器(adapters):一种用来修饰容器或仿函式或迭代器接口的东西。例如STL提供的queue和stack，虽然看似容器，其实只能算是一种容器配接器，因为他们的底部完全借重deque，所有动作都由底层的deque供应。
    1.6 配置器(allocators):负责空间配置和管理。从实作的角度看，配置器是一个实现了动态空间配置、空间管理、空间释放的class template。
    ![](../markdown图片/STL六大组件的交互关系.png)
# 第二章 空间配置器
1. **ptrdiff_t**:两个指针相减的结果的类型为ptrdiff_t,它是一种有符号整数类型。减法运算的值为两个指针在内存中的距离（以数组元素的长度为单位，而非字节），因为减法运算的结果将除以数组元素类型的长度。所以该结果与数组中存储的元素的类型无关。 

2. 
```
template <class T>
inline T* allocate(ptrdiff_t size, T*) {
    set_new_handler(0);
    T* tmp = (T*)(::operator new((size_t)(size * sizeof(T))));
    if (tmp == 0) {
        cerr << "out of memory" << endl;
        exit(1);
    }
    return tmp;
}

template <class T>
    inline void deallocate(T* buffer) {
        ::operator delete(buffer);
}
```

3. 