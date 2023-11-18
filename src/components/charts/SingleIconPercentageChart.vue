<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, onMounted, computed } from "vue";
import ChartIcon from "../utilities/ChartIcon.vue";
const targetItem = ref(null);
const mousePosition = ref({ x: null, y: null });
const selectedIndex = ref(null);

const props = defineProps(["chart_config", "activeChart", "series"]);
let targetData = { status: null };

const { color, categories, unit, itemPrimaryPercentage } = props.chart_config;
// get data
const chartData = computed(() => {
	const newData = props.series[0].data.map((item, index) => ({
		...item,
		color: color[index],
	}));
	return newData;
});

function toggleActive(e) {
	targetItem.value = e.target.dataset.name;
}
function toggleActiveToNull() {
	targetItem.value = null;
}
onMounted(() => {});
function handleDataSelection(index) {
	if (!props.chart_config.map_filter) {
		return;
	}
	if (index !== selectedIndex.value) {
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			props.chart_config.map_filter[0],
			props.chart_config.map_filter[1][index]
		);
		selectedIndex.value = index;
		mapStore.map.flyTo({
			center: props.chart_config.lngLat[0][index],
			zoom: 14,
			essential: true,
		});
	} else {
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`
		);
		selectedIndex.value = null;
	}
}
</script>

<template>
	<div
		v-if="activeChart === 'SingleIconPercentageChart'"
		class="iconPercentageChart"
	>
		<div class="singleIconPercentageChart__chart">
			<div class="singleIconPercentageChart__items">
				<div
					class="singleIconPercentageChart__item"
					v-for="(item, index) in chartData"
					:style="{
						filter:
							targetItem === item.type || !targetItem
								? 'none'
								: 'grayscale(90%)',
					}"
					:key="item.name"
					@click="handleDataSelection(0)"
					:class="{
						'active-item':
							targetItem === item.type || selectedIndex === 0,
						[`initial-animation-${index + 1}`]: true,
					}"
				>
					<chart-icon
						:icon-name="`singleIcon-${item.name}`"
					></chart-icon>
					<div class="singleIconPercentageChart__count">
						{{ item.count }}
					</div>
				</div>
			</div>
			<div class="singleIconPercentageChart__symbols">
				<div
					class="singleIconPercentageChart__symbol"
					v-for="item in chartData"
					:key="item.name"
					:data-name="item.type"
					@mouseleave="toggleActiveToNull"
					@mouseenter="toggleActive"
				>
					<span
						:style="{ backgroundColor: item.color }"
						class="singleIconPercentageChart__indicator"
					></span>
					<span>{{ item.type }}</span>
				</div>
			</div>

			<Teleport to="body">
				<div
					v-if="targetItem"
					class="iconPercentageChart__chart-info chart-tooltip"
					:style="tooltipPosition"
				>
					<h6>{{ targetData.name }}比例</h6>
					<span>{{ targetData.value }}{{ unit }}</span>
				</div>
			</Teleport>
		</div>
	</div>
</template>

<style scoped lang="scss">
:root {
	--iconPercentageChartFontColor: #fffffb;
}

.singleIconPercentageChart {
	max-height: 100%;
	position: relative;
	overflow-y: scroll;
	&__chart-info {
		position: fixed;
		z-index: 20;
	}
	color: var(--iconPercentageChartFontColor);
	&__title {
		margin: 1rem 0;
		display: flex;
		justify-content: space-around;
	}
	&__percentage {
		color: #db4d6d;
		font-size: 1.6rem;
		padding: 0 0.3em;
	}
	&__content {
		&:first-child {
			.iconPercentageChart__percentage {
				color: #2ea9df;
			}
		}
	}
	&__items {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-wrap: wrap;
	}
	&__item {
		padding: 0.5rem 4rem;
	}
	&__count {
		text-align: center;
	}
	&__symbols {
		display: flex;
		justify-content: space-around;
		margin: 1.5rem 1.5rem 0;
		color: var(--color-complement-text);
		font-size: var(--font-s);
		font-weight: 400;
	}
	&__symbol {
		display: flex;
		align-items: center;
		justify-content: center;
	}
	&__indicator {
		width: 1em;
		height: 1em;
		border-radius: 100%;
		display: inline-block;
		margin-right: 0.5em;
	}
}

.initial-animation {
	opacity: 0;
}

// .active-item {
// 	transform: translateY(-5px);
// }
@keyframes ease-in {
	0% {
		opacity: 0;
	}

	100% {
		opacity: 1;
	}
}

@for $i from 1 through 50 {
	.initial-animation-#{$i} {
		animation-name: ease-in;
		animation-duration: 0.1s;
		animation-delay: 0.02s * ($i - 1);
		animation-timing-function: linear;
		animation-fill-mode: forwards;
	}
}
</style>
