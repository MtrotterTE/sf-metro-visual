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

  let MontgomeryStationOutbound = 0;
  let MontgomeryStationOutboundNumVehicles = 0;
  let MontgomeryStationInbound = 0;
  let MontgomeryStationInboundNumVehicles = 0;

  // Maximum and minimum average time at station for color scaling
  let maximumAverageTimeAtStation = 50;
  let minimumAverageTimeAtStation = 0;

  // Cycle through the data
  console.log(Array.isArray(data1.value));
  Object.values(data1.value).forEach((stop) => {
    //console.log(stop);
    if (stop.direction_id === 0) { // outbound
      if (stop.atStation) { // vehicle at station
        if (stop.stationId === "17217" && stop.timeAtStop > 0) { // Embarcadero Station outbound
          EmbarcaderoStationOutbound += stop.timeAtStop;
          EmbarcaderoStationOutboundNumVehicles++;
        } else if (stop.stationId === "16994" && stop.timeAtStop > 0) { // Montgomery Station outbound
          MontgomeryStationOutbound += stop.timeAtStop;
          MontgomeryStationOutboundNumVehicles++;
        }
      } else { // vehicle at intersection

      }
    } else if (stop.direction_id === 1) { // inbound
      if (stop.atStation) { // vehicle at station
        if (stop.stationId === "16992" && stop.timeAtStop > 0) { // Embarcadero Station inbound
          EmbarcaderoStationInbound += stop.timeAtStop;
          EmbarcaderoStationInboundNumVehicles++;
        } else if (stop.stationId === "15731" && stop.timeAtStop > 0) { // Montgomery Station inbound
          MontgomeryStationInbound += stop.timeAtStop;
          MontgomeryStationInboundNumVehicles++;
        }
      } else { // vehicle at intersection

      }
    }
  });

  const outboundCX = 345;
  const inboundCX = 465;
  const stationCYOffset = 200;
  const radiusScaler = 0.01

  // Outbound label
  svg.append('text')
    .attr('x', 50)
    .attr('y', 800)
    .attr('transform', 'rotate(-90, 50, 800)')
    .attr('class', 'outbound-label')
    .text('Outbound');

  // Inbound label
  svg.append('text')
    .attr('x', 750)
    .attr('y', 620)
    .attr('transform', 'rotate(90, 750, 620)')
    .attr('class', 'inbound-label')
    .text('Inbound');

  // Embarcadero Station outbound
  let averageTimeEmbarcaderoOutbound = EmbarcaderoStationOutbound / EmbarcaderoStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset)
    .attr('r', EmbarcaderoStationOutbound * radiusScaler)
    .attr('class', 'embarcadero-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeEmbarcaderoOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Embarcadero Station (Outbound).<br>Total Time at Station: " + EmbarcaderoStationOutbound + " seconds<br>Total Vehicles: " + EmbarcaderoStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeEmbarcaderoOutbound.toFixed(2) + " seconds");
    })
    .on("mousemove", (event) => {
      tooltip.style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 20) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("opacity", 0);
    });
  
  svg.append('text')
    .attr('x', outboundCX - 200)
    .attr('y', stationCYOffset + 5)
    .attr('class', 'station-label')
    .text('Embarcadero Station');

  // Embarcadero Station inbound
  let averageTimeEmbarcaderoInbound = EmbarcaderoStationInbound / EmbarcaderoStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset)
    .attr('r', EmbarcaderoStationInbound * radiusScaler)
    .attr('class', 'embarcadero-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeEmbarcaderoInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Embarcadero Station (Inbound).<br>Total Time at Station: " + EmbarcaderoStationInbound + " seconds<br>Total Vehicles: " + EmbarcaderoStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeEmbarcaderoInbound.toFixed(2) + " seconds");
    })
    .on("mousemove", (event) => {
      tooltip.style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 20) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("opacity", 0);
    });

  svg.append('text')
    .attr('x', inboundCX + 30)
    .attr('y', stationCYOffset + 5)
    .attr('class', 'station-label')
    .text('Embarcadero Station');

  // Montgomery Station outbound
  let averageTimeMontgomeryOutbound = MontgomeryStationOutbound / MontgomeryStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 2)
    .attr('r', MontgomeryStationOutbound * radiusScaler)
    .attr('class', 'montgomery-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeMontgomeryOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Montgomery Station (Outbound).<br>Total Time at Station: " + MontgomeryStationOutbound + " seconds<br>Total Vehicles: " + MontgomeryStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeMontgomeryOutbound.toFixed(2) + " seconds");
    })
    .on("mousemove", (event) => {
      tooltip.style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 20) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("opacity", 0);
    });
  
  svg.append('text')
    .attr('x', outboundCX - 200)
    .attr('y', stationCYOffset * 2 + 5)
    .attr('class', 'station-label')
    .text('Montgomery Station');

  // Montgomery Station inbound
  let averageTimeMontgomeryInbound = MontgomeryStationInbound / MontgomeryStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 2)
    .attr('r', MontgomeryStationInbound * radiusScaler)
    .attr('class', 'montgomery-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeMontgomeryInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Montgomery Station (Inbound).<br>Total Time at Station: " + MontgomeryStationInbound + " seconds<br>Total Vehicles: " + MontgomeryStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeMontgomeryInbound.toFixed(2) + " seconds");
    })
    .on("mousemove", (event) => {
      tooltip.style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 20) + "px");
    })
    .on("mouseout", () => {
      tooltip.style("opacity", 0);
    });

  svg.append('text')
    .attr('x', inboundCX + 30)
    .attr('y', stationCYOffset * 2 + 5)
    .attr('class', 'station-label')
    .text('Montgomery Station');
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100..900;1,100..900&display=swap');

.page-title {
  font-family: "Roboto", sans-serif;
  text-align: center;
  font-size: 2rem;
  color: #C3BFBA;
  text-decoration: underline;
  font-weight: 700;
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

.tooltip {
  font-family: "Roboto", sans-serif;
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

.outbound-label, .inbound-label {
  font-family: "Roboto", sans-serif;
  fill: #C3BFBA;
  font-size: 3rem;
  font-weight: 300;
}

.station-label {
  font-family: "Roboto", sans-serif;
  fill: #C3BFBA;
  font-size: 1rem;
  font-weight: 500;
}
</style>