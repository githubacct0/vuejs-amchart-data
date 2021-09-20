<template>
  <div>
    <SeriesChart
      :highLightModule="highLightModules"
      :modelData="modelData"
      @drawComplete="emptyModelData"
    />
    <!-- bottom padding 48px because of fixed footer -->
    <!-- <v-container fluid class="pl-6 pr-6 pt-6 pb-12">
      <div class="hello" ref="chartdiv"></div>
    </v-container> -->
  </div>
</template>

<script>
import object_data_1 from "../../../../modelVersionData/290_16.json";
import object_data_2 from "../../../../modelVersionData/291_03.json";
import object_data_3 from "../../../../modelVersionData/291_01.json";
import object_data_4 from "../../../../modelVersionData/290_20.json";
import SeriesChart from "../../components/SerieseChart.vue";

export default {
  name: "HelloWorld",
  data: () => ({
    emptyChartData: false,
  }),
  computed: {
    highLightModules(){
      return ['models/290/versions/16']
    },
    modelData() {
      if (this.emptyChartData) return [];
      return [object_data_1, object_data_2, object_data_3, object_data_4];
    },
  },
  methods: {
    emptyModelData() {
    },
  },
  components: {
    SeriesChart,
  },
  mounted() {
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
