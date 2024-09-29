<template>
  <div class="container">
    <!-- CesiumViewer 组件 -->
    <CesiumViewer />

    <!-- 悬浮视频框 -->
    <div class="video-bar" v-if="isVideoBarVisible">
      <VideoBar />
      <button class="close-btn" @click="toggleVideoBar">Close</button>
    </div>

    <!-- 控制视频框显示的按钮 -->
    <button class="open-btn" v-if="!isVideoBarVisible" @click="toggleVideoBar">Open Video</button>
  </div>
</template>

<script>
import CesiumViewer from './CesiumViewer.vue';
import VideoBar from './VideoBar.vue';

export default {
  components: {
    CesiumViewer,
    VideoBar,
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
  },
};
</script>

<style scoped>
.container {
  position: relative;
  height: 100vh;
  width: 100vw;
}

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