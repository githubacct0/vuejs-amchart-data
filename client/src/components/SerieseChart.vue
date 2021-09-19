<template>
  <div>
    <v-row align="center" justify="center">
      <v-btn-toggle
        v-model="zoomOption"
        @change="zoomOptionSelected"
        tile
        color="primary"
        group
      >
        <v-btn> Week </v-btn>
        <v-btn> Month </v-btn>
        <v-btn> Year </v-btn>
        <v-btn> 2 Years </v-btn>
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
import am4themes_animated from "@amcharts/amcharts4/themes/animated";
import "@amcharts/amcharts4/charts";
am4core.useTheme(am4themes_animated);
export default {
  name: "HelloWorld",
  data: () => ({
    chart: null,
    zoomStartDate: "2015-01-21",
    zoomEndDate: "2021-08-20",
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
      console.log(v);
    },
    drawChart() {
      // Themes begin
      let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
      let data = [];
      let data_1 = this.modelData[0].data.getModelMetrics.cagrReturns; //object_data_1.data.getModelMetrics.cagrReturns;
      let data_2 = this.modelData[1].data.getModelMetrics.cagrReturns; //object_data_2.data.getModelMetrics.cagrReturns;

      for (let i = data_1.length - 1; i >= 0; i--) {
        const date_breaker = new Date(data_1[i].effectiveDate); //.toUTCString();
        const value_breaker = data_1[i].spreadReturn;
        data.push({ date1: date_breaker, value1: value_breaker });
      }
      for (let i = data_2.length - 1; i >= 0; i--) {
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
      // Set date label formatting
      dateAxis.dateFormats.setKey("day", "MMMM dt");
      dateAxis.periodChangeDateFormats.setKey("day", "MMMM dt");
      //dateAxis.renderer.grid.template.location = 0;
      dateAxis.renderer.labels.template.fill = am4core.color("#e59165");
      dateAxis.renderer.minGridDistance = 30;
      

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
      if (this.highLightModule != modelVersionName)
        scrollbarX.series.push(series);
      else scrollbarX.series.push(series2);
      chart.scrollbarX = scrollbarX;
      var scrollbarY = new am4charts.XYChartScrollbar();
      if (this.highLightModule != modelVersionName)
        scrollbarY.series.push(series);
      else scrollbarY.series.push(series2);
      chart.scrollbarY = scrollbarY;

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
