<template>
  <div class="container">
    <!-- CesiumViewer 组件 -->
    <CesiumViewer ref="cesiumViewer"/>

    <!-- 左上角的 ControlBar 悬浮框 -->
    <ControlBar @path-clicked="handlePathClick"/>

    <!-- 右下角的 VideoBar 悬浮框 -->
    <div class="video-bar" v-if="isVideoBarVisible">
      <VideoBar ref="videoBar"/>
      <button class="close-btn" @click="toggleVideoBar">关闭</button>
    </div>

    <!-- 控制视频框显示的按钮 -->
    <button class="open-btn" v-if="!isVideoBarVisible" @click="toggleVideoBar">打开视频</button>
  </div>
</template>

<script>
import CesiumViewer from './CesiumViewer.vue';
import VideoBar from './VideoBar.vue';
import ControlBar from './ControlBar.vue'; // 引入ControlBar组件

export default {
  components: {
    CesiumViewer,
    VideoBar,
    ControlBar, // 注册ControlBar组件
  },
  data() {
    return {
      isVideoBarVisible: true, // 控制视频框是否显示
    };
  },
  methods: {
    toggleVideoBar() {
      this.isVideoBarVisible = !this.isVideoBarVisible;
    },
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

/* VideoBar 和 Open/Close 按钮的样式保持不变 */
.video-bar {
  position: absolute;
  bottom: 20px;
  right: 20px;
  width: 300px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

.open-btn,
.close-btn {
  position: absolute;
  bottom: 20px;
  right: 20px;
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