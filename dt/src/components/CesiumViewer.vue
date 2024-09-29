<template>
    <div id="cesiumContainer" class="cesium-container"></div>
</template>

<script>
export default {
  name: 'CesiumViewer',
    data() {
        return {
            viewer: null,
            tianhebuildingUrl: 'http://localhost:8080/ai4city/3dtiles_output/tileset.json', //换成你自己tomcat的地址
            digitalManUrl: 'http://localhost:8080/ai4city/Cesium_Man.glb',
            // hkustBuildingUrl: 'http://localhost:8080/ai4city/HKUSTData/tileset.json',
            // taiyuanModelUrl: 'http://localhost:8080/ai4city/taiyuan_gltf/modelinfo.json',
        };
    },
    mounted() {
        Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI1MGNiNDIzYy1hYzNjLTRmYzItOTk4ZS0wZjJjYzhjMDAwMTAiLCJpZCI6NzE0ODYsImlhdCI6MTYzNTIzNzg3Mn0.i0iTqEVPssK9EGZWU5_wdYSN_1ZObmwsu00Y29b6N0A';
        this.viewer = new Cesium.Viewer('cesiumContainer',{
            animation:false,       //是否显示动画控件
            homeButton:false,       //是否显示home键
            geocoder:false,         //是否显示地名查找控件，如果设置为true，则无法查询
            baseLayerPicker:false, //是否显示图层选择控件
            timeline:false,        //是否显示时间线控件
            fullscreenButton:true, //是否全屏显示
            infoBox:true,         //是否显示点击要素之后显示的信息
            sceneModePicker:false,  //是否显示投影方式控件  三维/二维
            navigationInstructionsInitiallyVisible:false, //导航指令
            navigationHelpButton:false,     //是否显示帮助信息控件
            selectionIndicator:false, //是否显示指示器组件
        });
        //隐藏cesium的logo
        this.viewer._cesiumWidget._creditContainer.style.display = "none";

        this.load3dtiles();
        // this.loadDigitalMan();
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

        /**
         * 加载数字人
         * @returns {Promise<void>}
         * @description 从URL加载模型
         */
        loadDigitalMan() {
            // 已知的经纬度轨迹
            const positions = [
                { longitude: 113.3335155, latitude: 23.13276097, height: 0 },
                { longitude: 113.3344576, latitude: 23.1329691, height: 0 },
                // 可以继续添加更多的轨迹点
            ];

            // 轨迹时间段
            const startTime = Cesium.JulianDate.now();
            const totalTimeInSeconds = 60;
            const stopTime = Cesium.JulianDate.addSeconds(startTime, totalTimeInSeconds, new Cesium.JulianDate());

            // 设置Cesium动画的时间范围
            this.viewer.clock.startTime = startTime.clone();
            this.viewer.clock.stopTime = stopTime.clone();
            this.viewer.clock.currentTime = startTime.clone();
            this.viewer.clock.clockRange = Cesium.ClockRange.LOOP_STOP; // 循环播放
            this.viewer.clock.multiplier = 1; // 时间加速倍数，设为1确保按真实时间播放

            // 启动时钟的动画
            this.viewer.clock.shouldAnimate = true;

            // 创建 SampledPositionProperty 来储存平滑轨迹
            const positionProperty = new Cesium.SampledPositionProperty();

            // 根据轨迹点的数量计算每个点的时间间隔
            const timeInterval = totalTimeInSeconds / (positions.length - 1);

            // 遍历已知轨迹，设置每个位置的时间和坐标
            positions.forEach((pos, index) => {
                const time = Cesium.JulianDate.addSeconds(startTime, index * timeInterval, new Cesium.JulianDate()); // 根据新时间间隔设置时间
                const position = Cesium.Cartesian3.fromDegrees(pos.longitude, pos.latitude, pos.height);
                positionProperty.addSample(time, position);
            });

            // 加载GLB模型
            const entity = this.viewer.entities.add({
                position: positionProperty,
                orientation: new Cesium.VelocityOrientationProperty(positionProperty),
                model: {
                    uri: this.digitalManUrl, // 替换为你的GLB文件路径
                    scale: 1.0,
                },
            });

            // 让摄像头跟随模型
            this.viewer.trackedEntity = entity;
        },
    }
}

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