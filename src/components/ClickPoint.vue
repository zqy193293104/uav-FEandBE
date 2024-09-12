<template>
  <div>
    <!-- 按钮容器 -->
    <div class="button-container">
      <!-- 启用点击放置按钮 -->
      <button @click="enablePointPlacement">设置路径点</button>

      <!-- 清除所有点和线按钮 -->
      <button @click="clearPointsAndLines">清除所有路径</button>

      <!-- 发送 GPS 数据按钮 -->
      <button @click="connectAndSendGpsData">发送 GPS 数据到服务器</button>
    </div>

    <!-- 点击区域 -->
    <div class="click-area" @mousemove="updateCoordinates" @click="placePoint" ref="clickArea">
      <!-- 显示所有点击点 -->
      <div
        v-for="(point, index) in pixelPoints"
        :key="index"
        class="point"
        :style="{ top: point.y + 'px', left: point.x + 'px' }"
      ></div>
      
      <!-- SVG用于绘制连线 -->
      <svg class="line-canvas">
        <line
          v-for="(line, index) in lines"
          :key="'line-' + index"
          :x1="line.x1"
          :y1="line.y1"
          :x2="line.x2"
          :y2="line.y2"
          stroke="blue"
          stroke-width="2"
        />
      </svg>

      <!-- 显示 GPS 坐标 -->
      <div class="gps-display">
        GPS: {{ gps.lat.toFixed(6) }}, {{ gps.lng.toFixed(6) }}
      </div>
    </div>
  </div>
</template>



<script>
export default {
  data() {
    return {
      enableClick: false, // 控制是否启用点击放置点
      pixelPoints: [], // 存储点击时的像素坐标 (x, y)
      gpsPoints: [], // 存储点击点的GPS坐标 (lat, lng)
      gps: { lat: 0, lng: 0 }, // 当前显示的GPS坐标
      lines: [], // 存储连线的起始和终止坐标
      gridWidth: 800, // 网格的宽度
      gridHeight: 600, // 网格的高度
      gridTopLeftGps: { lat: 37.7749, lng: -122.4194 }, // 左上角GPS坐标
      gridBottomRightGps: { lat: 37.7049, lng: -122.3494 }, // 右下角GPS坐标
      clickCount: 0, // 记录点击次数
      socket: null, // WebSocket 实例
    };
  },
  methods: {
    calculateGpsCoordinates(relativeX, relativeY, gridRect) {
      const latRange = this.gridTopLeftGps.lat - this.gridBottomRightGps.lat;
      const lngRange = this.gridBottomRightGps.lng - this.gridTopLeftGps.lng;
      const lat = this.gridTopLeftGps.lat - (relativeY / gridRect.height) * latRange;
      const lng = this.gridTopLeftGps.lng + (relativeX / gridRect.width) * lngRange;
      return { lat, lng };
    },
    enablePointPlacement() {
      this.enableClick = true; // 启用点击放置点
      // this.clickCount = 0; // 重置点击计数
    },
    placePoint(event) {
      if (this.enableClick) {
        const rect = this.$refs.clickArea.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;

        // this.clickCount++;

        // if (this.clickCount === 1) {
        //   console.log("第一个点击未保存");
        //   return; // 不保存第一个点击
        // }

        const gpsCoords = this.calculateGpsCoordinates(x, y, rect);
        this.pixelPoints.push({ x, y });
        this.gpsPoints.push(gpsCoords);

        // 如果有足够的点，绘制连线
        if (this.pixelPoints.length > 1) {
          const previousPoint = this.pixelPoints[this.pixelPoints.length - 2];
          const currentPoint = this.pixelPoints[this.pixelPoints.length - 1];
          this.lines.push({
            x1: previousPoint.x,
            y1: previousPoint.y,
            x2: currentPoint.x,
            y2: currentPoint.y
          });
        }

        console.log('像素坐标', this.pixelPoints);
        console.log('GPS坐标', this.gpsPoints);
      }
    },
    updateCoordinates(event) {
      const rect = this.$refs.clickArea.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;
      this.gps = this.calculateGpsCoordinates(x, y, rect);
    },
    clearPointsAndLines() {
      this.pixelPoints = [];
      this.gpsPoints = [];
      this.lines = [];
      // this.clickCount = 0;
      console.log("所有点和线已清除");
    },
    connectAndSendGpsData() {
      // 创建 WebSocket 连接
      this.socket = new WebSocket("ws://your-websocket-server-url"); // 替换为你的 WebSocket 服务器 URL

      // 当连接建立后，发送 GPS 数据
      this.socket.onopen = () => {
        console.log("WebSocket 连接已建立");

        // 将 gpsPoints 转换为 JSON 格式
        const gpsDataJson = JSON.stringify(this.gpsPoints);

        // 发送 JSON 数据到服务器
        this.socket.send(gpsDataJson);
        console.log("GPS 数据已发送:", gpsDataJson);
      };

      // 监听 WebSocket 接收到消息
      this.socket.onmessage = (event) => {
        console.log("接收到来自服务器的消息：", event.data);
      };

      // 监听 WebSocket 连接关闭
      this.socket.onclose = () => {
        console.log("WebSocket 连接已关闭");
      };
    }
  }
};

</script>

<style scoped>
.click-area {
  top: 0;
  position: absolute;
  width: 100vw;
  height: 1400px;
  border: none;
  margin: 0;
  padding: 0;
  z-index: 1000;
}

.point {
  position: absolute;
  width: 10px;
  height: 10px;
  background-color: red;
  border-radius: 50%;
  z-index: 1001;
}

.line-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
  pointer-events: none; /* 防止SVG阻挡点击 */
  color: blueviolet;
}

.gps-display {
  position: fixed;
  bottom: 10px;
  right: 10px;
  background-color: rgba(55, 151, 101, 0.7);
  color: white;
  padding: 10px;
  border-radius: 5px;
  z-index: 1002;
}
.button-container {
  display: flex;
  justify-content: center; /* 按钮居中对齐 */
  align-items: center;
  padding: 10px;
  background-color: transparent; /* 设置背景为透明 */
  position: absolute; /* 固定在页面顶部 */
  top: 0;
  width: 100%; /* 占据整个页面宽度 */
  z-index: 1003; /* 确保在最上层 */
  box-shadow: none; /* 移除阴影效果 */
}

button {
  margin: 0 5px; /* 每个按钮之间的间距为5px */
  padding: 10px 20px;
  font-size: 14px;
  cursor: pointer;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #0056b3;
}

</style>
