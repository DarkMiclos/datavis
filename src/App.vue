<template>
  <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>

<script>
import HelloWorld from './components/Dominance.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .chart-container {
            position: relative;
            display: flex;
            align-items: center;
        }
        .crosshair {
            stroke: #888;
            stroke-width: 1px;
            stroke-dasharray: 5, 5;
        }
        .bullish {
            fill: #26A69A; /* Green for bullish candles */
            stroke: #26A69A;
        }
        .bearish {
            fill: #EF5350; /* Red for bearish candles */
            stroke: #EF5350;
        }
        .wick {
            stroke: #000;
            stroke-width: 1px;
        }
        .axis path, .axis line {
            stroke: #000;
        }
        .axis text {
            font-size: 12px;
        }
        .axis-tooltip {
            font-size: 12px;
            fill: #333;
        }
        .crosshair-marker {
            fill: #888;
            stroke: #fff;
            stroke-width: 1px;
        }
        .tooltip-background {
            fill: rgba(255, 255, 255, 0.9);
            stroke: #888;
            stroke-width: 1px;
        }
        .title {
            font-size: 18px;
            font-weight: bold;
            text-anchor: middle;
        }
        .slider-container {
            margin: 20px 0;
            text-align: center;
        }
        .slider {
            width: 600px;
            margin: 0 auto;
        }
        .change-rect {
            opacity: 0.3; /* Lower opacity than candles */
        }
        .change-text {
            font-size: 14px;
            fill: #333;
            text-anchor: middle;
        }
        .nav-button {
            font-size: 16px;
            padding: 10px 20px;
            margin: 0 10px;
            cursor: pointer;
            background-color: #f0f0f0;
            border: 1px solid #888;
            border-radius: 5px;
        }
        .nav-button:hover {
            background-color: #e0e0e0;
        }
        .area {
            fill: steelblue;
            opacity: 0.7;
        }
        .area-comparison {
            opacity: 0.5; /* Semi-transparent to see overlapping areas */
        }
        .axis path, .axis line {
            stroke: #000;
        }
        .legend {
            font-size: 14px;
            cursor: pointer;
        }
        .legend rect {
            width: 15px;
            height: 15px;
            cursor: pointer;
        }
        .legend.hidden {
            opacity: 0.3;
            text-decoration: line-through;
        }
</style>
