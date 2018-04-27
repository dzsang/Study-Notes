# 2018/4/8
## problem14
1. 连续输入直接用while(cin>>in)就好，win上用ctrl+'Z'结束输入；
2. string的寻找匹配字符串用find就好，如果没找到返回string::npos,找到返回匹配字符串的第一个的起始位置；   
---
    find_first_of 函数最容易出错的地方是和find函数搞混。它最大的区别就是如果在一个字符串str1中查找另一个字符串str2，如果str1中含有str2中的任何字符，则就会查找成功，而find则不同；
3. leetcode上面判题不要用runtime_error
4. 解题要注意时间，如果一段时间做不出来，直接
> **暴力求解，不要管优化**
5. 碰到数组类题目，可以考虑先排序后找规律，一般排序后可以从首尾两端开始遍历
# 4/11
## problem18
题目有要求保留唯一值，去掉重复的，可以用**set**保存结果
# 4/27
## problem19
member access within null pointer of type ‘struct ListNode’,是因为编译器不知道当前的结点与结点的下一个结点是否为NULL，而自己有没有声明，所以系统不确定，于是就报错。这也说明了链表中的结点使用的时候比其他数据结构要严格一些，一定要时刻保证使用的结点不为NULL，为NULL要做出相应处理。
## 
