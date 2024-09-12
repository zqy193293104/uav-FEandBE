<template>
  <div class="grid-container">
    <!-- 网格内容 -->
    <div v-for="item in 24" :key="item" class="grid-item">
      <div class="inner-grid">
        <div v-for="innerItem in 24" :key="innerItem" class="inner-item">
          Item {{ item }}-{{ innerItem }}
        </div>
      </div>
    </div>

  </div>
</template>

<!-- <script>
export default {
  data() {
    return {
      gps: { lat: 0, lng: 0 }, // 当前显示的GPS坐标
      gridWidth: 800, // 网格的宽度
      gridHeight: 600, // 网格的高度
      gridTopLeftGps: { lat: 37.7749, lng: -122.4194 }, // 左上角GPS坐标
      gridBottomRightGps: { lat: 37.7049, lng: -122.3494 } // 右下角GPS坐标
    };
  },
  methods: {
    updateCoordinates(event) {
      const gridRect = event.currentTarget.getBoundingClientRect();
      const relativeX = event.clientX - gridRect.left;
      const relativeY = event.clientY - gridRect.top;

      // 计算相对比例
      const latRange = this.gridTopLeftGps.lat - this.gridBottomRightGps.lat;
      const lngRange = this.gridBottomRightGps.lng - this.gridTopLeftGps.lng;

      // 计算当前鼠标位置对应的GPS坐标
      const lat = this.gridTopLeftGps.lat - (relativeY / this.gridHeight) * latRange;
      const lng = this.gridTopLeftGps.lng + (relativeX / this.gridWidth) * lngRange;

      this.gps = { lat, lng };
    }
  }
};
</script> -->

<style scoped>
.grid-container {
  display: grid;
  grid-template-columns: repeat(6, 1fr); /* 6列 */
  grid-template-rows: repeat(4, 1fr); /* 4行 */
  gap: 10px;
  margin-top: 50px; /* 设置 margin-top 让网格下移，避免与按钮重叠 */
  z-index: 1;
}

.grid-item {
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  padding: 10px;
  z-index: 1; /* 设置较低的 z-index，使其位于 click-area 之下 */
}

.inner-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr); /* 内部网格为8列 */
  grid-template-rows: repeat(3, 1fr); /* 3行 */
  gap: 2px;
}

.inner-item {
  background-color: #1d8cb1;
  border: 1px solid #1d71df;
  padding: 5px;
  text-align: center;
  z-index: 1;
}

/* .gps-display {
  position: fixed;
  bottom: 10px;
  right: 10px;
  background-color: rgba(55, 151, 101, 0.7);
  color: white;
  padding: 10px;
  border-radius: 5px;
  z-index: 999;
} */
</style>
