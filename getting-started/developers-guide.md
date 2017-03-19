---
layout: getting-started
---

# 开发者指南

在这里，手把手教你编译一个 WebAssembly 程序

### 依赖
编译 WebAssembly 之前，需要先从源码编译 LLVM ，以下是一些依赖：

- Git，Linux 和 OSX 这个应该是内置的，Windows 的话可以直接下载 [Git for Windows](https://git-scm.com/) 安装
- CMake，如果是 Linux 或 OSX，可以通过包管理器如 `apt-get` 或 `brew` 等安装，Windows 需要下载 [CMake installer](https://cmake.org/download/) 并安装
- 编译器，Linux 安装 [GCC](http://askubuntu.com/questions/154402/install-gcc-on-ubuntu-12-04-lts)，OSX 安装 [Xcode](https://itunes.apple.com/us/app/xcode/id497799835) ，Windows 安装 [VS2015+](https://www.visualstudio.com/downloads/)
- Python 2.7.x，Linux 和 OSX 这个应该是内置的，其它具体见 [这里](https://wiki.python.org/moin/BeginnersGuide/Download)

安装完后，请确保 `git` 、 `cmake` 和 `python` 环境变量配置正确

### 从源码构建 Emscripten
从源码构建 Emscripten 是通过 Emscripten SDK 自动进行的，步骤如下：

    $ git clone https://github.com/juj/emsdk.git
    $ cd emsdk
    $ ./emsdk install sdk-incoming-64bit binaryen-master-64bit
    $ ./emsdk activate sdk-incoming-64bit binaryen-master-64bit

以上步骤结束后，设置下 Emscripten 编译器环境变量

    $ source ./emsdk_env.sh

这个命令把相关的环境变量加入到 PATH 下，不过只限当前终端有效

如果是 Windows，仅仅把上述的 `./emsdk` 替换为 `emsdk` ，把 `source ./emsdk_env.sh` 替换为 `emsdk_env` 即可

### 编译并运行一个简单程序
现在我们构建 WebAssembly 可谓万事俱备只欠东风了，但还有些注意事项：

- `emcc` 需要传参数 `-s WASM=1` （默认编译出 asm.js）
- 如果想让 Emscripten 直接编译出可直接使用的 HTML 而不是 wasm 二进制，你需要指定输出后缀为 `.html`
- 最后，不可以直接打开 HTML 去看运行的结果，因为有跨域限制，我们需要去启动一个简单的 HTTP 服务器

以下命令会创建一个简单的 "hello world" 程序，编译部分已经加粗

<pre>
$ mkdir hello
$ cd hello
$ echo '#include &lt;stdio.h&gt;' &gt; hello.c
$ echo 'int main(int argc, char ** argv) {' &gt;&gt; hello.c
$ echo 'printf("Hello, world!\n");' &gt;&gt; hello.c
$ echo '}' &gt;&gt; hello.c
$ <b>emcc hello.c -s WASM=1 -o hello.html</b>
</pre>

我们可以直接使用 Emscripten SDK 提供的 `emrun` 来启一个 Web 服务器

    $ emrun --no_browser --port 8080 .

服务启动后可以<a href="http://localhost:8080/hello.html" target="_blank">浏览器访问这个</a>看效果，如果你看到控制台输出了 "Hello, world!"， 恭喜你，成功完成了第一个 WebAssembly 程序！