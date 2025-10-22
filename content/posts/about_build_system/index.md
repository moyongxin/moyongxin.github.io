+++
date = '2025-10-22T20:57:37+08:00'
draft = false
title = '写给新手看的：关于 C/C++ 的多文件项目与构建系统'
slug = 'about_build_system'
author = 'qwertyuiop'
tags = ["工程实践"]
keywords = ["C 语言", "C++", "构建系统", "多文件项目"]
readingTime = true
showFullContent = false
hideComments = false
+++

## 前言
在学习 C/C++ 语言的过程中，新手往往会对多文件项目、构建系统等概念感到困惑。~~应我的某位同学 fy 的邀请，我决定写下本文介绍为什么需要多文件项目，为什么需要构建系统。~~

本文将分为两个部分，前半部分介绍多文件项目的必要性，后半部分介绍构建系统的作用与常用工具。

## 多文件项目

### 为什么需要多文件项目？
想象你是一位软件工程师，正在开发一个 C 程序的项目。你可能会从编写单一的一个 `main.c` 文件开始，里面包含了此项目的所有代码，并能够实现你设想的基本功能。随着项目规模的不断扩大，你会发现代码量逐渐增多，功能变得越来越复杂。此时，单一文件的结构开始暴露出一些问题：
- 代码难以维护，所有代码都集中在一个文件中，可读性变差，查找和修改特定功能变得困难。
- 团队协作受限，多个开发者同时修改同一个文件容易引发冲突。
- 编译时间变长，每次修改代码都需要重新编译整个文件。

对于以上这些问题，我们似乎需要一种更好的组织代码的方式，把代码分散到多个文件中，便于维护、协作，减少不必要的编译时间。

### C/C++ 中支持多文件项目的机制
我们可以注意到，为了支持多文件项目，理论来说至少需要一下两种机制：
- 在不引入具体实现的情况下，在一个文件中引用另一个文件的代码。（将具体实现全部引入，编译时间的优势将不复存在）
- 在当前文件找不到某个函数的定义时，通过某种机制在其他文件中查找该函数的定义。

于是乎，上古程序员为 C 语言设计了如下机制：
- 函数的声明与定义可以分离。如此一来，我们可以在不引入函数具体的实现的情况下，使用这个函数。
- 变量的声明与定义也可以分离（`extern`）。如此一来，我们可以在不引入变量具体的存储空间的情况下，使用这个变量。
- 通过 `#include` 指令，可以在一个源文件中包含另一个文件，于是我们可以把函数的声明（接口）放在一个头文件（`.h` 文件）中，然后在需要使用这些函数的源文件（`.c` 或 `.cpp` 文件）中通过 `#include` 指令包含这个头文件。
- 将软件的构建过程分为编译和链接两个主要阶段。编译器将每个源文件（`.c` 或 `.cpp` 文件）单独编译成可重定向目标文件（`.o` 文件），然后链接器将这些目标文件链接在一起，生成最终的可执行文件。

### 关于可重定向目标文件和链接
可重定向目标文件（`.o` 文件）是编译器将源代码编译成机器代码的中间产物。一般情况下，每个源文件都会被编译成一个对应的目标文件，这些目标文件包含了机器代码，但还没有被链接成最终的可执行文件。里面还包含了一些符号信息，用于链接器在链接阶段解析函数和变量的引用。链接器会解析目标文件中的符号引用，找到对应的函数和变量定义，并将它们组合在一起，生成最终的可执行文件。这套机制使得我们可以将代码分散到多个文件中，并能够正确地找到函数和变量的定义，并且只需要重新编译修改过的文件，从而提高了编译效率。

> Note:
> 在现代的工具链中，还存在 LTO（Link Time Optimization，链接时优化），使用 LTO 时 `.o` 文件中不一定包含机器代码，而是包含中间表示（IR），链接器会在链接阶段对这些 IR 进行优化和生成最终的机器代码。

### 实操多文件项目的构建过程
假设我们有一个简单的 C 项目，包含以下文件：
- `main.c`：主程序文件，包含 `main` 函数。
- `some_module.c`：一个模块的实现文件，包含一些函数以及变量的定义。
- `some_module.h`：模块的头文件，包含函数以及变量的声明。

```c
// some_module.h
#pragma once
/* extern */ int helper_function(int a, char b); // 函数默认 extern
extern int global_var; // 变量需要显式声明为 extern

// some_module.c
#include "some_module.h"
int global_var = 42; // 变量定义
static int internal_var = 100; // 静态变量，仅在本文件可见
int helper_function(int a, char b) {
    return a + (int)b + global_var + internal_var;
} // 函数定义

// main.c
#include <stdio.h>
#include "some_module.h"

int main() {
    int result = helper_function(10, 'A');
    printf("Result: %d\n", result);
    global_var += 1;
    result = helper_function(10, 'A');
    printf("Result: %d\n", result);
    return 0;
}
```

我们在 `some_module.c` 实现了某个功能函数 `helper_function(int, char)`，为了引入它的声明，我们编写了 `some_module.h` 头文件，这样为了能够使用这个函数，我们只需要在需要用到它的源文件中包含 `some_module.h` 头文件即可。
比如 `main.c` 就将被展开为：
```c
/*
省略一万行 stdio.h 的内容
*/
int helper_function(int a, char b);
extern int global_var;

int main() {
    int result = helper_function(10, 'A');
    printf("Result: %d\n", result);
    global_var += 1;
    result = helper_function(10, 'A');
    printf("Result: %d\n", result);
    return 0;
}
```
这样 `main.c` 中就能合法使用 `helper_function` 函数和 `global_var` 变量了。

为了编译这个项目，我们首先需要将每个源文件编译成目标文件（我以 Windows 上的 MinGW 为例）：
```powershell
gcc -c main.c -o main.o
gcc -c some_module.c -o some_module.o
```

为了满足你们的好奇心，我们可以看一眼生成的 `main.o` 引用的符号以及里面的机器代码：

在我的环境中，输出如下：
```
main.o:     file format pe-x86-64

SYMBOL TABLE:
[  0](sec -2)(fl 0x00)(ty    0)(scl 103) (nx 1) 0x0000000000000000 main.c
File
[  2](sec  1)(fl 0x00)(ty   20)(scl   2) (nx 1) 0x0000000000000000 main
AUX tagndx 0 ttlsiz 0x0 lnnos 0 next 0
[  4](sec  8)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .rdata$.refptr.global_var
AUX scnlen 0x8 nreloc 1 nlnno 0 checksum 0x0 assoc 0 comdat 2
[  6](sec  1)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .text
AUX scnlen 0x75 nreloc 9 nlnno 0
[  8](sec  2)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .data
AUX scnlen 0x0 nreloc 0 nlnno 0
[ 10](sec  3)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .bss
AUX scnlen 0x0 nreloc 0 nlnno 0
[ 12](sec  4)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .rdata
AUX scnlen 0xc nreloc 0 nlnno 0
[ 14](sec  5)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .xdata
AUX scnlen 0xc nreloc 0 nlnno 0
[ 16](sec  6)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .pdata
AUX scnlen 0xc nreloc 3 nlnno 0
[ 18](sec  7)(fl 0x00)(ty    0)(scl   3) (nx 1) 0x0000000000000000 .rdata$zzz
AUX scnlen 0x41 nreloc 0 nlnno 0
[ 20](sec  8)(fl 0x00)(ty    0)(scl   2) (nx 0) 0x0000000000000000 .refptr.global_var
[ 21](sec  0)(fl 0x00)(ty   20)(scl   2) (nx 0) 0x0000000000000000 __main
[ 22](sec  0)(fl 0x00)(ty   20)(scl   2) (nx 0) 0x0000000000000000 helper_function
[ 23](sec  0)(fl 0x00)(ty   20)(scl   2) (nx 0) 0x0000000000000000 printf
[ 24](sec  0)(fl 0x00)(ty    0)(scl   2) (nx 0) 0x0000000000000000 global_var



Disassembly of section .text:

0000000000000000 <main>:
   0:   55                      push   %rbp
   1:   48 89 e5                mov    %rsp,%rbp
   4:   48 83 ec 30             sub    $0x30,%rsp
   8:   e8 00 00 00 00          call   d <main+0xd>
   d:   ba 41 00 00 00          mov    $0x41,%edx
  12:   b9 0a 00 00 00          mov    $0xa,%ecx
  17:   e8 00 00 00 00          call   1c <main+0x1c>
  1c:   89 45 fc                mov    %eax,-0x4(%rbp)
  1f:   8b 55 fc                mov    -0x4(%rbp),%edx
  22:   48 8d 05 00 00 00 00    lea    0x0(%rip),%rax        # 29 <main+0x29>
  29:   48 89 c1                mov    %rax,%rcx
  2c:   e8 00 00 00 00          call   31 <main+0x31>
  31:   48 8b 05 00 00 00 00    mov    0x0(%rip),%rax        # 38 <main+0x38>
  38:   8b 00                   mov    (%rax),%eax
  3a:   8d 50 01                lea    0x1(%rax),%edx
  3d:   48 8b 05 00 00 00 00    mov    0x0(%rip),%rax        # 44 <main+0x44>
  44:   89 10                   mov    %edx,(%rax)
  46:   ba 41 00 00 00          mov    $0x41,%edx
  4b:   b9 0a 00 00 00          mov    $0xa,%ecx
  50:   e8 00 00 00 00          call   55 <main+0x55>
  55:   89 45 fc                mov    %eax,-0x4(%rbp)
  58:   8b 55 fc                mov    -0x4(%rbp),%edx
  5b:   48 8d 05 00 00 00 00    lea    0x0(%rip),%rax        # 62 <main+0x62>
  62:   48 89 c1                mov    %rax,%rcx
  65:   e8 00 00 00 00          call   6a <main+0x6a>
  6a:   b8 00 00 00 00          mov    $0x0,%eax
  6f:   48 83 c4 30             add    $0x30,%rsp
  73:   5d                      pop    %rbp
  74:   c3                      ret
  75:   90                      nop
  76:   90                      nop
  77:   90                      nop
  78:   90                      nop
  79:   90                      nop
  7a:   90                      nop
  7b:   90                      nop
  7c:   90                      nop
  7d:   90                      nop
  7e:   90                      nop
  7f:   90                      nop
```
前半段是符号表，后半段是 `main` 函数的反汇编（说明其中已经是机器代码了）。
在符号表中，我们能看到我们引用的 `helper_function` 和 `global_var` （甚至是 `printf`）都被标记为未定义的符号（scl 2 表示外部符号），说明它们的定义不在 `main.o` 中，需要链接器在后续的链接阶段去其他目标文件中寻找它们的定义。

接下来，我们需要将所有的目标文件链接在一起，生成最终的可执行文件：
```powershell
gcc main.o some_module.o -o a.exe
```

执行它我们可以看到结果：
```
Result: 217
Result: 218
```

这其中，`gcc` 实际上是编译器驱动程序，它会调用了链接器（在 GNU 工具链中通常是 `ld`）来完成链接过程。链接器会解析目标文件中的符号引用，找到对应的函数和变量定义，并将它们组合在一起，生成最终的可执行文件 `a.exe`。
为了查看具体的链接过程，我们可以使用 `-v` 选项来查看详细信息：
```powershell
gcc -v main.o some_module.o -o a.exe
```
在我的环境下，输出如下：
```
Using built-in specs.
COLLECT_GCC=D:\mingw64\bin\gcc.exe
COLLECT_LTO_WRAPPER=E:/Programs/mingw64/bin/../libexec/gcc/x86_64-w64-mingw32/15.1.0/lto-wrapper.exe
Target: x86_64-w64-mingw32
Configured with: ../../../src/gcc-15.1.0/configure ...一堆配置参数...
Thread model: mcf
Supported LTO compression algorithms: zlib
gcc version 15.1.0 (x86_64-mcf-seh-rev0, Built by MinGW-Builds project)
...一堆内部参数...
 "E:/Programs/mingw64/bin/../libexec/gcc/x86_64-w64-mingw32/15.1.0/collect2.exe" ...一堆参数直接无视... -o a.exe "E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/../../../../x86_64-w64-mingw32/lib/../lib/crt2.o" "E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/crtbegin.o" ...一堆库目录... ".\\main.o" ".\\some_module.o" -lmingw32 -lgcc -lgcc_eh -lmingwex -lmsvcrt -lkernel32 -lmcfgthread -lkernel32 -lntdll -ladvapi32 -lshell32 -luser32 -lkernel32 -liconv -lmingw32 -lgcc -lgcc_eh -lmingwex -lmsvcrt -lkernel32 -lmcfgthread -lkernel32 -lntdll "E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/crtend.o"
COLLECT_GCC_OPTIONS='-o' 'a.exe' '-mtune=core2' '-march=nocona' '-dumpdir' 'a.'
```
这里我们可以看到链接器的一个驱动程序 `collect2.exe`（实际上它还是要调用 `ld`） 被调用，并且传入了我们生成的目标文件 `main.o` 和 `some_module.o`，以及一些标准库（`printf` 函数的定义就在其中）和启动文件。链接器会将这些文件中的符号进行解析和链接，最终生成可执行文件 `a.exe`。

你可能会好奇，如果发生了重定义或者定义会怎么样？比如我们在 `main.c` 中再定义一个 `helper_function` 函数。让我们试试看：
```
E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/../../../../x86_64-w64-mingw32/bin/ld.exe: .\some_module.o:some_module.c:(.text+0x0): multiple definition of `helper_function'; .\main.o:main.c:(.text+0x0): first defined here
collect2.exe: error: ld returned 1 exit status
```
在链接的时候不传入 `some_module.o`，只传入 `main.o`：
```
E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/../../../../x86_64-w64-mingw32/bin/ld.exe: .\main.o:main.c:(.text+0x18): undefined reference to `helper_function'
E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/../../../../x86_64-w64-mingw32/bin/ld.exe: .\main.o:main.c:(.text+0x51): undefined reference to `helper_function'
E:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/15.1.0/../../../../x86_64-w64-mingw32/bin/ld.exe: .\main.o:main.c:(.rdata$.refptr.global_var[.refptr.global_var]+0x0): undefined reference to `global_var'
collect2.exe: error: ld returned 1 exit status
```
可以看到，链接器都会发现这些问题，并在报错信息中提示我们（`multiple definition of ...`，`undefined reference to ...`）。

> 中场休息一下吧，似乎有点太长了。。。

## 构建系统
### 引入构建系统的动机
想象你的项目规模继续扩大，源文件数量增加到几十个、上百个，手动编写和维护编译命令变得非常繁琐和容易出错。每次修改代码后，你需要重新编译所有相关的文件，这不仅耗时，而且容易遗漏某些文件的编译，导致链接错误。
为了简化这个过程，上古程序员编写了一个能基于规则及文件最后更新时间执行相应命令的工具 `make`，
在这个工具的帮助下，我们可以通过编写一个 `Makefile` 文件来定义项目的构建规则和命令行，从而实现自动化构建。

`Makefile` 文件的风格大概是这样的：
```makefile
# 编译器
CC = gcc
# 编译选项
CFLAGS = -Wall -g
# 目标可执行文件
TARGET = a.exe
# 源文件
SRCS = main.c some_module.c
# 目标文件
OBJS = $(SRCS:.c=.o)

# 默认目标
all: $(TARGET)

# 链接目标文件生成可执行文件
$(TARGET): $(OBJS)
    $(CC) $(OBJS) -o $(TARGET)

# 编译源文件生成目标文件
%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@

# 清理生成的文件
clean:
    rm -f $(OBJS) $(TARGET)

# 声明伪目标
.PHONY: all clean
```
每次执行 `make` 的时候，它便会根据 `Makefile` 中的规则，自动判断哪些文件需要重新编译，并执行相应的命令行，从而简化了构建过程，提高了效率。

[Ninja](https://ninja-build.org/) 则是一个更简洁的构建系统，专注于速度和并行构建。它使用一个类似于 `Makefile` 的文件 `build.ninja` 来定义构建规则，但更加专注于高效执行，因此，它的功能相比 make，做了很多减法，并不适合人工编写，而是通常由其他工具生成。比如 Chromium 项目曾经使用的 gyp 和现在使用的 gn 都是生成 ninja 构建文件的工具。这些也是下一节要介绍的元构建系统。

### 元(Meta)构建系统的出现

随着项目规模的扩大，构建系统的复杂性也在增加。平台工具链的多样性，依赖关系的复杂性，项目内部规则的复杂性，为了应对这种复杂性，出现了一些元构建系统（Meta Build System），它们能够生成标准构建系统的配置文件（如 Makefile、Visual Studio 的项目文件等），从而简化构建过程。

早期最有影响力的元构建系统应该是 GNU 的 [Autotools](https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html) 全家桶，它包括 `autoconf`、`automake` 和 `libtool` 等工具，能够帮助开发者生成适用于不同平台的 Makefile，从而简化跨平台构建过程。但其配置过程相对复杂，学习曲线较陡峭。基本只能在 GNU 的一些项目中看到它的身影。

这些元构建系统通常提供了更高级别的抽象，允许开发者以更简洁的方式定义构建规则。例如，[CMake](https://cmake.org/) 就是一个广泛使用的元构建系统，它在社区中的接受度最高，和 Ninja 一起，基本上成为了 C/C++ 项目的事实标准。它使用了一个 DSL(Domain Specific Language)，让开发者定义简单的规则，它能够生成适用于不同平台和编译器的项目文件。

比如上面的例子用 cmake 改写
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject)

add_executable(a main.c some_module.c)
target_compile_options(a PRIVATE -Wall -g)
```

通过使用元构建系统，开发者可以更专注于项目的逻辑，而不必过多关注底层的构建细节。这种方式不仅提高了开发效率，也降低了因手动维护构建配置而导致的错误风险。

### 更灵活、易用的构建系统？
随着软件项目的复杂性不断增加，传统的构建系统和元构建系统在某些方面可能无法满足现代开发需求（可能有很复杂的测试逻辑，代码生成过程，分布式编译等），同时，开发者对构建系统的易用性也提出了更高的要求。为了应对这些挑战，一些更灵活和强大的构建系统应运而生，如 [SCons](https://github.com/SCons/scons)、[Bazel](https://bazel.build/)、[meson](https://mesonbuild.com/)、[xmake](https://xmake.io/) 等。
- SCons 直接使用 Python 作为配置语言，可以编写非常复杂的构建逻辑，支持增量构建和并行构建。
- Bazel 强调可扩展性，适用于大型项目，支持多语言构建和分布式构建。
- meson 专门设计了一门图灵不完备的 DSL 限制表达性，从而保证构建脚本的可预测性，并且注重速度和易用性。
- xmake 也是一款现代化的构建系统，使用 Lua 作为配置语言，简单易用。
- ...

我在知乎上也找到了几个比较有趣的构建系统，感兴趣的读者可以去看看：
- C 语言和 C++ 的构建系统如何选择？ - 韦易笑的回答 - 知乎
https://www.zhihu.com/question/631446273/answer/3609230745 可将编译参数写在代码文件中的非常有趣的构建系统
- SB: 使用 C# 为大型 C++ 项目实现构建与自动化系统 - SaeruHikari的文章 - 知乎
https://zhuanlan.zhihu.com/p/1956801299178854290 直接使用 C# 编写构建配置

## 结语
项目的概念和构建系统在软件工程中都十分重要，同时我们也能看到前人的智慧是如何一步步演进，解决了我们在开发过程中遇到的各种问题。希望本文能帮助新手更好地理解多文件项目和构建系统的概念，为今后的学习和工作打下坚实的基础。

<!-- CC-BY-SA 4.0 -->
"[写给新手看的：关于 C/C++ 的多文件项目与构建系统](https://blog.moyongxin.top/posts/about_build_system/index.md)" &copy; 2025 by [moyongxin](https://github.com/moyongxin) is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0)
{ style="color: color-mix(in srgb,var(--foreground) 65%,transparent); margin-bottom: 0px;" }
