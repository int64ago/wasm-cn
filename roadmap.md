---
layout: default
---
# 规划

## WebAssembly 共识

WebAssembly 社区成员由四个浏览器厂商组成：Chrome、Edge、Firefox 和 WebKit ，关于初版（[MVP](/docs/mvp/)）WebAssembly 的 API 和二进制格式已经达成共识，之后除非有已经充分试验的或意义重大的功能，规范不会做大的更改。这就意味着已经结束了浏览器预览阶段，浏览器发布的时候可以默认开启 WebAssembly ，从现在开始，所有新增特性都会保证向后兼容性。

共识包括 [JavaScript API](/docs/js/) 和 [二进制格式](/docs/binary-encoding/)（包括[解释器](https://github.com/WebAssembly/spec/tree/master/interpreter)）。现在就可以参考[开发者指南](/getting-started/developers-guide/)和 [MDN 资源](https://developer.mozilla.org/en-US/docs/WebAssembly)，结合 Emscripten 进行尝试， 你也可以参考下[其它工具](/getting-started/advanced-tools/)

可以通过[快速开始](/getting-started/developers-guide/)进行尝试，也可以直接给我们[反馈](/community/feedback/)

> 关于共识，通过[邮件列表](https://lists.w3.org/Archives/Public/public-webassembly/2017Feb/0002.html)了解更多

## 接下来的计划

WebAssembly 社区和贡献者接下来要完成以下事宜：

* 把 [design](https://github.com/webassembly/design)
  和 [spec interpreter](https://github.com/WebAssembly/spec/tree/master/interpreter) 两个仓库提取并合并到一个统一的仓库 [spec](https://github.com/WebAssembly/spec)
* 为 W3C WebAssembly 工作组制定一个新的规章制度
* 结束 [WebAssembly LLVM backend](https://github.com/llvm-mirror/llvm/tree/master/test/CodeGen/WebAssembly) 实验版本，切为稳定版（更新 Emscripten）
* 把 WebAssembly 集成到浏览器的开发者工具里
* 针对 [post-MVP features](/docs/future-features/) 开展工作

## 直到浏览器预览阶段已完成

- WebAssembly 的[二进制版本](/docs/binary-encoding/#high-level-structure)
已经冻结并标记为 `0x1`（之后所有新增的特性都会保证向下兼容性），可以通过升级 Emscripten 并且重新编译来更新 WebAssembly 模块

## 已经完成的里程碑

- 2015 年 4 月 - [WebAssembly 社区](https://www.w3.org/community/webassembly) 成立
- 2015 年 6 月 - 第一份[公告](https://github.com/WebAssembly/design/issues/150) [[1](https://blogs.msdn.microsoft.com/mikeholman/2015/06/17/working-on-the-future-of-compile-to-web-applications/)][[2](https://blog.mozilla.org/luke/2015/06/17/webassembly/)]发表
- 2016 年 3 月 - 核心的可行性特性确定 [[1](https://blogs.windows.com/msedgedev/2016/03/15/previewing-webassembly-experiments)] [[2](https://v8project.blogspot.com/2016/03/experimental-support-for-webassembly.html)] [[3](https://hacks.mozilla.org/2016/03/a-webassembly-milestone/)]
- 2016 年 10 月 - 多个可行性特性实现并发布浏览器预览 [[1](https://blogs.windows.com/msedgedev/2016/10/31/webassembly-browser-preview/)] [[2](http://v8project.blogspot.com/2016/10/webassembly-browser-preview.html)] [[3](https://hacks.mozilla.org/2016/10/webassembly-browser-preview)]
- 2017 年 2 月 - 官方 [LOGO](https://github.com/WebAssembly/design/issues/980)  敲定
- 2017 年 3 月 - [多个浏览器达成共识](https://lists.w3.org/Archives/Public/public-webassembly/2017Feb/0002.html)，同时结束了浏览器预览阶段
