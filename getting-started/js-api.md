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

[Linear memory](/docs/semantics/#linear-memory) is another important WebAssembly building block that is typically used to represent the entire heap of a compiled C/C++ application.  From a JavaScript perspective, linear memory (henceforth, just “memory”) can be thought of as a resizable `ArrayBuffer` that is carefully optimized for low-overhead sandboxing of loads and stores.


Memories can be created from JavaScript by supplying their initial size and, optionally, their maximum size:


```js
var memory = new WebAssembly.Memory({initial:10, maximum:100});
```


The first important thing to notice is that the unit of `initial` and `maximum` is *WebAssembly pages* which are fixed to be 64KiB.  Thus, `memory` above has an initial size of 10 pages, or 640KiB and a maximum size of 6.4MiB.


Since most byte-range operations in JavaScript already operate on `ArrayBuffer` and typed arrays, rather than defining a whole new set of incompatible operations, `WebAssembly.Memory` exposes its bytes by simply providing a `buffer` getter that returns an `ArrayBuffer`.  For example, to write `42` directly into the first word of linear memory:


```js
new Uint32Array(memory.buffer)[0] = 42;
```


Once created, a memory can be grown by calls to `Memory.prototype.grow`, where again the argument is specified in units of WebAssembly pages:


```js
memory.grow(1);
```


If a `maximum` is supplied upon creation, attempts to grow past this `maximum` will throw a `RangeError` exception.  The engines takes advantage of this supplied upper-bounds to reserve memory ahead of time which can make resizing more efficient.


Since an `ArrayBuffer`’s `byteLength` is immutable, after a successful `Memory.grow` operation, the`buffer` getter will return a *new* `ArrayBuffer` object (with the new `byteLength`) and any previous `ArrayBuffer` objects become “detached” (zero length, many operations throw).


Just like functions, linear memories can be defined inside a module or imported.  Similarly, a module may also optionally export its memory.  This means that JavaScript can get access to the memory of a WebAssembly instance either by creating a `new WebAssembly.Memory` and passing it in as an import *or* by receiving a `Memory` export.


For example, let’s take a WebAssembly module that sums an array of integers (replacing the body of the function with “...”):


```lisp
(module
  (memory (export "mem") 1)
  (func (export "accumulate") (param $ptr i32) (param $length i32) …))
```


Since this module *exports* its memory, given an `Instance` of this module called `instance`, we can use its exports' `mem` getter to create and populate an input array directly in the instance’s linear memory, as follows:


```js
var i32 = new Uint32Array(instance.exports.mem);
for (var i = 0; i < 10; i++)
  i32[i] = i;
var sum = instance.exports.accumulate(0, 10);
```


Memory *imports* work just like function imports, only `Memory` objects are passed as values instead of JS functions.  Memory imports are useful for two reasons:

- They allow JavaScript to fetch and create the initial contents of memory before or concurrent with module compilation.
- They allow a single `Memory` object to be imported by multiple instances, which is a critical building block for implementing [dynamic linking](/docs/dynamic-linking) in WebAssembly.
