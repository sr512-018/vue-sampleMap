<template>
  <div>
    <div style="position: relative; z-index: 1000;">
      <Slide>
        <label for="checkbox">
        <input id="checkbox" v-model="show" type="checkbox" />福岡行政区域</label>
        <label for="checkbox-saga">
        <input id="checkbox-saga" v-model="showSaga" type="checkbox" />佐賀行政区域</label>
      </Slide>
    </div>
    <l-map id="map" ref="map" :zoom="12" :center="[33.590188, 130.420685]" >
      <!-- 地図タイル -->
      <l-tile-layer v-for="tileProvider in tileProviders" :key="tileProvider.name"
        :name="tileProvider.name" :visible="tileProvider.visible" :url="tileProvider.url"
        layer-type="base" />
      <l-control-layers position="topright" />
      <!-- Json -->
      <l-geo-json v-if="show" :geojson="geojson" :options="options" :options-style="styleFunction" />
      <l-geo-json v-if="showSaga" :geojson="sagaGeojson" :options="options" :options-style="styleFunction" />
      <!-- マーカー -->
      <l-marker :lat-lng="[33.590188, 130.420685]">
        <l-popup>
          博多駅<br />
          <a href="https://ekitan.com/timetable/railway/station/7930"
            target="_blank">電車時刻表</a>
        </l-popup>
      </l-marker>
      <l-marker :lat-lng="[33.559082, 130.426627]">
        <l-popup>
          大橋駅<br />
          <a href="https://ekitan.com/timetable/railway/line-station/706-4/d2"
            target="_blank">電車時刻表</a>
        </l-popup>
      </l-marker>
      <l-control-scale position="bottomright" :imperial="false" :metric="true" />
    </l-map>
    <!-- 拡大縮小ボタン -->
    <div style="position: absolute; top: 200px; right: 10px; z-index: 1001;">
      <button @click="zoomIn" style="width: 40px; height: 40px; font-size: 30px;">+</button><br />
      <button @click="zoomOut" style="width: 40px; height: 40px; font-size: 30px;">-</button>
    </div>
    <!-- 左右回転ボタン -->
    <div style="position: absolute; top: 290px; right: 10px; z-index: 1001;">
      <button @click="rotateMap('left')" style="width: 40px; height: 40px; font-size: 30px;">↺</button><br />
      <button @click="rotateMap('right')" style="width: 40px; height: 40px; font-size: 30px;">↻</button>
    </div>
  </div>
</template>

<script>
import { Slide } from 'vue-burger-menu'
import { LMap, LTileLayer, LGeoJson, LControlLayers, LMarker, LPopup, LControlScale } from "vue2-leaflet";

export default {
  components: { LMap, LTileLayer, LGeoJson, LControlLayers, LMarker, LPopup, LControlScale, Slide },
  data() {
    return {
      geojson: null,
      show: false,
      showSaga: false,
      sagaGeojson: null,
      fillColor: "#b0c4de",
      clickedFeature: null,

      // 複数の地図タイルを参照、切替が可能
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
        let popupContent = '';
    if (feature.properties.N03_003 !== null) {
      popupContent += "<div>" + feature.properties.N03_003 + "</div>";
    }
    popupContent += "<div>" + feature.properties.N03_004 + "</div>";
    layer.bindPopup(popupContent);
        // レイヤーに対してのマウスの動作処理
        layer.on({
          click: this.handleClick,
          mouseover: this.highlightFeature,
          mouseout: this.resetHighlight,
        });
      };
    },
  },
  methods: {
    highlightFeature(event) {
      const layer = event.target;
      if (!this.clickedFeature || layer !== this.clickedFeature) {
      layer.setStyle({
        weight: 2,
        color: "#0000ff",
        fillColor: "#6495ed",
        fillOpacity: 0.5,
      });
      }
    },
    resetHighlight(event) {
      const layer = event.target;
      if (!this.clickedFeature || layer !== this.clickedFeature) {
      layer.setStyle({
        weight: 0.5,
        color: "#0000ff",
        opacity: 1,
        fillColor: this.fillColor,
        fillOpacity: 0.5,
      });
      }
    },
    handleClick(event) {
      const layer = event.target;
      // 他のレイヤーをクリックした時に色が移る
      if (this.clickedFeature && this.clickedFeature !== layer) {
        this.clickedFeature.setStyle({
          weight: 0.5,
          color: "#0000ff",
          opacity: 1,
          fillColor: this.fillColor,
          fillOpacity: 0.5,
        });
      }
      this.clickedFeature = layer; // クリックされた図形を保存
    },
    zoomIn() {
      this.$refs.map.mapObject.zoomIn();
    },
    zoomOut() {
      this.$refs.map.mapObject.zoomOut();
    },
    rotateMap(direction) {
      const map = this.$refs.map.mapObject;
      const currentRotation = map.getRotation(); // 現在の回転角度を取得

      // 回転方向に応じて角度を調整
      const newRotation = direction === 'left' ? currentRotation - 45 : currentRotation + 45;

      // 地図を新しい角度に回転させる
      map.setRotation(newRotation);
    }
  },
  async created() {
    // 福岡行政区域の取得
    const response = await fetch(
      "https://smartnews-smri.github.io/japan-topography/data/municipality/geojson/s0010/N03-21_40_210101.json"
    );
    const data = await response.json();
    this.geojson = data;
    // 佐賀行政区域の取得
    const res = await fetch(
        "https://smartnews-smri.github.io/japan-topography/data/municipality/geojson/s0010/N03-21_41_210101.json"
      );
      const data2 = await res.json();
      this.sagaGeojson = data2;
  },
};
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

#map {
  height: 100vh;
  width: 100vw;
}

.leaflet-control-zoom {
  display: none !important;
}

</style>