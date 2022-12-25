<template>
	<div class="container">
		<h4>{{chart_title}}</h4>
		<GChart
			:type="type"
			:data="data"
			:options="options"
		/>
	</div>
</template>

<script>
	import { GChart } from 'vue-google-charts';

	export default {
		name: 'ChartCanvas', 
		props: {
			timeseries: Array, 
			chart_type: String
		},
		components: {
			GChart
		},
		data() {
			return {
				type: 'LineChart'
			}
		}, 
		computed: {
			data() {
				if (this.chart_type === 'City') {
					return [[
						{label: 'Date', role: 'domain'}, 
						{role: 'tooltip', type: 'string', 'p': {html: true}}, 
						{label: 'Average Driving Speed', role: 'data'}, 
					]].concat(
						this.timeseries.map((row) => [
							new Date(row.timestamp + "Z"), 
							"<div>" + ((row.distance/1000)/(row.duration_in_traffic/60/60)).toFixed(2) + ' km/h - ' + new Date(row.timestamp + "Z").toTimeString().slice(0, 5) + ' ' + new Date(row.timestamp + "Z").toDateString().slice(4, -5) + "</div>", 
							(row.distance/1000)/(row.duration_in_traffic/60/60), 
						]))
				}
				else {
					return [[
						{label: 'Date', role: 'domain'}, 
						{role: 'tooltip', type: 'string', 'p': {'html': true}}, 
						{label: 'Driving Time', role: 'data'}, 
					]].concat(
						this.timeseries.map((row) => [
							new Date(row.timestamp + "Z"), 
							"<div>" + (row.duration_in_traffic/60).toFixed(0) + ' min - ' + new Date(row.timestamp + "Z").toTimeString().slice(0, 5) + ' ' + new Date(row.timestamp + "Z").toDateString().slice(4, -5) + "</div>", 
							row.duration_in_traffic/60 | 0, 
						]))
				}
				
			}, 
			chart_title() {
				if (this.chart_type === 'City') {
					return 'Average Driving Speed (km/h)'
				}
				else {
					return 'Driving Time (min)'
				}
			}, 
			options() {
				const base_json = {
					animation: {
						duration: 300,
						easing: 'inAndOut',
					},
					backgroundColor: 'transparent', 
					chartArea: {
						left: '7%',
						right: 0,
						width: '90%',
					},
					crosshair: {
						trigger: 'focus', 
						color: 'white', 
						orientation: 'vertical', 
						color: '#6c757d',
					},
					curveType: 'function',
					focusTarget: 'category',
					hAxis: {
						format: 'd MMM', 
						gridlines: {
							color: 'transparent', 
						}, 
						textStyle: {
							color: '#6c757d', 
						},
					},
					legend: 'none',
					tooltip: {
						trigger: 'both',
						isHtml: true,
					},
					vAxis: {
						gridlines: {
							color: '#25292B', 
							count: 5,
						}, 
						minorGridlines: {
							color: 'transparent'
						}, 
						textStyle: {
							color: '#6c757d', 
						}
					}, 
					width: '100%', 
					trendlines: { 0: {
						type: 'polynomial',
						degree: 1,
					} },
				}
				if (this.chart_type === 'City') {
					const speed_array = this.timeseries.map((row) => (row.distance/1000)/(row.duration_in_traffic/60/60))
					const first_half_mean = speed_array.slice(0, speed_array.length/2).reduce((a, b) => a + b, 0)/speed_array.length/2
					const second_half_mean = speed_array.slice(speed_array.length/2, speed_array.length).reduce((a, b) => a + b, 0)/speed_array.length/2
					if (first_half_mean > second_half_mean) {
						base_json.colors = ['#dc3545']
					}
					else if (first_half_mean < second_half_mean) {
						base_json.colors = ['#28a745']
					}
				} else {
					const duration_array = this.timeseries.map((row) => row.duration_in_traffic/60)
					const first_half_mean = duration_array.slice(0, duration_array.length/2).reduce((a, b) => a + b, 0)/duration_array.length/2
					const second_half_mean = duration_array.slice(duration_array.length/2, duration_array.length).reduce((a, b) => a + b, 0)/duration_array.length/2
					if (first_half_mean < second_half_mean) {
						base_json.colors = ['#dc3545']
					}
					else if (first_half_mean > second_half_mean) {
						base_json.colors = ['#28a745']
					}
				}
				return base_json
			}
		}
	}
</script>


<style scoped>
	.container {
		max-width: 500px;
		margin: 25px auto;
		padding: 0 15px;
		background-color: rgba(108,117,125,.06274509803921569);
		border-radius: 10px;
	}

	.container h4 {
		text-align: center;
		color: #6c757d;
		margin-bottom: 20px;
		margin-top: 32px;
		padding-top: 15px;
	}

	
</style>