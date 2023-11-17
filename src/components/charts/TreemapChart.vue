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

const chartOptions = ref({
	chart: {
		borderRadius: 5,
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	dataLabels: {
		formatter: function (val, { dataPointIndex }) {
			return dataPointIndex > 5 ? '' : val
		},
	},
	grid: {
		show: false,
	},
	legend: {
		show: false,
	},
	plotOptions: {
		treemap: {
			distributed: props.series.length < 2,
			shadeIntensity: 0,
		},
	},
	stroke: {
		colors: ['#282a2c'],
		show: true,
		width: 2,
	},
	tooltip: {
		custom: function ({ series, seriesIndex, dataPointIndex, w }) {
			// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
			return (
				'<div class="chart-tooltip">' +
				'<h6>' +
				w.globals.initialSeries[seriesIndex].data[dataPointIndex].x +
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
		labels: {
			show: false,
		},
		type: 'category',
	},
})

const sums = computed(() => {
	return props.series.map((series) => {
		let sum = 0
		series.data.forEach((item) => (sum += item.y))
		return Math.round(sum * 100) / 100
	})
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
	<div v-if="activeChart === 'TreemapChart'" class="treemapchart">
		<div class="treemapchart-title-wrap">
			<div
				class="treemapchart-title"
				v-for="(serie, index) in props.series"
				:key="index"
			>
				<h5>{{ props.series.length > 1 ? serie.name : '總合' }}</h5>
				<h6>{{ sums[index] }} {{ chart_config.unit }}</h6>
			</div>
		</div>
		<apexchart
			width="100%"
			type="treemap"
			:options="chartOptions"
			:series="series"
			@dataPointSelection="handleDataSelection"
		></apexchart>
	</div>
</template>

<style scoped lang="scss">
.treemapchart {
	&-title-wrap {
		display: flex;
		margin: 0.5rem 0 -0.5rem;
	}
	&-title {
		margin-right: 1rem;

		h5 {
			color: var(--color-complement-text);
		}

		h6 {
			color: var(--color-complement-text);
			font-size: var(--font-m);
			font-weight: 400;
		}
	}
}
</style>
