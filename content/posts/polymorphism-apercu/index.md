+++
date = '2026-05-29T19:54:24+08:00'
draft = false
title = '浅谈多态'
slug = 'polymorphism-apercu'
author = 'qwertyuiop'
tags = ["编程语言"]
keywords = ["多态", "polymorphism"]
readingTime = true
showFullContent = false
hideComments = false
+++

## 前言

多态（Polymorphism）是编程语言中的一个重要概念。本文将简要地描述多态的概念，以及一些常见的具体实现方式、它们的设计思路和优缺点。当然，因为是“浅谈”，所以不会涉及过于深入或理论的细节（~~比如编程语言理论中的 [System F](https://en.wikipedia.org/wiki/System_F)，其实我自己都忘了~~）。

~~在昨天的程设课上，老师和 ppt 上说 “C++ 从未公布其多态实现方式”，我觉得有失偏颇，于是决定写这么一篇文章。Credit 一下 fy 同学，让我下定决心写这篇文章（~~

## 多态是什么？

要说道多态是什么，首先我们要理解什么是接口。我们可以很抽象地理解它，认为它是“**对程序员可见的一系列契约（Contract[^1]）**”。这个定义是我的私货，很泛、很抽象，但是概括了各种编程范式中接口的共性。

### 接口？契约？

为了更好地理解接口，我们拿一个老生常谈的例子来说明：2D 图元。

2D 图元是一个抽象的概念，它可以是一个点、线段、矩形、圆、三角形等等。我们可以尝试用一些简短的语句来描述它们的共性（这其实是我们对它们的主观的抽象，要牢记，实际情况可能并没有这么简单，下面这个例子也存在有抽象失真和概念漂移的问题，不过我们先不关心这个问题）：

- 有一个位置（坐标）。
- 可以被绘制出来。
- 可以被移动。
- 可以被旋转。
- 可以被缩放。
- 可以被删除。

上面这些语句，便描述了一系列的契约，我们作为使用这些图元的程序员，就可以通过这些契约来使用它们，而**不需要关心它们的具体实现细节**，比方说，我们不用关心一个矩形和一个圆在绘制时的具体实现方式，也不需要在绘制时关心这个图元到底是什么。

我们用 C++ 来描述一下这些契约：

```cpp
class Shape2D {
public:
    virtual void draw() = 0; // 绘制图元
    virtual void move(double dx, double dy) = 0; // 移动图元
    virtual void rotate(double angle) = 0; // 旋转图元
    virtual void scale(double factor) = 0; // 缩放图元
    virtual void remove() = 0; // 删除图元
};
```

使用上面这个定义，我们就可以直接使用它来操作各种不同类型的图元，而**不需要关心它们的具体实现细节**：

```cpp
void foo(Shape2D* shape) {
    shape->draw();
    shape->move(10, 20);
    shape->rotate(45);
    shape->scale(2);
    shape->remove();
}
```

这些 C++ 代码只是一个例子，实际上我们并不一定需要这种面向对象的方式来实现。我们也能用类似重载的方式来实现契约：

```cpp
void draw(Point p) { /* 绘制点 */ }
void draw(Line l) { /* 绘制线段 */ }
void draw(Rectangle r) { /* 绘制矩形 */ }
void draw(Circle c) { /* 绘制圆 */ }
void draw(Triangle t) { /* 绘制三角形 */ }

// 使用时......
draw(primitive); // primitive 可以是 Point、Line、Rectangle、Circle、Triangle 中的任意一个
```

在上面的例子中，我们规定，将 2D 图元作为参数传递给 `draw` 函数，就可以绘制出这个图元。我们也可以规定，将 2D 图元作为参数传递给 `move` 函数，就可以移动这个图元，以此类推。（看出了“契约”的感觉吗，可以在这里好好回想一下）

这些契约，我们就可以称之为“**接口**”。

### 行为的分化

上面我们提到的接口，描述了一个抽象的概念（2D 图元）的共性，但是它们的具体实现方式可能会有很大的差异。比如说，绘制一个矩形和绘制一个圆的实现方式就完全不同，移动一个点和移动一个线段的实现方式也完全不同。这就是多态的核心：**行为的分化**。同一个接口可以有多种不同的实现方式，而我们作为调用者、使用者时，并不需要关心这些实现方式的细节，我们只需要关心接口本身，就可以了，编译器或者运行时会自动选择对应的实现，表现出不同的行为。

回到我们的 2D 图元例子：`foo(Shape2D* shape)` 函数只认 `Shape2D` 这个接口，但当你传入一个圆或一个矩形的指针时，调用 `shape->draw()` 会自动画出圆形或矩形。
即使是用重载的方式，调用 `draw(primitive)` 时，编译器会根据 `primitive` 的具体类型（`Point`、`Line` 等）自动选择正确的 `draw` 函数。这也是一种多态，只不过它的行为在编译期就确定了，我们称之为静态[^2]多态（或编译期多态）。

总的来说，接口定义了“**能做什么**”，多态实现了“**同样的调用，不同的做法**”。接下来我们就梳理一些常见的多态实现方式。

## 特设多态（Ad hoc Polymorphism）

我们先从看上去最简单的 Ad hoc 多态说起。

> Ad hoc 是一个拉丁文常用短语。这个短语的意思是“特设的、特定目的的（地）、即席的、临时的、将就的、专案的”。这个短语通常用来形容一些特殊的、不能用于其它方面的，为一个特定的问题、任务而专门设定的解决方案。
> —— [Wikipedia](https://zh.wikipedia.org/wiki/Ad_hoc)

特设多态指的是：同一个函数名（或运算符）针对不同的具体类型，执行完全不同的代码。

这种多态是利用函数重载或者运算符重载（它们本质是一致的）来实现的，比如[前文](#接口契约)中的 `draw(primitive)` 就是一个典型的 Ad hoc 多态的例子。我们通过函数重载来实现了对不同类型的图元的绘制，编译器在编译时，通过[重载决议](https://en.cppreference.com/cpp/language/overload_resolution)找到正确的函数，静态地决定调用哪个函数。

假设我们想实现一个“打印”功能，既能打印整数，也能打印字符串，还能打印布尔值。在 C++ 中，我们可以用重载来实现：

```cpp
void print(int x) { std::cout << "int: " << x; }
void print(const std::string& s) { std::cout << "string: " << s; }
void print(bool b) { std::cout << "bool: " << (b ? "true" : "false"); }

// 使用时......
print(42); // 调用 print(int)
print("Hello"); // 调用 print(const std::string&)
print(true); // 调用 print(bool)
```

再看一个运算符重载的例子（C++ 中的复数加法）：

```cpp
struct Complex { double re, im; };
Complex operator+(const Complex& a, const Complex& b) {
    return {a.re + b.re, a.im + b.im};
}
```

现在 `c1 + c2` 会调用这个自定义的加法，而 `1 + 2` 仍然使用内置的整数加法。同一个运算符 `+` 对不同类型表现出不同的行为。

这种多态完全是编译期行为[^3]，因此它在运行时是零开销的，最终生成的代码中只是简单的直接调用对应的函数。很自然地，我们可以发现它的缺点：

- 如果我们需要支持更多的类型，就需要为每个类型写更多的重载函数，即使这些类型之间有很多相似之处，代码量会迅速增加（最终编译生成的代码也会变大[^4]）。
- 无法在运行时根据对象的类型来选择函数，所有的决策都必须在编译时做出，这限制了它的灵活性。（比如某个对象的具体类型由用户的输入决定，那么它的具体类型显然是无法在编译器推导出来的）。
- 编译器需要执行复杂的重载决议规则（如隐式转换排名、最佳匹配、参数类型优先级）。不同语言的规则各异，容易产生意外结果或二义性错误（如 C++ 中的 `f(0)` 匹配 `f(int)` 还是 `f(long)`， `f(nullptr)` 匹配指针还是整数）。
- 难以扩展已有类型的行为，因为重载函数必须在定义时就列出所有支持的类型，无法在后续添加新的类型而不修改原有代码或需要在你的模块中添加实现代码（模块边界模糊）。比如说，如果我们想为 `double` 类型添加一个新的 `print` 函数，我们需要修改 `print` 所在的代码文件，添加一个新的重载函数 `void print(double x)`，或者是在你自己的模块的代码文件中添加，这不是软件工程的最佳实践。

### 一些扩展

C++ 中有 [ADL（Argument-Dependent Lookup，参数依赖查找）](https://en.cppreference.com/cpp/language/adl)的特性，它允许编译器在进行重载决议时，不仅考虑当前作用域中的函数，还会考虑与参数类型相关的命名空间中的函数。这使得我们可以在不修改原有代码的情况下，为已有类型添加新的行为，从而在一定程度上缓解了上述的扩展性问题。
比如说，我们有一个 `Point` 类型定义在 `geometry` 命名空间中：

```cpp
namespace geometry {
    struct Point { double x, y; };
}
```

我们想为 `Point` 添加一个新的 `print` 函数，我们可以在 `geometry` 命名空间中定义这个函数：

```cpp
namespace geometry {
    void print(const Point& p) {
        std::cout << "Point(" << p.x << ", " << p.y << ")";
    }
}
```

现在，当我们调用 `print(p)` （注意不需要 `using namespace geometry` 或者 `using geometry::print`！）时，编译器会通过 ADL 查找 `geometry` 命名空间中的 `print` 函数，并正确地调用它来打印 `Point` 对象。这种方式允许我们在不修改原有代码的情况下，为已有类型添加新的行为，从而提高了代码的可扩展性和模块化。C++ 标准库中的 `std::swap` 就是一个典型的利用 ADL 来实现可扩展性的例子，类的编写者可在类的命名空间[^5]中定义它们类的 `swap` 函数，用户调用 `std::swap` 时，编译器会通过 ADL 查找并调用正确的 `swap` 实现。

来看看其他编程语言的情况，Rust 不支持函数重载，但它可以使用 trait 来实现类似的功能：

```rust
trait Printable {
    fn print(&self);
}
impl Printable for i32 {
    fn print(&self) {
        println!("int: {}", self);
    }
}
impl Printable for String {
    fn print(&self) {
        println!("string: {}", self);
    }
}
impl Printable for bool {
    fn print(&self) {
        println!("bool: {}", self);
    }
}
```

Haskell 中的 type class 也可以实现类似的功能：

```haskell
class Printable a where
    print :: a -> IO ()
instance Printable Int where
    print x = putStrLn $ "int: " ++ show x
instance Printable String where
    print s = putStrLn $ "string: " ++ s
instance Printable Bool where
    print b = putStrLn $ "bool: " ++ show b
```

## 参数多态（Parametric Polymorphism）

参数多态一般是指的是同一个函数或类型定义可以适用于多种不同的类型，而不需要为每种类型写一个单独的定义。换句话说，代码只需要写一套（加上一些特例），它就可以对大量不同类型生效。它通常通过泛型或者模板来实现。比如说，在 C++ 中，我们可以使用模板来实现一个通用的 `swap`，它可以交换任何类型的两个对象：

```cpp
template<typename T>
void swap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}
```

可以发现一个很明显的特征：**类型本身是参数化的**，我们可以对类型参数进行操作、约束、推导等。比如说，我们可以在模板中使用 [Concept](https://en.cppreference.com/cpp/language/constraints) 来约束类型参数（也即是说，对于上述例子，这套代码应该在 T 满足什么条件的情况下可用？），或者使用 [SFINAE（Substitution Failure Is Not An Error）](https://en.cppreference.com/cpp/language/sfinae)来实现条件编译。C++ 的标准库被称为 STL（Standard Template Library），它大量使用了模板来实现各种数据结构和算法，使得这些数据结构和算法可以适用于任何类型。比如说，`std::vector` 是一个动态数组，它是一个模板类，可以存储任何类型的元素，一般情况下我们无需为了存储不同类型的元素而写多个版本的 `vector`，或在我们的类型里实现一些特有的方法来让它们能够被 `vector` 存储；`std::sort` 是一个模板函数，可以对任何类型的元素进行排序，只要这些元素支持比较操作（比如说 `<` 运算符）。参数多态使得我们能够编写更通用、更灵活的代码，减少代码重复，提高代码的可重用性和可维护性。

Rust 中的泛型也可以实现类似的功能：

```rust
fn swap<T>(a: &mut T, b: &mut T) {
    std::mem::swap(a, b);
}
```

Java 中也有泛型的功能，比如 `java.util.Collections` 中的 `List` 接口就是一个泛型接口：

```java
List<String> strings = new ArrayList<>();
List<Integer> integers = new ArrayList<>();

strings.add("Hello");
integers.add(42);
```

不过，C++/Rust 和 Java 的参数多态实际上有一些显著区别。

C++ 的模板、Rust 的泛型是通过**单态化**（Monomorphization）实现的，它们的编译器会为每个使用了模板的类型、每个类型参数组合的函数模板生成一个专门的版本。而 Java 的泛型是通过[**类型擦除**](https://en.wikipedia.org/wiki/Type_erasure)（Type Erasure）[^6]实现的，在编译时会将泛型类型擦除掉，所有的泛型类型都会被替换成它们的上界（通常是 `Object`），在运行时需要进行类型检查和动态分派[^7]，也就是说，Java 的泛型在编译时并不保留类型信息。

### 单态化

单态化的实现是显然的，编译器会为每个使用了模板的类型、每个类型参数组合的函数模板生成一个专门的版本。比如说，如果我们在 C++ 中使用了 `swap<int>` 和 `swap<double>`，编译器会生成两个版本的 `swap` 函数，一个用于交换整数，另一个用于交换双精度浮点数，几乎可以等价于我们自己编写了两个函数，它们名为 `swap<int>` 和 `swap<double>`。

这种策略的优点是显而易见的：

- 零运行时开销。生成的代码就是针对具体类型的直接操作，没有任何间接调用、虚表查找或类型检查。`swap<int>` 里就是整数寄存器的交换，`swap<double>` 里就是双精度浮点数的交换。编译器可以自由地对这些专用代码进行内联、常量传播、死代码消除等优化，效果等同于手写的专用函数。
- 类型安全。模板在实例化时会进行完整的类型检查，每个具体版本的参数类型、返回值类型都精确匹配，不会发生运行时类型错误。

但单态化也有其代价：

- 代码膨胀[^4]。每个不同的类型参数组合都会生成一份独立的机器码。如果程序中使用了 `vector<int>`、`vector<double>`、`vector<string>`，编译器会生成三份几乎完全相同的 vector 成员函数代码（只是操作的数据类型大小和拷贝方式不同）。在极端情况下，比如使用大量小型类型实例化同一个模板，二进制体积可能急剧膨胀。
- 编译时间增加。编译器需要为每个实例化生成、优化、生成代码，这比只处理一份泛型代码要耗时得多。这也是为什么大型 C++ 项目的编译时间往往以小时计。
- ABI 稳定性问题。由于单态化生成的代码内嵌在调用方中，如果模板库的实现发生变化，所有使用该模板的代码都需要重新编译。这使得动态库（.dll/.so）的更新变得困难，因为无法仅替换一个二进制文件而保持接口兼容。
- 不同模板实例实际上是不同的函数或类型，这实际上限制了灵活性。比如说，如果我们想在运行时根据用户输入的类型来选择使用哪个版本的 `swap<T>`，这是不可能的，因为每个版本都是独立的函数，无法通过一个统一的接口来动态分发调用，这是静态多态的一个固有限制。

为了缓解这些问题，语言和编译器提供了若干手段：

- C++ 的 `extern template` 可以显式禁止在某个编译单元中实例化模板，转而使用其他地方已经实例化的版本，从而减少重复代码。
- 链接时优化（LTO，Link-Time Optimization）和 ICF（Identical Code Folding）组合，可以跨编译单元去重相同的模板实例化代码，部分缓解代码膨胀。
- Rust 的 `Box<dyn Trait>` 提供了将单态化的泛型转换为动态分派的选项，允许在性能与体积之间做权衡，当某个泛型函数被太多不同类型实例化时，可以用 trait 对象替代，只保留一份代码。

### 类型擦除

类型擦除出现的动机是，我们能不能把类型信息在编译时擦除，只保留一个通用的类型信息，以实现动态分派？这就是类型擦除的思路。它通过在编译时将所有的泛型类型替换成一个通用的类型（比如 Java 中的 `Object`），并利用运行时进行类型检查和动态分派来实现多态。

我们这里利用 C++ 来举例子，虽然 C++ 本身并不使用类型擦除来实现泛型，但我们可以通过一些技巧来模拟这种行为，这样我们可以更好地理解类型擦除的概念。比如说，我们先来看看 tagged union：

```cpp
enum class TypeTag { Int, String, Bool };
struct Variant {
    TypeTag tag;
    union {
        int i;
        std::string s;
        bool b;
    };
};
```

在上面的代码中，我们定义了一个 `Variant` 结构体，它包含一个 `TypeTag` 枚举来标识当前存储的类型，以及一个匿名联合体来存储不同类型的值。我们可以通过检查 `tag` 来确定当前存储的是什么类型，然后进行相应的操作：

```cpp
void print(const Variant& v) {
    switch (v.tag) {
        case TypeTag::Int:
            std::cout << "int: " << v.i;
            break;
        case TypeTag::String:
            std::cout << "string: " << v.s;
            break;
        case TypeTag::Bool:
            std::cout << "bool: " << (v.b ? "true" : "false");
            break;
    }
}
```

上述例子中，我们在编译期一般并不能知道 `Variant` 里存储的是什么类型，我们只能在运行时通过检查 `tag` 来确定类型并进行相应的处理。这就是类型擦除的一个典型例子：我们在编译时擦除了具体的类型信息，所有的类型都被替换成了一个通用的 `Variant` 类型，而在运行时我们需要进行类型检查和动态分派来实现多态。这个例子主要体现了类型擦除可以动态分派的特性，虽然它没有真正实现 Java 那样的类型擦除，但它展示了类型擦除的核心思想。

然后我们再看一个很典型的例子：`std::function`。`std::function` 是一个通用的函数包装器，它可以存储任何可调用对象（函数指针、lambda 表达式、函数对象等），并提供一个统一的接口来调用它们。`std::function` 内部使用了类型擦除的技巧来实现这个功能，它通过一个抽象基类来定义一个统一的接口，然后为每个具体的可调用对象类型定义一个派生类来实现这个接口，在运行时通过动态分派来调用正确的函数，下面是一个简化版的 `std::function` 的（伪）实现：

```cpp
template<typename R, typename... Args>
class function {
    struct CallableBase {
        virtual R invoke(Args... args) = 0;
        virtual ~CallableBase() = default;
    };
    template<typename F>
    struct Callable : CallableBase {
        F f;
        Callable(F&& func) : f(std::forward<F>(func)) {}
        R invoke(Args... args) override { return f(std::forward<Args>(args)...); }
    };
    std::unique_ptr<CallableBase> callable;
public:
    template<typename F>
    function(F&& func) : callable(std::make_unique<Callable<F>>(std::forward<F>(func))) {}
    R operator()(Args... args) {
        return callable->invoke(std::forward<Args>(args)...);
    }
};
```

可以看到，`std::function` 内部定义了一个抽象基类 `CallableBase`，它有一个纯虚函数 `invoke` 来定义一个统一的接口；然后定义了一个模板派生类 `Callable<F>` 来实现这个接口，它存储了一个可调用对象 `f`，并在 `invoke` 中调用它；最后，`std::function` 通过一个 `std::unique_ptr<CallableBase>` 来存储一个指向抽象基类的指针，在构造函数中根据传入的可调用对象类型创建对应的派生类实例，并在调用时通过动态分派来调用正确的函数。

仔细分析可以得到，对于每种函数签名（`R(Args...)`），`std::function` 只需要生成一份代码（即 `function<R, Args...>` 的实现），而不需要为每个具体的可调用对象类型（每个 lambda 表达式拥有的类型、每个拥有 `operator()` 的类型、函数指针等等）生成一个专门的版本（虽然 `Callable<F>` 仍然需要单态化，但是对于 `std::function` 中可能存在的其他大量方法（构造函数等等），`Callable<F>` 的代码量是很少的），这样就缓解了代码膨胀的问题，但同时也可能会带来了性能上的损失，因为每次调用 `operator()` 都可能需要进行一次虚函数调用（即动态分派），而不是直接调用一个专门的函数。

单态化和类型擦除的选择，本质上是一个权衡：单态化可以带来更好的性能，因为编译器可以针对每个类型参数组合生成优化的代码，但它可能会导致代码膨胀[^8]，因为每个类型参数组合都会生成一个新的版本；而类型擦除可以减少代码膨胀，因为所有的泛型类型都被替换成了同一个类型，但它可能会带来性能上的损失，因为需要在运行时进行类型检查和动态分派[^7]。

### 类型约束

参数多态的一个重要方面是类型约束。我们在定义一个泛型函数或类型时，通常希望对类型参数进行一些约束（这也是一种契约！），以确保它们满足某些条件，从而使得我们的代码能够正确地工作。也可以利用这些约束，对部分满足条件的类型使用性能更高的实现[^9]。比如说，在 C++ 中，我们可以使用 Concept 来约束类型参数：

```cpp
template<typename T>
concept OstreamPrintable = requires(T a) {
    { std::cout << a } -> std::same_as<std::ostream&>;
};

void print(const OstreamPrintable auto& x) {
    std::cout << "OstreamPrintable: " << x;
}

template<OstreamPrintable T>
struct Printer {
    void operator()(const T& x) {
        std::cout << "OstreamPrintable: " << x;
    }
};
```

上述代码中，我们定义了一个 Concept `OstreamPrintable`，它要求类型 `T` 必须满足能够被 `std::cout` 输出的条件（即必须支持 `operator<<`），然后我们在 `print` 函数或者 `Printer` 类型中使用这个 Concept 来约束参数类型，这样只有满足 `OstreamPrintable` 条件的类型才能调用这个函数，否则编译器会报错。

在 Concept 出现之前，实际上 C++ 并没有类型约束的功能，我们可以使用 SFINAE 来实现类似的功能：

```cpp
template<typename T>
std::enable_if_t<std::is_arithmetic_v<T>, void> print(const T& x) {
    std::cout << "Arithmetic: " << x;
}
template<typename T>
std::enable_if_t<!std::is_arithmetic_v<T>, void> print(const T& x) {
    std::cout << "Non-arithmetic: " << x;
}
```

`std::enable_if_t` 根据条件选择返回一个类型或者导致编译失败。上面的代码中，我们定义了两个 `print` 函数模板，一个用于处理算术类型（如整数、浮点数等），另一个用于处理非算术类型。通过 `std::is_arithmetic_v<T>` 来判断类型 `T` 是否是算术类型，非算术类型的 `print` 函数会在编译时实例化错误，但不会引发编译失败，只是会在重载决议中排除，从而实现了对类型参数的约束。

Rust 对于类型约束的实现则是通过 trait 来实现的：

```rust
trait Printable {
    fn print(&self);
}
fn print<T: Printable>(x: &T) {
    x.print();
}
```

在上述代码中，我们定义了一个 trait `Printable`，它要求实现这个 trait 的类型必须提供一个 `print` 方法。然后我们定义了一个泛型函数 `print`，它接受一个类型参数 `T`，并使用 `T: Printable` 来约束这个类型参数，只有实现了 `Printable` trait 的类型才能调用这个函数。

### 总结

参数多态让代码可以使用类型参数的形式抽象，只需一份逻辑（可能需要一些特化），就能服务无限多种具体类型。它是现代语言中实现容器和算法复用的基石。你在编写 `std::vector<T>` 或 `Option<T>` 时，享受的正是参数多态带来的零成本抽象（或近似零成本）的好处。

下一节，我们将讨论另一类重要的多态，子类型多态（继承 + 虚函数 / 接口），它在形式上和参数多态截然不同。

## 子类型多态（Subtype Polymorphism）

子类型多态是面向对象编程中最常见的多态形式，它通过继承和虚函数（或接口）来实现。它允许我们通过一个基类指针或引用来调用派生类的函数，从而实现运行时的动态分派。比如我们[上文的例子](#接口契约)中的 `Shape2D` 类就是一个典型的子类型多态的例子。

子类型多态可以用**继承**以及类似的机制（比如 Java 的接口，Rust 的 dyn trait）实现。

> A subtype is a datatype that is related to another datatype (the supertype) by some notion of substitutability, meaning that program elements (typically subroutines or functions), written to operate on elements of the supertype, can also operate on elements of the subtype.
> —— [Wikipedia](https://en.wikipedia.org/wiki/Subtyping)
>
> 也就是说，继承和子类型其实是正交的概念。如果 S 是 T 的子类型，那么任何需要 T 类型值的上下文，都可以安全地使用 S 类型的值（Liskov Substitution Principle）。它关注的是接口契约，而不是代码如何复用。
>
> 而继承是一种代码复用机制，它允许我们在一个类中重用另一个类的实现细节。虽然在很多面向对象的语言中，继承和子类型多态通常是一起出现的，但它们并不是必须绑定在一起的。比如说，在 Rust 中，我们可以通过 trait 来实现子类型多态，而不需要使用继承；而在 C++（公有继承）/Java 中，继承同时也能达成子类型关系；C++ 的私有继承则一般不会产生子类型关系。

这些语言的设计细节和实现细节各不相同，但它们都提供了某种机制来支持子类型多态。

先来看看 C++ 中的子类型多态：

- C++ 中的子类型多态是通过公有继承和虚函数来实现的。我们可以定义一个基类，它包含一些虚函数，然后我们可以定义一些派生类来继承这个基类，并提供这些纯虚函数的具体实现。这样，我们就可以通过一个指向基类的指针或引用来调用这些函数，而不需要关心它们的具体类型，编译器会在运行时根据对象的实际类型来选择调用哪个函数。
- C++ 支持多重继承，这意味着一个类可以同时继承多个基类，从而实现更复杂的子类型关系。不过多重继承也带来了一些问题，比如说菱形继承问题（类型层级中有多个类有共同祖先），需要使用虚继承来解决。
- C++ 没有类似接口的概念，但我们可以通过定义一个纯虚类（即所有成员函数都是纯虚函数的类）来模拟接口的行为。

Java 中的子类型多态：

- Java 中的子类型多态是通过类继承和接口来实现的。我们可以定义一个基类，它包含一些方法（可以是抽象方法），然后我们可以定义一些派生类来继承这个基类，并提供这些方法的具体实现。我们也可以定义一些接口，接口中只包含抽象方法，然后让类去实现这些接口。这样，我们就可以通过一个指向基类或接口的引用来调用这些方法，而不需要关心它们的具体类型，Java 的运行时会根据对象的实际类型来选择调用哪个方法。
- Java 不支持多重继承，但它支持一个类实现多个接口，这样也可以实现复杂的子类型关系。
- Java 的接口中只能包含抽象方法（Java 8 之后允许接口中包含默认方法和静态方法，但它们仍然不能包含数据成员），而 C++ 的纯虚类可以包含成员变量，这也是两者的一个重要区别。

Rust 中的子类型多态：

- Rust 中的子类型多态是通过 trait 来实现的。我们可以定义一个 trait，它包含一些方法，然后我们可以定义一些类型来实现这个 trait，并提供这些方法的具体实现。这样，我们就可以通过一个 trait 对象（`&dyn Trait`、`Box<dyn Trait>`）来调用这些方法，而不需要关心它们的具体类型，根据对象的实际类型来选择调用哪个方法。
- Rust 中没有继承的概念。
- Rust 的 trait 其实是对类型的一种约束，它定义了一组方法的集合，任何实现了这个 trait 的类型都必须提供这些方法的具体实现，这也是一种契约的体现。

### 使用虚函数表实现的子类型多态的实现细节

子类型多态的实现通常是通过虚函数表（vtable）来实现的，例如一般的 C++ 实现。每个类都有一个虚函数表，它是一个指向函数指针数组的指针，这个数组包含了这个类的所有虚函数的地址。运行时就会读取虚函数表中的函数指针，进行间接调用，从而实现动态分派。

我们可以拿一个很 amber 的东西举例子：[微软的 COM 组件对象模型（Component Object Model）](https://learn.microsoft.com/en-us/windows/win32/com/component-object-model--com--portal)。

你一定听说过 Direct3D 或者 DXGI。[DXGI](https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/d3d10-graphics-programming-guide-dxgi) 负责管理图形设备的底层资源以及交换链之类的东西，它定义了一系列以 `IDXGI` 前缀开头的接口。这些接口正是 COM 组件。在 COM 中，任何接口都必须直接或间接地继承自 COM 的根接口 `IUnknown`。`IUnknown` 提供了两个核心服务：一是通过 `AddRef` 和 `Release` 实现的引用计数生命周期管理；二是通过 `QueryInterface` 实现的运行时接口查询能力。

我们来看看 C++ 中 和 `IDXGIObject` 的定义（从 `dxgi.h` 中提取）：

```cpp
// 这里是伪代码
struct IUnknown {
    virtual HRESULT QueryInterface(REFIID riid, void** ppvObject) = 0;
    virtual ULONG AddRef() = 0;
    virtual ULONG Release() = 0;
    // ...
};

// DXGI 的根接口 IDXGIObject 继承自 IUnknown
// ...
MIDL_INTERFACE("aec22fb8-76f3-4639-9be0-28eb43a67a2e")
IDXGIObject : public IUnknown
{
public:
    virtual HRESULT STDMETHODCALLTYPE SetPrivateData(
        /* [annotation][in] */
        _In_  REFGUID Name,
        /* [in] */ UINT DataSize,
        /* [annotation][in] */
        _In_reads_bytes_(DataSize)  const void *pData) = 0;

    virtual HRESULT STDMETHODCALLTYPE SetPrivateDataInterface(
        /* [annotation][in] */
        _In_  REFGUID Name,
        /* [annotation][in] */
        _In_opt_  const IUnknown *pUnknown) = 0;

    virtual HRESULT STDMETHODCALLTYPE GetPrivateData(
        /* [annotation][in] */
        _In_  REFGUID Name,
        /* [annotation][out][in] */
        _Inout_  UINT *pDataSize,
        /* [annotation][out] */
        _Out_writes_bytes_(*pDataSize)  void *pData) = 0;

    virtual HRESULT STDMETHODCALLTYPE GetParent(
        /* [annotation][in] */
        _In_  REFIID riid,
        /* [annotation][retval][out] */
        _COM_Outptr_  void **ppParent) = 0;

};
```

然后我们来看看 C 语言中 `IDXGIObject` 的定义（从 `dxgi.h` 中提取）：

```c
typedef struct IDXGIObjectVtbl
{
    BEGIN_INTERFACE

    DECLSPEC_XFGVIRT(IUnknown, QueryInterface)
    HRESULT ( STDMETHODCALLTYPE *QueryInterface )(
        IDXGIObject * This,
        /* [in] */ REFIID riid,
        /* [annotation][iid_is][out] */
        _COM_Outptr_  void **ppvObject);

    DECLSPEC_XFGVIRT(IUnknown, AddRef)
    ULONG ( STDMETHODCALLTYPE *AddRef )(
        IDXGIObject * This);

    DECLSPEC_XFGVIRT(IUnknown, Release)
    ULONG ( STDMETHODCALLTYPE *Release )(
        IDXGIObject * This);

    DECLSPEC_XFGVIRT(IDXGIObject, SetPrivateData)
    HRESULT ( STDMETHODCALLTYPE *SetPrivateData )(
        IDXGIObject * This,
        /* [annotation][in] */
        _In_  REFGUID Name,
        /* [in] */ UINT DataSize,
        /* [annotation][in] */
        _In_reads_bytes_(DataSize)  const void *pData);

    DECLSPEC_XFGVIRT(IDXGIObject, SetPrivateDataInterface)
    HRESULT ( STDMETHODCALLTYPE *SetPrivateDataInterface )(
        IDXGIObject * This,
        /* [annotation][in] */
        _In_  REFGUID Name,
        /* [annotation][in] */
        _In_opt_  const IUnknown *pUnknown);

    DECLSPEC_XFGVIRT(IDXGIObject, GetPrivateData)
    HRESULT ( STDMETHODCALLTYPE *GetPrivateData )(
        IDXGIObject * This,
        /* [annotation][in] */
        _In_  REFGUID Name,
        /* [annotation][out][in] */
        _Inout_  UINT *pDataSize,
        /* [annotation][out] */
                _Out_writes_bytes_(*pDataSize)  void *pData);

    DECLSPEC_XFGVIRT(IDXGIObject, GetParent)
    HRESULT ( STDMETHODCALLTYPE *GetParent )(
        IDXGIObject * This,
        /* [annotation][in] */
        _In_  REFIID riid,
        /* [annotation][retval][out] */
        _COM_Outptr_  void **ppParent);

    END_INTERFACE
} IDXGIObjectVtbl;

interface IDXGIObject
{
    CONST_VTBL struct IDXGIObjectVtbl *lpVtbl;
};
```

C/C++ 中的定义虽然乍一看非常不一样，但它们是 [ABI](https://en.wikipedia.org/wiki/Application_binary_interface) 兼容的，换句话说，编译器生成的 C++ 类的内存布局和 C 语言中定义的结构体是完全一致的，这就是 COM 接口设计的一个重要原则：**二进制兼容性**。

```cpp
IDXGIObject *obj = ...;
IUnknown *unk = (IUnknown*)obj;

// c++
unk->QueryInterface(...);

// c
unk->lpVtbl->QueryInterface(unk, ...);

// 这两种方式是等价的
```

所有的 COM 接口都遵守一个统一的二进制内存布局规则，其定义大致如下：

每个接口在 C 头文件中被定义为仅包含一个指向虚函数表的结构体。例如从 `dxgi.h` 中提取的定义可以看到：`interface IDXGIObject { CONST_VTBL struct IDXGIObjectVtbl *lpVtbl; };`。这里 `lpVtbl` 就是这个接口的虚表指针。

而虚函数表 `Vtbl` 本身是一个结构体数组，其中每个元素都是一个函数指针。对于 `IUnknown` 来说，它的虚表中固定包含 `QueryInterface`、`AddRef` 和 `Release` 三个函数指针。

当一个接口继承自另一个接口时（例如 `IDXGIFactory` 继承自 `IDXGIObject`），子接口的虚表会在其父接口虚表的基础上，将新的函数指针追加到末尾。这种布局使得基类指针能够安全地调用基类方法，因为它始终只使用虚表的前部，而子类的实现可以识别完整的虚表。

COM 虚表结构中的函数指针数量与函数顺序必须严格遵循二进制接口规范。例如，`IUnknown` 虚表结构 `IUnknownVtbl` 必须严格按照 `QueryInterface`、`AddRef`、`Release` 的顺序排列函数指针。当其他接口继承自 `IUnknown` 时，这个虚表结构会作为父级字段嵌入在派生接口虚表结构体的开头，并在其后追加派生接口自己的方法指针。

这种二进制层面的约定赋予了子类型多态极强的跨语言能力：不管你用的是 C++、C 还是 Rust，只要你按照 COM 约定的内存布局（即每个对象首位存放虚表指针，且虚表指针序列符合特定顺序），编译出来的二进制代码就能被其他语言直接调用，无需任何适配层。

Rust 的 trait 对象实际上和 C++ 的虚函数表实现非常类似，但是 trait 对象使用的是 fat pointer（胖指针），trait 对象的引用或指针本身（`&dyn Trait` 或 `Box<dyn Trait>` 等）包含了一个指向数据的指针和一个指向虚表的指针，而不是在对象的内存布局中直接存储一个虚表指针。我们说这是**非侵入式的**，而 C++ 的则是**侵入式的**。因此，Rust 在不使用动态分派时，没有虚表的开销。

可以看出，动态分派天生多一层内存访问（即通过虚表指针访问函数指针），而且每次调用都需要进行一次间接调用（即通过函数指针调用函数），这可能导致缓存未命中和分支预测失败，从而带来性能上的损失。并且无法静态地确定调用哪个函数，这也限制了编译器的优化能力（比如说内联、常量传播等）。因此，这种子类型多态虽然提供了极大的灵活性和可扩展性，但它的性能开销也是不可忽视的。

实际应用中，编译器可以通过一些优化手段来减少动态分派的开销，比如去虚化（devirtualization），在编译期确定需要调用的函数，直接调用它而不是通过虚表指针进行间接调用，也可以通过内联（inlining）来消除函数调用的开销。不过这些优化手段并不总是能够成功，它要求对象的动态类型是编译期可推导的，特别是在代码结构复杂或者使用了大量动态分派的情况下[^10]。

总的来说，这种子类型多态提供了极大的灵活性和可扩展性，但是同时我们也需要为这些灵活性支付对应的性能开销。对于性能敏感的代码，我们需要谨慎地使用子类型多态，或者考虑使用其他形式的多态（比如说参数多态）来实现相同的功能。

## 多态设计的多个维度

经过上面的讨论，我们可以尝试总结出不同多态机制的几个设计维度：

- 分派时机：静态 vs 动态。静态多态在编译时就确定了调用哪个函数，而动态多态则在运行时根据对象的实际类型来确定调用哪个函数。
- 代码复用机制：继承、组合、接口、模板等。不同的多态机制可能依赖于不同的代码复用机制，比如说子类型多态通常依赖于继承和虚函数，而参数多态通常依赖于模板或泛型。
- 类型约束机制：Concept、SFINAE、trait bounds 等。不同的多态机制可能提供不同的方式来约束类型参数，以确保它们满足某些条件。
- 对象模型：侵入式 vs 非侵入式。某些多态机制需要在对象的内存布局中直接存储一些额外的信息（比如说 C++ 的虚表指针），而另一些则不需要（比如说 Rust 的 trait 对象使用胖指针来存储虚表信息）。

我们可以发现，不同的多态机制在这些维度上有不同的设计选择，这些设计选择会影响它们的性能、灵活性、可扩展性等方面的特性。理解这些设计维度可以帮助我们更好地选择和使用不同的多态机制，以满足我们在实际开发中的需求。

## 结语

多态是实现抽象的利器，但是不同的多态机制有不同的设计权衡，暗含了不同的假设。

**注意灵活性的代价**。动态多态提供了极大的灵活性，但它的性能开销也是不可忽视的。对于性能敏感的代码，我们需要谨慎地使用动态多态，评估它们是不是必须的。

**进行正确的抽象**。虽然前面没有细说，但是进行正确抽象对于软件工程来说是很重要的。我们可能因为抽象的失真，而选择了过于灵活、庞杂的设计，导致系统难以维护和理解。我们需要在抽象的层次上进行权衡，选择合适的抽象来满足我们的需求，同时避免过度设计。

**软件工程没有银弹**。如此多种多样的多态机制，说明了软件工程中没有一种万能的解决方案。我们需要根据具体的场景和需求来选择合适的设计，而不是用一套设计打遍天下。

## 后记

本文中的观点大部分是从我的记忆以及搜集的资料中总结出来的，可能会有一些不准确或者不完整的地方，如果你发现了什么问题或者有更好的观点，欢迎在评论区留言讨论。

码字真累，码到后面都不知道天地为何物了，所以文章后面可能有点呓语的感觉。。虽然是古法写作，但是写出来感觉有点人机味，，求技术写作教程。。。

一些可能有用的 References:

- Klaus Iglberger, C++ Software Design: Design Principles and Patterns for High-Quality Software, O'Reilly Media, 2022.
- H. Abelson, G. J. Sussman, and J. Sussman, Structure and Interpretation of Computer Programs, 2nd ed. Cambridge: MIT Press, 1996.
- B. J. Pierce, Types and Programming Languages, MIT Press, 2002.
- A. Madhavapeddy and Y. Minsky, Real World OCaml: Functional Programming for the Masses, 2nd ed. Cambridge: Cambridge University Press, 2022.

Credits:

- fy (内容讨论、review)
- htx, yzr (review)
- Deepseek (资料搜集)

[^1]: Contract 这个词在软件工程中有一个专门的含义，指的是软件组件之间的协议或约定，规定了组件之间的交互方式、输入输出要求、错误处理等方面的细节。它类似于一个合同，明确了各方的责任和义务，以确保系统的正确性和可靠性。C++26 中引入了 Contract 这个概念，提供了[一种机制](https://en.cppreference.com/cpp/language/contracts)来定义和检查函数的前置条件、后置条件和不变式，不过这个 Contract 的定义实际上没有那么泛，注意区分。

[^2]: “静态”（Static）在编程语言中有很多不同的含义，这里指的是在编译期就确定的，也就是说，编译器在编译时就能够确定调用哪个函数，而不需要等到运行时才确定。C++ 中的虚函数一般情况下不能做到在编译时确定调用哪个函数，因此我们一般称它为“动态”（Dynamic）的。

[^3]: 其实也有运行时决定使用哪个函数重载的语言。。。

[^4]: 这种现象被称为[代码膨胀（Code Bloat）](https://en.wikipedia.org/wiki/Code_bloat)。

[^5]: 也可以使用 [hidden friend](https://www.modernescpp.com/index.php/argument-dependent-lookup-and-hidden-friends/) 的方式来定义 `swap` 函数，这样就不需要在命名空间中定义了。

[^6]: 实际上，类型擦除不是某些语言特有的，比如在 C++ 中，我们也可以使用类型擦除的技巧来实现类似 Java 的参数多态，比如说 `std::any` 或者 `std::function` 内部就使用了类型擦除，不过这通常会带来性能上的损失，因为需要在运行时进行类型检查和动态分派。

[^7]: 分派（Dispatch）就是选择调用哪个函数的过程，动态分派指的是在运行时根据对象的实际类型来选择调用哪个函数，而不是在编译时就确定了调用哪个函数。

[^8]: 代码膨胀实际上也有可能导致性能上的损失，其中一个方面是，CPU 的指令缓存是有限的，膨胀的代码会占用更多的缓存空间，从而影响性能。

[^9]: 一个不错的例子：<https://zhuanlan.zhihu.com/p/679782886>

[^10]: 即使有时候你可以人脑推断出来某个对象的动态类型，但编译器可能无法推断出来，参考[图灵停机问题](https://en.wikipedia.org/wiki/Halting_problem)。

<!-- CC-BY-SA 4.0 -->
&copy; 2026 [moyongxin](https://github.com/moyongxin). This website's content is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).
{ style="color: color-mix(in srgb,var(--foreground) 65%,transparent); margin-bottom: 0px;" }
