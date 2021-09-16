
# 1.简洁高效的方法（不过只能包含一个分隔符）：
```cpp
#include <vector>
#include <string>
#include <iostream>
using namespace std;
 
void SplitString(const string& s, vector<string>& v, const string& c)
{
    string::size_type pos1, pos2;
    pos2 = s.find(c);
    pos1 = 0;
    while(string::npos != pos2)
    {
        v.push_back(s.substr(pos1, pos2-pos1));
         
        pos1 = pos2 + c.size();
        pos2 = s.find(c, pos1);
    }
    if(pos1 != s.length())
        v.push_back(s.substr(pos1));
}
 
int main(){
    string s = "a,b,c,d,e,f";
    vector<string> v;
    SplitString(s, v,","); //可按多个字符来分隔;
    for(vector<string>::size_type i = 0; i != v.size(); ++i)
        cout << v[i] << " ";
    cout << endl;
    //输出: a b c d e f
}
```

当处理有空格的字符串时，还是很有用的！！使用void SplitString(const string& s, vector& v, const string& c)，和将v作为返回值都是可以的！


# 2.可包含多个分隔符的实现方式


```cpp
#include <vector>
#include <string>
#include <iostream>
using namespace std;
 
vector<string> split(const string &s, const string &seperator){
    vector<string> result;
    typedef string::size_type string_size;
    string_size i = 0;
     
    while(i != s.size()){
        //找到字符串中首个不等于分隔符的字母；
        int flag = 0;
        while(i != s.size() && flag == 0){
            flag = 1;
            for(string_size x = 0; x < seperator.size(); ++x)
                if(s[i] == seperator[x]){
                    ++i;
                    flag = 0;
                     break;
                    }
        }
         
        //找到又一个分隔符，将两个分隔符之间的字符串取出；
        flag = 0;
        string_size j = i;
        while(j != s.size() && flag == 0){
            for(string_size x = 0; x < seperator.size(); ++x)
                if(s[j] == seperator[x]){
                    flag = 1;
                     break;
                    }
            if(flag == 0)
                ++j;
        }
        if(i != j){
            result.push_back(s.substr(i, j-i));
            i = j;
        }
    }
    return result;
}
 
int main(){
//  string s = "a,b*c*d,e";
    string s;
    getline(cin,s);
    vector<string> v = split(s, ",*"); //可按多个字符来分隔;
    for(vector<string>::size_type i = 0; i != v.size(); ++i)
        cout << v[i] << " ";
    cout << endl;
    //输出: a b c d e
}
```



# 3. 用C语言中的strtok 函数来进行分割

原型:  char _strtok(char _str, const char *delim);strtok函数包含在头文件<string.h>中，对于字符数组可以采用这种方法处理。

```cpp
#include <string.h>
#include <stdio.h>
 
int main(){
    char s[] = "a,b*c,d";
    const char *sep = ",*"; //可按多个字符来分割
    char *p;
    p = strtok(s, sep);
    while(p){
        printf("%s ", p);
        p = strtok(NULL, sep);
    }
    printf("\n");
    return 0;
}
//输出: a b c d
```

From [https://www.cnblogs.com/carsonzhu/p/5859552.html](https://www.cnblogs.com/carsonzhu/p/5859552.html)


# 4 最优雅的实现
```cpp
void split(const std::string& s, std::vector<std::string>& tokens, const std::string& delimiters = " ")
{
    std::string::size_type lastPos = s.find_first_not_of(delimiters, 0);
    std::string::size_type pos = s.find_first_of(delimiters, lastPos);
    while (std::string::npos != pos || std::string::npos != lastPos)
    {
        tokens.push_back(s.substr(lastPos, pos - lastPos));
        lastPos = s.find_first_not_of(delimiters, pos);
        pos = s.find_first_of(delimiters, lastPos);
    }
}
```
