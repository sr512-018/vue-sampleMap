<template>
  <div>
    <div>
      <label for="checkbox">福岡行政区域</label>
      <input id="checkbox" v-model="show" type="checkbox" />
    </div>
    <l-map id="map"
      :zoom="zoom"
      :center="[33.590188, 130.420685]"
      style="height: 600px; width: 100%"
    >
      <l-tile-layer
        v-for="tileProvider in tileProviders"
        :key="tileProvider.name"
        :name="tileProvider.name"
        :visible="tileProvider.visible"
        :url="tileProvider.url"
        layer-type="base"
      />
      <l-control-layers position="topleft" />
      <l-geo-json
        v-if="show"
        :geojson="geojson"
        :options="options"
        :options-style="styleFunction"
      />
      <l-marker :lat-lng="[33.590188, 130.420685]">
        <l-popup>
          博多駅<br />
          <a href="https://ekitan.com/timetable/railway/station/7930" target="_blank">時刻表</a>
          </l-popup>
      </l-marker>
      <l-marker :lat-lng="[33.559082, 130.426627]">
        <l-popup>
          大橋駅<br />
          <a href="https://ekitan.com/timetable/railway/line-station/706-4/d2" target="_blank">時刻表</a>
          </l-popup>
      </l-marker>
      <l-control-scale position="bottomleft" :imperial="false" :metric="true"></l-control-scale>
    </l-map>
  </div>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LGeoJson,
  LControlLayers,
  LMarker,
  LPopup,
  LControlScale
} from "vue2-leaflet";

export default {
  components: {
    LMap,
    LTileLayer,
    LGeoJson,
    LControlLayers,
    LMarker,
    LPopup,
    LControlScale
  },
  data() {
    return {
      geojson: null,
      show: false,
      fillColor: "#b0c4de",
      zoom: 14,
      searchQuery: '',

      // 複数の地図タイルを設定、切替が可能
      tileProviders: [
        {
          name: "地理院タイル(淡色)",
          visible: true,
          url: "https://cyberjapandata.gsi.go.jp/xyz/pale/{z}/{x}/{y}.png",
        },
        {
          name: "地理院タイル(航空写真)",
          visible: false,
          url: "https://cyberjapandata.gsi.go.jp/xyz/seamlessphoto/{z}/{x}/{y}.jpg",
        },
      ],
    };
  },
  computed: {
    options() {
      return {
        onEachFeature: this.onEachFeatureFunction,
      };
    },
    // 図形のスタイル設定
    styleFunction() {
      const fillColor = this.fillColor;
      return () => {
        return {
          weight: 0.5,
          color: "#0000ff",
          opacity: 1,
          fillColor: fillColor,
          fillOpacity: 0.5,
        };
      };
    },
    // 読み込んだGeoJsonファイルの処理
    onEachFeatureFunction() {
      return (feature, layer) => {
        layer.bindPopup(
          // "<div>N03_003:" +
          //   feature.properties.N03_003 +
          "</div><div>" + feature.properties.N03_004 + "</div>",
        );
        // レイヤーに対してのマウスの動作処理
        layer.on({
          mouseover: this.highlightFeature,
          mouseout: this.resetHighlight,
        });
      };
    },
  },
  methods: {
    highlightFeature(event) {
      const layer = event.target;
      layer.setStyle({
        weight: 2,
        color: '#0000ff',
        fillColor: '#6495ed',
        fillOpacity: 0.5
      });
    },
    resetHighlight(event) {
      const layer = event.target;
      layer.setStyle({
        weight: 0.5,
        color: "#0000ff",
        opacity: 1,
        fillColor: this.fillColor,
        fillOpacity: 0.5,
      });
    },
},
  // git掲載の福岡行政区域の取得
  async created() {
    const response = await fetch(
      "https://smartnews-smri.github.io/japan-topography/data/municipality/geojson/s0010/N03-21_40_210101.json"
    );
    const data = await response.json();
    this.geojson = data;
    this.loading = false;
  },
};
</script>
