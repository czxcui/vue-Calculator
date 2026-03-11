<template>
  <div class="app-shell" :class="[`mode-${activeMode}`]">
    <aside class="calc-sidebar" :class="{ open: sidebarOpen }">
      <div class="sidebar-header">计算器</div>
      <nav class="sidebar-menu">
        <button
          v-for="mode in modes"
          :key="mode.id"
          class="sidebar-item"
          :class="{ active: activeMode === mode.id }"
          @click="selectMode(mode.id)"
        >
          <span class="sidebar-icon">{{ mode.icon }}</span>
          <span class="sidebar-label">{{ mode.label }}</span>
        </button>
      </nav>
      <div class="sidebar-footer">
        <button class="sidebar-item" @click="selectMode('settings')">
          <span class="sidebar-icon">⚙</span>
          <span class="sidebar-label">设置</span>
        </button>
      </div>
    </aside>
    <div v-if="sidebarOpen" class="overlay" @click="toggleSidebar"></div>
    <div v-if="historyOpen" class="history-overlay" @click="closeHistory"></div>

    <main class="calc-main">
      <header class="calc-topbar">
        <button class="icon-button" @click="toggleSidebar">☰</button>
        <div class="mode-title">
          <div class="mode-label">计算器</div>
          <div class="mode-sub">{{ activeModeLabel }}</div>
        </div>
        <div class="topbar-actions">
          <span v-if="memoryActive" class="memory-flag">M</span>
          <button class="icon-button" @click="toggleHistory">🕘</button>
        </div>
      </header>

      <section v-if="showDisplay" class="calc-display">
        <div class="calc-history">{{ historyValue }}</div>
        <div class="calc-output">{{ displayValue }}</div>
      </section>

      <section v-if="activeMode === 'standard'" class="calc-panel">
        <div class="memory-row">
          <button class="memory-key" :disabled="!memoryActive" @click="memoryClear">MC</button>
          <button class="memory-key" :disabled="!memoryActive" @click="memoryRecall">MR</button>
          <button class="memory-key" @click="memoryAdd">M+</button>
          <button class="memory-key" @click="memorySubtract">M-</button>
          <button class="memory-key" @click="memoryStore">MS</button>
        </div>
        <section class="calc-pad">
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
      </section>

      <section v-if="activeMode === 'scientific'" class="calc-panel">
        <div class="scientific-meta">
          <button class="meta-key" @click="toggleAngle">DEG</button>
          <button class="meta-key" @click="toggleFeMode">F-E</button>
        </div>
        <div class="memory-row">
          <button class="memory-key" :disabled="!memoryActive" @click="memoryClear">MC</button>
          <button class="memory-key" :disabled="!memoryActive" @click="memoryRecall">MR</button>
          <button class="memory-key" @click="memoryAdd">M+</button>
          <button class="memory-key" @click="memorySubtract">M-</button>
          <button class="memory-key" @click="memoryStore">MS</button>
          <button class="memory-key" @click="noop">Mv</button>
        </div>
        <div class="scientific-toolbar">
          <div class="dropdown" :class="{ open: trigOpen }">
            <button class="dropdown-toggle" @click="toggleTrig">三角学 ▾</button>
            <div class="dropdown-menu">
              <button class="dropdown-item" @click="applyUnary('sin')">sin</button>
              <button class="dropdown-item" @click="applyUnary('cos')">cos</button>
              <button class="dropdown-item" @click="applyUnary('tan')">tan</button>
              <button class="dropdown-item" @click="applyUnary('hyp')">hyp</button>
              <button class="dropdown-item" @click="applyUnary('sec')">sec</button>
              <button class="dropdown-item" @click="applyUnary('csc')">csc</button>
              <button class="dropdown-item" @click="applyUnary('cot')">cot</button>
            </div>
          </div>
          <div class="dropdown" :class="{ open: funcOpen }">
            <button class="dropdown-toggle" @click="toggleFunc">函数 ▾</button>
            <div class="dropdown-menu">
              <button class="dropdown-item" @click="applyUnary('abs')">|x|</button>
              <button class="dropdown-item" @click="applyUnary('floor')">⌊x⌋</button>
              <button class="dropdown-item" @click="applyUnary('ceil')">⌈x⌉</button>
              <button class="dropdown-item" @click="applyUnary('rand')">rand</button>
              <button class="dropdown-item" @click="toggleAngle">{{ angleUnitLabel }}</button>
              <button class="dropdown-item" @click="applyUnary('exp')">exp</button>
            </div>
          </div>
        </div>

        <section class="calc-pad calc-pad-scientific">
          <button class="key key-fn" @click="noop">2nd</button>
          <button class="key key-fn" @click="insertConstant('pi')">π</button>
          <button class="key key-fn" @click="insertConstant('e')">e</button>
          <button class="key key-muted" @click="clearAll">C</button>
          <button class="key key-muted" @click="backspace">⌫</button>

          <button class="key key-fn" @click="applyUnary('square')">x²</button>
          <button class="key key-fn" @click="applyUnary('reciprocal')">1/x</button>
          <button class="key key-fn" @click="applyUnary('abs')">|x|</button>
          <button class="key key-fn" @click="applyUnary('exp')">exp</button>
          <button class="key key-fn" @click="setOperator('mod')">mod</button>

          <button class="key key-fn" @click="applyUnary('sqrt')">√x</button>
          <button class="key key-fn" @click="noop">(</button>
          <button class="key key-fn" @click="noop">)</button>
          <button class="key key-fn" @click="applyUnary('fact')">n!</button>
          <button class="key key-operator" @click="setOperator('÷')">÷</button>

          <button class="key key-fn" @click="setOperator('^')">xʸ</button>
          <button class="key" @click="inputDigit('7')">7</button>
          <button class="key" @click="inputDigit('8')">8</button>
          <button class="key" @click="inputDigit('9')">9</button>
          <button class="key key-operator" @click="setOperator('×')">×</button>

          <button class="key key-fn" @click="applyUnary('pow10')">10ˣ</button>
          <button class="key" @click="inputDigit('4')">4</button>
          <button class="key" @click="inputDigit('5')">5</button>
          <button class="key" @click="inputDigit('6')">6</button>
          <button class="key key-operator" @click="setOperator('−')">−</button>

          <button class="key key-fn" @click="applyUnary('log')">log</button>
          <button class="key" @click="inputDigit('1')">1</button>
          <button class="key" @click="inputDigit('2')">2</button>
          <button class="key" @click="inputDigit('3')">3</button>
          <button class="key key-operator" @click="setOperator('+')">+</button>

          <button class="key key-fn" @click="applyUnary('ln')">ln</button>
          <button class="key" @click="toggleSign">±</button>
          <button class="key" @click="inputDigit('0')">0</button>
          <button class="key" @click="inputDecimal">.</button>
          <button class="key key-equals" @click="handleEquals">=</button>
        </section>
      </section>

      <section v-if="activeMode === 'programmer'" class="calc-panel">
        <div class="programmer-bases">
          <button
            v-for="base in programmerBases"
            :key="base"
            class="base-key"
            :class="{ active: programmerBase === base }"
            @click="setProgrammerBase(base)"
          >
            {{ base }}
          </button>
        </div>
        <div class="programmer-values">
          <div class="programmer-row">
            <span class="programmer-label">HEX</span>
            <span class="programmer-value">{{ programmerHex }}</span>
          </div>
          <div class="programmer-row">
            <span class="programmer-label">DEC</span>
            <span class="programmer-value">{{ programmerDec }}</span>
          </div>
          <div class="programmer-row">
            <span class="programmer-label">OCT</span>
            <span class="programmer-value">{{ programmerOct }}</span>
          </div>
          <div class="programmer-row">
            <span class="programmer-label">BIN</span>
            <span class="programmer-value">{{ programmerBin }}</span>
          </div>
        </div>
        <section class="calc-pad">
          <button class="key key-muted" @click="clearEntry">CE</button>
          <button class="key key-muted" @click="clearAll">C</button>
          <button class="key key-muted" @click="backspace">⌫</button>
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
      </section>

      <section v-if="activeMode === 'date'" class="calc-panel">
        <div class="date-panel">
          <div class="date-row">
            <label class="date-label">开始日期</label>
            <input v-model="dateStart" type="date" class="date-input" />
          </div>
          <div class="date-row">
            <label class="date-label">结束日期</label>
            <input v-model="dateEnd" type="date" class="date-input" />
          </div>
          <div class="date-result">{{ dateDiffText }}</div>
        </div>
      </section>

      <section v-if="activeMode === 'converter'" class="calc-panel">
        <div class="converter-panel">
          <div class="converter-row">
            <label class="date-label">类别</label>
            <select v-model="converterCategory" class="date-input">
              <option value="length">长度</option>
              <option value="mass">质量</option>
              <option value="temperature">温度</option>
            </select>
          </div>
          <div class="converter-grid">
            <div class="converter-card">
              <input v-model.number="converterFromValue" type="number" class="date-input" />
              <select v-model="converterFromUnit" class="date-input">
                <option v-for="unit in converterUnits" :key="unit" :value="unit">{{ unit }}</option>
              </select>
            </div>
            <div class="converter-card">
              <input :value="converterToValue" type="text" class="date-input" disabled />
              <select v-model="converterToUnit" class="date-input">
                <option v-for="unit in converterUnits" :key="unit" :value="unit">{{ unit }}</option>
              </select>
            </div>
          </div>
        </div>
      </section>
      <aside class="history-drawer" :class="{ open: historyOpen }" @click.stop>
        <div class="history-header">历史记录</div>
        <div class="history-list">
          <div v-if="historyEntries.length === 0" class="history-empty">暂无记录</div>
          <div v-for="entry in historyEntries" :key="entry.id" class="history-item">
            <div class="history-expression">{{ entry.expression }}</div>
            <div class="history-result">{{ entry.result }}</div>
          </div>
        </div>
        <button class="history-clear" @click="clearHistory">🗑</button>
      </aside>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

const modes = [
  { id: "standard", label: "标准", icon: "▦" },
  { id: "scientific", label: "科学", icon: "∑" },
  { id: "programmer", label: "程序员", icon: "</>" },
  { id: "date", label: "日期计算", icon: "📅" },
  { id: "converter", label: "转换器", icon: "⇄" },
];

const activeMode = ref("standard");
const sidebarOpen = ref(false);
const historyOpen = ref(false);
const historyEntries = ref([]);
const trigOpen = ref(false);
const funcOpen = ref(false);
const feMode = ref(false);

const displayValue = ref("0");
const historyValue = ref("");
const previousValue = ref(null);
const currentOperator = ref(null);
const waitingForNewValue = ref(false);
const lastOperator = ref(null);
const lastOperand = ref(null);
const memoryValue = ref(0);
const memoryActive = ref(false);
const angleUnit = ref("deg");

const dateStart = ref("");
const dateEnd = ref("");

const converterCategory = ref("length");
const converterFromValue = ref(0);
const converterFromUnit = ref("m");
const converterToUnit = ref("km");

const programmerBase = ref(10);
const programmerBases = [16, 10, 8, 2];

const showDisplay = computed(() =>
  ["standard", "scientific", "programmer"].includes(activeMode.value)
);

const activeModeLabel = computed(() => {
  const match = modes.find((mode) => mode.id === activeMode.value);
  return match ? match.label : "标准";
});

const angleUnitLabel = computed(() => (angleUnit.value === "deg" ? "DEG" : "RAD"));

const operatorLabels = {
  "+": "+",
  "−": "−",
  "×": "×",
  "÷": "÷",
  "^": "^",
  mod: "mod",
};

const selectMode = (modeId) => {
  if (modeId === "settings") {
    return;
  }

  activeMode.value = modeId;
  sidebarOpen.value = false;
};

const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value;
};

const toggleHistory = () => {
  historyOpen.value = !historyOpen.value;
};

const closeHistory = () => {
  historyOpen.value = false;
};

const clearHistory = () => {
  historyEntries.value = [];
};

const toggleTrig = () => {
  trigOpen.value = !trigOpen.value;
  if (trigOpen.value) {
    funcOpen.value = false;
  }
};

const toggleFunc = () => {
  funcOpen.value = !funcOpen.value;
  if (funcOpen.value) {
    trigOpen.value = false;
  }
};

const toggleFeMode = () => {
  feMode.value = !feMode.value;
};

const noop = () => {};

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

const dateDiffText = computed(() => {
  if (!dateStart.value || !dateEnd.value) {
    return "选择开始和结束日期";
  }

  const start = new Date(dateStart.value);
  const end = new Date(dateEnd.value);
  if (Number.isNaN(start.getTime()) || Number.isNaN(end.getTime())) {
    return "日期无效";
  }

  const diffMs = Math.abs(end.getTime() - start.getTime());
  const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
  return `相差 ${diffDays} 天`;
});

const converterUnitOptions = {
  length: ["m", "km", "cm", "mm"],
  mass: ["kg", "g", "lb"],
  temperature: ["C", "F", "K"],
};

const converterUnits = computed(() => converterUnitOptions[converterCategory.value] || []);

const convertValue = (value, fromUnit, toUnit, category) => {
  if (category === "temperature") {
    if (fromUnit === toUnit) return value;
    let celsius = value;
    if (fromUnit === "F") celsius = (value - 32) / 1.8;
    if (fromUnit === "K") celsius = value - 273.15;
    if (toUnit === "C") return celsius;
    if (toUnit === "F") return celsius * 1.8 + 32;
    if (toUnit === "K") return celsius + 273.15;
  }

  const factors = {
    length: {
      m: 1,
      km: 1000,
      cm: 0.01,
      mm: 0.001,
    },
    mass: {
      kg: 1,
      g: 0.001,
      lb: 0.45359237,
    },
  };

  const base = factors[category] || {};
  if (!base[fromUnit] || !base[toUnit]) {
    return value;
  }

  const valueInBase = value * base[fromUnit];
  return valueInBase / base[toUnit];
};

const converterToValue = computed(() => {
  const value = Number(converterFromValue.value);
  if (!Number.isFinite(value)) {
    return "";
  }

  const converted = convertValue(
    value,
    converterFromUnit.value,
    converterToUnit.value,
    converterCategory.value
  );
  return formatNumber(converted);
});

watch(converterCategory, (category) => {
  const defaults = converterUnitOptions[category] || [];
  converterFromUnit.value = defaults[0] || "";
  converterToUnit.value = defaults[1] || defaults[0] || "";
  converterFromValue.value = 0;
});

const programmerNumber = computed(() => {
  const value = Number(displayValue.value);
  if (!Number.isFinite(value)) {
    return 0;
  }
  return Math.trunc(value);
});

const formatProgrammerValue = (value, base) => value.toString(base).toUpperCase();

const programmerHex = computed(() => formatProgrammerValue(programmerNumber.value, 16));
const programmerDec = computed(() => formatProgrammerValue(programmerNumber.value, 10));
const programmerOct = computed(() => formatProgrammerValue(programmerNumber.value, 8));
const programmerBin = computed(() => formatProgrammerValue(programmerNumber.value, 2));

const setProgrammerBase = (base) => {
  programmerBase.value = base;
};

const toRadians = (value) => (angleUnit.value === "deg" ? (value * Math.PI) / 180 : value);

const factorial = (value) => {
  if (!Number.isInteger(value) || value < 0) {
    return NaN;
  }

  if (value > 170) {
    return Infinity;
  }

  let result = 1;
  for (let i = 2; i <= value; i += 1) {
    result *= i;
  }
  return result;
};

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
    case "^":
      return Math.pow(firstValue, secondValue);
    case "mod":
      return secondValue === 0 ? NaN : firstValue % secondValue;
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
    historyEntries.value = [
      {
        id: Date.now(),
        expression: `${formatNumber(leftValue)} ${operatorLabels[currentOperator.value]} ${formatNumber(operand)} =`,
        result: formatNumber(result),
      },
      ...historyEntries.value,
    ];
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
    historyEntries.value = [
      {
        id: Date.now(),
        expression: `${formatNumber(leftValue)} ${operatorLabels[lastOperator.value]} ${formatNumber(lastOperand.value)} =`,
        result: formatNumber(result),
      },
      ...historyEntries.value,
    ];
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
  } else if (type === "cube") {
    result = inputValue * inputValue * inputValue;
  } else if (type === "sqrt") {
    result = inputValue < 0 ? NaN : Math.sqrt(inputValue);
  } else if (type === "sin") {
    result = Math.sin(toRadians(inputValue));
  } else if (type === "cos") {
    result = Math.cos(toRadians(inputValue));
  } else if (type === "tan") {
    result = Math.tan(toRadians(inputValue));
  } else if (type === "hyp") {
    result = Math.sinh(toRadians(inputValue));
  } else if (type === "sec") {
    const cosValue = Math.cos(toRadians(inputValue));
    result = cosValue === 0 ? NaN : 1 / cosValue;
  } else if (type === "csc") {
    const sinValue = Math.sin(toRadians(inputValue));
    result = sinValue === 0 ? NaN : 1 / sinValue;
  } else if (type === "cot") {
    const tanValue = Math.tan(toRadians(inputValue));
    result = tanValue === 0 ? NaN : 1 / tanValue;
  } else if (type === "ln") {
    result = inputValue <= 0 ? NaN : Math.log(inputValue);
  } else if (type === "log") {
    result = inputValue <= 0 ? NaN : Math.log10(inputValue);
  } else if (type === "abs") {
    result = Math.abs(inputValue);
  } else if (type === "exp") {
    result = Math.exp(inputValue);
  } else if (type === "floor") {
    result = Math.floor(inputValue);
  } else if (type === "ceil") {
    result = Math.ceil(inputValue);
  } else if (type === "rand") {
    result = Math.random();
  } else if (type === "pow10") {
    result = Math.pow(10, inputValue);
  } else if (type === "fact") {
    result = factorial(inputValue);
  }

  displayValue.value = formatNumber(result);
  waitingForNewValue.value = true;
};

const insertConstant = (constant) => {
  let value = 0;
  if (constant === "pi") {
    value = Math.PI;
  } else if (constant === "e") {
    value = Math.E;
  }

  displayValue.value = formatNumber(value);
  waitingForNewValue.value = false;
};

const toggleAngle = () => {
  angleUnit.value = angleUnit.value === "deg" ? "rad" : "deg";
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

const memoryStore = () => {
  const inputValue = Number(displayValue.value);
  if (!Number.isFinite(inputValue)) {
    return;
  }

  memoryValue.value = inputValue;
  memoryActive.value = true;
};
</script>
