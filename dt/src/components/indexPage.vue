<template>
  <div class="container">
    <!-- VideoBar 主界面 -->
    <VideoBar ref="videoBar"/>

    <!-- 右下角的 CesiumViewer 悬浮框 -->
    <div class="cesium-viewer-window">
      <CesiumViewer ref="cesiumViewer"/>
    </div>
    <!-- 左上角的 ControlBar 悬浮框 -->
    <ControlBar @path-clicked="handlePathClick"/>

    <!-- 右上角的 MapLayer 悬浮框 -->
    <div class="map-layer-window">
      <MapLayer />
    </div>
  </div>
</template>

<script>
import CesiumViewer from './CesiumViewer.vue';
import VideoBar from './VideoBar.vue';
import ControlBar from './ControlBar.vue';
import MapLayer from './MapLayer.vue'; // 引入 MapLayer 组件

export default {
  components: {
    CesiumViewer,
    VideoBar,
    ControlBar,
    MapLayer, // 注册 MapLayer 组件
  },
  data() {
    return {
    };
  },
  methods: {
    handlePathClick(pathName) {
      // 传递路径名给 VideoBar 组件 修改VideoBar中的videoUrl
      this.$refs.videoBar.setVideoUrl(pathName);
      // 传递路径名给 CesiumViewer 组件 触发某个方法
      this.$refs.cesiumViewer.loadDigitalMan(pathName);
    },
  },
};
</script>

<style scoped>
.container {
  position: relative;
  height: 100vh;
  width: 100vw;
}

/* CesiumViewer 悬浮窗的样式 */
.cesium-viewer-window {
  position: absolute;
  bottom: 0px;
  right: 0px;
  width: 600px;
  height: 300px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

/* MapLayer 悬浮窗的样式 */
.map-layer-window {
  position: absolute;
  top: 0px;
  right: 0px;
  width: 600px;
  height: 300px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

.open-btn,
.close-btn {
  position: absolute;
  bottom: 0px;
  right: 0px;
  z-index: 1100;
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px;
  cursor: pointer;
}

.open-btn:hover,
.close-btn:hover {
  background-color: #0056b3;
}

.close-btn {
  bottom: 5px;
  right: 5px;
  background-color: #dc3545;
}

.close-btn:hover {
  background-color: #c82333;
}
</style>