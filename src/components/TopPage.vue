<script lang="ts" setup>
import { reactive } from "vue";

// ユーザ名の入力値を保持するため name という状態を保持する
const state = reactive({ name: "" });

// 上記 name の値を alert で表示する関数 (今後ゲームページに遷移するための関数として利用予定)
const startGame = () => {
  alert(`ユーザ名：${state.name}`);
};
</script>

<template>
  <h1>トップ画面</h1>
  <!--
    input タグに v-model という属性を付与することで入力値を state.name に紐付ける
  -->
  <input
    v-model="state.name"
    placeholder="名前を入力してください (10文字以内)"
    maxlength="10"
  />

  <!--
    button タグに v-on:click という属性を付与することでボタンクリック時に
    `<script>` 内で定義した startGame 関数を呼び出す
  -->
  <button v-on:click="startGame()">ゲームスタート！</button>

  <!--
    input タグへの入力値を state.name 経由で常時s確認可能にする (検証用途)
  -->
  <p>{{ state.name }}</p>
</template>

<style>
p {
  font-size: 1.2em;
  font-weight: bold;
}
</style>