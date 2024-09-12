<template>
  <div>
    <!-- 引入 GPS 和点击放置点功能的两个组件 -->
    <GridWithGPS />
    <ClickPoint ref="clickPoint" />
  </div>
</template>

<script>
import GridWithGPS from './components/GridWithGPS.vue';
import ClickPoint from './components/ClickPoint.vue';

export default {
  components: {
    GridWithGPS,
    ClickPoint
  }
};
</script>

<style>
</style>
