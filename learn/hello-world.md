---
outline: deep
---

# Hello World

我相信你一定是从 `Hello, World!` 开始学习其他语言的，在这里我们也不例外，我们来通过一个简单的程序，来向 zig 的世界打一声招呼！

先使用 `zig init-exe` 命令初始化一个项目，再将以下内容覆写到 `src/main.zig` 中。

```zig
const std = @import("std");

pub fn main() void {
    std.debug.print("Hello, World!\n", .{});
}
```

_很简单，不是吗？_

## 简单说明

以上程序中，我们先通过 `@import` 这个内置函数（在zig中有很多的内置函数，它们都以`@`开头，并且遵循[小驼峰命名法](#)）引入了 zig 的标准库。

通过在 `main` 函数（程序默认是从此处开始执行，这是规定）中使用在标准库 **debug** 包中定义的 `print` 函数来输出语句 _Hello, World!_

`print` 函数接受两个参数，类似于 C 的 `printf` 函数，第一个参数是要格式化的字符串，第二个是参量表，这里我们需要说的是，格式化字符串使用的是`{}`，
zig 会自动为我们根据后面的参量表推导出对应的类型，当 zig 无法推导时，我们需要显示声明要格式化的参量类型，例如字符串就是 `{s}`，整数就是 `{d}`，
更多的类型我们在后面会详细说明。我们传入的第二个参数是一个元组（**tuple**），它是一个元组（你可以把它看作是一个匿名结构体，这里你只需要知道一下就行）。

::: info 🅿️ 提示

好了，上面的内容你应该已经看完了，现在我要告诉你，正常使用 zig 打印字符串是不能这样子做的！

你是不是觉得自己被耍了？别担心，上面仅仅只是一个示例而已，来为你演示一下 zig 的使用！

:::

下面的内容可能有点难，你可以*暂时跳过这里*，后面再来学习！

## 换一种方式？

接下来，让我们换一种方式来讲述如何“正确”地使用 zig 打印出“Hello, World!”，不要认为这是一个简单的问题，这涉及到计算机相当底层的涉及哲学。

首先，我要告诉你，zig 并没有一个内置的打印功能，包含“输出”功能的包只有 `log` 包和 `debug` 包，zig 并没有内置类似与 `@print()` 这种函数。再来一个简单的例子告诉你，如何打印东西（**_但是请记住，以下示例代码不应用于生产环境中_**）。

```zig
const std = @import("std");

pub fn main() !void {
    var out = std.io.getStdOut().writer();
    var err = std.io.getStdErr().writer();

    try out.print("Hello {s}!\n", .{"out"});
    try err.print("Hello {s}!\n", .{"err"});
}
```

:::info 🅿️ 提示

`main` 函数的返回值是`!void`，这是联合错误类型，该语法将会告诉 zig 编译器会返回错误或者值，此处的意思是如果有返回值，一定是一个错误。

:::

这段代码将会分别输出 `Hello out!` 和 `Hello err!`，这里我需要向你讲述一下 `stdout` 和 `stderr` ，它们均是抽象的io（input and output）流句柄（关于流这个概念可能不好解释，你暂时就当作向水流一样的数据的就行）。`stdout` 用于正常的输出，它可能会出现错误导致写入失败。`stderr` 用于错误输出，我们假定 `stderr` 一定不会失败（这个是操作系统负责保证的），这就是它们的区别。

通过 `io` 命名空间（或者称之为包）获取到了标准输出和错误输出的 `writer` 句柄，这个句柄实现流`print`函数，我们只需要正常打印即可！

接下来加深一点难度，你有没有想过，这些`print`函数是如何实现的？

它们都是依靠系统调用来实现输出效果，但是这就面临着性能问题，我们知道系统调用会造成内核上下文切换的开销（系统调用的流程：执行系统调用，此时控制权会切换回内核，由内核执行完成进程需要的系统调用函数后再将控制权返回给进程），所以我们如何解决这个问题呢？可以增加一个缓冲区，等到要打印的内容都到一定程度后再一次性全部 `print`，那么此时的解决方式就如下：

```zig
const std = @import("std");

pub fn main() !void {
    var out = std.io.getStdOut().writer();
    var err = std.io.getStdErr().writer();

    // 获取buffer
    var out_buffer = std.io.bufferedWriter(out);
    var err_buffer = std.io.bufferedWriter(err);

    // 获取writer句柄
    var out_writer = out_buffer.writer();
    var err_writer = err_buffer.writer();

    // 通过句柄写入buffer
    try out_writer.print("Hello {s}!\n", .{"out"});
    try err_writer.print("Hello {s}!\n", .{"err"});

    // 尝试刷新buffer
    try out_buffer.flush();
    try err_buffer.flush();
}
```

此时我们就分别得到了使用缓冲区的 `stdout` 和 `stderr`， 性能更高了！

## 更进一步？

上面我们已经完成了带有缓冲区的“打印”，这很棒！

但是，它还没有多线程支持，所以我们可能需要添加一个**锁**来保证打印函数的先后执行顺序，你可以使用 `std.Thread.Mutex`，它的文档在[_这里_](https://ziglang.org/documentation/master/std/#A;std:Thread.Mutex)，但我更推荐你结合标准库的源码来了解它。

## 了解更多？

如果你想了解更多内容，可以看一看这个视频 [Advanced Hello World in Zig - Loris Cro](https://youtu.be/iZFXAN8kpPo?si=WNpp3t42LPp1TkFI)