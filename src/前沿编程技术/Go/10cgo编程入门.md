要使用CGO特性，需要安装C/C++构建工具链，在macOS和Linux下是要安装GCC，在windows下是需要安装MinGW工具。同时需要保证环境变量`CGO_ENABLED`被设置为1，这表示CGO是被启用的状态。

# 1 cgo启用语句

## 1.1 import "C"
**通过`import "C"`语句启用CGO特性**，**紧跟在这行语句前面的注释是一种特殊语法**，里面包含的是正常的C语言代码。当确保CGO启用的情况下，还可以在当前目录中包含C/C++对应的头文件。

示例如下：
```go
//第一个cgo的例子，使用C/C++的函数
package main

//
// 引用的C头文件需要在注释中声明，紧接着注释需要有import "C"，且这一行和注释之间不能有空格
//

/*
#include <myprint.h> //自定义头文件
#include <stdlib.h>
#include <unistd.h>
void myprint(char* s);//声明头文件中的函数
*/
import "C"

import (
	"fmt"
	"unsafe"
)

func main() {
	//使用C.CString创建的字符串需要手动释放。
	cs := C.CString("Hello World\n")
	C.myprint(cs)
	C.free(unsafe.Pointer(cs))
	fmt.Println("call C.sleep for 3s")
	C.sleep(3)
	return
}
```

## 1.2 #cgo
在`import "C"`语句前的注释中可以通过`#cgo`语句设置编译阶段和链接阶段的相关参数。编译阶段的参数主要用于定义相关宏和指定头文件检索路径。链接阶段的参数主要是指定库文件检索路径和要链接的库文件。`#cgo`语句主要影响**CFLAGS、CPPFLAGS、CXXFLAGS、FFLAGS和LDFLAGS**几个编译器环境变量。

- **CFLAGS**：对应C语言编译参数(以`.c`后缀名)
- **CPPFLAGS**：对应C/C++ 代码编译参数(_.c,_.cc,_.cpp,_.cxx)
- **CXXFLAGS**：对应纯C++编译参数(_.cc,_.cpp,*.cxx)
- **LDFLAGS：**对应静态库和动态库链接选项，必须使用绝对路径（cgo 中的 ${SRCDIR} 为当前目录的绝对路径）


使用示例如下：
```go
//使用C库,编译时GCC会自动找到libnumber.a或libnumber.so进行链接
package main

/*#cgo CFLAGS: -I./c_library
#cgo LDFLAGS: -L${SRCDIR}/c_library -l number
#include "number.h"
*/
import "C"
import "fmt"

func main() {
	fmt.Println(C.number_add_mod(10, 5, 12))
}
```

`#cgo`指令还支持**条件选择**，当满足某个操作系统或某个CPU架构类型时后面的编译或链接选项生效。比如下面是分别针对windows和非windows下平台的编译和链接选项：
```
// #cgo windows CFLAGS: -DX86=1
// #cgo !windows LDFLAGS: -lm
```

# 2 C与Go之间类型映射

## 2.1 基本类型转换
Go语言中数值类型和C语言数据类型基本上是相似的，以下是它们的对应关系:

| **C语言类型** | **CGO类型** | **Go语言类型** |
| --- | --- | --- |
| char | C.char | byte |
| singed char | C.schar | int8 |
| unsigned char | C.uchar | uint8 |
| short | C.short | int16 |
| unsigned short | C.ushort | uint16 |
| int | C.int | int32 |
| unsigned int | C.uint | uint32 |
| long | C.long | int32 |
| unsigned long | C.ulong | uint32 |
| long long int | C.longlong | int64 |
| unsigned long long int | C.ulonglong | uint64 |
| float | C.float | float32 |
| double | C.double | float64 |
| size_t | C.size_t | uint |


## 2.2 结构体、联合、枚举类型
C语言的结构体、联合、枚举类型不能作为匿名成员被嵌入到Go语言的结构体中。在Go语言中，我们可以通过`C.struct_xxx`来访问C语言中定义的`struct xxx`结构体类型。
```go
/*
struct A {
    int   type;  // type 是 Go 语言的关键字，此项被屏蔽
    float _type; // 将屏蔽CGO对 type 成员的访问
};
*/
import "C"
import "fmt"

func main() {
    var a C.struct_A
    fmt.Println(a._type) // _type 对应 _type
}
```
对于联合类型，我们可以通过`C.union_xxx`来访问C语言中定义的`union xxx`类型。但是Go语言中并不支持C语言联合类型，它们会被转为对应大小的**字节数组**。对于枚举类型，我们可以通过`C.enum_xxx`来访问C语言中定义的`enum xxx`结构体类型。
```go
/*
enum C {
    ONE,
    TWO,
};
*/
import "C"
import "fmt"

func main() {
    var c C.enum_C = C.TWO
    fmt.Println(c)
    fmt.Println(C.ONE)
    fmt.Println(C.TWO)
}
```

## 2.3 字符串和数组转换
CGO的C虚拟包提供了以下一组函数，用于Go语言和C语言之间数组和字符串的双向转换：
```go
// Go string to C string, C.free is needed).
func C.CString(string) *C.char

// Go []byte slice to C array, C.free is needed).
func C.CBytes([]byte) unsafe.Pointer

// C string to Go string
func C.GoString(*C.char) string

// C data with explicit length to Go string
func C.GoStringN(*C.char, C.int) string

// C data with explicit length to Go []byte
func C.GoBytes(unsafe.Pointer, C.int) []byte
```

# 3 C函数如何返回errno？
CGO也针对`<errno.h>`标准库的`errno`宏做的特殊支持：在CGO调用C函数时如果有两个返回值，那么第二个返回值将对应`errno`错误状态。对于void类型函数，这个特性依然有效。
```go
/*
#include <errno.h>
static int div(int a, int b) {
    if(b == 0) {
        errno = EINVAL;
        return 0;
    }
    return a/b;
}
*/
import "C"
import "fmt"
func main() {
    v0, err0 := C.div(2, 1)
    fmt.Println(v0, err0)
    v1, err1 := C.div(1, 0)
    fmt.Println(v1, err1)
}
```

# 4 一个完整的封装C函数的例子
该例子的重点是，**在封装C函数的模块里要提供外部类型和函数指针类型给其他go模块使用**，不能直接在其他模块使用封装模块中的C类型，因为不同模块cgo编译后C类型并不是统一类型，无法进行类型转换。
封装C标准库qsort函数：
```go
//封装C标准库函数qsort，给其他go文件或模块使用
package qsort

/*
#include <stdlib.h>
//qsort的比较函数指针
typedef int (*qsort_cmp_func_t)(const void* a, const void* b);
*/
import "C"
import "unsafe"

//将虚拟C包中的类型通过Go语言类型代替，在内部调用C函数时重新转型为C函数需要的类型
//因此外部用户将不再依赖qsort包内的虚拟C包，消除用户对CGO代码的直接依赖
type CompareFunc C.qsort_cmp_func_t

//封装qsort的go Sort函数
func Sort(base unsafe.Pointer, num int, size int, cmp CompareFunc) {
	C.qsort(base, C.size_t(num), C.size_t(size), C.qsort_cmp_func_t(cmp))
}
```
使用上面qsort库的其他库文件：
```go
package main

//extern int go_qsort_compare(void* a, void* b);
import "C"

import (
	"fmt"
	"qsort"
	"unsafe"
)

//export go_qsort_compare
func go_qsort_compare(a, b unsafe.Pointer) C.int {
	pa, pb := (*C.int)(a), (*C.int)(b)
	return C.int(*pa - *pb)
}

func main() {
	values := []int32{42, 9, 101, 95, 27, 25}

	qsort.Sort(unsafe.Pointer(&values[0]),
		len(values), int(unsafe.Sizeof(values[0])),
        //转换一下函数指针，使用qsort提供的类型，不直接使用C空间函数指针
		qsort.CompareFunc(C.go_qsort_compare),
	)
	fmt.Println(values)
}
```
