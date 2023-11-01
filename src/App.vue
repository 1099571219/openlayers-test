<script setup>
// import { Map, View } from 'ol'      // 地图实例方法、视图方法
// import Tile from 'ol/layer/Tile'    // 瓦片渲染方法
// import OSM from 'ol/source/OSM'    // OSM瓦片【OSM不能在实际开发中使用，因为OSM里中国地图的边界有点问题！！！！】
// import 'ol/ol.css'                 // 地图样式

import 'ol/ol.css';
import Draw from 'ol/interaction/Draw';
import Map from 'ol/Map';
import Overlay from 'ol/Overlay';
import View from 'ol/View';
import { Circle as CircleStyle, Fill, Stroke, Style } from 'ol/style';
import { LineString, Polygon } from 'ol/geom';
import { OSM, Vector as VectorSource } from 'ol/source';
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer';
import { getArea, getLength } from 'ol/sphere';
import { unByKey } from 'ol/Observable';
import XYZ from 'ol/source/XYZ'
import { ref, onMounted } from 'vue'
const mapElement = ref()
onMounted(() => {
  // const raster = new TileLayer({
  //   source: new OSM(),
  // });
  initMap()
  distance()
})


function initMap() {
  // 地图实例
  map = new Map({
    target: mapElement.value,                         // 对应页面里 id 为 map 的元素
    layers: [                              // 图层
      raster
    ],
    view: new View({                       // 地图视图
      projection: "EPSG:4326",             // 坐标系，有EPSG:4326和EPSG:3857
      center: [114.064839, 22.548857],     // 深圳坐标
      minZoom: 5,                          // 地图缩放最小级别
      zoom: 15                             // 地图缩放级别（打开页面时默认级别）
    })
  })
}

let raster = new TileLayer({
  title: "高德地图",
  source: new XYZ({
    url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}',
    wrapX: false
  })
});
let map = null
let helpTooltipElement = null
let feature = null;
let helpTooltip = null;
let draw = null;
let measureTooltipElement = null;
let measureTooltip = null;
let listener = null;
let mapMouseMove = null;

function distance() {
  let source = new VectorSource()  // 首先创建一个数据源，用来放置绘制过程中和绘制结束后的线段
  const layer = new VectorLayer({  // 添加一个图层，用来放置数据源，样式自己随便设置就可以了，我这里默认的官网
    source: source,
    style: new Style({
      fill: new Fill({
        color: 'rgba(255, 255, 255, 0.2)',
      }),
      stroke: new Stroke({
        color: '#ffcc33',
        width: 4,
      }),
      image: new CircleStyle({
        radius: 7,
        fill: new Fill({
          color: '#ffcc33',
        }),
      }),
    }),
  });
  mapMouseMove = map.on('pointermove', (ev) => {  // 给地图添加一个鼠标移动事件
    let helpMsg = '点击开始测量'    // 默认开始的操作提示文本
    if (feature) {  // feature 是全局的，判断有没有点击鼠标绘制过
      helpMsg = '双击结束测量'    // 如果之前点击绘制了就显示双击结束
    }
    helpTooltipElement.innerHTML = helpMsg;   // 设置dom的提示文字
    helpTooltip.setPosition(ev.coordinate);  // 设置位置跟着鼠标走
    helpTooltipElement.classList.remove('hidden')  // 让他显示出来
  })
  createHelpTooltip()   // 创建那个helpTooltipElement方法
  map.addLayer(layer)  // 把图层添加到地图
}

function createHelpTooltip() {
  if (helpTooltipElement) {
    helpTooltipElement.parentNode.removeChild(helpTooltipElement);
  }
  helpTooltipElement = document.createElement('div');
  helpTooltipElement.className = 'ol-tooltip hidden';
  helpTooltip = new Overlay({
    element: helpTooltipElement,
    offset: [15, 0],
    positioning: 'center-left',
  });
  map.addOverlay(helpTooltip);
}




</script>

<template>
  <div class="container">
    <div id="map" class="map" ref="mapElement"></div>
    <form class="otherForm">
      <label for="type">Measurement type &nbsp;</label>
      <select id="type">
        <option value="length">Length (LineString)</option>
        <option value="area">Area (Polygon)</option>
      </select>
    </form>
  </div>
</template>

<style>
.map {
  height: 100%;
  width: 100%;

}

.container {
  height: 100vh;
  width: 100vw;
}

.otherForm {
  position: fixed;
  bottom: 1vh;
  left: 2vw;
}


.ol-tooltip {
  position: relative;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 4px;
  color: white;
  padding: 4px 8px;
  opacity: 0.7;
  white-space: nowrap;
  font-size: 12px;
  cursor: default;
  user-select: none;
}

.ol-tooltip-measure {
  opacity: 1;
  font-weight: bold;
}

.ol-tooltip-static {
  background-color: #ffcc33;
  color: black;
  border: 1px solid white;
}

.ol-tooltip-measure:before,
.ol-tooltip-static:before {
  border-top: 6px solid rgba(0, 0, 0, 0.5);
  border-right: 6px solid transparent;
  border-left: 6px solid transparent;
  content: "";
  position: absolute;
  bottom: -6px;
  margin-left: -7px;
  left: 50%;
}

.ol-tooltip-static:before {
  border-top-color: #ffcc33;
}
</style>
