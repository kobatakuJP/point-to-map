<template>
  <div>
    <textarea
      :placeholder="`lat${separator}lon`"
      v-model="inputValue"
    ></textarea>
    {{ inputValue }}{{ latlons }}
    <div id="OSMCanvas"></div>
  </div>
</template>

<script>
import "ol/ol.css";
import { Map, View } from "ol";
import { transform } from "ol/proj";
import { Vector as VectorLayer, Tile as TitleLayer } from "ol/layer";
import { Vector as VectorSource } from "ol/source";
import Feature from "ol/Feature";
import {Circle as CircleStyle, Fill, Stroke, Style} from 'ol/style';
import {/*LineString,*/ Point} from 'ol/geom';
// import {getVectorContext} from 'ol/render';
import OSM from "ol/source/OSM";

const proj4 = require("proj4").default;

export default {
  data() {
    return {
      inputValue: "",
      separator: "|",
      FromEPSG: "4301",
      ToEPSG: "4326",
      MapEPSG: "3857",
      mapview: null,
      feature_style: {
        fillColor: "#98FB98",
        fillOpacity: 0.8,
        strokeColor: "#3CB371",
        strokeWidth: 2,
        pointRadius: 6,
      },
    };
  },
  mounted() {
    proj4.defs([
      [
        "EPSG:4301", //東京測地系/日本測地系 SRID=4301
        "+proj=longlat +ellps=bessel +towgs84=-146.414,507.337,680.507,0,0,0,0 +no_defs",
      ],
    ]);
    this.initialize();
  },
  computed: {
    latlons() {
      const lines = this.inputValue.split("\n");
      const str = lines.map((v) =>
        v.split(this.separator).map((s) => parseFloat(s.trim()))
      );
      return str;
    },
  },
  methods: {
    /**
     * From測地系からTo測地系に変換
     * @return [lon, lat]
     */
    changeEPSG(lon, lat) {
      return proj4(`EPSG:${this.FromEPSG}`, `EPSG:${this.ToEPSG}`, [lon, lat]);
    },
    initialize() {
      this.createMap();
      this.addPoint();
    },
    createMap() {
      this.mapview = new Map({
        target: "OSMCanvas",
        layers: [
          new TitleLayer({
            source: new OSM(),
          }),
          this.addPoint()
        ],
        view: new View({
          center: transform(
            [135, 35],
            `EPSG:${this.ToEPSG}`,
            `EPSG:${this.MapEPSG}`
          ),
          zoom: 5,
        }),
      });
    },
    createPoint(lon, lat) {
      return new Feature({
        geometry: new Point([lon, lat]).transform(
          `EPSG:${this.ToEPSG}`,
          `EPSG:${this.MapEPSG}`
        ),
        size: 20,
      });
    },
    addPoint() {
      var vectorSource = new VectorSource({
        features: [this.createPoint(135, 35)],
        wrapX: false,
      });
      var vector = new VectorLayer({
        source: vectorSource,
        style: new Style({
          image: new CircleStyle({
            radius: 10,
            fill: new Fill({ color: "#666666" }),
            stroke: new Stroke({ color: "#bada55", width: 1 }),
          }),
        }),
      });
      return vector;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#OSMCanvas {
  width: 100%;
  height: 400px;
}
</style>
