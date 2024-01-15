<template>
  <v-app>
    <v-main :style="{background: '#303952'}">
      <div id="container">
        <svg width="850" height="500">
          <rect x="20" y="450" :width="axisWidth" height="1" fill="#f8a5c2" />
          <g v-for="n in 9">
            <g :transform="`translate(${20 + (n-1) * axisWidth/8}, 440)`">
              <rect x="0" y="0" width="1" height="10" fill="#f8a5c2" />
              <text x="-5" y="-10" fill="#f8a5c2"> {{n - 1}}</text>
            </g>
          </g>

          <g v-for="(d, index) in data">
            <g :transform="`translate(${20 + d[sliderValue]/8 * axisWidth}, 200)`">
              <circle class="populationCircle" :r="populationScale(d.population)" fill="#c44569"  opacity="0.75" @mousedown="selectCircle(d)"  @mouseover="highlightCircle(d)" @mouseleave="clearHighlight"/>
              <g v-if="highlightedDataPoint == d && selectedDataPoint != d">
                <circle class="selection-circle" r="5" fill="#cf6a87" />
                <text class="selection-label" x="10" y="-170" fill="#cf6a87"> {{d.Country.toUpperCase() + ' ' + d[sliderValue]}} </text>
                <rect class="selection-line" fill="#cf6a87" height="180" x="0" y="-180" width="2" />
                <rect class="selection-line" fill="#cf6a87" height="6" x="-2" y="-180" width="6" />
              </g>
              <g v-if="selectedDataPoint == d">
                <circle class="selection-circle" r="5" fill="#cf6a87" />
                <text class="selection-label" x="10" y="-188" fill="#cf6a87"> {{d.Country.toUpperCase() + ' ' + d[sliderValue]}} </text>
                <rect class="selection-line" fill="#cf6a87" height="200" x="0" y="-200" width="2" />
                <rect class="selection-line" fill="#cf6a87" height="6" x="-2" y="-200" width="6" />
              </g>
            </g>
          </g>
        </svg>
        <div id="slider">
          <v-slider :style="{color: '#f8a5c2'}"
                    label="JAARTAL"
                    thumb-label="always"
                    :thumb-color = "'#c44569'"
                    min="1960"
                    max="2020"
                    step="1"
                    color="#c44569"
                    label-color="c44569"
                    v-model="sliderValue"
          ></v-slider>
        </div>

      </div>

    </v-main>
  </v-app>
</template>

<script setup>
import {onMounted, ref} from "vue";
import * as d3 from "d3";

const axisWidth = 710;
let data = ref([]);
let populationScale = null;
let sliderValue = ref(1989);
let highlightedDataPoint = ref(null);
let selectedDataPoint = ref(null);

function highlightCircle(d) {
  highlightedDataPoint.value = d;
}
function selectCircle(d) {
  selectedDataPoint.value = d;
}
function clearHighlight() {
  highlightedDataPoint.value = null;
}

onMounted(async () => {
  let fertData = await d3.csv('fertility_rate.csv');
  let popData = await d3.csv('population.csv');

  fertData = fertData.filter(r => countries.includes(r.Country));
  popData = popData.filter(r => countries.includes(r['Country Name']));

  for (let i = 0; i < fertData.length; i++) {
    const pop = popData.find(r => r['Country Name'] == fertData[i].Country);
    fertData[i].population = +pop['2021'];
    for (let j = 1960; j <= 2020; j++) {
      fertData[i]['' + j] = +fertData[i]['' + j];
    }
  }

  const populationExtents = d3.extent(fertData, d => d.population);
  populationScale = d3.scaleSqrt()
    .domain([0, populationExtents[1]])
    .range([0, 150]);

  fertData = fertData.sort((a,b) => {return b.population - a.population});
  data.value = fertData;
});

const countries = [
  "Belgium",
  "China",
  "Turkiye",
  "Germany",
  "Finland",
  "Niger"
];
</script>

<style>
#container {
  width: 750px;
}

svg {
  margin: 50px;
}

#slider {
  margin: 50px;
  box-sizing: content-box;
  width: 750px;
}

circle {
  fill: orange;
  opacity: 0.5;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
  font-family: 'Open Sans', sans-serif;
  background: #303952;
}

text {

  font-family: 'Open Sans', sans-serif;
}

svg {
  margin: 50px 50px 0 50px;
}

circle:hover {
  fill: #ea8685;
  opacity: 1;
}

#container {
  width: 750px;
}

#slider {
  margin: 50px;
  box-sizing: content-box;
  width: 750px;
}

.selection-label {

  font-weight: 800;
}

.populationCircle {
  transition: all 200ms ease;

}

.selection-circle {
  animation-name: scaleIn;
  animation-duration: 200ms;
  animation-timing-function: ease-in-out;
  pointer-events: none;
}

.selection-line {
  animation-name: scaleIn;
  animation-duration: 200ms;
  animation-timing-function: ease-in-out;
  pointer-events: none;
}

@keyframes scaleIn {
  0% {
    transform: scale(0)
  }
  100% {
    transform: scale(1);
  }
}
</style>
