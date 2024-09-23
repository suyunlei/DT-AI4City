<template>
    <div id="cesiumContainer" class="cesium-container"></div>
</template>

<script>
export default {
  name: 'CesiumViewer',
    data() {
        return {
            viewer: null,
            // tianhebuildingUrl: 'http://localhost:8080/tianhe_building_model/tileset.json',
            tianhebuildingUrl: 'http://localhost:8080/3dtiles_output/tileset.json',
            hkustBuildingUrl: 'http://localhost:8080/HKUSTData/tileset.json',
            taiyuanModelUrl: 'http://localhost:8080/taiyuan_gltf/modelinfo.json',
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

        this.load3dtiles();
        // this.loadModel();
    },
    methods: {
        /**
         * 加载HKUST建筑物
         * @returns {Promise<void>}
         * @description 从URL加载3D瓦片集
         */
        async load3dtiles(){
            try {
                const tileset = await Cesium.Cesium3DTileset.fromUrl(
                    this.tianhebuildingUrl
                );
                this.viewer.scene.primitives.add(tileset);
                this.viewer.zoomTo(tileset);
            } catch (error) {
                console.error(`Error creating tileset: ${error}`);
            }    // 监听点击事件
            this.viewer.screenSpaceEventHandler.setInputAction((movement) => {
                const pickedFeature = this.viewer.scene.pick(movement.position);
                if (Cesium.defined(pickedFeature)) {
                    // 清除之前选择的实体
                    this.viewer.selectedEntity = null;

                    // // 获取属性
                    // const properties = pickedFeature.getPropertyNames().map(name => {
                    //     return `${name}: ${pickedFeature.getProperty(name)}`;
                    // }).join('\n');
                    // console.log('属性信息:', properties);

                    // 弹出属性框
                    this.viewer.selectedEntity = new Cesium.Entity({
                        description: "its a building",
                    });

                    // 改变颜色为红色
                    pickedFeature.color = Cesium.Color.RED.withAlpha(0.8);
                }
            }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
        },
        /**
         * 加载gltf模型
         * @returns {Promise<void>}
         * @description 从URL加载模型
         */
        async loadModel() {
            try {
                const response = await fetch(this.taiyuanModelUrl);
                const modelData = await response.json();
                console.log(modelData);
                for (const [gltfName, modelInfo] of Object.entries(modelData)) {
                    const gltfUrl = `http://localhost:8080/taiyuan_gltf/${gltfName}`;
                    const { center } = modelInfo;
                    const position = Cesium.Cartesian3.fromDegrees(center[0], center[1]);
                    // 创建旋转矩阵，将模型从竖直方向旋转到水平面
                    const hpr = new Cesium.HeadingPitchRoll(0, Cesium.Math.toRadians(90), 0);

                    const model = await Cesium.Model.fromGltfAsync({
                        url: gltfUrl,
                        modelMatrix: Cesium.Transforms.headingPitchRollToFixedFrame(
                            position,
                            hpr
                        ),
                        scale: 1.0,
                    });
                    this.viewer.scene.primitives.add(model);
                }
            } catch (error) {
                console.error("Failed to load models:", error);
            }
    },

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