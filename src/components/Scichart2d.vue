
<template>
    <div class="hello">
      <div id="scichart-root" style="width: 600px; height: 400px; margin: auto;"></div>
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent } from "vue";
  import {
    SciChartSurface,
    NumericAxis,
    FastLineRenderableSeries,
    XyDataSeries,
    EllipsePointMarker,
    SweepAnimation,
    SciChartJsNavyTheme,
    NumberRange,
    MouseWheelZoomModifier,
    ZoomPanModifier,
    ZoomExtentsModifier
  } from "scichart";
  let chartXValues=[];
  let chartYValues=[];
  function fetchData() {
      fetch('https://rest.statica.pl/rest/stockquotes/gpw:PLKGHM000017?type=trades&dt_from=2010-01-01&limit=10000', {
        method: "GET",
        headers: {
          "Authorization" : 'Basic '+btoa('frontend2024:test')
        },
      })
        .then((response) => {
          response.json().then((data) => {
           for(let i = 0 ; i < data.length;i++){
            chartXValues.push(data[i].price);
            chartYValues.push(data[i].dt);
           }
          });
        })
        .catch((err) => {
          console.error(err);
        });
        return  {chartXValues,chartYValues} ;
      }
     
    
  async function initSciChart() {
   
    const { sciChartSurface, wasmContext } = await SciChartSurface.create("scichart-root", {
      theme: new SciChartJsNavyTheme(),
      title: "REC TEST",
      titleStyle: { fontSize: 16 }
    });
    const growBy = new NumberRange(0.1, 0.1);
    sciChartSurface.xAxes.add(new NumericAxis(wasmContext, { axisTitle: "X Axis", growBy }));
    sciChartSurface.yAxes.add(new NumericAxis(wasmContext, { axisTitle: "Y Axis", growBy }));
    sciChartSurface.renderableSeries.add(new FastLineRenderableSeries(wasmContext, {
      stroke: "steelblue",
      strokeThickness: 3,
      dataSeries: new XyDataSeries(wasmContext, {
        xValues: fetchData().chartXValues,
      yValues: fetchData().chartYValues,
    }),
    pointMarker: new EllipsePointMarker(wasmContext, { width: 11, height: 11, fill: "#fff" }),
    animation: new SweepAnimation({ duration: 300, fadeEffect: true })
  }));
  sciChartSurface.chartModifiers.add(new MouseWheelZoomModifier(), new ZoomPanModifier(), new ZoomExtentsModifier());
    return sciChartSurface;
  }

  export default defineComponent({
    data() {
      return {
        chartInitializationPromise: undefined,
      };
    },
    mounted() {
      console.log("SciChart2d.vue onMounted");
      this.chartInitializationPromise = initSciChart();
    },
    beforeUnmount() {
      console.log("SciChart2d.vue beforeUnmount");
      this.chartInitializationPromise.then((sciChartSurface) => {
        console.log("..deleting sciChartSurface");
        sciChartSurface.delete();
      });
    },
    name: "Scichart2d",
   
  });
  </script>
  
  <style scoped>
  h3 {
    margin: 40px 0 0;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }
  </style>