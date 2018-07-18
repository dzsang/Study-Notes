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
# 6/20
problem26   
1. 容器的erase(b,e)中的e是指要删除元素的下一个迭代器位置，返回的也是删除的最后一个元素的迭代器位置
2. 不要随便用-1当flag值，可能是输入的一种，用INT_MAX或者INT_MIN     

problem26   移出数组重复数据
```
int count = 0;
for(int i = 1; i < n; i++){
    if(A[i] == A[i-1]) count++;
    else A[i-count] = A[i];
}
return n-count;
```

problem28   实现strstr()
```
class Solution {
public:
    int strStr(string haystack, string needle) {
        size_t it = haystack.find(needle);
        return it != string::npos?it:-1;
    }
};
```

problem29   
> abs(INT32_MIN) == INT32_MIN   
llabs(INT32_MIN) == -INT32_MIN == INT32_MAX+1   
这里是由于机器中存放的是补码，最高位是符号位占掉一位，INT32_MIN去反溢出后符号位还是1，所以是-2147483648     
(ps:按照csapp的求补码方法，1000 = -1*2^4 = -8， 1111 = -1 *2^3 + 2^2 + 2^1 + 2^0 = -1)

# 6/21
problem 30    
这题本来是先把所有的words排列组合起来，但是开销太大了，要先求n！个结果还要保存，这是不能接受的      
然后注意到题目给定的单词的长度都是一样的，采取网上的方法，对类似的题目采用滑动窗口方法来做。  

```
class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        vector<int> res;
        if (s.empty() || words.empty())
            return res;
        unordered_map<string, int> wordTimes;
        for (int i = 0; i < words.size(); ++i){
            if (wordTimes.count(words[i]) == 0)
                wordTimes.insert(make_pair(words[i], 1));
            else wordTimes[words[i]]++;
        }
        int wordLen = words[0].size();

        for(int i = 0; i < wordLen; i++)
        {//为了不遗漏从s的每一个位置开始的子串，第一层循环为单词的长度
            unordered_map<string, int>wordTimes2;//当前窗口中单词出现的次数
            int winStart = i, cnt = 0;//winStart为窗口起始位置,cnt为当前窗口中的单词数目
            for(int winEnd = i; winEnd <= (int)s.size()-wordLen; winEnd+=wordLen)
            {//窗口为[winStart,winEnd)
                string word = s.substr(winEnd, wordLen);
                if(wordTimes.find(word) != wordTimes.end())
                {
                    if(wordTimes2.find(word) == wordTimes2.end())
                        wordTimes2[word] = 1;
                    else wordTimes2[word]++;

                    if(wordTimes2[word] <= wordTimes[word])
                        cnt++;
                    else
                    {//当前的单词在words中，但是它已经在窗口中出现了相应的次数，不应该加入窗口
                        //此时，应该把窗口起始位置向左移动到，该单词第一次出现的位置的下一个单词位置
                        for(int k = winStart; ; k += wordLen)
                        {
                            string tmpstr = s.substr(k, wordLen);
                            wordTimes2[tmpstr]--;
                            if(tmpstr == word)
                            {
                                winStart = k + wordLen;
                                break;
                            }
                            cnt--;
                        }
                    }
                    if(cnt == words.size())
                        res.push_back(winStart);
                }
                else
                {//发现不在L中的单词
                    winStart = winEnd + wordLen;
                    wordTimes2.clear();
                    cnt = 0;
                }
            }
        }
        return res;
    }
};
```  
problem32   
括号匹配之类的题目第一时间想到的就是栈，但是在压栈出栈的时候可以用索引值而不是对象本身，方便求长度之类的；  
然后这题的括号可以直接左右2次扫描来求，不用额外辅助空间
```
    int longestValidParentheses(string s) {
        int maxlen = 0;
        int left = 0, right = 0;
        for (int i = 0; i < s.size(); ++i){
            if (s[i] == 'c')
                left++;
            else right++;
            if (left == right)
                maxlen = max(maxlen, 2*right);
            else if (right >= left)
                left = right = 0;
        }
        left = right = 0;
        for (int i = s.size() - 1; i >= 0; i--) {
            if (s[i] == '(')
                left++;
            else
                right++;
            if (left == right) {
                maxlen = max(maxlen, 2*left);
            } else if (left >= right)
                left = right = 0;
        }
        return maxlen;
    }
```
problem34   
size_t是无符号数，用作循环条件使用的时候要小心减到负值后出错，编译器是不会报错的。

problem35   
vector的find用法，vector没有自带find函数，只能用通用的。
 vector<int>::iterator result = find( nums.begin(), nums.end(), target );
 vector.insert 是在迭代器指向的地方之前位置插入，返回指向插入值的迭代器
 
problem36   
string用char初始化方式: string s(n, 'c') 把s初始化为由连续n个字符c组成的串  

problem 37 数独求解  
```
class Solution {
public:
    void solveSudoku(vector<vector<char> > &board) {
        for(int i = 0; i < 9; i++)
            for(int j = 0; j < 9; j++)
                if(board[i][j] != '.')
                    fill(i, j, board[i][j] - '0');
        solver(board, 0);
    }
     
    bool solver(vector<vector<char> > &board, int index)
    {// 0 <= index <= 80，index表示接下来要填充第index个格子
        if(index > 80)return true;
        int row = index / 9, col = index - 9*row;
        if(board[row][col] != '.')
            return solver(board, index+1);
        for(int val = '1'; val <= '9'; val++)//每个为填充的格子有9种可能的填充数字
        {
            if(isValid(row, col, val-'0'))
            {
                board[row][col] = val;
                fill(row, col, val-'0');
                if(solver(board, index+1))return true;
                clear(row, col, val-'0');
            }
        }
        board[row][col] = '.';//注意别忘了恢复board状态
        return false;
    }
     
    //判断在第row行col列填充数字val后，是否是合法的状态
    bool isValid(int row, int col, int val)
    {
        if(rowValid[row][val] == 0 &&
           columnValid[col][val] == 0 &&
           subBoardValid[row/3*3+col/3][val] == 0)
           return true;
        return false;
    }
     
    //更新填充状态
    void fill(int row, int col, int val)
    {
        rowValid[row][val] = 1;
        columnValid[col][val] = 1;
        subBoardValid[row/3*3+col/3][val] = 1;
    }
     
    //清除填充状态
    void clear(int row, int col, int val)
    {
        rowValid[row][val] = 0;
        columnValid[col][val] = 0;
        subBoardValid[row/3*3+col/3][val] = 0;
    }
private:
    int rowValid[9][10];//rowValid[i][j]表示第i行数字j是否已经使用
    int columnValid[9][10];//columnValid[i][j]表示第i列数字j是否已经使用
    int subBoardValid[9][10];//subBoardValid[i][j]表示第i个小格子内数字j是否已经使用
};
```
# 6/28
DFS算法思想:一直往深处走，知道找到解或者走不下去为止
```
DFS(dep,...)//dep代表目前DFS的深度
{
    if (找到解)//设置边界条件
    {
        ...
        return;
    }
    枚举下一种情况，DFS(dep+1,...)
    //这里要注意，如果题目需要保存解的过程，前面压栈了的数据要释放
    
    return ans;
}
```
BFS算法:先进先出FIFO实现
```
初始化队列Q
Q={起点s};
标记s为已访问;
while (Q非空){
    取Q队首元素;u出队;
    if (u == 目标状态) {...}
    所有与u相邻且未被访问的点进入队列;
    标记u为已访问;
}
```
# 7/3 
<font color=red size=3 face="微软雅黑">
HARD
</font>

[problem 42：Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/solution/)
# 7/4
problem43  Multiply Strings     
我自己的方法
```
class Solution {
public:
    string multiply(string num1, string num2) {
        if (num1 == "0" || num2 == "0")
            return "0";
        reverse(num1.begin(), num1.end());
        reverse(num2.begin(), num2.end());
        for (int i = 0; i < num2.size(); ++i){
            char cf = '0';
            string temp = "";
            for (auto &d: num1){
                temp += ((num2[i]-'0')*(d-'0')+(cf-'0'))%10+'0';
                cf = ((num2[i]-'0')*(d-'0')+(cf-'0'))/10+'0';
            }
            if (cf != '0')
                temp += cf;
            reverse(temp.begin(), temp.end());
            for (int j = i; j >= 1; j--)
                temp += '0';
            ans = my_add(ans, temp);
        }
        return ans;
    }
    string my_add(string num1, string num2){
        if (num1 == "")
            return num2;
        reverse(num1.begin(), num1.end());
        reverse(num2.begin(), num2.end());
        for (int i = num2.size()-num1.size(); i>0; --i)
            num1+='0';
        string res = "";
        char cf = '0';
        for (int i = 0; i < num2.size(); ++i){
            res += ((num1[i]-'0')+(num2[i]-'0') + (cf-'0'))%10+'0';
            cf = ((num1[i]-'0')+(num2[i]-'0') + (cf-'0'))/10+'0';
        }
        if (cf != '0')
            res += cf;
        reverse(res.begin(), res.end());
        return res;
    }

private:
    string ans = "";
};
```
比较优秀的方法
```
class Solution {
public:
    string multiply(string num1, string num2) {
        if (num1.empty() || num2.empty())
            return "";

        std::reverse(num1.begin(), num1.end());
        std::reverse(num2.begin(), num2.end());
        std::string res(num1.size() + num2.size(),'0');
        for (int j = 0; j < num2.size(); j++)
        {
            int tmp = 0;
            int val = num2[j] - '0';
            for (int i = 0; i < num1.size(); i++)
            {
                tmp += (num1[i] - '0') * val + (res[i + j] - '0');
                res[i + j] = tmp % 10 + '0';
                tmp /= 10;
            }
            if (tmp != 0)
                res[num1.size() + j] = tmp + '0';
        }
        std::reverse(res.begin(), res.end());
        int count = 0;
        while (count < res.size() - 1 && res[count] == '0')
            count++;
        res.erase(0, count);
        return res;
    }
};
```
相比之下我自己的方法太按部就班了，这里要找到乘法的规律：两个数相乘，得到的结果位数不会超过输入的位数之和；然后做乘法的时候，**<font color=red size=3 face="微软雅黑">
i+j就是每次运算对应结果的位置</font>**，其上存放着每次运算后该位置的结果值，temp是每次运算的时候的进位值。
