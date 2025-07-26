<template>
  <div class="welcome">
    <h1 class="page-title">Tenco CityScale K Line Intersection Delays</h1>
    <div class="time-filter-controls">
      <label class="date-selector-label" for="timeFilter">Select Time:</label>
      <select id="timeFilter" v-model="selectedTime" @change="changeData(selectedTime)">
        <option value="combined">All Dates</option>
        <option value="05-16">May 16</option>
        <option value="05-15">May 15</option>
        <option value="05-14">May 14</option>
        <option value="05-13">May 13</option>
        <option value="05-12">May 12</option>
        <option value="05-11">May 11</option>
      </select>
    </div>
    <div class="hour-filter-controls">
      <label class="hour-selector-label" for="hourRangeFilter">Hour Range:</label>
      <select id="hourRangeFilter" v-model="selectedHourRange" @change="filterByHourRange(selectedHourRange)">
        <option value="all">All Hours</option>
        <option value="7-10">7:00 AM PDT - 10:00 AM PDT</option>
        <option value="10-15">10:00 AM PDT - 3:00 PM PDT</option>
        <option value="15-19">3:00 PM PDT - 7:00 PM PDT</option>
        <option value="19-21">7:00 PM PDT - 9:00 PM PDT</option>
      </select>
    </div>
    <div ref="chart" class="svg-area"></div>
    <div class="tooltip" id="tooltip"></div>
    <div class="legend-container">
      <div class="legend-content">
        <div class="legend-header">Rectangle:</div>
        <div class="legend-grid-container">
          <div class="legend-item">
            <span class="legend-color intersection-color"></span> Intersection
          </div>
          <div class="legend-item">
            <span class="legend-color station-color"></span> Station
          </div>
          <div class="legend-item">
            <span class="legend-color terminal-station-color"></span> Terminal Station
          </div>
        </div>
        <div class="terminal-note">Note: Train sits at terminal stations/intersections between trips for operator change, schedule, bathroom break, etc.</div>
      </div>
    </div>
  </div>  
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';

const chart = ref(null);
const data1 = ref(null);
const data2 = ref(null);
const data3 = ref(null);
const data4 = ref(null);
const data5 = ref(null);
const data6 = ref(null);
const fullTripData1 = ref(null);
const fullTripData2 = ref(null);
const fullTripData3 = ref(null);
const fullTripData4 = ref(null);
const fullTripData5 = ref(null);
const fullTripData6 = ref(null);
const combinedData = ref(null);
const fullTripCombinedData = ref(null);
const selectedTime = ref('combined'); // Default selected value
const selectedHourRange = ref('all'); // Default to 'all'

let startHour = "all";
let endHour = "all";
let dataToBeRendered; // Default to all data
let fullTripDataToBeRendered; // Default to all data

onMounted(async () => {
  try {
    const [file1, file2, file3, file4, file5, file6] = await Promise.all([
      import('../data/stopData-Curated-05-16.json'),
      import('../data/stopData-Curated-05-15.json'),
      import('../data/stopData-Curated-05-14.json'),
      import('../data/stopData-Curated-05-13.json'),
      import('../data/stopData-Curated-05-12.json'),
      import('../data/stopData-Curated-05-11.json')
    ]);

    const [fullTripFile1, fullTripFile2, fullTripFile3, fullTripFile4, fullTripFile5, fullTripFile6] = await Promise.all([
      import('../data/fullTrip-Curated-05-16.json'),
      import('../data/fullTrip-Curated-05-15.json'),
      import('../data/fullTrip-Curated-05-14.json'),
      import('../data/fullTrip-Curated-05-13.json'),
      import('../data/fullTrip-Curated-05-12.json'),
      import('../data/fullTrip-Curated-05-11.json')
    ]);

    data1.value = file1.default;
    data2.value = file2.default;
    data3.value = file3.default;
    data4.value = file4.default;
    data5.value = file5.default;
    data6.value = file6.default;

    fullTripData1.value = fullTripFile1.default;
    fullTripData2.value = fullTripFile2.default;
    fullTripData3.value = fullTripFile3.default;
    fullTripData4.value = fullTripFile4.default;
    fullTripData5.value = fullTripFile5.default;
    fullTripData6.value = fullTripFile6.default;

    // Combine all data into one array
    combinedData.value = [
      ...Object.values(data1.value),
      ...Object.values(data2.value),
      ...Object.values(data3.value),
      ...Object.values(data4.value),
      ...Object.values(data5.value),
      ...Object.values(data6.value)
    ];

    // Combine full trip data into one array
    fullTripCombinedData.value = [
      ...Object.values(fullTripData1.value),
      ...Object.values(fullTripData2.value),
      ...Object.values(fullTripData3.value),
      ...Object.values(fullTripData4.value),
      ...Object.values(fullTripData5.value),
      ...Object.values(fullTripData6.value)
    ];

    //console.log(data1.value);
    dataToBeRendered = combinedData.value;
    fullTripDataToBeRendered = fullTripCombinedData.value;
    runAfterLoad(dataToBeRendered, startHour, endHour, fullTripDataToBeRendered);
  } catch (error) {
    console.error('Error loading JSON files:', error);
  }
})

function runAfterLoad(dataFile, startHourFilter, endHourFilter, fullTripData) {
  console.log("start of runAfterLoad console");

  // svg canvas dimensions
  const width = 805;
  const height = 750;

  const outboundCY = 220;
  const inboundCY = 480;

  const svg = d3.select(chart.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  const tooltip = d3.select("#tooltip");

  // Background for underground section of line
  svg.append('rect')
    .attr('x', 900)
    .attr('y', 0)
    .attr('width', width)
    .attr('height', height)
    .attr('class', 'underground')
    .attr('fill', 'darkgray');
  
  // Line for K Line outbound
  svg.append('rect')
    .attr('x', 0)
    .attr('y', outboundCY - 1)
    .attr('width', width)
    .attr('height', 10)
    .attr('fill', 'steelblue');

  // Line for K Line inbound
  svg.append('rect')
    .attr('x', 0)
    .attr('y', inboundCY + 1)
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

  let WPDeweyIntersectionOutbound = 0;
  let WPDeweyIntersectionOutboundNumVehicles = 0;
  let WPDeweyIntersectionInbound = 0;
  let WPDeweyIntersectionInboundNumVehicles = 0;

  let JSMontereyIntersectionOutbound = 0;
  let JSMontereyIntersectionOutboundNumVehicles = 0;
  let JSMontereyIntersectionInbound = 0;
  let JSMontereyIntersectionInboundNumVehicles = 0;

  let MarketSteuartIntersectionOutbound = 0;
  let MarketSteuartIntersectionOutboundNumVehicles = 0;
  let MarketSteuartIntersectionInbound = 0;
  let MarketSteuartIntersectionInboundNumVehicles = 0;

  let WPVicenteIntersectionOutbound = 0;
  let WPVicenteIntersectionOutboundNumVehicles = 0;
  let WPVicenteIntersectionInbound = 0;
  let WPVicenteIntersectionInboundNumVehicles = 0;

  let MarketEurekaIntersectionOutbound = 0;
  let MarketEurekaIntersectionOutboundNumVehicles = 0;
  let MarketEurekaIntersectionInbound = 0;
  let MarketEurekaIntersectionInboundNumVehicles = 0;

  let MarketDuboceIntersectionOutbound = 0;
  let MarketDuboceIntersectionOutboundNumVehicles = 0;
  let MarketDuboceIntersectionInbound = 0;
  let MarketDuboceIntersectionInboundNumVehicles = 0;

  let MarketFranklinIntersectionOutbound = 0;
  let MarketFranklinIntersectionOutboundNumVehicles = 0;
  let MarketFranklinIntersectionInbound = 0;
  let MarketFranklinIntersectionInboundNumVehicles = 0;

  let HowlthOceanIntersectionOutbound = 0;
  let HowlthOceanIntersectionOutboundNumVehicles = 0;
  let HowlthOceanIntersectionInbound = 0;
  let HowlthOceanIntersectionInboundNumVehicles = 0;

  let MarketGGIntersectionOutbound = 0;
  let MarketGGIntersectionOutboundNumVehicles = 0;
  let MarketGGIntersectionInbound = 0;
  let MarketGGIntersectionInboundNumVehicles = 0;

  let HowlthBalboaParkIntersectionOutbound = 0;
  let HowlthBalboaParkIntersectionOutboundNumVehicles = 0;
  let HowlthBalboaParkIntersectionInbound = 0;
  let HowlthBalboaParkIntersectionInboundNumVehicles = 0;

  let OceanPlymouthIntersectionOutbound = 0;
  let OceanPlymouthIntersectionOutboundNumVehicles = 0;
  let OceanPlymouthIntersectionInbound = 0;
  let OceanPlymouthIntersectionInboundNumVehicles = 0;

  let OceanCerritosIntersectionOutbound = 0;
  let OceanCerritosIntersectionOutboundNumVehicles = 0;
  let OceanCerritosIntersectionInbound = 0;
  let OceanCerritosIntersectionInboundNumVehicles = 0;

  let WP15IntersectionOutbound = 0;
  let WP15IntersectionOutboundNumVehicles = 0;
  let WP15IntersectionInbound = 0;
  let WP15IntersectionInboundNumVehicles = 0;

  let MarketBatteryIntersectionOutbound = 0;
  let MarketBatteryIntersectionOutboundNumVehicles = 0;
  let MarketBatteryIntersectionInbound = 0;
  let MarketBatteryIntersectionInboundNumVehicles = 0;

  let CorbetIronIntersectionOutbound = 0;
  let CorbetIronIntersectionOutboundNumVehicles = 0;
  let CorbetIronIntersectionInbound = 0;
  let CorbetIronIntersectionInboundNumVehicles = 0;

  let MarketSanchezIntersectionOutbound = 0;
  let MarketSanchezIntersectionOutboundNumVehicles = 0;
  let MarketSanchezIntersectionInbound = 0;
  let MarketSanchezIntersectionInboundNumVehicles = 0;

  let MarketMontgomeryIntersectionOutbound = 0;
  let MarketMontgomeryIntersectionOutboundNumVehicles = 0;
  let MarketMontgomeryIntersectionInbound = 0;
  let MarketMontgomeryIntersectionInboundNumVehicles = 0;

  let MarketNoeIntersectionOutbound = 0;
  let MarketNoeIntersectionOutboundNumVehicles = 0;
  let MarketNoeIntersectionInbound = 0;
  let MarketNoeIntersectionInboundNumVehicles = 0;

  let MarketHydeIntersectionOutbound = 0;
  let MarketHydeIntersectionOutboundNumVehicles = 0;
  let MarketHydeIntersectionInbound = 0;
  let MarketHydeIntersectionInboundNumVehicles = 0;

  let TwinPeaksIntersectionOutbound = 0;
  let TwinPeaksIntersectionOutboundNumVehicles = 0;
  let TwinPeaksIntersectionInbound = 0;
  let TwinPeaksIntersectionInboundNumVehicles = 0;

  let MarketJonesIntersectionOutbound = 0;
  let MarketJonesIntersectionOutboundNumVehicles = 0;
  let MarketJonesIntersectionInbound = 0;
  let MarketJonesIntersectionInboundNumVehicles = 0;

  let MarketStorrieIntersectionOutbound = 0;
  let MarketStorrieIntersectionOutboundNumVehicles = 0;
  let MarketStorrieIntersectionInbound = 0;
  let MarketStorrieIntersectionInboundNumVehicles = 0;

  let MarketMasonIntersectionOutbound = 0;
  let MarketMasonIntersectionOutboundNumVehicles = 0;
  let MarketMasonIntersectionInbound = 0;
  let MarketMasonIntersectionInboundNumVehicles = 0;

  let EmbarcaderoFolsomIntersectionOutbound = 0;
  let EmbarcaderoFolsomIntersectionOutboundNumVehicles = 0;
  let EmbarcaderoFolsomIntersectionInbound = 0;
  let EmbarcaderoFolsomIntersectionInboundNumVehicles = 0;

  let BettyIntersectionOutbound = 0;
  let BettyIntersectionOutboundNumVehicles = 0;
  let BettyIntersectionInbound = 0;
  let BettyIntersectionInboundNumVehicles = 0;

  let DeweyPachecoIntersectionOutbound = 0;
  let DeweyPachecoIntersectionOutboundNumVehicles = 0;
  let DeweyPachecoIntersectionInbound = 0;
  let DeweyPachecoIntersectionInboundNumVehicles = 0;

  let MarketLagunaIntersectionOutbound = 0;
  let MarketLagunaIntersectionOutboundNumVehicles = 0;
  let MarketLagunaIntersectionInbound = 0;
  let MarketLagunaIntersectionInboundNumVehicles = 0;

  let MarketLarkinIntersectionOutbound = 0;
  let MarketLarkinIntersectionOutboundNumVehicles = 0;
  let MarketLarkinIntersectionInbound = 0;
  let MarketLarkinIntersectionInboundNumVehicles = 0;

  let SutroReservoirIntersectionOutbound = 0;
  let SutroReservoirIntersectionOutboundNumVehicles = 0;
  let SutroReservoirIntersectionInbound = 0;
  let SutroReservoirIntersectionInboundNumVehicles = 0;

  let outboundAverageDurationArrayStart = [];
  let outboundAverageIndexStart = 0;

  let outboundAverageDurationArrayEnd = [];
  let outboundAverageIndexEnd = 0;

  let inboundAverageDurationArrayStart = [];
  let inboundAverageIndexStart = 0;

  let inboundAverageDurationArrayEnd = [];
  let inboundAverageIndexEnd = 0;

  let outboundFullTrips = 0;
  let inboundFullTrips = 0;
  let outboundAveragesArray = [];
  let inboundAveragesArray = [];

  // Count full trips
  Object.values(fullTripData).forEach((trip) => {
    const date = new Date(trip.endTimestamp);
    const pdtDate = new Date(convertUTCToPDT(date));
    const hour = pdtDate.getHours();
    if ((startHourFilter === "all" || endHourFilter === "all") || ((parseInt(startHourFilter) <= parseInt(hour)) && (parseInt(hour) < parseInt(endHourFilter)))) {
      if (trip.direction === "outbound") {
        outboundFullTrips++;
        let totalTripTime = getTimeDifferenceInSeconds(trip.startTimestamp, trip.endTimestamp);
        outboundAveragesArray.push(totalTripTime);
      } else if (trip.direction === "inbound") {
        inboundFullTrips++;
        let totalTripTime = getTimeDifferenceInSeconds(trip.startTimestamp, trip.endTimestamp);
        inboundAveragesArray.push(totalTripTime);
      }
    }
  });
  // Cycle through the data
  Object.values(dataFile).forEach((stop) => {
    const date = new Date(stop.timestamp);
    const pdtDate = new Date(convertUTCToPDT(date));
    const hour = pdtDate.getHours();
    // if timestamp falls between hour filters (or any hour if at least one hour is "all")
    if ((startHourFilter === "all" || endHourFilter === "all") || ((parseInt(startHourFilter) <= parseInt(hour)) && (parseInt(hour) < parseInt(endHourFilter)))) {
      if (stop.direction_id === 0) { // outbound
        if (stop.atStation) { // vehicle at station
          if (stop.stationId === "17217") {// start of trip outbound
            const tripStart = {
              "timestamp": stop.timestamp,
              "trip_id": stop.trip_id,
              "timeAtStop": stop.timeAtStop,
              "trip_day": pdtDate.getDate(),
            };

            let tripStartExists = false

            outboundAverageDurationArrayStart.forEach((averageItem, index) => {
              if (averageItem.trip_id === stop.trip_id) {
                // replace averageItem with tripStart
                outboundAverageDurationArrayStart.splice(index, 1, tripStart);
                tripStartExists = true;
              }
            });

            if (!tripStartExists) {
              outboundAverageDurationArrayStart[outboundAverageIndexStart] = tripStart;
              outboundAverageIndexStart++;
            }
          }
          if (stop.stationId === "15418") {// end of trip outbound
            const tripStart = {
              "timestamp": stop.timestamp,
              "trip_id": stop.trip_id,
              "timeAtStop": stop.timeAtStop,
              "trip_day": pdtDate.getDate(),
            };

            let tripEndExists = false;

            outboundAverageDurationArrayEnd.forEach((averageItem, index) => {
              if (averageItem.trip_id === stop.trip_id) {
                // replace averageItem with tripStart
                outboundAverageDurationArrayEnd.splice(index, 1, tripStart);
                tripEndExists = true;
              }
            });

            if (!tripEndExists) {
              outboundAverageDurationArrayEnd[outboundAverageIndexEnd] = tripStart;
              outboundAverageIndexEnd++;
            }
          }
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
          } else if (stop.stationId === "15798" && stop.timeAtStop > 0) { // Ocean Ave & Miramar Way outbound
            OceanMiramarStationOutbound += stop.timeAtStop;
            OceanMiramarStationOutboundNumVehicles++;
          } else if (stop.stationId === "15795" && stop.timeAtStop > 0) { // Ocean Ave & Lee outbound
            OceanLeeStationOutbound += stop.timeAtStop;
            OceanLeeStationOutboundNumVehicles++;
          } else if (stop.stationId === "15785" && stop.timeAtStop > 0) { // Ocean Ave & CCSF Pedestrian Bridge
            OceanCCSFStationOutbound += stop.timeAtStop;
            OceanCCSFStationOutboundNumVehicles++;
          } else if (stop.stationId === "15418" && stop.timeAtStop > 0) { // Balboa Park BART Mezzanine Level Station
            BalboaParkStationOutbound += stop.timeAtStop;
            BalboaParkStationOutboundNumVehicles++;
          }
        } else if (stop.atIntersection && stop.timeAtStop > 0) { // vehicle at intersection
          if (stop.intersectionCrossStreet === "Embarcadero & Mission") {
            EmbarcaderoMissionIntersectionOutbound += stop.timeAtStop;
            EmbarcaderoMissionIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Embarcadero & Howard St") {
            EmbarcaderoHowardIntersectionOutbound += stop.timeAtStop;
            EmbarcaderoHowardIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "West Portal Ave & Dewey Blvd") {
            WPDeweyIntersectionOutbound += stop.timeAtStop;
            WPDeweyIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Junipero Serra Blvd & Monterey Blvd") {
            JSMontereyIntersectionOutbound += stop.timeAtStop;
            JSMontereyIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Steuart St") {
            MarketSteuartIntersectionOutbound += stop.timeAtStop;
            MarketSteuartIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "West Portal Ave & Vicente St") {
            WPVicenteIntersectionOutbound += stop.timeAtStop;
            WPVicenteIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Eureka St") {
            MarketEurekaIntersectionOutbound += stop.timeAtStop;
            MarketEurekaIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Duboce Ave") {
            MarketDuboceIntersectionOutbound += stop.timeAtStop;
            MarketDuboceIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Franklin St") {
            MarketFranklinIntersectionOutbound += stop.timeAtStop;
            MarketFranklinIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Howlth St & Ocean Ave") {
            HowlthOceanIntersectionOutbound += stop.timeAtStop;
            HowlthOceanIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Golden Gate Ave") {
            MarketGGIntersectionOutbound += stop.timeAtStop;
            MarketGGIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Ocean Ave & Balboa Park") {
            HowlthBalboaParkIntersectionOutbound += stop.timeAtStop;
            HowlthBalboaParkIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Ocean Ave & Plymouth Ave") {
            OceanPlymouthIntersectionOutbound += stop.timeAtStop;
            OceanPlymouthIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Ocean Ave & Cerritos Ave") {
            OceanCerritosIntersectionOutbound += stop.timeAtStop;
            OceanCerritosIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "West Portal Ave & 15th Ave") {
            WP15IntersectionOutbound += stop.timeAtStop;
            WP15IntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Battery St") {
            MarketBatteryIntersectionOutbound += stop.timeAtStop;
            MarketBatteryIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Corbett Ave & Iron Alley") {
            CorbetIronIntersectionOutbound += stop.timeAtStop;
            CorbetIronIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Sanchez St") {
            MarketSanchezIntersectionOutbound += stop.timeAtStop;
            MarketSanchezIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Montgomery St") {
            MarketMontgomeryIntersectionOutbound += stop.timeAtStop;
            MarketMontgomeryIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Noe St") {
            MarketNoeIntersectionOutbound += stop.timeAtStop;
            MarketNoeIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Hyde St") {
            MarketHydeIntersectionOutbound += stop.timeAtStop;
            MarketHydeIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Twin Peaks Blvd") {
            TwinPeaksIntersectionOutbound += stop.timeAtStop;
            TwinPeaksIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Jones St") {
            MarketJonesIntersectionOutbound += stop.timeAtStop;
            MarketJonesIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Storrie St") {
            MarketStorrieIntersectionOutbound += stop.timeAtStop;
            MarketStorrieIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Mason St") {
            MarketMasonIntersectionOutbound += stop.timeAtStop;
            MarketMasonIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Embarcadero & Folsom St") {
            EmbarcaderoFolsomIntersectionOutbound += stop.timeAtStop;
            EmbarcaderoFolsomIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Betty Sutro Meadow") {
            BettyIntersectionOutbound += stop.timeAtStop;
            BettyIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Dewey Blvd & Pacheco St") {
            DeweyPachecoIntersectionOutbound += stop.timeAtStop;
            DeweyPachecoIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Laguna St") {
            MarketLagunaIntersectionOutbound += stop.timeAtStop;
            MarketLagunaIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Larkin St") {
            MarketLarkinIntersectionOutbound += stop.timeAtStop;
            MarketLarkinIntersectionOutboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Sutro Reservoir") {
            SutroReservoirIntersectionOutbound += stop.timeAtStop;
            SutroReservoirIntersectionOutboundNumVehicles++;
          } else {
            console.log(stop.intersectionCrossStreet);
          }
        }
      } else if (stop.direction_id === 1) { // inbound
        if (stop.atStation) { // vehicle at station
          if (stop.stationId === "17778") {// start of trip inbound
            const tripStart = {
              "timestamp": stop.timestamp,
              "trip_id": stop.trip_id,
              "timeAtStop": stop.timeAtStop,
              "trip_day": pdtDate.getDate(),
            };

            let tripStartExists = false

            inboundAverageDurationArrayStart.forEach((averageItem, index) => {
              if (averageItem.trip_id === stop.trip_id) {
                // replace averageItem with tripStart
                inboundAverageDurationArrayStart.splice(index, 1, tripStart);
                tripStartExists = true;
              }
            });

            if (!tripStartExists) {
              inboundAverageDurationArrayStart[inboundAverageIndexStart] = tripStart;
              inboundAverageIndexStart++;
            }
          }
          if (stop.stationId === "16992") {// end of trip inbound
            const tripStart = {
              "timestamp": stop.timestamp,
              "trip_id": stop.trip_id,
              "timeAtStop": stop.timeAtStop,
              "trip_day": pdtDate.getDate(),
            };

            let tripEndExists = false;

            inboundAverageDurationArrayEnd.forEach((averageItem, index) => {
              if (averageItem.trip_id === stop.trip_id) {
                // replace averageItem with tripStart
                inboundAverageDurationArrayEnd.splice(index, 1, tripStart);
                tripEndExists = true;
              }
            });

            if (!tripEndExists) {
              inboundAverageDurationArrayEnd[inboundAverageIndexEnd] = tripStart;
              inboundAverageIndexEnd++;
            }
          }
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
          } else if (stop.stationId === "15787" && stop.timeAtStop > 0) { // Ocean Ave & Dorado Ter inbound
            OceanDoradoStationInbound += stop.timeAtStop;
            OceanDoradoStationInboundNumVehicles++;
          } else if (stop.stationId === "15797" && stop.timeAtStop > 0) { // Ocean Ave & Miramar inbound
            OceanMiramarStationInbound += stop.timeAtStop;
            OceanMiramarStationInboundNumVehicles++;
          } else if (stop.stationId === "15794" && stop.timeAtStop > 0) { // Ocean Ave & Lee Ave inbound
            OceanLeeStationInbound += stop.timeAtStop;
            OceanLeeStationInboundNumVehicles++;
          } else if (stop.stationId === "15784" && stop.timeAtStop > 0) { // Ocean Ave & CCSF inbound
            OceanCCSFStationInbound += stop.timeAtStop;
            OceanCCSFStationInboundNumVehicles++;
          } else if (stop.stationId === "17778" && stop.timeAtStop > 0) { // San Jose & Geneva inbound
            SanJoseGenevaStationInbound += stop.timeAtStop;
            SanJoseGenevaStationInboundNumVehicles++;
          }
        } else if (stop.atIntersection && stop.timeAtStop > 0) { // vehicle at intersection
          if (stop.intersectionCrossStreet === "Embarcadero & Mission") {
            EmbarcaderoMissionIntersectionInbound += stop.timeAtStop;
            EmbarcaderoMissionIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Embarcadero & Howard St") {
            EmbarcaderoHowardIntersectionInbound += stop.timeAtStop;
            EmbarcaderoHowardIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "West Portal Ave & Dewey Blvd") {
            WPDeweyIntersectionInbound += stop.timeAtStop;
            WPDeweyIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Junipero Serra Blvd & Monterey Blvd") {
            JSMontereyIntersectionInbound += stop.timeAtStop;
            JSMontereyIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Steuart St") {
            MarketSteuartIntersectionInbound += stop.timeAtStop;
            MarketSteuartIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "West Portal Ave & Vicente St") {
            WPVicenteIntersectionInbound += stop.timeAtStop;
            WPVicenteIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Eureka St") {
            MarketEurekaIntersectionInbound += stop.timeAtStop;
            MarketEurekaIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Duboce Ave") {
            MarketDuboceIntersectionInbound += stop.timeAtStop;
            MarketDuboceIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Franklin St") {
            MarketFranklinIntersectionInbound += stop.timeAtStop;
            MarketFranklinIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Howlth St & Ocean Ave") {
            HowlthOceanIntersectionInbound += stop.timeAtStop;
            HowlthOceanIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Golden Gate Ave") {
            MarketGGIntersectionInbound += stop.timeAtStop;
            MarketGGIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Ocean Ave & Balboa Park") {
            HowlthBalboaParkIntersectionInbound += stop.timeAtStop;
            HowlthBalboaParkIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Ocean Ave & Plymouth Ave") {
            OceanPlymouthIntersectionInbound += stop.timeAtStop;
            OceanPlymouthIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Ocean Ave & Cerritos Ave") {
            OceanCerritosIntersectionInbound += stop.timeAtStop;
            OceanCerritosIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "West Portal Ave & 15th Ave") {
            WP15IntersectionInbound += stop.timeAtStop;
            WP15IntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Battery St") {
            MarketBatteryIntersectionInbound += stop.timeAtStop;
            MarketBatteryIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Corbett Ave & Iron Alley") {
            CorbetIronIntersectionInbound += stop.timeAtStop;
            CorbetIronIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Sanchez St") {
            MarketSanchezIntersectionInbound += stop.timeAtStop;
            MarketSanchezIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Montgomery St") {
            MarketMontgomeryIntersectionInbound += stop.timeAtStop;
            MarketMontgomeryIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Noe St") {
            MarketNoeIntersectionInbound += stop.timeAtStop;
            MarketNoeIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Hyde St") {
            MarketHydeIntersectionInbound += stop.timeAtStop;
            MarketHydeIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Twin Peaks Blvd") {
            TwinPeaksIntersectionInbound += stop.timeAtStop;
            TwinPeaksIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Jones St") {
            MarketJonesIntersectionInbound += stop.timeAtStop;
            MarketJonesIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Storrie St") {
            MarketStorrieIntersectionInbound += stop.timeAtStop;
            MarketStorrieIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Mason St") {
            MarketMasonIntersectionInbound += stop.timeAtStop;
            MarketMasonIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Embarcadero & Folsom St") {
            EmbarcaderoFolsomIntersectionInbound += stop.timeAtStop;
            EmbarcaderoFolsomIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Betty Sutro Meadow") {
            BettyIntersectionInbound += stop.timeAtStop;
            BettyIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Dewey Blvd & Pacheco St") {
            DeweyPachecoIntersectionInbound += stop.timeAtStop;
            DeweyPachecoIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Laguna St") {
            MarketLagunaIntersectionInbound += stop.timeAtStop;
            MarketLagunaIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Market St & Larkin St") {
            MarketLarkinIntersectionInbound += stop.timeAtStop;
            MarketLarkinIntersectionInboundNumVehicles++;
          } else if (stop.intersectionCrossStreet === "Sutro Reservoir") {
            SutroReservoirIntersectionInbound += stop.timeAtStop;
            SutroReservoirIntersectionInboundNumVehicles++;
          } else {
            console.log(stop.intersectionCrossStreet);
          }
        }
      }
    }
  });

  // calculate averages
  let outboundTotalTime = 0;
  let outboundTotalIndex = outboundAveragesArray.length;
  outboundAveragesArray.forEach((time) => {
    outboundTotalTime += time;
  });

  let outboundAverageTimeInSeconds = outboundTotalTime / outboundTotalIndex;
  let outboundAverageTimeInMinutes = convertSecondsToMinutes(outboundAverageTimeInSeconds);

  // calculate averages
  let inboundTotalTime = 0;
  let inboundTotalIndex = inboundAveragesArray.length;
  inboundAveragesArray.forEach((time) => {
    inboundTotalTime += time;
  });
  let inboundAverageTimeInSeconds = inboundTotalTime / inboundTotalIndex;
  let inboundAverageTimeInMinutes = convertSecondsToMinutes(inboundAverageTimeInSeconds);

  // Make array of station data
  const stationsData = [
    {
      name: 'Embarcadero and& Folsom St',
      totalTime: EmbarcaderoFolsomIntersectionOutbound,
      numVehicles: EmbarcaderoFolsomIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Embarcadero and& Folsom St',
      totalTime: EmbarcaderoFolsomIntersectionInbound,
      numVehicles: EmbarcaderoFolsomIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Embarcadero and& Howard',
      totalTime: EmbarcaderoHowardIntersectionOutbound,
      numVehicles: EmbarcaderoHowardIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Embarcadero and& Howard',
      totalTime: EmbarcaderoHowardIntersectionInbound,
      numVehicles: EmbarcaderoHowardIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Embarcadero and& Mission',
      totalTime: EmbarcaderoMissionIntersectionOutbound,
      numVehicles: EmbarcaderoMissionIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Embarcadero and& Mission',
      totalTime: EmbarcaderoMissionIntersectionInbound,
      numVehicles: EmbarcaderoMissionIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Steuart St',
      totalTime: MarketSteuartIntersectionOutbound,
      numVehicles: MarketSteuartIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Steuart St',
      totalTime: MarketSteuartIntersectionInbound,
      numVehicles: MarketSteuartIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Embarcadero Station',
      totalTime: EmbarcaderoStationOutbound,
      numVehicles: EmbarcaderoStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Embarcadero Station',
      totalTime: EmbarcaderoStationInbound,
      numVehicles: EmbarcaderoStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Battery St',
      totalTime: MarketBatteryIntersectionOutbound,
      numVehicles: MarketBatteryIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Battery St',
      totalTime: MarketBatteryIntersectionInbound,
      numVehicles: MarketBatteryIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Montgomery Station',
      totalTime: MontgomeryStationOutbound,
      numVehicles: MontgomeryStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Montgomery Station',
      totalTime: MontgomeryStationInbound,
      numVehicles: MontgomeryStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Kearny St',
      totalTime: MarketMontgomeryIntersectionOutbound,
      numVehicles: MarketMontgomeryIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Kearny St',
      totalTime: MarketMontgomeryIntersectionInbound,
      numVehicles: MarketMontgomeryIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Powell Station',
      totalTime: PowellStationOutbound,
      numVehicles: PowellStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Powell Station',
      totalTime: PowellStationInbound,
      numVehicles: PowellStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Mason St',
      totalTime: MarketMasonIntersectionOutbound,
      numVehicles: MarketMasonIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Mason St',
      totalTime: MarketMasonIntersectionInbound,
      numVehicles: MarketMasonIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Golden Gate Ave',
      totalTime: MarketGGIntersectionOutbound,
      numVehicles: MarketGGIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Golden Gate Ave',
      totalTime: MarketGGIntersectionInbound,
      numVehicles: MarketGGIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Jones St',
      totalTime: MarketJonesIntersectionOutbound,
      numVehicles: MarketJonesIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Jones St',
      totalTime: MarketJonesIntersectionInbound,
      numVehicles: MarketJonesIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Civic Center Station',
      totalTime: CCStationOutbound,
      numVehicles: CCStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Civic Center Station',
      totalTime: CCStationInbound,
      numVehicles: CCStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Hyde St',
      totalTime: MarketHydeIntersectionOutbound,
      numVehicles: MarketHydeIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Hyde St',
      totalTime: MarketHydeIntersectionInbound,
      numVehicles: MarketHydeIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Larkin St',
      totalTime: MarketLarkinIntersectionOutbound,
      numVehicles: MarketLarkinIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Larkin St',
      totalTime: MarketLarkinIntersectionInbound,
      numVehicles: MarketLarkinIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Van Ness Station',
      totalTime: VNStationOutbound,
      numVehicles: VNStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Van Ness Station',
      totalTime: VNStationInbound,
      numVehicles: VNStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Franklin St',
      totalTime: MarketFranklinIntersectionOutbound,
      numVehicles: MarketFranklinIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Franklin St',
      totalTime: MarketFranklinIntersectionInbound,
      numVehicles: MarketFranklinIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Laguna St',
      totalTime: MarketLagunaIntersectionOutbound,
      numVehicles: MarketLagunaIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Laguna St',
      totalTime: MarketLagunaIntersectionInbound,
      numVehicles: MarketLagunaIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Duboce Ave',
      totalTime: MarketDuboceIntersectionOutbound,
      numVehicles: MarketDuboceIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Duboce Ave',
      totalTime: MarketDuboceIntersectionInbound,
      numVehicles: MarketDuboceIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Church Station',
      totalTime: ChurchStationOutbound,
      numVehicles: ChurchStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Church Station',
      totalTime: ChurchStationInbound,
      numVehicles: ChurchStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Sanchez St',
      totalTime: MarketSanchezIntersectionOutbound,
      numVehicles: MarketSanchezIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Sanchez St',
      totalTime: MarketSanchezIntersectionInbound,
      numVehicles: MarketSanchezIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Noe St',
      totalTime: MarketNoeIntersectionOutbound,
      numVehicles: MarketNoeIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Noe St',
      totalTime: MarketNoeIntersectionInbound,
      numVehicles: MarketNoeIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Castro Station',
      totalTime: CastroStationOutbound,
      numVehicles: CastroStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Castro Station',
      totalTime: CastroStationInbound,
      numVehicles: CastroStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Market St and& Eureka St',
      totalTime: MarketEurekaIntersectionOutbound,
      numVehicles: MarketEurekaIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Eureka St',
      totalTime: MarketEurekaIntersectionInbound,
      numVehicles: MarketEurekaIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Market St and& Storrie St',
      totalTime: MarketStorrieIntersectionOutbound,
      numVehicles: MarketStorrieIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Market St and& Storrie St',
      totalTime: MarketStorrieIntersectionInbound,
      numVehicles: MarketStorrieIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Corbett Ave and& Iron Alley',
      totalTime: CorbetIronIntersectionOutbound,
      numVehicles: CorbetIronIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Corbett Ave and& Iron Alley',
      totalTime: CorbetIronIntersectionInbound,
      numVehicles: CorbetIronIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Twin Peaks Blvd',
      totalTime: TwinPeaksIntersectionOutbound,
      numVehicles: TwinPeaksIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Twin Peaks Blvd',
      totalTime: TwinPeaksIntersectionInbound,
      numVehicles: TwinPeaksIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Sutro Reservoir',
      totalTime: SutroReservoirIntersectionOutbound,
      numVehicles: SutroReservoirIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Sutro Reservoir',
      totalTime: SutroReservoirIntersectionInbound,
      numVehicles: SutroReservoirIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Betty Sutro Meadow',
      totalTime: BettyIntersectionOutbound,
      numVehicles: BettyIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Betty Sutro Meadow',
      totalTime: BettyIntersectionInbound,
      numVehicles: BettyIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Forest Hill Station',
      totalTime: FHStationOutbound,
      numVehicles: FHStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Forest Hill Station',
      totalTime: FHStationInbound,
      numVehicles: FHStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Dewey Blvd and& Pacheco St',
      totalTime: DeweyPachecoIntersectionOutbound,
      numVehicles: DeweyPachecoIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Dewey Blvd and& Pacheco St',
      totalTime: DeweyPachecoIntersectionInbound,
      numVehicles: DeweyPachecoIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'West Portal Ave and& Dewey Blvd',
      totalTime: WPDeweyIntersectionOutbound,
      numVehicles: WPDeweyIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'West Portal Ave and& Dewey Blvd',
      totalTime: WPDeweyIntersectionInbound,
      numVehicles: WPDeweyIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'West Portal Station',
      totalTime: WPStationOutbound,
      numVehicles: WPStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'West Portal Station',
      totalTime: WPStationInbound,
      numVehicles: WPStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'West Portal Ave and& & Vicente St',
      totalTime: WPVicenteIntersectionOutbound,
      numVehicles: WPVicenteIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'West Portal Ave and& & Vicente St',
      totalTime: WPVicenteIntersectionInbound,
      numVehicles: WPVicenteIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'West Portal and& & 14th',
      totalTime: WP14StationOutbound,
      numVehicles: WP14StationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'West Portal and& & 14th',
      totalTime: WP14StationInbound,
      numVehicles: WP14StationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'West Portal Ave and& & 15th Ave',
      totalTime: WP15IntersectionOutbound,
      numVehicles: WP15IntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'West Portal Ave and& & 15th Ave',
      totalTime: WP15IntersectionInbound,
      numVehicles: WP15IntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'West Portal and& & Sloat Blvd',
      totalTime: WPSLStationOutbound,
      numVehicles: WPSLStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'West Portal and& & Sloat Blvd',
      totalTime: WPSLStationInbound,
      numVehicles: WPSLStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Junipero Serra Blvd and& & Monterey Blvd',
      totalTime: JSMontereyIntersectionOutbound,
      numVehicles: JSMontereyIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Junipero Serra Blvd and& & Monterey Blvd',
      totalTime: JSMontereyIntersectionInbound,
      numVehicles: JSMontereyIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Junipero Serra Blvd and& & Ocean Ave',
      totalTime: JSStationOutbound,
      numVehicles: JSStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Junipero Serra Blvd and& & Ocean Ave',
      totalTime: JSStationInbound,
      numVehicles: JSStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & San Leandro Way',
      totalTime: OSLStationOutbound,
      numVehicles: OSLStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & San Leandro Way',
      totalTime: OSLStationInbound,
      numVehicles: OSLStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Aptos',
      totalTime: OceanAptosStationOutbound,
      numVehicles: OceanAptosStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Aptos',
      totalTime: OceanAptosStationInbound,
      numVehicles: OceanAptosStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Cerritos Ave',
      totalTime: OceanCerritosIntersectionOutbound,
      numVehicles: OceanCerritosIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Ocean Ave and& & Cerritos Ave',
      totalTime: OceanCerritosIntersectionInbound,
      numVehicles: OceanCerritosIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Ocean Ave and Victoria St',
      totalTime: OceanVictoriaStationOutbound,
      numVehicles: OceanVictoriaStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and Fairfield Way',
      totalTime: OceanFairfieldStationInbound,
      numVehicles: OceanFairfieldStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and Jules Ave',
      totalTime: OceanJulesStationOutbound,
      numVehicles: OceanJulesStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and Dorado Ter',
      totalTime: OceanDoradoStationInbound,
      numVehicles: OceanDoradoStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Miramar Ave',
      totalTime: OceanMiramarStationOutbound,
      numVehicles: OceanMiramarStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Miramar Ave',
      totalTime: OceanMiramarStationInbound,
      numVehicles: OceanMiramarStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Plymouth Ave',
      totalTime: OceanPlymouthIntersectionOutbound,
      numVehicles: OceanPlymouthIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Ocean Ave and& & Plymouth Ave',
      totalTime: OceanPlymouthIntersectionInbound,
      numVehicles: OceanPlymouthIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Ocean Ave and& & Lee Ave',
      totalTime: OceanLeeStationOutbound,
      numVehicles: OceanLeeStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & Lee Ave',
      totalTime: OceanLeeStationInbound,
      numVehicles: OceanLeeStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & CCSF Pedestrian Bridge',
      totalTime: OceanCCSFStationOutbound,
      numVehicles: OceanCCSFStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'Ocean Ave and& & CCSF Pedestrian Bridge',
      totalTime: OceanCCSFStationInbound,
      numVehicles: OceanCCSFStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    },
    {
      name: 'Howlth St and& & Ocean Ave',
      totalTime: HowlthOceanIntersectionOutbound,
      numVehicles: HowlthOceanIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Howlth St and& & Ocean Ave',
      totalTime: HowlthOceanIntersectionInbound,
      numVehicles: HowlthOceanIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Ocean Ave and& & Balboa Park',
      totalTime: HowlthBalboaParkIntersectionOutbound,
      numVehicles: HowlthBalboaParkIntersectionOutboundNumVehicles,
      direction: 'outbound',
      isStation: false
    },
    {
      name: 'Ocean Ave and& & Balboa Park',
      totalTime: HowlthBalboaParkIntersectionInbound,
      numVehicles: HowlthBalboaParkIntersectionInboundNumVehicles,
      direction: 'inbound',
      isStation: false
    },
    {
      name: 'Balboa Park BART Station',
      totalTime: BalboaParkStationOutbound,
      numVehicles: BalboaParkStationOutboundNumVehicles,
      direction: 'outbound',
      isStation: true
    },
    {
      name: 'San Jose Ave and Geneva Ave Station',
      totalTime: SanJoseGenevaStationInbound,
      numVehicles: SanJoseGenevaStationInboundNumVehicles,
      direction: 'inbound',
      isStation: true
    }
  ];

  stationsData.reverse(); // Reverse the order of the stations to match the SVG layout

  // make axis
  svg.append('rect')
      .attr('x', 20)
      .attr('y', 0)
      .attr('width', 1)
      .attr('height', 750)
      .attr('fill', "#71797E");

  svg.append('rect')
    .attr('x', 0)
    .attr('y', outboundCY - 60 - 3)
    .attr('width', 2400)
    .attr('height', 1)
    .attr('fill', "#71797E");

  svg.append('text')
    .attr('x', 22)
    .attr('y', outboundCY - 60 - 3)
    .attr('class', 'minute-label')
    .text('20 seconds');

  svg.append('rect')
    .attr('x', 0)
    .attr('y', outboundCY - 120 - 3)
    .attr('width', 2400)
    .attr('height', 1)
    .attr('fill', "#71797E");

  svg.append('text')
    .attr('x', 22)
    .attr('y', outboundCY - 120 - 3)
    .attr('class', 'minute-label')
    .text('40 seconds');

  svg.append('rect')
    .attr('x', 0)
    .attr('y', outboundCY - 180 - 3)
    .attr('width', 2400)
    .attr('height', 1)
    .attr('fill', "#71797E");

  svg.append('text')
    .attr('x', 22)
    .attr('y', outboundCY - 180 - 3)
    .attr('class', 'minute-label')
    .text('60 seconds');

  svg.append('rect')
    .attr('x', 0)
    .attr('y', inboundCY + 60 + 12)
    .attr('width', 2400)
    .attr('height', 1)
    .attr('fill', "#71797E");

  svg.append('text')
    .attr('x', 22)
    .attr('y', inboundCY + 60 + 12)
    .attr('class', 'minute-label')
    .text('20 seconds');

  svg.append('rect')
    .attr('x', 0)
    .attr('y', inboundCY + 120 + 12)
    .attr('width', 2400)
    .attr('height', 1)
    .attr('fill', "#71797E");

  svg.append('text')
    .attr('x', 22)
    .attr('y', inboundCY + 120 + 12)
    .attr('class', 'minute-label')
    .text('40 seconds');

  svg.append('rect')
    .attr('x', 0)
    .attr('y', inboundCY + 180 + 12)
    .attr('width', 2400)
    .attr('height', 1)
    .attr('fill', "#71797E");
  
  svg.append('text')
    .attr('x', 22)
    .attr('y', inboundCY + 180 + 12)
    .attr('class', 'minute-label')
    .text('60 seconds');

  svg.append('text')
    .attr('x', 22)
    .attr('y', 355)
    .attr('class', 'minute-label time-axis-label')
    .attr('transform', 'rotate(90, 22, 355)')
    .attr("text-anchor", "middle")
    .text('Average Time in Seconds')

  // End of axis

  // Variable for total time saved at intersections
  let totalTimeSavedOutbound = 0;
  let totalTimeSavedInbound = 0;

  // Loop through stations data to create bars and labels
  stationsData.forEach((station, index) => {
    let averageTime = 0;
    if (station.direction === "outbound") {
      averageTime = station.numVehicles > 0 ? station.totalTime / outboundFullTrips : 0;
    } else if (station.direction === "inbound") {
      averageTime = station.numVehicles > 0 ? station.totalTime / inboundFullTrips : 0;
    }
    const isOutbound = station.direction === 'outbound';
    const isStation = station.isStation;
    const offsetX = 20;
    const cy = isOutbound ? outboundCY : inboundCY;
    let cx;
    if (index % 2 > 0) {
      cx = ((index - 1) * offsetX) + 20; // Adjust horizontal position for each station
    } else {
      cx = (index * offsetX) + 20; // Adjust horizontal position for each station
    }
    cx += 70; // push out right to make room for axis
    let height = averageTime;
    const scalar = 3; // Scale factor to adjust height
    height = height * scalar; // Scale the height

    // Add heights for non-terminal intersections to get average time saved
    if (isOutbound) {
      if (station.name !== "Embarcadero and& Folsom St" && station.name !== "Embarcadero and& Howard" && station.name !== "Embarcadero and& Mission" && station.name !== "Market St and& Steuart St"  && !station.isStation) {
        totalTimeSavedOutbound += averageTime;
      }
    } else {
      if (station.name !== "Embarcadero and& Folsom St" && station.name !== "Embarcadero and& Howard" && station.name !== "Embarcadero and& Mission" && station.name !== "Market St and& Steuart St"  && !station.isStation) {
        totalTimeSavedInbound += averageTime;
      }
    }

    svg.append('rect')
      .attr('x', cx)
      .attr('y', isOutbound ? cy - 2.5 - height : cy + 12.5)
      .attr('width', offsetX + 10)
      .attr('height', height)
      .attr('class', isStation ? `${station.name.toLowerCase().replace(/ /g, '-')}-rect station-rect` : `${station.name.toLowerCase().replace(/ /g, '-')}-rect intersection-rect`)
      .attr('fill', 'steelblue')
      .on("mouseover", (event, d) => {
        if (isStation) {
          if (station.totalTime > 3600) {
            if (isOutbound) {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Station: <span>${convertMinutesToHours(convertSecondsToMinutes(station.totalTime))}</span> hours<br>Total Vehicles Stopped at Station: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Station: <span>${outboundFullTrips}</span> vehicles<br>Average Time at Station: <span>${averageTime.toFixed(2)}</span> seconds`);
            } else {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Station: <span>${convertMinutesToHours(convertSecondsToMinutes(station.totalTime))}</span> hours<br>Total Vehicles Stopped at Station: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Station: <span>${inboundFullTrips}</span> vehicles<br>Average Time at Station: <span>${averageTime.toFixed(2)}</span> seconds`);
            }
          } else if (station.totalTime > 60) {
            if (isOutbound) {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Station: <span>${convertSecondsToMinutes(station.totalTime)}</span> minutes<br>Total Vehicles Stopped at Station: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Station: <span>${outboundFullTrips}</span> vehicles<br>Average Time at Station: <span>${averageTime.toFixed(2)}</span> seconds`);
            } else {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Station: <span>${convertSecondsToMinutes(station.totalTime)}</span> minutes<br>Total Vehicles Stopped at Station: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Station: <span>${inboundFullTrips}</span> vehicles<br>Average Time at Station: <span>${averageTime.toFixed(2)}</span> seconds`);
            }
          } else {
            if (isOutbound) {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Station: <span>${station.totalTime}</span> seconds<br>Total Vehicles Stopped at Station: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Station: <span>${outboundFullTrips}</span> vehicles<br>Average Time at Station: <span>${averageTime.toFixed(2)}</span> seconds`);
            } else {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Station: <span>${station.totalTime}</span> seconds<br>Total Vehicles Stopped at Station: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Station: <span>${inboundFullTrips}</span> vehicles<br>Average Time at Station: <span>${averageTime.toFixed(2)}</span> seconds`);
            }
          }
        } else {
          if (station.totalTime > 3600) {
            if (isOutbound) {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Intersection: <span>${convertMinutesToHours(convertSecondsToMinutes(station.totalTime))}</span> hours<br>Total Vehicles Stopped at Intersection: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Intersection: <span>${outboundFullTrips}</span> vehicles<br>Average Time at Intersection: <span>${averageTime.toFixed(2)}</span> seconds`);
            } else {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Intersection: <span>${convertMinutesToHours(convertSecondsToMinutes(station.totalTime))}</span> hours<br>Total Vehicles Stopped at Intersection: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Intersection: <span>${inboundFullTrips}</span> vehicles<br>Average Time at Intersection: <span>${averageTime.toFixed(2)}</span> seconds`);
            }
          } else if (station.totalTime > 60) {
            if (isOutbound) {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Intersection: <span>${convertSecondsToMinutes(station.totalTime)}</span> minutes<br>Total Vehicles Stopped at Intersection: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Intersection: <span>${outboundFullTrips}</span> vehicles<br>Average Time at Intersection: <span>${averageTime.toFixed(2)}</span> seconds`);
            } else {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Intersection: <span>${convertSecondsToMinutes(station.totalTime)}</span> minutes<br>Total Vehicles Stopped at Intersection: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Intersection: <span>${inboundFullTrips}</span> vehicles<br>Average Time at Intersection: <span>${averageTime.toFixed(2)}</span> seconds`);
            }
          } else {
            if (isOutbound) {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Intersection: <span>${station.totalTime}</span> seconds<br>Total Vehicles Stopped at Intersection: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Intersection: <span>${outboundFullTrips}</span> vehicles<br>Average Time at Intersection: <span>${averageTime.toFixed(2)}</span> seconds`);
            } else {
              tooltip.style("opacity", 1)
                .html(`<span>${station.name.replace("and&", "")}.</span><br>Total Time at Intersection: <span>${station.totalTime}</span> seconds<br>Total Vehicles Stopped at Intersection: <span>${station.numVehicles}</span> vehicles<br>Total Vehicles Passed Through Intersection: <span>${inboundFullTrips}</span> vehicles<br>Average Time at Intersection: <span>${averageTime.toFixed(2)}</span> seconds`);
            }
          }
        }
      })
      .on("mousemove", (event) => {
        tooltip.style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 20) + "px");
      })
      .on("mouseout", () => {
        tooltip.style("opacity", 0);
      });

    if (index % 2  === 0) {
      svg.append('rect')
        .attr('x', cx - 2)
        .attr('y', 235)
        .attr('width', offsetX + 14)
        .attr('height', 240)
        .attr('opacity', 0.35)
        .attr('fill', '#010101')
        .attr('style', 'z-index: -1;')
        .attr('rx', '4px');
    } 

    svg.append('text')
      .attr('x', cx + 17)
      .attr('y', 355)
      .attr('transform', 'rotate(90, ' + (cx + 17) + ', 355)')
      .attr("text-anchor", "middle")
      .attr('class', isStation ? `${station.name.toLowerCase().replace(/ /g, '-')}-label station-label label` : `${station.name.toLowerCase().replace(/ /g, '-')}-label intersection-label label`)
      .selectAll('tspan')
      .data(station.name.split('and&')) // Split the text by '&'
      .enter()
      .append('tspan')
      .attr('x', cx + 17) // Set x position for each line
      .attr('dy', (d, i) => i * 1.2 + 'em') // Set vertical offset for each line
      .text(d => d.trim()); // Add the text content
  });

  svg.append('rect')
    .attr('x', 320)
    .attr('y', 20)
    .attr('width', 300)
    .attr('height',90)
    .attr('opacity', 0.6)
    .attr('fill', '#010101')
    .attr('rx', '4px');

  svg.append('rect')
    .attr('x', 320)
    .attr('y', 650)
    .attr('width', 300)
    .attr('height',90)
    .attr('opacity', 0.6)
    .attr('fill', '#010101')
    .attr('rx', '4px');

  // Outbound labels
  svg.append('text')
    .attr('x', 470)
    .attr('y', 40)
    .attr('class', 'outbound-label label')
    .attr("text-anchor", "middle")
    .text('<-- Outbound <--');
  // Outbound Average
  svg.append('text')
    .attr('x', 470)
    .attr('y', 70)
    .attr('class', 'outbound-average-label label')
    .attr("text-anchor", "middle")
    .text('Average Trip Duration: ' + outboundAverageTimeInMinutes + ' minutes');

  // Inbound label
  svg.append('text')
    .attr('x', 470)
    .attr('y', height - 80)
    .attr('class', 'inbound-label label')
    .attr("text-anchor", "middle")
    .text('--> Inbound -->');
  // Inbound average
  svg.append('text')
    .attr('x', 470)
    .attr('y', height - 50)
    .attr('class', 'inbound-average-label label')
    .attr("text-anchor", "middle")
    .text('Average Trip Duration: ' + inboundAverageTimeInMinutes + ' minutes');

  svg.append('text')
    .attr('x', 470)
    .attr('y', height - 20)
    .attr('class', 'inbound-time-saved-label label')
    .attr("text-anchor", "middle")
    .text('Average Time Saved: ' + totalTimeSavedInbound.toFixed(1) + ' seconds');

  svg.append('text')
    .attr('x', 470)
    .attr('y', 100)
    .attr('class', 'outbound-time-saved-label label')
    .attr("text-anchor", "middle")
    .text('Average Time Saved: ' + totalTimeSavedOutbound.toFixed(1) + ' seconds');

  // Move underground rectangle to align with the west portal station rectangle
  const undergroundX = d3.select('.west-portal-station-rect').attr('x') - 2;
  d3.select('.underground').attr('x', undergroundX);

  // Move labels with differnt inbound and outbound names
  d3.select('.balboa-park-bart-station-label')
    .text(null) // Clear any existing text
    .append('tspan')
    .attr('x', 50) // Set x position for the first line
    .attr('y', 347)
    .attr('dy', '0em') // Set y offset for the first line
    .text('Balboa Park')
    .append('tspan')
    .attr('x', 50) // Set x position for the second line
    .attr('dy', '1.2em') // Set y offset for the second line
    .text('BART Station');
  d3.select('.san-jose-ave-and-geneva-ave-station-label')
    .text(null) // Clear any existing text
    .append('tspan')
    .attr('x', 160) // Set x position for the first line
    .attr('y', 347)
    .attr('dy', '0em') // Set y offset for the first line
    .text('San Jose Ave &')
    .append('tspan')
    .attr('x', 160) // Set x position for the second line
    .attr('dy', '1.2em') // Set y offset for the second line
    .text('Geneva Ave Station');
  d3.select('.ocean-ave-and-jules-ave-label')
    .text(null) // Clear any existing text
    .append('tspan')
    .attr('x', 340) // Set x position for the first line
    .attr('y', 347)
    .attr('dy', '0em') // Set y offset for the first line
    .text('Ocean Ave &')
    .append('tspan')
    .attr('x', 340) // Set x position for the second line
    .attr('dy', '1.2em') // Set y offset for the second line
    .text('Jules Ave');
  d3.select('.ocean-ave-and-dorado-ter-label')
    .text(null) // Clear any existing text
    .append('tspan')
    .attr('x', 440) // Set x position for the first line
    .attr('y', 347)
    .attr('dy', '0em') // Set y offset for the first line
    .text('Ocean Ave &')
    .append('tspan')
    .attr('x', 440) // Set x position for the second line
    .attr('dy', '1.2em') // Set y offset for the second line
    .text('Dorado Ter');
  d3.select('.ocean-ave-and-victoria-st-label')
    .text(null) // Clear any existing text
    .append('tspan')
    .attr('x', 380) // Set x position for the first line
    .attr('y', 347)
    .attr('dy', '0em') // Set y offset for the first line
    .text('Ocean Ave &')
    .append('tspan')
    .attr('x', 380) // Set x position for the second line
    .attr('dy', '1.2em') // Set y offset for the second line
    .text('Victoria St');
  d3.select('.ocean-ave-and-fairfield-way-label')
    .text(null) // Clear any existing text
    .append('tspan')
    .attr('x', 480) // Set x position for the first line
    .attr('y', 347)
    .attr('dy', '0em') // Set y offset for the first line
    .text('Ocean Ave &')
    .append('tspan')
    .attr('x', 480) // Set x position for the second line
    .attr('dy', '1.2em') // Set y offset for the second line
    .text('Fairfield Way');

  // Move labels with same inbound and outbound names
  d3.selectAll('.twin-peaks-blvd-label tspan').attr('y', 360);
  d3.selectAll('.sutro-reservoir-label tspan').attr('y', 360);
  d3.selectAll('.betty-sutro-meadow-label tspan').attr('y', 360);
  d3.selectAll('.station-label tspan').attr('y', 353);

  // Add horizontal line between Ocean Ave & Fairfield Way and Ocean Ave & Victoria St
svg.append('line')
  .attr('x1', 435) // x position of Ocean Ave & Fairfield Way
  .attr('y1', 347 + 10) // y position below the label
  .attr('x2', 415) // x position of Ocean Ave & Victoria St
  .attr('y2', 347 + 10) // y position below the label
  .attr('stroke', 'white')
  .attr('stroke-width', 1);

// Add horizontal line between Ocean Ave & Dorado Ter and Ocean Ave & Jules Ave
svg.append('line')
  .attr('x1', 395) // x position of Ocean Ave & Dorado Ter
  .attr('y1', 347 + 10) // y position below the label
  .attr('x2', 375) // x position of Ocean Ave & Jules Ave
  .attr('y2', 347 + 10) // y position below the label
  .attr('stroke', 'white')
  .attr('stroke-width', 1);

// Add horizontal line between San Jose Ave & Geneva Ave Station and Balboa Park BART Station
svg.append('line')
  .attr('x1', 115) // x position of San Jose Ave & Geneva Ave Station
  .attr('y1', 343) // y position below the label
  .attr('x2', 95) // x position of Balboa Park BART Station
  .attr('y2', 343) // y position below the label
  .attr('stroke', 'white')
  .attr('stroke-width', 1);
}

function changeData(selectedValue) {
  clearChart();
  if (selectedValue === "05-16") {
    dataToBeRendered = data1.value;
    fullTripDataToBeRendered = fullTripData1.value;
  } else if (selectedValue === "05-15") {
    dataToBeRendered = data2.value;
    fullTripDataToBeRendered = fullTripData2.value;
  } else if (selectedValue === "05-14") {
    dataToBeRendered = data3.value;
    fullTripDataToBeRendered = fullTripData3.value;
  } else if (selectedValue === "05-13") {
    dataToBeRendered = data4.value;
    fullTripDataToBeRendered = fullTripData4.value;
  } else if (selectedValue === "05-12") {
    dataToBeRendered = data5.value;
    fullTripDataToBeRendered = fullTripData5.value;
  } else if (selectedValue === "05-11") {
    dataToBeRendered = data6.value;
    fullTripDataToBeRendered = fullTripData6.value;
  } else if (selectedValue === "combined") {
    dataToBeRendered = combinedData.value;
    fullTripDataToBeRendered = fullTripCombinedData.value;
  }
  runAfterLoad(dataToBeRendered, startHour, endHour, fullTripDataToBeRendered);
}

function clearChart() {
  if (chart.value) {
    chart.value.innerHTML = ''; // Removes all content inside the div
  }
}

function filterByHourRange(hourRange) {
  let startHour = 'all';
  let endHour = 'all';

  if (hourRange === '7-10') {
    startHour = '7';
    endHour = '10';
  } else if (hourRange === '10-15') {
    startHour = '10';
    endHour = '15';
  } else if (hourRange === '15-19') {
    startHour = '15';
    endHour = '19';
  } else if (hourRange === '19-21') {
    startHour = '19';
    endHour = '21';
  }

  filterByHour(startHour, endHour);
}

function filterByHour(firstHour, lastHour) {
  if (parseInt(firstHour) >= parseInt(lastHour)) {
    alert("Start hour must be less than end hour.");
    return;
  }
  clearChart();
  startHour = firstHour;
  endHour = lastHour;
  runAfterLoad(dataToBeRendered, startHour, endHour, fullTripDataToBeRendered);
}

function convertUTCToPDT(utcTimestamp) {
  // Create a Date object from the UTC timestamp
  const utcDate = new Date(utcTimestamp + 'Z'); // ensure UTC interpretation

  // Convert to PDT using 'America/Los_Angeles' timezone parts
  const options = {
    timeZone: 'America/Los_Angeles',
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  };

  const formatter = new Intl.DateTimeFormat('en-US', options);
  const parts = formatter.formatToParts(utcDate);

  // Extract parts into an object
  const dateParts = {};
  parts.forEach(({ type, value }) => {
    dateParts[type] = value;
  });

  // Assemble the ISO-like string
  return `${dateParts.year}-${dateParts.month}-${dateParts.day}T${dateParts.hour}:${dateParts.minute}:${dateParts.second}`;
}

function getTimeDifferenceInSeconds(timestamp1, timestamp2) {
    const time1 = new Date(timestamp1).getTime();
    const time2 = new Date(timestamp2).getTime();
    const differenceInMilliseconds = Math.abs(time2 - time1);
    return differenceInMilliseconds / 1000;
}

function convertSecondsToMinutes(seconds) {
  return (seconds / 60).toFixed(1);
}

function convertMinutesToHours(minutes) {
  return (minutes / 60).toFixed(1);
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
  background: #D3D3D3;
}

.station-rect {
  stroke: lightblue;
  stroke-width: 3px;
  fill: lightblue;
}

.intersection-rect {
  stroke: steelblue;
  stroke-width: 3px;
  fill: steelblue
}

.tooltip {
  font-family: "Roboto", sans-serif;
  position: absolute;
  text-align: right;
  padding: 6px;
  font-size: 16px;
  background: #333;
  color: #fff;
  border-radius: 4px;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s ease;
  min-width: 270px;
}

.tooltip span:nth-of-type(1) {
  text-decoration: underline;
  line-height: 2.5;
}

.tooltip span {
  font-weight: 700;
  text-align: center;
}

.outbound-label, .inbound-label {
  font-family: "Roboto", sans-serif;
  fill: #010101;
  font-size: 16px;
  font-weight: 400;
  padding: 5px;
  border-radius: 2px;
}

.label {
  background: rgba(250, 250, 250);
  border-radius: 4px;
  border: 1px solid #010101;
  padding: 4px 12px;
}

.station-label.label, .intersection-label.label {
  font-family: "Roboto", sans-serif;
  fill: #010101;
  font-size: 12px;
  font-weight: 400;
  letter-spacing: 0.2px;
  padding: 2px 8px;
}

.height-label, .rectangle-height-label, .minute-label {
  font-family: "Roboto", sans-serif;
  font-weight: 600;
  font-size: 12px;
}

.rectangle-height-label {
  padding: 0 8px;
}

.intersection-label.label, .station-label.label, .label {
  background-color: #010101;
  font-weight: 300;
  stroke: #fff;
  fill: #fff;
  letter-spacing: 1px;
}

.date-selector-label, .hour-filter-controls label {
  color: #010101;
  font-family: "Roboto", sans-serif;
  margin-right: 8px;
}

#timeFilter, #hourRangeFilter {
  font-family: "Roboto", sans-serif;
  font-size: 14px;
  border-radius: 4px;
  margin-bottom: 4px;
  border: 1px solid #010101;
  background-color: #fff;
}

.time-filter-controls, .hour-filter-controls {
  display: inline-block;
  margin-right: 8px;
}

.hour-filter-controls label:nth-of-type(2) {
  margin-left: 8px;
}

.hour-filter-controls button {
  margin: 0 8px;
}

.legend-container {
  position: fixed;
  bottom: 16px;
  right: 16px;
  background: #F5F4F3;
  border: 1px solid #010101;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: height 0.3s ease, width 0.3s ease;
  width: 400px;
  height: auto;
}

.legend-toggle {
  width: 100%;
  background: #010101;
  color: #fff;
  border: none;
  padding: 8px;
  cursor: pointer;
  font-family: "Roboto", sans-serif;
  font-size: 14px;
  text-align: center;
}

.legend-content {
  padding: 8px;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-bottom: 4px;
  color: #010101;
}

.legend-color, .legend-text {
  display: inline-block;
  width: 30px;
  height: 30px;
  margin-right: 8px;
  border-radius: 4px;
}

.station-color {
  background: lightblue;
  border-radius: 0;
  border: 3px solid lightblue;
}

.intersection-color {
  background: steelblue;
  border-radius: 0;
  border: 3px solid steelblue;
}

.terminal-color {
  background: lightblue;
  border-radius: 0;
  border: 3px solid #010101;
}

.embarcadero-and\&-mission-rect, .embarcadero-and\&-howard-rect, .market-st-and\&-steuart-st-rect, .embarcadero-and\&-folsom-st-rect {
  stroke: #010101;
}

.terminal-station-color {
  background: lightblue;
  border-radius: 0;
  border: 3px solid #010101;
}

.surface-color {
  background: #D3D3D3;
  border-radius: 0;
  border: 1px dashed #010101;
}

.underground-color {
  background: darkgray;
  border-radius: 0;
  border: 1px dashed #010101;
}

.legend-header {
  color: #010101;
  text-decoration: underline;
  font-weight: 700;
}

.terminal-note {
  color: #010101;
  font-size: 12px;
}

.balboa-park-bart-station-rect, .san-jose-ave-and-geneva-ave-station-rect, .market-st-\&-steuart-st-rect, .embarcadero-\&-mission-rect, .embarcadero-\&-howard-rect, .embarcadero-\&-folsom-st-rect {
  stroke: #010101;
}

.legend-grid-container {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
}

.bottom-legend-right .legend-item {
  font-size: 12px;
}

.legend-grid-container .legend-item {
  width: 50%;
}

.legend-grid-container-bottom-content {
  display: flex;
}

.bottom-legend-left, .bottom-legend-right {
  width: 50%;
}
</style>