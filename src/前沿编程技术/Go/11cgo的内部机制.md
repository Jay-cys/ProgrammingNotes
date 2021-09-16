
# 1 中间生成文件
在一个Go源文件中，如果出现了`import "C"`指令则表示将调用cgo命令生成对应的中间文件。下图是cgo生成的中间文件的简单示意图：

![](.assets/1609255146861-8c5676ef-0ea9-4fe6-9188-2e43bb4148b3.png)

# 2 Cgo内存访问

如果在CGO处理的跨语言函数调用时涉及到了指针的传递，则可能会出现**Go语言和C语言共享某一段内存**的场景。我们知道C语言的内存在分配之后就是稳定的，但是Go语言因为函数栈的动态伸缩可能导致栈中内存地址的移动(这是Go和C内存模型的最大差异)。如果C语言持有的是移动之前的Go指针，那么以旧指针访问Go对象时会导致程序崩溃。

## 2.1 Go访问C内存
C语言空间的内存是稳定的，只要不是被人为提前释放，那么在Go语言空间可以放心大胆地使用。
比如下面示例，我们可以在Go中调用C的malloc和free创建、使用和释放内存，不用考虑内存地址移动的问题。
```go
package main

/*
#include <stdlib.h>

void* makeslice(size_t memsize) {
    return malloc(memsize);
}
*/
import "C"
import "unsafe"

func makeByteSlize(n int) []byte {
    p := C.makeslice(C.size_t(n))
    return ((*[1 << 31]byte)(p))[0:n:n]
}

func freeByteSlice(p []byte) {
    C.free(unsafe.Pointer(&p[0]))
}

func main() {
    s := makeByteSlize(1<<32+1) //创建一个超大的内存用于切片
    s[len(s)-1] = 255
    print(s[len(s)-1])
    freeByteSlice(s)
}
```

## 2.2 Go内存传入C语言函数
C/C++很多库都是需要通过指针直接处理传入的内存数据的，因此cgo中也有很多需要将Go内存传入C语言函数的应用场景。**Go的内存是不稳定的，goroutinue栈因为空间不足的原因可能会发生扩展，导致了原来的Go语言内存被移动到了新的位置，如果这时候还按照原来地址访问内存，就会导致内存越界**。为了简化并高效处理向C语言传入Go语言内存的问题，cgo针对该场景定义了专门的规则：**在CGO调用的C语言函数返回前，cgo保证传入的Go语言内存在此期间不会发生移动，C语言函数可以大胆地使用Go语言的内存**！

```go
package main

/*
#include<stdio.h>

void printString(const char* s, int n) {
    int i;
    for(i = 0; i < n; i++) {
        putchar(s[i]);
    }
    putchar('\n');
}
*/
import "C"
import "unsafe"
import "reflect"

func printString(s string) {
    p := (*reflect.StringHeader)(unsafe.Pointer(&s))
    //直接传入go的内存空间给C函数
    C.printString((*C.char)(unsafe.Pointer(p.Data)), C.int(len(s)))
}

func main() {
    s := "hello"
    printString(s)
}
```

