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
  const height = 4000;

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

  let PowellStationOutbound = 0;
  let PowellStationOutboundNumVehicles = 0;
  let PowellStationInbound = 0;
  let PowellStationInboundNumVehicles = 0;

  let CCStationOutbound = 0;
  let CCStationOutboundNumVehicles = 0;
  let CCStationInbound = 0;
  let CCStationInboundNumVehicles = 0;

  let VNStationOutbound = 0;
  let VNStationOutboundNumVehicles = 0;
  let VNStationInbound = 0;
  let VNStationInboundNumVehicles = 0;

  let ChurchStationOutbound = 0;
  let ChurchStationOutboundNumVehicles = 0;
  let ChurchStationInbound = 0;
  let ChurchStationInboundNumVehicles = 0;

  let CastroStationOutbound = 0;
  let CastroStationOutboundNumVehicles = 0;
  let CastroStationInbound = 0;
  let CastroStationInboundNumVehicles = 0;

  let FHStationOutbound = 0;
  let FHStationOutboundNumVehicles = 0;
  let FHStationInbound = 0;
  let FHStationInboundNumVehicles = 0;

  let WPStationOutbound = 0;
  let WPStationOutboundNumVehicles = 0;
  let WPStationInbound = 0;
  let WPStationInboundNumVehicles = 0;

  let WP14StationOutbound = 0;
  let WP14StationOutboundNumVehicles = 0;
  let WP14StationInbound = 0;
  let WP14StationInboundNumVehicles = 0;

  let WPSLStationOutbound = 0;
  let WPSLStationOutboundNumVehicles = 0;
  let WPSLStationInbound = 0;
  let WPSLStationInboundNumVehicles = 0;

  let JSStationOutbound = 0;
  let JSStationOutboundNumVehicles = 0;
  let JSStationInbound = 0;
  let JSStationInboundNumVehicles = 0;

  let OSLStationOutbound = 0;
  let OSLStationOutboundNumVehicles = 0;
  let OSLStationInbound = 0;
  let OSLStationInboundNumVehicles = 0;

  let OceanAptosStationOutbound = 0;
  let OceanAptosStationOutboundNumVehicles = 0;
  let OceanAptosStationInbound = 0;
  let OceanAptosStationInboundNumVehicles = 0;

  // Maximum and minimum average time at station for color scaling
  let maximumAverageTimeAtStation = 80;
  let minimumAverageTimeAtStation = 15;

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
        } else if (stop.stationId === "16995" && stop.timeAtStop > 0) { // Powell Station outbound
          PowellStationOutbound += stop.timeAtStop;
          PowellStationOutboundNumVehicles++;
        } else if (stop.stationId === "16997" && stop.timeAtStop > 0) { // Civic Center Station outbound
          CCStationOutbound += stop.timeAtStop;
          CCStationOutboundNumVehicles++;
        } else if (stop.stationId === "16996" && stop.timeAtStop > 0) { // Van Ness Station outbound
          VNStationOutbound += stop.timeAtStop;
          VNStationOutboundNumVehicles++;
        } else if (stop.stationId === "16998" && stop.timeAtStop > 0) { // Church Station outbound
          ChurchStationOutbound += stop.timeAtStop;
          ChurchStationOutboundNumVehicles++;
        } else if (stop.stationId === "16991" && stop.timeAtStop > 0) { // Castro Station outbound
          CastroStationOutbound += stop.timeAtStop;
          CastroStationOutboundNumVehicles++;
        } else if (stop.stationId === "16993" && stop.timeAtStop > 0) { // Forest Hill Station outbound
          FHStationOutbound += stop.timeAtStop;
          FHStationOutboundNumVehicles++;
        } else if (stop.stationId === "16739" && stop.timeAtStop > 0) { // West Portal Station outbound
          WPStationOutbound += stop.timeAtStop;
          WPStationOutboundNumVehicles++;
        } else if (stop.stationId === "17125" && stop.timeAtStop > 0) { // West Portal & 14th outbound
          WP14StationOutbound += stop.timeAtStop;
          WP14StationOutboundNumVehicles++;
        } else if (stop.stationId === "16503" && stop.timeAtStop > 0) { // West Portal & St. Francis outbound
          WPSLStationOutbound += stop.timeAtStop;
          WPSLStationOutboundNumVehicles++;
        } else if (stop.stationId === "17114" && stop.timeAtStop > 0) { // Junipero Serra Blvd & Ocean Ave outbound
          JSStationOutbound += stop.timeAtStop;
          JSStationOutboundNumVehicles++;
        } else if (stop.stationId === "15807" && stop.timeAtStop > 0) { // Ocean Ave & San Leandro Way outbound
          OSLStationOutbound += stop.timeAtStop;
          OSLStationOutboundNumVehicles++;
        } else if (stop.stationId === "15780" && stop.timeAtStop > 0) { // Ocean Ave & Aptos outbound
          OceanAptosStationOutbound += stop.timeAtStop;
          OceanAptosStationOutboundNumVehicles++;
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
        } else if (stop.stationId === "15417" && stop.timeAtStop > 0) { // Powell Station inbound
          PowellStationInbound += stop.timeAtStop;
          PowellStationInboundNumVehicles++;
        } else if (stop.stationId === "15727" && stop.timeAtStop > 0) { // Civic Center Station inbound
          CCStationInbound += stop.timeAtStop;
          CCStationInboundNumVehicles++;
        } else if (stop.stationId === "15419" && stop.timeAtStop > 0) { // Van Ness Station inbound
          VNStationInbound += stop.timeAtStop;
          VNStationInboundNumVehicles++;
        } else if (stop.stationId === "15726" && stop.timeAtStop > 0) { // Church Station inbound
          ChurchStationInbound += stop.timeAtStop;
          ChurchStationInboundNumVehicles++;
        } else if (stop.stationId === "15728" && stop.timeAtStop > 0) { // Castro Station inbound
          CastroStationInbound += stop.timeAtStop;
          CastroStationInboundNumVehicles++;
        } else if (stop.stationId === "15730" && stop.timeAtStop > 0) { // Forest Hill Station inbound
          FHStationInbound += stop.timeAtStop;
          FHStationInboundNumVehicles++;
        } else if (stop.stationId === "16740" && stop.timeAtStop > 0) { // West Portal Station inbound
          WPStationInbound += stop.timeAtStop;
          WPStationInboundNumVehicles++;
        } else if (stop.stationId === "16898" && stop.timeAtStop > 0) { // West Portal & 14th inbound
          WP14StationInbound += stop.timeAtStop;
          WP14StationInboundNumVehicles++;
        } else if (stop.stationId === "17109" && stop.timeAtStop > 0) { // West Portal & St. Francis inbound
          WPSLStationInbound += stop.timeAtStop;
          WPSLStationInboundNumVehicles++;
        } else if (stop.stationId === "17113" && stop.timeAtStop > 0) { // Junipero Serra Blvd & Ocean Ave inbound
          JSStationInbound += stop.timeAtStop;
          JSStationInboundNumVehicles++;
        } else if (stop.stationId === "15806" && stop.timeAtStop > 0) { // Ocean Ave & San Leandro Way inbound
          OSLStationInbound += stop.timeAtStop;
          OSLStationInboundNumVehicles++;
        } else if (stop.stationId === "15779" && stop.timeAtStop > 0) { // Ocean Ave & Aptos inbound
          OceanAptosStationInbound += stop.timeAtStop;
          OceanAptosStationInboundNumVehicles++;
        }
      } else { // vehicle at intersection

      }
    }
  });

  const outboundCX = 345;
  const inboundCX = 465;
  const stationCYOffset = 200;
  const radiusScaler = 0.015

  // Outbound label
  svg.append('text')
    .attr('x', 50)
    .attr('y', 800)
    .attr('transform', 'rotate(-90, 50, 800)')
    .attr('class', 'outbound-label')
    .text('<-- Outbound <--');

  // Inbound label
  svg.append('text')
    .attr('x', 750)
    .attr('y', 500)
    .attr('transform', 'rotate(90, 750, 500)')
    .attr('class', 'inbound-label')
    .text('<-- Inbound <--');

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
  
  // Powell Station outbound
  let averageTimePowellOutbound = PowellStationOutbound / PowellStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 3)
    .attr('r', PowellStationOutbound * radiusScaler)
    .attr('class', 'powell-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimePowellOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Powell Station (Outbound).<br>Total Time at Station: " + PowellStationOutbound + " seconds<br>Total Vehicles: " + PowellStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimePowellOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 3 + 5)
    .attr('class', 'station-label')
    .text('Powell Station');

  // Powell Station inbound
  let averageTimePowellInbound = PowellStationInbound / PowellStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 3)
    .attr('r', PowellStationInbound * radiusScaler)
    .attr('class', 'powell-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimePowellInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Powell Station (Inbound).<br>Total Time at Station: " + PowellStationInbound + " seconds<br>Total Vehicles: " + PowellStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimePowellInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 3 + 5)
    .attr('class', 'station-label')
    .text('Powell Station');

  // Civic Center Station outbound
  let averageTimeCCOutbound = CCStationOutbound / CCStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 4)
    .attr('r', CCStationOutbound * radiusScaler)
    .attr('class', 'cc-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeCCOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Civic Center Station (Outbound).<br>Total Time at Station: " + CCStationOutbound + " seconds<br>Total Vehicles: " + CCStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeCCOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 4 + 5)
    .attr('class', 'station-label')
    .text('Civic Center Station');

  // Civic Center Station inbound
  let averageTimeCCInbound = CCStationInbound / CCStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 4)
    .attr('r', CCStationInbound * radiusScaler)
    .attr('class', 'cc-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeCCInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Civic Center Station (Inbound).<br>Total Time at Station: " + CCStationInbound + " seconds<br>Total Vehicles: " + CCStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeCCInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 4 + 5)
    .attr('class', 'station-label')
    .text('Civic Center Station');

  // Van Ness Station outbound
  let averageTimeVNOutbound = VNStationOutbound / VNStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 5)
    .attr('r', VNStationOutbound * radiusScaler)
    .attr('class', 'vn-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeVNOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Van Ness Station (Outbound).<br>Total Time at Station: " + VNStationOutbound + " seconds<br>Total Vehicles: " + VNStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeVNOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 5 + 5)
    .attr('class', 'station-label')
    .text('Van Ness Station');

  // Van Ness Station inbound
  let averageTimeVNInbound = VNStationInbound / VNStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 5)
    .attr('r', VNStationInbound * radiusScaler)
    .attr('class', 'vn-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeVNInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Van Ness Station (Inbound).<br>Total Time at Station: " + VNStationInbound + " seconds<br>Total Vehicles: " + VNStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeVNInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 5 + 5)
    .attr('class', 'station-label')
    .text('Van Ness Station');

  // Church Station outbound
  let averageTimeChurchOutbound = ChurchStationOutbound / ChurchStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 6)
    .attr('r', ChurchStationOutbound * radiusScaler)
    .attr('class', 'church-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeChurchOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Church Station (Outbound).<br>Total Time at Station: " + ChurchStationOutbound + " seconds<br>Total Vehicles: " + ChurchStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeChurchOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 6 + 5)
    .attr('class', 'station-label')
    .text('Church Station');

  // Church Station inbound
  let averageTimeChurchInbound = ChurchStationInbound / ChurchStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 6)
    .attr('r', ChurchStationInbound * radiusScaler)
    .attr('class', 'church-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeChurchInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Church Station (Inbound).<br>Total Time at Station: " + ChurchStationInbound + " seconds<br>Total Vehicles: " + ChurchStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeChurchInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 6 + 5)
    .attr('class', 'station-label')
    .text('Church Station');

  // Castro Station outbound
  let averageTimeCastroOutbound = CastroStationOutbound / CastroStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 7)
    .attr('r', CastroStationOutbound * radiusScaler)
    .attr('class', 'castro-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeCastroOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Castro Station (Outbound).<br>Total Time at Station: " + CastroStationOutbound + " seconds<br>Total Vehicles: " + CastroStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeCastroOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 7 + 5)
    .attr('class', 'station-label')
    .text('Castro Station');

  // Castro Station inbound
  let averageTimeCastroInbound = CastroStationInbound / CastroStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 7)
    .attr('r', CastroStationInbound * radiusScaler)
    .attr('class', 'castro-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeCastroInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Castro Station (Inbound).<br>Total Time at Station: " + CastroStationInbound + " seconds<br>Total Vehicles: " + CastroStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeCastroInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 7 + 5)
    .attr('class', 'station-label')
    .text('Castro Station');

  // Forest Hill Station outbound
  let averageTimeFHOutbound = FHStationOutbound / FHStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 8)
    .attr('r', FHStationOutbound * radiusScaler)
    .attr('class', 'fh-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeFHOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Forest Hill Station (Outbound).<br>Total Time at Station: " + FHStationOutbound + " seconds<br>Total Vehicles: " + FHStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeFHOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 8 + 5)
    .attr('class', 'station-label')
    .text('Forest Hill Station');

  // Forest Hill Station inbound
  let averageTimeFHInbound = FHStationInbound / FHStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 8)
    .attr('r', FHStationInbound * radiusScaler)
    .attr('class', 'fh-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeFHInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Forest Hill Station (Inbound).<br>Total Time at Station: " + FHStationInbound + " seconds<br>Total Vehicles: " + FHStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeFHInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 8 + 5)
    .attr('class', 'station-label')
    .text('Forest Hill Station');

  // West Portal Station outbound
  let averageTimeWPOutbound = WPStationOutbound / WPStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 9)
    .attr('r', WPStationOutbound * radiusScaler)
    .attr('class', 'wp-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeWPOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("West Portal Station (Outbound).<br>Total Time at Station: " + WPStationOutbound + " seconds<br>Total Vehicles: " + WPStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeWPOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 9 + 5)
    .attr('class', 'station-label')
    .text('West Portal Station');

  // West Portal Station inbound
  let averageTimeWPInbound = WPStationInbound / WPStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 9)
    .attr('r', WPStationInbound * radiusScaler)
    .attr('class', 'wp-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeWPInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("West Portal Station (Inbound).<br>Total Time at Station: " + WPStationInbound + " seconds<br>Total Vehicles: " + WPStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeWPInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 9 + 5)
    .attr('class', 'station-label')
    .text('West Portal Station');

  // West Portal & 14th Station outbound
  let averageTimeWP14Outbound = WP14StationOutbound / WP14StationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 10)
    .attr('r', WP14StationOutbound * radiusScaler)
    .attr('class', 'wp14-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeWP14Outbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("West Portal & 14th Station (Outbound).<br>Total Time at Station: " + WP14StationOutbound + " seconds<br>Total Vehicles: " + WP14StationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeWP14Outbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 10 + 5)
    .attr('class', 'station-label')
    .text('West Portal & 14th Station');

  // West Portal & 14th Station inbound
  let averageTimeWP14Inbound = WP14StationInbound / WP14StationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 10)
    .attr('r', WP14StationInbound * radiusScaler)
    .attr('class', 'wp14-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeWP14Inbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1) 
        .html("West Portal & 14th Station (Inbound).<br>Total Time at Station: " + WP14StationInbound + " seconds<br>Total Vehicles: " + WP14StationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeWP14Inbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 10 + 5)
    .attr('class', 'station-label')
    .text('West Portal & 14th Station');

  // West Portal & Sloat Blvd (Saint Francis Cir) Station outbound
  let averageTimeWPSLOutbound = WPSLStationOutbound / WPSLStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 11)
    .attr('r', WPSLStationOutbound * radiusScaler)
    .attr('class', 'wpsl-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeWPSLOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("West Portal & Sloat Blvd (Saint Francis Cir) Station (Outbound).<br>Total Time at Station: " + WPSLStationOutbound + " seconds<br>Total Vehicles: " + WPSLStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeWPSLOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 11 + 5)
    .attr('class', 'station-label')
    .text('West Portal & Sloat Blvd (Saint Francis Cir) Station');

  // West Portal & Sloat Blvd (Saint Francis Cir) Station inbound
  let averageTimeWPSLInbound = WPSLStationInbound / WPSLStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 11)
    .attr('r', WPSLStationInbound * radiusScaler)
    .attr('class', 'wpsl-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeWPSLInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("West Portal & Sloat Blvd (Saint Francis Cir) Station (Inbound).<br>Total Time at Station: " + WPSLStationInbound + " seconds<br>Total Vehicles: " + WPSLStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeWPSLInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 11 + 5)
    .attr('class', 'station-label')
    .text('West Portal & Sloat Blvd (Saint Francis Cir) Station');

  // Junipero Serra Blvd & Ocean Ave Station outbound
  let averageTimeJSOutbound = JSStationOutbound / JSStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 12)
    .attr('r', JSStationOutbound * radiusScaler)
    .attr('class', 'js-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeJSOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Junipero Serra Blvd & Ocean Ave Station (Outbound).<br>Total Time at Station: " + JSStationOutbound + " seconds<br>Total Vehicles: " + JSStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeJSOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 12 + 5)
    .attr('class', 'station-label')
    .text('Junipero Serra Blvd & Ocean Ave Station'); 

  // Junipero Serra Blvd & Ocean Ave Station inbound
  let averageTimeJSInbound = JSStationInbound / JSStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 12)
    .attr('r', JSStationInbound * radiusScaler)
    .attr('class', 'js-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeJSInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Junipero Serra Blvd & Ocean Ave Station (Inbound).<br>Total Time at Station: " + JSStationInbound + " seconds<br>Total Vehicles: " + JSStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeJSInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 12 + 5)
    .attr('class', 'station-label')
    .text('Junipero Serra Blvd & Ocean Ave Station');

  // Ocean Ave & San Leandro Way Station outbound
  let averageTimeOSLOutbound = OSLStationOutbound / OSLStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 13)
    .attr('r', OSLStationOutbound * radiusScaler)
    .attr('class', 'osl-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeOSLOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Ocean Ave & San Leandro Way Station (Outbound).<br>Total Time at Station: " + OSLStationOutbound + " seconds<br>Total Vehicles: " + OSLStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeOSLOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 13 + 5)
    .attr('class', 'station-label')
    .text('Ocean Ave & San Leandro Way Station');

  // Ocean Ave & San Leandro Way Station inbound
  let averageTimeOSLInbound = OSLStationInbound / OSLStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 13)
    .attr('r', OSLStationInbound * radiusScaler)
    .attr('class', 'osl-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeOSLInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Ocean Ave & San Leandro Way Station (Inbound).<br>Total Time at Station: " + OSLStationInbound + " seconds<br>Total Vehicles: " + OSLStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeOSLInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 13 + 5)
    .attr('class', 'station-label')
    .text('Ocean Ave & San Leandro Way Station');

  // Ocean Ave & Aptos Ave Station outbound
  let averageTimeOceanAptosOutbound = OceanAptosStationOutbound / OceanAptosStationOutboundNumVehicles;
  svg.append('circle')
    .attr('cx', outboundCX)
    .attr('cy', stationCYOffset * 14)
    .attr('r', OceanAptosStationOutbound * radiusScaler)
    .attr('class', 'ocean-aptos-outbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeOceanAptosOutbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Ocean Ave & Aptos Ave Station (Outbound).<br>Total Time at Station: " + OceanAptosStationOutbound + " seconds<br>Total Vehicles: " + OceanAptosStationOutboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeOceanAptosOutbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 14 + 5)
    .attr('class', 'station-label')
    .text('Ocean Ave & Aptos Ave Station');

  // Ocean Ave & Aptos Ave Station inbound
  let averageTimeOceanAptosInbound = OceanAptosStationInbound / OceanAptosStationInboundNumVehicles;
  svg.append('circle')
    .attr('cx', inboundCX)
    .attr('cy', stationCYOffset * 14)
    .attr('r', OceanAptosStationInbound * radiusScaler)
    .attr('class', 'ocean-aptos-inbound station-circle')
    .attr('fill', d3.interpolateRdYlGn(1 - (averageTimeOceanAptosInbound / maximumAverageTimeAtStation)))
    .on("mouseover", (event, d) => {
      tooltip.style("opacity", 1)
        .html("Ocean Ave & Aptos Ave Station (Inbound).<br>Total Time at Station: " + OceanAptosStationInbound + " seconds<br>Total Vehicles: " + OceanAptosStationInboundNumVehicles + " vehicles<br>Average Time at Station: " + averageTimeOceanAptosInbound.toFixed(2) + " seconds");
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
    .attr('y', stationCYOffset * 14 + 5)
    .attr('class', 'station-label')
    .text('Ocean Ave & Aptos Ave Station');
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