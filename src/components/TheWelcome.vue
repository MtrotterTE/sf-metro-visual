<template>
  <div class="welcome">
    <h1 class="page-title">Tenco CityScale K Line Visualization</h1>
    <div ref="chart" class="svg-area"></div>
    <div class="tooltip" id="tooltip"></div>
  </div>  
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';

const chart = ref(null);
const data1 = ref(null)
const data2 = ref(null)
const data3 = ref(null)

onMounted(async () => {
  try {
    const [file1, file2, file3] = await Promise.all([
      import('../data/stopData-Curated-05-16.json'),
      import('../data/stopData-Curated-05-15.json'),
      import('../data/stopData-Curated-05-14.json')
    ]);

    data1.value = file1.default;
    data2.value = file2.default;
    data3.value = file3.default;

    runAfterLoad();
  } catch (error) {
    console.error('Error loading JSON files:', error);
  }
})

function runAfterLoad() {
  console.log('This runs after all JSONs are loaded!')
  // any additional logic here

  // svg canvas dimensions
  const width = 800;
  const height = 2400;

  const svg = d3.select(chart.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  const tooltip = d3.select("#tooltip");

  svg.append('rect')
    .attr('x', 340)
    .attr('y', 0)
    .attr('width', 10)
    .attr('height', height)
    .attr('fill', 'steelblue');

  svg.append('rect')
    .attr('x', 460)
    .attr('y', 0)
    .attr('width', 10)
    .attr('height', height)
    .attr('fill', 'steelblue');

  let EmbarcaderoStationOutbound = 0;
  let EmbarcaderoStationOutboundNumVehicles = 0;
  let EmbarcaderoStationInbound = 0;
  let EmbarcaderoStationInboundNumVehicles = 0;

  // Cycle through the data
  console.log(Array.isArray(data1.value));
  Object.values(data1.value).forEach((stop) => {
    //console.log(stop);
    if (stop.direction_id === 0) { // outbound
      if (stop.atStation) { // vehicle at station
        if (stop.stationId === "17217" && stop.timeAtStop > 0) { // Embarcadero Station outbound
          EmbarcaderoStationOutbound += stop.timeAtStop;
          EmbarcaderoStationOutboundNumVehicles++;
        }
      } else { // vehicle at intersection

      }
    } else if (stop.direction_id === 1) { // inbound
      if (stop.atStation) { // vehicle at station
        if (stop.stationId === "16992" && stop.timeAtStop > 0) { // Embarcadero Station inbound
          EmbarcaderoStationInbound += stop.timeAtStop;
          EmbarcaderoStationInboundNumVehicles++;
        }
      } else { // vehicle at intersection

      }
    }
  });

  console.log("Embarcadero Station Outbound Total Time: " + EmbarcaderoStationOutbound);
  console.log("Embarcadero Station Outbound Number of Vehicles: " + EmbarcaderoStationOutboundNumVehicles);
  console.log("Embarcadero Station Inbound Total Time: " + EmbarcaderoStationInbound);
  console.log("Embarcadero Station Inbound Number of Vehicles: " + EmbarcaderoStationInboundNumVehicles);

  const outboundCX = 345;
  const inboundCX = 465;
  const stationCYOffset = 100;
  const radiusScaler = 0.01

  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset)
    .attr('r', EmbarcaderoStationOutbound * radiusScaler)
    .attr('class', 'embarcadero-outbound station-circle')
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html(EmbarcaderoStationOutbound);
    })
    .on("mousemove", (event) => {
      tooltip.style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 20) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("opacity", 0);
    });

  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset)
    .attr('r', EmbarcaderoStationInbound * radiusScaler)
    .attr('class', 'embarcadero-inbound station-circle')
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html(EmbarcaderoStationInbound);
    })
    .on("mousemove", (event) => {
      tooltip.style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 20) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("opacity", 0);
    });
}
</script>

<style>
.page-title {
  text-align: center;
  font-size: 2rem;
  color: #C3BFBA;
  text-decoration: underline;
}

.svg-area {
  display: flex;
  justify-content: center;
  align-items: center;
  height: fit-content;
  border: 1px solid #ccc;
}

.station-circle {
  stroke: #fff;
  stroke-width: 3px;
}

.embarcadero-outbound {
  fill: #FF5733; /* Outbound color */
}

.embarcadero-inbound {
  fill: #33C3FF; /* Inbound color */
}

.tooltip {
  position: absolute;
  text-align: center;
  padding: 6px;
  font-size: 12px;
  background: #333;
  color: #fff;
  border-radius: 4px;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s ease;
}
</style>