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

            position_array: [
                [
                    { longitude: 113.3271608, latitude: 23.15780621, height: 0 },
                    { longitude: 113.3262337, latitude: 23.15830679, height: 0 },
                ], //path5
                [
                    { longitude: 113.325142, latitude: 23.15790303, height: 0 },
                    { longitude: 113.3255582, latitude: 23.15703157, height: 0 },
                ], // path4
                [
                    { longitude: 113.3356463, latitude: 23.13050184, height: 0 },
                    { longitude: 113.3357251, latitude: 23.13125939, height: 0 },
                ] // path3
            ],

            running_man: null, //正在运动的小人
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
            fullscreenButton:false, //是否全屏显示
            infoBox:true,         //是否显示点击要素之后显示的信息
            sceneModePicker:false,  //是否显示投影方式控件  三维/二维
            navigationInstructionsInitiallyVisible:false, //导航指令
            navigationHelpButton:false,     //是否显示帮助信息控件
            selectionIndicator:false, //是否显示指示器组件
        });
        //隐藏cesium的logo
        this.viewer._cesiumWidget._creditContainer.style.display = "none";

        this.viewer.clock.currentTime = Cesium.JulianDate.fromIso8601('2024-09-30T10:00:00Z');

        this.load3dtiles();
        this.addPoints();
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
                // this.viewer.zoomTo(tileset);
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
        loadDigitalMan(path_name) {
            // 移除之前的小人
            if(this.running_man){
                this.viewer.entities.remove(this.running_man);
            }

            let positions = undefined;
            if(path_name == '侨源阁实验区路线'){
                positions = this.position_array[0];
            } else if(path_name == '石牌村实验区路线'){
                positions = this.position_array[2];
            } else {
                console.error('路径名错误');
                return;
            }

            // 轨迹时间段
            const startTime = Cesium.JulianDate.now();
            const totalTimeInSeconds = 80;
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
                    scale: 3.0,
                },
            });
            console.log('小人加载完成',entity);
            this.running_man = entity; //存储当前小人
            this.viewer.trackedEntity = entity; //跟踪小人
            // // 将视角flyto到小人位置
            // this.viewer.flyTo(entity, {
            //     offset: new Cesium.HeadingPitchRange(0, Cesium.Math.toRadians(-40), 20),
            // });
            // // 监听鼠标点击事件
            // this.viewer.screenSpaceEventHandler.setInputAction((movement) => {
            //     const pickedFeature = this.viewer.scene.pick(movement.position);
            //     if (Cesium.defined(pickedFeature)) {
            //         console.log('pickedFeature:', pickedFeature);
            //         // 清除之前选择的实体
            //         this.viewer.selectedEntity = null;
            //         // 视角跟随
            //         this.viewer.trackedEntity = this.running_man;
            //     }
            // }, Cesium.ScreenSpaceEventType.RIGHT_CLICK);
        },

        /**
         * 加入石牌村和侨源阁两个点
         */
        addPoints(){
            const point1 = this.viewer.entities.add({
                name: '石牌村',
                position: Cesium.Cartesian3.fromDegrees(113.337679, 23.13389766, 0),
                point: {
                    pixelSize: 20,
                    color: Cesium.Color.BLUE,
                    outlineColor: Cesium.Color.WHITE,
                    outlineWidth: 2,
                },
                label: {
                    text: '石牌村',
                    font: '24px sans-serif',
                    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
                    fillColor: Cesium.Color.BLUE,
                    outlineColor: Cesium.Color.WHITE,
                    outlineWidth: 2,
                    verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
                    pixelOffset: new Cesium.Cartesian2(0, -20),
                },
            });
            const point2 = this.viewer.entities.add({
                name: '侨源阁',
                position: Cesium.Cartesian3.fromDegrees(113.3268036, 23.15720613, 0),
                point: {
                    pixelSize: 20,
                    color: Cesium.Color.RED,
                    outlineColor: Cesium.Color.WHITE,
                    outlineWidth: 2,
                },
                label: {
                    text: '侨源阁',
                    font: '24px sans-serif',
                    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
                    fillColor: Cesium.Color.RED,
                    outlineColor: Cesium.Color.WHITE,
                    outlineWidth: 2,
                    verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
                    pixelOffset: new Cesium.Cartesian2(0, -20),
                },
            });
            console.log('点加载完成',point1,point2);
            console.log(point1.position.getValue());
            //计算point1与point2的中点
            // const midPoint = Cesium.Cartesian3.midpoint(point1.position.getValue(), point2.position.getValue(), new Cesium.Cartesian3());
            
            // this.viewer.camera.flyTo({
            //     destination: midPoint,
            //     orientation: {
            //         heading: Cesium.Math.toRadians(0),
            //         pitch: Cesium.Math.toRadians(-90),
            //         roll: 0.0,
            //     },
            //     duration: 3,
            // });
            this.viewer.camera.position = new Cesium.Cartesian3(-2325406.1209549042, 5390488.236070972, 2489910.381484629);
            this.viewer.camera.direction = new Cesium.Cartesian3(0.3262126487280823, -0.8362938915421779, 0.4406788340492982);
            this.viewer.camera.up = new Cesium.Cartesian3(-0.19988969391568479, 0.39461307974219284, 0.8968414729274488);
        },
    }
}

</script>

<style>
.cesium-container {
  width: 600px;
  height: 300px;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>