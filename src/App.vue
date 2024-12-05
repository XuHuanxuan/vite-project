<script setup>
import Calculator from "./components/Calculator.vue"
import Calculator from "./components/Game2048.vue"
</script>

<template>
  <div class="container">
    <h1>Vue Calculator</h1>
    <Calculator />
    <div class="card">
      <Game2048 />
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 30px;
}
</style>
