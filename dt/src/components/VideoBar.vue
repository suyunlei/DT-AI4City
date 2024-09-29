<template>
    <div class="video-container">
      <video controls width="100%">
        <source src="dt/src/assets/video/path5.mp4" type="video/mp4" />
        Your browser does not support the video tag.
      </video>
    </div>
  </template>
  
  <script>
  export default {
    name: "VideoBar",
  };
  </script>
  
  <style scoped>
  .video-container {
    width: 100%;
    padding: 10px;
    background-color: #fff;
  }
  </style>