<script lang="ts" setup>
import { reactive } from "vue";
import { useRouter } from "vue-router";

// `useRouter` を利用してルーターを取得する
const router = useRouter();
const state = reactive({ name: "" });

const startGame = () => {
  // ルーターの `push` を利用してゲームページに遷移する
  // また、画面遷移の際にユーザ名をクエリでゲームページに渡す
  router.push({ name: "PlayPage", query: { name: state.name } });
};
</script>

<template>
  <h1>トップ画面</h1>
  <input
    v-model="state.name"
    placeholder="名前を入力してください (10文字以内)"
    maxlength="10"
  />
  <button v-on:click="startGame()">ゲームスタート！</button>
  <p>{{ state.name }}</p>
</template>

<style scoped>
p {
  font-size: 1.2em;
  font-weight: bold;
}
</style>