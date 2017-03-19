---
layout: default
lead: WebAssembly 或者 <i>wasm</i> 是一个体积小、加载快并且兼容 Web 的全新格式
---
<div class="flash flash-warn">
  WebAssembly 是由主流浏览器厂商组成的 <a href="https://www.w3.org/community/webassembly/">W3C 社区团体</a> 制定的一个新的规范
</div>
<div class="row">
  <div class="bubble col-xs-12 col-md-6">
    <h3>高效</h3>
    <p>WebAssembly 有一套完整的<a href="/docs/semantics/">语义</a>，实际上 wasm 是体积小且加载快的<a href="/docs/binary-encoding/">二进制格式</a>， 其目标就是充分发挥<a href="/docs/portability/#assumptions-for-efficient-execution">硬件能力</a>以达到原生执行效率</p>
  </div>
  <div class="bubble col-xs-12 col-md-6">
    <h3>安全</h3>
    <p>在现有的 JavaScript 虚拟机里， WebAssembly 程序运行在一个沙箱化的<a href="/docs/semantics/#linear-memory">执行环境</a>里，<a href="/docs/web/">嵌入网站</a>运行时会强制启用同源策略以及浏览器安全策略</p>
  </div>
</div>
<div class="row">
  <div class="bubble col-xs-12 col-md-6">
    <h3>开放</h3>
    <p>WebAssembly 虽然是二进制格式，但是也可以转为友好的<a href="/docs/text-format/">文本格式</a>进行调试、测试、试验、优化、学习，甚至直接手写，具体可以通过 wasm 模块的<a href="/docs/faq/#will-webassembly-support-view-source-on-the-web">查看源码</a>进行查看</p>
  </div>
  <div class="bubble col-xs-12 col-md-6">
    <h3>标准</h3>
    <p>WebAssembly 一开始就被设计为作为 <a href="/docs/web/">Web 的一部分</a>， 因此可以直接调用 Web APIs，当然，在<a href="/docs/non-web/">非 Web</a> 环境里也可以使用</p>
  </div>
</div>