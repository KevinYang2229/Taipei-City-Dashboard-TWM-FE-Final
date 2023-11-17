<!-- eslint-disable no-console -->
<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { computed, ref } from 'vue'
import { useMapStore } from '../../store/mapStore'

const props = defineProps([
	'chart_config',
	'activeChart',
	'series',
	'map_config',
])
const mapStore = useMapStore()

const parsedSeries = computed(() => {
	const toParse = [...props.series[0].data]

	const output = toParse.map((item) => {
		return [item.y]
	})
	return output
})

const chartOptions = ref({
	chart: {
		width: 380,
		type: 'polarArea',
	},
	labels: props.chart_config.categories ? props.chart_config.categories : [],
	fill: {
		opacity: 1,
	},
	stroke: {
		width: 1,
		colors: undefined,
	},
	yaxis: {
		show: false,
	},
	legend: {
		position: 'bottom',
	},
	plotOptions: {
		polarArea: {
			rings: {
				strokeWidth: 0,
			},
			spokes: {
				strokeWidth: 0,
			},
		},
	},
	theme: {
		monochrome: {
			enabled: true,
			shadeTo: 'light',
			shadeIntensity: 0.6,
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
	<div v-if="activeChart === 'PolarAreaChart'" class="polarareachart">
		<apexchart
			width="100%"
			type="polarArea"
			:options="chartOptions"
			:series="parsedSeries"
			@dataPointSelection="handleDataSelection"
		></apexchart>
	</div>
</template>

<style scoped lang="scss">
.polarareachart {
	&-title {
		display: flex;
		justify-content: center;
		flex-direction: column;
		margin: 0.5rem 0 -0.5rem;

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
