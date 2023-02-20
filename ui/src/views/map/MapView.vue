<template>
  <div class="map-container">
    <div class="map-feature">
      <pre>{{ selectedFeature }}</pre>
    </div>
    <div class="map" id="map"></div>
  </div>
</template>

<script>
import data from "../../data/features.json";
import Map from "ol/Map.js";
import View from "ol/View.js";
import "ol/ol.css";
import { Fill, Stroke, Style, Text, RegularShape } from "ol/style.js";
import { Cluster, OSM, Vector as VectorSource } from "ol/source.js";
import { Tile as TileLayer, Vector as VectorLayer } from "ol/layer.js";
import { boundingExtent } from "ol/extent.js";
import GeoJSON from "ol/format/GeoJSON";
import { useGeographic } from "ol/proj.js";
import Vue from "vue";

export default {
  data() {
    return {
      airfieldsGeoJSON: data,
      selectedFeature: null,
    };
  },
  mounted() {
    useGeographic();
    this.initMap();
  },
  computed: {},
  methods: {
    cleanData() {
      const clean = this.airfieldsGeoJSON.features.filter((x) => x.geometry !== null);
      this.airfieldsGeoJSON.features = clean;
    },
    vectorSource() {
      return new VectorSource({
        features: new GeoJSON().readFeatures(this.airfieldsGeoJSON),
      });
    },
    clusterSource(source) {
      return new Cluster({
        distance: 50,
        minDistance: 0,
        source: source,
      });
    },
    rasterLayer() {
      return new TileLayer({
        source: new OSM(),
      });
    },
    initMap() {
      this.cleanData();
      const map = new Map({
        layers: [this.rasterLayer(), this.vectorLayer()],
        target: "map",
        controls: [],
        view: new View({
          center: [0, 0],
          zoom: 2,
        }),
      });
      Vue.prototype.$map = map;
      this.initMapClick();
    },
    initMapClick() {
      this.$map.on("click", (e) => {
        this.vectorLayer()
          .getFeatures(e.pixel)
          .then((clickedFeatures) => {
            if (clickedFeatures.length) {
              const features = clickedFeatures[0].get("features");
              if (features.length > 1) {
                const extent = boundingExtent(
                  features.map((r) => r.getGeometry().getCoordinates())
                );
                this.$map
                  .getView()
                  .fit(extent, { duration: 1000, padding: [50, 50, 50, 50] });
              }
            }
          });
      });
    },
    vectorLayer() {
      return new VectorLayer({
        source: this.clusterSource(this.vectorSource()),
        style: function (feature) {
          const size = feature.get("features").length;
          const stroke = new Stroke({ color: "#8F73FF", width: 2 });
          const fill = new Fill({ color: "#000000" });
          return new Style({
            image: new RegularShape({
              fill: fill,
              stroke: stroke,
              radius: 14 / Math.SQRT2,
              radius2: 11,
              points: 6,
              angle: 11,
              scale: [1, 1],
            }),
            text: new Text({
              text: size.toString(),
              textAlign: "center",
              font: "10px sans-serif",
              fill: new Fill({
                color: "#fff",
              }),
            }),
          });
        },
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
