<template>
    <div class="video-container">
      <video controls width="100%">
        <source :src="videoUrl[0]" type="video/mp4" />
        Your browser does not support the video tag.
      </video>
    </div>
</template>
  
  <script>
  export default {
    name: "VideoBar",
    data() {
      return {
        videoUrl: [
            "http://localhost:8080/ai4city/path5.mp4",
        ],
      };
    },
  };
  </script>
  
  <style scoped>
  .video-container {
    width: 100%;
    padding: 10px;
    background-color: #fff;
  }
  </style>