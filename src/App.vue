<script setup lang="ts">
</script>

<template>
  <!--
    アクセスするページの URL に応じたコンポーネントを表示する
    ルーターの保持するルーティング情報から表示するコンポーネントは決定される
  -->
  <router-view />
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
