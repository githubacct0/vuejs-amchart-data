<template>
  <div>
    <v-row align="center" justify="center">
      <v-btn-toggle
        v-model="zoomOption"
        @change="zoomOptionSelected"
        color="primary"
        group
      >
        <v-btn elevation="1" value="1w"> 1W </v-btn>
        <v-btn elevation="1" value="1m"> 1M </v-btn>
        <v-btn elevation="1" value="3m"> 3M </v-btn>
        <v-btn elevation="1" value="6m"> 6M </v-btn>
        <v-btn elevation="1" value="1y"> 1Y </v-btn>
        <v-btn elevation="1" value="2y"> 2Y </v-btn>
        <v-btn elevation="1" value="3y"> 3Y </v-btn>
        <v-btn elevation="1" value="max"> MAX </v-btn>
      </v-btn-toggle>
      <v-col cols="12" sm="6" md="4">
        <v-menu
          ref="menu"
          v-model="menu"
          :close-on-content-click="false"
          :return-value.sync="date"
          transition="scale-transition"
          offset-y
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="date"
              label="Picker in menu"
              prepend-icon="mdi-calendar"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker v-model="date" no-title scrollable range>
            <v-spacer></v-spacer>
            <v-btn text color="primary" @click="menu = false"> Cancel </v-btn>
            <v-btn text color="primary" @click="$refs.menu.save(date)">
              OK
            </v-btn>
          </v-date-picker>
        </v-menu>
      </v-col>
    </v-row>
    <!-- bottom padding 48px because of fixed footer -->
    <v-container fluid class="pl-6 pr-6 pt-6 pb-12">
      <div class="hello" ref="chartdiv"></div>
    </v-container>
  </div>
</template>

<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4charts from "@amcharts/amcharts4/charts";
//import am4themes_animated from "@amcharts/amcharts4/themes/animated";
//import "@amcharts/amcharts4/charts";
//am4core.useTheme(am4themes_animated);
export default {
  name: "HelloWorld",
  data: () => ({
    chart: null,
    zoomStartDate: "2015-01-21",
    zoomEndDate: "2021-08-20",
    highLightModuleData: null,
    dateAxis: null,
    zoomOption: "",
    date: null,
    menu: false,
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
    zoomOptionSelected(v) {
      if (!this.highLightModuleData) return;
      let cagrReturns =
        this.highLightModuleData.data.getModelMetrics.cagrReturns;
      let startDate = new Date(
        cagrReturns[cagrReturns.length - 1].effectiveDate
      );
      let endDate = new Date(cagrReturns[cagrReturns.length - 1].effectiveDate);
      switch (v) {
        case "1w":
          endDate.setDate(endDate.getDate() + 7);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "1m":
          endDate.setMonth(endDate.getMonth() + 1);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "3m":
          endDate.setMonth(endDate.getMonth() + 3);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "6m":
          endDate.setMonth(endDate.getMonth() + 6);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "1y":
          endDate.setFullYear(endDate.getFullYear() + 1);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "2y":
          endDate.setFullYear(endDate.getFullYear() + 2);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "3y":
          endDate.setFullYear(endDate.getFullYear() + 3);
          this.dateAxis.zoomToDates(startDate, endDate);
          break;
        case "max":
          this.dateAxis.zoomToDates(
            startDate,
            new Date(cagrReturns[0].effectiveDate)
          );

          break;
      }
    },
    RoundNumber(num, decimalPlaces = 2) {
      var p = Math.pow(10, decimalPlaces);
      return Math.round(num * p) / p;
    },
    drawChart() {
      // Themes begin
      let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
      let data = [];
      //let data_1 = this.modelData[0].data.getModelMetrics.cagrReturns; //object_data_1.data.getModelMetrics.cagrReturns;
      //let data_2 = this.modelData[1].data.getModelMetrics.cagrReturns; //object_data_2.data.getModelMetrics.cagrReturns;

      this.modelData.forEach((modelData, index) => {
        let cagrReturns = modelData.data.getModelMetrics.cagrReturns;
        let versionName = modelData.data.getModelMetrics.modelVersionName;
        for (let i = cagrReturns.length - 1; i >= 0; i--) {
          const date_breaker = new Date(cagrReturns[i].effectiveDate); //.toUTCString();
          const value_breaker = cagrReturns[i].spreadReturn;
          data.push({
            ["date" + index + 1]: date_breaker,
            ["value" + index + 1]: this.RoundNumber(value_breaker),
          });
        }
        if (this.highLightModule == versionName)
          this.highLightModuleData = modelData;
      });

      //   for (let i = data_2.length - 1; i >= 0; i--) {
      //     const date_breaker = new Date(data_2[i].effectiveDate); //.toUTCString();
      //     const value_breaker = data_2[i].spreadReturn;
      //     data.push({ date2: date_breaker, value2: value_breaker });
      //   }
      // eslint-disable-next-line no-redeclare
      // for (var i = 0; i < 360; i++) {
      //   price2 += Math.round((Math.random() < 0.5 ? 1 : -1) * Math.random() * 100);
      //   data.push({ date2: new Date(2017, 0, i), price2: price2 });
      // }

      chart.data = data;

      var dateAxis = chart.xAxes.push(new am4charts.DateAxis());
      // Set date label formatting
      dateAxis.dateFormats.setKey("day", "MMM dt");
      dateAxis.periodChangeDateFormats.setKey("day", "MMM dt");
      //dateAxis.renderer.grid.template.location = 0;
      dateAxis.renderer.labels.template.fill = am4core.color("#e59165");
      dateAxis.renderer.minGridDistance = 30;

      var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
      valueAxis.tooltip.disabled = true;
      valueAxis.renderer.labels.template.fill = am4core.color("#e59165");

      valueAxis.renderer.minWidth = 60;
      //let greyHex = "#808080";
      let modelVersionName;
      let selectedSeeries;
      this.modelData.forEach((data, index) => {
        var series = chart.series.push(new am4charts.LineSeries());
        series.dataFields.dateX = "date" + index + 1;
        series.dataFields.valueY = "value" + index + 1;
        let versionName = data.data.getModelMetrics.modelVersionName;
        series.tooltipText = `{valueY}`;
        series.strokeWidth = 3;
        //reduce opacity if highLightModule not matched
        if (this.highLightModule != versionName) {
          series.tooltip.background.opacity = 0.2;
          series.strokeOpacity = 0.2;
          series.strokeWidth = 1;
          //series.fill = am4core.color(greyHex);
          //series.stroke = am4core.color(greyHex);
        } else {
          selectedSeeries = series;
          modelVersionName = versionName;
        }
      });

      //   var series = chart.series.push(new am4charts.LineSeries());
      //   // series.name = "2015";
      //   series.dataFields.dateX = "date1";
      //   series.dataFields.valueY = "value1";
      //   let modelVersionName =
      //     this.modelData[0].data.getModelMetrics.modelVersionName;
      //   series.tooltipText = `${modelVersionName}\nDate:{dateX}\nValue:{valueY}`;
      //   //series.fill = am4core.color("#e59165");
      //   //series.stroke = am4core.color("#e59165");
      //   series.strokeWidth = 2;

      //   var series2 = chart.series.push(new am4charts.LineSeries());
      //   // series2.name = "2017";
      //   series2.dataFields.dateX = "date2";
      //   series2.dataFields.valueY = "value2";
      //   let modelVersionName2 =
      //     this.modelData[1].data.getModelMetrics.modelVersionName;
      //   series2.tooltipText = `${modelVersionName2}\nDate:{dateX}\nValue:{valueY}`;
      //   //series2.fill = am4core.color("#dfcc64");
      //   //series2.stroke = am4core.color("#dfcc64");
      //   series2.strokeWidth = 2;

      chart.cursor = new am4charts.XYCursor();

      var scrollbarX = new am4charts.XYChartScrollbar();
      if (this.highLightModule == modelVersionName)
        scrollbarX.series.push(selectedSeeries);
      chart.scrollbarX = scrollbarX;
      var scrollbarY = new am4charts.XYChartScrollbar();
      if (this.highLightModule == modelVersionName)
        scrollbarY.series.push(selectedSeeries);
      chart.scrollbarY = scrollbarY;

      chart.legend = new am4charts.Legend();
      chart.legend.parent = chart.plotContainer;
      chart.legend.zIndex = 100;

      dateAxis.renderer.grid.template.strokeOpacity = 0.07;
      valueAxis.renderer.grid.template.strokeOpacity = 0.07;
      this.dateAxis = dateAxis;
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
      //chart options
      //   chart.marginRight = 40;
      //   chart.marginLeft = 40;
      //   chart.autoMarginOffset = 20;
      //   chart.mouseWheelZoomEnabled = true;
      //   chart.dataDateFormat = "YYYY-MM-DD";

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
