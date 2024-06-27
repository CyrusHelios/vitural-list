<script setup lang="ts">
import faker from "faker";
import VituralList from "./components/VituralList.vue";
import DynamicVituralList from "./components/DynaimcVituralList.vue";
const itemSize = 50; // 每项的高度
const estimatedItemSize = 75; // 预估每项的高度
const listData: Record<"id" | "value", number>[] = [];
for (let i = 0; i < 1000; i++) {
  listData.push({ id: i, value: i });
}
const dynamicListData: Record<"id" | "value", number>[] = [];
for (let i = 0; i < 10000; i++) {
  dynamicListData.push({ id: i, value: faker.lorem.sentences() });
}
</script>

<template>
  <div class="container">
    <VituralList :item-size="itemSize" :list-data="listData"></VituralList>
    <DynamicVituralList
      :estimated-item-size="estimatedItemSize"
      :list-data="dynamicListData"
    ></DynamicVituralList>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  gap: 32px;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
