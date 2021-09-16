
# 1 为什么需要反射
反射就是程序能够在运行时检查变量和值，**求出它们的类型**。（注：C++用的是模板编程）比如下面的例子，我们要根据输入的结构体参数字段的值，生成SQL语句。显而易见，对于定义的两个完全不同的结构体，我们需要在函数中判断传入的时什么类型，不然生成的SQL语句就是错的。这就需要在运行时判断输入的参数时什么类型
```go
type order struct {
    ordId      int
    customerId int
}
type employee struct {
    name string
    id int
    address string
    salary int
    country string
}
//这里要判断q的具体类型，不然获取字段值就会出错
func createQuery(q interface{}) string {
    i := fmt.Sprintf("insert into order values(%d, %d)", q.ordId, q.customerId)
    return i
}
func main() {
    o := order{
        ordId:      1234,
        customerId: 567,
    }
    fmt.Println(createQuery(o))
}
```

# 2 reflect包
在Go语言中，`reflect`实现了运行时反射。`reflect`包会帮助识别 `interface{}`变量的底层具体类型和具体值。常用方法如下：

- `reflect.Type`表示 `interface{}` 的具体类型，`reflect.TypeOf()`函数可以返回 `reflect.Type` 
- `reflect.Value` 表示它的具体值。`reflect.ValueOf()`函数返回 `reflect.Value`
- `Kind()`表示`interface{}`的基本类型
- `NumField()` 方法返回结构体中字段的数量，而 `Field(i int)` 方法返回字段 `i` 的 `reflect.Value`
- `Int()` 和 `String()` 可以帮助我们分别取出 `reflect.Value` 作为 `int64` 和 `string`

示例如下：
```go
package main
type order struct {
    ordId      int
    customerId int
}

func createQuery(q interface{}) {
    t := reflect.TypeOf(q)
    k := t.Kind()
    fmt.Println("Type ", t) //输出main.order
    fmt.Println("Kind ", k) //输出struct
}
```
有了反射机制，我们可以实现第一小节SQL的例子了：
```go
func createQuery(q interface{}) string{
    if reflect.ValueOf(q).Kind() == reflect.Struct {//判断是不是结构体
        t := reflect.TypeOf(q).Name()//获取类型名称
        query := fmt.Sprintf("insert into %s values(", t)
        v := reflect.ValueOf(q)//获取值
        for i := 0; i < v.NumField(); i++ {//获取字段个数
            switch v.Field(i).Kind() {
            case reflect.Int:
                if i == 0 {
                    query = fmt.Sprintf("%s%d", query, v.Field(i).Int())//取出当前字段的值，加入到query中
                } else {
                    query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
                }
            case reflect.String:
                if i == 0 {
                    query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
                } else {
                    query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
                }
            default:
                fmt.Println("Unsupported type")
                return
            }
        }
        query = fmt.Sprintf("%s)", query)
        fmt.Println(query)
        return query
    }
    fmt.Println("unsupported type")
}
```
**必须注意：**反射是Go语言中非常强大和高级的概念，我们应该小心谨慎地使用它。**使用反射编写清晰和可维护的代码是十分困难的。你应该尽可能避免使用它**，只在必须用到它时，才使用反射。
