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
import { transform } from "ol/proj";
import Map from "ol/Map";
import View from "ol/View";
import TitleLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";

const proj4 = require("proj4").default;

export default {
  data() {
    return {
      inputValue: "",
      separator: "|",
      FromEPSG: "4301",
      ToEPSG: "4326",
      mapview: null,
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
      this.mapview.on("click", (ev) => {
        var lonlat = transform(ev.coordinate, "EPSG:3857", "EPSG:4326");
        this.selectLatLong = lonlat;
        this.$parent.selectLonLat = lonlat;
        this.$emit("panretMessage");
      });
    },
    createMap() {
      this.mapview = new Map({
        target: "OSMCanvas",
        layers: [
          new TitleLayer({
            source: new OSM(),
          }),
        ],
        view: new View({
          center: [0, 0],
          zoom: 0,
        }),
      });
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
