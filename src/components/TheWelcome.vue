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
    makeLabels();
  } catch (error) {
    console.error('Error loading JSON files:', error);
  }
})

function runAfterLoad() {
  // svg canvas dimensions
  const width = 2200;
  const height = 750;

  const outboundCY = 250;
  const inboundCY = 450;

  const svg = d3.select(chart.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  const tooltip = d3.select("#tooltip");

  // Background for underground section of line
  svg.append('rect')
    .attr('x', 900)
    .attr('y', 0)
    .attr('width', width - 900)
    .attr('height', height)
    .attr('fill', 'darkgray');
  
  // Line for K Line outbound
  svg.append('rect')
    .attr('x', 0)
    .attr('y', outboundCY)
    .attr('width', width)
    .attr('height', 10)
    .attr('fill', 'steelblue');

  // Line for K Line inbound
  svg.append('rect')
    .attr('x', 0)
    .attr('y', inboundCY)
    .attr('width', width)
    .attr('height', 10)
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

  let OceanVictoriaStationOutbound = 0;
  let OceanVictoriaStationOutboundNumVehicles = 0;
  let OceanFairfieldStationInbound = 0;
  let OceanFairfieldStationInboundNumVehicles = 0;

  let OceanJulesStationOutbound = 0;
  let OceanJulesStationOutboundNumVehicles = 0;
  let OceanDoradoStationInbound = 0;
  let OceanDoradoStationInboundNumVehicles = 0;

  let OceanMiramarStationOutbound = 0;
  let OceanMiramarStationOutboundNumVehicles = 0;
  let OceanMiramarStationInbound = 0;
  let OceanMiramarStationInboundNumVehicles = 0;

  let OceanLeeStationOutbound = 0;
  let OceanLeeStationOutboundNumVehicles = 0;
  let OceanLeeStationInbound = 0;
  let OceanLeeStationInboundNumVehicles = 0;

  let OceanCCSFStationOutbound = 0;
  let OceanCCSFStationOutboundNumVehicles = 0;
  let OceanCCSFStationInbound = 0;
  let OceanCCSFStationInboundNumVehicles = 0;

  let BalboaParkStationOutbound = 0;
  let BalboaParkStationOutboundNumVehicles = 0;
  let SanJoseGenevaStationInbound = 0;
  let SanJoseGenevaStationInboundNumVehicles = 0;

  let EmbarcaderoMissionIntersectionOutbound = 0;
  let EmbarcaderoMissionIntersectionOutboundNumVehicles = 0;
  let EmbarcaderoMissionIntersectionInbound = 0;
  let EmbarcaderoMissionIntersectionInboundNumVehicles = 0;

  let EmbarcaderoHowardIntersectionOutbound = 0;
  let EmbarcaderoHowardIntersectionOutboundNumVehicles = 0;
  let EmbarcaderoHowardIntersectionInbound = 0;
  let EmbarcaderoHowardIntersectionInboundNumVehicles = 0;

  // Maximum and minimum average time at station for color scaling
  let maximumAverageTimeAtStation = 80;
  let minimumAverageTimeAtStation = 0;

  // Cycle through the data
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
        } else if (stop.stationId === "16503" && stop.timeAtStop > 0) { // West Portal & Sloat Blvd (Saint Francis Cir) outbound
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
        } else if (stop.stationId === "15808" && stop.timeAtStop > 0) { // Ocean Ave & Victoria St outbound
          OceanVictoriaStationOutbound += stop.timeAtStop;
          OceanVictoriaStationOutboundNumVehicles++;
        } else if (stop.stationId === "15793" && stop.timeAtStop > 0) { // Ocean Ave & Jules outbound
          OceanJulesStationOutbound += stop.timeAtStop;
          OceanJulesStationOutboundNumVehicles++;
        } else if (stop.stationId === "15798" && stop.timeAtStop > 0) { // Ocean Ave & Fairfield Way outbound
          OceanMiramarStationOutbound += stop.timeAtStop;
          OceanMiramarStationOutboundNumVehicles++;
        } else if (stop.stationId === "15795" && stop.timeAtStop > 0) { // Ocean Ave & Dorado Ter outbound
          OceanLeeStationOutbound += stop.timeAtStop;
          OceanLeeStationOutboundNumVehicles++;
        } else if (stop.stationId === "15785" && stop.timeAtStop > 0) { // Ocean Ave & Dorado Ter outbound
          OceanCCSFStationOutbound += stop.timeAtStop;
          OceanCCSFStationOutboundNumVehicles++;
        } else if (stop.stationId === "15418" && stop.timeAtStop > 0) { // Ocean Ave & Dorado Ter outbound
          BalboaParkStationOutbound += stop.timeAtStop;
          BalboaParkStationOutboundNumVehicles++;
        }
      } else if (stop.atIntersection) { // vehicle at intersection
        if (stop.intersectionCrossStreet === "Embarcadero & Mission") {
          EmbarcaderoMissionIntersectionOutbound += stop.timeAtStop;
          EmbarcaderoMissionIntersectionOutboundNumVehicles++;
        } else if (stop.intersectionCrossStreet === "Embarcadero & Howard St") {
          console.log(stop);
          EmbarcaderoHowardIntersectionOutbound += stop.timeAtStop;
          EmbarcaderoHowardIntersectionOutboundNumVehicles++;
        } else {
          console.log(stop.intersectionCrossStreet);
        }
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
        } else if (stop.stationId === "17109" && stop.timeAtStop > 0) { // West Portal & Sloat Blvd (Saint Francis Cir) inbound
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
        } else if (stop.stationId === "15788" && stop.timeAtStop > 0) { // Ocean Ave & Fairfield Way inbound
          OceanFairfieldStationInbound += stop.timeAtStop;
          OceanFairfieldStationInboundNumVehicles++;
        } else if (stop.stationId === "15787" && stop.timeAtStop > 0) { // Ocean Ave & Jules inbound
          OceanDoradoStationInbound += stop.timeAtStop;
          OceanDoradoStationInboundNumVehicles++;
        } else if (stop.stationId === "15797" && stop.timeAtStop > 0) { // Ocean Ave & Dorado Ter inbound
          OceanMiramarStationInbound += stop.timeAtStop;
          OceanMiramarStationInboundNumVehicles++;
        } else if (stop.stationId === "15794" && stop.timeAtStop > 0) { // Ocean Ave & Miramar Ave inbound
          OceanLeeStationInbound += stop.timeAtStop;
          OceanLeeStationInboundNumVehicles++;
        } else if (stop.stationId === "15784" && stop.timeAtStop > 0) { // Ocean Ave & Miramar Ave inbound
          OceanCCSFStationInbound += stop.timeAtStop;
          OceanCCSFStationInboundNumVehicles++;
        } else if (stop.stationId === "17778" && stop.timeAtStop > 0) { // Ocean Ave & Miramar Ave inbound
          SanJoseGenevaStationInbound += stop.timeAtStop;
          SanJoseGenevaStationInboundNumVehicles++;
        }
      } else if (stop.atIntersection) { // vehicle at intersection
        if (stop.intersectionCrossStreet === "Embarcadero & Mission") {
          EmbarcaderoMissionIntersectionInbound += stop.timeAtStop;
          EmbarcaderoMissionIntersectionInboundNumVehicles++;
        } else if (stop.intersectionCrossStreet === "Embarcadero & Howard St") {
          console.log(stop);
          EmbarcaderoHowardIntersectionInbound += stop.timeAtStop;
          EmbarcaderoHowardIntersectionInboundNumVehicles++;
        } else {
          console.log(stop.intersectionCrossStreet);
        }
      }
    }
  });

  // Make array of station data
  const stationsData = [
    {
      name: 'Embarcadero Station',
      totalTime: EmbarcaderoStationOutbound,
      numVehicles: EmbarcaderoStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Embarcadero Station',
      totalTime: EmbarcaderoStationInbound,
      numVehicles: EmbarcaderoStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Montgomery Station',
      totalTime: MontgomeryStationOutbound,
      numVehicles: MontgomeryStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Montgomery Station',
      totalTime: MontgomeryStationInbound,
      numVehicles: MontgomeryStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Powell Station',
      totalTime: PowellStationOutbound,
      numVehicles: PowellStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Powell Station',
      totalTime: PowellStationInbound,
      numVehicles: PowellStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Civic Center Station',
      totalTime: CCStationOutbound,
      numVehicles: CCStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Civic Center Station',
      totalTime: CCStationInbound,
      numVehicles: CCStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Van Ness Station',
      totalTime: VNStationOutbound,
      numVehicles: VNStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Van Ness Station',
      totalTime: VNStationInbound,
      numVehicles: VNStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Church Station',
      totalTime: ChurchStationOutbound,
      numVehicles: ChurchStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Church Station',
      totalTime: ChurchStationInbound,
      numVehicles: ChurchStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Castro Station',
      totalTime: CastroStationOutbound,
      numVehicles: CastroStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Castro Station',
      totalTime: CastroStationInbound,
      numVehicles: CastroStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Forest Hill Station',
      totalTime: FHStationOutbound,
      numVehicles: FHStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Forest Hill Station',
      totalTime: FHStationInbound,
      numVehicles: FHStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'West Portal Station',
      totalTime: WPStationOutbound,
      numVehicles: WPStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'West Portal Station',
      totalTime: WPStationInbound,
      numVehicles: WPStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'West Portal & 14th',
      totalTime: WP14StationOutbound,
      numVehicles: WP14StationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'West Portal & 14th',
      totalTime: WP14StationInbound,
      numVehicles: WP14StationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'West Portal & Sloat Blvd (Saint Francis Cir)',
      totalTime: WPSLStationOutbound,
      numVehicles: WPSLStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'West Portal & Sloat Blvd (Saint Francis Cir)',
      totalTime: WPSLStationInbound,
      numVehicles: WPSLStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Junipero Serra Blvd & Ocean Ave',
      totalTime: JSStationOutbound,
      numVehicles: JSStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Junipero Serra Blvd & Ocean Ave',
      totalTime: JSStationInbound,
      numVehicles: JSStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & San Leandro Way',
      totalTime: OSLStationOutbound,
      numVehicles: OSLStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & San Leandro Way',
      totalTime: OSLStationInbound,
      numVehicles: OSLStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Aptos',
      totalTime: OceanAptosStationOutbound,
      numVehicles: OceanAptosStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Aptos',
      totalTime: OceanAptosStationInbound,
      numVehicles: OceanAptosStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Victoria St',
      totalTime: OceanVictoriaStationOutbound,
      numVehicles: OceanVictoriaStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Fairfield Way',
      totalTime: OceanFairfieldStationInbound,
      numVehicles: OceanFairfieldStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Jules Ave',
      totalTime: OceanJulesStationOutbound,
      numVehicles: OceanJulesStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Dorado Ter',
      totalTime: OceanDoradoStationInbound,
      numVehicles: OceanDoradoStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Miramar Ave',
      totalTime: OceanMiramarStationOutbound,
      numVehicles: OceanMiramarStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Miramar Ave',
      totalTime: OceanMiramarStationInbound,
      numVehicles: OceanMiramarStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Lee Ave',
      totalTime: OceanLeeStationOutbound,
      numVehicles: OceanLeeStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Lee Ave',
      totalTime: OceanLeeStationInbound,
      numVehicles: OceanLeeStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & CCSF Pedestrian Bridge',
      totalTime: OceanCCSFStationOutbound,
      numVehicles: OceanCCSFStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & CCSF Pedestrian Bridge',
      totalTime: OceanCCSFStationInbound,
      numVehicles: OceanCCSFStationInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Balboa Park BART Mezzanine Level Station',
      totalTime: BalboaParkStationOutbound,
      numVehicles: BalboaParkStationOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'San Jose Ave & Geneva Ave Station',
      totalTime: SanJoseGenevaStationInbound,
      numVehicles: SanJoseGenevaStationInboundNumVehicles,
      direction: 'inbound'
    }
  ];

  stationsData.reverse(); // Reverse the order of the stations to match the SVG layout

  // Make array of intersection data
  const intersectionsData = [
    {
      name: 'Montgomery Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Montgomery Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Powell Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Powell Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Civic Center Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Civic Center Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Van Ness Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Van Ness Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Church Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Church Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Castro Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Castro Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Forest Hill Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Forest Hill Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'West Portal Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'West Portal Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'West Portal & 14th',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'West Portal & 14th',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'West Portal & Sloat Blvd (Saint Francis Cir)',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'West Portal & Sloat Blvd (Saint Francis Cir)',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Junipero Serra Blvd & Ocean Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Junipero Serra Blvd & Ocean Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & San Leandro Way',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & San Leandro Way',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Aptos',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Aptos',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Victoria St',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Fairfield Way',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Jules Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Dorado Ter',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Miramar Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Miramar Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & Lee Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & Lee Ave',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Ocean Ave & CCSF Pedestrian Bridge',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'Ocean Ave & CCSF Pedestrian Bridge',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Balboa Park BART Mezzanine Level Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'outbound'
    },
    {
      name: 'San Jose Ave & Geneva Ave Station',
      totalTime: 1,
      numVehicles: 1,
      direction: 'inbound'
    },
    {
      name: 'Embarcadero & Mission Intersection',
      totalTime: EmbarcaderoMissionIntersectionOutbound,
      numVehicles: EmbarcaderoMissionIntersectionOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Embarcadero & Mission Intersection',
      totalTime: EmbarcaderoMissionIntersectionInbound,
      numVehicles: EmbarcaderoMissionIntersectionInboundNumVehicles,
      direction: 'inbound'
    },
    {
      name: 'Embarcadero & Howard Intersection',
      totalTime: EmbarcaderoHowardIntersectionOutbound,
      numVehicles: EmbarcaderoHowardIntersectionOutboundNumVehicles,
      direction: 'outbound'
    },
    {
      name: 'Embarcadero & Howard Intersection',
      totalTime: EmbarcaderoHowardIntersectionInbound,
      numVehicles: EmbarcaderoHowardIntersectionInboundNumVehicles,
      direction: 'inbound'
    }
  ];

  // Outbound labels
  svg.append('text')
    .attr('x', 770)
    .attr('y', 40)
    .attr('class', 'outbound-label label')
    .text('<-- Outbound <--');

  // Inbound label
  svg.append('text')
    .attr('x', 770)
    .attr('y', height - 20)
    .attr('class', 'inbound-label label')
    .text('--> Inbound -->');

  // Loop through stations data to create bars and labels
  const heightScalar = 0.03; // Scale the height of the bars based on average time at station
  stationsData.forEach((station, index) => {
    const averageTime = station.totalTime / station.numVehicles;
    const isOutbound = station.direction === 'outbound';
    const offsetX = 40;
    const cy = isOutbound ? outboundCY : inboundCY;
    let cx;
    if (index % 2 > 0) {
      cx = ((index - 1) * offsetX) + 20; // Adjust horizontal position for each station
    } else {
      cx = (index * offsetX) + 20; // Adjust horizontal position for each station
    }
    const height = averageTime;

    svg.append('rect')
      .attr('x', cx)
      .attr('y', isOutbound ? cy - 2 - height : cy + 12)
      .attr('width', offsetX - 10)
      .attr('height', height)
      .attr('class', `${station.name.toLowerCase().replace(/ /g, '-')}-rect station-rect`)
      .attr('fill', 'steelblue')
      .on("mouseover", (event, d) => {
        tooltip.style("opacity", 1)
          .html(`${station.name}.<br>Total Time at Station: ${station.totalTime} seconds<br>Total Vehicles: ${station.numVehicles} vehicles<br>Average Time at Station: ${averageTime.toFixed(2)} seconds`);
      })
      .on("mousemove", (event) => {
        tooltip.style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 20) + "px");
      })
      .on("mouseout", () => {
        tooltip.style("opacity", 0);
      });

    svg.append('text')
      .attr('x', cx)
      .attr('y', isOutbound ? cy + 20 : cy - 70)
      .attr('class', 'station-label label')
      .text(station.name)
      .call(wrapText, 40);
  });

  // Loop through intersections data to create bars and labels
  intersectionsData.forEach((intersection, index) => {
    const averageTime = intersection.totalTime / intersection.numVehicles;
    const isOutbound = intersection.direction === 'outbound';
    const offsetX = 40;
    const cy = isOutbound ? outboundCY : inboundCY;
    let cx;
    if (index % 2 > 0) {
      cx = ((index - 1) * offsetX) + 20; // Adjust horizontal position for each intersection
    } else {
      cx = (index * offsetX) + 20; // Adjust horizontal position for each intersection
    }
    const height = averageTime;
    
    svg.append('rect')
      .attr('x', cx + 40)
      .attr('y', isOutbound ? cy - 2 - height : cy + 12)
      .attr('width', offsetX - 10)
      .attr('height', height)
      .attr('class', `${intersection.name.toLowerCase().replace(/ /g, '-')}-rect intersection-rect`)
      .attr('fill', 'steelblue')
      .on("mouseover", (event, d) => {
        tooltip.style("opacity", 1)
          .html(`${intersection.name}.<br>Total Time at Intersection: ${intersection.totalTime} seconds<br>Total Vehicles: ${intersection.numVehicles} vehicles<br>Average Time at Intersection: ${averageTime.toFixed(2)} seconds`);
      })
      .on("mousemove", (event) => {
        tooltip.style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 20) + "px");
      })
      .on("mouseout", () => {
        tooltip.style("opacity", 0);
      });
    
    svg.append('text')
      .attr('x', cx + 40)
      .attr('y', isOutbound ? cy + 20 : cy - 70)
      .attr('class', 'station-label label')
      .text(intersection.name)
      .call(wrapText, 40);
  });
}

function makeLabels() {
  function makeBG(elem) {
    var svgns = "http://www.w3.org/2000/svg"
    var bounds = elem.getBBox()
    var bg = document.createElementNS(svgns, "rect")
    var style = getComputedStyle(elem)
    var padding_top = parseInt(style["padding-top"])
    var padding_left = parseInt(style["padding-left"])
    var padding_right = parseInt(style["padding-right"])
    var padding_bottom = parseInt(style["padding-bottom"])
    bg.setAttribute("x", bounds.x - parseInt(style["padding-left"]))
    bg.setAttribute("y", bounds.y - parseInt(style["padding-top"]))
    bg.setAttribute("width", bounds.width + padding_left + padding_right)
    bg.setAttribute("height", bounds.height + padding_top + padding_bottom)
    bg.setAttribute("fill", style["background-color"])
    bg.setAttribute("rx", style["border-radius"])
    bg.setAttribute("stroke-width", style["border-top-width"])
    bg.setAttribute("stroke", style["border-top-color"])
    if (elem.hasAttribute("transform")) {
      bg.setAttribute("transform", elem.getAttribute("transform"))
    }
    elem.parentNode.insertBefore(bg, elem)
  }

  var texts = document.querySelectorAll(".label")
  for (var i = 0; i < texts.length; i++) {
    makeBG(texts[i])
  }
}

function wrapText(textSelection, width) {
  textSelection.each(function() {
    const text = d3.select(this);
    const words = text.text().split(/\s+/).reverse();
    let word;
    let line = [];
    let lineNumber = 0;
    const lineHeight = 1.1; // ems
    const y = text.attr("y");
    const dy = parseFloat(text.attr("dy")) || 0;
    let tspan = text.text(null)
      .append("tspan")
      .attr("x", text.attr("x"))
      .attr("y", y)
      .attr("dy", dy + "em");

    let lines = 1;
    while (word = words.pop()) {
      line.push(word);
      tspan.text(line.join(" "));
      if (tspan.node().getComputedTextLength() > width) {
        line.pop();
        tspan.text(line.join(" "));
        line = [word];
        tspan = text.append("tspan")
          .attr("x", text.attr("x"))
          .attr("y", y)
          .attr("dy", ++lineNumber * lineHeight + dy + "em")
          .text(word);
          lines++;
      }
    }
  });
}

</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100..900;1,100..900&display=swap');

.page-title {
  font-family: "Roboto", sans-serif;
  text-align: center;
  font-size: 2rem;
  color: #010101;
  text-decoration: underline;
  font-weight: 700;
}

.svg-area {
  display: flex;
  justify-content: center;
  align-items: center;
  height: fit-content;
  border: 1px solid black;
}

.station-rect {
  stroke: #fff;
  stroke-width: 3px;
}

.intersection-rect {
  stroke: #010101;
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
  font-size: 16px;
  font-weight: 400;
  background-color: #010101;
  padding: 5px;
  border-radius: 2px;
}

.label {
  background: #010101;
  border-radius: 3px;
  border: 1px solid #C3BFBA;
  padding: 4px 12px;
}

.station-label.label {
  font-family: "Roboto", sans-serif;
  fill: #C3BFBA;
  font-size: 10px;
  font-weight: 400;
  padding: 2px;
}
</style>