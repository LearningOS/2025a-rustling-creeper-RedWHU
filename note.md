### How To StartUp?
``` 
ob@ob-VirtualBox:~/miniob/rustos/2025a-rustling-creeper-RedWHU$ rustlings watch

```
### exercise/无法跳转
```
rustlings lsp

2. 利用 rust-project.json (适用于 rust-analyzer)
高级用法：如果 exercises/ 目录结构复杂或是作为作业/习题项目，你可以用 rust-analyzer 的 rust-project.json
（你的 main.rs 里有 LspArgs，说明支持自动生成 rust-project.json）

运行 cargo run -- lsp 或 rustlings lsp，它会在项目根目录生成 rust-project.json
确认 VSCode 已启用 rust-analyzer，并且项目根目录有 rust-project.json
重启 VSCode，rust-analyzer 会自动分析 exercises/ 下的代码
→ 此时 Ctrl+点击跳转应该有效
```
知识点网站：https://doc.rust-lang.org/book/ch06-02-match.html

# 1. 枚举与模式匹配（Enums and Pattern Matching）

## 1.1 枚举类型

Rust 中的枚举（`enum`）允许你定义一个类型，该类型的值可以是一组限定值之一。每个枚举成员（variant）可以带有附加数据。例如：

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}
```

可以为某些枚举成员附加数据：

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

## 1.2 模式匹配（match）

Rust 的 `match` 控制流结构非常强大，可以将一个值与多个模式进行比较，并根据匹配结果执行不同的代码。它类似于其他语言的 switch，但更安全、更灵活。

### 1.2.1 基本用法

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

- `match` 后跟要匹配的表达式（如 `coin`）。
- 每个分支由一个模式和对应的代码块组成，使用 `=>` 分隔。

### 1.2.2 分支代码块

- 分支代码简短时可省略 `{}`。
- 多行代码需用 `{}` 包裹，最后一个表达式为该分支的返回值。

```rust
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

## 1.3 模式绑定数据

模式可以绑定到枚举成员中的数据，实现数据提取：

```rust
enum UsState {
    Alabama,
    Alaska,
    // ...
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
        Coin::Quarter(state) => {
            println!("State quarter from {state:?}!");
            25
        }
    }
}
```
- 当匹配到 `Coin::Quarter(state)` 时，`state` 变量会绑定到该枚举成员内部的数据。

## 1.4 匹配标准库枚举 Option<T>

Rust 标准库的 `Option<T>` 枚举用于表示可能存在或不存在的值：

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}
```
- `Some(i)` 可以解包出内部数据。
- Rust 强制要求 match 分支覆盖所有可能性，否则编译报错。

## 1.5 穷举性与兜底分支

Rust 的 `match` 必须穷举所有可能情况。可以用 catch-all（兜底）分支：

```rust
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other),
}
```
- `other` 会绑定未被匹配到的值。

不关注具体值时可用占位符 `_`：

```rust
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => reroll(),
}
```
- `_` 匹配所有未被前面分支覆盖的值。

## 1.6 要点总结

- 枚举让你定义有限的一组类型变体，可以带数据。
- `match` 可安全地处理多种分支，且必须穷举所有情况。
- 模式绑定让你轻松提取枚举成员中的数据。
- `Option<T>` 让你用类型系统防止空值错误。
- `match` 的 catch-all 分支和 `_` 占位符让你优雅处理其他情况。

## 2. 使用 if let 和 let...else 实现简洁的控制流

### 2.1 if let语法

- `if let` 允许你只关心某一种模式的值，忽略其他情况，语法更简洁。例如：

    ```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {max}");
    }
    ```
- 这里的意思是：
    如果 config_max 是 Some(...)，就把里面的值绑定到变量 max 上，执行大括号里的代码。
    如果是 None，什么也不做。
    max 这个变量只在 if let 的花括号 { ... } 里面有效。
- 这样写只在值为 `Some` 时执行代码，变量会自动绑定到模式中数据。

- 与 `match` 相比，`if let` 少了无用分支，但**不强制穷举所有情况**，需根据实际需求权衡。

### 2.2 if let ... else

- 可以配合 `else` 使用，处理主分支和兜底分支，等价于 `match` 的 `_` 分支：

    ```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {state:?}!");
    } else {
        count += 1;
    }
    ```

### 2.3 let...else语法

- Rust 1.65 后引入 `let...else`，适合在模式不匹配时直接进入 else 分支（通常是 return），否则绑定变量后继续主流程：

    ```rust
    fn describe_state_quarter(coin: Coin) -> Option<String> {
        let Coin::Quarter(state) = coin else {
            return None;
        };

        if state.existed_in(1900) {
            Some(format!("{state:?} is pretty old, for America!"))
        } else {
            Some(format!("{state:?} is relatively new."))
        }
    }
    ```

- 这种写法让主流程更清晰，直接表达“幸福路径”。

### 2.4 要点总结

- `if let` 适用于只关心一种模式的场景，代码更简洁但不穷举所有分支。
- `if let ... else` 补全主分支和兜底分支，等价于 `match` 的 `_` 分支。
- `let...else` 让主流程更清晰，遇到不匹配直接走 else，主逻辑更“幸福路径”。
- 选择 `match`、`if let` 或 `let...else`，要根据分支复杂度和是否需要穷举检查来权衡。
- Rust 的枚举和 `Option<T>` 强化类型安全，优雅表达复杂分支逻辑。
# 3. 包与Crate（Packages and Crates）

## 3.1 Crate是什么？

- **Crate** 是 Rust 编译器处理的最小代码单元。即使你只编译一个源文件，该文件也是一个 crate。
- crate 可以包含模块（module），模块可以在其他文件中定义并被一起编译。

## 3.2 Crate的两种类型

- **二进制 crate（binary crate）**：可以编译为可执行程序，必须有 `main` 函数。例如命令行工具、服务器等。
- **库 crate（library crate）**：没有 `main` 函数，不会编译为可执行文件，而是提供可复用功能（比如第三方库）。比如 `rand` crate 提供随机数功能。
- 通常所说的 “crate”，指的是库 crate，也与 “库（library）” 概念互换。

## 3.3 crate root

- **crate root** 是编译器从哪里开始编译 crate 的源文件，构成 crate 的根模块。
- 比如 `src/main.rs` 是二进制 crate 的 crate root，`src/lib.rs` 是库 crate 的 crate root。

## 3.4 Package是什么？

- **Package（包）** 是一个或多个 crate 的集合，提供一组功能。
- 每个 package 都有一个 `Cargo.toml` 文件，描述如何构建这些 crate。
- **Cargo** 本身也是一个 package，包含命令行工具的二进制 crate和供其它项目复用的库 crate。
- 一个 package 可以包含任意数量的二进制 crate，但最多只能有一个库 crate。每个 package 至少包含一个 crate（库或二进制）。

## 3.5 创建包的过程

- 使用命令 `cargo new my-project` 创建包：
    ```
    $ cargo new my-project
    $ ls my-project
    Cargo.toml
    src
    $ ls my-project/src
    main.rs
    ```
- 结果：目录下有 `Cargo.toml`（包描述文件）和 `src` 目录，里面有 `main.rs`。
- `Cargo.toml` 没有直接标注 `src/main.rs`，因为 Cargo 遵循惯例：
    - `src/main.rs` 是二进制 crate 的 crate root，名称与包名一致。
    - 如果有 `src/lib.rs`，则为库 crate，其 crate root 也是该文件，名称与包名一致。
    - 多个二进制 crate可以放在 `src/bin` 目录，每个文件对应一个独立的二进制 crate。

## 3.6 总结要点

- crate 是 Rust 的代码编译单元，有二进制和库两种类型。
- package 是 crate 的集合，有自己的描述文件 `Cargo.toml`。
- 一个 package 至少有一个 crate，最多一个库 crate，可以有多个二进制 crate。
- Cargo 默认约定 crate 的入口文件位置，无需手动配置。

# 4. 定义模块以控制作用域和可见性（Defining Modules to Control Scope and Privacy）

## 4.1 模块系统简介

本节将介绍 Rust 的模块系统，包括模块、路径（path）、`use` 关键字用于引入路径、`pub` 关键字用于公开项，还会涉及 `as` 关键字、外部包与 glob 操作符。

### 模块速查表

- **从 crate 根开始**：编译时，编译器首先查找 crate 根文件（一般是 `src/lib.rs` 或 `src/main.rs`）。
- **声明模块**：在 crate 根文件中用 `mod 模块名;` 声明模块。
    - 编译器会在以下位置查找模块内容：
        - 直接用花括号包裹（内联）
        - `src/模块名.rs`
        - `src/模块名/mod.rs`
- **声明子模块**：在非根文件中声明子模块，如 `mod vegetables;`。
    - 编译器查找位置：
        - 花括号内联
        - `src/父模块/子模块.rs`
        - `src/父模块/子模块/mod.rs`
- **路径访问模块内容**：只要可见，可以用完整路径引用模块内容，如 `crate::garden::vegetables::Asparagus`。
- **私有与公开**：模块内容默认对父模块私有。公开模块和项需用 `pub` 修饰。
- **use 关键字**：在作用域内用 `use` 创建路径的别名，简化访问。例如 `use crate::garden::vegetables::Asparagus;` 后可直接用 `Asparagus`。
- **模块结构举例**：

    ```
    backyard
    ├── Cargo.lock
    ├── Cargo.toml
    └── src
        ├── garden
        │   └── vegetables.rs
        ├── garden.rs
        └── main.rs
    ```

    `src/main.rs` 内容：

    ```rust
    use crate::garden::vegetables::Asparagus;
    pub mod garden;

    fn main() {
        let plant = Asparagus {};
        println!("I'm growing {plant:?}!");
    }
    ```

    `src/garden.rs` 内容：

    ```rust
    pub mod vegetables;
    ```

    `src/garden/vegetables.rs` 内容：

    ```rust
    #[derive(Debug)]
    pub struct Asparagus {}
    ```

## 4.2 模块分组相关代码

- 模块用于在 crate 内组织代码，提升可读性和可复用性。
- 模块还能控制项的可见性，默认私有，用 `pub` 可公开。
- 例如：模拟餐厅的库 crate，可用嵌套模块组织前厅和后厨相关功能。

    ```rust
    mod front_of_house {
        mod hosting {
            fn add_to_waitlist() {}
            fn seat_at_table() {}
        }
        mod serving {
            fn take_order() {}
            fn serve_order() {}
            fn take_payment() {}
        }
    }
    ```

- 这样可以根据功能分组代码，便于维护和查找。

## 4.3 模块树结构

- `src/main.rs` 和 `src/lib.rs` 是 crate 根文件，其内容构成根模块 `crate`。
- 示例模块树：

    ```
    crate
     └── front_of_house
         ├── hosting
         │   ├── add_to_waitlist
         │   └── seat_at_table
         └── serving
             ├── take_order
             ├── serve_order
             └── take_payment
    ```

- 模块可嵌套，兄弟关系、父子关系明确，类似操作系统的文件目录树，用于代码组织和查找。
## 4.4 路径引用、use 关键字与模块公开的实践总结

### 4.4.1 路径（Path）的基本用法

- 路径是模块树里的“地址”，类似于文件系统路径。
- **绝对路径**：从 crate 根开始，例如 `crate::front_of_house::hosting::add_to_waitlist()`。
- **相对路径**：从当前模块开始，例如 `front_of_house::hosting::add_to_waitlist()`，也可用 `self`、`super`。
- 绝对路径类似于 Linux 的 `/usr/bin/xxx`，相对路径类似于当前目录下的 `bin/xxx`。
- 选择哪种路径取决于你的代码结构和维护习惯。

### 4.4.2 访问权限与 pub 公开声明

- Rust 所有项默认对父模块私有（模块、函数、结构体、枚举、常量等）。
- 父模块不能访问子模块的私有内容，但子模块可以访问父模块的项。
- 用 `pub` 关键字将模块或项声明为公开，允许外部模块访问。
- 只有模块和内部项都加 `pub`，外部模块才能访问。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```

### 4.4.3 super 关键字：引用父模块内容

- 用 `super::项名` 可以从当前模块访问父模块内容，类似文件系统的 `../xxx`。
- 适合父子模块之间紧密协作，便于后期模块重构和移动。

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }
    fn cook_order() {}
}
```

### 4.4.4 结构体和枚举的公开规则

- 结构体加 `pub`：结构体本身公开，但字段默认私有，需要单独加 `pub` 才能被外部访问。
- 枚举加 `pub`：所有枚举成员自动公开，无需单独加 `pub`。

```rust
pub struct Breakfast {
    pub toast: String,         // 字段 toast 公开
    seasonal_fruit: String,    // 字段 seasonal_fruit 私有
}
pub enum Appetizer {
    Soup,
    Salad,
}
```

### 4.4.5 use 关键字：路径引入与简化

- 用 `use` 可以简化长路径，在当前作用域直接使用项名。
- 推荐函数用父模块路径，类型用完整路径，有助于代码可读性和定位来源。

```rust
use crate::front_of_house::hosting;
hosting::add_to_waitlist();

use std::collections::HashMap;
let mut map = HashMap::new();
```

### 4.4.6 作用域问题与 use 的局限

- `use` 只在声明它的作用域内有效。
- 如果函数在子模块，需要在子模块内再写 `use`，或用 `super::xxx` 访问父模块引入的名字。

### 4.4.7 名字冲突与 as 重命名

- 当同名项来自不同模块时，可以用 `as` 给一个起别名，避免冲突。

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn fun1() -> Result { /* ... */ }
fn fun2() -> IoResult<()> { /* ... */ }
```

### 4.4.8 pub use —— 重导出（Re-export）

- 普通 use 只让当前作用域可见，外部不能用简化名。
- `pub use` 可让外部也用你的简化名，适合做 API 封装与重导出。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
pub use crate::front_of_house::hosting;
pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### 4.4.9 引入外部包

- 先在 `Cargo.toml` 中声明依赖，比如 `rand = "0.8.5"`。
- 再用 `use rand::Rng;` 或 `use std::collections::HashMap;` 引入需要的 trait 或类型。

### 4.4.10 嵌套路径与 glob 操作符

- 可以用 `{}` 一次性引入同一模块下多个项，减少 use 行数。

```rust
use std::{cmp::Ordering, io};
use std::io::{self, Write};
```

- 用 `use 路径::*;` 一次性引入路径下所有公开项，但需慎用，容易引发名字冲突或难以追踪来源。
- glob 操作符一般用于测试或特殊场景。

```rust
use std::collections::*;
```

### 4.4.11 实践建议与总结

- 路径用于定位模块树中的项，分为绝对和相对路径。
- Rust 默认私有，必须用 pub 公开模块和项，才能被外部访问。
- super 用于访问父模块内容，便于模块重构。
- 结构体字段默认私有，枚举成员默认公开。
- use 简化路径，提升代码可读性。
- 推荐把主要功能放在库 crate，二进制 crate 仅作入口，提高复用性和 API 质量。
- 遇到名字冲突用 as 重命名，pub use 可做 API 封装和重导出。
- 外部包和标准库都用 use 引入，支持嵌套和 glob 简化 use 列表，但 glob 慎用。

## 4.5 将模块拆分到不同文件

### 4.5.1 为什么要拆分模块

- 当模块内容较多时，可以将其移动到单独的文件，便于管理和导航，提高项目可维护性。

### 4.5.2 拆分顶层模块

- 只保留 `mod 模块名;` 声明，去掉模块体，表示该模块定义在同名文件中。
  ```rust
  // src/lib.rs
  mod front_of_house;

  pub use crate::front_of_house::hosting;

  pub fn eat_at_restaurant() {
      hosting::add_to_waitlist();
  }
  ```
- 新建 `src/front_of_house.rs`，放入原本模块体的内容：
  ```rust
  // src/front_of_house.rs
  pub mod hosting {
      pub fn add_to_waitlist() {}
  }
  ```

### 4.5.3 拆分子模块

- 子模块（如 hosting）也是类似做法，将其声明为 `pub mod hosting;`，表示定义在 `src/front_of_house/hosting.rs` 文件中。
  ```rust
  // src/front_of_house.rs
  pub mod hosting;
  ```
  ```rust
  // src/front_of_house/hosting.rs
  pub fn add_to_waitlist() {}
  ```

- 注意：子模块文件必须放在父模块目录下，否则编译器会认为它是顶层模块。

### 4.5.4 模块文件查找规则

- 顶层模块声明后，Rust 会依次查找以下文件：
  - `src/模块名.rs`（推荐）
  - `src/模块名/mod.rs`（旧风格，仍支持）
- 子模块则查找：
  - `src/父模块/子模块.rs`（推荐）
  - `src/父模块/子模块/mod.rs`（旧风格）
- 切勿同模块同时用两种风格，否则编译报错。可以不同模块用不同风格，但不推荐混用，会让项目难以理解。

### 4.5.5 文件拆分后的路径和作用域

- 拆分文件后，模块树结构、路径引用和函数调用都无需修改，依然有效。
- 没有影响 `use` 的用法，`pub use crate::front_of_house::hosting` 依然正确。

### 4.5.6 重要注意事项

- `mod` 只是声明模块，并指定其代码所在文件，**不是像 C/C++ 的 include**，不会简单地把文件内容“拼接”到当前文件。
- 一个文件只需被 mod 声明一次，之后都按路径访问，无需重复 include。
- Rust 的文件结构和模块树紧密对应，利于大型项目组织。

### 4.5.7 旧风格（mod.rs）和新风格的比较

- 新风格推荐用 `模块名.rs`，目录下只用一个文件，易于识别和管理。
- 旧风格用 `mod.rs`，目录下可能有多个 mod.rs，容易混淆，现代项目不推荐用。

### 4.5.8 总结

- Rust 支持将包分为多个 crate，每个 crate可拆分为多个模块，模块可拆分至不同文件。
- 通过绝对路径和相对路径引用模块项，`use` 可简化路径。
- 模块默认私有，用 `pub` 公开。
- 文件拆分不会影响模块树结构和路径引用，便于代码维护和扩展。

---
# 5. 常用集合（Common Collections）

## 5.1 Vec：使用向量存储值列表

### 5.1.1 Vec 向量简介

- Vec<T>（向量）是 Rust 最常用的集合类型之一。它能存储多个同类型的数据，并将这些值在内存中连续排列。
- 适用于保存一组同类型数据，比如文件的每一行文本，或购物车中各个商品的价格。

### 5.1.2 创建向量

- 创建一个空向量可以用 `Vec::new()`，通常需要类型注解，因为没有初始值，Rust 不知道你要存什么类型。
    ```rust
    let v: Vec<i32> = Vec::new();
    ```
- 更常见的是用 `vec!` 宏创建有初值的向量，Rust 会自动推断类型。
    ```rust
    let v = vec![1, 2, 3];
    ```

### 5.1.3 向量的更新

- 用 `push` 方法可以向向量尾部添加元素。需注意变量要用 `mut` 修饰为可变。
    ```rust
    let mut v = Vec::new();
    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
    ```

### 5.1.4 读取向量元素

- 通过下标索引或 `get` 方法访问向量元素。
    ```rust
    let v = vec![1, 2, 3, 4, 5];
    let third: &i32 = &v[2];
    let third: Option<&i32> = v.get(2);
    ```
- 索引越界时，`v[100]` 会导致 panic 程序崩溃；而 `v.get(100)` 返回 None，更适合处理用户输入等不确定情况。

### 5.1.5 向量与借用规则

- 持有元素引用时不能修改向量（比如 push），否则编译报错。原因是 push 可能导致内存重新分配，原有引用失效。
    ```rust
    let mut v = vec![1, 2, 3, 4, 5];
    let first = &v[0];
    v.push(6); // 编译错误：不能同时有可变和不可变借用
    ```

### 5.1.6 迭代向量元素

- 用 for 循环遍历元素，获得不可变引用：
    ```rust
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
    ```
- 想修改每个元素，用可变引用：
    ```rust
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
    ```

### 5.1.7 用枚举存储多类型元素

- Vec 只能存储同类型数据。若需存储多种类型，可定义枚举，让不同类型作为枚举 variant。
    ```rust
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
    ```
- Rust 需在编译期知道所有可能类型，枚举方式适合已知类型场景。如果类型不确定，需用 trait object（后面章节讨论）。

### 5.1.8 向量的释放（Drop）

- 向量和它的所有元素会在离开作用域时自动释放，无需手动管理内存。
    ```rust
    {
        let v = vec![1, 2, 3, 4];
        // v 在这里有效
    } // v 及内部元素在此被释放
    ```

### 5.1.9 小结

- Vec 是 Rust 管理同类型值的基础集合类型，支持高效读写和安全的内存管理。
- 通过枚举可实现多类型存储，借用规则确保数据安全。
- 推荐熟悉 Vec 的 API，灵活使用 push、get、pop、迭代等方法。

---
## 5.2 使用字符串（String）存储 UTF-8 编码文本

### 5.2.1 String 类型简介

- Rust 有两种常用字符串类型：
    - `String`：可增长、可变、拥有所有权的 UTF-8 编码字符串（标准库类型）。
    - `&str`：字符串切片，通常是对某段已存在 UTF-8 编码数据的引用（核心语言类型）。
- 二者都大量用于标准库，均为 UTF-8 编码。

### 5.2.2 创建字符串

- 创建空字符串：
    ```rust
    let mut s = String::new();
    ```
- 用 `to_string` 或 `String::from` 从字符串字面量创建字符串：
    ```rust
    let data = "initial contents";
    let s = data.to_string();
    let s2 = String::from("initial contents");
    ```
- 可存储任何 UTF-8 编码内容，包括多语言文本：
    ```rust
    String::from("こんにちは");
    String::from("Здравствуйте");
    ```

### 5.2.3 更新字符串

- 用 `push_str` 添加字符串切片（不会取得参数所有权）：
    ```rust
    let mut s = String::from("foo");
    s.push_str("bar");
    ```
- 用 `push` 添加单个字符：
    ```rust
    let mut s = String::from("lo");
    s.push('l'); // "lol"
    ```

### 5.2.4 字符串拼接

- 用 `+` 操作符拼接字符串，注意第一个参数会被 move：
    ```rust
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // s1 被 move，s2 可继续使用
    ```
- 多字符串拼接推荐用 `format!` 宏，易读且不会移动参数所有权：
    ```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    let s = format!("{s1}-{s2}-{s3}");
    ```

### 5.2.5 索引和切片

- Rust 字符串不能用下标访问单个字符（如 s[0]），会编译报错。原因：
    - String 是底层 Vec<u8> 包装，UTF-8 编码下一个字符可能占多个字节，简单下标会导致意外或错误。
    - 索引操作预期 O(1) 时间，但字符串无法保证。
- 可以用范围切片获取字节串，但要保证切片边界是字符边界，否则运行时 panic：
    ```rust
    let hello = "Здравствуйте";
    let s = &hello[0..4]; // "Зд"
    // &hello[0..1] 会 panic，因不是字符边界
    ```

### 5.2.6 字符串的三种层次

- 字节（bytes）：底层存储，Vec<u8>
- Unicode 标量值（char）：Rust 的 char 类型
- 字符簇（grapheme cluster）：自然语言里的“字母”或“字符”，标准库不直接支持，可用第三方 crate

### 5.2.7 字符串迭代

- 遍历 Unicode 标量值（char）：
    ```rust
    for c in "Зд".chars() {
        println!("{c}");
    }
    ```
- 遍历原始字节：
    ```rust
    for b in "Зд".bytes() {
        println!("{b}");
    }
    ```

### 5.2.8 字符串的复杂性与安全性

- Rust 默认要求正确处理 UTF-8 字符串，暴露底层复杂性，防止后期出现难以调试的非 ASCII 编码错误。
- 标准库提供很多实用方法（如 contains、replace）帮助处理字符串。

### 5.2.9 小结

- String 类型封装了底层 Vec<u8>，支持高效、灵活的字符串操作。
- 不能直接索引单个字符，需用迭代、切片或专用方法。
- 推荐查阅文档，灵活使用字符串 API，安全处理多语言和 Unicode。

---
## 5.3 HashMap：用哈希表存储键值对

### 5.3.1 HashMap 类型简介

- HashMap<K, V> 用于存储键值映射。每个键（K 类型）对应一个值（V 类型），底层使用哈希函数决定其在内存中的位置。
- 常见于需要用“键”而不是“索引”查找数据的场景，比如统计比赛分数、配置表、员工部门归类等。
- Rust 里的 HashMap 不是自动引入 prelude，需要 `use std::collections::HashMap;`。

### 5.3.2 创建和插入元素

- 创建空哈希表，用 `HashMap::new()`，插入用 `insert`：
    ```rust
    use std::collections::HashMap;
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    ```

### 5.3.3 访问哈希表元素

- 用 `get` 方法访问值，返回 `Option<&V>`，可用 `copied().unwrap_or(默认值)` 处理：
    ```rust
    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
    ```
- 用 for 循环遍历所有键值对（顺序不保证）：
    ```rust
    for (key, value) in &scores {
        println!("{key}: {value}");
    }
    ```

### 5.3.4 所有权与哈希表

- 插入实现 Copy trait 的类型（如 i32）时是拷贝；插入 String 等拥有所有权的类型时是 move，哈希表成为新 owner。
    ```rust
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");
    let mut map = HashMap::new();
    map.insert(field_name, field_value); // field_name/field_value 失效
    ```

### 5.3.5 哈希表的更新和查找/插入模式

- 同一键只能有一个值，多次插入同一键会覆盖旧值。
    ```rust
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25); // "Blue" -> 25
    ```
- 用 `entry` 和 `or_insert` 只在键不存在时插入：
    ```rust
    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);
    ```
- 用 `entry` 能获得该键的可变引用，便于基于原值更新。例如统计单词出现次数：
    ```rust
    let text = "hello world wonderful world";
    let mut map = HashMap::new();
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }
    ```

### 5.3.6 HashMap 默认哈希算法与安全性

- 默认使用 SipHash，安全性高但不是最快，如果有性能需求可指定其它 hasher（实现 BuildHasher trait）。
- 第三方 crate 提供多种哈希算法。

### 5.3.7 小结与练习建议

- HashMap 是存储键值对的基础集合，支持有效查找、插入和更新。
- 适合统计、分组、映射等场景，所有权规则需注意。
- 推荐结合向量和字符串完成如下练习：
    - 统计整数列表的中位数和众数（需用向量和哈希表）。
    - 实现字符串的 pig latin 转换（结合 UTF-8 编码处理）。
    - 用哈希表和向量做公司员工部门管理与查询（按部门分组、按姓名排序）。
- 查阅标准库文档，灵活利用 HashMap 的 API！

---

# 6. 错误处理（Error Handling，翻译）

## 6.1 使用 panic! 处理不可恢复错误

有时候代码中会发生无法解决的严重问题，这时 Rust 提供了 panic! 宏来中止程序。导致 panic 的方式主要有两种：一种是执行某些操作（如访问数组越界）引发 panic，另一种是直接调用 panic! 宏。不论哪种方式，程序都会进入 panic 状态——默认情况下，Rust 会打印错误信息，展开栈（unwind），清理现场并退出程序。你还可以通过环境变量让 Rust 在 panic 时显示调用栈，以便更好地定位错误来源。

### 6.1.1 展开栈与中止（Unwinding or Aborting）

默认情况下，发生 panic 时程序会展开栈，即逐层返回并清理每个函数的数据。这一过程较为耗时，因此 Rust 允许你选择另一种方案：立即中止（abort），这会直接结束程序而不做清理工作。这种情况下，程序占用的内存由操作系统负责回收。如果你希望生成的二进制文件尽可能小，可以在 Cargo.toml 的对应 [profile] 部分加入 panic = 'abort'。例如在 release 模式下使用：

```toml
[profile.release]
panic = 'abort'
```

### 6.1.2 调用 panic! 的示例

```rust
fn main() {
    panic!("crash and burn");
}
```

运行后输出类似：

```
thread 'main' panicked at src/main.rs:2:5:
crash and burn
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

其中，src/main.rs:2:5 表示 panic! 发生在第2行第5个字符。一般情况下，这个报错会指向你自己的代码，但如果 panic! 是库代码内部触发的，显示的位置则是别人代码的位置。

### 6.1.3 利用 backtrace 调查 panic 来源

你可以设置环境变量 RUST_BACKTRACE=1 获得调用栈信息，这样能更好地定位问题。例如：

```rust
fn main() {
    let v = vec![1, 2, 3];
    v[99];
}
```

访问 v[99] 会触发 panic，因为向量只有3个元素。Rust 会直接终止程序，并提示越界错误，防止出现 C 语言中常见的缓冲区越界和安全漏洞。

如果你运行：

```sh
RUST_BACKTRACE=1 cargo run
```

就会看到类似如下的调用栈：

```
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
   1: core::panicking::panic_fmt
   2: core::panicking::panic_bounds_check
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
   6: panic::main
   7: core::ops::function::FnOnce::call_once
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

在这个输出中，调用栈第6行显示了你自己的 main 函数里发生了问题。定位 panic 时，从上往下查找，直到看到你自己写的文件，就是问题的根源。

### 6.1.4 总结

- panic! 用于处理不可恢复的严重错误，默认会展开栈并清理内存，也可以配置为直接中止程序。  
- Rust 的 panic 会防止缓冲区越界等安全隐患。  
- 通过 RUST_BACKTRACE 环境变量，可以获得详细的调用栈，帮助定位问题。  
- 调试时，从调用栈中找到你自己的代码文件，即为问题起点。

稍后章节还会讲何时应该用 panic! 处理错误，何时不应该用。接下来会介绍如何使用 Result 类型进行错误恢复。

---
## 6.2 使用 Result 处理可恢复错误

大多数错误并不严重到需要让整个程序终止。有时候，某个函数失败的原因是可以解释和响应的。例如，如果你尝试打开一个文件但文件不存在，你可能希望新建一个文件，而不是直接终止进程。

### 6.2.1 Result 枚举简介

Rust 的 Result 枚举定义如下：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
- T 表示成功时返回的值类型（Ok）。
- E 表示错误时返回的错误类型（Err）。
- Result 是泛型，可以用于各种不同的场景和类型。

### 6.2.2 文件操作中的 Result

例如，尝试打开文件：

```rust
use std::fs::File;
fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```
File::open 返回的是 Result<File, std::io::Error>，表示要么获得一个文件句柄（Ok），要么返回一个 IO 错误（Err）。

### 6.2.3 用 match 处理 Result

通常需要根据 Result 的值决定下一步操作：

```rust
let greeting_file = match greeting_file_result {
    Ok(file) => file,
    Err(error) => panic!("Problem opening the file: {error:?}"),
};
```
- 成功时返回文件句柄，失败时 panic! 终止程序。

### 6.2.4 匹配不同类型的错误

如果希望对不同错误类型做不同处理，例如文件不存在时新建文件，其他错误时 panic：

```rust
use std::io::ErrorKind;
let greeting_file = match greeting_file_result {
    Ok(file) => file,
    Err(error) => match error.kind() {
        ErrorKind::NotFound => match File::create("hello.txt") {
            Ok(fc) => fc,
            Err(e) => panic!("Problem creating the file: {e:?}"),
        },
        _ => panic!("Problem opening the file: {error:?}"),
    },
};
```

### 6.2.5 更简洁的错误处理：unwrap_or_else

可以用 Result 的方法来简化嵌套 match：

```rust
let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
    if error.kind() == ErrorKind::NotFound {
        File::create("hello.txt").unwrap_or_else(|error| {
            panic!("Problem creating the file: {error:?}");
        })
    } else {
        panic!("Problem opening the file: {error:?}");
    }
});
```

### 6.2.6 快捷方法：unwrap 和 expect

- `unwrap`：如果 Result 是 Ok，返回值；如果是 Err，则 panic。
- `expect`：同 unwrap，但可以自定义 panic! 的错误信息。

```rust
let greeting_file = File::open("hello.txt").expect("hello.txt should be included in this project");
```

### 6.2.7 错误传播（Error Propagation）

有时候，函数内部遇到错误时不直接处理，而是把错误传递给调用者：

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");
    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    let mut username = String::new();
    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```
- 函数返回 Result<String, io::Error>，调用者可以决定如何处理错误。

### 6.2.8 错误传播简化：? 操作符

Rust 提供了问号（?）操作符用于简化错误处理：

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
- 只要遇到 Err，直接返回给调用者。

更简洁的写法：

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

### 6.2.9 ? 操作符的适用范围

- ? 只能用于返回类型为 Result 或 Option 的函数。
- 不能在返回类型为 () 的 main 函数中使用，否则编译错误。
- 解决方法：让 main 返回 Result<(), E> 或手动用 match 处理错误。

```rust
fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;
    Ok(())
}
```
- main 返回 Ok(()) 时进程退出码为 0，返回 Err 时为非 0。

### 6.2.10 ? 也可用于 Option

例如：

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```
- 如果某一步是 None，直接返回 None。

> 注意：? 不能自动转换 Result 和 Option，需要用 ok/ok_or 等方法显式转换。

---

继续后续将介绍何时应该用 panic!，何时应该用 Result，以及如何选择错误处理策略。
## 6.3 应该 panic! 还是返回 Result？

那么，什么时候应该调用 panic!，什么时候应该返回 Result 呢？  
如果代码发生 panic，则无法恢复。你可以对任何错误情况都 panic!，但这样做就是强行把错误变成不可恢复错误。如果你返回 Result，调用者就有选择权：可以尝试恢复，也可以在收到 Err 时自己 panic!。因此，**定义可能失败的函数时，推荐默认返回 Result。**

### 6.3.1 示例、原型代码和测试场景

在写示例代码、原型代码或测试时，直接 panic! 往往更简洁。比如用 unwrap/expect（可能 panic），这表示你还没决定怎么处理错误，或者只是为了演示。  
测试里如果某方法失败，整个测试理应失败，因此 panic! 的 unwrap/expect 是合理的。

### 6.3.2 你比编译器更清楚逻辑时

当你能确定某操作一定成功（比如解析硬编码 IP 地址），但编译器无法推断时，可以用 expect，并在参数里备注原因：

```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1"
    .parse()
    .expect("Hardcoded IP address should be valid");
```

如果 IP 地址是硬编码且有效，expect 是安全的。如果未来 IP 来源变成用户输入，就应改用更健壮的错误处理。

### 6.3.3 错误处理的指导原则

建议在**可能导致程序进入坏状态**时 panic!。坏状态指破坏了某种假设、保证、契约或不变式（如收到无效、矛盾或缺失值），且满足以下之一：
- 这种坏状态**不可预期**（不是偶尔发生的正常错误，如用户输入格式错误）。
- 你的后续代码假定一定不是坏状态，否则就得步步校验，很繁琐。
- 没法用类型系统表达这种约束。

如果别人调用你的库时传入了不合理值，最好返回错误让调用者决定如何处理。但如果继续运行可能导致不安全或有害，直接 panic! 更好，提醒调用者在开发阶段修正 bug。  
如果调用外部代码返回了你无法修复的非法状态，也可以 panic!。

但如果失败是**可预期的**（如解析器收到格式错误数据、HTTP请求被限流），更适合返回 Result，让调用者自己处理。

如果你的代码操作有安全风险（如用无效值），应先验证输入，并在无效时 panic!。这就是标准库在数组越界时 panic! 的原因，防止安全漏洞。  
很多函数有契约：只有满足输入要求才保证行为。契约违背时 panic! 是合理的，因为这是调用者的 bug，且无法恢复。应该在 API 文档里说明哪些情况会 panic!。

多处手动校验参数会很繁琐，Rust 的类型系统可以帮你自动检查。例如参数类型用 u32 就不可能为负数；如果参数类型不是 Option，就一定有值，不必在运行时再检查。

### 6.3.4 用自定义类型做校验

可以用自定义类型进一步确保值有效。例如猜数游戏要求输入范围 1~100：

普通做法：

```rust
let guess: i32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
if guess < 1 || guess > 100 {
    println!("The secret number will be between 1 and 100.");
    continue;
}
```

更好的做法：封装为 Guess 类型，只允许创建 1~100 的值：

```rust
pub struct Guess {
    value: i32,
}
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }
        Guess { value }
    }
    pub fn value(&self) -> i32 {
        self.value
    }
}
```
这样别的代码只能通过 Guess::new 创建 Guess，保证范围有效。Guess 的字段设置为私有，防止外部代码绕过校验。

用 Guess 作为参数/返回值，函数就不用再反复检查范围了。

### 6.3.5 总结

Rust 的错误处理机制（panic! 和 Result）让你能写出更健壮的代码。  
- panic! 表示程序进入了无法处理的状态，直接终止进程。
- Result 用类型系统表达操作可能失败，可恢复。

合理使用 panic! 和 Result，让你的代码在遇到问题时更可靠。

接下来将介绍泛型的原理和用法。

---
# 7. 泛型类型、Trait 和生命周期（翻译）

每种编程语言都有工具来有效地处理概念的重复。在 Rust 中，这样的工具之一就是泛型（generics）：它们是具体类型或其他属性的抽象替代物。我们可以表达泛型的行为，或者它们与其他泛型的关系，而无需在编译和运行代码时知道它们将被什么具体内容替代。

函数可以接受某种泛型类型的参数，而不是像 i32 或 String 这样的具体类型，正如函数可以接受未知值的参数以对多个具体值运行相同的代码一样。实际上，在第 6 章我们已经用过泛型（如 Option<T>），第 8 章用过 Vec<T> 和 HashMap<K, V>，第 9 章用过 Result<T, E>。在本章，你将学习如何用泛型定义你自己的类型、函数和方法！

我们首先回顾如何通过提取函数来减少代码重复。然后使用同样的技术，把仅参数类型不同的两个函数变成一个泛型函数。我们还会讲解如何在结构体和枚举定义中使用泛型类型。

接着你将学习如何用 trait 以泛型方式定义行为。你可以把 trait 与泛型类型结合起来，以限制泛型类型只接受具有某种特定行为的类型，而不是任意类型。

最后，我们会讨论生命周期（lifetimes）：这是泛型的一种，用于告诉编译器引用之间的关系。生命周期让我们为编译器提供足够信息，以保证在更多情况下引用不会失效。

---

## 7.1 通过提取函数消除重复

泛型允许我们用占位符来代表多种类型，从而消除代码重复。在深入了解泛型语法之前，让我们先看看如何通过提取函数来消除重复——不涉及泛型类型，仅仅用函数参数占位具体值。然后我们会用同样的技术提取出一个泛型函数！通过识别可以提取为函数的重复代码，你会逐渐发现可以用泛型消除的重复代码。

### 7.1.1 查找最大值的重复代码

```rust
// 文件名：src/main.rs
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {largest}");
}
```
**列表 10-1：在数字列表中查找最大值**

### 7.1.2 多处重复的代码

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];
    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }
    println!("The largest number is {largest}");

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = &number_list[0];
    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }
    println!("The largest number is {largest}");
}
```
**列表 10-2：分别在两个数字列表中查找最大值的代码**

### 7.1.3 抽象函数消除重复

```rust
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is {result}");

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];
    let result = largest(&number_list);
    println!("The largest number is {result}");
}
```
**列表 10-3：用抽象函数查找两个数字列表最大值的代码**

#### 7.1.4 小结

1. 识别重复代码。
2. 提取为函数，定义输入输出。
3. 用函数替换重复实例。

---

## 7.2 泛型数据类型（Generic Data Types）

我们用泛型来定义函数签名、结构体等项目，然后可以用多种具体数据类型来使用这些定义。首先，我们来看如何用泛型定义函数、结构体、枚举和方法。然后我们会讨论泛型对代码性能的影响。

### 7.2.1 函数定义中的泛型

```rust
fn largest_i32(list: &[i32]) -> &i32 { /* ... */ }
fn largest_char(list: &[char]) -> &char { /* ... */ }
```
**列表 10-4：类型和函数名不同但逻辑相同的函数**

#### 7.2.1.1 用泛型合并函数

```rust
fn largest<T>(list: &[T]) -> &T { /* ... */ }
```
但要能比较大小，需要加 trait bound：

```rust
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T { /* ... */ }
```
`PartialOrd` trait 让类型可比较。

### 7.2.2 结构体定义中的泛型

```rust
struct Point<T> {
    x: T,
    y: T,
}
let integer = Point { x: 5, y: 10 };
let float = Point { x: 1.0, y: 4.0 };
```
**列表 10-6：Point<T> 结构体**

#### 7.2.2.1 多类型字段

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
let integer_and_float = Point { x: 5, y: 4.0 };
```
**列表 10-8：让 x 和 y 类型可不同**

### 7.2.3 枚举定义中的泛型

Option 和 Result 都是泛型枚举：

```rust
enum Option<T> { Some(T), None }
enum Result<T, E> { Ok(T), Err(E) }
```

### 7.2.4 方法定义中的泛型

```rust
struct Point<T> { x: T, y: T }
impl<T> Point<T> {
    fn x(&self) -> &T { &self.x }
}
```
**列表 10-9：泛型结构体的方法定义**

#### 7.2.4.1 只对特定类型实现方法

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 { /* ... */ }
}
```
**列表 10-10：仅对 Point<f32> 实例实现方法**

#### 7.2.4.2 方法与结构体用不同泛型参数

```rust
struct Point<X1, Y1> { x: X1, y: Y1 }
impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> { /* ... */ }
}
```
**列表 10-11：结构体和方法用不同泛型类型**

---

## 7.3 泛型代码的性能

你可能会担心用泛型类型参数会不会有运行时性能损失。好消息是：**用泛型不会让程序变慢**。

Rust 通过编译时的“单态化”（monomorphization）来实现泛型代码的高效。编译器会为每种用到的具体类型生成专属代码。

示例：

```rust
let integer = Some(5);
let float = Some(5.0);
```
编译时会变成：

```rust
enum Option_i32 { Some(i32), None }
enum Option_f64 { Some(f64), None }
let integer = Option_i32::Some(5);
let float = Option_f64::Some(5.0);
```

**单态化让 Rust 的泛型在运行时极其高效。**

---

## 7.4 Trait：定义共享行为

Trait 定义某种类型拥有的功能，并能与其他类型共享。可以用 trait 抽象定义共享行为，也可以用 trait bound 指定泛型类型必须具有某种行为。

Trait 类似其他语言中的接口（interface），但有区别。

### 7.4.1 定义 Trait

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```
**列表 10-12：Summary trait，定义了 summarize 方法**

### 7.4.2 在类型上实现 Trait

```rust
pub struct NewsArticle { /* ... */ }
impl Summary for NewsArticle {
    fn summarize(&self) -> String { /* ... */ }
}
pub struct SocialPost { /* ... */ }
impl Summary for SocialPost {
    fn summarize(&self) -> String { /* ... */ }
}
```
**列表 10-13：在 NewsArticle 和 SocialPost 上实现 Summary trait**

实现 trait 的语法和普通方法类似，只是 impl 后跟 trait 名，再用 for 指定类型。

### 7.4.3 默认实现

Trait 可以有方法默认实现。

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```
**列表 10-14：带默认实现的 Summary trait**

类型可以用默认实现，也可以重写。

#### 7.4.3.1 默认实现可以调用其它 trait 方法

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```
实现时只需定义 `summarize_author`，`summarize` 默认可用。

### 7.4.4 Trait 作为参数

Trait 可用于函数参数，泛型约束类型必须实现某 trait。

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```
也可用 trait bound 语法：

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```
如果有多个参数：

- 不同类型可用 `impl Trait`
- 相同类型用 trait bound

### 7.4.5 多重 trait bound

```rust
pub fn notify(item: &(impl Summary + Display)) { /* ... */ }
pub fn notify<T: Summary + Display>(item: &T) { /* ... */ }
```
where 子句让签名更简洁：

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{ /* ... */ }
```

### 7.4.6 返回实现 trait 的类型

```rust
fn returns_summarizable() -> impl Summary {
    SocialPost { /* ... */ }
}
```
只能返回单一类型。返回多种类型需用 trait object（第 18 章）。

### 7.4.7 条件实现方法/trait

可对泛型类型加 trait bound，**条件实现方法**：

```rust
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) { /* ... */ }
}
```
也可 blanket 实现 trait：

```rust
impl<T: Display> ToString for T { /* ... */ }
```

---

## 7.5 总结

Trait 和 trait bound 让我们用泛型参数减少重复，也能规定类型必须实现某种行为。编译器用 trait bound 检查类型行为，错误在编译时发现而不是运行时，提高了安全性和性能。

---

## 7.6 生命周期（Lifetimes）详解与总结

### 7.6.1 生命周期的作用
生命周期（lifetime）是 Rust 的一种泛型类型，但它不是关注“类型”的行为，而是关注“引用”在内存中的有效性。生命周期确保引用在使用时总是有效，防止悬垂引用（dangling reference）。

### 7.6.2 生命周期防止悬垂引用
Rust 的编译器有一个“借用检查器”（borrow checker），它会比较作用域，确保所有借用都是有效的。例如：

```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x;
    }
    println!("r: {r}"); // 这里 r 指向了已经离开作用域的 x，编译不通过
}
```
Rust 会报错，因为 r 的生命周期比 x 长，r 持有的是“无效的”引用。

### 7.6.3 生命周期标注（Lifetime Annotation）
有时候 Rust 不能自动推断出引用的生命周期关系，需要我们手动标注。生命周期标注用于描述参数和返回值之间的关系，让编译器能正确分析引用是否有效。例如：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```
这里 `'a` 是生命周期参数，表示 x、y 和返回值的生命周期一致。这样 Rust 能确保返回值引用总是有效。

### 7.6.4 生命周期的实际意义
生命周期标注并不会改变引用实际存在的时间，只是“说明”它们之间的关系。
当函数返回引用时，返回值的生命周期必须和参数的生命周期相关联，否则会有悬垂引用风险。

### 7.6.5 结构体中的生命周期
结构体如果包含引用类型字段，也必须标注生命周期。例如：

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```
这说明 ImportantExcerpt 不能比 part 指向的内容活得更久。

### 7.6.6 生命周期省略规则（Lifetime Elision）
大多数常见情况 Rust 能自动推断不需要显式标注生命周期，有三条省略规则：

1. 每个引用参数有自己的生命周期参数。
2. 如果只有一个引用参数，则它的生命周期直接赋给返回值。
3. 如果有多个输入生命周期，但其中一个是 &self 或 &mut self（方法签名），那么返回值生命周期就是 self 的生命周期。

例如：

```rust
fn first_word(s: &str) -> &str {
    // 省略生命周期，因为规则能自动推断
}
```

### 7.6.7 特殊生命周期 `'static`
所有字符串字面量都拥有 `'static` 生命周期，即它们在程序整个运行期间都是有效的。例如：

```rust
let s: &'static str = "I have a static lifetime.";
```

### 7.6.8 泛型、trait bound 和生命周期一起用
可以同时指定泛型类型参数、trait bound 和生命周期参数。例如：

```rust
fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: std::fmt::Display,
{
    println!("Announcement! {ann}");
    if x.len() > y.len() { x } else { y }
}
```
这里 `'a` 是生命周期参数，T 是泛型参数，`T: Display` 是 trait bound。

### 7.6.9 总结
- 生命周期让引用的安全性在编译期就得到保证，防止悬垂引用或非法内存访问。
- 大多数情况下 Rust 能自动推断生命周期，只有涉及多种引用关系时才需手动标注。
- 生命周期和泛型、trait bound一样，都是 Rust 泛型系统的一部分，让代码灵活又安全。
- 生命周期标注描述的是引用间的关系，不会影响实际内存的存活时间。
- Rust 的生命周期机制虽然上手有点陌生，但它是实现高性能和安全的关键特性之一。只要理解了“生命周期是为了描述引用之间的关系”，并善用生命周期省略规则，就能写出既灵活又安全的代码。
# 8 如何编写测试

Rust 提供了强大的自动化测试支持，帮助我们确保代码逻辑的正确性。测试本质上是**验证程序行为是否符合预期**，而不仅仅是类型系统和编译器能保证的部分。本节详细介绍 Rust 测试机制、语法和常用技巧。

---

## 测试函数的基本结构

- 测试是普通的 Rust 函数，只需加上属性 `#[test]` 即可成为测试函数。
- 测试通常**准备数据** → **调用代码** → **断言结果**。

### 示例

```rust
#[test]
fn it_works() {
    let result = add(2, 2);
    assert_eq!(result, 4); // 断言：result 等于 4
}
```

- `#[test]`：告诉测试框架该函数是测试。
- `assert_eq!`：比较两个值是否相等，若不等则测试失败。

---

## 测试模块与可见性

- 测试通常写在 `#[cfg(test)] mod tests` 模块中。
- 测试模块是普通的 Rust 模块，遵循模块可见性规则。
- 若需要访问外部代码，需 `use super::*;`。

### 示例

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn exploration() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

---

## 断言宏详解

### 1. `assert!`

- 检查表达式是否为真，为假则 panic 导致测试失败。

```rust
assert!(result > 0);
```

### 2. `assert_eq!` 和 `assert_ne!`

- `assert_eq!(a, b)`：断言 a == b，测试失败时输出 a、b 的值。
- `assert_ne!(a, b)`：断言 a != b，失败时同样输出详细信息。

```rust
assert_eq!(sum, 42);
assert_ne!(answer, "no");
```

- 这两个宏对比时会自动打印左右值，方便定位问题。

### 3. 自定义失败信息

- 可以在断言后加格式化字符串作为自定义提示。

```rust
assert!(result.contains("Carol"), "Greeting should contain name, got `{}`", result);
```

---

## 测试失败与 panic

- 测试失败的本质是函数触发 panic。
- 可以直接用 `panic!` 宏让测试失败并输出自定义信息。

```rust
#[test]
fn fail_test() {
    panic!("Make this test fail");
}
```

---

## 检查 panic 的测试

- 用 `#[should_panic]` 属性修饰测试函数，断言它会 panic，否则算失败。

```rust
#[test]
#[should_panic]
fn test_panics() {
    panic!("This is a deliberate panic");
}
```

- 可以用 `expected = "xxx"` 指定 panic 消息的一部分，更精确地匹配异常原因。

```rust
#[test]
#[should_panic(expected = "out of range")]
fn test_out_of_range() {
    guess_new(200);
}
```

---

## 使用 Result<T, E> 的测试

- 测试函数可以返回 `Result<(), E>`，这样可用 `?` 操作符处理错误。
- 测试通过时返回 `Ok(())`，失败时返回 `Err(...)`。

```rust
#[test]
fn it_works() -> Result<(), String> {
    let result = add(2, 2);
    if result == 4 {
        Ok(())
    } else {
        Err(String::from("two plus two does not equal four"))
    }
}
```

- 注意：使用 Result 的测试不支持 `#[should_panic]`。

---

## 运行和管理测试

- 运行所有测试：`cargo test`
- 只运行指定测试：`cargo test test_name`
- 可以忽略测试（详见后续章节）

---

## 典型测试模式总结

1. **等值断言**：assert_eq!、assert_ne!
2. **条件断言**：assert!
3. **异常断言**：#[should_panic]
4. **自定义提示**：assert!(cond, "message {}", info)
5. **测试返回 Result 支持 ? 运算符**

---

## 小结

- Rust 测试机制简单易用，断言宏丰富，支持精确定位和自定义提示。
- 推荐：每个功能都写测试，及时发现逻辑错误。
- 测试不仅验证正确性，还能作为代码文档，指导后续开发和维护。

---
## 控制测试的运行方式

Rust 的测试框架允许开发者灵活控制测试运行的方式，包括并发、输出、筛选和忽略等。了解这些控制方法有助于提高测试效率和调试体验。

---

## 测试运行模式与命令行参数

- `cargo test` 会编译你的代码并运行所有测试（默认并行）。
- 有些参数传给 `cargo test`，有些参数传给测试二进制文件，二者用 `--` 分隔。
- 查看帮助：  
  - `cargo test --help` 查看 cargo 层参数
  - `cargo test -- --help` 查看测试运行器参数

---

## 并行与串行运行测试

- **默认并行**（多线程）运行所有测试，加快测试速度。
- 测试间必须**互不依赖**，不能共享状态（如文件、环境变量），否则可能互相干扰导致假失败。
- 例如，多个测试同时读写同一个文件可能导致冲突。

### 串行运行测试

如需串行运行所有测试，可指定线程数为 1：

```bash
cargo test -- --test-threads=1
```

这样测试会逐个运行，适用于有共享状态或需严格顺序的场景。

---

## 测试输出展示

- **默认行为**：通过 `println!` 打印的内容只会在测试失败时显示，测试通过则被捕获不显示。
- 这样可以保持测试结果简洁，专注于通过/失败信息。

### 显示所有测试输出

如果想看所有测试的标准输出（包括通过的测试），加 `--show-output`：

```bash
cargo test -- --show-output
```

---

## 只运行指定测试

- 可以通过传递测试名，只运行部分测试：

```bash
cargo test test_name
```

- 只运行名字包含指定字符串的所有测试：

```bash
cargo test partial_name
```

例如，下面代码有三条测试：

```rust
#[test]
fn add_two_and_two() { ... }
#[test]
fn add_three_and_two() { ... }
#[test]
fn one_hundred() { ... }
```

- 只运行 `one_hundred` 测试：

```bash
cargo test one_hundred
```

- 运行所有名字包含 `add` 的测试：

```bash
cargo test add
```

---

## 忽略部分测试

- 某些测试可能很耗时或特殊，不希望每次都运行，可以加上 `#[ignore]` 属性：

```rust
#[test]
#[ignore]
fn expensive_test() {
    // 可能运行一小时的测试代码
}
```

- 默认情况下被忽略的测试不会运行，但会在测试结果中列出。

### 只运行被忽略的测试

```bash
cargo test -- --ignored
```

### 包含所有测试（包括 ignored）

```bash
cargo test -- --include-ignored
```

---

## 总结关键点

- Rust 测试默认并行运行，需避免测试间共享和互相干扰。
- 可以控制测试线程数，串行运行所有测试。
- 输出只在测试失败时显示，通过时可用 `--show-output` 展示所有输出。
- 可通过测试名/部分名筛选运行指定测试。
- 用 `#[ignore]` 跳过特殊测试，需要时可单独运行。
- 灵活控制测试运行方式，提高效率和调试便利性。

---
## 测试组织

Rust 社区将测试分为两大类：**单元测试（unit tests）**和**集成测试（integration tests）**。这两类测试互补，确保你的库在细粒度和整体层面都能正确工作。

---

## 单元测试（Unit Tests）

- 目的：**独立测试代码的每个单元**，快速定位问题。
- 位置：放在 `src` 目录下，与被测试代码同文件。
- 组织方式：每个文件内通常有一个 `#[cfg(test)] mod tests` 测试模块。
- `#[cfg(test)]`：编译选项，只有运行 `cargo test` 时才编译测试代码，不影响正常构建和产物体积。

### 示例：

```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

### 测试私有函数

Rust 的模块系统允许测试私有函数（无 `pub` 修饰），只需在测试模块用 `use super::*;` 引入父模块内容：

```rust
pub fn add_two(a: u64) -> u64 {
    internal_adder(a, 2)
}

fn internal_adder(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn internal() {
        let result = internal_adder(2, 2);
        assert_eq!(result, 4);
    }
}
```

---

## 集成测试（Integration Tests）

- 目的：**像外部用户一样测试库的公共 API**，验证各模块协作正确。
- 位置：顶层 `tests` 目录，每个文件一个 crate，独立编译。
- 每个测试文件需 `use crate_name::public_fn` 显式导入 API。
- 无需 `#[cfg(test)]`，cargo 会自动只在测试时编译。

### 示例目录结构

```
adder
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

### 示例代码

```rust name=tests/integration_test.rs
use adder::add_two;

#[test]
fn it_adds_two() {
    let result = add_two(2);
    assert_eq!(result, 4);
}
```

### 运行指定集成测试文件

```bash
cargo test --test integration_test
```

---

## 集成测试的辅助模块

- 多个测试文件可用 `tests/common/mod.rs` 共享辅助代码。
- 不能用 `tests/common.rs`，否则会被当成测试 crate，显示在测试结果中。
- 使用 `mod common;` 引入，调用其中函数。

### 示例

```
tests/
├── common/
│   └── mod.rs
└── integration_test.rs
```

```rust name=tests/common/mod.rs
pub fn setup() {
    // 通用测试初始化代码
}
```

```rust name=tests/integration_test.rs
use adder::add_two;
mod common;

#[test]
fn it_adds_two() {
    common::setup();
    let result = add_two(2);
    assert_eq!(result, 4);
}
```

---

## 二进制 crate 的集成测试

- 如果项目只有 `src/main.rs`，不能直接为 main 代码写集成测试。
- 推荐将主要逻辑放到 `src/lib.rs`，main.rs 只做调用，这样可对 lib 进行集成测试。

---

## 小结

- 单元测试关注细节、可测试私有代码，写在源码文件内。
- 集成测试关注整体协作、只测试公共 API，写在 `tests` 目录下，每个文件单独 crate。
- Rust 测试机制灵活，能覆盖从底层到高层的所有场景，是确保代码行为正确的关键工具。

---
# 9. 函数式语言特性：迭代器与闭包（详细代码解读）

Rust 支持现代函数式编程语言的特性，尤其是**迭代器（Iterators）**和**闭包（Closures）**，让代码简洁、表达力强、安全高效。本节将详细逐行解读闭包相关代码示例。

---

## 9.1 闭包：捕获环境的匿名函数

闭包是可以捕获其定义时环境变量的匿名函数。它们既可以赋值给变量，也可以作为参数传递。相比普通函数，闭包能自动捕获作用域中的变量，实现代码复用和行为定制。

---

### 1. 闭包的基本语法与环境捕获

闭包的定义和使用：

```rust
let x = 5;                                 // 定义一个变量 x，值为 5
let closure = |y| x + y;                   // 定义一个闭包，参数是 y，返回 x + y。闭包自动捕获 x 的值
println!("{}", closure(3));                 // 调用闭包，传入 3，实际执行 x + 3，即 5 + 3，输出 8
```

- `let x = 5;` 创建一个普通变量。
- `let closure = |y| x + y;` 定义一个闭包，参数为 y，自动捕获外部变量 x。
- `closure(3)` 实际等价于 5 + 3，因为此闭包会记住 x 的值。

---

### 2. 闭包类型推断与可选类型标注

闭包参数和返回值类型通常自动推断，也可以手动指定：

```rust
let add = |x: i32, y: i32| -> i32 { x + y };  // 明确指定参数类型和返回值类型
```

- `|x: i32, y: i32| -> i32 { x + y }` 是完整写法，参数类型和返回类型都显式声明。

函数与闭包的写法对比：

```rust
fn add_one_v1(x: u32) -> u32 { x + 1 }        // 普通函数定义
let add_one_v2 = |x: u32| -> u32 { x + 1 };   // 类型完全标注的闭包
let add_one_v3 = |x| { x + 1 };               // 无类型标注，类型由上下文推断
let add_one_v4 = |x| x + 1;                   // 单表达式闭包可省略大括号
```

- 前两种写法类型明确，后两种需依赖推断。Rust 编译器会根据首次调用推断类型，后续调用必须一致。

---

### 3. 闭包的捕获方式详解

闭包可以三种方式捕获环境变量：

- **不可变借用（&T）**：只读环境变量。
- **可变借用（&mut T）**：可修改环境变量。
- **所有权转移（T）**：获得变量所有权，适用于多线程或跨作用域场景。

具体示例：

```rust
let list = vec![1, 2, 3];                    // 定义一个整型向量
let show = || println!("{:?}", list);        // 闭包只读 list，不修改
show();                                      // 输出 [1, 2, 3]

let mut list = vec![1, 2, 3];                // 定义可变向量
let mut change = || list.push(4);            // 闭包可修改 list（push 新元素）
change();                                    // list 变为 [1, 2, 3, 4]

let list = vec![1, 2, 3];                    // 定义向量
std::thread::spawn(move || println!("{:?}", list)); // move 关键字使闭包获得 list 的所有权
```

- 第一个例子：闭包只读外部变量，后续还可访问 list。
- 第二个例子：闭包修改外部变量，必须声明变量为 `mut`，闭包也需 `mut`。
- 第三个例子：闭包通过 `move` 获得变量所有权，适合用于线程，避免借用冲突。

---

### 4. 闭包的 Fn/FnMut/FnOnce Trait 及标准库方法

根据捕获环境变量的方式，闭包会自动实现：

- **FnOnce**：所有闭包至少实现。消耗所有权，只能调用一次。
- **FnMut**：可多次调用，可修改环境变量。
- **Fn**：可多次调用，只读或不捕获环境变量。

#### 标准库方法例子

- `Option::unwrap_or_else` 要求闭包实现 `FnOnce`，因为只会调用一次：

    ```rust
    let opt: Option<i32> = None;
    let v = opt.unwrap_or_else(|| 42); // 闭包只会被调用一次
    ```

- `slice::sort_by_key` 要求闭包实现 `FnMut`，因为会对每个元素调用多次：

    ```rust
    let mut arr = [3, 1, 2];
    arr.sort_by_key(|x| *x); // 闭包被多次调用
    ```

---

### 5. 总结

- 闭包是 Rust 函数式编程的核心特性，可以灵活捕获环境变量。
- 类型推断让闭包书写简洁，也可以手动标注类型。
- 自动实现 Fn/FnMut/FnOnce trait，影响闭包如何被标准库方法接受。
- 闭包与迭代器等组合使用，可让数据处理代码更简洁高效。
- 理解闭包的捕获方式和 trait 实现，有助于编写更高效、更安全的 Rust 代码。

---
## 9.2 迭代器：按序处理元素（详细代码解读）

Rust 的迭代器模式让你可以优雅、高效地处理数据序列，无需手动实现遍历逻辑。本节将逐行详细讲解关键代码，帮助你理解每一步的作用和底层机制。

---

### 1. 创建迭代器

```rust
let v1 = vec![1, 2, 3];        // 创建一个向量，包含三个整数
let v1_iter = v1.iter();       // 调用 .iter() 方法，返回一个遍历 v1 的迭代器（元素类型为 &i32）
```
- `vec![1, 2, 3]` 创建了一个动态数组（向量）。
- `.iter()` 返回一个迭代器，每个元素是 v1 中元素的不可变引用（&i32）。
- 将迭代器赋值给变量 `v1_iter`，后续可以用它遍历 v1。

---

### 2. 使用迭代器

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {val}");
}
```
- `for val in v1_iter` 语法自动消耗迭代器，每次循环取得一个元素引用。
- `println!("Got: {val}")` 打印当前元素的值。
- Rust 编译器自动处理迭代器的生命周期和可变性，无需关心迭代细节。
- 注意：一旦 for 循环结束，迭代器已被消耗，不能重复使用。

---

### 3. Iterator Trait 和 next 方法

Rust 的迭代器都实现了标准库的 `Iterator` trait：

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```
- `type Item;`：定义迭代器返回元素的类型（如 &i32）。
- `next(&mut self)`：每次调用返回下一个元素（包裹在 Option 里）。
    - 如果有元素，返回 `Some(item)`。
    - 如果序列结束，返回 `None`。

#### 例子：手动调用 next

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];               // 创建一个向量
    let mut v1_iter = v1.iter();          // 创建迭代器，需声明为 mut，因为 next 会改变迭代器状态

    assert_eq!(v1_iter.next(), Some(&1)); // 取第一个元素，返回 &1
    assert_eq!(v1_iter.next(), Some(&2)); // 取第二个元素，返回 &2
    assert_eq!(v1_iter.next(), Some(&3)); // 取第三个元素，返回 &3
    assert_eq!(v1_iter.next(), None);     // 没有更多元素，返回 None
}
```
- 每次 next 都会消耗迭代器一个元素，指针向后移动。
- 用 `assert_eq!` 验证每一步的返回值，确保迭代正确。
- 迭代器只能从前到后遍历一次，不能回退。

---

### 4. 消耗迭代器的方法

Rust 的 Iterator trait 有许多方法可以“一次性消耗”迭代器，如 `sum`：

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();           // 创建迭代器
    let total: i32 = v1_iter.sum();    // sum 消耗迭代器，累加所有元素，返回总和
    assert_eq!(total, 6);              // 验证总和为 6
}
```
- `sum()` 方法会遍历每个元素，累计求和，最终返回结果。
- 一旦 sum 消耗了迭代器，迭代器不能再用。
- 其它类似方法还有 `count`, `max`, `min` 等。

---

### 5. 迭代器适配器：返回新的迭代器

“迭代器适配器”不会直接消耗迭代器，而是返回一个新的迭代器。例如 `map`：

```rust
let v1: Vec<i32> = vec![1, 2, 3];                       // 创建一个整数向量
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();    // 用 map 生成新迭代器，每个元素加一，最后 collect 收集成新向量
assert_eq!(v2, vec![2, 3, 4]);
```
- `map(|x| x + 1)`：对每个元素应用闭包 `|x| x + 1`，返回新迭代器。
- `collect()`：消耗新迭代器，收集所有元素到一个集合（如 Vec）。
- 注意：如果没有 collect/sum/count 等消耗方法，map 不会真正执行，因为迭代器是**惰性**的。

---

### 6. 捕获环境的闭包与迭代器适配器

许多迭代器适配器（如 filter）都可以用捕获外部变量的闭包：

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
```
- `shoes.into_iter()`：获得所有权，遍历每个 Shoe 实例。
- `filter(|s| s.size == shoe_size)`：用闭包筛选鞋码匹配的鞋子。
    - 闭包 `|s| s.size == shoe_size` 捕获了外部变量 `shoe_size`。
- `collect()`：把筛选结果收集到新向量里。

#### 测试示例

```rust
#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_size(shoes, 10); // 只选出鞋码为 10 的鞋子

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```
- 用断言确保只返回鞋码为 10 的鞋子。
- 展现了迭代器+闭包组合的强大筛选能力。

---

### 7. 总结

- Rust 的迭代器让你用更简洁、更安全的方式处理序列数据。
- 迭代器是惰性的，只有被消耗时才真正执行。
- 可以链式组合多个适配器，灵活处理数据。
- 闭包能捕获环境变量，配合迭代器适配器非常强大。
- 推荐：多用迭代器和闭包，简化代码逻辑，提高可读性和安全性。

---