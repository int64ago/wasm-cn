---
layout: getting-started
---

# 高级工具

有很多工具可以帮助 WebAssembly 构建和处理原文件或二进制文件，如果你是一位编译器开发者或者对 WebAssembly 底层格式有兴趣，这些工具就再合适不过了

这里有两种不同类型的工具，主要针对编译器开发者或者尝试分析经过 [Emscripten](http://kripken.github.io/emscripten-site/) 转换的 WebAssembly 的二进制内容的开发者：

*   [WABT](https://github.com/WebAssembly/wabt) - WebAssembly 二进制工具
*   [Binaryen](https://github.com/WebAssembly/binaryen) - 交叉编译工具

## WABT: WebAssembly 二进制工具

这个工具用于 WebAssembly 二进制格式和可读的文本格式之间转换。文本格式一种 [S 表达式](https://en.wikipedia.org/wiki/S-expression) ，主要为了让编译器开发人员更好的去分析和调试等

注意，S 表达式可以被 WABT 支持并不是 WebAssembly 自身定义的，它只是众多可以表示 WebAssembly 内容的文本格式的一种，仅仅选了一种方便 WABT 转换的格式而已。开发者可以很方便地定义其它的文本类型，只要能涵盖 WebAssembly 机器语义即可

### wasm2wast

这个工具用于把 WebAssembly 二进制转为 S 表达式，输入是二进制文件，输出是可读的文本格式

为了优化算法、跟踪或者插入调试钩子等目的，开发者可以编辑产生的文本格式，然后再转回二进制格式

### wast2wasm

这个工具恰好跟 **wasm2wast** 执行流相反，把 S 表达式的文件转为二进制文件

把 **wasm2wast** 和 **wast2wasm** 结合起来用可以做到无损地用外部工具操纵 WebAssembly 的内容

#### wasm-interp

这是一个命令行解释器，可以让开发者独立地运行 WebAssembly 二进制程序，它基于栈机器实现了直接解释 WebAssembly 二进制，这点不同于浏览器（加载的时候先将 WebAssembly 二进制转为原生代码）

用这个可以在脱离浏览器的环境下进行单元测试或验证二进制文件等

## Binaryen

[Binaryen](https://github.com/WebAssembly/binaryen) 可以理解为是一个编辑器后端，把 WebAssembly 作为最终输出文件。它包含了 C API 以及实现了自己的内部中间表达式（[IR](https://en.wikipedia.org/wiki/Intermediate_representation)），同时也针对中间表达式做了很多优化，支持并行编译

比如，binaryen 已经被用作 **[asm2wasm](https://github.com/WebAssembly/binaryen/blob/master/src/asm2wasm.h)** 的一部分，这个是用于把 asm.js 转为 WebAssembly 文件。而且它结合 [LLVM](http://llvm.org/) 还可以做到把 [Rust](https://www.rust-lang.org/en-US/) 编译到 WebAssembly

编译器开发、性能优化等开发者应该充分利用 binaryen 以及它包含的工具集，可以方便地编译 WebAssembly、汇编、反汇编、把 asm.js 或 LLVM .s 文件转为 WebAssembly 等

工具开发者一直积极地跟进 binaryen 功能完善
