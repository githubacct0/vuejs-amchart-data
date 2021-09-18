<template>
  <div>
    <SeriesChart highLightChart="chartname" :modelData="modelData" />
    <!-- bottom padding 48px because of fixed footer -->
    <!-- <v-container fluid class="pl-6 pr-6 pt-6 pb-12">
      <div class="hello" ref="chartdiv"></div>
    </v-container> -->
  </div>
</template>

<script>
// import * as am4core from "@amcharts/amcharts4/core";
// import * as am4charts from "@amcharts/amcharts4/charts";
// import am4themes_animated from "@amcharts/amcharts4/themes/animated";
import object_data_1 from "../../../../modelVersionData/290_16.json";
import object_data_2 from "../../../../modelVersionData/291_03.json";
// am4core.useTheme(am4themes_animated);
import SeriesChart from '../../components/SerieseChart.vue';

export default {
  name: "HelloWorld",
  computed:{
    modelData(){
      return [object_data_1,object_data_2];
    }
  },
  components:{
    SeriesChart
    },
  mounted() {
    // let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
    // let data = [];
    // let data_1 = object_data_1.data.getModelMetrics.cagrReturns;
    // let data_2 = object_data_2.data.getModelMetrics.cagrReturns;
    // for (let i = 0; i < data_1.length; i++) {
    //   const date_breaker = new Date(data_1[i].effectiveDate).toUTCString();
    //   const value_breaker = data_1[i].spreadReturn;
    //   data.push({ date1: date_breaker, value1: value_breaker });
    // }
    // for (let i = 0; i < data_2.length; i++) {
    //   const date_breaker = new Date(data_2[i].effectiveDate).toUTCString();
    //   const value_breaker = data_2[i].spreadReturn;
    //   data.push({ date2: date_breaker, value2: value_breaker });
    // }

    // // eslint-disable-next-line no-redeclare
    // // for (var i = 0; i < 360; i++) {
    // //   price2 += Math.round((Math.random() < 0.5 ? 1 : -1) * Math.random() * 100);
    // //   data.push({ date2: new Date(2017, 0, i), price2: price2 });
    // // }

    // chart.data = data;

    // var dateAxis = chart.xAxes.push(new am4charts.DateAxis());
    // dateAxis.renderer.grid.template.location = 0;
    // dateAxis.renderer.labels.template.fill = am4core.color("#e59165");

    // var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
    // valueAxis.tooltip.disabled = true;
    // valueAxis.renderer.labels.template.fill = am4core.color("#e59165");

    // valueAxis.renderer.minWidth = 60;

    // var series = chart.series.push(new am4charts.LineSeries());
    // // series.name = "2015";
    // series.dataFields.dateX = "date1";
    // series.dataFields.valueY = "value1";
    // series.tooltipText = "{valueY.value}";
    // series.fill = am4core.color("#e59165");
    // series.stroke = am4core.color("#e59165");
    // series.strokeWidth = 3;

    // var series2 = chart.series.push(new am4charts.LineSeries());
    // // series2.name = "2017";
    // series2.dataFields.dateX = "date2";
    // series2.dataFields.valueY = "value2";
    // series2.tooltipText = "{valueY.value}";
    // series2.fill = am4core.color("#dfcc64");
    // series2.stroke = am4core.color("#dfcc64");
    // series2.strokeWidth = 3;

    // chart.cursor = new am4charts.XYCursor();

    // var scrollbarX = new am4charts.XYChartScrollbar();
    // scrollbarX.series.push(series);
    // chart.scrollbarX = scrollbarX;

    // chart.legend = new am4charts.Legend();
    // chart.legend.parent = chart.plotContainer;
    // chart.legend.zIndex = 100;

    // dateAxis.renderer.grid.template.strokeOpacity = 0.07;
    // valueAxis.renderer.grid.template.strokeOpacity = 0.07;

    // this.chart = chart;
  },

  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  },
};
</script>
<style scoped>
.hello {
  width: 100%;
  height: 500px;
}
</style>
