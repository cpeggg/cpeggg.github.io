---
title: rust学习
tags: rust
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->
最近捡起了rust，将其中一些知识点进行记录

<!--more-->

## 学习资源

<https://github.com/rust-lang/book> 官方教程

<https://docs.microsoft.com/zh-cn/learn/paths/rust-first-steps/> 微软出的教程

<https://doc.rust-lang.org/nightly/std/all.html> 常用api的文档查询

《深入浅出Rust》还不错，适合在有知识背景下在地铁上翻翻快速阅读

## 一些笔记

### rustup

1. 使用`rustup install stable/beta/nighty`来安装不同版本rust：
   1. stable稳定版
   2. beta dev向稳定过渡版本，nighty经验证的功能会在这上面开放
   3. nighty dev版
   
   所以一般装stable即可
2. 使用`rustup update`一键升级rustc及相关工具版本
3. 使用`rustup self uninstall`卸载所有组件
4. Proxy：`export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup`，具体切换到清华源的教程可见[清华镜像站](https://mirror.tuna.tsinghua.edu.cn/help/rustup/)

### Cargo

1. cargo new的时候默认自动创建`.gitignore`和`.git`，使用`--vcs=none`来去掉此行为
2. cargo build默认生成unoptimized和debug模式的binary，使用`--release`来生成release版本代码。同时cargo build并且会在项目根目录下生成Cargo.lock（类似Gemfile.lock）来进行自动化的版本控制
3. 使用cargo check来检查代码是否可编译（但不编译生成文件）
4. 使用cargo run进行`build+运行`的操作，其同样可以使用`--release`来进行控制
5. cargo update可以更新项目所依赖的crates
6. cargo build时下载的第三方crate更改代理到[清华镜像站](https://mirror.tuna.tsinghua.edu.cn/help/crates.io-index.git/)
7. 如果对import的crate不熟悉，可以用`cargo doc --open`来打开和当前项目所依赖的各个crate文档

### 基本语法

#### 变量

1. 对于unused的variable，编译器会抛出warning，这当中包括一些函数调用的返回值（如Ok和Err，因此其需要再用expect等函数修饰调用，或者用match关键字来针对返回值进行处理）
   举个栗子：
   ```rust
      // way 1
      let number:u32 = number.trim().parse().expect("Failed to parse");
      // way 2
      let number:u32 = match number.trim().parse(){
         Ok(num) => num,
         Err(_) => println("Failed to parse"),  
      };
   ```
2. 整数溢出问题：
   
   对于整数溢出，debug编译模式下会对整数的运算添加check，从而在运行程序的时候能够动态检测整数溢出的情况并抛出panic，而在release下则不会有上述的行为，因而会发生类似C/C++下的整数溢出行为。**在rust设计中，依赖隐式的整数溢出来设计逻辑被认为是错误行为**

   对于这种情况，可以使用如下的一些方法来进行明确处理：
   - `wrapping_*`：显式C/C++的wrapping处理方式
   - `checked_*`：如果发生溢出，返回None
   - `overflowing_*`：返回值和一个是否发生溢出行为的bool
   - `saturating_*`：返回变量返回的最大/最小值

3. char类型占4bytes（unicode），使用u8来单字节表示，char使用单引号的形式赋常量值，u8则使用对应的数字（声明u8类型，使用类似`8u8`的赋值，或使用`b''`的形式表示）。需要区分的是，在`str`或者是`String`中，由于使用UTF8进行编码，因此中间字符的实际占用大小可能为1-4个字节不等。
4. tuple/struct/tuple struct的区别：
   - tuple对成员不命名，本身也没有名字，使用.0 .1 .2 这样来访问成员（和array有区别，array中仍然用[0] [1] [2]这样去访问）
   - struct对成员命名，本身有名字（类似c/c++中的struct定义，相当于定义了一个类型），使用对应命名来访问成员
   - tuple struct对成员不命名，本身有名字（类似c/c++中的struct定义，相当于定义了一个类型），使用.0 .1 .2这样来访问成员
   关于其赋值时的简化写法以及语法糖参见《深入浅出Rust》中的相应内容
5. 数组的index为usize类型
6. rust支持通过类似`(1..6)`这样的方式构建`std::ops::Range<Integer>`类型，但这和通过`[1, 2, 3, 4, 5]`构建出来的`[{integer}; 5]`类型有区别
7. 通过`String::from/new`这样构造出来的变量类型为`alloc::string::String`；通过`="xxx"`这样构造出来的为`&str`，也可以使用`=b"xxxx"`来构造`&[u8]`类型的变量

#### 函数、代码块

1. 有如下的代码：
   ```rust
   fn main() {
      let y = {
         let x = 3;
         x + 1                       // no ';' here!
      };

      println!("The value of y is: {}", y);
   }
   ```
   可以看到，这里的`x + 1`后没有`;`，这表示将`x + 1`视作表达式，并将其计算得到的值返回用于`y`的赋值，而如果在`x + 1`后添加分号，则大括号间的内容变成了代码块，相应的y虽然能被赋值为`()`但实际上这是没有意义的，在之后的println中也会显示无法打印的错误。
2. 与之类似的也有函数的定义：对于有返回值的函数，不使用单独的return，而是在需要return的位置直接写需要return的表达式即可，如：
   ```rust
   fn plus_one(x:i32) -> i32{
      x + 1                          // no ';' here!
   }
   ```
   当然，也可以显式用return返回表达式的值，此时是仍然需要分号的

#### 控制流

1. if语句和C/C++类似，区别在于：
   1. 条件不需要用`()`包起来
   2. 条件表达式的结果必须是bool类型
   没有`elif`语句
2. 新增`loop`关键字表示无限循环
   1. 通过在loop前添加标签符号（loop label）可以在nested loop中break/continue上层的loop
   2. 和普通代码块类似，loop可以通过break语句来返回特定值，此时的break需要带`;`
3. while和C/C++类似，除了条件不用`()`包起来
4. 有类似python的`for ... in ...`语句

#### 所有权(Ownership)

1. 对于复杂结构类型，`let s1 = s2`会导致浅拷贝的发生，在c/c++/python中，浅拷贝会导致对一个变量的修改会对另一个变量的值产生影响，在Rust的设计当中，浅拷贝被视为move操作，因此对被move的变量的操作将被视为invalidated reference而会抛出编译错误，如果需要仍然使用之前被move的变量，使用类型中的`.copy()`方法来进行深拷贝
2. 为了解决上述复杂的move导致的所有权迁移关系，引用（reference）的概念被提出，和C++中的类似，函数在使用引用参数时需要声明对应的变量类型为`&<type>`，但和C++不同的是，在调用该函数时，传参时同样需要使用`&`来显式地“取”数据的地址进行传递。mut的性质将通过reference进行传递（即reference前后不改变是否可mutate的性质）
3. 对于2.中的mut引用，被引用的变量必须也为mut。同时，一份数据在其数据周期中只能有最多一个mutable引用：
   ```rust
   let mut s0 = String::from("hello");
   let r1 = &mut s0;
   let r2 = &mut s0; // compile error
   println!("{}", r1);
   println!("{}", r2);
   ```
   但是下述的代码是可以编译通过且运行的：
   ```rust
   let mut s0 = String::from("hello");
   let r1 = &mut s0;
   println!("{}", r1);
   let r2 = &mut s0;
   println!("{}", r2);  // change to println!("{} {}", r1, r2); will raise same compile error
   ```
   之所以有如上的情况，是因为编译器认为`r1`的使用范围到println为止，而r2是在r1不再使用后开始被定义和使用的
4. 同3.类似，当数据有immutable引用时，在引用生效范围内不能再有mutable引用
5. 为了预防dangling pointer，引用必须在被引用的数据生命周期内使用
6. slice type：返回一个序列中的特定连续片段，同时不拥有其所有权，也同样会使被slice的变量的mutable引用操作产生编译错误，如下面的代码：
   ```rust
   fn main() {
      let mut s = String::from("hello world");
      let word = first_word(&s);  // immutable ref
      s.clear();                  // mutable ref here, raise compile error
      println!("the first word is: {}", word);
   }
   ```
   `&str`类型即是一种slice
7. ```rust
   let a = [1, 2, 3, 4, 5];
   let slice = &a[1..3];
   assert_eq!(slice, &[2, 3]);
   ```

#### Structs

1. struct声明的时候不需要分号结尾
2. 可以类似python的命名传参的方式，特定改变传参顺序，如：
   ```rust
   struct User{
      active: bool,
      username: String,
      email: String,
      sign_in_count: u64
   }
   // -- snip --
   fn build_user(username: String, email: String) -> User {
      User{
         username,
         email,
         active: true,
         sign_in_count: 0
      }
   }
   ```
3. tuple struct使用样例：
   ```rust
   struct Color(i32, i32, i32);
   struct Point(i32, i32, i32);
   fn main() {
      let black = Color(0, 0, 0);
      let origin = Point(0, 0, 0);
   }
   ```
4. struct可以定义为空：
   ```rust
   struct AlwaysEqual;
   fn main() {
      let subject = AlwaysEqual;
   }
   ```
5. impl来为struct定义方法，传入参数需要为`&self`，impl实现的方法可以与成员变量同名，上述实现的方法称为`Associated Functions`，impl中可以定义多个相关函数，也可以在不同的impl但针对同一struct的代码块中分别定义
6. 不使用`&self`作为传入参数的`Associated Functions`不能称为struct的方法，因此不能用`.`来访问和调用，只能通过`::`来访问和调用，如：
   ```rust
   impl Rectangle {
      fn square(size: u32) -> Rectangle {
         Rectangle {
            width: size,
            height: size,
         }
      }
   }
   // -- snip --
   let rect = Rectangle::square(5);
   ```

#### 枚举类型

1. enum中的成员同样可以通过`::`来访问
2. enum可以append数据，如：
   ```rust
   enum IpAddr {
      V4(String),
      V6(String),
   }
   let home = IpAddr::V4(String::from("127.0.0.1"));
   let loopback = IpAddr::V6(String::from("::1"));
   ```
   上述的`String`也可以是包括`struct`在内的，甚至各个成员append的数据类型也可以不同
3. `Option<T>`：由于Rust中默认不能将变量赋值为null/None（为了防止和其他类型混淆），那么为了解决应用中需要表示特定数据可能为空的情况（如成绩Score可能为一个特定值，也可能尚未录入而为null），引入`Option<T>`来表示一个特定数据的值。
   
   还是以成绩`Score`为例，对于打了95分的同学，有：
   ```rust
   let some_score1:Option<i32> = Some(5);
   let some_score2 = Some(5);
   ```
   对于尚未录入成绩的空值，有：
   ```rust
   let absent_score:Option<i32> = None;
   ```
   需要注意的是，在使用Some赋值时，`Option<i32>`可以不用指定（因为可以通过`Some`中的值自动推断），但使用`None`赋值时，必须使用特定的类型为`Option`指定的类型模版进行具体的初始化。同时，使用`Option<T>`赋值的变量的类型不是`T`而是`Option<T>`，因此对于其的值不能用类型`T`的方式来直接处理（如当`T`为`i32`，则不能将`Option<i32>`的值和`i32`的值进行直接运算。
4. `match`类似switch，可以使用`{}`来扩展对应case中的执行语句，使用`other`或`_`作为default情况处理
5. 特殊地，当`match`和`Some`结合使用时，可以有如下的使用形式：
   ```rust
   let x = Some(5);
   let y = match x {
      Some(i) => i,
      _ => 0,
   }; // y == 5 here
   let z = match x {
      Some(i) => Some(i+1),
      _ => 0,
   }; // z == Some(6) here
   ```
   其中的`Some`也可以是enum类型，相应的括号内的变量则对应是enum中的append数据
6. `if let`语句可以合并match中的特定情况赋值操作，其中的else可以用来实现match中的`other`情况

### 包管理

1. 使用`cargo new --lib`来创建代码库的repo
2. crate是当前代码库的默认最大module，module通过`mod`来进行树形定义
3. 通过在需要向module外暴露使用的数据、函数前添加`pub`关键字来make public
4. 通过`use`关键字来使用module或者定义在module下的数据、函数：
   ```rust
   mod front_of_house {
      pub mod hosting {
         pub fn add_to_waitlist() {}
      }
   }
   use crate::front_of_house::hosting;
   pub fn eat_at_restaurant() {
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
      hosting::add_to_waitlist();
   }
   ```
5. 通过`self`和`super`进行2.中树上的相对路径module访问
6. rust支持和python中类似的`as`关键字
7. `use`同样支持被再次`pub`来再暴露给其他module使用
8. 对于外部的crates，通过在`Cargo.toml`声明后cargo会在编译前期自动下载相关依赖
9. `use`支持同路径下的合并：
   ```rust
   use std::cmp::Ordering;
   use std::io;
   // equals to
   use std::{cmp::Ordering, io};

   use std::io;
   use std::io::Write;
   // equals to
   use std::io::{self, Write};
   ```
10. `use`中的`*`则是指：将对应module下的所有public对象导入
11. 为了将不同的module能够分散到不同的文件当中去，需要按2.中的树形建立对应的文件夹以及相应的`module.rs`文件，在上层文件中还需要单独写一行`mod xxx_module;`来告诉编译器在另一个同名的`.rs`文件中去加载module的内容

### 一些常用数据类型（类比c++中的stl）

1. 使用`&`来引用访问vector中的元素会产生(im)mutable borrow
2. 使用`.get`方法来产生`Option<T>`结果，可以对超出边界的情况进行处理
3. vector内的元素类型需保持一致，但可以用enum中的类型append来进行扩展
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
4. String类支持push_str()方法和push()方法，分别接受`str`类型和`char`类型的输入
5. 在Rust的`String/str`类型中，由于单个字符使用unicode表示，类似`"你好"[0]`的indexing操作可能有如下两种歧义：
   1. 取第一个字符`'你'`
   2. 取该字符串对应的bytes中的第一个字节
   因此为了避免歧义，Rust禁止了这种index方式，使用`.chars().nth()` or `.bytes().nth()`来进行indexing。

   另一个Rust要禁止这种indexing方式的原因是，通常indexing的复杂度期望是`O(1)`，而实际上Rust需要消耗额外的时间复杂度来从字符串开头到指定index位置之间的内容中有多少的有效字符。
6. 但和访问单个字符/字节的需求不同，`str`和`String`支持使用切片(slicing)的方式来进行数据访问，此时切片的上下标表示的是字符串对应的字节序列中的对应位置，如：
   ```rust
   let hello = "Здравствуйте";
   let s = &hello[0..4];
   ```
   会得到`"Зд"`的结果，因为这两个字符刚好占4个字节数，而如果是
   ```rust
   let s = &hello[0..1];
   ```
   则会产生运行错误（而非编译错误），因为此时下标1还在字符'3'的字节范围内：
   ```
   panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`'
   ```
7. Rust中通过两个vector来创建hashmap的方法：
   ```rust
   use std::collections::HashMap;
   let teams = vec![String::from("Blue"), String::from("Yellow")];
   let initial_scores = vec![10, 50];

   let mut scores: HashMap<_, _> = teams.into_iter().zip(initial_scores.into_iter()).collect();
   ```
8. 需要注意的是，当使用String等有深浅拷贝之分的类型变量对hashmap进行赋值时，其执行的是浅拷贝操作，也因此，用来赋值的变量在赋值给hashmap之后不能再被使用，如以下的使用场景：
   ```rust
   let mut scores = HashMap::new();
   scores.insert(String::from("Blue"), 10);
   scores.insert(String::from("Yellow"), 50);

   let team_name = String::from("Blue");
   let score = scores.get(&team_name);
   ```
   中可以看到，如果需要访问某个键的值时，需要再重新let赋值一个变量来进行访问。
9. 使用hashmap中的`entry()`方法来访问某个键的值，搭配`or_insert()`方法来对未赋值的keyvalue进行赋值，`or_insert()`方法返回`&mut V`，因此对于如下的代码。有：
   ```rust
   let text = "hello world wonderful world";
   let mut map = HashMap::new();
   for word in text.split_whitespace() {
      let count = map.entry(word).or_insert(0);
      *count += 1;
   }
   println!("{:?}", map);
   ```
   最终结果为`{"hello": 1, "world": 2, "wonderful": 1}`，因为count为keyvalue的可变引用，同时可以看到，对于类型为`&mut`的变量，使用`*`来修改指向的变量值时，该引用本身不需要为`mut`

### 错误处理

1. 当`panic!`宏被触发时，程序会打印错误信息，回溯并清理程序栈帧，然后退出程序。如果需要注重程序大小，减少unwind操作而将对应的处理交给os来清理和释放的话，可以在`Cargo.toml`文件中添加：
   ```
   [profile.release]
   panic = 'abort'
   ```
   来显式告诉编译器编译`panic!`时的处理方式
2. `panic!`返回的是`never type`，通俗而言，返回这种类型的函数一般表示该函数*不会返回*，因此其可以用在`match语句`中和其他的正常返回值进行并列。比如如下的代码
   ```rust
   let f = File::open("test.txt");
   let f = match f{
      Ok(file) => file,
      Err(err) => panic!("Failed to open test.txt with {}", err),
   };
   ```
   中，`panic!`因为返回never type，其可以和正常返回的`struct File`的Ok情况并列，这里的`panic!`如果换成`println!`则会编译失败。

   类似的还有`return语句`，如：
   ```rust
   fn read_username_from_file() -> Result<String, io::Error> { // 这里应该是表示可以返回实例化类型为String或io:Error的Result类型
      let f = File::open("hello.txt");
      let mut f = match f {
         Ok(file) => file,
         Err(e) => return Err(e), // 可以和struct File并列
      };
      let mut s = String::new();
      match f.read_to_string(&mut s) {
         Ok(_) => Ok(s),
         Err(e) => Err(e),
      }
   }
   ```
   关于`never`原语的更多信息可参见[官方文档](https://doc.rust-lang.org/nightly/std/primitive.never.html)
3. 使用`use std::io::ErrorKind`来对错误类型进行单独判断和进一步处理，如`ErrorKind::NotFound`
4. `unwrap`和`expect`可以快捷处理执行中遇到的错误，`expect`可以自定义panic时输出的信息
5. 使用`?`来更进一步替代`unwrap`的功能，`?`本身的返回值可以接着用，接着`?`。与`unwrap`不同的是，`?`要求调用其的caller本身需要有`Result<T,E>`、`Option<T>`等类型的返回值才可以编译通过。