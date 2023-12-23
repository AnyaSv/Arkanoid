<template>
    <h1>Arkanoid</h1>
    <GameBoard  />
 </template>
 
 <script setup>
 import GameBoard from './components/GameBoard.vue'
 import { ref } from 'vue'
 
 const score = ref(0)
 
 function onScore()
 {
    score.value++
 }
 </script>