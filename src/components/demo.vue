<template>
    <div class="index-container">
        <div id="map" ref="map" class="container-middle">

        </div>
    </div>
</template>

<script>
    import "ol/ol.css";
    import {
        Tile as TileLayer
    } from "ol/layer";
    import XYZ from "ol/source/XYZ";
    import {
        defaults as defaultControls
    } from "ol/control";
    import Map from "ol/Map.js";
    import View from "ol/View.js";
    import Overlay from "ol/Overlay";
    import Projection from 'ol/proj/Projection'
    import VectorTileLayer from 'ol/layer/VectorTile';
    import VectorTileSource from 'ol/source/VectorTile';
    import MVT from 'ol/format/MVT';
    import {getArea} from 'ol/sphere';

    export default {
        name: 'mapCenter',
        data() {
            return {
                properties: {},
                config: {},
                map: {},
                view: null,
                layers: null,
                mapZoom: 7.9,
                boundarySource: null,
                boundaryLayer: null,
                layerList: [],
                currentLayer: {},
                currentLayerFileList: [],
                baseUrl: process.env.VUE_APP_BASE_API,
                currentShowLayer: [],
            }
        },
        mounted() {
            this.initMap();
        },
        methods: {
            initMap() {

                const map = new Map({
                    target: "map",
                    view: new View({
                        center: [116.405285, 39.904989], //中心点经纬度
                        zoom: 5, //图层缩放大小
                        projection: 'EPSG:3857',
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
                    projection: 'EPSG:4326',
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
                    projection: 'EPSG:4326',
                });
                const tdtcvaLayer = new TileLayer({
                    source: sourceCVA,
                });
                this.map.addLayer(tdtcvaLayer);

                let layert =  new VectorTileLayer({
                    declutter: true,
                    source: new VectorTileSource({
                        format: new MVT(),
                        url:
                            'http://localhost:8080/tile/' +
                            '{z}/{x}/{y}'
                    }),
                });
                this.map.addLayer(layert);

            }
        },
    }
</script>

<style lang="scss" scoped>
    .index-container {
        position: relative;
        height: 100%;
        width: 100%;
    }

    .container-middle {
        height: 800px;
        width: 100%;
    }

    ::v-deep .ol-viewport {
        border: 4px solid #6699CCFF;

        border-radius: 10px;
    }
</style>
