<template>
  <div>
    <div>
      <!-- Main info -->
      <h1 class="large-co2-label z-index-1">Current atmospheric CO2 level</h1>
      <h2 class="large-co2-value z-index-1" v-if="currentCO2 >= 0">{{ currentCO2 }}</h2>
      <div class="increases z-index-1">
        <div class="increase">
          <div class="increase-indicator">
            <div class="increase-number">{{ Math.abs(increaseSinceLastYear) }} PPM</div>
            <indicator :up="increaseSinceLastYear > 0"></indicator>
          </div>
          since last year
        </div>
        <div class="increase">
          <div class="increase-indicator">
            <div class="increase-number">{{ Math.abs(increaseSinceLastMonth) }} PPM</div>
            <indicator :up="increaseSinceLastMonth > 0"></indicator>
          </div>
          since last month
        </div>
        <div class="increase">
          <div class="increase-indicator">
            <div class="increase-number">{{ Math.abs(increaseSinceLastWeek) }} PPM</div>
            <indicator :up="increaseSinceLastWeek > 0"></indicator>
          </div>
          since last week
        </div>
        <div class="increase">
          <div class="increase-indicator">
            <div class="increase-number">{{ Math.abs(increaseSinceYesterday) }} PPM</div>
            <indicator :up="increaseSinceYesterday > 0"></indicator>
          </div>
          since yesterday
        </div>
      </div>
      <svg id="graph"></svg>
    </div>
    <div class="bottom-right-corner">
      <div style="display: flex; flex-direction: column; padding-top: 1em; padding-bottom: 1em; margin-right: 2em">
        <p class="adaptive-font-color" style="font-size: 2em; font-color: black; margin: 0; margin-bottom: 0.5em; font-weight: bold">Carbon Emissions Matter</p>
        <div>
          <a href="https://github.com/Booligoosh/carbon-tab" class="attribution-font adaptive-font-color">The CarbonTab Project,</a>
          <a href="https://github.com/yyjlincoln/carbon-tab" class="attribution-font adaptive-font-color">modified</a>.
        </div>
        <a href="https://boxicons.com" class="adaptive-font-color attribution-font">"Boxicons" is used in this project</a>
      </div>
      <div>
        <img src="/public/img/logo.png" class="bottom-logo" />
      </div>
    </div>
  </div>
</template>

<script>
import store from '../store';
import axios from 'axios';
import Graph from './graph.js';
import indicator from './indicator.vue';

const increaseDecimalPlaces = 2;

var graph;

export default {
  components: { indicator },
  data() {
    return {};
  },
  store,
  created() {
    // Only check for updates every hour (to reduce cloud function strain)
    if (Date.now() - this.$store.getters.lastUpdateEpoch > 3600000) {
      // 1000*60*60 aka 1 hour
      this.updateCO2();
    }
  },
  mounted() {
    graph = new Graph();
    graph.setData(this.$store.getters.CO2Levels);
  },
  methods: {
    updateCO2() {
      axios.get(`https://functions.carbon-tab.ethan.link/.netlify/functions/getCO2`).then(response => {
        store.dispatch(`setCO2Levels`, response.data);
        store.dispatch(`setLastUpdateEpoch`, Date.now());
        graph.setData(reponse.data);
      });
    },
    padWithZeroes(number, desiredLength) {
      number = number.toString();
      while (number.length < desiredLength) {
        number = `0` + number;
      }
      return number;
    },
  },
  computed: {
    currentCO2() {
      return this.$store.getters.currentCO2;
    },
    CO2Levels() {
      return this.$store.getters.CO2Levels;
    },
    yesterdayDate() {
      return Object.keys(this.CO2Levels).reverse()[1];
    },
    lastWeekDate() {
      let date = new Date();
      date.setDate(date.getDate() - 7);
      return `${date.getFullYear()}-${this.padWithZeroes(date.getMonth() + 1, 2)}-${this.padWithZeroes(date.getDate(), 2)}`;
    },
    lastMonthDate() {
      let date = new Date();
      date.setMonth(date.getMonth() - 1);
      return `${date.getFullYear()}-${this.padWithZeroes(date.getMonth() + 1, 2)}-${this.padWithZeroes(date.getDate(), 2)}`;
    },
    lastYearDate() {
      let date = new Date();
      date.setFullYear(date.getFullYear() - 1);
      return `${date.getFullYear()}-${this.padWithZeroes(date.getMonth() + 1, 2)}-${this.padWithZeroes(date.getDate(), 2)}`;
    },
    increaseSinceLastYear() {
      return (this.currentCO2 - this.CO2Levels[this.lastYearDate].cycle).toFixed(increaseDecimalPlaces);
    },
    increaseSinceLastMonth() {
      return (this.currentCO2 - this.CO2Levels[this.lastMonthDate].cycle).toFixed(increaseDecimalPlaces);
    },
    increaseSinceLastWeek() {
      return (this.currentCO2 - this.CO2Levels[this.lastWeekDate].cycle).toFixed(increaseDecimalPlaces);
    },
    increaseSinceYesterday() {
      return (this.currentCO2 - this.CO2Levels[this.yesterdayDate].cycle).toFixed(increaseDecimalPlaces);
    },
  },
};
</script>

<style>
* {
  box-sizing: border-box;
  cursor: default;
}
.bottom-right-corner {
  position: absolute;
  bottom: 0;
  right: 0;
  padding: 5em;
  display: flex;
  flex-direction: row;
  justify-content: right;
  text-align: right;
  vertical-align: bottom;
}

body {
  margin: 0;
  padding: 5em;
  background: white;
  color: hsl(204, 6%, 16%);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif;
  min-height: 100vh;
}
.adaptive-font-color {
  color: hsl(204, 6%, 16%);
}
@media screen and (prefers-color-scheme: dark) {
  body {
    background: hsl(204, 6%, 16%);
    color: white;
  }
  .adaptive-font-color {
    color: white;
  }
}
.bottom-logo {
  border-radius: 2em;
  width: 12em;
  height: 12em;
  pointer-events: none;
  text-align: right;
}
.attribution-font {
  text-decoration: none;
  font-weight: bold;
  opacity: 0.5;
  text-align: right;
  margin: 0;
  font-size: 1.5em;
}
svg {
  position: absolute;
  top: 0;
  left: 0;
}
.legend {
  transition: transform 100ms;
}
.legend text {
  fill: #fff;
  font-family: sans-serif;
  font-weight: bold;
  alignment-baseline: middle;
}
.legend__title {
  fill: rgba(0, 0, 0, 0.6);
  width: 208px;
  height: 33px;
}
.legend__body {
  fill: rgba(0, 0, 0, 0.4);
  width: 208px;
  height: 66px;
  y: 33px;
}
.legend__color {
  cx: 16px;
  r: 6px;
}
.date-label {
  alignment-baseline: hanging;
}
</style>

<style scoped>
.large-co2-label,
.large-co2-value,
.increases {
  width: max-content;
}
.large-co2-label {
  margin: 0;
}
.large-co2-value {
  font-size: 10em;
  margin: 0;
}
.large-co2-value:after {
  content: 'PPM';
  font-size: 0.3em;
  opacity: 0.5;
  font-weight: 500;
  margin-left: 0.5em;
}
.increase-indicator {
  display: flex;
  flex-direction: row;
}
.logo {
  width: 3rem;
  position: fixed;
  bottom: 3rem;
  right: 3rem;
  opacity: 0.5;
}
.logo:hover {
  opacity: 0.75;
}
.monthly-chart {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.increases {
  display: flex;
  margin-top: 0.5em;
  opacity: 0.8;
}
.increase {
  text-align: center;
  font-size: 16px;
  width: max-content;
  margin-right: 30px;
}
.increase-number {
  font-size: 1.5em;
  font-weight: bold;
}
.z-index-1 {
  z-index: 1;
  position: relative;
}
</style>
