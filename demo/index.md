---
layout: default
markdown: kramdown
permalink: demo/
---
# Tanks!

[![](screenshot.jpg)](Tanks/)
<div id="play-wasm" class="btn-block">
  <a class="btn btn-primary" href="Tanks/" role="button">试试吧！</a>
</div>
<div id="play-asm" class="btn-block hide-btn-block">
  <a class="btn hide-asm-support" href="Tanks/" role="button">回退到 asm.js</a>
  <span class="btn-comment btn-comment-error hide-asm-support">你的浏览器还不支持 WebAssembly ，<a href="/roadmap/">了解更多</a></span>
</div>

游戏 [Tanks!](https://unity3d.com/learn/tutorials/projects/tanks-tutorial) 是一个 Unity 的教学实例，已经转为 WebAssembly 格式了。沙地里开动坦克并且射击敌方坦克，这是一个多人游戏，蓝色坦克通过 W/A/S/D 按键移动，按空格键发射子弹；红色坦克通过方向键移动，通过回车键发射子弹

<script type="text/javascript" >
(function() {
  // detect WebAssembly support
  var support = (typeof WebAssembly === 'object');
  
  // toggle button wasm/asm.js button visibility
  if (!support) {
    var wasmButton = document.getElementById('play-wasm');
    wasmButton.className += ' hide-btn-block';
    var asmButton = document.getElementById('play-asm');
    asmButton.className = asmButton.className.replace(/(?:^|\s)hide-btn-block(?!\S)/, '');
  }
})();
</script>
