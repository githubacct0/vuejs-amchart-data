<template>
  <!-- bottom padding 48px because of fixed footer -->
  <v-container fluid class="pl-6 pr-6 pt-6 pb-12">
    <div class="hello" ref="chartdiv"></div>
  </v-container>
</template>

<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4charts from "@amcharts/amcharts4/charts";
import am4themes_animated from "@amcharts/amcharts4/themes/animated";
import "@amcharts/amcharts4/charts";
am4core.useTheme(am4themes_animated);
export default {
  name: "HelloWorld",
  data: () => ({
    chart: null,
    zoomStartDate: "2015-01-21",
    zoomEndDate: "2021-08-20",
  }),
  props: {
    highLightModule: {
      required: false,
      type: String,
      default: "",
    },
    modelData: {
      required: false,
      type: Array,
      default: () => [],
    },
  },
  methods: {
    drawChart() {
      // Themes begin
      let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
      let data = [];
      let data_1 = this.modelData[0].data.getModelMetrics.cagrReturns; //object_data_1.data.getModelMetrics.cagrReturns;
      let data_2 = this.modelData[1].data.getModelMetrics.cagrReturns; //object_data_2.data.getModelMetrics.cagrReturns;
      for (let i = 0; i < data_1.length; i++) {
        const date_breaker = new Date(data_1[i].effectiveDate); //.toUTCString();
        const value_breaker = data_1[i].spreadReturn;
        data.push({ date1: date_breaker, value1: value_breaker });
      }
      for (let i = 0; i < data_2.length; i++) {
        const date_breaker = new Date(data_2[i].effectiveDate); //.toUTCString();
        const value_breaker = data_2[i].spreadReturn;
        data.push({ date2: date_breaker, value2: value_breaker });
      }
      // eslint-disable-next-line no-redeclare
      // for (var i = 0; i < 360; i++) {
      //   price2 += Math.round((Math.random() < 0.5 ? 1 : -1) * Math.random() * 100);
      //   data.push({ date2: new Date(2017, 0, i), price2: price2 });
      // }

      chart.data = data;

      var dateAxis = chart.xAxes.push(new am4charts.DateAxis());
      dateAxis.renderer.grid.template.location = 0;
      dateAxis.renderer.labels.template.fill = am4core.color("#e59165");

      var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
      valueAxis.tooltip.disabled = true;
      valueAxis.renderer.labels.template.fill = am4core.color("#e59165");

      valueAxis.renderer.minWidth = 60;
      let greyHex = "#808080";
      var series = chart.series.push(new am4charts.LineSeries());
      // series.name = "2015";
      series.dataFields.dateX = "date1";
      series.dataFields.valueY = "value1";
      let modelVersionName =
        this.modelData[0].data.getModelMetrics.modelVersionName;
      series.tooltipText = `${modelVersionName}\nDate:{dateX}\nValue:{valueY}`;
      //series.fill = am4core.color("#e59165");
      //series.stroke = am4core.color("#e59165");
      series.strokeWidth = 2;

      var series2 = chart.series.push(new am4charts.LineSeries());
      // series2.name = "2017";
      series2.dataFields.dateX = "date2";
      series2.dataFields.valueY = "value2";
      let modelVersionName2 =
        this.modelData[1].data.getModelMetrics.modelVersionName;
      series2.tooltipText = `${modelVersionName2}\nDate:{dateX}\nValue:{valueY}`;
      //series2.fill = am4core.color("#dfcc64");
      //series2.stroke = am4core.color("#dfcc64");
      series2.strokeWidth = 2;

    //reduce opacity if highLightModule not matched
    if (this.highLightModule != modelVersionName) {
        series.tooltip.background.opacity = 0.2;
        series.strokeOpacity = 0.3;
        series.fill = am4core.color(greyHex);
        series.stroke = am4core.color(greyHex);
      } else {
        series2.tooltip.background.opacity = 0.2;
        series2.strokeOpacity = 0.3;
        series2.fill = am4core.color(greyHex);
        series2.stroke = am4core.color(greyHex);
      }

      chart.cursor = new am4charts.XYCursor();

      var scrollbarX = new am4charts.XYChartScrollbar();
      scrollbarX.series.push(series);
      chart.scrollbarX = scrollbarX;

      chart.legend = new am4charts.Legend();
      chart.legend.parent = chart.plotContainer;
      chart.legend.zIndex = 100;

      dateAxis.renderer.grid.template.strokeOpacity = 0.07;
      valueAxis.renderer.grid.template.strokeOpacity = 0.07;

      //zoom function need to be developed using veutify buttons
      // on selection of week , month , years need to redredraw chart with zoomed value
      //const zoomStartDate = this.zoomStartDate;
      //const zoomEndDate = this.zoomEndDate;
      chart.events.on("ready", function () {
        //date here will
        //dateAxis.zoomToDates(new Date(zoomStartDate), new Date(zoomEndDate));
      });

      chart.zoomOutButton.align = "left";
      chart.zoomOutButton.valign = "bottom";
      chart.zoomOutButton.marginLeft = 10;
      chart.zoomOutButton.marginBottom = 10;

      this.chart = chart;
    },
  },
  mounted() {
    this.drawChart();
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
