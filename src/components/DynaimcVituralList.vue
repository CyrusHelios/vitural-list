<script setup lang="ts">
import { computed, ref, onMounted, watch, nextTick } from "vue";
interface Props {
  estimatedItemSize: number;
  listData: Array<Record<"id" | "value", number>>;
}
interface itemPosition {
  index: number; // 列表项的索引
  height: number; // 列表的真实高度
  top: number; // 列表项上边框距离顶部的偏移量，后续就是渲染区域滚回的距离
  bottom: number; // 列表项下边框距离顶部的偏移量，后续通过 bottom 找到开始渲染的索引
  updated: boolean; // 是否重新计算过
}
const props = defineProps<Props>();

const bufferCount = 1;
const screenHeight = 450; // 可视区域的高度
const visibleCount = Math.ceil(screenHeight / props.estimatedItemSize); // 可显示的列表项数

const startIndex = ref<number>(0); // 开始索引
const itemPosition: itemPosition[] = [];
const containerRef = ref();
const itemRef = ref();
const listHeight = ref<number>(0); // 总高度

const endIndex = computed(() =>
  Math.min(startIndex.value + visibleCount + bufferCount, props.listData.length)
); // 结束索引
const startRenderedIndex = computed(() =>
  Math.max(0, startIndex.value - bufferCount)
); // 开始渲染的索引
const visibleData = computed(() => {
  return props.listData.slice(startRenderedIndex.value, endIndex.value);
}); // 列表的渲染数据
const startOffset = computed(() => {
  // 获取列表距离顶部的偏移量
  return itemPosition[startRenderedIndex.value].top;
}); // 渲染区域的偏移量

// 初始化position数据
props.listData.forEach((_, index) => {
  itemPosition[index] = {
    index,
    top: props.estimatedItemSize * index,
    height: props.estimatedItemSize,
    updated: false,
    bottom: (index + 1) * props.estimatedItemSize,
  };
});

onMounted(() => {
  updateItemPosition();
});

watch(startIndex, () => {
  nextTick(() => {
    updateItemPosition();
  });
});

const updateItemPosition = () => {
  const nodes = itemRef.value;
  let isNeedUpdateAll = false;
  for (const node of nodes) {
    const index = +(node.dataset.index ?? "");
    const isUpdated = itemPosition[index].updated;
    if (isUpdated) continue;
    const domHeight = getDOMHeight(node);
    itemPosition[index].updated = true;
    isNeedUpdateAll = true; // 后面的节点需要更新信息
    itemPosition[index].height = domHeight;
  }
  if (isNeedUpdateAll) {
    const startUpdateIndex = +(nodes[0].dataset.index ?? "");

    for (let i = startUpdateIndex; i < itemPosition.length; i++) {
      itemPosition[i].top = i > 0 ? itemPosition[i - 1].bottom : 0;
      itemPosition[i].bottom = itemPosition[i].top + itemPosition[i].height;
    }
  }

  listHeight.value = itemPosition[itemPosition.length - 1].bottom;
};

const onScroll = () => {
  // 获取渲染的起始索引
  startIndex.value = getStartIndex(containerRef.value.scrollTop);
};
// 在 list 数组中到第一个大于等于 value 的索引
const binarySearch = (list: Array<itemPosition>, value: number): number => {
  let start = 0,
    end = list.length,
    mid = 0;

  while (start < end) {
    mid = Math.floor((start + end) / 2);
    if (list[mid].bottom < value) {
      start = mid + 1;
    } else {
      end = mid;
    }
  }

  return +(list[end].index || 0);
};
const getCssVarialbes = (value: number) => value + "px";
const getDOMHeight = (node: Element) => node.getBoundingClientRect().height;
const getStartIndex = (scrollTop: number) => {
  return binarySearch(itemPosition, scrollTop);
};
</script>
<template>
  <div class="infinite-list-container" ref="containerRef" @scroll="onScroll">
    <div class="infinite-list-phantom"></div>
    <div
      class="infinite-list"
      :style="{ transform: `translate3d(0px, ${startOffset}px, 0px)` }"
    >
      <div
        class="infinite-list-item"
        v-for="item in visibleData"
        ref="itemRef"
        :data-index="item.id"
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
    padding: 16px 0;
    border-bottom: 1px solid #999;
  }

  &-container {
    position: relative;
    overflow: auto;
    margin: 0 auto;
    width: 400px;
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
