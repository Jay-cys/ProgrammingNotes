
# enum定义

## int型enum
enum枚举定义和C++基本一样，出了最后的分号。使用方式类似C++ `enum class`：
```rust
fn main() {
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind, //作为struct的成员
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };
}
```

## 非int型enum
和C++`enum class`一样，enum默认是int类型，当然也可以自定义其他类型。Rust enum比C++强大的是：

- rust enum可以单独为每个enum值定义类型，**不同enum值类型可以不一样**（C++不行）
- rust enum可以为同一个enum值**定义多个类型**，使用`()`
- rust enum可以为enum值定义struct类型，使用`{}`
```rust
//string类型的enum值
enum IpAddr {
	V4(String),
	V6(String),
}
let home = IpAddr::V4(String::from("127.0.0.1"));
//多类型的enum值
enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

let home = IpAddr::V4(127, 0, 0, 1);
//struct的enum值
enum Message {
    Quit,
    Move { x: i32, y: i32 },
}
```

# match语句
**switch 语法很经典，但在 Rust 中并不支持**，很多语言摒弃 switch 的原因都是因为 switch 容易存在因忘记添加 break 而产生的串接运行问题。Rust 通过 match 语句来实现分支结构：
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
enum可以定义非int类型，那么match语句也需要支持其他类型的enum值，比如：
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {//匹配到此类型的enum，state这里表示参数名
            println!("State quarter from {:?}!", state);//打印enum的具体值
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));//指定enum值
}
```

# if let语句
如果只判断某一个enum值，用match就有点多余。在C++我们可以直接使用if语句判断，而在rust中这是不允许的，会报错`an implementation of `std::cmp::PartialEq` might be missing for "XXX"`。Rust中可以使用`if let`判断一个enum值：
```rust
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn main() {
    let coin = Coin::Penny;
    let mut count = 0;
    if let Coin::Quarter(state) = coin { //类似C++的if判断
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
}
```
