<template>
<div>
  <!-- Title -->
  <v-container>
      <v-row
          align="center"
          justify="center"
        >
      <h4>National Investment in Education in 2016 (% GDP) </h4>
      </v-row>
  </v-container>

<!-- choropleth map of national investment in education (compare western europe and eastern asia)-->
  <l-map
  :center="[50.8182, 40.2275]" 
  :zoom="3" 
  :maxZoom="4"
  :minZoom="2.5"
  style="height: 500px;" 
  :options="mapOptions">
    <l-choropleth-layer 
      :data="nationalInvestData" 
      :title="titleSetting"
      titleKey="time" 
      idKey="country_name" 
      :value="value" 
      :extraValues="extraValues" 
      geojsonIdKey="name" 
      :geojson="geojson" 
      :colorScale="colorScale">
        <template slot-scope="props">
          <l-info-control 
            :item="props.currentItem" 
            :unit="props.unit" 
            title="National Investment" 
            placeholder="Hover over a country"/>
          <l-reference-chart 
            title="National Investment" 
            :colorScale="colorScale" 
            :min="props.min" 
            :max="props.max" 
            position="topright"/>
        </template>
    </l-choropleth-layer>
</l-map>

<b-card  border-variant="light" class="text-left">
  According to the suggested benchmark, most of the chosen countries reach the given educational commitment. Besides, countries in 
  North Europe performs quite well with around 7% GDP's devotion or even higher. 
</b-card>
        

<!-- chart of average salary VS family expenditure on education -->
<apexchart type="bar" height="440" :options="chartOptions" :series="series"></apexchart>
<br/>Note: 1) HK* is Hong Kong, China; UAE* is United Arab Emirates; China* is mainland, China.
<br/>2) Family expenditure covers from the pre-primary to the tertiary level.
</div>
</template>

<script>
import 'leaflet/dist/leaflet.css';
import {LMap} from 'vue2-leaflet';
import { InfoControl, ReferenceChart, ChoroplethLayer } from 'vue-choropleth';
import {geojson} from '../../assets/json/countries_smallv';
import {nationalInvestData,family_exp,avg_salary} from '../../assets/json/reason1';
import VueApexCharts from 'vue-apexcharts';

export default {
    components: { 
    LMap,
    'l-info-control': InfoControl, 
    'l-reference-chart': ReferenceChart, 
    'l-choropleth-layer': ChoroplethLayer,
    apexchart: VueApexCharts
  },
  data() {
    return {
      geojson, // polygon of countries
      nationalInvestData,
      family_exp,
      avg_salary,
      // settings for choropleth map
      colorScale: ['rgb(255,204,102)','rgb(204,153,48)','rgb(204,0,0)','rgb(153,51,51)',],
      value: {
        key: "invest",
        metric: "% GDP"
      },
      extraValues: [{
        key: "country_name",
        metric: ""
      }],
      mapOptions: {
        attributionControl: false
      },
      currentStrokeColor: '3d3213',
      type: "stackedbar2d",
      width: "100%",
      height: "100%",
      dataFormat: "json",
      // settings for chart
      series: [{
            name: 'Family Expenditure',
            data: family_exp
          },
          {
            name: 'Average Salary',
            data: avg_salary
          }
          ],
          chartOptions: {
            chart: {
              type: 'bar',
              height: 440,
              stacked: true,
              toolbar: {
                show: false
              }
            },
            colors: ["#6F6F6E", "#FFC300"],
            plotOptions: {
              bar: {
                horizontal: true,
                barHeight: '80%',
            
              },
            },
            dataLabels: {
              enabled: false
            },
            stroke: {
              width: 1,
              colors: ["#fff"]
            },
            
            grid: {
              xaxis: {
                lines: {
                  show: false
                }
              }
            },
            yaxis: {
              min: -132200,
              max: 132200,
              title: {
                // text: 'country',
              },
            },
            tooltip: {
              shared: false,
              x: {
                formatter: function (val) {
                  return val
                }
              },
              y: {
                formatter: function (val) {
                  return "$ "+Math.abs(val)
                }
              }
            },
            title: {
              text: 'Family Expenditure on Education VS Yearly Average Salary',
              style: {
                fontSize: '15px'
              },
            },
            xaxis: {
              categories: ['HK*', 'UAE*','Singapore','USA','China*','Australia','Malaysia','UK','Mexico','Canada','India',
              'Indonesia','Egypt','France'
              ],
              title: {
                text: '$ USD'
              },
              labels: {
                formatter: function (val) {
                  return "$ " + Math.abs(Math.round(val)) 
                }
              }
            },
          },
          
    }
  },
}
</script>


<style>

</style>