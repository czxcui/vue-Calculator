<template>
  <div class="calculator">
    <header class="calc-header">
      <div class="calc-title">计算器</div>
      <div class="calc-subtitle">标准</div>
    </header>
    <section class="calc-display">
      <div class="calc-history">0</div>
      <div class="calc-output">0</div>
    </section>
    <section class="calc-pad">
      <button class="key key-muted">%</button>
      <button class="key key-muted">CE</button>
      <button class="key key-muted">C</button>
      <button class="key key-muted">⌫</button>

      <button class="key key-muted">1/x</button>
      <button class="key key-muted">x²</button>
      <button class="key key-muted">√x</button>
      <button class="key key-operator">÷</button>

      <button class="key">7</button>
      <button class="key">8</button>
      <button class="key">9</button>
      <button class="key key-operator">×</button>

      <button class="key">4</button>
      <button class="key">5</button>
      <button class="key">6</button>
      <button class="key key-operator">−</button>

      <button class="key">1</button>
      <button class="key">2</button>
      <button class="key">3</button>
      <button class="key key-operator">+</button>

      <button class="key">±</button>
      <button class="key">0</button>
      <button class="key">.</button>
      <button class="key key-equals">=</button>
    </section>
  </div>
</template>

<script setup></script>
