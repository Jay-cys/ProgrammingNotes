C++11开始支持**属性attribute**机制，可以使用属性进行一些格外描述，让编译器进行不同的操作。如下是当前支持的标准属性，这些属性都可以通过宏`**__has_cpp_attribute(属性记号)**`判断编译器是否支持：

| **属性记号** | **属性使用格式** | **标准** | **作用** |
| --- | --- | --- | --- |
| carries_dependency | [[carries_dependency]] | (C++11) | 允许编译器跳过不必要的内存栅栏指令 |
| deprecated | [[deprecated]] | (C++14) | 表示该元素被弃用，编译器会给一个告警 |
| fallthrough | [[fallthrough]] | (C++17) | 用以switch case，表示故意在case结尾不加break |
| maybe_unused | [[maybe_unused]] | (C++17) | 关闭编译器提示参数或变量未使用的告警 |
| no_unique_address | [[no_unique_address]] | (C++20) | 表示struct中某成员拥有空类型特征 |
| nodiscard | [[nodiscard]] | (C++17) | 用于函数返回值前，该函数返回值必须被使用或处理，否则提示warning |
|  | [[nodiscard("string")]] | (C++20) | C++20增强，可以自定义warning内容 |
| noreturn | [[noreturn]] | (C++11) | 用于函数前面，表示该函数不会再返回调用端。比如在函数内直接结束进程 |
| unlikely | [[unlikely]] | (C++20) | 用于if判断后或switch case前面，表示某个分支的可能性大小，用于帮助编译器优化代码 |
| likely | [[likely]] | (C++20) |  |

​

特别说一下no_unique_address，比较复杂，有以下特性：①同类型的子对象或成员不占用同一个地址；②当地址不够分配时，则按照一般做法扩展空间，继续为未分配地址的no_unique_address属性成员分配地址，直至全部分配完毕；③该属性对空类型（没有非静态数据成员）有效。​

使用示例：
```cpp
//C++ attributes使用示例
#include <iostream>
using namespace std;

//使用nodiscard属性，返回值必须被处理使用
[[nodiscard("barret")]] int func1()
{
    return 1;
}

//使用maybe_unused属性，关闭未使用的告警
int func2(int a, [[maybe_unused]] int b)
{
    return 2;
}

//noreturn属性，表示函数调用栈不会返回
[[noreturn]] void forceExitProgram()
{
    std::exit(1); //结束进程
}
bool isIdLicensed(int id)
{
    if (true)
        forceExitProgram();
    else
        return true;
}

//deprecated属性，表示该C++元素被弃用
[[deprecated]] int func3()
{
    return 3;
}

//no_uniquire_address属性
struct A
{ };  // 空类型
 
struct B
{
    long long v;
    [[no_unique_address]] A a, b;
};

int main()
{
    // func1(); //编译告警
    // forceExitProgram();
    // func2(1, 2);
    // // bool isLicensed{isIdLicensed(42)};
    // cout << func3() << endl;

    // int input = 0;
    // cout << "input a value:";
    // cin >> input;
    // switch (input)
    // {
    //     [[unlikely]] case 0 : //表示该分支发生可能性很小
    //         cout << "i am 0"<<endl;
    //         break;
    //     [[likely]] case 1: //表示该分支很可能发生
    //         cout << "i am 1"<<endl;
    //         [[fallthrough]] //故意不加break；
    //     default:
    //         cout << "i am not 0"<<endl;
    //         break;
    // }

    //使用struct B
    B b;
    std::cout << &b.v << std::endl; // 输出v地址
    std::cout << &b.a << std::endl; // a地址为 &v + 1
    std::cout << &b.b << std::endl; // b地址为 &v + 2
    std::cout << sizeof(B) << std::endl; // 输出 16
}
```
