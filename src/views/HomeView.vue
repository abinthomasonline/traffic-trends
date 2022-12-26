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
    async fetchTimeseries(chart) {
      if (chart.sha256 in this.timeseries_dict) {
        this.current_chart = chart.sha256;
        this.chart_type = chart.chart_type;
      }
      else {
        const url = 'data/timeseries/sha256-' + chart.sha256 + '.json';
        const response = await fetch(url);
        const data = await response.json();
        this.timeseries_dict[chart.sha256] = data;
        this.current_chart = chart.sha256;
        this.chart_type = chart.chart_type;
      }
    }, 
    async loadCache(chart_ids) {
      for (let i = 0; i < chart_ids.length; i++) {
        const url = 'data/timeseries/sha256-' + chart_ids[i] + '.json';
        // const response = await fetch(url);
        // const data = await response.json();
        // this.timeseries_dict[chart_ids[i]] = data;
        fetch(url).then(response => response.json()).then(data => {
          this.timeseries_dict[chart_ids[i]] = data;
          console.log('loaded ' + i);
        });
      }
    }
  }, 
  data() {
    return {
      timeseries_dict: {}, 
      current_chart: null, 
      chart_type: null
    }
  }
}
</script>

<template>
  <h1>Traffic Trends</h1>
  <p>tracking road traffic data of indian cities</p>
  <ChartSelector @chart-selected="fetchTimeseries" @load-cache="loadCache"/>
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