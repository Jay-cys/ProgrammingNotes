- 包（Packages）： Cargo 的一个功能，它允许你构建、测试和分享 crate。
- Crates ：一个模块的树形结构，它形成了库或二进制文件。
- 模块（Modules）和 use： 允许你控制作用域和路径的私有性。
- 路径（path）：一个命名例如结构体、函数或模块等项的方式
# Package
使用`cargo new`可以创建两种crate：

- binary二进制文件：`cargo new name`，此方式会在src下创建**main.rs**文件
   - 如果将文件放在 s**rc/bin** 目录下，一个包可以拥有多个二进制 crate：每个 src/bin 下的文件都会被编译成一个独立的二进制crate  
- lib库文件：`cargo new --lib name`，此方式会在src下创建**lib.rs**文件
   - 在src下的rs文件，默认会生成**与文件名同名的模块**，可以在其他文件中使用`mod name;`导入使用（C++的前置声明）。
   - 多级嵌套模块在src下对应多级目录。

如果一个包同时含有 src/main.rs 和 src/lib.rs，则它有两个 crate：一个库和一个二进制项，且名字都与包同。


Cargo的package目录结构如下：
```cpp
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

- Cargo.toml与Cargo.lock存储在项目的根目录中。
- 源代码进入src目录。
- 默认库文件是src/lib.rs。
- 默认的可执行文件是src/main.rs。
- 其他可执行文件可以放入src/bin/*.rs。
- 集成测试进入tests目录（单元测试进入他们正在测试的每个文件中）。
- 示例可执行文件放在examples目录中。
- 基准测试进入benches目录
# 路径引用模块

- 绝对路径（absolute path）从crate根开始，以`crate名`或者`字面值crate`开头。
- 相对路径（relative path）从当前模块开始，以 self(本模块开始)、 super(父模块开始)或当前模块的标识符开头
- 绝对路径和相对路径都后跟一个或多个由双冒号（ :: ）分割的标识符
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
//绝对路径
crate::front_of_house::hosting::add_to_waitlist();
//相对路径，本函数与上面的模块在同一层级
front_of_house::hosting::add_to_waitlist();
self::front_of_house::hosting::add_to_waitlist();
```


# use导入模块
直接使用路径太繁琐了，我们可以使用`use`导入需要的模块，简化调用，类似C++的`using`。
```rust
//导入了hosting模块，可以直接调用，不需要那么长的路径了
use crate::front_of_house::hosting;
hosting::add_to_waitlist();
```


其他技巧

- 和C++的`using`一样，`use`也可以定义别名，格式为：`use XXX as xxx;`。
- `use`还可以把多个模块在同一行中导入，减少行数：`use std::{mod1,mod2};`
