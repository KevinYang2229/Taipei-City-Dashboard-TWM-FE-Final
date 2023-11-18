<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, onMounted, computed } from "vue";
import ChartIcon from "../utilities/ChartIcon.vue";
const targetItem = ref(null);
const mousePosition = ref({ x: null, y: null });

const props = defineProps(["chart_config", "activeChart", "series"]);
let targetData = { status: null };

const { color, categories, unit } = props.chart_config;

const chartIconTotal = 50;
let iconColorPrimary = color[0];
let iconColorSecondary = color[1];
let iconNamePrimary = categories[0];
let iconNameSecondary = categories[1];

const chartDataTotalNumber = computed(() => {
	let total = 0;
	props.series.forEach((item) => {
		total += item.data;
	});
	return total;
});

// calculate percentage
const primaryPercentage = computed(() => {
	return Math.round(
		(props.series[0].data / chartDataTotalNumber.value) * 100
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
	return chartIconTotal - primaryIconNumber.value;
});

onMounted(() => {
	// startAnimation();
});

const toggleActive = (e) => {
	targetItem.value = e.target.dataset.name;
	targetData.name = e.target.childNodes[0].attributes["data-name"].nodeValue;
	targetData.value =
		e.target.childNodes[0].attributes["data-value"].nodeValue;
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
const getPositon = (index) => {
	if (index === 1) {
		iconPositionX = 0;
		iconPositionY = 0;
		return `${iconPositionX},${iconPositionY}`;
	}
	if (index % 10 === 1) {
		// 找出數字尾數為 1 的數字
		iconPositionX = 0;
		iconPositionY += 38 + 4;
	} else {
		iconPositionX += 38;
	}
	return `${iconPositionX},${iconPositionY}`;
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
			<!-- <p class="number">{{ counter }}</p> -->
			<h2 class="iconPercentageChart__content">
				{{ iconNamePrimary
				}}<span
					class="iconPercentageChart__percentage"
					:style="{ color: iconColorPrimary }"
					>{{ primaryPercentage }}</span
				>%
			</h2>
			<h2 class="iconPercentageChart__content">
				{{ iconNameSecondary
				}}<span
					class="iconPercentageChart__percentage"
					:style="{ color: iconColorSecondary }"
					>{{ secondaryPercentage }}</span
				>%
			</h2>
		</div>
		<div class="iconPercentageChart__chart">
			<svg
				viewBox="0 0 382 250"
				fill="none"
				xmlns="http://www.w3.org/2000/svg"
			>
				<g
					:data-name="item"
					v-for="(item, index) in chartIconTotal"
					:key="item"
					:class="`initial-animation-${item} initial-animation`"
					:transform="`translate(${getPositon(index + 1)})`"
					:data-value="primaryPercentage"
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
					<!-- <chart-icon v-else icon-name="woman"></chart-icon> -->
					<chart-icon v-else icon-name="oldman"></chart-icon>
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
		font-size: 1.6rem;
		padding: 0 0.3em;
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
