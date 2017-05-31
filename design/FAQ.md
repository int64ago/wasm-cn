# 问答

## 既然有了 asm.js， 为什么还造一个轮子？

... 其实还有 pthreads ([Mozilla pthreads][], [Chromium pthreads][]) 和
SIMD ([simd.js][], [Chromium SIMD][], [simd.js in asm.js][])

  [Mozilla pthreads]: https://blog.mozilla.org/javascript/2015/02/26/the-path-to-parallel-javascript/
  [Chromium pthreads]: https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/d-0ibJwCS24
  [simd.js]: https://hacks.mozilla.org/2014/10/introducing-simd-js/
  [Chromium SIMD]: https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/2PIOEJG_aYY
  [simd.js in asm.js]: http://discourse.specifiction.org/t/request-for-comments-simd-js-in-asm-js/676

WebAssembly 主要有两个好处：

1. 采用的原生编码后的二进制相对于 JavaScript 解析速度快了不少（[实验][experiments] 表明加速超过 20 倍），在移动端，大的源文件可能需要 20-40 秒仅仅为了*解析*，因此原生编码（特别是结合其它[工具流][streaming]达到更好的压缩效果）对于提升代码加载速度很重要

2. 为了避免重复经历 asm.js 在 [AOT][]-[compilability][] 上的限制，同时在没有 [针对 asm.js 的优化措施][specific asm.js optimizations]的前提下能够得到好的性能，提出一个新的东西可以*更容易*去添加一些[特性 :unicorn:][future features]来达到原生性能的目的

  [experiments]: BinaryEncoding.md#why-a-binary-encoding-instead-of-a-text-only-representation
  [streaming]: https://www.w3.org/TR/streams-api/
  [AOT]: http://asmjs.org/spec/latest/#ahead-of-time-compilation
  [compilability]: https://blog.mozilla.org/luke/2014/01/14/asm-js-aot-compilation-and-startup-performance/
  [specific asm.js optimizations]: https://blog.mozilla.org/luke/2015/02/18/microsoft-announces-asm-js-optimizations/#asmjs-opts

当然，每次新标准出现都会带来新的问题（维护、反对声、体积），不过瑕不掩瑜。WebAssembly 允许浏览器直接在现有的 JavaScript 引擎里引入（复用现有的编译器后端和 ESM 前端）。总体而言，WebAssembly 可以认为是 JavaScript 一个大的特性，而不是浏览器的一个扩展

综上，即使引擎针对 asm.js 进行了优化，WebAssembly 好处还是很明显的

## WebAssembly 使用场景是什么？

WebAssembly 设计是针对[大量使用场景](UseCases.md)的


## WebAssembly 可以通过垫片实现吗？

我们认为是可以的，早期已经有很多[原型](https://github.com/WebAssembly/polyfill-prototype-1)例子[[1](https://lukewagner.github.io/AngryBotsPacked), [2](https://lukewagner.github.io/PlatformerGamePacked)]展示了从类似 WebAssembly 的二进制格式解码为 asm.js 是高效的

并且随着 WebAssembly 设计的改变，已经有[更多](https://github.com/WebAssembly/polyfill-prototype-2)采用垫片的[实验](https://github.com/WebAssembly/binaryen/blob/master/src/wasm2asm.h)

总的说来，浏览器接受 WebAssembly 的速度被乐观估计了，这是好事，但是在垫片上推进工程是很容易受阻的

当前，用垫片把 WebAssembly 模拟为 asm.js 不是很紧迫，已经有反向的垫片可以[把 asm.js 模拟为 WebAssembly](https://github.com/WebAssembly/binaryen/blob/master/src/asm2wasm.h)，最终分发为 asm.js 还是 WebAssembly 都是可以的。而且还可以通过调整 emscripten 的[参数](https://github.com/kripken/emscripten/wiki/WebAssembly)构建出 asm.js 和 WebAssembly 同时可以运行的代码，节省了在客户端加载垫片的时间。对于性能要求不高的代码，可以选择一个已经编译好的 WebAssembly 解释器 [binaryen.js](https://github.com/WebAssembly/binaryen/blob/master/test/binaryen.js/test.js).

最后，用垫片实现 WebAssembly 确实是个有意思的想法，而且原则上是可能的


## Is WebAssembly only for C/C++ programmers?

As explained in the [high-level goals](HighLevelGoals.md), to achieve a Minimum
Viable Product, the initial focus is on [C/C++](CAndC++.md).

However, by [integrating with JavaScript at the ES6 Module interface](Modules.md#integration-with-es6-modules),
web developers don't need to write C++ to take advantage of libraries that others have written; 
reusing a modular C++ library can be as simple as [using a module from JavaScript](http://jsmodules.io).

Beyond the MVP, another [high-level goal](HighLevelGoals.md)
is to improve support for languages other than C/C++.  This includes [allowing WebAssembly code to
allocate and access garbage-collected (JavaScript, DOM, Web API) objects
:unicorn:][future dom].
Even before GC support is added to WebAssembly, it is possible to compile a language's VM 
to WebAssembly (assuming it's written in portable C/C++) and this has already been demonstrated 
([1](http://ruby.dj), [2](https://kripken.github.io/lua.vm.js/lua.vm.js.html),
[3](https://syntensity.blogspot.com/2010/12/python-demo.html)).  However, "compile the VM" strategies 
increase the size of distributed code, lose browser devtools integration, can have cross-language
cycle-collection problems and miss optimizations that require integration with the browser.


## Which compilers can I use to build WebAssembly programs?

WebAssembly initially focuses on [C/C++](CAndC++.md), and a new, clean
WebAssembly backend is being developed in upstream clang/LLVM, which can then be
used by LLVM-based projects like [Emscripten][] and [PNaCl][].

As WebAssembly evolves it will support more languages than C/C++, and we hope
that other compilers will support it as well, even for the C/C++ language, for
example [GCC][]. The WebAssembly working group found it easier to start with
LLVM support because they had more experience with that toolchain from their
[Emscripten][] and [PNaCl][] work.

  [Emscripten]: http://emscripten.org
  [PNaCl]: http://gonacl.com
  [GCC]: https://gcc.gnu.org

We hope that proprietary compilers also gain WebAssembly support, but we'll let
vendors speak about their own platforms.

The [WebAssembly Community Group][] would be delighted to collaborate with more
compiler vendors, take their input into consideration in WebAssembly itself, and
work with them on ABI matters.

  [WebAssembly Community Group]: https://www.w3.org/community/webassembly/


## Will WebAssembly support View Source on the Web?

Yes! WebAssembly defines a [text format](TextFormat.md) to be rendered when
developers view the source of a WebAssembly module in any developer tool. Also,
a specific goal of the text format is to allow developers to write WebAssembly
modules by hand for testing, experimenting, optimizing, learning and teaching
purposes. In fact, by dropping all the
[coercions required by asm.js validation](http://asmjs.org/spec/latest/#introduction),
the WebAssembly text format should be much more natural to read and write than
asm.js. Outside the browser, command-line and online tools that convert between
text and binary will also be made readily available.  Lastly, a scalable form of
source maps is also being considered as part of the WebAssembly
[tooling story](Tooling.md).


## What's the story for Emscripten users?

Existing Emscripten users will get the option to build their projects to
WebAssembly, by flipping a flag. Initially, Emscripten's asm.js output would be
converted to WebAssembly, but eventually Emscripten would use WebAssembly
throughout the pipeline. This painless transition is enabled by the
[high-level goal](HighLevelGoals.md) that WebAssembly integrate well with the
Web platform (including allowing synchronous calls into and out of JavaScript)
which makes WebAssembly compatible with Emscripten's current asm.js compilation
model.


## Is WebAssembly trying to replace JavaScript?

No! WebAssembly is designed to be a complement to, not replacement of,
JavaScript. While WebAssembly will, over time, allow many languages to be
compiled to the Web, JavaScript has an incredible amount of momentum and will
remain the single, privileged (as described
[above](FAQ.md#is-webassembly-only-for-cc-programmers)) dynamic language of the
Web. Furthermore, it is expected that JavaScript and WebAssembly will be used
together in a number of configurations:

* Whole, compiled C++ apps that leverage JavaScript to glue things together.
* HTML/CSS/JavaScript UI around a main WebAssembly-controlled center canvas,
  allowing developers to leverage the power of web frameworks to build
  accessible, web-native-feeling experiences.
* Mostly HTML/CSS/JavaScript app with a few high-performance WebAssembly modules
  (e.g., graphing, simulation, image/sound/video processing, visualization,
  animation, compression, etc., examples which we can already see in asm.js
  today) allowing developers to reuse popular WebAssembly libraries just like
  JavaScript libraries today.
* When WebAssembly
  [gains the ability to access garbage-collected objects :unicorn:][future dom],
  those objects will be shared with JavaScript, and not live in a walled-off
  world of their own.


## Why not just use LLVM bitcode as a binary format?

The [LLVM](http://llvm.org/) compiler infrastructure has a lot to recommend it:
it has an existing intermediate representation (LLVM IR) and binary encoding
format (bitcode). It has code generation backends targeting many architectures
is actively developed and maintained by a large community. In fact
[PNaCl](http://gonacl.com) already uses LLVM as a basis for its binary
format. However the goals and requirements that LLVM was designed to meet are
subtly mismatched with those of WebAssembly.

WebAssembly has several requirements and goals for its Instruction Set
Architecture (ISA) and binary encoding:

* Portability: The ISA must be the same for every machine architecture.
* Stability: The ISA and binary encoding must not change over time (or change
  only in ways that can be kept backward-compatible).
* Small encoding: The representation of a program should be as small as possible
  for transmission over the Internet.
* Fast decoding: The binary format should be fast to decompress and decode for
  fast startup of programs.
* Fast compiling: The ISA should be fast to compile (and suitable for either
  AOT- or JIT-compilation) for fast startup of programs.
* Minimal [nondeterminism](Nondeterminism.md): The behavior of programs should
  be as predictable and deterministic as possible (and should be the same on
  every architecture, a stronger form of the portability requirement stated
  above).

LLVM IR is meant to make compiler optimizations easy to implement, and to
represent the constructs and semantics required by C, C++, and other languages
on a large variety of operating systems and architectures. This means that by
default the IR is not portable (the same program has different representations
for different architectures) or stable (it changes over time as optimization and
language requirements change). It has representations for a huge variety of
information that is useful for implementing mid-level compiler optimizations but
is not useful for code generation (but which represents a large surface area for
codegen implementers to deal with).  It also has undefined behavior (largely
similar to that of C and C++) which makes some classes of optimization feasible
or more powerful, but which can lead to unpredictable behavior at runtime.
LLVM's binary format (bitcode) was designed for temporary on-disk serialization
of the IR for link-time optimization, and not for stability or compressibility
(although it does have some features for both of those).

None of these problems are insurmountable. For example PNaCl defines a small
portable
[subset](https://developer.chrome.com/native-client/reference/pnacl-bitcode-abi)
of the IR with reduced undefined behavior, and a stable version of the bitcode
encoding. It also employs several techniques to improve startup
performance. However, each customization, workaround, and special solution means
less benefit from the common infrastructure. We believe that by taking our
experience with LLVM and designing an IR and binary encoding for our goals and
requirements, we can do much better than adapting a system designed for other
purposes.

Note that this discussion applies to use of LLVM IR as a standardized
format. LLVM's clang frontend and midlevel optimizers can still be used to
generate WebAssembly code from C and C++, and will use LLVM IR in their
implementation similarly to how PNaCl and Emscripten do today.


## Why is there no fast-math mode with relaxed floating point semantics?

Optimizing compilers commonly have fast-math flags which permit the compiler to
relax the rules around floating point in order to optimize more
aggressively. This can include assuming that NaNs or infinities don't occur,
ignoring the difference between negative zero and positive zero, making
algebraic manipulations which change how rounding is performed or when overflow
might occur, or replacing operators with approximations that are cheaper to
compute.

These optimizations effectively introduce nondeterminism; it isn't possible to
determine how the code will behave without knowing the specific choices made by
the optimizer. This often isn't a serious problem in native code scenarios,
because all the nondeterminism is resolved by the time native code is
produced. Since most hardware doesn't have floating point nondeterminism,
developers have an opportunity to test the generated code, and then count on it
behaving consistently for all users thereafter.

WebAssembly implementations run on the user side, so there is no opportunity for
developers to test the final behavior of the code. Nondeterminism at this level
could cause distributed WebAssembly programs to behave differently in different
implementations, or change over time. WebAssembly does have
[some nondeterminism](Nondeterminism.md) in cases where the tradeoffs warrant
it, but fast-math flags are not believed to be important enough:

 * Many of the important fast-math optimizations happen in the mid-level
   optimizer of a compiler, before WebAssembly code is emitted. For example,
   loop vectorization that depends on floating point reassociation can still be
   done at this level if the user applies the appropriate fast-math flags, so
   WebAssembly programs can still enjoy these benefits. As another example,
   compilers can replace floating point division with floating point
   multiplication by a reciprocal in WebAssembly programs just as they do for
   other platforms.
 * Mid-level compiler optimizations may also be augmented by implementing them
   in a [JIT library](JITLibrary.md) in WebAssembly. This would allow them to
   perform optimizations that benefit from having
   [information about the target](FeatureTest.md) and information about the
   source program semantics such as fast-math flags at the same time. For
   example, if SIMD types wider than 128-bit are added, it's expected that there
   would be feature tests allowing WebAssembly code to determine which SIMD
   types to use on a given platform.
 * When WebAssembly
   [adds an FMA operator :unicorn:][future floating point],
   folding multiply and add sequences into FMA operators will be possible.
 * WebAssembly doesn't include its own math functions like `sin`, `cos`, `exp`,
   `pow`, and so on. WebAssembly's strategy for such functions is to allow them
   to be implemented as library routines in WebAssembly itself (note that x86's
   `sin` and `cos` instructions are slow and imprecise and are generally avoided
   these days anyway). Users wishing to use faster and less precise math
   functions on WebAssembly can simply select a math library implementation
   which does so.
 * Most of the individual floating point operators that WebAssembly does have
   already map to individual fast instructions in hardware. Telling `add`,
   `sub`, or `mul` they don't have to worry about NaN for example doesn't make
   them any faster, because NaN is handled quickly and transparently in hardware
   on all modern platforms.
 * WebAssembly has no floating point traps, status register, dynamic rounding
   modes, or signalling NaNs, so optimizations that depend on the absence of
   these features are all safe.


## What about `mmap`?

The [`mmap`](http://pubs.opengroup.org/onlinepubs/009695399/functions/mmap.html)
syscall has many useful features. While these are all packed into one overloaded
syscall in POSIX, WebAssembly unpacks this functionality into multiple
operators:

* the MVP starts with the ability to grow linear memory via a
  [`grow_memory`](Semantics.md#resizing) operator;
* proposed
  [future features :unicorn:][future memory control] would
  allow the application to change the protection and mappings for pages in the
  contiguous range `0` to `memory_size`.

A significant feature of `mmap` that is missing from the above list is the
ability to allocate disjoint virtual address ranges. The reasoning for this
omission is:

* The above functionality is sufficient to allow a user-level libc to implement
  full, compatible `mmap` with what appears to be noncontiguous memory
  allocation (but, under the hood is just coordinated use of `memory_resize` and
  `mprotect`/`map_file`/`map_shmem`/`madvise`).
* The benefit of allowing noncontiguous virtual address allocation would be if
  it allowed the engine to interleave a WebAssembly module's linear memory with
  other memory allocations in the same process (in order to mitigate virtual
  address space fragmentation). There are two problems with this:

  - This interleaving with unrelated allocations does not currently admit
    efficient security checks to prevent one module from corrupting data outside
    its heap (see discussion in
    [#285](https://github.com/WebAssembly/design/pull/285)).
  - This interleaving would require making allocation nondeterministic and
    nondeterminism is something that WebAssembly generally
    [tries to avoid](Nondeterminism.md).


## Why have wasm32 and wasm64, instead of just an abstract `size_t`?

The amount of linear memory needed to hold an abstract `size_t` would then also
need to be determined by an abstraction, and then partitioning the linear memory
address space into segments for different purposes would be more complex. The
size of each segment would depend on how many `size_t`-sized objects are stored
in it. This is theoretically doable, but it would add complexity and there would
be more work to do at application startup time.

Also, allowing applications to statically know the pointer size can allow them
to be optimized more aggressively. Optimizers can better fold and simplify
integer expressions when they have full knowledge of the bitwidth. And, knowing
memory sizes and layouts for various types allows one to know how many trailing
zeros there are in various pointer types.

Also, C and C++ deeply conflict with the concept of an abstract `size_t`.
Constructs like `sizeof` are required to be fully evaluated in the front-end
of the compiler because they can participate in type checking. And even before
that, it's common to have predefined macros which indicate pointer sizes,
allowing code to be specialized for pointer sizes at the very earliest stages of
compilation. Once specializations are made, information is lost, scuttling
attempts to introduce abstractions.

And finally, it's still possible to add an abstract `size_t` in the future if
the need arises and practicalities permit it.


## Why have wasm32 and wasm64, instead of just using 8 bytes for storing pointers?

A great number of applications don't ever need as much as 4 GiB of memory.
Forcing all these applications to use 8 bytes for every pointer they store would
significantly increase the amount of memory they require, and decrease their
effective utilization of important hardware resources such as cache and memory
bandwidth.

The motivations and performance effects here should be essentially the same as
those that motivated the development of the
[x32 ABI](https://en.wikipedia.org/wiki/X32_ABI) for Linux.

Even Knuth found it worthwhile to give us his opinion on this issue at point,
[a flame about 64-bit pointers](http://www-cs-faculty.stanford.edu/~uno/news08.html).

## Will I be able to access proprietary platform APIs (e.g. Android / iOS)?

Yes but it will depend on the _WebAssembly embedder_. Inside a browser you'll 
get access to the same HTML5 and other browser-specific APIs which are also 
accessible through regular JavaScript. However, if a wasm VM is provided as an 
[“app execution platform”](NonWeb.md) by a specific vendor, it might provide 
access to [proprietary platform-specific APIs](Portability.md#api) of e.g. 
Android / iOS. 

[future features]: FutureFeatures.md
[future dom]: FutureFeatures.md#gcdom-integration
[future floating point]: FutureFeatures.md#additional-floating-point-operators
[future memory control]: FutureFeatures.md#finer-grained-control-over-memory
