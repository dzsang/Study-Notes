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