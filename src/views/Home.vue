<!--
 * @Author: heht
 * @Date: 2022-03-31 21:21:25
 * @LastEditors: heht
 * @LastEditTime: 2022-07-15 17:08:37
 * @Description:图片预加载
-->
<template>
  <div id="map" class="mapCon"></div>
</template>

<script>
import "ol/ol.css";
import Map from "ol/Map";
import TileLayer from "ol/layer/Tile";
import View from "ol/View";
import { TileArcGISRest } from "ol/source";
import TileGrid from "ol/tilegrid/TileGrid";
import { ImageArcGISRest } from "ol/source";
import { Image as ImageLayer } from "ol/layer";
import TileImage from "ol/source/TileImage";
import * as size from "ol/size";
import qs from "qs";

import proj4 from "proj4";
import { register } from "ol/proj/proj4";

export default {
  name: "Home",
  components: {},
  data() {
    return {
      jpgUrl: require("@/assets/pic-jpg.jpg"),
      pngUrl: require("@/assets/pic-png.png"),
      svgUrl: require("@/assets/pic-svg.svg"),
      webpUrl: require("@/assets/pic-webp.webp"),
      resolutions: [
        0.703125, 0.3515625, 0.17578125, 0.087890625, 0.0439453125,
        0.02197265625, 0.010986328125, 0.0054931640625, 0.00274658203125,
        0.001373291015625, 0.0006866455078125, 0.00034332275390625,
        0.000171661376953125, 0.0000858306884765625, 0.00004291534423828125,
        0.000021457672119140625, 0.000010728836059570312,
        0.000005364418029785156, 0.000002682209014892578,
        0.000001341104507446289,
      ],
    };
  },

  mounted() {
    this.mapInit();
    // this.preLoadImg();
  },

  methods: {
    mapInit() {
      proj4.defs("EPSG:4490", "+proj=longlat +ellps=GRS80 +no_defs");
      register(proj4);
      const map = new Map({
        target: "map",
        view: new View({
          center: [112.39, 30],
          zoom: 6,
          // minZoom: 6,
          // maxZoom: 6,
          projection: "EPSG:4490",
          resolutions: this.resolutions,
        }),
      });
      const tileLayer = this.addArcGISTileLayer({
        url: "http://58.246.138.178:8943/arcgis/rest/services/YZT/YX2016/MapServer",
      });

      const dynamicLayer = this.addArcGISImageLayer({
        url: "http://58.246.138.178:8943/arcgis/rest/services/YZT/YZTYLSJ/MapServer",
        layerIDs: [31],
      });
      map.addLayer(tileLayer);
      map.addLayer(dynamicLayer);
    },

    addArcGISTileLayer({ url }) {
      return new TileLayer({
        source: new TileArcGISRest({
          url: url,
        }),
      });
    },

    addArcGISImageLayer({ url, layerIDs }) {
      return new ImageLayer({
        source: new ImageArcGISRest({
          ratio: 1,
          params: {
            LAYERS: `show:${layerIDs.join(",")}`,
          },
          url: url,
        }),
      });
    },

    createArcGISDynamicSource(layerInfo, eyemap) {
      const tileGrid = new TileGrid({
        resolutions: eyemap.getView().getResolutions(),
        origin: [0, 0],
      });

      const source = new TileImage({
        tileGrid,
        projection: eyemap.getView().getProjection(),
        crossOrigin: "anonymous",
      });

      source.setTileUrlFunction(function (tileCoord, pixelRatio) {
        if (tileGrid.getResolutions().length <= tileCoord[0]) {
          return undefined;
        }
        const tileExtent = tileGrid.getTileCoordExtent(tileCoord);

        let tileSize = size.toSize(
          tileGrid.getTileSize(tileCoord[0]),
          this.tmpSize
        );
        if (pixelRatio !== 1) {
          tileSize = size.scale(tileSize, pixelRatio, this.tmpSize);
        }
        const params = {
          F: "image",
          FORMAT: "PNG32",
          TRANSPARENT: true,
          SIZE: tileSize.join(","),
          BBOX: tileExtent.join(","),
          BBOXSR: 4490,
          IMAGESR: 4490,
          DPI: 96,
        };
        return `${layerInfo.url}/export?${qs.stringify(params)}`;
      });

      // source.updateParams = function (params = {}) {
      //   this.params_ = {
      //     ...(this.params_ || {}),
      //     ...params,
      //   };
      // };
      return source;
    },

    preLoadImg() {
      const imgArr = new Array();
      for (let i = 0; i < 100; i++) {
        imgArr[i] = new Image(800, 400);
        imgArr[i].src = this.jpgUrl;
        const imgCon = document.getElementById("img-container");
        imgCon.appendChild(imgArr[i]);
      }
    },
  },
};
</script>

<style scoped>
.mapCon {
  width: 100%;
  height: 100%;
}
.img {
  width: 100%;
  height: 600px;
}
</style>
