<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<!-- TODO: BUG
	1. [圖形渲染問題] 第一次 load 正常，在切換儀表板列表又切回來 boxPlot 時，BoxPlot 的圖沒長出來 (DOM .apexcharts-boxPlot-area 元素有生成，但似乎沒正常 render 出圖形)，重整 (= reload) 才會長出 BoxPlot 的圖形
	2.[X軸定位問題] hover 的區域並沒有對齊 X 刻度的中間（開啟 crosshair 比較好比對），資料是抓官方的 data，官方 demo 是置中，不知道為什麼目前這個圖表是貼齊 BoxPlot 的右側
	ref: https://apexcharts.com/javascript-chart-demos/box-whisker-charts/basic/
-->

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
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	stroke: {
		colors: props.chart_config.color,
	},
	tooltip: {
		custom: function ({ seriesIndex, dataPointIndex, w }) {
			const data = w.globals.initialSeries[seriesIndex].data[dataPointIndex]
			const [q0, q2, q4] = data.y // q1, q3 are not used
			return (
				'<div class="chart-tooltip">' +
				"<div style='margin-right: 0.4em'><h6>最大值</h6>" +
				'<span>' +
				q4 +
				'</span></div>' +
				"<div style='margin-right: 0.4em'><h6>中位數</h6>" +
				'<span>' +
				q2 +
				'</span></div>' +
				'<div><h6>最小值</h6>' +
				'<span>' +
				q0 +
				'</span></div>' +
				'</div>'
			)
		},
	},
	xaxis: {
		tooltip: {
			enabled: false,
		},
		crosshairs: {
			// show: false,
		},
		axisTicks: {
			show: false,
		},
		axisBorder: {
			color: '#555',
			height: '0.8',
		},
		type: 'datetime',
	},
	grid: {
		show: false,
	},
	plotOptions: {
		boxPlot: {
			colors: {
				upper: props.chart_config.color[0] ?? 'rgb(31, 155, 133)',
				lower: props.chart_config.color[1] ?? 'rgb(128, 227, 212)',
			},
		},
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
	<div v-if="activeChart === 'BoxPlotChart'">
		<!-- BoxPlotChart -->
		<apexchart
			width="100%"
			height="260px"
			type="boxPlot"
			:options="chartOptions"
			:series="series"
			@dataPointSelection="handleDataSelection"
		></apexchart>
	</div>
</template>
