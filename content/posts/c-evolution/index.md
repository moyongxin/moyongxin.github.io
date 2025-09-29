+++
date = '2025-09-29T13:59:26+08:00'
draft = true
title = 'C 语言的演进（至 C23）, 以及和 C++ 的部分差异'
slug = 'c-evolution'
author = 'qwertyuiop'
tags = ["编程语言"]
keywords = ["C 语言", "C23", "C++", "C/C++ 的差异"]
readingTime = true
showFullContent = false
hideComments = false
+++

## 前言
C 语言是一门历史悠久的语言，由 *Dennis Ritchie* 在 20 世纪 70 年代创造[^1]。随后其被 *ANSI* 和 *ISO* 组织标准化，经历了多个版本的演进。本文将整理 C 语言标准的主要版本及其演进历程（主要介绍语言演进而非标准库），并对比 C 语言与 C++ 语言的一些关键差异。~~防止 C 语言课考试混淆。~~

## C 语言的演进

### K&R C
*Brian Kernighan* 和 *Dennis Ritchie* 于 1978 年出版的 *The C Programming Language*[^2] 一书中描述的 C 语言版本，通常称为 K&R C。该版本没有正式的标准，但奠定了 C 语言的基础。

该版本的重要特点包括：
- 函数返回值默认为 `int` 类型，且无需显式写出 `int`。
  ```c
  long long_ret() {
    return 42;
  }
  int_ret() {
    return 13;
  }
  ```
- 函数声明无需原型，参数需要在函数定义的参数列表中写出。
  ```c
  /* K&R C */
  int add(); /* 声明 */
  int add(a, b)
  int a;
  int b;
  {
    return a + b;
  }

  /* ANSI C */
  int add(int, int); /* 声明 */
  int add(int a, int b) {
    return a + b;
  }
  ```
- 没有 `void`、 `void *` 和 `enum` 类型，`const` 和 `volatile` 关键字也不存在。

### C89/C90 (ANSI C/ISO C)
1989 年，*ANSI* 完成了对 C 语言的标准化，发布了 ANSI C 标准（也称为 C89）。1990 年，*ISO* 采纳了该标准，发布了 ISO C 标准（C90）[^3]。这是 C 语言的第一个正式标准，并且得到了广泛的支持。大部分 C 代码都符合该标准。

此标准在 K&R C 的基础上引入了许多新特性（部分从 C++ 吸收而来），包括但不限于：
- 引入了函数原型，在函数声明中指定参数类型。
- 引入了 `void` 类型，可用于无返回值的函数。
- 引入了 `const` 和 `volatile` 关键字，用于修饰变量。
- 引入了 `enum` 枚举类型。
- 函数可返回 `struct` 和 `union` 类型。
- 对一些类型的大小进行了规范化，如 `char` 至少为 8 位，`short` 至少为 16 位，`int` 至少为 16 位，`long` 至少为 32 位。
- 使用 `...` 和 `va_list` 的可变参数列表函数。
- ...

### C99
C99 从 C++ 中吸收了更多特性，并引入一些新的特性[^4]。
- `restrict` 关键字，用于指示该指针不会产生别名。
- `inline` 关键字。
- `// ...` 的单行注释。
- 更多算术类型：
  - `_Bool` 布尔类型
  - `long long` 整数类型，至少 64 位
  - `_Complex` 复数类型
  - `_Imaginary` 虚数类型
- 柔性数组成员(*Flexible Array Member*):
    ```c
    struct S {
        int n;
        double d[];
    };

    void func() {
        struct S *s = malloc(sizeof(struct S) + 10 * sizeof(double));
        s->n = 10;
        for (int i = 0; i < s->n; i++) {
            s->d[i] = i * 1.1; // 合法访问
        }
    }
    ```
- 可变长度数组(*Variable Length Array*, VLA):
    ```c
    void func(int n) {
        int arr[n]; // n 不是常量表达式，arr 的长度在运行时确定
        for (int i = 0; i < n; i++) {
            arr[i] = i * 2;
        }
    }
    ```
- 复合字面量(*Compound Literal*)和指定初始化(*Designated Initialization*):
    ```c
    struct Point { int x, y; };
    (struct Point){20, 30}; // 复合字面量
    struct Point p2 = {.x = 10, .y = 20}; // 指定初始化
    ```
- `for` 循环中的初始化语句可声明变量:
    ```c
    for (int i = 0 /* since C99 */; i < 10; i++) {
        // ...
    }

    int i; /* C89 */
    for (i = 0; i < 10; i++) {
        // ...
    }
    ```
- 变量声明和其他语句可以混合在一起:
    ```c
    void func() {
        int x = 10;
        printf("%d\n", x);
        int y = 20; // 合法
        printf("%d\n", y);
    }
    ```
- ...

### C11
- 匿名 `struct` 和 `union`:
    ```c
    struct S {
        int n;
        union {
            struct {
                int i;
                char c;
            }; // 匿名 struct
            float f;
        }; // 匿名 union
    };

    void func() {
        struct S s;
        s.n = 1;
        s.i = 42;
        s.c = 'A';
        s.f = 3.14f;
    }
    ```
- 细化的[求值顺序](https://en.cppreference.com/w/c/language/eval_order.html)
- `_Noreturn` 关键字，表示函数不会返回。
- `_Static_assert` 关键字，用于在编译时进行断言检查。
- `_Atomic`, `_Thread_local` 以及 `<threads.h>` 支持的多线程编程。
- `_Alignas`, `_Alignof` 用于处理类型的对齐。
- `_Generic` 关键字，用于实现基本的泛型。
- 基本 Unicode 支持:
    ```c
    char16_t u16ch = u'你'; // UTF-16 字符
    char32_t u32ch = U'你';   // UTF-32 字符
    char u8str[] = u8"你好"; // UTF-8 字符串
    char16_t u16str[] = u"你好"; // UTF-16 字符串
    char32_t u32str[] = U"你好";   // UTF-32 字符串
    ```
- ...

### C17/C18
C17/C18 主要是对 C11 的修正和澄清，弃用了一些特性，没有引入新的语言特性[^5]。

### C23
C23 是最新的 C 语言标准[^6]。

- 使用 `auto` 关键字进行类型推断（类似 C++ 同名关键字）:
    ```c
    auto x = 42; // x 被推断为 int 类型
    auto y = 3.14; // y 被推断为 double 类型
    ```
- `_Decimal32`, `_Decimal64`, `_Decimal128` 三种十进制浮点类型。
- `BitInt(n)` 位整数类型，`n` 为位数。
- 数位分隔符 `'`，用于增强数字字面量的可读性:
    ```c
    int million = 1'000'000;
    float pi = 3.14'159'265f;
    ```
- 二进制字面量:
    ```c
    int b = 0b1010; // 二进制字面量，等于十进制的 10
    ```
- 空初始化器：
    ```c
    int a = {}; // 等价于 int a = 0;
    int arr[5] = {}; // 等价于 int arr[5] = {0, 0, 0, 0, 0};
    struct { int x, y; } p = {}; // 等价于 struct { int x, y; } p = {0, 0};
    ```
- 引入 `char8_t` 类型，专门用于表示 UTF-8 字符，同时更改了 UTF-8 字符及字符串字面量的类型:
    ```c
    char8_t u8ch = u8'你'; // UTF-8 字符
    char8_t u8str[] = u8"你好"; // UTF-8 字符串
    ```
- `nullptr` 常量及其类型 `nullptr_t`，类似 C++ 的 `nullptr`，表示空指针。
- 属性（类似于 C++ 的属性）:
  - `[[deprecated]]`
  - `[[fallthrough]]`
  - `[[maybe_unused]]`
  - `[[nodiscard]]`
  - `[[noreturn]]`
  - `[[reproducible]]`
  - `[[unsequenced]]`
- 大量的旧特性被改名且变为关键字:
  - `_Alignof` -> `alignof`
  - `_Alignas` -> `alignas`
  - `_Static_assert` -> `static_assert`
  - `_Thread_local` -> `thread_local`
  - `_Bool` -> `bool`
  - `true`, `false` 关键字
- ...

## C 语言与 C++ 的部分差异
C 语言和 C++ 语言（去除面向对象及元编程部分）虽然有很多相似之处，但也存在部分差异。

- C 语言允许 `void *` 指针与其他类型指针之间进行隐式转换，而 C++ 语言则要求显式转换。
```c
int *p = malloc(sizeof(int) * 10); // void * 隐式转换为 int *
```
```cpp
int *p = static_cast<int *>(malloc(sizeof(int) * 10)); // 需要显式转换
```
- C 语言中的整数类型可以隐式转换为枚举类型，而 C++ 语言则不允许这种隐式转换。
```c
enum Color { RED = 0, GREEN = 1, BLUE = 2 };
Color c = 1; // 合法，隐式转换
```
```cpp
enum Color { RED = 0, GREEN = 1, BLUE = 2 };
// Color c = 1; // 非法，不能隐式转换
Color c = static_cast<Color>(1); // 需要显式转换
```
- C 语言的 `const` 变量可以不初始化，而 C++ 语言要求 `const` 变量必须初始化。
- C 语言的 `goto` 可以跳过变量初始化，而 C++ 语言不允许这样做。
```c
void func() {
    goto skip; // 可以跳过初始化
    int x = 10;
skip:
    return;
}
```
```cpp
void func() {
    goto skip; // 非法，不能跳过初始化
    int x = 10;
skip:
    return;
}
```
- 使用结构体，联合体和枚举时，C 语言需要指定 `struct`, `union` 和 `enum`，而 C++ 语言则不需要。
```c
struct Point { int x, y; };
struct Point p; // 需要指定 struct
```
```cpp
struct Point { int x, y; };
Point p; // 不需要指定 struct
```
- C23 之前，C 语言的空函数列表 `()` 表示函数可以接受任意数量和类型的参数，而 C++ 语言的空函数列表表示函数不接受任何参数。
- C/C++ 都允许定义嵌套结构体，但是该内层结构体定义的作用域不同，C 语言中内层结构体在外层结构体的作用域外也有定义，而 C++ 语言中内层结构体的作用域是外层结构体的作用域。
- C 语言的指定初始化比 C++20 的更为灵活:
```c
struct S { int x, y, z; };
struct S s1 = {.y = 2, .x = 1}; // 合法
int arr[5] = {[0] = 3, [2] = 1}; // 合法
```
```cpp
struct S { int x, y, z; };
struct S s1 = {.y = 2, .x = 1}; // 非法，必须按声明顺序初始化
int arr[5] = {[0] = 3, [2] = 1}; // 非法，数组不支持指定初始化
```
- `NULL` 宏在 C 语言中一般定义为 `((void *)0)`，而在 C++ 语言中定义为 `0` 或 `nullptr`(C++11 后)。
- C++ 中不包括的部分 C 语言特性:
  - 可变长度数组 (*Variable Length Array*, VLA)
  - 柔性数组成员 (*Flexible Array Member*)
  - `restrict` 关键字
  - `_Complex` 和 `_Imaginary` 类型


[^1]: https://en.wikipedia.org/wiki/C_(programming_language)
[^2]: Kernighan, Brian W.; Ritchie, Dennis M. (1978). The C Programming Language (1st ed.). Englewood Cliffs: Prentice Hall. ISBN 978-0-13-110163-0. LCCN 77028983. OCLC 3608698. OL 4558528M. Wikidata Q63565563.
[^3]: https://en.wikipedia.org/wiki/ANSI_C
[^4]: https://en.cppreference.com/w/c/99.html
[^5]: https://en.cppreference.com/w/c/17.html
[^6]: https://en.cppreference.com/w/c/23.html

<!-- CC-BY-SA 4.0 -->
"[C 语言的演进（至 C23）, 以及和 C++ 的部分差异](https://blog.moyongxin.top/posts/c-evolution/index.md)" &copy; 2025 by [moyongxin](https://github.com/moyongxin) is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0)
{ style="color: color-mix(in srgb,var(--foreground) 65%,transparent); margin-bottom: 0px;" }
