<template>
  <div class="control-bar">
    <h3>社区列表</h3>
    <ul>
      <li v-for="(community, index) in communities" :key="index">
        <div @click="toggleCommunity(index)">
          {{ community.name }}
        </div>
        <ul v-if="community.expanded">
          <li v-for="(path, pathIndex) in community.paths" :key="pathIndex" @click="clickPath(index, pathIndex)">
            {{ path }}
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "ControlBar",
  data() {
    return {
      communities: [
        {
          name: "石牌村",
          expanded: false,
          paths: ["path4", "path2"],
        },
        {
          name: "侨源阁",
          expanded: false,
          paths: ["path5", "Path2"],
        },
      ],
    };
  },
  methods: {
    toggleCommunity(index) {
      this.communities[index].expanded = !this.communities[index].expanded;
    },
    clickPath(communityIndex, pathIndex) {
        // 获取当前社区的路径名
        const pathName = this.communities[communityIndex].paths[pathIndex];
        // 触发自定义事件，传递路径名给父组件
        this.$emit('path-clicked', pathName);
    },
  },
};
</script>

<style scoped>
.control-bar {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 200px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  padding: 10px;
}

.control-bar h3 {
  margin: 0 0 10px;
  font-size: 18px;
}

.control-bar ul {
  list-style: none;
  padding: 0;
}

.control-bar ul li {
  cursor: pointer;
  margin: 5px 0;
}

.control-bar ul li ul {
  margin-left: 15px;
}
</style>