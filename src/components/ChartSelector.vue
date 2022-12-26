<template>
	<div class="container">
		<div class="button" @click="selectTab('City')" :class="{selected: tab_selected==='City'}">City</div>
		<div class="button" @click="selectTab('Route')" :class="{selected: tab_selected==='Route'}">Route</div>
	</div>
	<div class="container">
		<div class="dropdown" v-if="tab_selected==='City'">
			<label for="city_dropdown">City</label>
			<select id="city_dropdown" v-model="city_selected" @change="selectChart('City')">
				<option v-for="option in city_options" :value="option.city">{{option.city_label}}
				</option>
			</select>
		</div>
		<div class="dropdown" v-if="tab_selected==='Route'">
			<label for="origin_dropdown">Origin</label>
			<select id="origin_dropdown" v-model="origin_selected" @change="selectChart('Route')">
				<option 
					v-for="option in origin_destination_pairs.filter(option => option.destination === destination_selected)" 
					:value="option.origin"
					>{{option.origin_label}}
				</option>
			</select>
		</div>
		<div class="dropdown" v-if="tab_selected==='Route'">
			<label for="destination_dropdown">Destination</label>
			<select id="destination_dropdown" v-model="destination_selected" @change="selectChart('Route')">
				<option 
					v-for="option in origin_destination_pairs.filter(option => option.origin === origin_selected)" 
					:value="option.destination"
					>{{option.destination_label}}
				</option>
			</select>
		</div>
	</div>
</template>

<script>
export default {
	name: 'ChartSelector',
	data() {
		return {
			tab_selected: '',
			city_options: [],
			origin_destination_pairs: [], 
			city_selected: '',
			origin_selected: '',
			destination_selected: ''
		}
	},
	async created() {
		this.city_options = await this.fetchCities();
		this.origin_destination_pairs = await this.fetchOriginDestinationPairs();

		this.tab_selected = 'City';
		this.city_selected = this.city_options[0].city;
		this.origin_selected = this.origin_destination_pairs[0].origin;
		this.destination_selected = this.origin_destination_pairs.filter(option => option.origin === this.origin_selected)[0].destination;

		this.selectChart(this.tab_selected);
	},
	methods: {
		async fetchCities() {
			const response = await fetch('data/cities.json');
			const data = await response.json();
			return data;
		},
		async fetchOriginDestinationPairs() {
			const response = await fetch('data/origin_destination_pairs.json');
			const data = await response.json();
			return data;
		},
		selectTab(chart) {
			this.tab_selected = chart;
			this.selectChart(this.tab_selected);
		}, 
		selectChart(chart_type) {
			if (chart_type === 'City') {
				this.$emit('chart-selected', {
					sha256: this.city_options.find(option => option.city === this.city_selected).sha256, 
					chart_type: chart_type
				});
			} else {
				this.$emit('chart-selected', {
					sha256: this.origin_destination_pairs.find(option => option.origin === this.origin_selected && option.destination === this.destination_selected).sha256, 
					chart_type: chart_type
				});
			}
		}
	}, 
	emits: ['chart-selected']
}
</script>

<style scoped>

.container {
	max-width: 500px;
	margin: 16px auto;
	display: flex;
	padding: 0 15px;
}

.button {
	width: 80px;
	height: 50px;
	padding-left: 20px;
	padding-right: 20px;
	margin-right: 5px;
	background-color: rgba(108,117,125,.06274509803921569);
	border-top-left-radius: 5px;
	border-top-right-radius: 5px;
	cursor: pointer;
	color: rgba(108,117,125,.6);
	font-size: 15px;
	font-weight: 700;
	text-align: center;
	line-height: 50px;
}

.button:hover {
	background-color: rgba(108,117,125,.12549019607843137);
}

.button.selected {
	background-color: #1e1e30;
	color: #bdbdbd;
}

.dropdown {
	width: 45%;
	margin-right: 10px;
}

.dropdown label {
	color: rgba(108,117,125,.6);
	font-size: 12px;
	padding: 8px;
	display: block;
}

.dropdown select {
	width: 100%;
	height: 35px;
	background-color: #1e1e30;
	border: 2px solid hsla(0,0%,58%,.10196078431372549);
	border-radius: 4px;
	padding: 8px;
	color: #6c757d;
	font-weight: 500;
}

*:focus {
   outline: 0;
 }
</style>
