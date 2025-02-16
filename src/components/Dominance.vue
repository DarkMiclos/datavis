<template>
    <div id="wrapper">
        <div>
            <svg id="dominance"></svg>
        </div>
        <div>
            <svg id="fearGreed"></svg>
        </div>
        <div>
            <svg id="lineChart"></svg>
        </div>
        <div class="chart-container">
        <button id="prev-button" class="nav-button">Previous</button>
        <div id="chart"></div>
        <button id="next-button" class="nav-button">Next</button>
        </div>
        <div class="slider-container">
            <label for="date-slider">Date Range:</label>
            <div class="slider" id="date-slider"></div>
        </div>
        <div class="chart-container">
        <button class="nav-button" id="prev-button-candle">Previous</button>
        <div id="chart-candle"></div>
        <button class="nav-button" id="next-button-candle">Next</button>
        </div>
        <div class="slider-container">
            <label for="date-slider">Date Range:</label>
            <div class="slider" id="date-slider-candle"></div>
        </div>
        <div class="chart-container">
            <div id="chart-comparison"></div>
        </div>
        <div class="slider-container">
            <label for="date-slider">Date Range:</label>
            <div class="slider" id="date-slider-comparison"></div>
        </div>
    </div>
</template>

<script>
import axios from "axios"
import * as d3 from "d3"
import { defineComponent, onMounted } from "vue"
import { sliderBottom } from 'd3-simple-slider';

export default defineComponent({
mounted() {
    // Call chart creation functions when the component mounts
    this.createDominancePieChart();
    this.createFearGreedGauge()
    this.createLineChart();
    this.createCandleStick();
    this.createBtcPriceChart();
    this.createMcapComparison();
  },
  methods: {
    //Calls the latest prices
    async callApiLatest() {

      return await axios.get("/api/v1/cryptocurrency/listings/latest", {
        headers: {
              'X-CMC_PRO_API_KEY': '2ddb25f6-3d62-42fb-82eb-be00fa8b1d01',
              'Access-Control-Allow-Origin': '*',
        },
      })
      .then(res =>  res.data.data)
    },
    //Calls the latest Fear and Greed Index
    async callApiFearGreed() {

      return await axios.get("/api/v3/fear-and-greed/latest", {
        headers: {
              'X-CMC_PRO_API_KEY': '2ddb25f6-3d62-42fb-82eb-be00fa8b1d01',
              'Access-Control-Allow-Origin': '*',
        },
      })
      .then(res =>  res.data.data)
    },
    async callApiFearGreedHistorical() {

    return await axios.get("/api/v3/fear-and-greed/historical", {
      headers: {
            'X-CMC_PRO_API_KEY': '2ddb25f6-3d62-42fb-82eb-be00fa8b1d01',
            'Access-Control-Allow-Origin': '*',
      },
    })
    .then(res =>  res.data.data)
    },
    formatMarketCap(value) {
    if (value >= 1e12) {  // Trillion
        const num = value / 1e12;
        return `${num.toFixed(num >= 100 ? 0 : num >= 10 ? 1 : 2)}T`;
    } else if (value >= 1e9) {  // Billion
        const num = value / 1e9;
        return `${num.toFixed(num >= 100 ? 0 : num >= 10 ? 1 : 2)}B`;
    }
    return value.toString();  // For smaller values
    },
    async createDominancePieChart() {
    const width = 500;
    const height = 500;
    
    // Fetch data and calculate percentages (keeping your existing logic)
    let dataset = await this.callApiLatest();
    let sum = 0;
    let remaining = 0;
    let tmp = [];
    
    for (const value of dataset) {
        sum = sum + parseInt(value.quote.USD.market_cap);
    }
    remaining = sum;
    
    for (const value of dataset) {
        let percent = parseInt(value.quote.USD.market_cap) / sum * 100;
        if (percent > 5) {
            remaining = remaining - value.quote.USD.market_cap;
            tmp.push({'name': value.name, 'value': percent, 'mcap':parseInt(value.quote.USD.market_cap)});
        }
    }
    tmp.push({'name': 'Other', 'value': remaining / sum * 100, 'mcap':remaining});

    // Color scale
    const color = d3.scaleOrdinal(d3.schemeSet3);

    // Create SVG
    const svg = d3.select("#dominance")
        .attr("width", width)
        .attr("height", height);

    // Pie layout
    const pie = d3.pie()
        .value(d => d.value)
        .sort(null);  // Prevent automatic sorting

    const data = pie(tmp);

    // Arc generator
    const arc = d3.arc()
        .innerRadius(100)
        .outerRadius(200)
        .cornerRadius(10)
        .padAngle(0.01);

    // Hover arc (slightly larger for hover effect)
    const arcHover = d3.arc()
        .innerRadius(100)
        .outerRadius(210)  // Slightly larger radius for hover
        .cornerRadius(10)
        .padAngle(0.01);

    // Main group
    const mainG = svg.append('g')
        .attr("transform", `translate(${width/2}, ${height/2})`);

    // Create groups for each pie slice
    // Create groups for each pie slice
    const slices = mainG.selectAll(".arc")
        .data(data)
        .enter()
        .append("g")
        .attr("class", "arc")
        .style("cursor", "pointer")  // Add cursor to entire group
        .on("mouseover", function(event, d) {
            // Animate pie slice
            d3.select(this)
                .select("path")
                .transition()
                .duration(200)
                .attr("d", arcHover);

            // Animate main label
            d3.select(this)
                .select(".label-text")
                .transition()
                .duration(200)
                .style("font-size", "14px")
                .style("fill", "#000");

            // Show and animate tooltip
            d3.select(this)
                .select(".tooltip-text")
                .style("opacity", 1)
                .transition()
                .duration(200)
                .style("font-size", "12px")
                .style("fill", "#333");
        })
        .on("mouseout", function(event, d) {
            // Reset pie slice
            d3.select(this)
                .select("path")
                .transition()
                .duration(200)
                .attr("d", arc);

            // Reset main label
            d3.select(this)
                .select(".label-text")
                .transition()
                .duration(200)
                .style("font-size", "12px")
                .style("fill", "#333");

            // Hide and reset tooltip
            d3.select(this)
                .select(".tooltip-text")
                .transition()
                .duration(200)
                .style("opacity", 0)
                .style("font-size", "10px")
                .style("fill", "#666");
        });

    // Add paths (pie slices)
    slices.append("path")
        .attr("d", arc)
        .attr("fill", d => color(d.data.name))
        .attr("stroke", "white")
        .attr("stroke-width", 2);

    // Add labels with percentage
    slices.append("text")
        .attr("class", "label-text")
        .attr("transform", d => `translate(${arc.centroid(d)})`)
        .attr("text-anchor", "middle")
        .style("font-size", "12px")
        .style("fill", "#333")
        .text(d => `${d.data.name} (${d.data.value.toFixed(1)}%)`);

    // Add tooltip text
    slices.append("text")
        .attr("class", "tooltip-text")
        .attr("transform", d => `translate(${arc.centroid(d)})`)
        .attr("dy", "20px")
        .attr("text-anchor", "middle")
        .style("font-size", "10px")
        .style("fill", "#666")
        .style("opacity", 0)
        .text(d => `Market Cap: ${this.formatMarketCap(d.data.mcap)}`);
  // Add total market cap in the center
    mainG.append("text")
        .attr("class", "center-label")
        .attr("text-anchor", "middle")
        .attr("dy", "0.35em")  // Slight vertical offset for better centering
        .style("font-size", "16px")
        .style("font-weight", "bold")
        .style("fill", "#333")
        .text(`Total: ${this.formatMarketCap(sum)}`);
    },
    async createFearGreedGauge() {
    const width = 240; // Reduced from 300
    const height = 200; // Kept the reduced height
    const radius = Math.min(width, height) / 2 - 40; // Reduced radius for smaller chart

    const data = await this.callApiFearGreed();
    const value = data.value;
    const classification = data.value_classification;

    // Create SVG with centered viewBox
    const svg = d3.select("#fearGreed")
        .attr("width", width)
        .attr("height", height)
        .attr("viewBox", `0 0 ${width} ${height}`)
        .append("g")
        .attr("transform", `translate(${width / 2}, ${height / 2})`); // Center the gauge

    const scale = d3.scaleLinear()
        .domain([0, 100])
        .range([-Math.PI/2, Math.PI/2]);

    const color = d3.scaleOrdinal()
        .domain(['Extreme Fear', 'Fear', 'Neutral', 'Greed', 'Extreme Greed'])
        .range(['#E63946', '#E76F51', '#F4A261', '#2A9D8F', '#264653']);

    const arc = d3.arc()
        .innerRadius(radius * 0.7)
        .outerRadius(radius)
        .startAngle(d => scale(d.start))
        .endAngle(d => scale(d.end));

    const zones = [
        {start: 0, end: 20, label: 'Extreme Fear'},
        {start: 20, end: 40, label: 'Fear'},
        {start: 40, end: 60, label: 'Neutral'},
        {start: 60, end: 80, label: 'Greed'},
        {start: 80, end: 100, label: 'Extreme Greed'}
    ];

    // Draw zones with hover effects
    const zonesG = svg.selectAll('.zone')
        .data(zones)
        .enter()
        .append('g')
        .attr('class', 'zone-group')
        .on('mouseover', function(event, d) {
            // Animate the zone
            d3.select(this).select('path')
                .transition()
                .duration(200)
                .attr('d', d3.arc()
                    .innerRadius(radius * 0.7)
                    .outerRadius(radius + 8) // Reduced hover expansion
                    .startAngle(scale(d.start))
                    .endAngle(scale(d.end)));
            
            // Show tooltip
            d3.select(this).select('.tooltip')
                .style('opacity', 1);
        })
        .on('mouseout', function(event, d) {
            // Reset the zone
            d3.select(this).select('path')
                .transition()
                .duration(200)
                .attr('d', arc);
            
            // Hide tooltip
            d3.select(this).select('.tooltip')
                .style('opacity', 0);
        });

    zonesG.append('path')
        .attr('class', 'zone')
        .attr('d', arc)
        .attr('fill', d => color(d.label));

    // Add invisible tooltips with adjusted positioning
    zonesG.append('text')
        .attr('class', 'tooltip')
        .attr('dy', '0.35em')
        .attr('transform', d => {
            const pos = scale((d.start + d.end) / 2);
            const x = (radius + 25) * Math.cos(pos); // Reduced tooltip distance
            const y = (radius + 25) * Math.sin(pos);
            return `translate(${x}, ${y}) rotate(${pos * 180 / Math.PI})`;
        })
        .style('text-anchor', d => scale((d.start + d.end) / 2) > 0 ? 'start' : 'end')
        .style('font-size', '10px') // Smaller font for tooltips
        .style('fill', '#fff')
        .style('opacity', 0)
        .text(d => d.label);

    // Needle with hover animation
    const needle = svg.append('line')
        .attr('class', 'needle')
        .attr('x1', 0)
        .attr('y1', 0)
        .attr('x2', 0)
        .attr('y2', -radius * 0.6)
        .attr('stroke', 'black')
        .attr('stroke-width', 3) // Reduced stroke width
        .attr('transform', `rotate(${scale(value) * 180 / Math.PI})`)
        .on('mouseover', function() {
            d3.select(this)
                .transition()
                .duration(200)
                .attr('stroke-width', 5); // Reduced hover stroke width
        })
        .on('mouseout', function() {
            d3.select(this)
                .transition()
                .duration(200)
                .attr('stroke-width', 3);
        });

    svg.append('circle')
        .attr('cx', 0)
        .attr('cy', 0)
        .attr('r', 8) // Reduced center circle size
        .attr('fill', '#333');

    // Adjust the position of the current value text
    svg.append('text')
        .attr('class', 'current-value')
        .attr('text-anchor', 'middle')
        .attr('dy', '0.35em')
        .attr('y', radius * 0.3)
        .style('font-size', '12px') // Smaller font
        .text(`${value} ${classification}`);

    // Adjust the position of the title text
    svg.append('text')
        .attr('class', 'title')
        .attr('text-anchor', 'middle')
        .attr('y', -radius - 8) // Adjusted for smaller size
        .style('font-size', '12px') // Smaller font
        .text('Fear and Greed Index');
},
async createLineChart() {
    const data = await this.callApiFearGreedHistorical();

    // Convert timestamps to Date objects
    data.forEach(d => {
        d.date = new Date(parseInt(d.timestamp) * 1000);
    });

    // Set up the dimensions and margins
    const margin = { top: 30, right: 30, bottom: 60, left: 60 };
    const width = 800 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    // Create SVG
    const svg = d3.select("#lineChart")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

    // Scales
    const x = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
        .range([0, width]);

    const y = d3.scaleLinear()
        .domain([0, 100]) // Assuming the max value is 100
        .range([height, 0]);

    // Define color mapping based on value_classification
    const colorMap = {
        "Extreme Fear": '#E63946',  // Red
        "Fear": '#E76F51',          // Orange
        "Neutral": '#F4A261',       // Light Orange
        "Greed": '#2A9D8F',         // Teal
        "Extreme Greed": '#264653'  // Dark Blue
    };

    // Line generator
    const line = d3.line()
        .x(d => x(d.date))
        .y(d => y(d.value));

    // Add X axis
    svg.append("g")
        .attr("transform", `translate(0, ${height})`)
        .call(d3.axisBottom(x).tickFormat(d3.timeFormat("%b %d")));

    // Add Y axis
    svg.append("g")
        .call(d3.axisLeft(y));

    // Instead of a single line, we'll draw line segments between consecutive points
    // to allow for color changes based on classification
    svg.selectAll(".line-segment")
        .data(data.slice(0, -1)) // Exclude the last point to avoid out-of-bounds
        .enter()
        .append("path")
        .attr("fill", "none")
        .attr("stroke", (d, i) => colorMap[d.value_classification] || "steelblue") // Fallback color
        .attr("stroke-width", 1.5)
        .attr("d", (d, i) => {
            const nextPoint = data[i + 1];
            return line([d, nextPoint]); // Draw a line segment between current and next point
        });

    // Set up tooltip
    const tooltip = d3.select("body").append("div")
        .attr("class", "tooltip")
        .style("opacity", 0)
        .style("position", "absolute")
        .style("background-color", "white")
        .style("border", "solid")
        .style("border-width", "1px")
        .style("border-radius", "5px")
        .style("padding", "10px");

    // Add dots for each data point with tooltips, size change on hover, and color based on classification
    svg.selectAll("dot")
        .data(data)
        .enter()
        .append("circle")
        .attr("cx", d => x(d.date))
        .attr("cy", d => y(d.value))
        .attr("r", 3) // Initial radius
        .attr("fill", d => colorMap[d.value_classification] || "steelblue") // Fallback color
        .on("mouseover", function(event, d) {
            // Increase the size of the dot
            d3.select(this)
                .transition()
                .duration(200)
                .attr("r", 6); // Larger radius on hover

            // Show tooltip
            tooltip.transition()
                .duration(200)
                .style("opacity", .9);
            tooltip.html(`Date: ${d3.timeFormat("%b %d, %Y")(d.date)}<br>Value: ${d.value}<br>Classification: ${d.value_classification}`)
                .style("left", (event.pageX + 10) + "px")
                .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", function(d) {
            // Reset the size of the dot
            d3.select(this)
                .transition()
                .duration(200)
                .attr("r", 3); // Back to original radius

            // Hide tooltip
            tooltip.transition()
                .duration(500)
                .style("opacity", 0);
        });

    // Add X axis label
    svg.append("text")
        .attr("transform", `translate(${width / 2}, ${height + margin.top + 20})`)
        .style("text-anchor", "middle")
        .text("Date");

    // Add Y axis label
    svg.append("text")
        .attr("transform", "rotate(-90)")
        .attr("y", 0 - margin.left)
        .attr("x", 0 - (height / 2))
        .attr("dy", "1em")
        .style("text-anchor", "middle")
        .text("Fear and Greed Index");

    // Add title
    svg.append("text")
        .attr("x", width / 2)
        .attr("y", 0 - (margin.top / 2))
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .style("text-decoration", "underline")
        .text("Fear and Greed Index Over Time");

    // Optionally, add a legend to show the color mapping
    const legend = svg.append("g")
        .attr("transform", `translate(${width - 120}, 10)`);

    const legendData = Object.entries(colorMap);
    legend.selectAll(".legend-item")
        .data(legendData)
        .enter()
        .append("g")
        .attr("class", "legend-item")
        .attr("transform", (d, i) => `translate(0, ${i * 20})`)
        .each(function(d) {
            const g = d3.select(this);
            g.append("rect")
                .attr("width", 10)
                .attr("height", 10)
                .attr("fill", d[1]);
            g.append("text")
                .attr("x", 15)
                .attr("y", 10)
                .text(d[0])
                .style("font-size", "12px");
        });
    },
    createBtcPriceChart() {
    // Set up chart dimensions
const margin = { top: 40, right: 30, bottom: 50, left: 80 };
const width = 800 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

// List of CSV files
const csvFiles = [
    "coin_Bitcoin.csv",
    "coin_Ethereum.csv",
    "coin_BinanceCoin.csv",
    "coin_Cardano.csv",
    "coin_Cosmos.csv",
    "coin_Dogecoin.csv",
    "coin_Solana.csv",
    "coin_Uniswap.csv",
    "coin_XRP.csv",
    "coin_Aave.csv"
];
let currentCsvIndex = 0;

// Create SVG container
const svg = d3.select("#chart")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

// Initialize scales
let x = d3.scaleTime().range([0, width]);
let y = d3.scaleLinear().range([height, 0]);

// Initialize groups and elements
const areaPath = svg.append("path").attr("class", "area");
const xAxis = svg.append("g").attr("class", "axis").attr("transform", `translate(0,${height})`);
const yAxis = svg.append("g").attr("class", "axis");
const crosshair = svg.append("g").attr("class", "crosshair").style("display", "none");
crosshair.append("line").attr("x1", 0).attr("x2", 0).attr("y1", 0).attr("y2", height);
crosshair.append("line").attr("x1", 0).attr("x2", width).attr("y1", 0).attr("y2", 0);
const focus = svg.append("circle").attr("r", 4).style("fill", "steelblue").style("display", "none");
const xAxisMarker = svg.append("circle").attr("class", "crosshair-marker").attr("r", 5).style("display", "none");
const yAxisMarker = svg.append("circle").attr("class", "crosshair-marker").attr("r", 5).style("display", "none");
const xAxisTooltipGroup = svg.append("g").style("display", "none");
const xAxisTooltipBackground = xAxisTooltipGroup.append("rect").attr("class", "tooltip-background");
const xAxisTooltip = xAxisTooltipGroup.append("text").attr("class", "axis-tooltip").attr("text-anchor", "middle").attr("y", height + 40);
const yAxisTooltipGroup = svg.append("g").style("display", "none");
const yAxisTooltipBackground = yAxisTooltipGroup.append("rect").attr("class", "tooltip-background");
const yAxisTooltip = yAxisTooltipGroup.append("text").attr("class", "axis-tooltip").attr("text-anchor", "end").attr("x", -10);
const changeGroup = svg.append("g").attr("class", "change-group").style("display", "none");
const changeRect = changeGroup.append("rect").attr("class", "change-rect");
const changeText = changeGroup.append("text").attr("class", "change-text");

// Variables to track clicks
let clickCount = 0;
let selectedPoints = [];
let currentData = [];

// Add overlay for interactivity
const overlay = svg.append("rect")
    .attr("width", width)
    .attr("height", height)
    .style("fill", "none")
    .style("pointer-events", "all");

// Function to create or update the BTC price chart
function createBtcPriceChart(csvFile) {
    d3.csv(csvFile).then(data => {
        // Parse the data
        const parseDate = d3.timeParse("%Y-%m-%d %H:%M:%S");
        data.forEach(d => {
            d.Date = parseDate(d.Date);
            d.Close = +d.Close;
        });

        // Sort data by date
        data.sort((a, b) => a.Date - b.Date);
        currentData = data;

        // Update title
        svg.select(".title").remove();
        svg.append("text")
            .attr("class", "title")
            .attr("x", width / 2)
            .attr("y", -10)
            .text(data[0].Name || csvFile);

        // Update scales
        x.domain(d3.extent(data, d => d.Date));
        y.domain([0, d3.max(data, d => d.Close)]);

        // Define the area
        const area = d3.area()
            .x(d => x(d.Date))
            .y0(height)
            .y1(d => y(d.Close));

        // Update area path
        areaPath.datum(data).attr("d", area);

        // Update axes
        xAxis.call(d3.axisBottom(x));
        yAxis.call(d3.axisLeft(y));

        // Reset click state
        clickCount = 0;
        selectedPoints = [];
        changeGroup.style("display", "none");

        // Update slider
        updateSlider(data);

        // Bind event handlers
        overlay.on("mouseover", () => {
            crosshair.style("display", null);
            focus.style("display", null);
            xAxisMarker.style("display", null);
            yAxisMarker.style("display", null);
            xAxisTooltipGroup.style("display", null);
            yAxisTooltipGroup.style("display", null);
        })
        .on("mouseout", () => {
            crosshair.style("display", "none");
            focus.style("display", "none");
            xAxisMarker.style("display", "none");
            yAxisMarker.style("display", "none");
            xAxisTooltipGroup.style("display", "none");
            yAxisTooltipGroup.style("display", "none");
            if (clickCount === 1) {
                changeGroup.style("display", "none");
            }
        })
        .on("mousemove", mousemove)
        .on("click", clickHandler);
    }).catch(error => {
        console.error(`Error loading ${csvFile}:`, error);
    });
}

function mousemove(event) {
    const x0 = x.invert(d3.pointer(event)[0]);
    const i = d3.bisector(d => d.Date).left(currentData, x0, 1);
    const d0 = currentData[i - 1];
    const d1 = currentData[i] || d0;
    const d = x0 - d0.Date > d1.Date - x0 ? d1 : d0;

    crosshair.select("line:nth-child(1)")
        .attr("x1", x(d.Date))
        .attr("x2", x(d.Date));
    crosshair.select("line:nth-child(2)")
        .attr("y1", y(d.Close))
        .attr("y2", y(d.Close));
    focus.attr("cx", x(d.Date)).attr("cy", y(d.Close));
    xAxisMarker.attr("cx", x(d.Date)).attr("cy", height);
    yAxisMarker.attr("cx", 0).attr("cy", y(d.Close));

    const dateText = d3.timeFormat("%Y-%m-%d")(d.Date);
    xAxisTooltip.attr("x", x(d.Date)).text(dateText);
    const xBbox = xAxisTooltip.node().getBBox();
    xAxisTooltipBackground
        .attr("x", xBbox.x - 5)
        .attr("y", xBbox.y - 2)
        .attr("width", xBbox.width + 10)
        .attr("height", xBbox.height + 4);

    const closeText = d.Close.toFixed(2);
    yAxisTooltip.attr("y", y(d.Close)).text(closeText);
    const yBbox = yAxisTooltip.node().getBBox();
    yAxisTooltipBackground
        .attr("x", yBbox.x - 5)
        .attr("y", yBbox.y - 2)
        .attr("width", yBbox.width + 10)
        .attr("height", yBbox.height + 4);

    if (clickCount === 1) {
        showPercentageChange(selectedPoints[0], d);
    }
}

function clickHandler(event) {
    const x0 = x.invert(d3.pointer(event)[0]);
    const i = d3.bisector(d => d.Date).left(currentData, x0, 1);
    const d0 = currentData[i - 1];
    const d1 = currentData[i] || d0;
    const d = x0 - d0.Date > d1.Date - x0 ? d1 : d0;

    clickCount++;
    if (clickCount === 1) {
        selectedPoints = [d];
        changeGroup.style("display", null);
    } else if (clickCount === 2) {
        selectedPoints.push(d);
        showPercentageChange(selectedPoints[0], selectedPoints[1]);
    } else {
        clickCount = 1;
        selectedPoints = [d];
        changeGroup.style("display", null);
    }
}

function showPercentageChange(point1, point2) {
    // Ensure points are ordered by date to determine start and end
    const [startPoint, endPoint] = point1.Date < point2.Date ? [point1, point2] : [point2, point1];

    // Calculate percentage change based on Close prices
    const price1 = startPoint.Close;
    const price2 = endPoint.Close;
    const percentageChange = ((price2 - price1) / price1) * 100;

    // Determine color based on actual price change
    const color = percentageChange >= 0 ? "#26A69A" : "#EF5350"; // Green for up, red for down

    // Show change group
    changeGroup.style("display", null);

    // Draw rectangle between the two points
    const x1 = x(startPoint.Date);
    const x2 = x(endPoint.Date);
    const y1 = y(startPoint.Close);
    const y2 = y(endPoint.Close);
    changeRect
        .attr("x", Math.min(x1, x2))
        .attr("y", Math.min(y1, y2))
        .attr("width", Math.abs(x2 - x1))
        .attr("height", Math.abs(y2 - y1))
        .attr("fill", color);

    // Position text at the midpoint of the rectangle
    const midX = (x1 + x2) / 2;
    const midY = (y1 + y2) / 2;
    changeText
        .attr("x", midX)
        .attr("y", midY - 10) // Position above the rectangle
        .text(`${percentageChange.toFixed(2)}%`)
        .attr("fill", color);
}

function updateSlider(data) {
    d3.select("#date-slider").select("svg").remove();
    const dateExtent = d3.extent(data, d => d.Date);
    const slider = sliderBottom()
        .min(dateExtent[0])
        .max(dateExtent[1])
        .width(600)
        .tickFormat(d3.timeFormat("%Y-%m-%d"))
        .ticks(5)
        .default([dateExtent[0], dateExtent[1]])
        .on("onchange", val => {
            const filteredData = data.filter(d => d.Date >= val[0] && d.Date <= val[1]);
            x.domain(d3.extent(filteredData, d => d.Date));
            y.domain([0, d3.max(filteredData, d => d.Close)]);
            areaPath.datum(filteredData).attr("d", d3.area().x(d => x(d.Date)).y0(height).y1(d => y(d.Close)));
            xAxis.call(d3.axisBottom(x));
            yAxis.call(d3.axisLeft(y));
            if (selectedPoints.length === 2) {
                showPercentageChange(selectedPoints[0], selectedPoints[1]);
            } else if (selectedPoints.length === 1) {
                const x0 = x.invert(d3.pointer(event)[0]);
                const i = d3.bisector(d => d.Date).left(data, x0, 1);
                const d0 = data[i - 1];
                const d1 = data[i] || d0;
                const d = x0 - d0.Date > d1.Date - x0 ? d1 : d0;
                showPercentageChange(selectedPoints[0], d);
            }
        });

    d3.select("#date-slider")
        .append("svg")
        .attr("width", 700)
        .attr("height", 100)
        .append("g")
        .attr("transform", "translate(50,30)")
        .call(slider);
}

// Load initial chart
createBtcPriceChart(csvFiles[currentCsvIndex]);

// Add navigation button event listeners
d3.select("#prev-button").on("click", () => {
    currentCsvIndex = (currentCsvIndex - 1 + csvFiles.length) % csvFiles.length;
    createBtcPriceChart(csvFiles[currentCsvIndex]);
});

d3.select("#next-button").on("click", () => {
    currentCsvIndex = (currentCsvIndex + 1) % csvFiles.length;
    createBtcPriceChart(csvFiles[currentCsvIndex]);
});
    },
    createCandleStick() {
    // Set up chart dimensions
const margin = { top: 40, right: 30, bottom: 50, left: 80 }; // Increased top margin for title
const width = 800 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

// List of CSV files
const csvFiles = [
    "coin_Bitcoin.csv",
    "coin_Ethereum.csv",
    "coin_BinanceCoin.csv",
    "coin_Cardano.csv",
    "coin_Cosmos.csv",
    "coin_Dogecoin.csv",
    "coin_Solana.csv",
    "coin_Uniswap.csv",
    "coin_XRP.csv",
    "coin_Aave.csv"
];
let currentCsvIndex = 0;

// Create SVG container
const svg = d3.select("#chart-candle")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

// Function to load and render chart
function loadChart(csvFile) {
    // Clear existing elements
    svg.selectAll("*").remove();
    d3.select("#date-slider-candle").selectAll("*").remove();

    // Load and process data
    d3.csv(csvFile).then(data => {
        // Parse the data
        const parseDate = d3.timeParse("%Y-%m-%d %H:%M:%S"); // Match CSV date format
        data.forEach(d => {
            d.Date = parseDate(d.Date); // Parse date string to Date object
            d.Close = +d.Close; // Convert string to number
            d.Open = +d.Open;
            d.High = +d.High;
            d.Low = +d.Low;
        });

        // Sort data by date (if not already sorted)
        data.sort((a, b) => a.Date - b.Date);

        // Get the chart title from the first row's Name (or adjust as needed)
        const chartTitle = data[0].Name || "Candlestick Chart";

        // Add title to the chart
        svg.append("text")
            .attr("class", "title")
            .attr("x", width / 2)
            .attr("y", -10)
            .text(chartTitle);

        // Set up initial scales (will be updated by slider)
        let x = d3.scaleTime()
            .domain(d3.extent(data, d => d.Date))
            .range([0, width]);

        let y = d3.scaleLinear()
            .domain([d3.min(data, d => d.Low), d3.max(data, d => d.High)]) // Use Low and High for full range
            .range([height, 0]);

        // Add x-axis
        const xAxis = svg.append("g")
            .attr("class", "axis")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(x));

        // Add y-axis
        const yAxis = svg.append("g")
            .attr("class", "axis")
            .call(d3.axisLeft(y));

        // Add candlesticks
        const candlestickGroup = svg.append("g")
            .attr("class", "candlesticks");

        function drawCandlesticks(filteredData) {
            // Calculate bar width based on the number of data points
            const barWidth = Math.min(10, width / filteredData.length * 0.6); // Ensure bars are not too wide

            // Update candlesticks
            const candles = candlestickGroup.selectAll(".candle")
                .data(filteredData, d => d.Date);

            // Enter
            const candlesEnter = candles.enter()
                .append("g")
                .attr("class", "candle");

            // Add wicks (high-low lines)
            candlesEnter.append("line")
                .attr("class", "wick")
                .attr("x1", d => x(d.Date))
                .attr("x2", d => x(d.Date))
                .attr("y1", d => y(d.High))
                .attr("y2", d => y(d.Low));

            // Add bodies (open-close rectangles)
            candlesEnter.append("rect")
                .attr("x", d => x(d.Date) - barWidth / 2)
                .attr("y", d => y(Math.max(d.Open, d.Close)))
                .attr("width", barWidth)
                .attr("height", d => Math.abs(y(d.Open) - y(d.Close)))
                .attr("class", d => d.Close > d.Open ? "bullish" : "bearish");

            // Update
            candles.select(".wick")
                .attr("x1", d => x(d.Date))
                .attr("x2", d => x(d.Date))
                .attr("y1", d => y(d.High))
                .attr("y2", d => y(d.Low));

            candles.select("rect")
                .attr("x", d => x(d.Date) - barWidth / 2)
                .attr("y", d => y(Math.max(d.Open, d.Close)))
                .attr("width", barWidth)
                .attr("height", d => Math.abs(y(d.Open) - y(d.Close)))
                .attr("class", d => d.Close > d.Open ? "bullish" : "bearish");

            // Exit
            candles.exit().remove();
        }

        // Initial draw
        drawCandlesticks(data);

        // Add crosshair
        const crosshair = svg.append("g")
            .attr("class", "crosshair")
            .style("display", "none");

        crosshair.append("line")
            .attr("x1", 0)
            .attr("x2", 0)
            .attr("y1", 0)
            .attr("y2", height); // Vertical line

        crosshair.append("line")
            .attr("x1", 0)
            .attr("x2", width)
            .attr("y1", 0)
            .attr("y2", 0); // Horizontal line

        // Add focus circle for data point (positioned at close price)
        const focus = svg.append("circle")
            .attr("r", 4)
            .style("fill", "steelblue")
            .style("display", "none");

        // Add crosshair markers on axes
        const xAxisMarker = svg.append("circle")
            .attr("class", "crosshair-marker")
            .attr("r", 5)
            .style("display", "none");

        const yAxisMarker = svg.append("circle")
            .attr("class", "crosshair-marker")
            .attr("r", 5)
            .style("display", "none");

        // Add axis tooltips (text labels with background)
        const xAxisTooltipGroup = svg.append("g")
            .style("display", "none");

        const xAxisTooltipBackground = xAxisTooltipGroup.append("rect")
            .attr("class", "tooltip-background");

        const xAxisTooltip = xAxisTooltipGroup.append("text")
            .attr("class", "axis-tooltip")
            .attr("text-anchor", "middle")
            .attr("y", height + 40); // Position below x-axis

        const yAxisTooltipGroup = svg.append("g")
            .style("display", "none");

        const yAxisTooltipBackground = yAxisTooltipGroup.append("rect")
            .attr("class", "tooltip-background");

        const yAxisTooltip = yAxisTooltipGroup.append("text")
            .attr("class", "axis-tooltip")
            .attr("text-anchor", "end")
            .attr("x", -10); // Position to the left of y-axis

        // Add group for percentage change visualization
        const changeGroup = svg.append("g")
            .attr("class", "change-group")
            .style("display", "none");

        const changeRect = changeGroup.append("rect")
            .attr("class", "change-rect");

        const changeText = changeGroup.append("text")
            .attr("class", "change-text");

        // Variables to track clicks
        let clickCount = 0;
        let selectedPoints = [];

        // Add overlay for interactivity
        const overlay = svg.append("rect")
            .attr("width", width)
            .attr("height", height)
            .style("fill", "none")
            .style("pointer-events", "all")
            .on("mouseover", () => {
                crosshair.style("display", null);
                focus.style("display", null);
                xAxisMarker.style("display", null);
                yAxisMarker.style("display", "none"); // Hide y-axis marker for candlestick
                xAxisTooltipGroup.style("display", null);
                yAxisTooltipGroup.style("display", null);
            })
            .on("mouseout", () => {
                crosshair.style("display", "none");
                focus.style("display", "none");
                xAxisMarker.style("display", "none");
                yAxisMarker.style("display", "none");
                xAxisTooltipGroup.style("display", "none");
                yAxisTooltipGroup.style("display", "none");
                if (clickCount === 1) {
                    changeGroup.style("display", "none");
                }
            })
            .on("mousemove", mousemove)
            .on("click", clickHandler);

        // Bisector to find the closest data point
        const bisectDate = d3.bisector(d => d.Date).left;

        function mousemove(event) {
            const x0 = x.invert(d3.pointer(event)[0]);
            const i = bisectDate(data, x0, 1);
            const d0 = data[i - 1];
            const d1 = data[i] || d0;
            const d = x0 - d0.Date > d1.Date - x0 ? d1 : d0;

            // Update crosshair position
            crosshair.select("line:nth-child(1)")
                .attr("x1", x(d.Date))
                .attr("x2", x(d.Date));

            crosshair.select("line:nth-child(2)")
                .attr("y1", y(d.Close))
                .attr("y2", y(d.Close));

            // Update focus circle position (at close price)
            focus.attr("cx", x(d.Date))
                .attr("cy", y(d.Close));

            // Update crosshair markers on axes
            xAxisMarker.attr("cx", x(d.Date))
                .attr("cy", height);

            // Update axis tooltips
            const dateText = d3.timeFormat("%Y-%m-%d")(d.Date);
            xAxisTooltip.attr("x", x(d.Date))
                .text(dateText);

            // Update x-axis tooltip background
            const xBbox = xAxisTooltip.node().getBBox();
            xAxisTooltipBackground
                .attr("x", xBbox.x - 5)
                .attr("y", xBbox.y - 2)
                .attr("width", xBbox.width + 10)
                .attr("height", xBbox.height + 4);

            const closeText = d.Close.toFixed(2);
            yAxisTooltip.attr("y", y(d.Close))
                .text(closeText);

            // Update y-axis tooltip background
            const yBbox = yAxisTooltip.node().getBBox();
            yAxisTooltipBackground
                .attr("x", yBbox.x - 5)
                .attr("y", yBbox.y - 2)
                .attr("width", yBbox.width + 10)
                .attr("height", yBbox.height + 4);

            // If one point is selected, dynamically update the change visualization
            if (clickCount === 1) {
                showPercentageChange(selectedPoints[0], d);
            }
        }

        function clickHandler(event) {
            const x0 = x.invert(d3.pointer(event)[0]);
            const i = bisectDate(data, x0, 1);
            const d0 = data[i - 1];
            const d1 = data[i] || d0;
            const d = x0 - d0.Date > d1.Date - x0 ? d1 : d0;

            clickCount++;

            if (clickCount === 1) {
                // First click: store the point and start dynamic visualization
                selectedPoints = [d];
                changeGroup.style("display", null);
            } else if (clickCount === 2) {
                // Second click: store the point and lock in the visualization
                selectedPoints.push(d);
                showPercentageChange(selectedPoints[0], selectedPoints[1]);
            } else {
                // Third click: reset to behave like the first click
                clickCount = 1;
                selectedPoints = [d];
                changeGroup.style("display", null);
            }
        }


function showPercentageChange(point1, point2) {
    // Ensure points are ordered by date to determine start and end
    const [startPoint, endPoint] = point1.Date < point2.Date ? [point1, point2] : [point2, point1];

    // Calculate percentage change based on Close prices
    const price1 = startPoint.Close;
    const price2 = endPoint.Close;
    const percentageChange = ((price2 - price1) / price1) * 100;

    // Determine color based on actual price change (not candlestick direction)
    const color = percentageChange >= 0 ? "#26A69A" : "#EF5350"; // Green for up, red for down

    // Show change group
    changeGroup.style("display", null);

    // Draw rectangle between the two points
    const x1 = x(startPoint.Date);
    const x2 = x(endPoint.Date);
    const y1 = y(startPoint.Close);
    const y2 = y(endPoint.Close);
    changeRect
        .attr("x", Math.min(x1, x2))
        .attr("y", Math.min(y1, y2))
        .attr("width", Math.abs(x2 - x1))
        .attr("height", Math.abs(y2 - y1))
        .attr("fill", color);

    // Position text at the midpoint of the rectangle
    const midX = (x1 + x2) / 2;
    const midY = (y1 + y2) / 2;
    changeText
        .attr("x", midX)
        .attr("y", midY - 10) // Position above the rectangle
        .text(`${percentageChange.toFixed(2)}%`)
        .attr("fill", color);
}

        // Add date range slider
        const dateExtent = d3.extent(data, d => d.Date);
        const slider = sliderBottom()
            .min(dateExtent[0])
            .max(dateExtent[1])
            .width(600)
            .tickFormat(d3.timeFormat("%Y-%m-%d"))
            .ticks(5)
            .default([dateExtent[0], dateExtent[1]])
            .on("onchange", val => {
                // Filter data based on selected date range
                const filteredData = data.filter(d => d.Date >= val[0] && d.Date <= val[1]);

                // Update x scale domain
                x.domain(d3.extent(filteredData, d => d.Date));

                // Update y scale domain based on filtered data (use Low and High for full range)
                y.domain([d3.min(filteredData, d => d.Low), d3.max(filteredData, d => d.High)]);

                // Update candlesticks
                drawCandlesticks(filteredData);

                // Update x-axis
                xAxis.call(d3.axisBottom(x));

                // Update y-axis
                yAxis.call(d3.axisLeft(y));

                // Update overlay to reflect new x and y scales
                overlay.on("mousemove", mousemove);

                // If two points are selected, update the percentage change visualization
                if (selectedPoints.length === 2) {
                    showPercentageChange(selectedPoints[0], selectedPoints[1]);
                } else if (selectedPoints.length === 1) {
                    // If one point is selected, update dynamically with the current mouse position
                    const x0 = x.invert(d3.pointer(event)[0]);
                    const i = bisectDate(data, x0, 1);
                    const d0 = data[i - 1];
                    const d1 = data[i] || d0;
                    const d = x0 - d0.Date > d1.Date - x0 ? d1 : d0;
                    showPercentageChange(selectedPoints[0], d);
                }
            });

        // Add slider to the page
        d3.select("#date-slider-candle")
            .append("svg")
            .attr("width", 700)
            .attr("height", 100)
            .append("g")
            .attr("transform", "translate(50,30)")
            .call(slider);

    }).catch(error => {
        console.error("Error loading the CSV file:", error);
    });
}

// Initial load
loadChart(csvFiles[currentCsvIndex]);

// Add button event listeners
d3.select("#prev-button-candle").on("click", () => {
    currentCsvIndex = (currentCsvIndex - 1 + csvFiles.length) % csvFiles.length; // Wrap around
    loadChart(csvFiles[currentCsvIndex]);
});

d3.select("#next-button-candle").on("click", () => {
    currentCsvIndex = (currentCsvIndex + 1) % csvFiles.length; // Wrap around
    loadChart(csvFiles[currentCsvIndex]);
});
    },
    createMcapComparison() {
    // Set up chart dimensions
const margin = { top: 40, right: 150, bottom: 50, left: 80 }; // Increased right margin for legend
const width = 900 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

// List of CSV files
const csvFiles = [
    "coin_Bitcoin.csv",
    "coin_Ethereum.csv",
    "coin_BinanceCoin.csv",
    "coin_Cardano.csv",
    "coin_Cosmos.csv",
    "coin_Dogecoin.csv",
    "coin_Solana.csv",
    "coin_Uniswap.csv",
    "coin_XRP.csv",
    "coin_Aave.csv"
];

// Color scale for different cryptocurrencies
const colorScale = d3.scaleOrdinal(d3.schemeCategory10);

// Create SVG container
const svg = d3.select("#chart-comparison")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

// Add title
svg.append("text")
    .attr("class", "title")
    .attr("x", width / 2)
    .attr("y", -10)
    .text("Market Cap Comparison");

// Initialize scales
let x = d3.scaleTime().range([0, width]);
let y = d3.scaleLinear().range([height, 0]);

// Initialize groups and elements
const areaGroup = svg.append("g").attr("class", "areas");
const xAxis = svg.append("g").attr("class", "axis").attr("transform", `translate(0,${height})`);
const yAxis = svg.append("g").attr("class", "axis");

// Store data and visibility state
let allData = [];
const visibility = new Map();

// Function to update y-axis scale based on visible data
function updateYScale(data) {
    const visibleData = data.filter(d => visibility.get(d.name));
    const visibleMarketCaps = visibleData.flatMap(d => d.data.map(e => e.Marketcap));
    y.domain([0, d3.max(visibleMarketCaps) || 1]); // Avoid zero domain
    yAxis.call(d3.axisLeft(y));
}

// Function to load and render the chart
function createMarketCapChart() {
    // Load all CSV files
    Promise.all(csvFiles.map(file => d3.csv(file))).then(datasets => {
        // Parse and process each dataset
        const parseDate = d3.timeParse("%Y-%m-%d %H:%M:%S");
        datasets.forEach((data, index) => {
            data.forEach(d => {
                d.Date = parseDate(d.Date);
                d.Marketcap = +d.Marketcap;
            });
            data.sort((a, b) => a.Date - b.Date);
            allData.push({
                name: data[0].Name || csvFiles[index],
                data: data,
                color: colorScale(index)
            });
            visibility.set(data[0].Name || csvFiles[index], true); // Initially visible
        });

        // Update scales with full data
        const allDates = allData.flatMap(d => d.data.map(e => e.Date));
        const allMarketCaps = allData.flatMap(d => d.data.map(e => e.Marketcap));
        x.domain(d3.extent(allDates));
        y.domain([0, d3.max(allMarketCaps)]);

        // Update axes
        xAxis.call(d3.axisBottom(x));
        yAxis.call(d3.axisLeft(y));

        // Draw areas
        drawAreas(allData);

        // Add legend
        addLegend();

        // Add slider
        addSlider();
    }).catch(error => {
        console.error("Error loading CSV files:", error);
    });
}

function drawAreas(data) {
    // Define the area generator
    const area = d3.area()
        .x(d => x(d.Date))
        .y0(height)
        .y1(d => y(d.Marketcap));

    // Update areas
    const areas = areaGroup.selectAll(".area-comparison")
        .data(data, d => d.name);

    // Enter
    areas.enter()
        .append("path")
        .attr("class", "area-comparison")
        .attr("d", d => area(d.data))
        .attr("fill", d => d.color)
        .style("display", d => visibility.get(d.name) ? null : "none");

    // Update
    areas
        .attr("d", d => area(d.data))
        .attr("fill", d => d.color)
        .style("display", d => visibility.get(d.name) ? null : "none");

    // Exit
    areas.exit().remove();
}

function addLegend() {
    const legend = svg.selectAll(".legend")
        .data(allData, d => d.name);

    const legendEnter = legend.enter()
        .append("g")
        .attr("class", "legend")
        .attr("transform", (d, i) => `translate(${width + 20},${i * 20})`)
        .on("click", (event, d) => {
            // Toggle visibility
            const isVisible = visibility.get(d.name);
            visibility.set(d.name, !isVisible);
            
            // Update class for visual feedback
            d3.select(event.currentTarget)
                .classed("hidden", isVisible);
            
            // Update y-axis scale based on visible data
            updateYScale(allData);

            // Redraw areas
            drawAreas(allData);
        });

    legendEnter.append("rect")
        .attr("x", 0)
        .attr("y", 0)
        .attr("fill", d => d.color);

    legendEnter.append("text")
        .attr("x", 20)
        .attr("y", 12)
        .text(d => d.name);

    // Update existing legend items
    legend.select("rect")
        .attr("fill", d => d.color);
    legend.select("text")
        .text(d => d.name);
}

function addSlider() {
    // Find the common date range across all datasets
    const allDates = allData.flatMap(d => d.data.map(e => e.Date));
    const dateExtent = d3.extent(allDates);

    // Remove any existing slider
    d3.select("#date-slider-comparison").select("svg").remove();

    const slider = sliderBottom()
        .min(dateExtent[0])
        .max(dateExtent[1])
        .width(600)
        .tickFormat(d3.timeFormat("%Y-%m-%d"))
        .ticks(5)
        .default([dateExtent[0], dateExtent[1]])
        .on("onchange", val => {
            // Filter each dataset based on the date range
            const filteredData = allData.map(d => ({
                name: d.name,
                data: d.data.filter(e => e.Date >= val[0] && e.Date <= val[1]),
                color: d.color
            }));

            // Update x scale with filtered data
            const filteredDates = filteredData.flatMap(d => d.data.map(e => e.Date));
            x.domain(d3.extent(filteredDates));

            // Update y scale with visible filtered data
            const visibleFilteredData = filteredData.filter(d => visibility.get(d.name));
            const visibleFilteredMarketCaps = visibleFilteredData.flatMap(d => d.data.map(e => e.Marketcap));
            y.domain([0, d3.max(visibleFilteredMarketCaps) || 1]); // Avoid zero domain

            // Update axes
            xAxis.call(d3.axisBottom(x));
            yAxis.call(d3.axisLeft(y));

            // Redraw areas with filtered data
            drawAreas(filteredData);
        });

    d3.select("#date-slider-comparison")
        .append("svg")
        .attr("width", 700)
        .attr("height", 100)
        .append("g")
        .attr("transform", "translate(50,30)")
        .call(slider);
}

// Initialize the chart
createMarketCapChart();
    }
  }
})
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

#dominance, #fearGreed {
    flex: 1;
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    
    /* Center the SVG within the flex item */
    display: flex;
    justify-content: center; /* Center horizontally */
    align-items: center;     /* Center vertically */
}

</style>
