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

const chartOptions = ref({
	chart: {
		stacked: true,
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	dataLabels: {
		enabled: props.chart_config.categories ? false : true,
		offsetY: 20,
	},
	grid: {
		show: false,
	},
	legend: {
		show: props.chart_config.categories ? true : false,
	},
	plotOptions: {
		bar: {
			borderRadius: 5,
		},
	},
	stroke: {
		colors: ['#282a2c'],
		show: true,
		width: 2,
	},
	tooltip: {
		// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
		custom: function ({ series, seriesIndex, dataPointIndex, w }) {
			return (
				'<div class="chart-tooltip">' +
				'<h6>' +
				w.globals.labels[dataPointIndex] +
				`${
					props.chart_config.categories
						? '-' + w.globals.seriesNames[seriesIndex]
						: ''
				}` +
				'</h6>' +
				'<span>' +
				series[seriesIndex][dataPointIndex] +
				` ${props.chart_config.unit}` +
				'</span>' +
				'</div>'
			)
		},
	},
	xaxis: {
		axisBorder: {
			show: false,
		},
		axisTicks: {
			show: false,
		},
		categories: props.chart_config.categories
			? props.chart_config.categories
			: [],
		labels: {
			offsetY: 5,
		},
		type: 'category',
	},
})

const selectedIndex = ref(null)

function handleDataSelection(e, chartContext, config) {
	if (!props.chart_config.map_filter) {
		return
	}
	const toFilter = config.dataPointIndex
	mapStore.clearLayerFilter(
		`${props.map_config[0].index}-${props.map_config[0].type}`
	)
	mapStore.removePopup()
	mapStore.clearMaker()

	if (toFilter !== selectedIndex.value) {
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			props.chart_config.map_filter[0],
			props.chart_config.map_filter[1][toFilter]
		)
		selectedIndex.value = toFilter
		mapStore.map.flyTo({
			center: props.chart_config.lngLat[0][toFilter],
			zoom: 14,
			essential: true,
		})
		mapStore.customPopup(
			props.chart_config.lngLat[0][toFilter],
			props.chart_config.treeLimit[0][toFilter]
		)
	} else {
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`
		)
		selectedIndex.value = null
		mapStore.map.flyTo({
			center: '',
			zoom: 12,
			essential: true,
		})
	}
}
</script>

<template>
	<div v-if="activeChart === 'ColumnChart'">
		<apexchart
			width="100%"
			height="270px"
			type="bar"
			:options="chartOptions"
			:series="series"
			@dataPointSelection="handleDataSelection"
		></apexchart>
	</div>
</template>

<style scoped>
.custom-popup {
	background-color: #888;
	border: 2px solid #333;
	border-radius: 5px;
	padding: 10px;
	max-width: 200px;
	box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2); /* 阴影 */
	text-align: center;
	font-size: 16px;
	color: #fff;
}

.custom-popup h2 {
	margin: 0;
	padding: 0;
	font-size: 20px;
	color: #0074d9;
}

.custom-popup p {
	margin: 0;
	padding: 0;
}
</style>
