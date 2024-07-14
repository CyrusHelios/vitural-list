<script setup lang="ts">
import { computed, ref } from "vue";
interface Props {
  itemSize: number;
  listData: Array<Record<"id" | "value", number>>;
}
const props = defineProps<Props>();

const bufferCount = 2; // 上下缓冲的数目
const screenHeight = 300; // 可视区域的高度
const listHeight = props.listData.length * props.itemSize; // 总高度
const visibleCount = Math.ceil(screenHeight / props.itemSize); // 可显示的列表项数

const containerRef = ref<HTMLElement>();
const startIndex = ref<number>(0); // 开始索引

const endIndex = computed(() =>
  Math.min(startIndex.value + visibleCount + bufferCount, props.listData.length)
); // 结束索引
const startRenderedIndex = computed(() =>
  Math.max(0, startIndex.value - bufferCount)
); // 开始渲染的索引
const visibleData = computed(() => {
  return props.listData.slice(startRenderedIndex.value, endIndex.value);
}); // 列表显示的数据
/**
 * 由于滚动可视区域也会导致渲染区域的滚动，所以需要将渲染区域滚回可视区域，
 * 假设可视区域的滚动距离为 scrollTop，如果渲染区域总是滚回 scrollTop，
 * 那么渲染区域总是会渲染 startRenderedIndex 开头的完整 item，但大部分情况下，
 * 可视区域滚动之后，item 是不完整显示的，比如滚动距离是 120，那实际滚回距离应该是 100
 * 剩下的 20 是不需要滚回来的，所以就可以得出滚回的距离是 startRenderedIndex * itemSize
 */
const listOffset = computed(() => {
  return startRenderedIndex.value * props.itemSize;
}); // 渲染区域相对于可视区域的滚回距离
const onScroll = () => {
  const scrollTop = containerRef.value!.scrollTop;
  startIndex.value = Math.floor(scrollTop / props.itemSize);
};
const getCssVarialbes = (value: number) => value + "px";
</script>
<template>
  <div class="infinite-list-container" ref="containerRef" @scroll="onScroll">
    <div class="infinite-list-phantom"></div>
    <div
      class="infinite-list"
      :style="{ transform: `translate3d(0px, ${listOffset}px, 0px)` }"
    >
      <div
        class="infinite-list-item"
        v-for="item in visibleData"
        :key="item.id"
      >
        {{ item.value }}
      </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
.infinite-list {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  text-align: center;

  &-item {
    margin: 0;
    border-bottom: 1px solid #999;
    line-height: v-bind(getCssVarialbes(itemSize));
    height: v-bind(getCssVarialbes(itemSize));
    box-sizing: border-box;
  }

  &-container {
    position: relative;
    overflow: auto;
    margin: 0 auto;
    width: 250px;
    height: v-bind(getCssVarialbes(screenHeight));
    border: 1px solid red;
  }

  &-phantom {
    height: v-bind(getCssVarialbes(listHeight));
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
  }
}
</style>
