<template>
    <div id="map" class="map"></div>
  </template>
  
  <script>
  import 'ol/ol.css'; // 引入 OpenLayers 的样式
  import { Map, View } from 'ol';
  import GeoJSON from 'ol/format/GeoJSON';
  import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer';
  import { OSM, Vector as VectorSource } from 'ol/source';
  import { Style, Stroke, Fill } from 'ol/style';
  import { fromLonLat } from 'ol/proj';
  
  export default {
    name: 'OpenLayersMap',
    data() {
      return {
        map: null,
      };
    },
    mounted() {
      this.initMap();
    },
    methods: {
      initMap() {
        // 创建两个GeoJSON源
        const geojsonSource1 = new VectorSource({
          url: 'http://localhost:8080/ai4city/qiaoyuange.geojson', // 替换为第一个行政边界的GeoJSON路径
          format: new GeoJSON(),
        });
  
        const geojsonSource2 = new VectorSource({
          url: 'http://localhost:8080/ai4city/shipaiqiao.geojson', // 替换为第二个行政边界的GeoJSON路径
          format: new GeoJSON(),
        });
  
        // 定义两个图层
        const vectorLayer1 = new VectorLayer({
          source: geojsonSource1,
          style: new Style({
            stroke: new Stroke({
              color: '#ff0000', // 红色边界
              width: 2,
            }),
            fill: new Fill({
              color: 'rgba(255, 0, 0, 0.1)', // 红色填充，透明度0.1
            }),
            label: new Text({
                text: '石牌桥社区', // 在这里定义标签文本
                font: '12px Calibri,sans-serif',
                fill: new Fill({
                    color: '#000', // 标签颜色
                }),
                stroke: new Stroke({
                    color: '#fff', // 标签描边
                    width: 2,
                }),
                offsetY: -12, // 标签偏移，使其不压在线上
            }),
          }),
        });
  
        const vectorLayer2 = new VectorLayer({
          source: geojsonSource2,
          style: new Style({
            stroke: new Stroke({
              color: '#0000ff', // 蓝色边界
              width: 2,
            }),
            fill: new Fill({
                color: 'rgba(0, 0, 255, 0.1)', // 蓝色填充，透明度0.1
            }),
            label: new Text({
                text: '侨源阁社区', // 在这里定义标签文本
                font: '12px Calibri,sans-serif',
                fill: new Fill({
                    color: '#000', // 标签颜色
                }),
                stroke: new Stroke({
                    color: '#fff', // 标签描边
                    width: 2,
                }),
                offsetY: -12, // 标签偏移，使其不压在线上
            }),
          }),
        });
  
        // 创建地图
        this.map = new Map({
          target: 'map',
          layers: [
            new TileLayer({
              source: new OSM(),
            }),
            vectorLayer1,
            vectorLayer2,
          ],
          view: new View({
            center: fromLonLat([113.33, 23.13]), // 使用从EPSG:4326转换为EPSG:3857的中心点
            zoom: 12, // 设置初始缩放级别
          }),
        });
  
        // // 当GeoJSON源加载完成后，将视角聚焦在vectorLayer1上
        // geojsonSource1.once('change', () => {
        //   if (geojsonSource1.getState() === 'ready') {
        //     const extent = geojsonSource1.getExtent(); // 获取vectorLayer1的边界范围
        //     this.map.getView().fit(extent, { duration: 1000, padding: [20, 20, 20, 20] }); // 动画平移视图并聚焦到图层
        //   }
        // });
      },
      // 切换视图到指定的图层
      changeViewTo(pathName) {
        let layerIndex = 1;
        if(pathName === '石牌村实验区路线') {
          layerIndex = 2;
        } else if(pathName === '侨源阁实验区路线') {
          layerIndex = 1;
        }
        const layers = this.map.getLayers().getArray();
        const layer = layers[layerIndex];
        const extent = layer.getSource().getExtent();
        this.map.getView().fit(extent, { duration: 1000, padding: [20, 20, 20, 20] });
      },
    },
  };
  </script>
  
  <style scoped>
  .map {
    width: 600px;
    height: 300px;
    margin: 0;
    padding: 0;
    overflow: hidden;
  }
  </style>