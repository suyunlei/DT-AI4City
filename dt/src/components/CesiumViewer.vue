<template>
    <div id="cesiumContainer" class="cesium-container"></div>
</template>

<script>
export default {
  name: 'CesiumViewer',
    data() {
        return {
            viewer: null,
            hkustBuildingUrl: 'http://localhost:8080/HKUSTData/tileset.json'
            // hkustBuildingUrl: 'http://localhost:8080/onegeo/tileset.json'
        };
    },
    mounted() {
        Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI1MGNiNDIzYy1hYzNjLTRmYzItOTk4ZS0wZjJjYzhjMDAwMTAiLCJpZCI6NzE0ODYsImlhdCI6MTYzNTIzNzg3Mn0.i0iTqEVPssK9EGZWU5_wdYSN_1ZObmwsu00Y29b6N0A';
        this.viewer = new Cesium.Viewer('cesiumContainer',{
            animation:false,       //是否显示动画控件
            homeButton:true,       //是否显示home键
            geocoder:true,         //是否显示地名查找控件，如果设置为true，则无法查询
            baseLayerPicker:true, //是否显示图层选择控件
            timeline:false,        //是否显示时间线控件
            fullscreenButton:true, //是否全屏显示
            infoBox:true,         //是否显示点击要素之后显示的信息
            sceneModePicker:true,  //是否显示投影方式控件  三维/二维
            navigationInstructionsInitiallyVisible:false, //导航指令
            navigationHelpButton:false,     //是否显示帮助信息控件
            selectionIndicator:false, //是否显示指示器组件
        });
        //隐藏cesium的logo
        this.viewer._cesiumWidget._creditContainer.style.display = "none";
        this.loadHKUSTBuilding();
    },
    methods: {
        async loadHKUSTBuilding(){
            try {
                const tileset = await Cesium.Cesium3DTileset.fromUrl(
                    this.hkustBuildingUrl
                );
                this.viewer.scene.primitives.add(tileset);
                this.viewer.zoomTo(tileset);
            } catch (error) {
                console.error(`Error creating tileset: ${error}`);
            }

                // 监听点击事件
            this.viewer.screenSpaceEventHandler.setInputAction((movement) => {
                const pickedFeature = this.viewer.scene.pick(movement.position);
                if (Cesium.defined(pickedFeature)) {
                    // 清除之前选择的实体
                    this.viewer.selectedEntity = null;
                    console.log(pickedFeature);
                }
            }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
        }
    }
};
</script>

<style>
.cesium-container {
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>