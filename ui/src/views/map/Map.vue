<template>
  <div class="map-container">
    <div class="map-feature"><pre>{{ selectedFeature }}</pre></div>
    <div class="map" id="map"></div>
  </div>
</template>

<script>
import VectorSource from "ol/source/Vector";
import data from "../../data/features.json";
import Select from "ol/interaction/Select";
import VectorLayer from "ol/layer/Vector";
import TileLayer from "ol/layer/Tile.js";
import GeoJSON from "ol/format/GeoJSON";
import Link from "ol/interaction/Link";
import OSM from "ol/source/OSM.js";
import View from "ol/View";
import Map from "ol/Map";
import Vue from "vue";

export default {
  data() {
    return {
      airportsGeoJSON: data,
      selectedFeature: null,
    };
  },
  mounted() {
    this.initMap();
  },
  computed: {
    layers() {
      return [this.createTileLayer(), this.createVectorLayer()];
    },
  },
  methods: {
    initMap() {
      const { layers, createView } = this;
      const map = new Map({
        layers,
        view: createView(),
        target: "map",
        controls: [],
      });
      const selectInteraction = new Select();
      selectInteraction.getFeatures().on("add", (e) => {
        this.selectedFeature = e.target.array_[0].values_;
      });
      map.addInteraction(new Link());
      map.addInteraction(selectInteraction);
      Vue.prototype.$map = map;
      // console.log(this.$map.target);
    },
    createVectorFormat() {
      return new GeoJSON({
        featureProjection: "EPSG:3857",
        dataProjection: "EPSG:4326",
      });
    },
    createVectorSource() {
      const format = this.createVectorFormat();
      return new VectorSource({
        features: format.readFeatures(this.airportsGeoJSON),
      });
    },
    createVectorLayer() {
      const source = this.createVectorSource();
      return new VectorLayer({ source });
    },
    createTileLayer() {
      return new TileLayer({ source: new OSM() });
    },
    createView() {
      return new View({
        center: [0, 0],
        zoom: 2,
      });
    },
  },
};
</script>

<style lang="less">
.map-container {
  width: 100%;
  height: 100%;
  .map {
    width: 100%;
    height: 100%;
  }
}
</style>
