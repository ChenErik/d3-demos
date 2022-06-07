<script setup lang="ts">
import { ref } from 'vue'
import Demo1 from './components/demo1.vue'
import Demo2 from './components/demo2.vue'
import Logo from './components/Logo.vue'
const start = ref(23)
const end = ref(71)
const inputNum = ref(0)
</script>

<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <Demo1 v-model:start="start" v-model:end="end" />
  <Demo2 v-model="inputNum" text="test" :min="0" :max="160" :standard="100" />
  <Logo />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
