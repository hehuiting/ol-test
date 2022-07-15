<!--
 * @Author: heht
 * @Date: 2022-03-31 21:21:25
 * @LastEditors: heht
 * @LastEditTime: 2022-06-28 15:12:40
 * @Description: 
-->
<template>
  <div id="app">
    <!-- <div id="map" class="map"></div> -->
    <router-view />
  </div>
</template>

<script>
import "ol/ol.css";
import Map from "ol/Map";
import TileLayer from "ol/layer/Tile";
import View from "ol/View";
import proj4 from "proj4";
import { TileWMS } from "ol/source";
import { get as getProjection } from "ol/proj";
import { register } from "ol/proj/proj4";
export default {
  mounted() {
    // this.mapInit();
  },

  methods: {
    mapInit() {
      // 1、注册坐标系
      proj4.defs(
        "EPSG:21781",
        "+proj=somerc +lat_0=46.95240555555556 " +
          "+lon_0=7.439583333333333 +k_0=1 +x_0=600000 +y_0=200000 +ellps=bessel " +
          "+towgs84=674.4,15.1,405.3,0,0,0,0 +units=m +no_defs"
      );

      proj4.defs(
        "EPSG:4522",
        "+proj=tmerc +lat_0=0 +lon_0=102 +k=1 +x_0=34500000 +y_0=0 +ellps=GRS80 +units=m +no_defs"
      );
      register(proj4);

      // 2、定义该坐标系视图范围
      let proj21781 = getProjection("EPSG:21781");
      proj21781.setExtent([485071.54, 75346.36, 828515.78, 299941.84]);

      const proj4522 = getProjection("EPSG:4522");
      proj4522.setExtent([
        34599537.78543863, 3313030.6622967534, 34603202.65556276,
        3316518.9696675064,
      ]);

      let layers = {};

      //  3、底图坐标4326
      layers["wms4522"] = new TileLayer({
        source: new TileWMS({
          url: "http://8.142.9.0:8080/geoserver/WebGisdl/wms",
          crossOrigin: "anonymous",
          params: {
            LAYERS: "WebGisdl:t2019_511802_dltb",
            VERSION: "1.1.0",
          },
          projection: "EPSG:4522",
        }),
      });

      // 4、叠加图层坐标系 21781
      layers["wms21781"] = new TileLayer({
        source: new TileWMS({
          crossOrigin: "anonymous",
          params: {
            LAYERS: "ch.swisstopo.pixelkarte-farbe-pk1000.noscale",
            FORMAT: "image/jpeg",
          },
          url: "https://wms.geo.admin.ch/",
          projection: "EPSG:21781",
        }),
      });

      //  5、在3857的视图空间里加载不同坐标系的图层
      new Map({
        layers: [layers["wms4522"]],
        target: "map",
        view: new View({
          projection: "EPSG:4326",
          center: [0, 0],
          zoom: 2,
        }),
      });
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
  width: 100%;
}
#map {
  height: 100%;
  width: 100%;
}
#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
