# LLVM 编码规范

**LLVM Coding Standards [官网](https://llvm.org/docs/CodingStandards.html) | 历史翻译版本 [Github](https://github.com/zxhio/coding)**

- [导论](#intro)
- [语言、库和标准](#lang)
- [C++ 标准版本](#cppver)
- [C++ 标准库](#stdlib)
- [Go 代码准则](#go_guid)
- [机械的代码问题](#src_issues)
    - [代码格式化](#src_fmt)
    - [注释](#comment)
        - [头文件](#comment_header)
        - [类概述](#class_overview)
        - [method information](#method_info)
    - [注释格式化](#comment_fmt)
    - [使用Doxygen注释](#comment_doxygen)
    - [错误和警告消息](#err_warn_msg)
    - [#include 风格](#include_style)
    - [代码宽度](#src_width)
    - [空格](#whitespace)
        - [格式化lambda如同代码块](#lambda_fmt)
        - [花括号初始化列表](#init_list_fmt)
    - [语言和编译器问题](#lang_compile_issue)
        - [像错误一样对待编译警告](#compile_warn)
        - [编写可移植的代码](#portable_code)
        - [不要使用 RTTI 或 Exceptions](#no_rtti_exception)
        - [不要使用静态构造函数](#no_static_ctor)
        - [使用class和struct关键字](#class_struct)
        - [不要使用花括号初始化列表调用构造函数](#no_brace_call_ctor)
        - [使用auto类型推导提高代码可读性](#auto)
        - [小心auto带来不必要的拷贝](#beware_auto_copy)
        - [小心对指针排序带来的不确定性](#beware_order_pointer)
        - [小心相同元素不确定的排序顺序](#beware_equal_order)
- [风格问题](#fmt_issue)
    - [上层问题](#high_level_issue)
        - [头文件独立](#header_self_include)
        - [库层次](#lib_layering)
        - [尽可能减少 #include](#little_include)
        - [保持私有的内部头文件](#intenal_header)
        - [使用 namespace 限定符来实现之前声明的函数](#namespace_impl)
        - [提前退出或 continue 以简化代码](#early_exit_continue)
        - [return 之后不要使用 else](#no_else_after_return)
        - [将判断循环转换为判断函数](#predicate_loop_func)
    - [底层问题](#low_level_issue)
        - [合适的类型、函数、变量和枚举命名](#name)
        - [善用断言](#assert)
        - [不要使用 using namesapce std](#no_namespace_std)
        - [为头文件中的类提供虚函数锚](#virtual_method)
        - [不要在全覆盖枚举的switch 中使用 default](#no_default_in_switch)
        - [尽可能的使用基于范围的循环](#range_based_loop)
        - [不要每次通过循环取 end()](#eval_end)
        - [禁止包含 iostream](#forbidden_iostream)
        - [使用 raw_stream](#raw_stream)
        - [避免 std::endl](avoid_std_endl)
        - [不要在class中声明的函数中使用 inline](#no_class_inline)
    - [微观细节](#micro_details)
        - [括号之前的空格](#space_before_parentheses)
        - [推荐前置自增](#preincrement)
        - [namespace 缩进](#namespace_indent)
        - [匿名 namespace](#anonymous_namespace)
- [其它参考](#see_also)

## <span id="intro">导论</span>

This document describes coding standards that are used in the LLVM project. Although no coding standards should be regarded as absolute requirements to be followed in all instances, coding standards are particularly important for large-scale code bases that follow a library-based design (like LLVM).

While this document may provide guidance for some mechanical formatting issues, whitespace, or other “microscopic details”, these are not fixed standards. Always follow the golden rule: **If you are extending, enhancing, or bug fixing already implemented code, use the style that is already being used so that the source is uniform and easy to follow.**

Note that some code bases (e.g. libc++) have special reasons to deviate from the coding standards. For example, in the case of libc++, this is because the naming and other conventions are dictated by the C++ standard.

There are some conventions that are not uniformly followed in the code base (e.g. the naming convention). This is because they are relatively new, and a lot of code was written before they were put in place. Our long term goal is for the entire codebase to follow the convention, but we explicitly do not want patches that do large-scale reformatting of existing code. On the other hand, it is reasonable to rename the methods of a class if you’re about to change it in some other way. Please commit such changes separately to make code review easier.

The ultimate goal of these guidelines is to increase the readability and maintainability of our common source base.


本文档介绍在 *LLVM* 工程中使用的编码规范。尽管在任何情况下编码规范不应该被视为绝对遵循的需求，但是编码规范对于遵循基于库的设计（例如 LLVM）的大规模代码库而言尤为重要。

本文可能给一些机械的格式问题，空格或者其它细微的细节提供指引，不过没有固定的规范。始终遵守黄金规则： **使用当前的代码规范对已经实现的代码进行扩展，增强或者bug修复，这样源码风格统一并且易于遵守。**

注意，某些代码库（如libc++）有特殊的理由偏离此代码规范。比方说libc++，其命名和其它约定是由C++标准决定的。

在代码库中有一些约定没有被统一遵循（如命名约定）。这是因为这些约定相对较新而非常多的代码在这些约定出现之前就已经存在了。我们长期目标是整个代码库都遵循约定，但是我们明确不希望出现对现有代码进行大规模重新格式化的补丁。另一方面，如果你要以其它方式进行修改，则重命名类的方法是合理的。请分别提交此类修改，以便更容易地进行代码审查。

这些准则的最终目标就是提高我们公共源码库的可读性和可维护性。

## <span id="lang">语言、库和标准</span>

Most source code in LLVM and other LLVM projects using these coding standards is C++ code. There are some places where C code is used either due to environment restrictions, historical restrictions, or due to third-party source code imported into the tree. Generally, our preference is for standards conforming, modern, and portable C++ code as the implementation language of choice.

大部分在 *LLVM* 和其它 *LLVM* 项目中使用该代码规范的源码为C++代码。由于一些环境、历史限制或者引入至源码树中的第三方源码导致在某些地方存在C代码。通常而言，我们倾向于符合标准，现代和可移植的C++代码作为选择实现的语言。

## <span id="cppver">C++ 标准版本</span>

Unless otherwise documented, LLVM subprojects are written using standard C++14 code and avoid unnecessary vendor-specific extensions.

Nevertheless, we restrict ourselves to features which are available in the major toolchains supported as host compilers (see Getting Started with the LLVM System page, section Software).

Each toolchain provides a good reference for what it accepts:

- Clang: https://clang.llvm.org/cxx_status.html
- GCC: https://gcc.gnu.org/projects/cxx-status.html#cxx14
- MSVC: https://msdn.microsoft.com/en-us/library/hh567368.aspx

除非另有说明，否则 *LLVM* 子项目使用 *C++14* 标准编写并且避免不必要的厂商定制扩展。然而，我们将自己限制在宿主编译器支持的主要工具链中的可用功能之中（见[LLVM系统入门](https://llvm.org/docs/GettingStarted.html), 软件部分）。

每一个工具链都提供了一个好的参考：
- Clang: https://clang.llvm.org/cxx_status.html
- GCC: https://gcc.gnu.org/projects/cxx-status.html#cxx14
- MSVC: https://msdn.microsoft.com/en-us/library/hh567368.aspx

## <span id="stdlib">C++ 标准库</span>

Instead of implementing custom data structures, we encourage the use of C++ standard library facilities or LLVM support libraries whenever they are available for a particular task. LLVM and related projects emphasize and rely on the standard library facilities and the LLVM support libraries as much as possible.

LLVM support libraries (for example, ADT) implement specialized data structures or functionality missing in the standard library. Such libraries are usually implemented in the llvm namespace and follow the expected standard interface, when there is one.

When both C++ and the LLVM support libraries provide similar functionality, and there isn’t a specific reason to favor the C++ implementation, it is generally preferable to use the LLVM library. For example, llvm::DenseMap should almost always be used instead of std::map or std::unordered_map, and llvm::SmallVector should usually be used instead of std::vector.

We explicitly avoid some standard facilities, like the I/O streams, and instead use LLVM’s streams library (raw_ostream). More detailed information on these subjects is available in the LLVM Programmer’s Manual.

For more information about LLVM’s data structures and the tradeoffs they make, please consult [that section of the programmer’s manual](https://llvm.org/docs/ProgrammersManual.html#picking-the-right-data-structure-for-a-task).

只要能用于特定的任务，我们推荐使用C++标准库或者LLVM支持的库设施，而不是自定义用户数据结构。LLVM和相关项目尽可能的突出和依赖标准库和LLVM支持的库设施。

LLVM支持的库（例如ADT）实现了标准库缺少的特有数据结构或者功能。一些库经常被实现在`llvm`命名空间内并且遵循预期的标准接口。

当C++标准库和LLVM支持的库都支持一个相似的功能时，并且没有特定的理由去偏向C++标准库的实现，这种情况通常更倾向于使用LLVM的库。举个例子：`llvm::DenseMap` 几乎总是应该被使用而不是 `std::map` 或者 `std::unordered_map`，并且 `llvm::SmallVector` 应该总是代替使用 `std::vector`。我们明确的避免使用一些标准库设施，如标准 I/O stream，取而代之的是使用 LLVM 的 stream库（[raw_ostream](https://llvm.org/docs/CodingStandards.html#raw-ostream)）。更多相关主题的细节信息参考 [LLVM 开发者手册](https://llvm.org/docs/ProgrammersManual.html).

更多关于LLVM数据结构和做出权衡的信息，请参考 [that section of the programmer’s manual](https://llvm.org/docs/ProgrammersManual.html#picking-the-right-data-structure-for-a-task)。


## <span id="go_guid">Go 代码准则</span>

Any code written in the Go programming language is not subject to the formatting rules below. Instead, we adopt the formatting rules enforced by the gofmt tool.

Go code should strive to be idiomatic. Two good sets of guidelines for what this means are Effective Go and Go Code Review Comments.

任何使用Go语言的写下的代码不受以下（指C++）的格式化规则约束，相反的，我们使用[gofmt](https://golang.org/cmd/gofmt/)工具强制执行格式化规则。

Go 代码应该尽量符合习惯。[Effective Go](https://golang.org/doc/effective_go.html) 和 [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) 是两套好的准则。

## <span id="src_issues">机械的代码问题</span>

### <span id="src_fmt">代码格式化</span>

#### <span id="comment">注释</span>

Comments are important for readability and maintainability. When writing comments, write them as English prose, using proper capitalization, punctuation, etc. Aim to describe what the code is trying to do and why, not how it does it at a micro level. Here are a few important things to document:

注释对于可读性和可维护性非常重要。当写注释时，使用适当的大小写、标点符号等将其写成英文句子。专注于描述代码试图做什么和为什么这么做，不要从微观的角度去描述代码做的事情。以下是一些重要的记录：

#### <span id="comment_header">头文件</span>

Every source file should have a header on it that describes the basic purpose of the file. The standard header looks like this:

A few things to note about this particular format: The “-*- C++ -*-” string on the first line is there to tell Emacs that the source file is a C++ file, not a C file (Emacs assumes .h files are C files by default).

> This tag is not necessary in .cpp files. The name of the file is also on the first line, along with a very short description of the purpose of the file.

The next section in the file is a concise note that defines the license that the file is released under. This makes it perfectly clear what terms the source code can be distributed under and should not be modified in any way.

The main body is a Doxygen comment (identified by the /// comment marker instead of the usual //) describing the purpose of the file. The first sentence (or a passage beginning with \brief) is used as an abstract. Any additional information should be separated by a blank line. If an algorithm is based on a paper or is described in another source, provide a reference.

每一个源码文件应该有一个描述该文件基本用途的头部首部，标准的首部看起来像这样：

```cpp
//===-- llvm/Instruction.h - Instruction class definition -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains the declaration of the Instruction class, which is the
/// base class for all of the VM instructions.
///
//===----------------------------------------------------------------------===//
```

注意其中的一些特殊的格式：`-*- C++ -*-` 用于指示Emacs该文件为C++文件而不是C文件，（在Emacs中头文件默认为C文件）。
> 注意该标记在 .cpp 文件中不是必要的。文件名及一句简短的文件用途描述存在在文件的第一行。

文件的下一部分是一个简短的说明，它定义了文件发布所依据的 *license*。这样就清楚的说明了源代码在什么条件下可以发布，并且不能以任何形式进行修改。

主体部分是一些描述该文件用途的*Doxygen*注释（通过 `///` 注释记号而不是 `//`来标记）。第一句（段落以 `\brief`开始）用作于摘要。任何附加的信息都应该以空行分隔。如果一个算法是基于一篇论文或者在其它来源说明的，则提供一个引用。

#### <span id="class_overview">类概述</span>

Classes are a fundamental part of an object-oriented design. As such, a class definition should have a comment block that explains what the class is used for and how it works. Every non-trivial class is expected to have a doxygen comment block.

类是面向对象设计的基础部分。一个类定义应该有一个描述该类的用途和如何工作的注释块。每一个非平凡类都应该有一个*doxygen*注释块。

#### <span id="method_info">方法信息</span>

Methods and global functions should also be documented. A quick note about what it does and a description of the edge cases is all that is necessary here. The reader should be able to understand how to use interfaces without reading the code itself.

Good things to talk about here are what happens when something unexpected happens, for instance, does the method return null?

方法和全局函数同样应该被记录。简单备注一下它的功能和对边缘情况的描述。读者应该能够理解如何在不阅读代码本书的情况下使用接口。

这里更值得讨论的事情是当意外出现时会发生什么，例如，方法是否返回null？

### <span id="comment_fmt">注释格式化</span>

In general, prefer C++-style comments (// for normal comments, /// for doxygen documentation comments). There are a few cases when it is useful to use C-style (/* */) comments however:
1. When writing C code to be compatible with C89.
2. When writing a header file that may be #included by a C source file.
3. When writing a source file that is used by a tool that only accepts C-style comments.
4. When documenting the significance of constants used as actual parameters in a call. This is most helpful for bool parameters, or passing 0 or nullptr. The comment should contain the parameter name, which ought to be meaningful. For example, it’s not clear what the parameter means in this call:
    ```cpp
    Object.emitName(nullptr);
    ```
    An in-line C-style comment makes the intent obvious:
    ```cpp
    Object.emitName(/*Prefix=*/nullptr);
    ```

Commenting out large blocks of code is discouraged, but if you really have to do this (for documentation purposes or as a suggestion for debug printing), use #if 0 and #endif. These nest properly and are better behaved in general than C style comments.

通常而言，更倾向于C++风格的注释（`//` 为一般的注释，`///` 为*doxygen*文档注释）。不过有一小部分情况下C风格注释也是有用的：
1. 兼容C89的C代码
2. 只被C源文件包含的头文件
3. C源文件内只接受C风格的注释
4. 记录调用中实参常量的重要性。这对用于bool类型、0或nullptr有极大帮助。注释应该包含有意义的参数名字。举个例子，有一个参数含义不清晰的调用：
    ```cpp
    Object.emitName(nullptr);
    ```
    一行C风格注释使其意图变得明显：
    ```cpp
    Object.emitName(/*Prefix=*/nullptr);
    ```

不推荐对大规模代码块进行注释，如果必须要注释的话（记录意图或者作为调试打印的建议），使用 `#if 0` 和 `#endif`。 适当的嵌套是比一般的使用C风格注释更好的行为。

### <span id="comment_doxygen">使用doxygen注释</span>

Use the \file command to turn the standard file header into a file-level comment.

Include descriptive paragraphs for all public interfaces (public classes, member and non-member functions). Avoid restating the information that can be inferred from the API name. The first sentence (or a paragraph beginning with \brief) is used as an abstract. Try to use a single sentence as the \brief adds visual clutter. Put detailed discussion into separate paragraphs.

To refer to parameter names inside a paragraph, use the \p name command. Don’t use the \arg name command since it starts a new paragraph that contains documentation for the parameter.

Wrap non-inline code examples in \code ... \endcode.

To document a function parameter, start a new paragraph with the \param name command. If the parameter is used as an out or an in/out parameter, use the \param [out] name or \param [in,out] name command, respectively.

To describe function return value, start a new paragraph with the \returns command.

A minimal documentation comment:

A documentation comment that uses all Doxygen features in a preferred way:


使用 `\file` 指令将标准文件首部转为文件级别的注释。

包括所有的公共接口（公共类、成员和非成员函数）描述性段落。避免重复可以从 API 名字中获取的信息。第一句（对段落而言以 `\brief` 开始）用作于一个概述。用单个句子作为 `\brief` 的补充，具体的细节讨论放在下一段落之中。

使用 `\p name` 指令在段落注释中引用参数的名称。不要在包含参数描述的新段落中使用 `\arg name` 指令。

使用 `\code ... \endcode` 指令来包装非内联代码。

新起一个段落使用 `\param name` 指令来描述一个函数的参数，如果是用来输出或者输入输出的参数，使用 `\param [out] name` 或者 `\param [in,out] name` 指令来表示。

新起一段使用 `\returns` 来描述函数的返回。

一个最简短的文档注释 。。。
```cpp
/// Sets the xyzzy property to \p Baz.
void setXyzzy(bool Baz);
```

一个首选的文档注释是使用所有的 *doxygen* 特性。
```cpp
/// Does foo and bar.
///
/// Does not do foo the usual way if \p Baz is true.
///
/// Typical usage:
/// \code
///   fooBar(false, "quux", Res);
/// \endcode
///
/// \param Quux kind of foo to do.
/// \param [out] Result filled with bar sequence on foo success.
///
/// \returns true on success.
bool fooBar(bool Baz, StringRef Quux, std::vector<int> &Result);
```

Don’t duplicate the documentation comment in the header file and in the implementation file. Put the documentation comments for public APIs into the header file. Documentation comments for private APIs can go to the implementation file. In any case, implementation files can include additional comments (not necessarily in Doxygen markup) to explain implementation details as needed.

Don’t duplicate function or class name at the beginning of the comment. For humans it is obvious which function or class is being documented; automatic documentation processing tools are smart enough to bind the comment to the correct declaration.

不要在头文件和实现的文件中复制文档注释。将对公共API的文档注释放在头文件中，对私有API的文档注释可以放到实现的文件中。无论如何，可以按需在实现文件按中添加（在*doxygen*制作中不是必需的）额外的注释信息来解释实现的细节。

不要在注释的起始处复制函数或者类名称。对读者而言，哪个函数或者类被记录下来是显而易见的；自动的文档处理工具可以足够聪明的将注释绑定到正确的声明上去。

避免:
```cpp
// Example.h:

// example - Does something important.
void example();

// Example.cpp:

// example - Does something important.
void example() { ... }
```

推荐:
```cpp
// Example.h:

/// Does something important.
void example();

// Example.cpp:

/// Builds a B-tree in order to do foo.  See paper by...
void example() { ... }
```

### <span id="err_warn_msg">错误和警告消息</span>

Clear diagnostic messages are important to help users identify and fix issues in their inputs. Use succinct but correct English prose that gives the user the context needed to understand what went wrong. Also, to match error message styles commonly produced by other tools, start the first sentence with a lower-case letter, and finish the last sentence without a period, if it would end in one otherwise. Sentences which end with different punctuation, such as “did you forget ‘;’?”, should still do so.

清晰的诊断消息对于帮助用户识别和修复问题而言非常重要，使用简短但正确的英文陈述可以给出需要的上下文让用户理解。同样，产生的错误信息的格式需要其它工具来匹配，第一句应该以小写字母开始，如果以一种其它方式结束这句话则结尾不带有句号。句子使用不同的标点符号结尾，像一些 “did you forget ‘;’?”, 则应该继续保持不同。

良好的错误消息示例：
```c
error: file.o: section header 3 is corrupt. Size is 10 when it should be 20
```

不好的消息，因为没有提供有用的信息并且格式错误：
```c
error: file.o: Corrupt section header.
```

As with other coding standards, individual projects, such as the Clang Static Analyzer, may have preexisting styles that do not conform to this. If a different formatting scheme is used consistently throughout the project, use that style instead. Otherwise, this standard applies to all LLVM tools, including clang, clang-tidy, and so on.

If the tool or project does not have existing functions to emit warnings or errors, use the error and warning handlers provided in Support/WithColor.h to ensure they are printed in the appropriate style, rather than printing to stderr directly.

When using report_fatal_error, follow the same standards for the message as regular error messages. Assertion messages and llvm_unreachable calls do not necessarily need to follow these same styles as they are automatically formatted, and thus these guidelines may not be suitable.

其它独立项目中的编码规范，如 Clang-Static-Analyzer，可能遗留不遵循该规范的风格。如果一个不同的格式化方案始终用于整个项目，则沿用那个风格。否则，该规范用于 LLVM 中的所有工具，包括 clang、clang-tidy 等等。

如果这个工具或项目中没有可以产生警告和错误的功能，则使用 `Support/WithColor.h` 提供的错误和警告Handler来确保这些信息可以以适当的格式打印出来，而不是直接打印至 `stderr` 中。

当使用 `report_fatal_error`，遵循相同的规范保持错误消息的规律。断言消息和`llvm_unreachable`调用则没有必要遵循这种自动格式化的格式，因此这些参考不一定合适。

### <span id="include_style">include 风格</span>

Immediately after the header file comment (and include guards if working on a header file), the minimal list of #includes required by the file should be listed. We prefer these #includes to be listed in this order:
- Main Module Header
- Local/Private Headers
- LLVM project/subproject headers (clang/..., lldb/..., llvm/..., etc)
- System #includes

and each category should be sorted lexicographically by the full path.

The Main Module Header file applies to .cpp files which implement an interface defined by a .h file. This #include should always be included first regardless of where it lives on the file system. By including a header file first in the .cpp files that implement the interfaces, we ensure that the header does not have any hidden dependencies which are not explicitly #included in the header, but should be. It is also a form of documentation in the .cpp file to indicate where the interfaces it implements are defined.

LLVM project and subproject headers should be grouped from most specific to least specific, for the same reasons described above. For example, LLDB depends on both clang and LLVM, and clang depends on LLVM. So an LLDB source file should include lldb headers first, followed by clang headers, followed by llvm headers, to reduce the possibility (for example) of an LLDB header accidentally picking up a missing include due to the previous inclusion of that header in the main source file or some earlier header file. clang should similarly include its own headers before including llvm headers. This rule applies to all LLVM subprojects.

紧接文件首部注释（和包含保护，如果是头文件）之后，该文件需要的最小 `#include` 列表应该被列出来。我们推荐 `#include` 顺序如下：
- 主模块头文件
- 局部的/私有的头文件
- LLVM 项目/子项目头文件 （clang/..., lldb/..., llvm/..., etc）
- 系统文件 `#include`

并且每一个种类的头文件完整路径应该做一个排序。

主模块头文件适用于声明实现接口的 `.cpp` 文件的 `.h` 文件。该 `#include` 不管处在什么文件系统下总是应该第一个被包含。通过在实现该接口的第一包含头文件可以确保头文件没有隐藏需要但没有显式包含的依赖。同样的，这也是在 `.cpp`中一种指明接口声明在何处的记录。

LLVM 项目和子项目的头文件应该从最高优先级到最低优先级分组，理由同上。举个例子，LLDB 同时依赖 clang 和 LLVM，并且 clang 依赖 LLVM。所以一个 LLDB 源文件应该最先包含 LLDB ，其次是 clang 头文件，其次是 LLVM 头文件，这样做是为了在源文件中的头文件或更前的头文件包含情况下降低LLDB头文件包含缺失的可能。clang 同样的应该在包含LLVM文件之前包含其自身的头文件。这个规则适用于所有的 LLVM 的子项目。

### <span id="src_width">代码宽度</span>

Write your code to fit within 80 columns.

There must be some limit to the width of the code in order to allow developers to have multiple files side-by-side in windows on a modest display. If you are going to pick a width limit, it is somewhat arbitrary but you might as well pick something standard. Going with 90 columns (for example) instead of 80 columns wouldn’t add any significant value and would be detrimental to printing out code. Also many other projects have standardized on 80 columns, so some people have already configured their editors for it (vs something else, like 90 columns).

调整代码在80列以内。

代码的宽度限制是为了允许开发者同时在一个窗口中并排打开多个文件而可以合适的显示。如果你打算选择一个宽度限制，这个限制是任意的但是你同样需要选择一个标准。选择90列代替80列并不增加任何有意义的值并且将对输出代码有坏处。此外，有非常多个其它项目对80列进行了标准化，因此有些人为它对编辑器做了配置（对比其它的宽度，例如90列）。

### <span id="whitespace">空格</span>

In all cases, prefer spaces to tabs in source files. People have different preferred indentation levels, and different styles of indentation that they like; this is fine. What isn’t fine is that different editors/viewers expand tabs out to different tab stops. This can cause your code to look completely unreadable, and it is not worth dealing with.

As always, follow the Golden Rule above: follow the style of existing code if you are modifying and extending it.

Do not add trailing whitespace. Some common editors will automatically remove trailing whitespace when saving a file which causes unrelated changes to appear in diffs and commits.

在所有情况下，在源文件中较之制表符更推荐空格。对于缩进级别和缩进的格式人们有不同的偏爱，这很好。不好的事情是不同的编辑器/阅读器将制表符扩展为不同的制表长度。这会造成你的代码看起来完全的不可读并且这样的情况不值得去处理。

一如既往，遵循黄金规则如上：如果要修改和扩展代码则遵循已存在的代码风格。

不要在结尾添加空格，一些常用的编辑器会在保存文件时自动的移除结尾的空格，这样导致在diff或者commit时产生不相关的改变。

#### <span id="lambda_fmt">类代码块格式化lambda</span>

When formatting a multi-line lambda, format it like a block of code. If there is only one multi-line lambda in a statement, and there are no expressions lexically after it in the statement, drop the indent to the standard two space indent for a block of code, as if it were an if-block opened by the preceding part of the statement:

像格式化代码块一样格式化多行的lambda。如果语句中只有一个多行lambda，并且lambda之后没有词法表达式，将代码块下降标准的2空格缩进进行缩进，如同前面的语句打开的 if-block。
```cpp
std::sort(foo.begin(), foo.end(), [&](Foo a, Foo b) -> bool {
  if (a.blah < b.blah)
    return true;
  if (a.baz < b.baz)
    return true;
  return a.bam < b.bam;
});
```

To take best advantage of this formatting, if you are designing an API which accepts a continuation or single callable argument (be it a function object, or a std::function), it should be the last argument if at all possible.

If there are multiple multi-line lambdas in a statement, or additional parameters after the lambda, indent the block two spaces from the indent of the []:

为了充分利用这种格式，如果你将设计一个接受延长或者可调用的参数（函数对象，或者 std::function）的API，它应该尽可能的作为最后一个参数。

如果一条语句中有多个多行lambda表达式，或者额外的参数在lambda之后，从 `[]` 的缩进开始以2空格缩进这个代码块：
```cpp
dyn_switch(V->stripPointerCasts(),
           [] (PHINode *PN) {
             // process phis...
           },
           [] (SelectInst *SI) {
             // process selects...
           },
           [] (LoadInst *LI) {
             // process loads...
           },
           [] (AllocaInst *AI) {
             // process allocas...
           });
```

#### <span id="init_list_fmt">花括号初始化列表</span>

Starting from C++11, there are significantly more uses of braced lists to perform initialization. For example, they can be used to construct aggregate temporaries in expressions. They now have a natural way of ending up nested within each other and within function calls in order to build up aggregates (such as option structs) from local variables.

The historically common formatting of braced initialization of aggregate variables does not mix cleanly with deep nesting, general expression contexts, function arguments, and lambdas. We suggest new code use a simple rule for formatting braced initialization lists: act as-if the braces were parentheses in a function call. The formatting rules exactly match those already well understood for formatting nested function calls. Examples:

This formatting scheme also makes it particularly easy to get predictable, consistent, and automatic formatting with tools like Clang Format.

从C++11开始，花括号列表执行初始化的用途明显变多。如，在表达式中构造临时集合。现在，他们有一种自然的方式结束，可以互相嵌套，也可以嵌套在函数叼哦那个之中，以便于从局部变量中构建集合（如结构体）。

原来常见的花括号集合变量初始化的格式化不能清晰的混合深层的嵌套、一般的表达式上下文、函数参数和 lambda。我们建议使用一条简单的规则来格式化新代码中的花括号初始化列表：在函数调用中将花括号`{}`作为圆括号`()`来对待。该格式化规则与已经易懂的嵌套函数调用格式化规则完全匹配。如：

```cpp
foo({a, b, c}, {1, 2, 3});

llvm::Constant *Mask[] = {
    llvm::ConstantInt::get(llvm::Type::getInt32Ty(getLLVMContext()), 0),
    llvm::ConstantInt::get(llvm::Type::getInt32Ty(getLLVMContext()), 1),
    llvm::ConstantInt::get(llvm::Type::getInt32Ty(getLLVMContext()), 2)};
```

这个格式化方案同样让工具例如 clang-format 可以非常简单的预测、一致的自动格式化。

### <span id="lang_compile_issue">语言和编译器问题</span>

#### <span id="compile_warn">像错误一样对待编译警告</span>

Compiler warnings are often useful and help improve the code. Those that are not useful, can be often suppressed with a small code change. For example, an assignment in the if condition is often a typo:

编译警告往往是有效的并且可以有利于改进代码。而那些无用的警告通过代码小改动就可以被消除。例如，在 `if` 条件中使用赋值是一个经常出现的问题：

```cpp
if (V = getValue()) {
  ...
}
```

Several compilers will print a warning for the code above. It can be suppressed by adding parentheses:

多数的编译器会针对于以上代码打印一条警告。可以通过添加一个括号来消除警告：

```cpp
if ((V = getValue())) {
  ...
}
```

#### <span id="portable_code">编写可移植的代码</span>

In almost all cases, it is possible to write completely portable code. When you need to rely on non-portable code, put it behind a well-defined and well-documented interface.

在绝大数的情况下，尽可能的编写完全可移植的代码。当你需要依赖于不可移植的代码时，将其放入良好声明和良好的文档接口中。

#### <span id="no_rtti_exception">不要使用 RTTI 或 Exceptions</span>

In an effort to reduce code and executable size, LLVM does not use exceptions or RTTI (runtime type information, for example, dynamic_cast<>).

That said, LLVM does make extensive use of a hand-rolled form of RTTI that use templates like isa<>, cast<>, and dyn_cast<>. This form of RTTI is opt-in and can be added to any class.

为了尽量减小代码和可执行文件的大小，LLVM 未使用异常或者RTTI（运行时信息，如 dynamic_cast<>）。也就是说，LLVM确实使用了大量的手动滚动的RTTI的形式，该形式使用了像 [isa<>，cast<> 和 dyn_cast<>](https://llvm.org/docs/ProgrammersManual.html#isa) 之类的模板。这种形式的RTTI是可选的并且可以[添加至任何类](https://llvm.org/docs/HowToSetUpLLVMStyleRTTI.html)中。

#### <span id="no_static_ctor">不要使用静态构造函数</span>

Static constructors and destructors (e.g., global variables whose types have a constructor or destructor) should not be added to the code base, and should be removed wherever possible.

Globals in different source files are initialized in arbitrary order <https://yosefk.com/c++fqa/ctors.html#fqa-10.12>, making the code more difficult to reason about.

Static constructors have negative impact on launch time of programs that use LLVM as a library. We would really like for there to be zero cost for linking in an additional LLVM target or other library into an application, but static constructors undermine this goal.

静态构造函数和析构函数（如，包含构造和析构函数的全局变量）不应该添加到代码中，并且无论在哪都尽可能的删除它。

全局变量在不同的文件中的初始化顺序是任意的 <https://yosefk.com/c++fqa/ctors.html#fqa-10.12>，这样使代码更加难以推理。

将LLVM作为库使用时，静态构造函数在程序启动的时候具有负面的影响。我们真正希望将额外的LLVM目标或者其它库链接至应用程序中的开销为零，但是静态构造函数违背了这个目标。

#### <span id="class_struct">使用class和struct关键字</span>

In C++, the class and struct keywords can be used almost interchangeably. The only difference is when they are used to declare a class: class makes all members private by default while struct makes all members public by default.
- All declarations and definitions of a given class or struct must use the same keyword. For example:
- struct should be used when all members are declared public.

在C++中，`class` 和 `struct` 关键字在绝大数的情况下可以互换。仅仅的区别在于定义一个 class 时所有的成员默认为 private，而struct默认为 public。
- 给定 `class` 和 `struct` 的所有生命和定义必须使用同一个关键字
    ```cpp
    // Avoid if `Example` is defined as a struct.
    class Example;

    // OK.
    struct Example;

    struct Example { ... };
    ```
- 当所有的成员都为公开声明时使用 `struct`
    ```cpp
    // Avoid using `struct` here, use `class` instead.
    struct Foo {
    private:
      int Data;
    public:
      Foo() : Data(0) { }
      int getData() const { return Data; }
      void setData(int D) { Data = D; }
    };

    // OK to use `struct`: all members are public.
    struct Bar {
      int Data;
      Bar() : Data(0) { }
    };
    ```

#### <span id="no_brace_call_ctor">不要使用花括号初始化列表调用构造函数</span>

Starting from C++11 there is a “generalized initialization syntax” which allows calling constructors using braced initializer lists. Do not use these to call constructors with non-trivial logic or if you care that you’re calling some particular constructor. Those should look like function calls using parentheses rather than like aggregate initialization. Similarly, if you need to explicitly name the type and call its constructor to create a temporary, don’t use a braced initializer list. Instead, use a braced initializer list (without any type for temporaries) when doing aggregate initialization or something notionally equivalent. Examples:

从C++11开始的通用初始化语法，允许使用花括号初始化列表来调用构造函数。调用非平凡逻辑的构造函数或者调用一些特定的构造函数时不要使用这个语法，这些情况应该看起来像函数调用一样使用圆括号而不是聚合初始化。同样的，如果你需要显式化类型名称并且调用其构造函数作为一个临时对象，也不要使用花括号初始化列表。相反的，当执行聚合初始化或等效的概念时，使用花括号初始化列表（无任何临时变量类型）。如：

```cpp
class Foo {
public:
  // Construct a Foo by reading data from the disk in the whizbang format, ...
  Foo(std::string filename);

  // Construct a Foo by looking up the Nth element of some global data ...
  Foo(int N);

  // ...
};

// The Foo constructor call is reading a file, don't use braces to call it.
std::fill(foo.begin(), foo.end(), Foo("name"));

// The pair is being constructed like an aggregate, use braces.
bar_map.insert({my_key, my_value});
```

If you use a braced initializer list when initializing a variable, use an equals before the open curly brace:

当使用初始化列表初始化一个变量时，在开括号之前使用一个等号：
```cpp
int data[] = {0, 1, 2, 3};
```

#### <span id="auto"> 使用auto类型推导提高代码可读性</span>

Some are advocating a policy of “almost always auto” in C++11, however LLVM uses a more moderate stance. Use auto if and only if it makes the code more readable or easier to maintain. Don’t “almost always” use auto, but do use auto with initializers like cast<Foo>(...) or other places where the type is already obvious from the context. Another time when auto works well for these purposes is when the type would have been abstracted away anyways, often behind a container’s typedef such as std::vector<T>::iterator.

Similarly, C++14 adds generic lambda expressions where parameter types can be auto. Use these where you would have used a template.

一些人主张在C++11种几乎总是使用auto的原则，然而LLVM保持更温和的态度。仅在如果使用auto可以提升代码的可读性或者更利于维护的情况下使用。并不几乎总是使用 `auto`，而是在初始化像 `cast<Foo>(...)` 或者类型可以明显的从上下文中获取的地方中使用。其它时候 `auto` 良好用于其用途是无论如何该类型都会被抽象化，经常出现在容器的 `typedef` 之后如 `std::vector<T>::iterator`。

同样的，C++14 新增类型可以为auto的通用lambda表达式，可以在原本使用模板的地方使用它。

#### <span id="beware_auto_copy">小心auto带来的不必要拷贝</span>

The convenience of auto makes it easy to forget that its default behavior is a copy. Particularly in range-based for loops, careless copies are expensive.

Use auto & for values and auto * for pointers unless you need to make a copy.

`auto` 的便利性更容易遗忘它默认是拷贝的行为，尤其是在基于范围的循环，粗心的代价很昂贵。

值使用 `auto &` 指针使用 `auto *` 除非你需要进行拷贝。

```cpp
// Typically there's no reason to copy.
for (const auto &Val : Container) { observe(Val); }
for (auto &Val : Container) { Val.change(); }

// Remove the reference if you really want a new copy.
for (auto Val : Container) { Val.change(); saveSomewhere(Val); }

// Copy pointers, but make it clear that they're pointers.
for (const auto *Ptr : Container) { observe(*Ptr); }
for (auto *Ptr : Container) { Ptr->change(); }
```

#### <span id="beware_order_pointer">小心对指针排序带来的不确定性</span>

In general, there is no relative ordering among pointers. As a result, when unordered containers like sets and maps are used with pointer keys the iteration order is undefined. Hence, iterating such containers may result in non-deterministic code generation. While the generated code might work correctly, non-determinism can make it harder to reproduce bugs and debug the compiler.

In case an ordered result is expected, remember to sort an unordered container before iteration. Or use ordered containers like vector/MapVector/SetVector if you want to iterate pointer keys.

通常而言，指针之间没有相对的顺序。但结果是，当`set`和`map`之类的无序容器使用指针作为key值时，其迭代器的顺序是未定义的。因此，迭代这些容器时可能导致不确定的代码产生。而这些产生的代码可能正常的运行，不确定性很难重现bug和调试编译器。

如果期望结果有序，切记在迭代器之前对无序容器进行排序。或者你想迭代指针的key则使用顺序容器像 `vector/MapVector/SetVector`。

#### <span id="beware_equal_order">小心相同元素不确定的排序顺序</span>

std::sort uses a non-stable sorting algorithm in which the order of equal elements is not guaranteed to be preserved. Thus using std::sort for a container having equal elements may result in non-deterministic behavior. To uncover such instances of non-determinism, LLVM has introduced a new llvm::sort wrapper function. For an EXPENSIVE_CHECKS build this will randomly shuffle the container before sorting. Default to using llvm::sort instead of std::sort.

`std::sort` 使用了一个不稳定的排序算法，相同元素的顺序不能保证被保留下来。因此使用`std::sort`对一个具有相同元素的容器排序时可能出现不确定的行为。为了发现这些不确定的情况，LLVM 引入了一个新的 `llvm::sort` 封装函数。使用 `EXPENSIVE_CHECKS` 编译时在排序之前随机的打乱容器内的顺序。默认使用 `llvm::sort` 代替 `std::sort`。

## <span id="fmt_issue">风格问题</span>

### <span id="high_level_issue">上层问题</span>

#### <span id="header_self_include">头文件独立</span>

Header files should be self-contained (compile on their own) and end in .h. Non-header files that are meant for inclusion should end in .inc and be used sparingly.

All header files should be self-contained. Users and refactoring tools should not have to adhere to special conditions to include the header. Specifically, a header should have header guards and include all other headers it needs.

There are rare cases where a file designed to be included is not self-contained. These are typically intended to be included at unusual locations, such as the middle of another file. They might not use header guards, and might not include their prerequisites. Name such files with the .inc extension. Use sparingly, and prefer self-contained headers when possible.

In general, a header should be implemented by one or more .cpp files. Each of these .cpp files should include the header that defines their interface first. This ensures that all of the dependences of the header have been properly added to the header itself, and are not implicit. System headers should be included after user headers for a translation unit.

头文件应该保持独立（独立编译）并且以`.h`结尾。非头文件的包含应该以`.inc`结尾并且谨慎使用。

所有的头文件都应该是独立的。用户或者重构工具不应该强制附加特定条件才能够包含这个头文件。特别的，一个头文件应该存在头文件保护和所有需要的其他头文件。

有一些及少见的情况，设计被包含的文件不是独立的。它们往往有意的被包含在不常用的地方，比如一个文件的中间位置。它们可能不使用头文件保护并且可能不包括必要的先决条件。这些文件的名称以 `.inc` 扩展。谨慎使用这种文件，尽可能使用独立的头文件。

通常而言，一个头文件应该被一个或多个 `.cpp` 实现。每一个 `.cpp` 文件都应该首先包含定义接口的头文件。确保头文件所有的依赖都能够显式正确的添加至该头文件中。

#### <span id="lib_layering">库层次</span>

A directory of header files (for example include/llvm/Foo) defines a library (Foo). Dependencies between libraries are defined by the LLVMBuild.txt file in their implementation (lib/Foo). One library (both its headers and implementation) should only use things from the libraries listed in its dependencies.

Some of this constraint can be enforced by classic Unix linkers (Mac & Windows linkers, as well as lld, do not enforce this constraint). A Unix linker searches left to right through the libraries specified on its command line and never revisits a library. In this way, no circular dependencies between libraries can exist.

This doesn’t fully enforce all inter-library dependencies, and importantly doesn’t enforce header file circular dependencies created by inline functions. A good way to answer the “is this layered correctly” would be to consider whether a Unix linker would succeed at linking the program if all inline functions were defined out-of-line. (& for all valid orderings of dependencies - since linking resolution is linear, it’s possible that some implicit dependencies can sneak through: A depends on B and C, so valid orderings are “C B A” or “B C A”, in both cases the explicit dependencies come before their use. But in the first case, B could still link successfully if it implicitly depended on C, or the opposite in the second case)

头文件目录（如 include/llvm/Foo）定义了库（Foo）。库之间的依赖在它们实现（lib/Foo）中的 LLVMBuild.txt 定义。一个库应该仅使用依赖所列出库的内容。

一些经典的Unix链接器（Mac & Windows 链接器，比如lld，不强制执行）可以强制执行某些约束。Unix 链接器从左往右在命令行中搜索特定的库并且不会重复访问同一个库。这种情况下，库之间就不存在循环依赖关系。

这样不会完全强制的执行所有的相互库依赖，且重要的是不会执行由内联函数带来的头文件循环依赖。回答是否正确分层的一个好方法是判断 Unix 链接器是否可以正确链接使用非内联函数代替内联函数的程序。（对于所有依赖的有效顺序 - 由于链接的方案是线性的，因此可能潜伏一些隐式的依赖：A 依赖于B和C，所以有效的顺序为“CBA” 或者“BCA”，两种显示的依赖都在使用之前出现。但对于第一种情况，B如果隐式的依赖于C仍然可以链接成功，或者在第二种情况，相反的依赖也可以链接成功）

#### <span id="little_include">尽可能的减少 #include</span>

`#include` hurts compile time performance. Don’t do it unless you have to, especially in header files.

But wait! Sometimes you need to have the definition of a class to use it, or to inherit from it. In these cases go ahead and #include that header file. Be aware however that there are many cases where you don’t need to have the full definition of a class. If you are using a pointer or reference to a class, you don’t need the header file. If you are simply returning a class instance from a prototyped function or method, you don’t need it. In fact, for most cases, you simply don’t need the definition of a class. And not #includeing speeds up compilation.

It is easy to try to go too overboard on this recommendation, however. You must include all of the header files that you are using — you can include them either directly or indirectly through another header file. To make sure that you don’t accidentally forget to include a header file in your module header, make sure to include your module header first in the implementation file (as mentioned above). This way there won’t be any hidden dependencies that you’ll find out about later.

`#include` 降低了编译的性能，在不是必需的时候不要包含，尤其是在头文件中。

有一些情况是你需要获取到类的定义再去使用或者继承它，这些情况则在文件的首部进行 `#include`。然而也要想到，存在非常多的情况是不需要拥有类的完整定义的。如果正在使用类指针或引用，则不必包含该头文件。如果只是的从原型函数或者方法中返回一个类的实例，也不需要头文件。实际上，在大多数的情况下，你根本不需要类的定义。不进行 `#include` 可以加速编译。

这个建议很容易导致偏激的态度，然而，你**必须**包含所有正在使用的头文件，无论是直接还是间接从其它文件包含。为确保你不会意外的忘记在你的模块头文件中包含头文件，确认在实现文件中**第一个**包含你的模块头文件（就像上面提到的）。这样，你后面就会发现不会再有隐藏的依赖了。

#### <span id="intenal_header">保持私有的内部头文件</span>

Many modules have a complex implementation that causes them to use more than one implementation (.cpp) file. It is often tempting to put the internal communication interface (helper classes, extra functions, etc) in the public module header file. Don’t do this!

If you really need to do something like this, put a private header file in the same directory as the source files, and include it locally. This ensures that your private interface remains private and undisturbed by outsiders.

> It’s okay to put extra implementation methods in a public class itself. Just make them private (or protected) and all is well.

很多模块实现非常复杂导致使用了多个实现文件（`.cpp`文件）。不要尝试将公共交互接口（帮助类，扩展函数等）放入到公共的模块头文件中。

如果你真的需要这么做的话，在相同的目录下放入一个私有的头文件让源文件局部包含。这样确保你的私有接口保留了私有属性并且不向外发布。

> 允许将扩展实现的方法放入到一个公共类自身中，但是需要让它们设为私有（保护），一切就正常。

#### <span id="namespace_impl">使用 namespace 限定符莱实现前置声明的函数</span>

When providing an out of line implementation of a function in a source file, do not open namespace blocks in the source file. Instead, use namespace qualifiers to help ensure that your definition matches an existing declaration. Do this:

当源文件中提供一个非内联（或者叫外部实现）的函数实现时，不要在源文件中打开 namespace 块（namespace xx {}）。相反的，使用 namespace 标记符来帮助确保你的定义匹配上已经存在的声明。像这样：
```cpp
// Foo.h
namespace llvm {
int foo(const char *s);
}

// Foo.cpp
#include "Foo.h"
using namespace llvm;
int llvm::foo(const char *s) {
  // ...
}
```

Doing this helps to avoid bugs where the definition does not match the declaration from the header. For example, the following C++ code defines a new overload of llvm::foo instead of providing a definition for the existing function declared in the header:

这样做能够避免实现不能匹配头文件中声明的bug。举个例子，下面的C++代码定义了一个新的重载`llvm::foo`来取代在头文件中存在函数声明的实现：
```cpp
// Foo.cpp
#include "Foo.h"
namespace llvm {
int foo(char *s) { // Mismatch between "const char *" and "char *"
}
} // end namespace llvm
```

This error will not be caught until the build is nearly complete, when the linker fails to find a definition for any uses of the original function. If the function were instead defined with a namespace qualifier, the error would have been caught immediately when the definition was compiled.

Class method implementations must already name the class and new overloads cannot be introduced out of line, so this recommendation does not apply to them.

这个错误在快构建完成之前不能够被捕获，直到产生函数调用找不到函数定义的链接错误。如果这个函数使用namespace标记符来代替，这个错误直接在这个定义编译时被捕获。

类方法实现必须类已经被命名并且新的重载不能够引入到外部，所以这个建议对它们不起作用。

#### <span id="early_exit_continue">提前退出或 continue 以简化代码</span>

When reading code, keep in mind how much state and how many previous decisions have to be remembered by the reader to understand a block of code. Aim to reduce indentation where possible when it doesn’t make it more difficult to understand the code. One great way to do this is by making use of early exits and the continue keyword in long loops. Consider this code that does not use an early exit:

在阅读代码时，读者需要在记住有多少状态及前置条件去阅读代码块。为了减少尽可能缩进而又不是代码变得更难理解，在长循环中使用提前退出或者`continue`关键字是一个好的方法。观察未使用提前退出的代码：
```cpp
Value *doSomething(Instruction *I) {
  if (!I->isTerminator() &&
      I->hasOneUse() && doOtherThing(I)) {
    ... some long code ....
  }

  return 0;
}
```

This code has several problems if the body of the 'if' is large. When you’re looking at the top of the function, it isn’t immediately clear that this only does interesting things with non-terminator instructions, and only applies to things with the other predicates. Second, it is relatively difficult to describe (in comments) why these predicates are important because the if statement makes it difficult to lay out the comments. Third, when you’re deep within the body of the code, it is indented an extra level. Finally, when reading the top of the function, it isn’t clear what the result is if the predicate isn’t true; you have to read to the end of the function to know that it returns null.

It is much preferred to format the code like this:

这份代码当`if`作用体非常庞大时有一些问题。当你正在查看这个函数的顶部时，对于是只做和结束指令相关的事情还是其他的判断操作是不够清晰的。其二，由于if语句使注释难以布局，相对的也更难以去描述这个判断为何很重要。其三，当你深入代码时，它已经被缩进了一级。最后，当阅读函数的顶部代码时，if判断的结构是不是为真的结果不够清晰；你必须要阅读到函数的底端才知道返回了 null。

更好的代码格式如下：
```cpp
Value *doSomething(Instruction *I) {
  // Terminators never need 'something' done to them because ...
  if (I->isTerminator())
    return 0;

  // We conservatively avoid transforming instructions with multiple uses
  // because goats like cheese.
  if (!I->hasOneUse())
    return 0;

  // This is really just here for example.
  if (!doOtherThing(I))
    return 0;

  ... some long code ....
}
```

This fixes these problems. A similar problem frequently happens in for loops. A silly example is something like this:

这修复了上面提到的那些问题。一个类似的问题频繁的出现在`for`循环中，一个简单的例子如下：

```cpp
for (Instruction &I : BB) {
  if (auto *BO = dyn_cast<BinaryOperator>(&I)) {
    Value *LHS = BO->getOperand(0);
    Value *RHS = BO->getOperand(1);
    if (LHS != RHS) {
      ...
    }
  }
}
```

When you have very, very small loops, this sort of structure is fine. But if it exceeds more than 10-15 lines, it becomes difficult for people to read and understand at a glance. The problem with this sort of code is that it gets very nested very quickly. Meaning that the reader of the code has to keep a lot of context in their brain to remember what is going immediately on in the loop, because they don’t know if/when the if conditions will have elses etc. It is strongly preferred to structure the loop like this:

当你的循环非常非常简短的循环时，这种结构是良好的。但是当循环超过10-15行时，用户在扫视时会变得难以阅读和理解。这种代码存在的问题是缩进的非常的快，这意味着读者脑袋必须要保持大量的上下文，来记住在这个循环中随即发生的事情，因为他们不知道`if`条件是否或者何时存在 `else`。更强烈推荐的循环结构如下：

```cpp
for (Instruction &I : BB) {
  auto *BO = dyn_cast<BinaryOperator>(&I);
  if (!BO) continue;

  Value *LHS = BO->getOperand(0);
  Value *RHS = BO->getOperand(1);
  if (LHS == RHS) continue;

  ...
}
```

This has all the benefits of using early exits for functions: it reduces nesting of the loop, it makes it easier to describe why the conditions are true, and it makes it obvious to the reader that there is no else coming up that they have to push context into their brain for. If a loop is large, this can be a big understandability win.

在函数中使用尽快退出的好处是：减少循环的缩进，更易于描述为何条件为真，而没有 `else` 出现则在读者的脑袋中变得更加明显。如果一个循环非常的庞大，这些做法具有巨大的理解优势。

#### <span id="no_else_after_return">return 之后不要使用 else</span>

For similar reasons as above (reduction of indentation and easier reading), please do not use 'else' or 'else if' after something that interrupts control flow — like return, break, continue, goto, etc. For example:

同样的理由如上（减少缩进和易于理解），请不要使用 `else`或者`else if`在终止的控制流之后，就像是 `return`, `break`, `continue`, `goto` 等等，如：

```cpp
case 'J': {
  if (Signed) {
    Type = Context.getsigjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_sigjmp_buf;
      return QualType();
    } else {
      break; // Unnecessary.
    }
  } else {
    Type = Context.getjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_jmp_buf;
      return QualType();
    } else {
      break; // Unnecessary.
    }
  }
}
```

更好的做法如下：
```cpp
case 'J':
  if (Signed) {
    Type = Context.getsigjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_sigjmp_buf;
      return QualType();
    }
  } else {
    Type = Context.getjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_jmp_buf;
      return QualType();
    }
  }
  break;
```

这个例子更好的写法如下：
```cpp
case 'J':
  if (Signed)
    Type = Context.getsigjmp_bufType();
  else
    Type = Context.getjmp_bufType();

  if (Type.isNull()) {
    Error = Signed ? ASTContext::GE_Missing_sigjmp_buf :
                     ASTContext::GE_Missing_jmp_buf;
    return QualType();
  }
  break;
```

The idea is to reduce indentation and the amount of code you have to keep track of when reading the code.

这样做的目的是减少缩进和在阅读代码时需要跟踪的代码量。

#### <span id="predicate_loop_func">将判断循环转换为判断函数</span>

It is very common to write small loops that just compute a boolean value. There are a number of ways that people commonly write these, but an example of this sort of thing is:

通过短循环计算一个 boolean 值是非常常见的写法。有一系列的常见方式，比如这种：
```cpp
bool FoundFoo = false;
for (unsigned I = 0, E = BarList.size(); I != E; ++I)
  if (BarList[I]->isFoo()) {
    FoundFoo = true;
    break;
  }

if (FoundFoo) {
  ...
}
```

Instead of this sort of loop, we prefer to use a predicate function (which may be static) that uses early exits:

相对这种循环的方式，我们更偏向使用一个使用尽早退出的判断函数（可能是静态）：
```cpp
/// \returns true if the specified list has an element that is a foo.
static bool containsFoo(const std::vector<Bar*> &List) {
  for (unsigned I = 0, E = List.size(); I != E; ++I)
    if (List[I]->isFoo())
      return true;
  return false;
}
...

if (containsFoo(BarList)) {
  ...
}
```

There are many reasons for doing this: it reduces indentation and factors out code which can often be shared by other code that checks for the same predicate. More importantly, it forces you to pick a name for the function, and forces you to write a comment for it. In this silly example, this doesn’t add much value. However, if the condition is complex, this can make it a lot easier for the reader to understand the code that queries for this predicate. Instead of being faced with the in-line details of how we check to see if the BarList contains a foo, we can trust the function name and continue reading with better locality.

这样做有很多理由：减少了缩进并且分离出可以让其它代码检测相同判断的共享代码。更重要的是，让你强制为这个函数起一个名字，并且强制让你再为它写上注释。在这个简短的例子中没有添加很多值，然而如果这个if条件非常复杂，读者通过这个判断可以非常容易地理解代码。而不是一开始就面对如何检测 `BarList` 中是否包含了 `foo` 的内联细节，我们可以信任这个函数名称并且继续在更好的位置进行阅读。

### <span id="low_level_issue">底层问题</span>

#### <span id="name">合适的类型、函数、变量和枚举命名</span>

Poorly-chosen names can mislead the reader and cause bugs. We cannot stress enough how important it is to use descriptive names. Pick names that match the semantics and role of the underlying entities, within reason. Avoid abbreviations unless they are well known. After picking a good name, make sure to use consistent capitalization for the name, as inconsistency requires clients to either memorize the APIs or to look it up to find the exact spelling.

In general, names should be in camel case (e.g. TextFileReader and isLValue()). Different kinds of declarations have different rules:
- **Type names** (including classes, structs, enums, typedefs, etc) should be nouns and start with an upper-case letter (e.g. TextFileReader).
- **Variable names** should be nouns (as they represent state). The name should be camel case, and start with an upper case letter (e.g. Leader or Boats).
- **Function names** should be verb phrases (as they represent actions), and command-like function should be imperative. The name should be camel case, and start with a lower case letter (e.g. openFile() or isFoo()).
- **Enum declarations** (e.g. enum Foo {...}) are types, so they should follow the naming conventions for types. A common use for enums is as a discriminator for a union, or an indicator of a subclass. When an enum is used for something like this, it should have a Kind suffix (e.g. ValueKind).
- **Enumerators** (e.g. enum { Foo, Bar }) and public member variables should start with an upper-case letter, just like types. Unless the enumerators are defined in their own small namespace or inside a class, enumerators should have a prefix corresponding to the enum declaration name. For example, enum ValueKind { ... }; may contain enumerators like VK_Argument, VK_BasicBlock, etc. Enumerators that are just convenience constants are exempt from the requirement for a prefix. For instance:

选择糟糕的命名会误导读者和造成bug。我们怎么强调*描述性*名称的重要性都不足为过。在合理的范围内选择语义和基本实体规则相匹配的命名。避免缩写除非约定成俗。在取好一个名字之后，确保为命名使用一致的大写，因为不一致的大写需要客户端去记忆这个API或者查找准确的拼写。

通常而言，命名应该使用驼峰式（如 `TextFileReader` 和 `isLValue()`）。不同种类的声明有不同的规则：
- **类型命名** （包括 classes，structs，enums，typedef 等）应该是动词并且以大写字母开头（如 TextFileReader）。
- **变量命名** 应该是名词（代表状态）。命名应该是大写字母开头的驼峰大小写（如 `Leader` 或 `Boats`）。
- **函数命名** 应该是动词短语（代表动作）并且命令式的函数应该是祈使语气的。命名应该是小写字母开头的驼峰写法（如 `openFile()` 或 `isFoo()`）。
- **枚举声明** 如（`emum { ... }`）是类型，所以应该遵循类型的命名规则。枚举的常见用法是作为`union`的区分符或内部类的指示符。当一个枚举这样使用的时候应该增加一个 `Kind` 后缀（如 `ValueKind`）。
- **枚举成员**（如 `enum {Foo, Bar}`）和**公开成员变量**像类型一样，以大写开头。除非枚举成员是在它自己的小命名空间或内部类中定义，否则应该有一个枚举声明名称对应的前缀。例如， `enum ValueKind {...};` 可能包含枚举成员像 `VK_Argument`，`VK_BasicBlock` 等等。枚举变量作为便利的常量可以不需要前缀，举个栗子：
    ```cpp
    enum {
      MaxSize = 42,
      Density = 12
    };
    ```

As an exception, classes that mimic STL classes can have member names in STL’s style of lower-case words separated by underscores (e.g. begin(), push_back(), and empty()). Classes that provide multiple iterators should add a singular prefix to begin() and end() (e.g. global_begin() and use_begin()).

Here are some examples:

作为例外，模仿的STL类可以拥有以STL式的小写字母下划线分隔命名的成员。（如：`begin()`, `push_back()` 和 `empty()`）。提供多个迭代器的类应该为`begin()`和`end()`添加一个单数前缀（如 `global_begin()`和`use_begin()`）。

一个简单的例子：
```cpp
class VehicleMaker {
  ...
  Factory<Tire> F;            // Avoid: a non-descriptive abbreviation.
  Factory<Tire> Factory;      // Better: more descriptive.
  Factory<Tire> TireFactory;  // Even better: if VehicleMaker has more than one
                              // kind of factories.
};

Vehicle makeVehicle(VehicleType Type) {
  VehicleMaker M;                         // Might be OK if scope is small.
  Tire Tmp1 = M.makeTire();               // Avoid: 'Tmp1' provides no information.
  Light Headlight = M.makeLight("head");  // Good: descriptive.
  ...
}
```

#### <span id="assert">善用断言</span>

Use the “assert” macro to its fullest. Check all of your preconditions and assumptions, you never know when a bug (not necessarily even yours) might be caught early by an assertion, which reduces debugging time dramatically. The “<cassert>” header file is probably already included by the header files you are using, so it doesn’t cost anything to use it.

To further assist with debugging, make sure to put some kind of error message in the assertion statement, which is printed if the assertion is tripped. This helps the poor debugger make sense of why an assertion is being made and enforced, and hopefully what to do about it. Here is one complete example:

充分的使用`assert`宏。检测你所有的前置条件和假设，你永远不知道的一个bug（甚至不是你的）可能被及早地捕获，而显著的减少调试的时间。`<cassert>` 头文件可能已经被其它在使用的头文件包含，所以你可以不花费任何代价去使用它。

为了进一步的协助调试，确保将一些种类的错误消息放入到断言语句中，当断言被触发时消息就被打印出来。这样可以帮助弱鸡调试器理解断言是为何存在和执行的，并且所期望做的事情。这里有一个完整的示例：
```cpp
inline Value *getOperand(unsigned I) {
  assert(I < Operands.size() && "getOperand() out of range!");
  return Operands[I];
}
```

一些更多的示例：
```cpp
assert(Ty->isPointerType() && "Can't allocate a non-pointer type!");

assert((Opcode == Shl || Opcode == Shr) && "ShiftInst Opcode invalid!");

assert(idx < getNumSuccessors() && "Successor # out of range!");

assert(V1.getType() == V2.getType() && "Constant types must be identical!");

assert(isa<PHINode>(Succ->front()) && "Only works on PHId BBs!");
```

In the past, asserts were used to indicate a piece of code that should not be reached. These were typically of the form:

在过去，断言惯用于指示一小段不应该被达成的代码。一个典型的形式如下：
```cpp
assert(0 && "Invalid radix for integer literal");
```

This has a few issues, the main one being that some compilers might not understand the assertion, or warn about a missing return in builds where assertions are compiled out.

Today, we have something much better: llvm_unreachable:

这存在着一些问题，最主要的一个是一些编译器可能不理解该断言，或者在编译完成构建断言时发出了缺少返回的警告。现在，我们有一个更好的东西： `llvm_unreachable`:
```cpp
llvm_unreachable("Invalid radix for integer literal");
```

When assertions are enabled, this will print the message if it’s ever reached and then exit the program. When assertions are disabled (i.e. in release builds), llvm_unreachable becomes a hint to compilers to skip generating code for this branch. If the compiler does not support this, it will fall back to the “abort” implementation.

Use llvm_unreachable to mark a specific point in code that should never be reached. This is especially desirable for addressing warnings about unreachable branches, etc., but can be used whenever reaching a particular code path is unconditionally a bug (not originating from user input; see below) of some kind. Use of assert should always include a testable predicate (as opposed to assert(false)).

Neither assertions or llvm_unreachable will abort the program on a release build. If the error condition can be triggered by user input then the recoverable error mechanism described in LLVM Programmer’s Manual should be used instead. In cases where this is not practical, report_fatal_error may be used.

Another issue is that values used only by assertions will produce an “unused value” warning when assertions are disabled. For example, this code will warn:

当断言被激活时，如果断言曾被达成则将打印这条消息然后退出程序。当断言被禁用时（如release构建），`llvm_unreachable` 成为编译器为该分支忽略生成代码的指引。当编译器不支持该实现时，将退化成 `abort`的实现。

使用 `llvm_unreachable` 在代码中标记一个应该永远不会达成的特定的点。这对于解决有关不可达到的分支等的警告非常理想，而在特定路径到达是无条件的错误情况下使用就是某种bug（不是源自用户输入，见下）。`assert`的使用总是应该包含一个可以测试的判断（与 `assert(false)` 相反）。

其它的问题是当断言被禁用时，仅在断言中使用的值将产生一个“值未使用”的警告。举个例子，这些代码将发出警告：
```cpp
unsigned Size = V.size();
assert(Size > 42 && "Vector smaller than it should be");

bool NewToSet = Myset.insert(Value);
assert(NewToSet && "The value shouldn't be in the set yet");
```

These are two interesting different cases. In the first case, the call to V.size() is only useful for the assert, and we don’t want it executed when assertions are disabled. Code like this should move the call into the assert itself. In the second case, the side effects of the call must happen whether the assert is enabled or not. In this case, the value should be cast to void to disable the warning. To be specific, it is preferred to write the code like this:

代码有两种有趣的情况。在第一种情况中，`V.size()` 调用仅仅在断言时有效，并且我们不让该调用在断言被禁用的时候执行，这种情况下我们应该将这个调用移入到这个断言内。在第二种情况中，调用的副作用一定会发生无论这个断言禁用与否。将值转换成`void`来禁用这个警告。具体来说，更好的写法如下：
```cpp
assert(V.size() > 42 && "Vector smaller than it should be");

bool NewToSet = Myset.insert(Value); (void)NewToSet;
assert(NewToSet && "The value shouldn't be in the set yet");
```

#### <span id="no_namespace_std">不要使用 using namesapce std</span>

In LLVM, we prefer to explicitly prefix all identifiers from the standard namespace with an “std::” prefix, rather than rely on “using namespace std;”.

In header files, adding a 'using namespace XXX' directive pollutes the namespace of any source file that #includes the header, creating maintenance issues.

In implementation files (e.g. .cpp files), the rule is more of a stylistic rule, but is still important. Basically, using explicit namespace prefixes makes the code clearer, because it is immediately obvious what facilities are being used and where they are coming from. And more portable, because namespace clashes cannot occur between LLVM code and other namespaces. The portability rule is important because different standard library implementations expose different symbols (potentially ones they shouldn’t), and future revisions to the C++ standard will add more symbols to the std namespace. As such, we never use 'using namespace std;' in LLVM.

The exception to the general rule (i.e. it’s not an exception for the std namespace) is for implementation files. For example, all of the code in the LLVM project implements code that lives in the ‘llvm’ namespace. As such, it is ok, and actually clearer, for the .cpp files to have a 'using namespace llvm;' directive at the top, after the #includes. This reduces indentation in the body of the file for source editors that indent based on braces, and keeps the conceptual context cleaner. The general form of this rule is that any .cpp file that implements code in any namespace may use that namespace (and its parents’), but should not use any others.

在LLVM中，我们更推荐显式的在标准命名空间内的标识符添加前缀`std::`，而不是依靠`using namespace std;`。

在头文件中，添加一个"using namespace XXX"指令污染所有包含这个头文件的源文件的命名空间，将造成维护的问题。

在实现的文件中（如 `.cpp`文件），这个规则更多的是格式化的规则，但是仍然很重要。基本的，显式的使用命名空间前缀让代码变得清晰，因为使用了什么设备和来自哪里的都显而易见。并且更具移植性，因为命名空间冲突不可能发生在LLVM和其它命名空间之间。这个可移植性的规则是非常重要的因为不同的标准库实现暴露不同的符号（潜台词是不应该这样），并且向C++标准添加新特性修订版将在标准命名空间内添加更多的符号。所以，我们永远不要使用 'using namespace std;' 在LLVM中。

一般规则的例外（`std`命名空间不是例外）是实现文件。打个比方，LLVM项目中所有实现的代码都位于llvm命名空间中。所以在`.cpp`文件include之后使用 `using namespace std;`指令是可行的，且实际上更清晰。在基于花括号缩进的源码编辑器可以减少主体的缩进，并且让上下文在概念上更加清晰。此规则的一般形式是，在任何命名空间内实现代码的任何`.cpp`文件都可以使用该命名空间（及其父命名空间），但不应该使用其它的命名空间。

#### <span id="virtual_method">为头文件中的类提供虚函数锚</span>

If a class is defined in a header file and has a vtable (either it has virtual methods or it derives from classes with virtual methods), it must always have at least one out-of-line virtual method in the class. Without this, the compiler will copy the vtable and RTTI into every .o file that #includes the header, bloating .o file sizes and increasing link times.

如果一个定义在头文件内的类存在虚表（含有虚表或者从有虚表继承），在类中必须总是至少含有一个非内联的虚方法。否则，编译器将复制 vtable 和 RTTI 信息到每一个包含该头文件中的 `.o` 文件中，导致 `.o` 文件大小膨胀并且增加链接耗时。

#### <span id="no_default_in_switch">不要在全覆盖枚举的switch 中使用 default</span>

-Wswitch warns if a switch, without a default label, over an enumeration does not cover every enumeration value. If you write a default label on a fully covered switch over an enumeration then the -Wswitch warning won’t fire when new elements are added to that enumeration. To help avoid adding these kinds of defaults, Clang has the warning -Wcovered-switch-default which is off by default but turned on when building LLVM with a version of Clang that supports the warning.

A knock-on effect of this stylistic requirement is that when building LLVM with GCC you may get warnings related to “control may reach end of non-void function” if you return from each case of a covered switch-over-enum because GCC assumes that the enum expression may take any representable value, not just those of individual enumerators. To suppress this warning, use llvm_unreachable after the switch.

`-Wswitch` 在 switch 没有默认标签并且枚举没有完全覆盖枚举值情况下发出警告。如果在全覆盖枚举的switch中写了默认的标签，那么以后再添加枚举值得时候 `-Wswitch`将不会再发出警告。为了避免添加此类默认的枚举，Clang 可以使用 `-Wcoverd-switch-default` 选项发出警告，默认情况下保持关闭但是使用支持该选项的 Clang 构建 LLVM 时被打开。

这种风格上的需求带来一系列连锁反应是当使用 GCC 编译 LLVM 时，如果你从每种枚举都覆盖的case中返回，可能得到 "control may reach end of non-void function" 相关的警告，这是因为 GCC 假设枚举表达式可以取任何可以表达的值，而不仅仅是那些独立的枚举值。在switch后使用 `llvm_unreachable`来消除这些警告。（感觉有点像是在黑GCC）

#### <span id="range_based_loop">尽可能的使用基于范围的循环</span>

The introduction of range-based for loops in C++11 means that explicit manipulation of iterators is rarely necessary. We use range-based for loops wherever possible for all newly added code. For example:

C++11引入的基于范围的`for`循环意味着显式对迭代器的操作不是必要的。我们尽可能地在新添加的代码中使用基于范围的`for`循环。如：

```cpp
BasicBlock *BB = ...
for (Instruction &I : *BB)
  ... use I ...
```

#### <span id="eval_end">不要每次通过循环计算 end()</span>

In cases where range-based for loops can’t be used and it is necessary to write an explicit iterator-based loop, pay close attention to whether end() is re-evaluated on each loop iteration. One common mistake is to write a loop in this style:

如果不能使用基于范围的`for`循环，则有必要显式的写出基于迭代器的的循环，并且注意`end()`是否在每个循环中都重新被计算。一个常见的错误循环格式如下：
```cpp
BasicBlock *BB = ...
for (auto I = BB->begin(); I != BB->end(); ++I)
  ... use I ...
```

The problem with this construct is that it evaluates “BB->end()” every time through the loop. Instead of writing the loop like this, we strongly prefer loops to be written so that they evaluate it once before the loop starts. A convenient way to do this is like so:

这个结构的问题式每次都计算了`BB->end()`，这个过程贯穿了整个循环。与这种写法相反，我们强烈推荐的循环写法是在循环开始计算一次，一个方便的方法如下：
```cpp
BasicBlock *BB = ...
for (auto I = BB->begin(), E = BB->end(); I != E; ++I)
  ... use I ...
```

The observant may quickly point out that these two loops may have different semantics: if the container (a basic block in this case) is being mutated, then “BB->end()” may change its value every time through the loop and the second loop may not in fact be correct. If you actually do depend on this behavior, please write the loop in the first form and add a comment indicating that you did it intentionally.

Why do we prefer the second form (when correct)? Writing the loop in the first form has two problems. First it may be less efficient than evaluating it at the start of the loop. In this case, the cost is probably minor — a few extra loads every time through the loop. However, if the base expression is more complex, then the cost can rise quickly. I’ve seen loops where the end expression was actually something like: “SomeMap[X]->end()” and map lookups really aren’t cheap. By writing it in the second form consistently, you eliminate the issue entirely and don’t even have to think about it.

The second (even bigger) issue is that writing the loop in the first form hints to the reader that the loop is mutating the container (a fact that a comment would handily confirm!). If you write the loop in the second form, it is immediately obvious without even looking at the body of the loop that the container isn’t being modified, which makes it easier to read the code and understand what it does.

While the second form of the loop is a few extra keystrokes, we do strongly prefer it.

可以迅速察觉出来两个循环可能有不同的语义：如果一个容器（一个基本的代码块）正在被改变，然后`BB->end()`可能每次经过循环时都改变它的值，并且第二次循环可能实际上不正确。如果真正依赖这种行为，那么使用第一种循环的形式，并且添加注释来说明这样做是有意的。

为什么我们更推荐第二种形式（正确的时候）？第一种循环写法的形式有两个问题。第一种是它可能比在循环开始计算更为低效。这种情况，消耗可能较小，每次循环有一些额外的加载操作。然而当基础表达式更为复杂时，消耗将直线上升。见过的一些end表达式实际上是类似 `SomeMap[X]->end()`并且map查找消耗相当高。坚持使用第二种形式，你完全可以消除这种问题，甚至都不需要去考虑这种情况。

第二种（甚至更大）的问题是第一种形式的循环暗示读者循环正在改变容器（注释手动确定的事实下）。如果你写了第二种形式的循环，甚至在不观察循环体的情况下，容器不发生改变也是显而易见的，这样更利于代码阅读和理解做了什么。

即使第二种循环的形式有一些额外的按键，我们依然强烈推荐。

#### <span id="forbidden_iostream">禁止包含 iostream</span>

The use of #include `<iostream>` in library files is hereby forbidden, because many common implementations transparently inject a static constructor into every translation unit that includes it.

Note that using the other stream headers (`<sstream>` for example) is not problematic in this regard — just `<iostream>`. However, raw_ostream provides various APIs that are better performing for almost every use than std::ostream style APIs.

**禁止**在库中使用 `iostream` 头文件，因为很多实现隐含的（透明）将静态构造函数引入每一个包含这个头文件的编译单元中。

注意，在这种考虑下使用其它流的头文件（比如`sstream`）是没有问题的，就 `iostream` 有问题。然而 `raw_ostream` 提供了多种APIs，他们几乎在各种用途上都比`std::ostream`样式的APIs在性能上都更强。

> 新代码应该总是使用 raw_ostream 来写文件，或者 llvm::MemoryBuffer 来读文件。

#### <span id="raw_stream">使用 raw_stream</span>

LLVM includes a lightweight, simple, and efficient stream implementation in llvm/Support/raw_ostream.h, which provides all of the common features of std::ostream. All new code should use raw_ostream instead of ostream.

Unlike std::ostream, raw_ostream is not a template and can be forward declared as class raw_ostream. Public headers should generally not include the raw_ostream header, but use forward declarations and constant references to raw_ostream instances.

LLVM 包含一个轻量的，简单和高效的流实现，位于 `llvm/Support/raw_ostream.h`，并且提供了所有关于`std::ostream`的通用特性。所有新的代码应该使用`raw_ostream` 而不是 `ostream`。

和 `std::ostream` 不同，`raw_ostream` 不是模板并且作为 `class raw_ostream` 可以被前向声明。公共头文件应该不包含该 raw_ostream 头文件，而是应该使用前向声明和常量`raw_ostream`实例。

#### <span id="avoid_std_endl">避免 std::endl</span>

The `std::endl` modifier, when used with `iostreams` outputs a newline to the output stream specified. In addition to doing this, however, it also flushes the output stream. In other words, these are equivalent:

`std::endl` 修饰符与 `iostream` 一起使用，像特定的输出流中输出一个换行。这样做之外，然而它还会对输出流进行 `flush` 操作，换句话说，它们是等效的：

```cpp
std::cout << std::endl;
std::cout << '\n' << std::flush;
```

Most of the time, you probably have no reason to flush the output stream, so it’s better to use a literal '\n'.

大多数情况下，你可能没有理由去 flush 输出流，所以更好的做法使用字面量 `'\n'`.

#### <span id="no_class_inline">不要在class中声明的函数中使用 inline</span>

A member function defined in a class definition is implicitly inline, so don’t put the inline keyword in this case.

定义在类中的成员函数实现是隐式内敛的，所以这种情况下不要使用 `inline` 关键字：

不要这样做：
```cpp
class Foo {
public:
  inline void bar() {
    // ...
  }
};
```

而是这样：
```cpp
class Foo {
public:
  void bar() {
    // ...
  }
};
```

### <span id="micro_details">微观细节</span>

This section describes preferred low-level formatting guidelines along with reasoning on why we prefer them.

这个部分描述了推荐低级的格式化规则及推荐它们的理由。

#### <span id="space_before_parentheses">括号之前的空格</span>

Put a space before an open parenthesis only in control flow statements, but not in normal function call expressions and function-like macros. For example:

在控制流语句的开括号前添加空格，而在一般的函数调用表达式或者类函数的宏前这样做。举个例子：

```cpp
if (X) ...
for (I = 0; I != 100; ++I) ...
while (LLVMRocks) ...

somefunc(42);
assert(3 != 4 && "laws of math are failing me");

A = foo(42, 92) + bar(X);
```

The reason for doing this is not completely arbitrary. This style makes control flow operators stand out more, and makes expressions flow better.

这样做的理由并不是任意的，这种格式让控制流操作符更加突出并且让表达式更流畅。

#### <span id="preincrement">推荐前置自增</span>

Hard fast rule: Preincrement (++X) may be no slower than postincrement (X++) and could very well be a lot faster than it. Use preincrementation whenever possible.

The semantics of postincrement include making a copy of the value being incremented, returning it, and then preincrementing the “work value”. For primitive types, this isn’t a big deal. But for iterators, it can be a huge issue (for example, some iterators contains stack and set objects in them… copying an iterator could invoke the copy ctor’s of these as well). In general, get in the habit of always using preincrement, and you won’t have a problem.

固定规则：前置自增（++X）可能不比后置自增（X++）慢并且很可能比后置自增快得多。无论何时京可能使用前置自增。

后置自增的语义上包含拷贝被递增的值，返回它，再前递增这个 “工作值”。对于基本类型，这样做没什么大不了的。但是对于迭代器，这样做是一个很大的问题（举个例子，一些迭代器中包含 stack 和 set 对象，拷贝这些迭代器同样可能会调用这些对象的拷贝构造函数）。通常而言，养成总是使用前置自增的习惯，就不会有问题。

#### <span id="namespace_indent">namespace 缩进</span>

In general, we strive to reduce indentation wherever possible. This is useful because we want code to fit into 80 columns without excessive wrapping, but also because it makes it easier to understand the code. To facilitate this and avoid some insanely deep nesting on occasion, don’t indent namespaces. If it helps readability, feel free to add a comment indicating what namespace is being closed by a }. For example:

通常，无论再哪我们都努力的尽可能减少缩进。这样是很有用的，因为我们希望代码容纳在80列的范围内，但同样是因为这样做让代码变得更容易理解。为了促进这样做并且避免某种场合下疯狂的深层次嵌套，不要缩进命名空间。如果这样对可读性有利，那么顺手添加一行注释，来指示`}`关闭了哪个命名空间。

```cpp
namespace llvm {
namespace knowledge {

/// This class represents things that Smith can have an intimate
/// understanding of and contains the data associated with it.
class Grokable {
...
public:
  explicit Grokable() { ... }
  virtual ~Grokable() = 0;

  ...

};

} // end namespace knowledge
} // end namespace llvm
```

Feel free to skip the closing comment when the namespace being closed is obvious for any reason. For example, the outer-most namespace in a header file is rarely a source of confusion. But namespaces both anonymous and named in source files that are being closed half way through the file probably could use clarification.

当由于任何原因而关闭的命名空间很明显时，请随时跳过结束注释。例如，头文件中最外面的名称空间很少引起混乱。但是匿名命名空间和源文件中半途被关闭的命名空间可能需要说明。

#### <span id="anonymous_namespace">匿名 namespace</span>

After talking about namespaces in general, you may be wondering about anonymous namespaces in particular. Anonymous namespaces are a great language feature that tells the C++ compiler that the contents of the namespace are only visible within the current translation unit, allowing more aggressive optimization and eliminating the possibility of symbol name collisions. Anonymous namespaces are to C++ as “static” is to C functions and global variables. While “static” is available in C++, anonymous namespaces are more general: they can make entire classes private to a file.

The problem with anonymous namespaces is that they naturally want to encourage indentation of their body, and they reduce locality of reference: if you see a random function definition in a C++ file, it is easy to see if it is marked static, but seeing if it is in an anonymous namespace requires scanning a big chunk of the file.

Because of this, we have a simple guideline: make anonymous namespaces as small as possible, and only use them for class declarations. For example:

在讨论完一般的命名空间后，你可能特别想了解匿名命名空间。匿名命名空间是个很好的语言特性，告诉C++编译器该命名空间的内容仅在当前编译单元内可见，允许更多积极的优化和消除命名符号的冲突。对C++而言，匿名命名空间等同 `static`对于C函数和全局变量，命名空间更加的通用：它可以使真个类私用于文件。

匿名命名空间的问题在于它们自然希望鼓励缩进其主体，并且降低了引用的位置：如果你在C++文件中随机看函数定义，则很容易看到它是否标记为静态，但它是否位于匿名命名空间中，需要扫描文件的很大一部分。

由此，我们有一个简单的规则：让匿名命名空间尽可能的小，并且仅在类声明时使用，举个栗子：

```cpp
namespace {
class StringSort {
...
public:
  StringSort(...)
  bool operator<(const char *RHS) const;
};
} // end anonymous namespace

static void runHelper() {
  ...
}

bool StringSort::operator<(const char *RHS) const {
  ...
}
```

Avoid putting declarations other than classes into anonymous namespaces:

避免将类之外的声明放入命名空间：

```cpp
namespace {

// ... many declarations ...

void runHelper() {
  ...
}

// ... many declarations ...

} // end anonymous namespace
```

When you are looking at “runHelper” in the middle of a large C++ file, you have no immediate way to tell if this function is local to the file. In contrast, when the function is marked static, you don’t need to cross-reference faraway places in the file to tell that the function is local.

当你看到大规模的C++文件中部看到 "runHelper" 时，你无法立即判断这是否是文件局部函数。相反，如果这个文件被标记为静态，则你不需要在这个文件中交叉查看更远的地方就能说出这个文件是局部的。

## <span id="see_also">其它参考</span>

A lot of these comments and recommendations have been culled from other sources. Two particularly important books for our work are:

1. Effective C++ by Scott Meyers. Also interesting and useful are “More Effective C++” and “Effective STL” by the same author.
2. Large-Scale C++ Software Design by John Lakos

If you get some free time, and you haven’t read them: do so, you might learn something.

大量注释和推荐是从其它来源中挑选出来的，对于我们工作特别重要的两本书：
1. Effective C++, Scott Meyers 著。同样有趣和有用的是“More Effective C++” 和 “Effective STL”，它们是来自同一个作者.
2. Large-Scale C++ Software Design, John Lakos著。

If you get some free time, and you haven’t read them: do so, you might learn something.

如果你有一些空间时间的化，并且还没有阅读过他们：那么，你可能会学到一些东西。