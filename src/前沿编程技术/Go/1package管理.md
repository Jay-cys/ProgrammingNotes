
# 1 main包和main函数
所有可执行的 Go 程序都必须包含一个 main 函数。这个函数是程序运行的入口。main 函数应该放置于 main 包中。键入 `go install dirname`，编译程序。该命令会在文件夹内搜索拥有 main 函数的文件。接下来，它编译并产生一个名为 `dirname` （在 windows 下是 dirname`.exe`）的二进制文件，该二进制文件放置于工作区的 bin 文件夹。

# 2 自定义包
**属于某一个包的源文件都应该放置于一个单独命名的文件夹里。按照 Go 的惯例，应该****用包名命名该文件夹**。

## 2.1 导入包
为了使用自定义包，我们必须要先导入它。导入自定义包的语法为 `import path`。我们必须指定自定义包相对于工作区内 `src` 文件夹的相对路径。我们目前的文件夹结构是：
```bash
src
    geometry
        geometry.go
        rectangle
            rectprops.go
```
`import "geometry/rectangle"` 这一行会导入 rectangle 包（指定目录名）。
有时候我们导入一个包，只是为了确保它进行了初始化，而无需使用包中的任何函数或变量。这种情况也可以使用空白标识符_：
```go
package main 
import (
    _ "geometry/rectangle" //仅导入，不使用
)
func main() {

}
```

## 2.2 导出包内符号
在 Go 中，**任何以大写字母开头的变量或者函数都是被导出的名字，**其它包只能访问被导出的函数和变量。

## 2.3 init函数
所有包都可以包含一个 `init` 函数。init 函数不应该有任何返回值类型和参数，在我们的代码中也不能显式地调用它。init 函数的形式如下：
```go
func init() {  
}
```
init 函数可用于执行初始化任务，也可用于在开始执行之前验证程序的正确性。包的初始化顺序如下：

1. 首先初始化包级别（Package Level）的变量
1. 紧接着调用 init 函数。包可以有多个 init 函数（在一个文件或分布于多个文件中），它们按照编译器解析它们的顺序进行调用。

# 3 go module管理

## 3.1 导入本地包
上面第二节是传统的自定义包和使用的过程，如果开启了go module功能，上面的步骤就无法编译通过了。可以通过`go env`查看配置。如果开启了go module，需要采用下面新的方式导入自定义本地包：

- 在被调用文件的目录下执行`go mod init`，使本目录变为一个module，会生成一个go.mod文件
- 同理，在调用者的目录下也执行`go mod init`
- 在调用者目录，修改go.mod的属性，指定需要调用的本地包的相对路径（很重要，不然找不到包）
- 之后就可以在调用者的源文件中import相应的package name


具体示例如下：有以下目录结构，被调用者在子目录，调用者在另一个目录（位置没要求，只要在GOPATH中即可，不一定要在父目录）：
```bash
> ls -R
.:
9_use_mymod.go  mymod

./mymod:
mymod.go
```
下面我们需要将两个目录都变为module，在两个目录分别执行`go mod init`，生成go.mod文件。下面是两个源文件的内容：
```go
//9_use_mymod.go 使用自定义的package，并调用函数
package main

import "mymod" //调用自定义的包

func main() {
	mymod.SayHi()
}

//mymod.go 自定义的package，被其他go文件使用
package mymod

import "fmt"

func SayHi() {
	fmt.Println("Hi! from package mymod")
}
```
如果这时候直接编译，会提示找不到mymod。我们需要在调用者的go.mod文件中设置mymod的具体问题，通过以下命令修改go.mod的配置：
```bash
# go mod: -replace=./mymod: need old[@v]=new[@w] (missing =)
go mod edit -replace mymod=./mymod
# 这里设置了mymod的具体位置，使用的相对路径，也可以设置绝对路径
```
之后就可以正常编译了

# 4 其他命令

- `go get url`：下载go package，放在src目录下
- `go doc package_name [函数名}`：查看包的帮助信息，或具体的函数信息。我们自己添加的注释和函数定义，也可以通过go doc生成，和标准库中信息差不多
- `go fmt file`：格式化go源文件




