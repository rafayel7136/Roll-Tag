<script setup>
import Ao3TagDice from './components/Ao3TagDice.vue'
</script>

<template>
  <main class="page">
    <h1>Roll-Tag</h1>
    <Ao3TagDice />
  </main>
</template>

<style scoped>
.page {
  max-width: 760px;
  margin: 0 auto;
  padding: 40px 20px;
}

h1 {
  font-size: 40px;
  margin-bottom: 24px;
  border-bottom: 1px solid #e5e7eb;
  padding-bottom: 16px;
}
</style>
