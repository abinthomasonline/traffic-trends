<script>
import ChartCanvas from '@/components/ChartCanvas.vue'
import ChartSelector from '@/components/ChartSelector.vue'

export default {
  name: 'HomeView',
  components: {
    ChartCanvas,
    ChartSelector
  }, 
  methods: {
    async updateTimeseries(chart) {
      this.current_chart = chart.sha256;
      this.chart_type = chart.chart_type;
    }, 
    async loadCache() {
      const url = 'data/timeseries.json';
      const response = await fetch(url);
      const data = await response.json();
      this.timeseries_dict = data;
    }
  }, 
  data() {
    return {
      timeseries_dict: {}, 
      current_chart: null, 
      chart_type: null
    }
  }, 
  created() {
    this.loadCache();
  }
}
</script>

<template>
  <h1>Traffic Trends</h1>
  <p>tracking road traffic data of indian cities</p>
  <ChartSelector @chart-selected="updateTimeseries"/>
  <ChartCanvas v-if="current_chart" :timeseries="timeseries_dict[current_chart]" :chart_type="chart_type"/>
</template>

<style scoped>
  h1 {
    text-align: center;
    color: #6c757d;
    font-weight: 700;
    margin-bottom: 10px;
  }
  p {
    text-align: center;
    color: #6c757d;
    font-weight: 700;
    margin-top: 0;
    margin-bottom: 32px;
  }
</style>