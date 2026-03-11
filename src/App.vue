<template>
  <div class="calculator">
    <header class="calc-header">
      <div class="calc-title">计算器</div>
      <div class="calc-subtitle">标准</div>
    </header>
    <section class="calc-display">
      <div class="calc-history">{{ historyValue }}</div>
      <div class="calc-output">{{ displayValue }}</div>
    </section>
    <section class="calc-pad">
      <button class="key key-memory" @click="memoryClear">MC</button>
      <button class="key key-memory" @click="memoryRecall">MR</button>
      <button class="key key-memory" @click="memoryAdd">M+</button>
      <button class="key key-memory" @click="memorySubtract">M-</button>

      <button class="key key-fn" @click="applyUnary('sin')">sin</button>
      <button class="key key-fn" @click="applyUnary('cos')">cos</button>
      <button class="key key-fn" @click="applyUnary('tan')">tan</button>
      <button class="key key-fn" @click="applyUnary('ln')">ln</button>

      <button class="key key-fn" @click="applyUnary('log')">log</button>
      <button class="key key-fn" @click="applyUnary('abs')">|x|</button>
      <button class="key key-fn" @click="applyUnary('exp')">eˣ</button>
      <button class="key key-fn" @click="insertConstant('pi')">π</button>

      <button class="key key-muted" @click="handlePercent">%</button>
      <button class="key key-muted" @click="clearEntry">CE</button>
      <button class="key key-muted" @click="clearAll">C</button>
      <button class="key key-muted" @click="backspace">⌫</button>

      <button class="key key-muted" @click="applyUnary('reciprocal')">1/x</button>
      <button class="key key-muted" @click="applyUnary('square')">x²</button>
      <button class="key key-muted" @click="applyUnary('sqrt')">√x</button>
      <button class="key key-operator" @click="setOperator('÷')">÷</button>

      <button class="key" @click="inputDigit('7')">7</button>
      <button class="key" @click="inputDigit('8')">8</button>
      <button class="key" @click="inputDigit('9')">9</button>
      <button class="key key-operator" @click="setOperator('×')">×</button>

      <button class="key" @click="inputDigit('4')">4</button>
      <button class="key" @click="inputDigit('5')">5</button>
      <button class="key" @click="inputDigit('6')">6</button>
      <button class="key key-operator" @click="setOperator('−')">−</button>

      <button class="key" @click="inputDigit('1')">1</button>
      <button class="key" @click="inputDigit('2')">2</button>
      <button class="key" @click="inputDigit('3')">3</button>
      <button class="key key-operator" @click="setOperator('+')">+</button>

      <button class="key" @click="toggleSign">±</button>
      <button class="key" @click="inputDigit('0')">0</button>
      <button class="key" @click="inputDecimal">.</button>
      <button class="key key-equals" @click="handleEquals">=</button>
    </section>
  </div>
</template>

<script setup>
import { ref } from "vue";

const displayValue = ref("0");
const historyValue = ref("");
const previousValue = ref(null);
const currentOperator = ref(null);
const waitingForNewValue = ref(false);
const lastOperator = ref(null);
const lastOperand = ref(null);
const memoryValue = ref(0);
const memoryActive = ref(false);

const operatorLabels = {
  "+": "+",
  "−": "−",
  "×": "×",
  "÷": "÷",
};

const formatNumber = (value) => {
  if (!Number.isFinite(value)) {
    return "错误";
  }

  const normalized = value.toString();
  const cleaned = normalized.replace("e", "E");

  if (cleaned.length <= 14) {
    return cleaned;
  }

  const exponential = value.toExponential(8).replace("e", "E");
  return exponential;
};

const updateHistory = () => {
  if (previousValue.value === null || currentOperator.value === null) {
    historyValue.value = "";
    return;
  }

  historyValue.value = `${formatNumber(previousValue.value)} ${operatorLabels[currentOperator.value]}`;
};

const setHistoryExpression = (text) => {
  historyValue.value = text;
};

const toRadians = (value) => (value * Math.PI) / 180;

const inputDigit = (digit) => {
  if (displayValue.value === "错误") {
    displayValue.value = "0";
  }

  if (waitingForNewValue.value) {
    displayValue.value = digit;
    waitingForNewValue.value = false;
    return;
  }

  displayValue.value = displayValue.value === "0" ? digit : `${displayValue.value}${digit}`;
};

const inputDecimal = () => {
  if (displayValue.value === "错误") {
    displayValue.value = "0";
  }

  if (waitingForNewValue.value) {
    displayValue.value = "0.";
    waitingForNewValue.value = false;
    return;
  }

  if (!displayValue.value.includes(".")) {
    displayValue.value = `${displayValue.value}.`;
  }
};

const clearAll = () => {
  displayValue.value = "0";
  historyValue.value = "";
  previousValue.value = null;
  currentOperator.value = null;
  waitingForNewValue.value = false;
  lastOperator.value = null;
  lastOperand.value = null;
};

const clearEntry = () => {
  displayValue.value = "0";
  waitingForNewValue.value = false;
};

const backspace = () => {
  if (waitingForNewValue.value || displayValue.value === "错误") {
    displayValue.value = "0";
    waitingForNewValue.value = false;
    return;
  }

  if (displayValue.value.length > 1) {
    displayValue.value = displayValue.value.slice(0, -1);
  } else {
    displayValue.value = "0";
  }
};

const toggleSign = () => {
  if (displayValue.value === "错误") {
    displayValue.value = "0";
    return;
  }

  if (displayValue.value === "0") {
    return;
  }

  displayValue.value = displayValue.value.startsWith("-")
    ? displayValue.value.slice(1)
    : `-${displayValue.value}`;
};

const calculate = (firstValue, secondValue, operator) => {
  switch (operator) {
    case "+":
      return firstValue + secondValue;
    case "−":
      return firstValue - secondValue;
    case "×":
      return firstValue * secondValue;
    case "÷":
      return secondValue === 0 ? NaN : firstValue / secondValue;
    default:
      return secondValue;
  }
};

const setOperator = (operator) => {
  if (displayValue.value === "错误") {
    return;
  }

  if (waitingForNewValue.value && previousValue.value !== null) {
    currentOperator.value = operator;
    updateHistory();
    return;
  }

  const inputValue = Number(displayValue.value);

  if (previousValue.value === null) {
    previousValue.value = inputValue;
  } else if (!waitingForNewValue.value && currentOperator.value) {
    const result = calculate(previousValue.value, inputValue, currentOperator.value);
    displayValue.value = formatNumber(result);
    previousValue.value = result;
  }

  currentOperator.value = operator;
  waitingForNewValue.value = true;
  updateHistory();
};

const handleEquals = () => {
  if (displayValue.value === "错误") {
    return;
  }

  const inputValue = Number(displayValue.value);

  if (currentOperator.value) {
    const leftValue = previousValue.value;
    const operand = waitingForNewValue.value ? previousValue.value : inputValue;
    const result = calculate(previousValue.value, operand, currentOperator.value);

    lastOperator.value = currentOperator.value;
    lastOperand.value = operand;
    displayValue.value = formatNumber(result);
    previousValue.value = result;
    setHistoryExpression(
      `${formatNumber(leftValue)} ${operatorLabels[currentOperator.value]} ${formatNumber(operand)} =`
    );
    currentOperator.value = null;
    waitingForNewValue.value = true;
    return;
  }

  if (lastOperator.value && lastOperand.value !== null) {
    const leftValue = Number(displayValue.value);
    const result = calculate(leftValue, lastOperand.value, lastOperator.value);
    displayValue.value = formatNumber(result);
    setHistoryExpression(
      `${formatNumber(leftValue)} ${operatorLabels[lastOperator.value]} ${formatNumber(lastOperand.value)} =`
    );
    waitingForNewValue.value = true;
  }
};

const handlePercent = () => {
  if (displayValue.value === "错误") {
    return;
  }

  const inputValue = Number(displayValue.value);
  if (previousValue.value !== null && currentOperator.value) {
    const percentValue = (previousValue.value * inputValue) / 100;
    displayValue.value = formatNumber(percentValue);
  } else {
    displayValue.value = formatNumber(inputValue / 100);
  }
};

const applyUnary = (type) => {
  if (displayValue.value === "错误") {
    return;
  }

  const inputValue = Number(displayValue.value);
  let result = inputValue;

  if (type === "reciprocal") {
    result = inputValue === 0 ? NaN : 1 / inputValue;
  } else if (type === "square") {
    result = inputValue * inputValue;
  } else if (type === "sqrt") {
    result = inputValue < 0 ? NaN : Math.sqrt(inputValue);
  } else if (type === "sin") {
    result = Math.sin(toRadians(inputValue));
  } else if (type === "cos") {
    result = Math.cos(toRadians(inputValue));
  } else if (type === "tan") {
    result = Math.tan(toRadians(inputValue));
  } else if (type === "ln") {
    result = inputValue <= 0 ? NaN : Math.log(inputValue);
  } else if (type === "log") {
    result = inputValue <= 0 ? NaN : Math.log10(inputValue);
  } else if (type === "abs") {
    result = Math.abs(inputValue);
  } else if (type === "exp") {
    result = Math.exp(inputValue);
  }

  displayValue.value = formatNumber(result);
  waitingForNewValue.value = true;
};

const insertConstant = (constant) => {
  let value = 0;
  if (constant === "pi") {
    value = Math.PI;
  }

  displayValue.value = formatNumber(value);
  waitingForNewValue.value = false;
};

const memoryClear = () => {
  memoryValue.value = 0;
  memoryActive.value = false;
};

const memoryRecall = () => {
  if (!memoryActive.value) {
    return;
  }

  displayValue.value = formatNumber(memoryValue.value);
  waitingForNewValue.value = false;
};

const memoryAdd = () => {
  const inputValue = Number(displayValue.value);
  if (!Number.isFinite(inputValue)) {
    return;
  }

  memoryValue.value += inputValue;
  memoryActive.value = true;
};

const memorySubtract = () => {
  const inputValue = Number(displayValue.value);
  if (!Number.isFinite(inputValue)) {
    return;
  }

  memoryValue.value -= inputValue;
  memoryActive.value = true;
};
</script>
