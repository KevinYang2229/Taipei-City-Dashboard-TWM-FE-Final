<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, onMounted, computed } from 'vue'
const targetItem = ref(null)
const mousePosition = ref({ x: null, y: null })

const props = defineProps(['chart_config', 'activeChart', 'series'])
let targetData = { status: null }

const {color, categories, unit, itemPrimaryPercentage} = props.chart_config

const chartData = {
	number: 50,
}
let iconColorPrimary = color[0]
let iconColorSecondary = color[1]
let iconNamePrimary = categories[0]
let iconNameSecondary = categories[1]


onMounted(() => {})

function toggleActive(e) {
	targetItem.value = e.target.dataset.name
	targetData.name = e.target.childNodes[0].attributes['data-name'].nodeValue
	targetData.value = e.target.childNodes[0].attributes['data-value'].nodeValue
}
function initActiveToNull(e) {
	targetItem.value = null
}

const tooltipPosition = computed(() => {
	return {
		left: `${mousePosition.value.x - 10}px`,
		top: `${mousePosition.value.y - 54}px`,
	}
})
const updateMouseLocation = (e) => {
	mousePosition.value.x = e.pageX
	mousePosition.value.y = e.pageY
}

// 取得畫面上第一個 icon 要顯示的數量
const itemPrimaryCount = (() => Math.round((itemPrimaryPercentage / 100) * chartData.number))

const getDataStatus = computed(() => {
	let activeCount = itemPrimaryCount()
	let result = (index) => (index <= activeCount ? '男生' : '女生')
	return result
})
const itemSecondaryPercentage = computed(() => 100 - itemPrimaryPercentage)
const lighterPrimaryColor = computed(() => `${iconColorPrimary}57`)
</script>

<template>
	<div v-if="activeChart === 'SingleIconPercentageChart'" class="iconPercentageChart">
		<div class="iconPercentageChart__title">
			<h2 class="iconPercentageChart__content">
				有效<span class="iconPercentageChart__percentage">{{
					itemPrimaryPercentage
				}}</span
				>%
			</h2>
		</div>
		<div class="iconPercentageChart__chart">
			<svg
				:data-name="item"
				width="36"
				height="36"
				viewBox="0 0 36 36"
				fill="none"
				xmlns="http://www.w3.org/2000/svg"
				v-for="(item, index) in chartData.number"
				:key="item"
				:class="`initial-animation-${item} initial-animation`"
				@mouseenter="toggleActive"
				@mouseleave="initActiveToNull"
				@mousemove="updateMouseLocation"
			>
				<g :data-name="getDataStatus(index)" :data-value="itemPrimaryPercentage">
					<path
						d="M18 6.75C19.864 6.75 21.375 5.23896 21.375 3.375C21.375 1.51104 19.864 0 18 0C16.136 0 14.625 1.51104 14.625 3.375C14.625 5.23896 16.136 6.75 18 6.75Z"
						:fill="index+1 <= itemPrimaryCount() ? iconColorPrimary:lighterPrimaryColor"
					/>
					<path
						d="M13.5 15.1875V34.3125C13.5 35.2445 14.2555 36 15.1875 36C16.1195 36 16.875 35.2445 16.875 34.3125V23.625C16.875 23.0037 17.3787 22.5 18 22.5C18.6213 22.5 19.125 23.0037 19.125 23.625V34.3125C19.125 35.2445 19.8805 36 20.8125 36C21.7445 36 22.5 35.2445 22.5 34.3125V15.1875C22.5 14.8768 22.7518 14.625 23.0625 14.625C23.3732 14.625 23.625 14.8768 23.625 15.1875V20.8125C23.625 21.7445 24.3805 22.5 25.3125 22.5C26.2445 22.5 27 21.7445 27 20.8125V14.625C27 10.8971 23.9779 7.875 20.25 7.875H15.75C12.0221 7.875 9 10.8971 9 14.625V20.8125C9 21.7445 9.75552 22.5 10.6875 22.5C11.6195 22.5 12.375 21.7445 12.375 20.8125V15.1875C12.375 14.8768 12.6268 14.625 12.9375 14.625C13.2482 14.625 13.5 14.8768 13.5 15.1875Z"
						:fill="index+1 <= itemPrimaryCount() ? iconColorPrimary:lighterPrimaryColor"
					/>
				</g>
			</svg>

			<Teleport to="body">
				<div
					v-if="targetItem"
					class="iconPercentageChart__chart-info chart-tooltip"
					:style="tooltipPosition"
				>
					<h6>{{ targetData.name }}比例</h6>
					<span
						>{{ targetData.value }}{{ unit }}</span
					>
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
		color: #db4d6d;
		font-size: 1.6rem;
		padding: 0 0.3em;
		
	}
	&__content{
		&:first-child {
			.iconPercentageChart__percentage{
				color: #2ea9df;
			}
		}
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