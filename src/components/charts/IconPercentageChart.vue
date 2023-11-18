<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, onMounted, computed, reactive } from "vue";
import ChartIcon from "../utilities/ChartIcon.vue";
const targetItem = ref(null);
const mousePosition = ref({ x: null, y: null });

const props = defineProps(["chart_config", "activeChart", "series"]);
let targetData = { status: null };

const { color, categories, unit } = props.chart_config;

const activeData = ref({ ...props.series[props.series.length - 1] });
// const activeData = reactive(props.series[props.series.length - 1]);

const chartIconTotal = 50;
let iconColorPrimary = color[0];
let iconColorSecondary = color[1];
let iconNamePrimary = categories[0];
let iconNameSecondary = categories[1];

const chartDataTotalNumber = computed(() => {
	return activeData.value;
});

// calculate percentage
const primaryPercentage = computed(() => {
	return Math.round(
		(activeData.value.data[0].count / activeData.value.count) * 100
	);
});
const secondaryPercentage = computed(() => {
	return 100 - primaryPercentage.value;
});

// calculate how much icon number in chart
const primaryIconNumber = computed(() => {
	return Math.round((primaryPercentage.value * chartIconTotal) / 100);
});
const secondaryIconNumber = computed(() => {
	return activeData.value.count - primaryIconNumber.value;
});

onMounted(() => {
	// startAnimation();
});

const toggleActive = (e) => {
	targetItem.value = e.target.dataset.name;
	targetData.name = e.target.dataset.name;
	targetData.value = e.target.dataset.value;
};
const initActiveToNull = () => {
	targetItem.value = null;
};

const tooltipPosition = computed(() => {
	return {
		left: `${mousePosition.value.x - 10}px`,
		top: `${mousePosition.value.y - 54}px`,
	};
});
const updateMouseLocation = (e) => {
	mousePosition.value.x = e.pageX;
	mousePosition.value.y = e.pageY;
};

let iconPositionX = 0;
let iconPositionY = 0;
let xWidth = 38;
let yHeight = 38;
const getPositon = (index) => {
	if (index === 1) {
		iconPositionX = 0;
		iconPositionY = 0;
		return `${iconPositionX},${iconPositionY}`;
	}
	if (index % 10 === 1) {
		// 找出數字尾數為 1 的數字
		iconPositionX = 0;
		iconPositionY += yHeight + 4;
	} else {
		iconPositionX += xWidth - 8;
	}
	return `${iconPositionX},${iconPositionY}`;
};

const updateChartData = (year) => {
	let newData = props.series.filter((e) => e.year === year)[0];
	activeData.value.count = newData.count;
	activeData.value.year = newData.year;
	activeData.value.data = newData.data;
};

// const counter = ref(0);
// const targetValue = 10;
// const animationDuration = 2000; // in milliseconds
// let isAnimating = false;
// let intervalId = null;

// const startAnimation = () => {
// 	if (!isAnimating) {
// 		isAnimating = true;
// 		intervalId = setInterval(animateNumber, animationDuration / 100);
// 	}
// };

// const animateNumber = () => {
// 	if (counter.value < targetValue) {
// 		counter.value++;
// 	} else {
// 		isAnimating = false;
// 		clearInterval(intervalId);
// 	}
// };
</script>

<template>
	<div
		v-if="activeChart === 'IconPercentageChart'"
		class="iconPercentageChart"
	>
		<div class="iconPercentageChart__title">
			<!-- <div v-for="(item, index) in series" key="index">
				<h2 class="iconPercentageChart__content">
					{{ iconNamePrimary
					}}<span
						class="iconPercentageChart__percentage"
						:style="{ color: iconColorPrimary }"
						>{{ primaryPercentage }}</span
					>%
				</h2>
				<p>{{ item.data }}</p>
			</div> -->
			<div class="iconPercentageChart__content">
				<h2>
					{{ iconNamePrimary
					}}<span
						class="iconPercentageChart__percentage"
						:style="{ color: iconColorPrimary }"
						>{{ primaryPercentage }}</span
					>%
				</h2>
				<p>總人數：{{ activeData.data[0].count }}</p>
			</div>
			<div class="iconPercentageChart__content">
				<h2>
					{{ iconNameSecondary
					}}<span
						class="iconPercentageChart__percentage"
						:style="{ color: iconColorSecondary }"
						>{{ secondaryPercentage }}</span
					>%
				</h2>
				<p>總人數：{{ activeData.data[1].count }}</p>
			</div>
		</div>
		<div class="iconPercentageChart__buttons">
			<button
				@click="updateChartData(item.year)"
				class="iconPercentageChart__button"
				:class="activeData.year === item.year ? 'active' : ''"
				type="button"
				v-for="item in series"
				key="item.year"
			>
				{{ item.year }}
			</button>
		</div>
		<div class="iconPercentageChart__chart">
			<svg
				viewBox="0 0 310 220"
				fill="none"
				xmlns="http://www.w3.org/2000/svg"
			>
				<g
					:data-name="
						index < primaryIconNumber
							? series[0].name
							: series[1].name
					"
					v-for="(item, index) in chartIconTotal"
					:key="item"
					:class="`initial-animation-${item} initial-animation`"
					:transform="`translate(${getPositon(index + 1)})`"
					:data-value="
						index < primaryIconNumber
							? primaryPercentage
							: secondaryPercentage
					"
					:fill="
						index < primaryIconNumber
							? iconColorPrimary
							: iconColorSecondary
					"
					@mouseenter="toggleActive"
					@mouseleave="initActiveToNull"
					@mousemove="updateMouseLocation"
				>
					<chart-icon
						v-if="index < primaryIconNumber"
						icon-name="man"
					></chart-icon>
					<chart-icon v-else icon-name="woman"></chart-icon>
				</g>
			</svg>

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

.iconPercentageChart {
	max-height: 100%;
	position: relative;
	overflow-y: scroll;
	&__chart {
		margin: 0 2rem;
	}
	&__chart-info {
		position: fixed;
		z-index: 20;
	}
	color: var(--iconPercentageChartFontColor);
	&__title {
		margin-bottom: 1rem;
		display: flex;
		justify-content: space-around;
	}
	&__percentage {
		font-size: 1.3rem;
		padding: 0 0.3em;
	}
	&__content {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		p {
			color: var(--color-complement-text);
			margin-top: 0.2 rem;
		}
	}
	&__buttons {
		color: var(--color-complement-text);
		margin: 0 2rem;
		display: flex;
		justify-content: space-around;
		margin-bottom: 0.5rem;
	}
	&__button {
		color: var(--color-complement-text);
		border: solid 1px var(--color-complement-text);
		padding: 0 6px;
		border-radius: 5px;
	}
	&__button.active {
		color: #fff;
		background-color: #ffffff2e;
	}
}

.initial-animation {
	opacity: 0;
}
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
