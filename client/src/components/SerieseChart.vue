<template>
  <div>
    <v-layout>
      <v-flex xs6 sm6>
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
      </v-flex>
      <v-flex xs2 sm2>
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
              prepend-icon="mdi-calendar"
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
      <v-flex xs2 sm2>
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
              prepend-icon="mdi-calendar"
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
      <v-flex xs2 sm2></v-flex>
    </v-layout>

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
    highLightModuleData: null,
    dateAxis: null,
    zoomOption: "",
    calendarDate: null,
    calendarEndDate: null,
    startDateMenu: false,
    endDateMenu: false,
  }),
  props: {
    highLightModule: {
      required: false,
      type: Array,
      default: () => [],
    },
    modelData: {
      required: false,
      type: Array,
      default: () => [],
    },
  },
  computed: {
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
      this.highLightModuleData = this.modelData[0];
      this.modelData.forEach((modelData, index) => {
        let cagrReturns = modelData.data.getModelMetrics.cagrReturns;
        for (let i = cagrReturns.length - 1; i >= 0; i--) {
          const date_breaker = new Date(cagrReturns[i].effectiveDate); //.toUTCString();
          const value_breaker = cagrReturns[i].spreadReturn;
          data.push({
            ["date" + index + 1]: date_breaker,
            ["value" + index + 1]: this.RoundNumber(value_breaker),
          });
        }
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
      dateAxis.renderer.minGridDistance = 60;

      var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
      valueAxis.tooltip.disabled = true;
      valueAxis.renderer.labels.template.fill = am4core.color("#e59165");

      //valueAxis.renderer.minWidth = 200;
      let colorsToPickFrom = [
        "#0064ba",
        "#69cb69",
        "#ffa72",
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
          //   series.tooltip.background.opacity = 0.2;
          //   series.strokeOpacity = 0.2;
          series.strokeWidth = 1;
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

      //   var scrollbarX = new am4charts.XYChartScrollbar();
      //   if (this.highLightModule == modelVersionName)
      //     scrollbarX.series.push(selectedSeeries);
      //   chart.scrollbarX = scrollbarX;
      //   var scrollbarY = new am4charts.XYChartScrollbar();
      //   if (this.highLightModule == modelVersionName)
      //     scrollbarY.series.push(selectedSeeries);
      //   chart.scrollbarY = scrollbarY;

      this.addChartGrips(chart);
      //chart scroll

      chart.legend = new am4charts.Legend();
      chart.legend.parent = chart.plotContainer;
      chart.legend.zIndex = 100;

      dateAxis.renderer.grid.template.strokeOpacity = 0.07;
      valueAxis.renderer.grid.template.strokeOpacity = 0.07;
      this.dateAxis = dateAxis;
      //zoom function need to be developed using veutify buttons
      // on selection of week , month , years need to redredraw chart with zoomed value
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
      this.$emit("drawComplete");
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
