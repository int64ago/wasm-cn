---
layout: getting-started
---

# 理解 JS API

_我们假设你已经有了一个 .wasm 模块了，可以是 [直接从 C/C++ 编译得到](/getting-started/developers-guide/) 或者 [从 S 表达式汇编得到](/getting-started/advanced-tools/#wabt-the-webassembly-binary-toolkit)_

## 加载并且运行

因为[未来支持特性](/docs/future-features/)允许 WebAssembly 模块可以像 ES6 的模块一样进行加载（如： `<script type='module'>`），因此 WebAssembly 现在必须通过 JavaScript 加载，基本加载分为以下三个过程：

- 把 `.wasm` 里的字节放到一个指定类型的数组或者 `ArrayBuffer` 里
- 把字节编译输出到 `WebAssembly.Module`
- 通过输入方法实例化 `WebAssembly.Module`，最终得到可以调用的输出


下面详细解释下：

第一步是有很多方法得到类型化数组或者 `ArrayBuffer` ：网络（XHR 或 fetch）、从 IndexedDB 读取的 `File`、甚至直接通过 JavaScript 写方法得到

接下来编译字节使用的是异步方法 `WebAssembly.compile` ，返回一个 Promise 并且 resolves 传的是 `WebAssembly.Module` 。一个 `Module` 对象是无状态的并且支持[结构克隆](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)，这意味着编译好的代码可以直接存储在 IndexedDB 或者通过 `postMessage` 传输

最后一步的*实例化* `Module` 是通过把 `Module` 和其依赖的 `imports` 作为参数 new 一个 `WebAssembly.Instance`，`实例化`对象类似[函数闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming))，依赖上下文环境并且不可结构克隆

我们可以合并最后两步为一个`实例化`操作，输入为 `bytes` 和 `imports`，异步输出 `Instance`：

```js
function instantiate(bytes, imports) {
  return WebAssembly.compile(bytes).then(m => new WebAssembly.Instance(m, imports));
}
```

为了实战一把，我们先要介绍另一个 JS API：

## 函数的导入和导出

类似 ES6 的模块，WebAssembly 也可以导出导出函数（后面还会介绍对象），先来看个简单例子，里面包含从 `imports` 里导入函数 `i` 并且最终导出为模块 `e`：

```lisp
;; simple.wasm
(module
  (func $i (import "imports" "i") (param i32))
  (func (export "e")
    i32.const 42
    call $i))
```

（这里，我们没有通过写 C/C++ 代码然后编译为 WebAssembly，而是直接写[文本格式](/docs/text-format/)代码然后[汇编](/getting-started/advanced-tools/#wabt-the-webassembly-binary-toolkit)输出为二进制文件 `simple.wasm`）

从这个模块中我们可以得到一些信息：首先，WebAssembly 的输入有二级命名空间；这个例子里的内部名 `$i` 是从 `imports.i` 拿到的，简单说，我们必须从实例化对象参数里的 `imports` 对象里映射这个二级命名空间

```js
var importObject = { imports: { i: arg => console.log(arg) } };
```



把所有步骤放一起，我们可以通过一个简单的 Promise 链实例化我们的模块：

```js
fetch('simple.wasm').then(response => response.arrayBuffer())
.then(bytes => instantiate(bytes, importObject))
.then(instance => instance.exports.e());
```

最后一行是执行我们导出的 WebAssembly 函数，最终实际上是调用导入的 JS 函数 `console.log(42)`

## 内存

[线性内存](/docs/semantics/#linear-memory) 是 WebAssembly 另一个很重要的概念，用于描述整个编译后的 C/C++ 程序的堆栈。从一个 JavaScript 程序员角度看的话，线性内存（或者简称为”内存“，下同）可以认为是一个可扩展的 `ArrayBuffer`，这个是被高度优化的，使其可以用尽可能少的资源维持一个沙箱环境

内存可以从 JavaScript 创建，可以制定初始值以及最大值：

```js
var memory = new WebAssembly.Memory({initial:10, maximum:100});
```

需要注意的是，`initial` 和 `maximum` 的单位是 *WebAssembly 页*，目前是固定的 64KB ，因此上面分配的内存实际是十页，或者说是 640KB，最大值是 6.4MB

因为大部分 JavaScript 的字节操作都可以直接作用于 `ArrayBuffer` 上，而不是额外定义一组不兼容的操作，`WebAssembly.Memory` 通过一个 返回值是 `ArrayBuffer` 的 `buffer` 指针来操纵它的字节。比如，需要在线性内存的第一个字节处写入 `42`：

```js
new Uint32Array(memory.buffer)[0] = 42;
```

一旦内存区域创建了，内存的扩充可以通过调用 `Memory.prototype.grow` 实现：

```js
memory.grow(1);
```

如果达到了 `maximum`，继续尝试扩充会得到 `RangeError` 异常，引擎通过这种限制内存上限来保证更高效合理的使用内存

因为 `ArrayBuffer` 的 `byteLength` 是不能改变的，因此每次成功执行 `Memory.grow` 操作后，`buffer` 都会返回一个新的 `ArrayBuffer`（有新的 `byteLength`），之前的 `ArrayBuffer` 会变成一个游离的对象

跟函数一样，线性内存可以在模块内部定义也可以传进去。类似的，一个模块也可以导出其内存，这意味着 JavaScript 可以通过创建一个 `new WebAssembly.Memory` 并传入模块*或者*接收一个 `Memory` 的导出来访问 WebAssembly 的内存

例如，有个计算数组和的 WebAssembly 的模块（函数体被换成了 `...`）：

```lisp
(module
  (memory (export "mem") 1)
  (func (export "accumulate") (param $ptr i32) (param $length i32) …))
```

这个模块导出了它的内存，假设模块叫 `instance`，我们可以使用导出的 `mem` 指针去操纵内部内存，如下：

```js
var i32 = new Uint32Array(instance.exports.mem);
for (var i = 0; i < 10; i++)
  i32[i] = i;
var sum = instance.exports.accumulate(0, 10);
```

内存*传参*原理跟函数传参类似，仅仅是 `Memory` 对象作为值传递，而不是 JS 函数。内存传参非常有用，主要由于以下两个原因：

- 允许 JavaScript 可以在模块编译前或者编译中获取或者设置内存的初始值
- 允许一个 `Memory` 对象被多个实例传入，这对于实现 WebAssembly 的[动态链接](/docs/dynamic-linking)至关重要
