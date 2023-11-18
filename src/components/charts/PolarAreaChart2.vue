<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, computed } from 'vue'
import { useMapStore } from '../../store/mapStore'

const props = defineProps([
	'chart_config',
	'activeChart',
	'series',
	'map_config',
])
const mapStore = useMapStore()

const parseSeries = computed(() => {
	let output = {}
	let parsedSeries = []
	let parsedCatefories = []
	for (let i = 0; i < props.series[0].data.length; i++) {
		parsedSeries.push(props.series[0].data[i].y)
		parsedCatefories.push(props.series[0].data[i].x)
	}
	output.series = parsedSeries
	output.categories = parsedCatefories
	return output
})

// chartOptions needs to be in the bottom since it uses computed data
const chartOptions = ref({
	chart: {
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	labels: parseSeries.value.categories ?? [],
	legend: {
		show: parseSeries.value.series.length > 1 ? true : false,
		position: 'bottom',
	},
	fill: {
		opacity: 1,
	},
	stroke: {
		// width: 0,
		colors: ['#282a2c'],
	},
	plotOptions: {
		polarArea: {
			rings: {
				strokeWidth: 1,
				strokeColor: '#555',
			},
			spokes: {
				strokeWidth: 0,
			},
		},
	},
	tooltip: {
		custom: function ({ seriesIndex, w, series }) {
			// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
			return (
				'<div class="chart-tooltip">' +
				'<h6>' +
				w.globals.seriesNames[seriesIndex] +
				'</h6>' +
				'<span>' +
				series[seriesIndex] +
				` ${props.chart_config.unit}` +
				'</span>' +
				'</div>'
			)
		},
		enabled: true,
	},
})

const selectedIndex = ref(null)

function handleDataSelection(e, chartContext, config) {
	if (!props.chart_config.map_filter) {
		return
	}
	if (config.dataPointIndex !== selectedIndex.value) {
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			props.chart_config.map_filter[0],
			props.chart_config.map_filter[1][config.dataPointIndex]
		)
		selectedIndex.value = config.dataPointIndex
	} else {
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`
		)
		selectedIndex.value = null
	}
}
</script>

<template>
	<div v-if="activeChart === 'PolarAreaChart2'" class="polarchart">
		<!-- PolarAreaChart -->
		<apexchart
			width="100%"
			height="260px"
			type="polarArea"
			:options="chartOptions"
			:series="parseSeries.series"
			@dataPointSelection="handleDataSelection"
		>
		</apexchart>
	</div>
</template>

<style scoped lang="scss">
.polarchart {
	margin-top: 15px;
}
</style>
