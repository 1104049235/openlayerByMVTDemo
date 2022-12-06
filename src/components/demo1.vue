<template>
  <div class="index-container" >
    <div id="map" ref="map"  class="container-middle">

    </div>
  </div>
</template>

<script>
  import Map from 'ol/Map';
  import MapboxVector from 'ol/layer/MapboxVector';
  import View from 'ol/View';
  import Projection from "ol/proj/Projection";
  import "ol/ol.css";
  import VectorTileLayer from 'ol/layer/VectorTile';
  import VectorTileSource from 'ol/source/VectorTile';
  import MVT from 'ol/format/MVT';
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
  },
  methods: {
    initMap() {

      const map = new Map({
        target: 'map',
        layers: [
          new MapboxVector({
            styleUrl: 'mapbox://styles/mapbox/bright-v9',
            accessToken:
                    'Your Mapbox access token from https://mapbox.com/ here',
          }),
        ],
        view: new View({
          center: [0, 0],
          zoom: 2,
        }),
      });
      console.log('Projection',map.getView().getProjection())

      // const projection4326 = new Projection({
      //   code: 'EPSG:4326',
      //   units: 'degrees',
      //   axisOrientation: 'neu'
      // })
      let layert =  new VectorTileLayer({
        declutter: true,
        source: new VectorTileSource({
          format: new MVT(),
          url:
                  'http://localhost:8080/tile/' +
                  '{z}/{x}/{y}'
        }),
      });
      map.addLayer(layert);
    },
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
  height: 800px;
  width: 100%;
}
::v-deep .ol-viewport{
  border: 4px solid #6699CCFF;

  border-radius: 10px;
}
</style>
