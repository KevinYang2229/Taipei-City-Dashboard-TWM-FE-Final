<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref } from 'vue'
import { useMapStore } from '../../store/mapStore'

const props = defineProps([
	'chart_config',
	'activeChart',
	'series',
	'map_config',
])
const mapStore = useMapStore()

// chartOptions needs to be in the bottom since it uses computed data
const chartOptions = ref({
	chart: {
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	labels: props.chart_config.categories ? props.chart_config.categories : [],
	legend: {
		show: props.series.length > 1 ? true : false,
	},
	grid: {
		show: false,
	},
	markers: {
		strokeWidth: 1,
		strokeColors: '#282a2c',
		strokeOpacity: 0.5,
	},
	dataLabels: {
		enabled: false, // 不顯示數字
		style: {
			colors: ['#282a2c'],
		},
	},
	xaxis: {
		tickAmount: 7,
		axisBorder: {
			color: '#555',
			height: '0.8',
		},
		axisTicks: {
			show: false,
		},
		crosshairs: {
			show: false,
		},
		tooltip: {
			enabled: false,
		},
	},
	yaxis: {
		tickAmount: 7,
	},
	plotOptions: {},
	tooltip: {
		custom: function ({ seriesIndex, w, series, dataPointIndex }) {
			// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
			return (
				'<div class="chart-tooltip">' +
				'<h6>' +
				w.globals.seriesNames[seriesIndex] +
				'</h6>' +
				'<span>' +
				series[seriesIndex][dataPointIndex] +
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
	if (config.seriesIndex !== selectedIndex.value) {
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			props.chart_config.map_filter[0],
			props.chart_config.map_filter[1][config.seriesIndex]
		)
		selectedIndex.value = config.seriesIndex
	} else {
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`
		)
		selectedIndex.value = null
	}
}
</script>

<template>
	<div v-if="activeChart === 'BubbleChart'">
		<!-- BubbleChart -->
		<apexchart
			width="100%"
			height="260px"
			type="bubble"
			:options="chartOptions"
			:series="series"
			@dataPointSelection="handleDataSelection"
		>
		</apexchart>
	</div>
</template>
