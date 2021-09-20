<template>
  <div>
    <v-layout row>
      <v-flex md12 xs12 sm12>
        <v-select
          v-model="highLightModule"
          label="Select Module To Highlight"
          :items="moduleNamesArray"
          item-text="name"
          item-value="name"
          clearable
          class="ml-16 mr-16 mt-10"
          multiple
          chips
        >
          <template v-slot:selection="{ item }">
            <v-chip
              class="ma-2"
              close
              :color="item.color"
              label
              @click:close="removeSelection(item)"
            >
              <span>{{ item.name }}</span>
            </v-chip>
          </template>
          <template v-slot:item="{ item }">
            <v-list-item-action>
              <v-icon v-if="highLightModule.includes(item.name)"
                >check_box</v-icon
              >
              <v-icon v-else>check_box_outline_blank</v-icon>
            </v-list-item-action>
            <v-list-item-title :style="`background-color:${item.color}`">
              <div class="ma-4">{{ item.name }}</div>
            </v-list-item-title>
          </template>
        </v-select>
      </v-flex>

      <v-flex xs5 sm5>
        <v-btn-toggle
          v-model="zoomOption"
          @change="zoomOptionSelected"
          class="ml-16"
          color="primary"
          group
        >
          <v-btn elevation="1" value="1w"> 1W </v-btn>
          <v-btn elevation="1" value="1m"> 1M </v-btn>
          <v-btn elevation="1" value="3m"> 3M </v-btn>
          <v-btn elevation="1" value="6m"> 6M </v-btn>
          <v-btn elevation="1" value="1y"> 1Y </v-btn>
          <v-btn elevation="1" value="2y"> 2Y </v-btn>
          <!-- <v-btn elevation="1" value="3y"> 3Y </v-btn> -->
          <v-btn elevation="1" value="max"> MAX </v-btn>
        </v-btn-toggle>
      </v-flex>
      <v-flex xs1 sm1>
        <v-menu
          ref="menu"
          v-model="startDateMenu"
          :close-on-content-click="false"
          transition="scale-transition"
          offset-y
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="calendarDate"
              label="From"
              class="mr-1"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker
            v-model="calendarDate"
            no-title
            scrollable
            @change="rangeDateSelected"
            @input="setStartDate"
          >
          </v-date-picker>
        </v-menu>
      </v-flex>
      <v-flex xs1 sm1>
        <v-menu
          ref="menu1"
          v-model="endDateMenu"
          :close-on-content-click="false"
          transition="scale-transition"
          offset-y
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="calendarEndDate"
              label="To"
              readonly
              :disabled="!calendarDate"
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker
            :min="calendarDate"
            @change="rangeDateSelected"
            @input="setEndDate"
            no-title
            scrollable
          >
          </v-date-picker>
        </v-menu>
      </v-flex>
    </v-layout>

    <!-- bottom padding 48px because of fixed footer -->
    <v-container fluid class="pl-6 pr-6 pt-6 pb-12">
      <div class="hello" style="height: 650px" ref="chartdiv"></div>
    </v-container>
  </div>
</template>

<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4charts from "@amcharts/amcharts4/charts";
export default {
  name: "HelloWorld",
  data: () => ({
    chart: null,
    highLightModuleData: null,
    highLightModule: ["models/290/versions/16"],
    seriesCreated: [],
    dateAxis: null,
    zoomOption: "",
    calendarDate: null,
    calendarEndDate: null,
    startDateMenu: false,
    endDateMenu: false,
  }),
  props: {
    modelData: {
      required: false,
      type: Array,
      default: () => [],
    },
  },
  computed: {
    moduleNamesArray() {
      let colorsToPickFrom = [
        "#0064ba",
        "#69cb69",
        "#ff33cc",
        "#bb5900",
        "#f5cb42",
        "#f5cb42",
      ];
      return this.modelData.map((x, index) => ({
        name: x.data.getModelMetrics.modelVersionName,
        color: colorsToPickFrom[index],
      }));
    },
    rangeMinDate() {
      if (!this.highLightModuleData) return;
      let cagrReturns =
        this.highLightModuleData.data.getModelMetrics.cagrReturns;
      return cagrReturns[cagrReturns.length - 1].effectiveDate;
    },
    rangeMaxDate() {
      if (!this.highLightModuleData) return;
      let cagrReturns =
        this.highLightModuleData.data.getModelMetrics.cagrReturns;
      return cagrReturns[0].effectiveDate;
    },
  },
  methods: {
    removeSelection(item) {
      let index = this.highLightModule.indexOf(item.name);
      this.highLightModule.splice(index, 1);
    },
    setStartDate(date) {
      this.startDateMenu = false;
      this.calendarDate = date;
    },
    setEndDate(date) {
      this.endDateMenu = false;
      this.calendarEndDate = date;
    },
    rangeDateSelected() {
      if (this.calendarDate && this.calendarEndDate)
        this.dateAxis.zoomToDates(
          new Date(this.calendarDate),
          new Date(this.calendarEndDate)
        );
    },
    zoomOptionSelected(v) {
      this.calendarDate = null;
      this.calendarEndDate = null;
      if (!this.highLightModuleData) return;
      let cagrReturns =
        this.highLightModuleData.data.getModelMetrics.cagrReturns;
      let today = new Date();
      let startDate = new Date();
      switch (v) {
        case "1w":
          startDate.setDate(startDate.getDate() - 7);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "1m":
          startDate.setMonth(startDate.getMonth() - 1);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "3m":
          startDate.setMonth(startDate.getMonth() - 3);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "6m":
          startDate.setMonth(startDate.getMonth() - 6);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "1y":
          startDate.setFullYear(startDate.getFullYear() - 1);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "2y":
          startDate.setFullYear(startDate.getFullYear() - 2);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "3y":
          startDate.setFullYear(startDate.getFullYear() - 3);
          this.dateAxis.zoomToDates(startDate, today);
          break;
        case "max":
          this.dateAxis.zoomToDates(
            new Date(cagrReturns[0].effectiveDate),
            today
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
      this.highLightModuleData = this.modelData[0];
      this.modelData.forEach((modelData, index) => {
        let cagrReturns = modelData.data.getModelMetrics.cagrReturns;
        cagrReturns = cagrReturns.sort(function (a, b) {
          let aDate = new Date(a.effectiveDate);
          let bDate = new Date(b.effectiveDate);
          return aDate.getTime() - bDate.getTime();
        });
        console.log("sort", cagrReturns);
        for (let i = 0; i < cagrReturns.length; i++) {
          const date_breaker = new Date(cagrReturns[i].effectiveDate); //.toUTCString();
          const value_breaker = cagrReturns[i].spreadReturn;
          data.push({
            ["date" + index + 1]: date_breaker,
            ["value" + index + 1]: this.RoundNumber(value_breaker),
          });
        }
      });

      chart.data = data;

      var dateAxis = chart.xAxes.push(new am4charts.DateAxis());
      // Set date label formatting
      //dateAxis.dateFormats.setKey("day", "MMM dt");
      //dateAxis.periodChangeDateFormats.setKey("day", "MMM dt");
      dateAxis.renderer.grid.template.location = 0;
      dateAxis.renderer.minGridDistance = 50;
      dateAxis.renderer.labels.template.fill = am4core.color("#e59165");
      

      var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
      valueAxis.tooltip.disabled = true;
      valueAxis.renderer.labels.template.fill = am4core.color("#e59165");

      let colorsToPickFrom = [
        "#0064ba",
        "#69cb69",
        "#ff33cc",
        "#bb5900",
        "#f5cb42",
        "#f5cb42",
      ];
      this.modelData.forEach((data, index) => {
        var series = chart.series.push(new am4charts.LineSeries());
        series.dataFields.dateX = "date" + index + 1;
        series.dataFields.valueY = "value" + index + 1;
        let versionName = data.data.getModelMetrics.modelVersionName;
        series.tooltipText = `{valueY}`;
        series.strokeWidth = 3;
        if (colorsToPickFrom[index]) {
          series.fill = am4core.color(colorsToPickFrom[index]);
          series.stroke = am4core.color(colorsToPickFrom[index]);
        }
        //reduce opacity if highLightModule not matched
        if (!this.highLightModule.includes(versionName)) {
          series.tooltip.background.opacity = 0.4;
          series.strokeOpacity = 0.4;
          series.strokeWidth = 1;
        }
        this.seriesCreated.push({ series: series, name: versionName });
      });

      chart.cursor = new am4charts.XYCursor();

      this.addChartGrips(chart);

      dateAxis.renderer.grid.template.strokeOpacity = 0.07;
      valueAxis.renderer.grid.template.strokeOpacity = 0.07;
      this.dateAxis = dateAxis;

      chart.svgContainer.autoResize = false;

      this.chart = chart;
    },
    addChartGrips(chart) {
      // Add scrollbar
      chart.scrollbarX = new am4core.Scrollbar();
      let gripFillColorHex = "#c1c1c1";
      let strokeColorHex = "#8b8b8b";
      chart.scrollbarX.startGrip.background.fill =
        am4core.color(gripFillColorHex);
      chart.scrollbarX.endGrip.background.fill =
        am4core.color(gripFillColorHex);
      chart.scrollbarX.thumb.background.fill = am4core.color(gripFillColorHex);

      chart.scrollbarX.startGrip.icon.stroke = am4core.color(strokeColorHex);
      chart.scrollbarX.endGrip.icon.stroke = am4core.color(strokeColorHex);

      let hoverColorHex = "#8b8b8b";
      let gripColorHex = "#727272";
      // Applied on hover
      chart.scrollbarX.startGrip.background.states.getKey(
        "hover"
      ).properties.fill = am4core.color(hoverColorHex);
      chart.scrollbarX.endGrip.background.states.getKey(
        "hover"
      ).properties.fill = am4core.color(hoverColorHex);
      chart.scrollbarX.thumb.background.states.getKey("hover").properties.fill =
        am4core.color(hoverColorHex);

      // Applied on mouse down
      chart.scrollbarX.startGrip.background.states.getKey(
        "down"
      ).properties.fill = am4core.color(gripColorHex);
      chart.scrollbarX.endGrip.background.states.getKey(
        "down"
      ).properties.fill = am4core.color(gripColorHex);
      chart.scrollbarX.thumb.background.states.getKey("down").properties.fill =
        am4core.color(gripColorHex);
      //chart scroll end

      // Add scrollbar
      chart.scrollbarY = new am4core.Scrollbar();

      chart.scrollbarY.startGrip.background.fill =
        am4core.color(gripFillColorHex);
      chart.scrollbarY.endGrip.background.fill =
        am4core.color(gripFillColorHex);
      chart.scrollbarY.thumb.background.fill = am4core.color(gripFillColorHex);

      chart.scrollbarY.startGrip.icon.stroke = am4core.color(strokeColorHex);
      chart.scrollbarY.endGrip.icon.stroke = am4core.color(strokeColorHex);

      // Applied on hover
      chart.scrollbarY.startGrip.background.states.getKey(
        "hover"
      ).properties.fill = am4core.color(hoverColorHex);
      chart.scrollbarY.endGrip.background.states.getKey(
        "hover"
      ).properties.fill = am4core.color(hoverColorHex);
      chart.scrollbarY.thumb.background.states.getKey("hover").properties.fill =
        am4core.color(hoverColorHex);

      // Applied on mouse down
      chart.scrollbarY.startGrip.background.states.getKey(
        "down"
      ).properties.fill = am4core.color(gripColorHex);
      chart.scrollbarY.endGrip.background.states.getKey(
        "down"
      ).properties.fill = am4core.color(gripColorHex);
      chart.scrollbarY.thumb.background.states.getKey("down").properties.fill =
        am4core.color(gripColorHex);
      //chart scroll end
    },

    // grips for y axis
  },
  mounted() {
    this.drawChart();
  },
  watch: {
    highLightModule(modules) {
      this.seriesCreated.forEach((seriesObject) => {
        if (modules.includes(seriesObject.name)) {
          seriesObject.series.tooltip.background.opacity = 1;
          seriesObject.series.strokeOpacity = 1;
          seriesObject.series.strokeWidth = 3;
        } else {
          seriesObject.series.tooltip.background.opacity = 0.4;
          seriesObject.series.strokeOpacity = 0.4;
          seriesObject.series.strokeWidth = 1;
        }
      });
    },
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  },
};
</script>

