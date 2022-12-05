<template>
  <div class="index-container" >
    <div id="map" ref="map"  class="container-middle">

    </div>
  </div>
</template>

<script>
import "ol/ol.css";
import {
  Tile as TileLayer } from "ol/layer";
import XYZ from "ol/source/XYZ";
import {
  defaults as defaultControls } from "ol/control";
import Map from "ol/Map.js";
import View from "ol/View.js";
import Overlay from "ol/Overlay";
import VectorLayer from "ol/layer/Vector";
import { Style, Stroke ,Circle ,Fill } from "ol/style";
import VectorSource from "ol/source/Vector";
import GeoJSON from "ol/format/GeoJSON";
import VectorTileLayer from 'ol/layer/VectorTile';
import VectorTileSource from 'ol/source/VectorTile';
import MVT from 'ol/format/MVT';
import { getArea } from 'ol/sphere';
export default {
  name: 'mapCenter',
  data(){
    return {
      properties:{},
      config:{},
      map: {},
      view: null,
      layers: null,
      mapZoom: 7.9,
      boundarySource: null,
      boundaryLayer: null,
      layerList:[],
      currentLayer:{},
      currentLayerFileList:[],
      baseUrl: process.env.VUE_APP_BASE_API,
      currentShowLayer:[],
    }
  },
  mounted() {
    this.initMap();
    this.getLayers();
  },
  methods: {
    showLayer(vector){
      let style  =  vector.getStyle();
      vector.setStyle(null);
      vector.getSource().once('change', function(evt){
        const source = evt.target;
        console.log(source.getState())
        console.log(source.getState())
        if (source.getState() === 'ready') {
          const numFeatures = source.getFeatures().length;
          console.log("Count after change: " + numFeatures);
          console.log(source.getFeatures()[0])
          source.getFeatures().forEach(it=>{
            const geom = it.getGeometry()
            const area = getArea(geom,{
              projection:'EPSG:4326'
            });
            let  output = 0;
            output = Math.round(area * 100) / 100;
            if(output > 500){
              it.setStyle(style);
              console.log(output)
            }
          })
        }
      });

    }

    ,
    initMap() {
      const map = new Map({
        target: "map",
        view: new View({
          center: [  116.405285, 39.904989], //中心点经纬度
          zoom: 2.32, //图层缩放大小
          projection: "EPSG:4326",
        }),
        controls: defaultControls({
          zoom: true,
          attribution: false,
          rotate: false,
        }),
      });
      this.map = map;
      let that = this;
      // 添加地图
      let url = "http://t{0-7}.tianditu.com/DataServer?x={x}&y={y}&l={z}";
      url = `${
          url}&T=vec_c&tk=e7ef01b7042f229cfd38e6b6198c0fbf`;
      const source = new XYZ({
        url: url,
        projection: "EPSG:4326",
      });
      const tdtLayer = new TileLayer({
        source: source,
      });
      this.map.addLayer(tdtLayer);
      // 添加注记
      url = "http://t{0-7}.tianditu.com/DataServer?x={x}&y={y}&l={z}";
      url = `${
          url}&T=cva_c&tk=e7ef01b7042f229cfd38e6b6198c0fbf`;
      const sourceCVA = new XYZ({
        url: url,
        projection: "EPSG:4326",
      });
      const tdtcvaLayer = new TileLayer({
        source: sourceCVA,
      });
      this.map.addLayer(tdtcvaLayer);


      let layert =  new VectorTileLayer({
        declutter: true,
        source: new VectorTileSource({
          format: new MVT(),
          projection: "EPSG:4326",
          url:
              'http://localhost:8081/tile/' +
              '{z}/{x}/{y}'
        }),
      });
      this.map.addLayer(layert);

      let  overlayForm = new Overlay({  // 创建一个图层
        element: this.$refs.dialogRef.$el,   // 图层绑定我们上边的弹窗
        autoPan: true,
        autoPanAnimation: {
          duration: 250,
        }
      })
      overlayForm.setPosition(undefined)   // 设置弹窗位置，刚开始我们不让他显示，就是undefined就行
      this.map.addOverlay(overlayForm)  // 然后把弹窗的图层加载到地图上

      this.onZoom();
    },
    getLayers(){
      listLayer().then(res=>{
        this.layerList = res.data;
      })
    },
    onZoom(){
      this.map.getView().on('change:resolution', () => {
        console.log(this.map.getView().getZoom())
      })
    }
  },
}
</script>

<style lang="scss"  scoped>
.index-container{
  position: relative;
  height: 100%;
  width: 100%;
}
.container-middle{
  height: 600px;
  width: 100%;
}
::v-deep .ol-viewport{
  border: 4px solid #6699CCFF;

  border-radius: 10px;
}
</style>
