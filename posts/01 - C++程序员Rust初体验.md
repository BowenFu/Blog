# C++程序员Rust初体验——第1部分

#### Rust VS C++？

Rust相比C++有什么优势呢？有了Rust编译器，程序员可以对自己写出的代码更有自信，写代码的时候可以有更少顾虑，在键盘上恣意表达自我的思想，放飞自我。有了编译器提供的保护绳，走钢丝的你或你的程序能免于各种危险。

Rust编译器有多难搞定，大家可能都有所耳闻。传说中写rust就是一次次被编译器蹂躏。

为什么rust会设计成这样呢？为了语言的表现力和对底层的控制力。

Rust没有垃圾回收机制，所以需要owner语义。Rust不像纯函数编程语言如Haskell那样所有数据都是immutable，所以需要区分mut与否。要想多线程编程中没有race conditions，就要避免share一个mutable变量，这又引入了另一层的检查。

C++程序员往往需要学习大量的best practices才能写出靠谱点的代码，而写rust代码make compiler happy就可以了。

作为C++程序员，学习Rust也可以让我们熟悉更多best practices，写出更安全的代码。

#### Rust sample

让我们来看一段rust代码

```rust
use std::io;
fn main() {
    println!("Guess the number!");
    println!("Please input your guess.");
    let mut guess = String::new();
    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");
    println!("You guessed: {}", guess);
}
```

整体来说，和C++语法很接近。

`use std::io`相当于`using std::xxx`。

`fn`用于定义函数。同C++一样，Rust程序的入口也是`main`函数。

接着两行向屏幕输出了两句话。C++中可以用`printf`或是`std::cout`来表示。

```rust
let mut guess = String::new();
```

`Type::new()`类似于C++的构造函数，不过rust并不像C++一样具有"特殊"函数，这里的`new()`你要是愿意也可以命名为`construct()`或是别的你喜欢的名字。不过虽然没有特殊含义，`new()`是属于某一`trait`的，你需要遵守这一约定。该函数属于类，与任何实例没有关系，类似于C++的static成员函数。

Rust中变量默认是immutable的，这是一个很好的设计，需要mutable的时候需要显式声明`mut`。C++为了向C进行兼容，只能添加一个`const`，默认的是non-const的。C++最佳实践一般建议大家对变量都加上`const`，这样更便于编译器优化，也可以大大减少误修改了其他变量。同时可以规避不少多线程编程中的竞速问题，因为相当于你需要显式地声明哪些变量需要修改，这个过程提醒了你自己，我是在多个线程间共享一个mutable变量吗？rust设计时丢掉了这个包袱，直接就设计成了默认const。

rust声明为`mut`并不表示后面所有使用该变量处都可以直接修改。修改的地方你需要再次用`mut`声明你的引用是mutable的。

这段代码等效的C++版本如下（注意其中引入的guessRef和guessCref，也可以使用C++标准库中的std::ref/std::cref来模拟。）

```c++
#include <iostream>
#include <string>
#include <cassert>

int main()
{
    std::cout << "Guess the number!" << std::endl;
    std::cout << "Please input your guess." << std::endl;
    auto guess = std::string{};
    auto& guessRef = guess;
    auto const& guessCref = guess;
    
    std::cin >> guessRef;
    assert(!std::cin.fail());

    std::cout << "You guessed: " << guessCref << std::endl;
    return 0;
}
```



注意到rust代码中有一个链式调用,`io::stdin().read_line(...).expect(...);`可以理解成`read_line`返回了一个object，和C中常用的错误码差不多，但是它可以调用函数`expect(...)`。OK表示执行成功，否则表示遇到了一个错误。遇到错误的时候`expect(...)`会打印出提供的错误信息，然后终止程序。我们后续会介绍，这里的返回值类型实际上是枚举类型，或者称为ADT（代数数据类型），Haskell或其他函数式编程语言中可以看到相关的概念。C++17引入的`std::variant`可以认为是类似的，`std::optional`可以认为是`std::variant`的特例。

 `println!("You guessed: {}", guess);`这种格式化输出大家应该在python等语言中提前见过。C中的printf也类似，不过不是类型安全的。C++20引入新的format库，可以提供类似的格式化操作。不过需要注意的是，println!调用的是宏而不是函数。!表示调用宏。
 
 -----
 
 上面的C++代码虽然可以跑起来，但是其实并不完全等效于rust版本，有一个小小的差异？
`println!("You guessed: {}", guess);`
这句如果是`println!("You guessed: {}", &guess);`，那就和C++版本一致了。
现在的版本等效于什么呢？

`std::cout << "You guessed: " << std::string(std::move(guess)) << std::endl;`
guess传值的话默认会transfer，这其实很像C++11中弃用的std::auto_ptr，拷贝构造中原object的资源会被偷走。

在C++11中就等于进行了移动构造。这句执行之后，guess已经不是有效值了。
C++最佳实践中有一条（特别是泛型编程中），对右值引用参数或转发引用参数的最后一次调用时，参数传递同样使用右值引用或转发引用，避免可能的拷贝。
当然，这一条在这里并不能带来什么影响。

等效的C++修正版本应该为：
```c++
#include <iostream>
#include <string>
#include <cassert>

int main()
{
    std::cout << "Guess the number!" << std::endl;
    std::cout << "Please input your guess." << std::endl;
    auto guess = std::string{};
    auto& guessRef = guess;
    
    std::cin >> guessRef;
    assert(!std::cin.fail());

    std::cout << "You guessed: " << std::string(std::move(guess)) << std::endl;
    return 0;
}
```
