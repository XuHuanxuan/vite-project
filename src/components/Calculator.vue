<template>
  <div class="calculator">
    <div class="display">
      <div class="expression">{{ expression || '0' }}</div>
      <div class="result">{{ display || '0' }}</div>
    </div>
    <div class="mode-switch">
      <button @click="toggleMode" :class="{ active: scientificMode }">Scientific Mode</button>
    </div>
    <div class="buttons" :class="{ 'scientific-mode': scientificMode }">
      <template v-if="scientificMode">
        <button @click="square" class="function">x²</button>
        <button @click="squareRoot" class="function">√</button>
        <button @click="power" class="function">xʸ</button>
        <button @click="factorial" class="function">n!</button>
        
        <button @click="sin" class="function">sin</button>
        <button @click="cos" class="function">cos</button>
        <button @click="tan" class="function">tan</button>
        <button @click="log" class="function">log</button>
        
        <button @click="ln" class="function">ln</button>
        <button @click="pi" class="function">π</button>
        <button @click="e" class="function">e</button>
        <button @click="abs" class="function">|x|</button>
      </template>
      
      <button @click="clear" class="operator">C</button>
      <button @click="changeSign" class="operator">±</button>
      <button @click="percentage" class="operator">%</button>
      <button @click="divide" class="operator">÷</button>
      
      <button @click="append('7')" class="number">7</button>
      <button @click="append('8')" class="number">8</button>
      <button @click="append('9')" class="number">9</button>
      <button @click="multiply" class="operator">×</button>
      
      <button @click="append('4')" class="number">4</button>
      <button @click="append('5')" class="number">5</button>
      <button @click="append('6')" class="number">6</button>
      <button @click="subtract" class="operator">-</button>
      
      <button @click="append('1')" class="number">1</button>
      <button @click="append('2')" class="number">2</button>
      <button @click="append('3')" class="number">3</button>
      <button @click="add" class="operator">+</button>
      
      <button @click="append('0')" class="number zero">0</button>
      <button @click="dot" class="number">.</button>
      <button @click="equal" class="operator">=</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Calculator',
  data() {
    return {
      display: '',
      expression: '',
      previousValue: null,
      operator: null,
      newNumber: false,
      scientificMode: false
    }
  },
  methods: {
    toggleMode() {
      this.scientificMode = !this.scientificMode
    },
    clear() {
      this.display = ''
      this.expression = ''
      this.previousValue = null
      this.operator = null
      this.newNumber = false
    },
    append(number) {
      if (this.newNumber) {
        this.display = ''
        this.newNumber = false
      }
      this.display = `${this.display}${number}`
      if (!this.operator) {
        this.expression = this.display
      }
    },
    updateDisplay(value, expressionText) {
      this.display = `${parseFloat(value.toFixed(8))}`
      this.expression = expressionText
      this.newNumber = true
    },
    square() {
      const value = parseFloat(this.display)
      const result = value * value
      this.updateDisplay(result, `sqr(${value})`)
    },
    squareRoot() {
      const value = parseFloat(this.display)
      const result = Math.sqrt(value)
      this.updateDisplay(result, `√(${value})`)
    },
    power() {
      this.setPendingOperation('^')
    },
    factorial() {
      const value = parseInt(this.display)
      let result = 1
      for(let i = 2; i <= value; i++) {
        result *= i
      }
      this.updateDisplay(result, `${value}!`)
    },
    sin() {
      const value = parseFloat(this.display)
      const result = Math.sin(value * Math.PI / 180) // 使用角度
      this.updateDisplay(result, `sin(${value}°)`)
    },
    cos() {
      const value = parseFloat(this.display)
      const result = Math.cos(value * Math.PI / 180)
      this.updateDisplay(result, `cos(${value}°)`)
    },
    tan() {
      const value = parseFloat(this.display)
      const result = Math.tan(value * Math.PI / 180)
      this.updateDisplay(result, `tan(${value}°)`)
    },
    log() {
      const value = parseFloat(this.display)
      const result = Math.log10(value)
      this.updateDisplay(result, `log(${value})`)
    },
    ln() {
      const value = parseFloat(this.display)
      const result = Math.log(value)
      this.updateDisplay(result, `ln(${value})`)
    },
    pi() {
      this.display = Math.PI.toString()
      this.expression = 'π'
    },
    e() {
      this.display = Math.E.toString()
      this.expression = 'e'
    },
    abs() {
      const value = parseFloat(this.display)
      const result = Math.abs(value)
      this.updateDisplay(result, `|${value}|`)
    },
    setPendingOperation(operator) {
      if (this.previousValue === null) {
        this.previousValue = parseFloat(this.display)
      } else if (!this.newNumber) {
        this.calculate()
      }
      this.operator = operator
      this.expression = `${this.previousValue} ${operator}`
      this.newNumber = true
    },
    calculate() {
      const current = parseFloat(this.display)
      let result = 0
      switch (this.operator) {
        case '+':
          result = this.previousValue + current
          break
        case '-':
          result = this.previousValue - current
          break
        case '×':
          result = this.previousValue * current
          break
        case '÷':
          result = this.previousValue / current
          break
        case '^':
          result = Math.pow(this.previousValue, current)
          break
      }
      this.display = `${parseFloat(result.toFixed(8))}`
      this.previousValue = result
    },
    add() {
      this.setPendingOperation('+')
    },
    subtract() {
      this.setPendingOperation('-')
    },
    multiply() {
      this.setPendingOperation('×')
    },
    divide() {
      this.setPendingOperation('÷')
    },
    equal() {
      if (this.operator && !this.newNumber) {
        this.expression = `${this.previousValue} ${this.operator} ${this.display} =`
        this.calculate()
        this.previousValue = null
        this.operator = null
        this.newNumber = true
      }
    }
  }
}
</script>

<style scoped>
.calculator {
  width: 280px;
  margin: 0 auto;
  padding: 20px;
  border-radius: 10px;
  background-color: #f0f0f0;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.display {
  background-color: #fff;
  padding: 15px;
  text-align: right;
  margin-bottom: 20px;
  border-radius: 5px;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
}

.expression {
  font-size: 16px;
  color: #666;
  min-height: 20px;
  margin-bottom: 5px;
}

.result {
  font-size: 28px;
  color: #333;
  min-height: 35px;
}

.mode-switch {
  margin-bottom: 15px;
  text-align: center;
}

.mode-switch button {
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  background-color: #e0e0e0;
  cursor: pointer;
  transition: background-color 0.2s;
}

.mode-switch button.active {
  background-color: #4CAF50;
  color: white;
}

.buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.buttons.scientific-mode {
  grid-template-columns: repeat(4, 1fr);
}

button {
  padding: 15px;
  font-size: 18px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.2s, transform 0.1s;
}

button:active {
  transform: scale(0.95);
}

.number {
  background-color: #fff;
  color: #333;
}

.number:hover {
  background-color: #f8f8f8;
}

.operator {
  background-color: #ff9500;
  color: white;
}

.operator:hover {
  background-color: #ffaa33;
}

.function {
  background-color: #4CAF50;
  color: white;
}

.function:hover {
  background-color: #45a049;
}

.zero {
  grid-column: span 2;
}
</style>
