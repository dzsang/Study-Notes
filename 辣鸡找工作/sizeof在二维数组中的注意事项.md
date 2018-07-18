# sizeof在二维数组中的注意事项
sizeof在C++ Primer中的示例定义如下
```
Sales_item item, *p;
sizeof(Sales_item); // size required to hold an object of type Sales_item
sizeof item; // size of item's type, e.g., sizeof(Sales_item)
sizeof *p;   // size of type to which p points, e.g., sizeof(Sales_item)
```
上面三种方式可以用来求Sales_item这个类型的大小，从这个例子可以看出sizeof的使用方式是
```
sizeof (type name);
sizeof (expr);
sizeof expr;
```
先复习一下最最基础的东西，打印sizeof对内建变量操作之后的结果。
```
int main(int argc, char* argv[])
{
     std::cout<<sizeof(int)<<std::endl;       //4
     std::cout<<sizeof(char)<<std::endl;      //1
     std::cout<<sizeof(long)<<std::endl;      //对32位机器,结果为4。对64位机器，结果为8。
     std::cout<<sizeof(float)<<std::endl;     //4
     std::cout<<sizeof(double)<<std::endl;    //8
     return 0;
}
```
现在看看sizeof与数组，特别是int型一维数组，char型一维数组，char型二维数组在一起都会发生什么。
```
int int_array[3]={5,6,7};
int int_array2[]={5,6,7,8};
std::cout<<sizeof(int_array)/sizeof(int)<<std::endl;
std::cout<<sizeof(int_array2)/sizeof(int)<<std::endl;
```
上面的答案分别是3,4。注意到其中int_array2[ ]的括号中没有写明数组大小依然是语法正确的。
把刚才的过程在char型数组上重复一遍如下
```
char char_array3[3]={'e','f','g'};
char char_array4[]={'e','f','g','h'};
std::cout<<sizeof(char_array3)/sizeof(char)<<std::endl;
std::cout<<sizeof(char_array4)/sizeof(char)<<std::endl;
```
上面的答案分别是3,4.同样地，注意到其中char_array4[ ]的括号中没有写明数组大小依然是语法正确的。

现在改用一个字符串去初始化字符数组，而不是一个字符一个字符地去初始化字符数组。看看sizeof的结果是什么。
```
char char_array5[4]={"efg"};
char char_array6[]={"efgh"};
std::cout<<sizeof(char_array5)/sizeof(char)<<std::endl;
std::cout<<sizeof(char_array6)/sizeof(char)<<std::endl;
```
答案是4,5。注意此处的答案不再是3,4了。因为字符串"efg"其实代表了字符'e', 'f',  'g', '\0'，末尾有字符串结束符'\0'。因此需要注意字符串所占据的空间比所能看见得到的空间多了一个'\0'，也就是大小上多了一个1.
下面这几个个比较迷惑人：
```
char* char_array7[]={"apple","peach","lotus"};
```
注意char_array7这个数组是个一维数组，数组中的元素的类型是char*而不是char，数组元素的类型是指向char的指针。由于字符串常量"apple"也代表一个常量指针，所以这个数组初始化方式是，使用字符串常量所表示的常量指针进行初始化。
```
char char_array8[2][6]={"apple","peach"};
```
注意char_array8这个数组是个二维数组，其中char_array8[2][6]中的2代表高维，6代表低维。也就是说，XXX[ ][ ][ ][ ]多维数组是按照维度从高到低的顺序进行书写的。这个数组表示了大小为6的一维数组一共有2个，它们组合在一起成了现在的二维数组。其中一维数组的初始化方式和char_array6[ ]的初始化方式类似。“apple”这个字符串包含了结尾的'\0'和前面5个字符，一共6个字符。
类似char_array6的简化，上面的char_array8也可以这么写。
```
char char_array9[][6]={"apple","peach"};
```
上面三个例子说明,字符串常量既可以整体表达一个常量指针，也可以分解使用其中的每个字符，赋值给一个一维数组。

现在用sizeof来看看上面两个例子的情况。
```
char *char_array7[]={"apple","peach","lotus"};
char char_array8[2][6]={"apple","peach"};
char char_array9[][6]={"apple","peach"};
std::cout<<sizeof(char_array7)/sizeof(char*)<<std::endl;
std::cout<<sizeof(char_array8)/sizeof(char)<<std::endl;
std::cout<<sizeof(char_array9)/sizeof(char)<<std::endl;
```
上面例子的正确答案是3,12,12。因为char_array7含有3个char*类型的指针；char_array8含有12个字符；char_array9含有12个字符。
注意多维数组在简化的时候，不能简化得太随意。
```
char char_array10[ ][ ]={"apple","peach"};//编译错误 multidimensional array must have bounds for all dimensions except the first
char char_array11[2][ ]={"apple","peach"};//编译错误 multidimensional array must have bounds for all dimensions except the first
```
这说明，在多维数组在简化的时候，只能省略一个括号中的数字，并且只能省略高维度的数字，也就是左侧的括号中的数字。否则编译器就爆炸了