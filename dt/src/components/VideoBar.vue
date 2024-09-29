<template>
  <div class="video-container" ref="videoBar">
    <video ref="videoPlayer" controls width="100%">
      <source :src="localVideoUrl" type="video/mp4" />
      Your browser does not support the video tag.
    </video>
  </div>
</template>

<script>
export default {
  name: "VideoBar",
  data() {
    return {
      localVideoUrl: '',  // 使用局部数据属性存储 videoUrl 的值
    };
  },
  methods: {
    setVideoUrl(pathName) {
      // 根据路径名设置视频地址
      this.localVideoUrl = `http://localhost:8080/ai4city/${pathName}.mp4`;
      this.loadAndPlayVideo();
    },
    loadAndPlayVideo() {
      const videoPlayer = this.$refs.videoPlayer;
      videoPlayer.load();  // 重新加载视频
      videoPlayer.play()
        .catch(error => {
          console.error("Video play was rejected", error);
        });
    },
  },
  mounted() {
    if (this.localVideoUrl) {
      this.loadAndPlayVideo();
    }
  },
};
</script>

<style scoped>
.video-container {
  position: relative;
  width: 100%;
  height: 100%;
}

video {
  width: 100%;
  height: auto;
}
</style>