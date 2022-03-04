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

<https://skyao.io/learning-rust/std.html>有自己的学习笔记、std库和core库的笔记

<https://github.com/sunface/rust-course> 一个开源的中文书目

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
         Err(_) => println!("Failed to parse"),  
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
8. Rust支持使用`as`关键字来进行有限的类型转换

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
2. struct初始化的时候，特定的成员参数必须使用显式指定/同名指定的方式来命名，不可顺序默认赋值，如：
   ```rust
   struct Config{
      query: String,
      filename: String,
   }
   fn main(){
      let config1 = Config{
         query: String::from("config1query"),
         filename: String::from("config1filename"),
      };
      let query = String::from("config2query");
      let filename = String::from("config2filename");
      let config2 = Config{query, filename};
      // 下面的方法不可以，必须显式指定
      let x = String::from("config3query");
      let y = String::from("config3filename");
      let config3 = Config{x, y};
   }
   ```
3. 可以类似python的命名传参的方式，特定改变传参顺序，如：
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
6. 当传入的参数为`&mut self`而非`&self`时，定义的函数称为类的静态成员函数（`&self`则对应为类的成员函数）
7. 不使用`&self`作为传入参数的`Associated Functions`不能称为struct的方法，因此不能用`.`来访问和调用，只能通过`::`来访问和调用，如：
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
   还有常见的构造函数`::new()`也是属于Associated Functions，*Associated Functions可能采用静态生成的方式，非Associated的采用动态函数表的调用方式*

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
3. 通过在需要向module外暴露使用的数据、函数前添加`pub`关键字来make public，private函数可以被子mod使用
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
6. `use`同样支持被再次`pub`来再暴露给其他module使用
7. 对于外部的crates，通过在`Cargo.toml`声明后cargo会在编译前期自动下载相关依赖
8. `use`支持同路径下的合并：
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
9.  `use`中的`*`则是指：将对应module下的所有public对象导入
10. 为了将不同的module能够分散到不同的文件当中去，需要按2.中的树形建立对应的文件夹以及相应的`module.rs`文件，在上层文件中还需要单独写一行`mod xxx_module;`来告诉编译器在另一个同名的`.rs`文件中去加载module的内容

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

### 模版、通用数据类型、特征和生命周期

1. 和C++中类似，Rust中也支持根据不同数据类型来执行相同操作的模版函数定义，如下列代码：
   ```rust
   fn largest<T>(list: &[T]) -> T {
      let mut largest = list[0];
      for &item in list {
         if item > largest {
               largest = item;
         }
      }
      largest
   }
   fn main() {
      let number_list = vec![34, 50, 25, 100, 65];
      let result = largest(&number_list);
      println!("The largest number is {}", result);
      let char_list = vec!['y', 'm', 'a', 'q'];
      let result = largest(&char_list);
      println!("The largest char is {}", result);
   }
   ```
   这里的`fn largest<T>(list: &[T]) -> T`表示函数`largest`在类型`T`上定义为模版函数，这个函数有一个类型为`&[T]`的参数`list`，返回类型为`T`的值。但编译时会报错，因为Rust认为不是所有的类型T都满足`if item > largest`中的大小比较操作，因此应将largest的定义为：
   ```rust
   fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T 
   ```
   但Rust还会接着报错，因为`largest = list[0];`的操作需要类型T满足可以深拷贝（或无深浅拷贝）的情况下才可以进行
2. 在有了模版的概念之后，可以引出`Option`和`Result`的定义，如下：
   ```rust
   enum Option<T> {
      Some(T),
      None,
   }
   enum Result<T, E> {
      Ok(T),
      Err(E),
   }
   ```
3. Rust的编译器中包含一个名为`borrow checker`的检测系统，该系统通过检查每个变量的生命周期，来检查所有引用的合法性。

#### Traits
1. traits的目的主要告诉编译器，一种类型拥有并能够和其他的数据类型共享数据，这种共享的过程可以通过traits来进行抽象化定义。和Java中的Interface有些类似。Rust中trait是非常重要的概念，它承担了类似C++中通过纯虚类表达接口的意图。Rust中强调组合优先继承的思想，不支持struct级的继承，但支持trait的接口继承，这和Java等编程语言一样。
2. traits支持重载。
3. traits本身也可以作为变量类型放在函数定义当中，这样做的时候说明该函数接受所有impl了该traits的类型的参数
4. 对于有多个参数需要使用traits通用定义变量类型时，可以使用如下的简写形式：
   ```rust
   pub fn notify(item1: &impl Summary, item2: &impl Summary) {
   ```
   ```rust
   pub fn notify<T: Summary>(item1: &T, item2: &T) {
   ```
   类似的，如果还是不同的traits：
   ```rust
   pub fn notify(item: &(impl Summary + Display)) {
   ```
   ```rust
   pub fn notify<T: Summary + Display>(item: &T) {
   ```
   甚至更复杂地，通过`where`关键字，有：
   ```rust
   fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
   ```
   ```rust
   fn some_function<T, U>(t: &T, u: &U) -> i32
      where T: Display + Clone,
            U: Clone + Debug
   {
   ```
5. 类似的，返回值也可以为traits所限定的返回类型

#### Lifetime Annotation Syntax

为了进一步明确引用的生命周期，**生命周期标注（Lifetime Annotation Syntax）**被提出，首先看如下的代码：
```rust
fn longest(x: &str, y: &str) -> &str {
   if x.len() > y.len() {
      x
   } else {
      y
   }
}
```
以上的函数在编译时，由于Rust判断该函数的返回值为一个引用值，但根据函数签名，Rust无法判断这个引用值究竟是来自x还是y的引用，因此我们需要使用特定的语法对返回值的生命周期进行声明：
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
   if x.len() > y.len() {
      x
   } else {
      y
   }
}
```
可以看到，通过在声明变量类型中加入特定的命名标记（以`'`开头，且名称通常很短，位置在`&`和类型之间），编译器现在知道了传入的x, y参数有相同的生命周期，且返回值的生命周期和x, y保持一致。

那么接下来，将这个函数进行实际应用：
```rust
fn main() {
   let string1 = String::from("long string is long");
   let result;
   {
      let string2 = String::from("xyz");
      result = longest(string1.as_str(), string2.as_str());
   }
   println!("The longest string is {}", result);
}
```
可以看到编译仍然是不通过的，回想对longest的定义，longest的返回值应该和传入参数有相同的lifetime，而这里的string2的生命周期结束后，其返回值result仍然在被使用，因此编译器发现了生命周期的不一致并抛出了编译错误，因此将代码修改成如下形式后即可编译通过：
```rust
fn main() {
   let string1 = String::from("long string is long");
   let result; // 这里不需要mut，因为此时并未初始化具体的值
   let string2 = String::from("xyz");
   {
      result = longest(string1.as_str(), string2.as_str());
   }
   println!("The longest string is {}", result);
}
```
使用类似如下的形式来声明不同生命周期模版：
```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str 
```
当去掉`'`时，就回到了之前的模版类型定义，可以将二者混合起来，如：
```rust
use std::fmt::Display;
fn longest_with_an_announcement<'a, T>(
   x: &'a str,
   y: &'a str,
   ann: T,
) -> &'a str
where
   T: Display,
{
   println!("Announcement! {}", ann);
   if x.len() > y.len() {
      x
   } else {
      y
   }
}
```

需要提到的一种特殊生命周期声明是`'static`声明，该声明表示对应的变量的生命周期伴随整个程序，比如常量字符串，常量字符串的定义通常默认为
```rust
let s: & 'static str = "Hello world!";
```
这样的形式，该字符串实际上是直接写到了静态数据段上。

> 在使用`'static`进行声明时，需要注意该数据是否真正需要保持整个程序运行过程中都可以存活。尤其是当你准备创建dangling reference或不匹配的lifetimes时，虽然Rust编译器可能会提示你通过添加`'static`来解决这一问题，但你实际上应该是解决实际的程序逻辑错误而非直接简单粗暴添加`'static`来解决。

### 自动化测试
这一部分之前接触的语言中没有怎么了解过。Rust的自动化测试通过`cargo test`来完成，其会采取和build不同的方式来编译src文件，主要区别在于，其仅会拿需要测试的代码来进行编译（来节省时间），同时其在生成过程中*应该*也和正常build不同，会添加cfg记录的一些额外代码。

被用来测试的入口函数本身不能带任何参数

1. 自动化测试针对`cargo new xxx --lib`的项目进行自动化测试，对于需要测试的函数，需要在其前面添加`#[test]`来标注，再通过`cargo test [--release]`来进行对特定函数的测试
2. 使用`assert!(cond)`、`assert!(cond, format, ...)`、`assert_eq!(state1, state2)`等语句来在关键位置进行判断
3. 需要注意的是，在`src/lib.rs`中写代码时，如果将`struct`、`impl`等定义写在了`mod`的外面，在`mod`内使用时需要先添加一行`use super::*`才可以使用
4. 通过在`#[test]`后加一行`#[should_panic]`来指定模块需要panic产生的测试，后加`#[ignore]`来声明暂时不测试
5. cargo test可以并发进行，形式如`cargo test --release -- --test-threads=8`，类型的参数还有`--show-output`：打印输出、`--ignored`：专门测试标注为`#[ignore]`的函数
6. `cargo test func_name`可以针对某个函数进行测试，或者是针对包括`func_name`的所有函数进行测试

### 函数闭包、生成器
1. 闭包概念类似python中的lambda，Rust支持对闭包的参数、返回值类型的自动推断
2. 对于可迭代的数据类型（如vector），通过调用其`.iter()`方法可以得到类似python中的generator
3. iterator可以再调用`.map()`来依次对每个元素进行计算，使用`.collect()`对计算的结果进行收集
   ```rust
   let v1: Vec<i32> = vec![1, 2, 3];
   let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
   assert_eq!(v2, vec![2, 3, 4]);
   ```
4. 使用`.filter()`来对生成器结果进行筛选：
   ```rust
   shoes.into_iter().filter(|s| s.size == shoe_size).collect()
   ```
5. 生成器还包括如下的一些方法：
   ```rust
   fn using_other_iterator_trait_methods() {
      let sum: u32 = Counter::new()
         .zip(Counter::new().skip(1))
         .map(|(a, b)| a * b)
         .filter(|x| x % 3 == 0)
         .sum();
      assert_eq!(18, sum);
   }
   ```

### 关于Cargo和crates.io
1. 调整优化等级
   ```
   [profile.dev]
   opt-level = 0

   [profile.release]
   opt-level = 3
   ```
2. 通过在成员前使用`\\\`来写针对其的文档，可以使用markdown语法，将生成HTML格式的文档并使用`cargo doc`来生成，添加`--open`参数来查看文档，类似的还有`\\!`
3. *关于如何将项目发布到crates.io暂时没看*

### 智能指针
1. 在Rust中，引用实际上也是一种指针，但除了引用数据之外没有其他功能；智能指针是一种除了能像指针一样引用数据之外，还能有其他元数据和功能的一种数据结构的统称。和引用只是“借用”数据不同，智能指针实际上拥有其指向数据的所有权。
2. 实际上`String`、`Vec<T>`这些也是智能指针的一种，因为他们拥有一部分内存空间并允许你通过他们来控制这些内存空间上的数据，同时他们还有一些其他的数据（如长度、encoding格式等）来保证其数据特性。
3. 智能指针大部分情况下使用`struct`来实现，但和普通struct不同的是，智能指针必须单独实现`Deref`和`Drop`两种方法：
   - Deref：一种模版方法，允许该智能指针结构体能够通过引用或智能指针本身来实现像引用一样的数据访问功能。对于没有定义该方法的智能指针，不能使用`*`来访问其数据，对于定义了该方法的智能指针，`*p`实际等价于`*(p.deref())`，如下面的代码：
   ```rust
   use std::ops::Deref;
   // 为智能指针MyBox定义Deref方法
   impl<T> Deref for MyBox<T> {
      type Target = T;

      fn deref(&self) -> &T {
         &self.0
      }
   }
   // 定义智能指针MyBox
   struct MyBox<T>(T);
   impl<T> MyBox<T> {
      fn new(x: T) -> MyBox<T> {
         MyBox(x)
      }
   }
   fn hello(name: &str) {
      println!("Hello, {}!", name);
   }
   fn main() {
      let m = MyBox::new(String::from("Rust"));
      hello(&m);
   }
   ```
   上面的代码是可以正常编译并运行的，m在这里是一个`MyBox<String>`类型，调用`hello(&m)`时，首先将变成`hello(&(m.deref()))`，m.deref将`MyBox<String>`变成`String`类型。同时，`String`本身也是一个智能指针，因此其deref方法提供了将String类型数据转变为内含的字符串切片类型数据的能力，因此`&String`类型最终得以转为`&str`成为符合`fn hello`调用的参数类型。
   - Drop：一种模版方法，当对应的智能指针变量不再使用时调用（相当于析构），Drop方法不能显式调用。可以通过std库中的Drop方法来提前清理释放inscope的变量内存。定义Drop方法类似Deref
4. 许多库会使用其自定义的智能指针，在这里主要介绍标准库中最常见的三种智能指针：
   - `Box<T>`：用来在堆上分配数据，实际上最终是用仅包含一个元素的`tuple struct`来实现的
   - `Rc<T>`：一种通过引用计数来实现同一个数据有多个拥有着的智能指针，如创建链表时，指向下一个节点的指针需要为`Rc`而非`Box`
   - `Ref<T>`以及`RefMut<T>`：需要通过`RefCell<T>`来访问，一种在运行时而非编译时应用借用策略的智能指针
5. `interior mutability`：一种immutable类型，但通过其的一些API可以修改其内部的一些值
6. 使用`Box::new(value)`来为特定的值分配堆上空间
7. 使用`Rc::new(value)`来为特定的值分配计数智能指针，通过`Rc::clone(&rccounter)`来创建新指针并增加访问计数，通过`Rc::strong_count(&rccounter)`来获取访问计数的数字
8. 关于`deref`的mut继承问题，Rust进行了如下限定：
   - From `&T` to `&U` when `T: Deref<Target=U>`
   - From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
   - From `&mut T` to `&U` when `T: Deref<Target=U>`
   which means未定义defrefmut方法时，得到的数据都是immutable的
9. *有关`Ref`/`RefMut`的内容暂时没看*，相当于是把借用策略交给编程者来管理
10. *有关Reference Cycle的内容暂时没看*，大致是说当使用`Rc<T>`等智能指针当形成循环依赖时，可能造成的内存破坏问题

### 并发控制
1. 并发分为两类：*Concurrent programming*指程序的各部分独立运行；*parallel programming*则是指程序的各部分能同时运行。有关这两者的问题通常统称为并发(Concurrency)问题。
2. 随着Rust的发展，开发人员发现其设计中的*ownership*和*type systems*是安全管理内存和并发控制的关键所在，这一特性也被称为Rust的*fearless concurrency*
3. 使用`thread::spawn(func)`来创建线程，调用返回值的`.join().unwrap()`方法来等待子线程完成任务
4. 如果子线程需要使用主线程中的变量，由于Rust无法知道新生成的线程会运行多久，因此其无法判断其借用的值是否会一直有效（比如变量可能会drop掉，或者因为在主线程中out of scope而回收掉，等等原因），从而导致编译检查失败。为了解决这个问题，在`spawn(func)`前添加move关键字（`spawn(move func)`）来告诉Rust：对于在func中使用到的所有变量，子线程获得其所有权（也就是主线程、其他子线程都不再拥有其所有权）
5. 更通用的线程间通信手段是使用channel来完成，一种常用的channel为*mpsc(multiple producer single customer)*，一种FIFO队列通信原语
   1. mpsc不随sender的结束而关闭。子线程中被发送的数据将被转移所有权（即在子线程之后的代码中不能再被使用）
   2. 对recv()得到的数据类型通过send处的数据传入类型自动推断，async channel的buffer size无限大
   3. 如果只发送一个数据，接收端可以调用`recv()`方法来获取数据；如果通过多次调用`send()`发送了多个数据，可以将rx当作iterator来遍历channel中接收到的所有数据（而非去调用`rx.recv()`来获取
   4. 当有多个producer时，不能使用同一个`tx`，应调用`tx.clone()`后传给其他的producer
6. 另一种常用的多线程通信的方式是使用共享内存，虽然在Go的设计哲学中有*“do not communicate by sharing memory.”*，但借助*ownership*和*type systems*的帮助，在Rust中还是提供了互斥锁（mutex，mutual exclusion）来对其进行管理，并且能帮助多个线程对共享内存进行正确地操作
   1. 由于Mutex常需要在多个线程中使用，而`Mutex<T>`本身并没有实现Copy方法，导致不能通过move关键字或clone等方式来解决所有权的问题。一种想法是使用上一节中提到的`Rc<T>`来包裹使用，但由于move关键字中包含的Send()方法不能针对`Rc<T>`类型使用，因此这种方案行不通。基于这样的想法，可以使用原子操作的Rc，即`Arc<T>`类型来完成此问题
   2. **Similarities Between `RefCell<T>/Rc<T>` and `Mutex<T>/Arc<T>`** ：与`Rc<T>`循环引用可能造成内存泄漏相比，`Mutex<T>`在实际运行时可能会造成死锁，可尝试通过标准库中的`MutexGuard`、以及一些各个语言中通用的解决手段来解决死锁问题。
7. 实际上，Rust语言中并没有特别多的并发特性，上述的绝大多数都是来自于标准库的实现（而非语言自身特性）。然而，Rust真正的并发特性来自于`std::marker`中的`Sync`和`Send`特性：
   - Send特性：允许不同线程之间对数据所有权的交接。几乎所有的类型都支持Send特性，但之前所述的`Rc<T>`（因为不同线程对计数器的修改如果不是原子操作，则可能会引入问题），以及裸指针，等类型不行。
   - Sync特性：允许来自不同线程对数据的访问操作。换句话说：
      > any type `T` is `Sync` if `&T` (an immutable reference to `T`) is `Send`

### 面向对象编程
1. Rust本身的一些特性其实不是面向对象的设计，但通过Rust这门语言可以完成面向对象的功能。
2. Rust和一些面向对象编程的性质间的关系如下：
   1. 可以通过pub/无pub来实现public/private控制，完成封装，即将实现细节不暴露给使用者
   2. Rust从设计上而言，并不是一个强调通过继承来实现代码复用的语言。但出于使用继承的目的，Rust有对应的解决方式：
      1. 如果是出于对父类的代码复用（即继承）的目的，那么可以通过Rust的traits机制来实现类似的效果，如：
         ```rust
         // 通过traits定义类似Java中的interface接口
         pub trait Summary {
            fn summarize_author(&self) -> String;
            fn summarize(&self) -> String {
               format!("(Read more from {}...)", self.summarize_author())
            }
         }
         pub struct Tweet {
            pub username: String,
            pub content: String,
            pub reply: bool,
            pub retweet: bool,
         }
         impl Summary for Tweet {
            fn summarize_author(&self) -> String {
               format!("@{}", self.username)
            }
         }
         ```
         上述代码中，任意impl了Summary的类型就都有了默认的`summarize_author`和`summarize`方法。而且被impl的类型还可以重载此方法。
      2. 如果是出于多态的目的，Rust其实也可以用`dyn traits`来实现。对普遍多态的反对意见主要包括：
         1. 子类随着多态往往可能包含太多定义在基类，但子类完全不需要用到的方法
         2. 上述的基类方法应用在子类时可能会造成问题
3. *`dyn traits`部分暂时没看，<https://zhuanlan.zhihu.com/p/109990547>中也有相关介绍*

### 其他一些高级用法

#### unsafe
为了解决Rust静态分析的保守性所带来的扩展能力受限特点，unsafe关键字用来提供部分用来通过编译检查的更灵活特性。

通过在特定的操作前后使用`unsafe{}`进行包裹来说明unsafe部分。同时，可以直接用unsafe来修饰fn、impl，但调用fn、impl的代码仍需要用unsafe包裹

unsafe提供的superpower包括：
1. 对raw pointer的解引用
2. 调用unsafe function/method
3. 读写mutable的静态变量
4. 使用unsafe的类型模版
5. 读写union成员

which means unsafe并不会关掉包括引用检查在内的一些其他检查

#### Advanced traits, types, functions, closures和macros

### 其他一些有意思的地方
1. Rust在接受命令行参数的时候，通常使用`std::env::args`来处理，但如果此时给出的命令行参数中包含非法的unicode字符，使用`::args`会造成程序panic，更稳妥的方式是使用`::args_os`来进行处理，但这也通常可能导致在不同OS上出现不同表示的情况，处理起来也更麻烦
2. rustbook中的*An I/O Project: Building a Command Line Program*一章非常不错（以及*Iteratos and Closure*章节最后对代码的优化）！建议前面的内容都差不多熟悉后跟着这一章一起做
3. 关于Iterator和循环那种运行效率更高，Rust官方做的测试是Iterator略快于循环。但官方关于二者的效率差异，有如下的解释：
   > The point is this: iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s zero-cost abstractions, by which we mean using the abstraction imposes no additional runtime overhead. 
   
   关于zero-cost abstraction，有：

   > This is analogous to how Bjarne Stroustrup, the original designer and implementor of C++, defines zero-overhead in “Foundations of C++” (2012): In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.
4. Rust使用libloading动态加载动态链接库：<https://docs.rs/libloading/latest/libloading/>，并使用`extern "C"`来使用C的方式来调用外部函数，调用的代码通常需要使用`unsafe`来包裹
5. 链表的创建：参见`use crate::List::{Cons, Nil};`的相关内容