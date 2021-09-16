
# 1 std::function
C++11 **std::function** 是一种通用、多态的函数封装，它的实例可以对任何可以调用的目标实体进行存储、复制和调用操作，它也是对 C 中现有的可调用实体的一种类型安全的包裹（相对来说，函数指针的调⽤不是类型安全的），换句话说，就是函数的容器。当我们有了函数的容器之后便能够更加方便的将函数、函数指针作为对象进行处理。
example_9 code
```cpp
#include <iostream>
#include <functional>  //必须头文件

int foo(int a)
{
    return a;
}

int main()
{
    std::function<int(int)> func = foo; //包装了一个返回值为int，参数为int的函数

    int important = 10;
    std::function<int(int)> func2 = [&](int value)->int{
        return 1 + value + important;
    };

    std::cout<<func(10)<<std::endl;
    std::cout<<func2(10)<<std::endl;

}
```



# 2 std::array
std::array 保存在栈内存中，相比堆内存中的 std::vector，我们就能够灵活的访问这里面的元素，从而获得更好的性能。使用std::array能够让代码变得更加现代，且封装了一些操作函数，同时还能够友好的使用标准库中的容器算法等等，比如 std::sort。
语法格式：
```cpp
    std::array<int, 4> arr{1, 2, 3, 4}; //固定大小，不能隐式转为指针

    // C风格接口传参
    // foo (arr , arr . size ()); // 非 法 , 无 法 隐 式 转 换
    foo(&arr[0], arr.size());    //显示转换为指针
    foo(arr.data(), arr.size()); //显示转换为指针

    // 使用std::sort 
    std::sort(arr.begin(), arr.end());
```

# 3 std::tuple
关于元组的使用有三个核心的函数：

1. std::make_tuple: 构造元组
1. std::get: 获得元组某个位置的值
1. std::tie: 元组拆包


**例子**example_11 code:
```cpp
#include <iostream>
#include <tuple>
#include <string>

int main()
{
	auto t = std::make_tuple(3.8, "A", "Barret Ren");
	//get获取元素
	std::cout<<std::get<0>(t)<<std::endl;
	std::cout<<std::get<1>(t)<<std::endl;
	std::cout<<std::get<2>(t)<<std::endl;

	//tie进行拆包
	double a;
	std::string b;
	std::string c;
    
	std::tie(a, b, c)=t;
	std::cout<<c<<std::endl;
}
```

std::tuple 虽然有效，但是标准库提供的功能有限，没办法满足运行期索引和迭代的需求，好在我们还有其他的方法可以实现。需要boost库的支持。


# 4 std::size()
C++17开始，可以使用std::size()获取传统C数组的个数。`int myArray[]={1,2,3,4};``std::cout<<std::size(myArray)<<std::endl;`

# 5 std::optional
optional意为可选值，需要传入一个类型，表示该类型的值存在或不存在。通常用于函数返回值，比如下面的用法：
```cpp
//std::optional使用示例，表示可以存在或不存在的值
#include <iostream>
#include <optional>

using namespace std;

//定义一个函数，返回值是可选值，既可以是int，也可以是其他非类型
optional<int> getData(bool it)
{
    if (it)
        return 42;
    return nullopt;
}

int main()
{
    //调用函数，保存返回值
    optional<int> data1{getData(true)};
    optional<int> data2{getData(false)};
    //判断变量是否有值(两种方式)，如果有打印值
    if (data1.has_value())
    {
        cout << data1.value() << endl;
    }
    if (data2)
    {//不经判断直接访问会抛出exception
        cout << *data2 << endl;
    }
    //data2没有值，但可以通过以下方式打印它保存了什么
    cout << "what is in data? " << data2.value_or(0) << endl;
}
```

# 6 emplace操作
C++11开始支持emplace操作，emplace和push_back/insert之类的修改方式不同的是，emplace**直接占用容器的内存空间创建需要保存的对象（原位调用构造函数），不需要提前创建对象再复制或移动（move）到容器中**。emplace主要有以下基本成员函数，不同类型的容器支持情况不同：

- `emplace(const_iterator pos, Args&&... args)`：直接于 pos 前插入元素到容器中
- `emplace_after(const_iterator pos, Args&&... args)`：在容器中的指定位置后插入新元素
- `emplace_hint(const_iterator hint, Args&&... args)`：插入新元素到尽可能接近恰在 hint 前的位置
- `emplace_front(Args&&... args)`：插入新元素到容器起始
   - C++11返回void
   - C++17开始返回构造对象的引用
- `emplace_back(Args&&... args)`：添加新元素到容器尾
   - C++11返回void
   - C++17开始返回构造对象的引用

# 7 std::span
span是C++20引入，表示连续相邻的序列，可以用于表示vector，array和C原始数组**的一部分或全部，类似于**[**string_view**](https://www.yuque.com/barret/snelnn/ht2gln#tWLD5)。span主要包含两个成员：**指向容器类型T的指针和表示的范围大小**。span只是顺序序列容器的一个**指针视图**，它并不会复制原始数据（**可以通过span修改数据，因为内部保存容器数据指针）**，所以作为参数传递时更加快捷。​

两个比较重要的成员函数：

- `data()`：返回原始容器或数组的指针，可以用这个指针来修改数据
- `subspan(std::size_t Offset,

 std::size_t Count)`：从span基础上再创建新的子视图

示例如下：
```cpp
void print(span<int> values)
{
    for (const auto &value : values)
    {
        cout << value << " ";
    }
    cout << endl;
}

int main()
{
    vector v{11, 22, 33, 44, 55, 66};
    //直接传vector，因隐式转换为全范围的span
    print(v);

    //创建全范围span作为参数
    span mySpan{v};
    print(mySpan);

    //从全范围span创建子span
    span subspan{mySpan.subspan(2, 3)};
    print(subspan);

    //从vector创建span，因为第一个参数是指针，所以需要调用data()
    print({v.data() + 2, 3});

    //从array创建span
    array<int, 5> arr{5, 4, 3, 2, 1};
    print(arr);
    print({arr.data() + 2, 3});

    //从C数组创建span
    int carr[]{9, 8, 7, 6, 5};
    print(carr);          // The entire C-style array.
    print({carr + 2, 3}); // A subview of the C-style array.
}
```
 
