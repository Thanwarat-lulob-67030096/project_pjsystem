<template>
  <div class="calculator-wrapper">
    <div class="calculator">
      <div class="calc-header">
        <div class="header-icon">
          <svg
            width="18"
            height="18"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <rect
              x="4"
              y="2"
              width="16"
              height="20"
              rx="2"
            />
            <line
              x1="8"
              y1="6"
              x2="16"
              y2="6"
            />
            <line
              x1="8"
              y1="10"
              x2="10"
              y2="10"
            />
            <line
              x1="14"
              y1="10"
              x2="16"
              y2="10"
            />
            <line
              x1="8"
              y1="14"
              x2="10"
              y2="14"
            />
            <line
              x1="14"
              y1="14"
              x2="16"
              y2="14"
            />
            <line
              x1="8"
              y1="18"
              x2="10"
              y2="18"
            />
            <line
              x1="14"
              y1="18"
              x2="16"
              y2="18"
            />
          </svg>
        </div>
        <span class="header-title">Calculator</span>
        <div class="header-badge">Basic</div>
      </div>

      <div class="display">
        <div class="expression">{{ expression || '&nbsp;' }}</div>
        <div
          class="result"
          :class="{ 'result-small': display.length > 10 }"
        >
          {{ display }}
        </div>
      </div>

      <div
        v-if="history.length"
        class="history-bar"
      >
        <span class="history-label">Last:</span>
        <span class="history-value">{{ history[history.length - 1] }}</span>
      </div>

      <div class="buttons">
        <button
          class="btn btn-clear span-2"
          @click="clear"
        >
          AC
        </button>
        <button
          class="btn btn-func"
          @click="toggleSign"
        >
          +/-
        </button>
        <button
          class="btn btn-func"
          @click="percent"
        >
          %
        </button>
        <button
          class="btn btn-operator"
          :class="{ active: activeOp === '/' }"
          @click="setOp('/')"
        >
          ÷
        </button>

        <button
          class="btn btn-number"
          @click="input('7')"
        >
          7
        </button>
        <button
          class="btn btn-number"
          @click="input('8')"
        >
          8
        </button>
        <button
          class="btn btn-number"
          @click="input('9')"
        >
          9
        </button>
        <button
          class="btn btn-operator"
          :class="{ active: activeOp === '*' }"
          @click="setOp('*')"
        >
          ×
        </button>

        <button
          class="btn btn-number"
          @click="input('4')"
        >
          4
        </button>
        <button
          class="btn btn-number"
          @click="input('5')"
        >
          5
        </button>
        <button
          class="btn btn-number"
          @click="input('6')"
        >
          6
        </button>
        <button
          class="btn btn-operator"
          :class="{ active: activeOp === '-' }"
          @click="setOp('-')"
        >
          −
        </button>

        <button
          class="btn btn-number"
          @click="input('1')"
        >
          1
        </button>
        <button
          class="btn btn-number"
          @click="input('2')"
        >
          2
        </button>
        <button
          class="btn btn-number"
          @click="input('3')"
        >
          3
        </button>
        <button
          class="btn btn-operator"
          :class="{ active: activeOp === '+' }"
          @click="setOp('+')"
        >
          +
        </button>

        <button
          class="btn btn-number span-2"
          @click="input('0')"
        >
          0
        </button>
        <button
          class="btn btn-number"
          @click="inputDot"
        >
          .
        </button>
        <button
          class="btn btn-equals"
          @click="calculate(false)"
        >
          =
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'VuexyCalculator',
  data() {
    return {
      display: '0',
      expression: '',
      currentVal: '0',
      prevVal: null,
      operator: null,
      waitingForOperand: false,
      activeOp: null,
      history: [],
      justCalculated: false
    }
  },
  methods: {
    input(digit) {
      if (this.justCalculated) {
        this.currentVal = digit
        this.expression = ''
        this.justCalculated = false
      } else if (this.waitingForOperand) {
        this.currentVal = digit
        this.waitingForOperand = false
      } else {
        this.currentVal = this.currentVal === '0' ? digit : this.currentVal + digit
      }
      this.display = this.currentVal
    },
    inputDot() {
      if (this.waitingForOperand) {
        this.currentVal = '0.'
        this.waitingForOperand = false
        this.display = this.currentVal
        return
      }
      if (!this.currentVal.includes('.')) {
        this.currentVal += '.'
        this.display = this.currentVal
      }
    },
    setOp(op) {
      if (this.prevVal !== null && !this.waitingForOperand) {
        this.calculate(true)
      }
      this.prevVal = parseFloat(this.currentVal)
      this.operator = op
      this.activeOp = op
      this.waitingForOperand = true
      this.justCalculated = false

      const symbols = { '+': '+', '-': '−', '*': '×', '/': '÷' }
      this.expression = `${this.formatNum(this.prevVal)} ${symbols[op]}`
    },
    calculate(silent = false) {
      if (this.prevVal === null || this.operator === null) return

      const a = this.prevVal
      const b = parseFloat(this.currentVal)
      const symbols = { '+': '+', '-': '−', '*': '×', '/': '÷' }
      let result

      switch (this.operator) {
        case '+':
          result = a + b
          break
        case '-':
          result = a - b
          break
        case '*':
          result = a * b
          break
        case '/':
          result = b !== 0 ? a / b : 'Error'
          break
        default:
          return
      }

      if (!silent) {
        const expr = `${this.formatNum(a)} ${symbols[this.operator]} ${this.formatNum(b)} =`
        this.expression = expr
        if (result !== 'Error') {
          this.history.push(
            `${this.formatNum(a)} ${symbols[this.operator]} ${this.formatNum(b)} = ${this.formatNum(result)}`
          )
          if (this.history.length > 5) this.history.shift()
        }
        this.activeOp = null
      }

      const resultStr = result === 'Error' ? 'Error' : this.formatNum(result)
      this.display = resultStr
      this.currentVal = result === 'Error' ? '0' : String(result)
      this.prevVal = silent ? result : null
      this.operator = silent ? this.operator : null
      this.waitingForOperand = !silent
      this.justCalculated = !silent
    },
    clear() {
      this.display = '0'
      this.expression = ''
      this.currentVal = '0'
      this.prevVal = null
      this.operator = null
      this.waitingForOperand = false
      this.activeOp = null
      this.justCalculated = false
    },
    toggleSign() {
      const val = parseFloat(this.currentVal)
      this.currentVal = String(-val)
      this.display = this.currentVal
    },
    percent() {
      const val = parseFloat(this.currentVal)
      this.currentVal = String(val / 100)
      this.display = this.formatNum(val / 100)
    },
    formatNum(n) {
      if (n === 'Error') return 'Error'
      const num = parseFloat(n)
      if (isNaN(num)) return '0'
      const str = parseFloat(num.toPrecision(10)).toString()
      return str
    }
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap');

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.calculator-wrapper {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f4f5fa;
  font-family: 'DM Sans', sans-serif;
  padding: 2rem;
}

.calculator {
  width: 360px;
  background: #ffffff;
  border-radius: 20px;
  box-shadow:
    0 4px 24px rgba(115, 103, 240, 0.08),
    0 1px 4px rgba(0, 0, 0, 0.04);
  overflow: hidden;
}

/* Header */
.calc-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 16px 20px 12px;
  border-bottom: 1px solid #f0f0f7;
}

.header-icon {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  background: linear-gradient(135deg, #7367f0, #9e95f5);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  flex-shrink: 0;
}

.header-title {
  font-size: 15px;
  font-weight: 600;
  color: #5e5873;
  flex: 1;
}

.header-badge {
  font-size: 11px;
  font-weight: 500;
  padding: 3px 10px;
  border-radius: 20px;
  background: #ede9fd;
  color: #7367f0;
  letter-spacing: 0.3px;
}

/* Display */
.display {
  padding: 20px 24px 12px;
  background: #fafafa;
  min-height: 110px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-end;
  border-bottom: 1px solid #f0f0f7;
}

.expression {
  font-family: 'DM Mono', monospace;
  font-size: 13px;
  color: #b9b9c3;
  min-height: 20px;
  margin-bottom: 6px;
  word-break: break-all;
  text-align: right;
}

.result {
  font-family: 'DM Mono', monospace;
  font-size: 40px;
  font-weight: 500;
  color: #5e5873;
  word-break: break-all;
  text-align: right;
  line-height: 1;
  transition: font-size 0.15s;
}

.result.result-small {
  font-size: 28px;
}

/* History */
.history-bar {
  padding: 8px 20px;
  background: #f8f7fd;
  border-bottom: 1px solid #f0f0f7;
  display: flex;
  gap: 8px;
  align-items: center;
  overflow: hidden;
}

.history-label {
  font-size: 11px;
  color: #b9b9c3;
  font-weight: 500;
  flex-shrink: 0;
}

.history-value {
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  color: #7367f0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Buttons grid */
.buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1px;
  background: #f0f0f7;
  border-top: 1px solid #f0f0f7;
}

.btn {
  height: 72px;
  border: none;
  cursor: pointer;
  font-family: 'DM Sans', sans-serif;
  font-size: 18px;
  font-weight: 500;
  transition: all 0.12s ease;
  outline: none;
  position: relative;
  overflow: hidden;
}

.btn::after {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(255, 255, 255, 0);
  transition: background 0.12s;
}

.btn:active::after {
  background: rgba(0, 0, 0, 0.06);
}

.span-2 {
  grid-column: span 2;
}

/* Number buttons */
.btn-number {
  background: #ffffff;
  color: #5e5873;
}

.btn-number:hover {
  background: #fafafa;
  color: #4a4368;
}

/* Function buttons (AC, +/-, %) */
.btn-func {
  background: #f4f5fa;
  color: #7367f0;
  font-weight: 600;
}

.btn-func:hover {
  background: #ede9fd;
}

.btn-clear {
  background: #f4f5fa;
  color: #ea5455;
  font-weight: 600;
}

.btn-clear:hover {
  background: #fde9e9;
}

/* Operator buttons */
.btn-operator {
  background: #ffffff;
  color: #7367f0;
  font-size: 22px;
  font-weight: 400;
}

.btn-operator:hover {
  background: #ede9fd;
}

.btn-operator.active {
  background: #7367f0;
  color: #ffffff;
}

/* Equals button */
.btn-equals {
  background: linear-gradient(135deg, #7367f0, #9e95f5);
  color: #ffffff;
  font-size: 22px;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(115, 103, 240, 0.35);
}

.btn-equals:hover {
  background: linear-gradient(135deg, #6a5fe8, #9589f2);
  box-shadow: 0 6px 16px rgba(115, 103, 240, 0.45);
}

.btn-equals:active {
  transform: scale(0.97);
}

/* Rounded corners for edge buttons */
.buttons .btn:first-child {
  border-radius: 0;
}

/* last row rounding */
.buttons .btn:nth-last-child(1) {
  border-radius: 0 0 20px 0;
}
.buttons .btn:nth-last-child(3) {
  border-radius: 0 0 0 20px;
}
</style>