<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<!-- This component has two modes 'normal mapview charts' / 'basic map layers' -->
<!-- The different modes are controlled by the prop "isMapLayer" (default false) -->

<script setup>
import { computed, ref } from 'vue'
import { useDialogStore } from '../../store/dialogStore'
import { useMapStore } from '../../store/mapStore'

import { chartTypes } from '../../assets/configs/apexcharts/chartTypes'

import TimeBarInput from '../utilities/TimeBarInput.vue'

const mapStore = useMapStore()
const dialogStore = useDialogStore()

const props = defineProps({
	// The complete config (incl. chart data) of a dashboard component will be passed in
	content: { type: Object },
	isMapLayer: { type: Boolean, default: false },
})

// The default active chart is the first one in the list defined in the dashboard component
const activeChart = ref(props.content.chart_config.types[0])
// Stores whether the component is toggled on or not
const checked = ref(false)
// use condition
const isCondition = ref(false)

// Parses time data into display format
const dataTime = computed(() => {
	if (!props.content.time_from) {
		return '固定資料'
	}
	if (!props.content.time_to) {
		return props.content.time_from.slice(0, 10)
	}
	return `${props.content.time_from.slice(
		0,
		10,
	)} ~ ${props.content.time_to.slice(0, 10)}`
})

// If any map layers are loading, disable the toggle
const shouldDisable = computed(() => {
	if (!props.content.map_config) return false

	const allMapLayerIds = props.content.map_config.map(
		(el) => `${el.index}-${el.type}`,
	)

	return (
		mapStore.loadingLayers.filter((el) => allMapLayerIds.includes(el)).length >
		0
	)
})

// Open and closes the component as well as communicates to the mapStore to turn on and off map layers
function handleToggle() {
	isCondition.value = false
	if (!props.content.map_config) {
		if (checked.value) {
			dialogStore.showNotification('info', '本組件沒有空間資料，不會渲染地圖')
		}
		return
	}
	if (checked.value) {
		mapStore.addToMapLayerList(
			props.content.map_config,
			props.content?.map_source,
		)
	} else {
		mapStore.turnOffMapLayerVisibility(props.content.map_config)
	}
}
// Toggles between chart types defined in the dashboard component
// Also clear any map filters applied
function changeActiveChart(chartName) {
	activeChart.value = chartName
	mapStore.clearLayerFilter(
		`${props.content.map_config[0].index}-${props.content.map_config[0].type}`,
	)
}

// 這邊取得 input time bar 結果
const showInputValue = (payload) => {
	// eslint-disable-next-line no-console
	mapStore.addLayerFilter(
		`${props.content.map_config[0].index}-${props.content.map_config[0].type}`,
		props.content.map_config[0].condition.field,
		+payload,
	)
}
</script>

<template>
	<div
		:class="{
			componentmapchart: true,
			checked: checked,
			maplayer: isMapLayer && checked,
		}"
	>
		<div class="componentmapchart-header">
			<div>
				<div>
					<h3>{{ content.name }}</h3>
					<!-- <span
						v-if="content.chart_config.map_filter && content.map_config"
						@click="
							dialogStore.showNotification(
								'info',
								'本組件有篩選地圖功能，點擊圖表資料點以篩選',
							)
						"
						>tune</span
					> -->
					<span
						v-if="content.chart_config.map_filter && content.map_config"
						@click="isCondition = !isCondition"
						class="componentmapchart-icon"
						>tune</span
					>
					<span
						v-if="content.map_config"
						@click="
							dialogStore.showNotification(
								'info',
								'本組件有空間資料，點擊開關以顯示地圖',
							)
						"
						>map</span
					>
					<span
						v-if="content.history_data"
						@click="
							dialogStore.showNotification(
								'info',
								'回到儀表板頁面並點擊「組件資訊」以查看',
							)
						"
						>insights</span
					>
				</div>
				<h4 v-if="checked">{{ `${content.source} | ${dataTime}` }}</h4>
			</div>
			<div class="componentmapchart-header-toggle">
				<!-- The class "toggleswitch" could be edited in /assets/styles/toggleswitch.css -->
				<label class="toggleswitch">
					<input
						type="checkbox"
						@change="handleToggle"
						v-model="checked"
						:disabled="shouldDisable"
					/>
					<span class="toggleswitch-slider"></span>
				</label>
			</div>
		</div>
		<template v-if="isCondition && checked">
			<TimeBarInput
				@adjust="showInputValue"
				:step="props.content.map_config[0].condition.step"
				:min="props.content.map_config[0].condition.min"
				:max="props.content.map_config[0].condition.max"
				:unit="props.content.map_config[0].condition.unit"
			/>
		</template>
		<template v-else>
			<div
				class="componentmapchart-control"
				v-if="props.content.chart_config.types.length > 1 && checked"
			>
				<button
					v-for="item in props.content.chart_config.types"
					@click="changeActiveChart(item)"
					:key="`${props.content.index}-${item}-mapbutton`"
				>
					{{ chartTypes[item] }}
				</button>
			</div>
			<div class="componentmapchart-chart" v-if="checked && content.chart_data">
				<!-- The components referenced here can be edited in /components/charts -->
				<component
					v-for="item in content.chart_config.types"
					:activeChart="activeChart"
					:is="item"
					:key="`${props.content.index}-${item}-mapchart`"
					:chart_config="content.chart_config"
					:series="content.chart_data"
					:map_config="content.map_config"
				>
				</component>
			</div>
			<div v-else-if="checked" class="componentmapchart-loading">
				<div></div>
			</div>
		</template>
	</div>
</template>

<style scoped lang="scss">
.componentmapchart {
	width: calc(100% - var(--font-m) * 2);
	max-width: calc(100% - var(--font-m) * 2);
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	position: relative;
	padding: var(--font-m);
	border-radius: 5px;
	background-color: var(--color-component-background);

	&-icon {
		cursor: pointer;
	}

	&-condition {
		margin-top: 16px;
		margin-bottom: auto;
	}

	&-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;

		h3 {
			font-size: var(--font-m);
		}

		h4 {
			color: var(--color-complement-text);
			font-size: var(--font-s);
			font-weight: 400;
		}

		div:first-child {
			div {
				display: flex;
				align-items: center;
			}

			span {
				margin-left: 8px;
				color: var(--color-complement-text);
				font-family: var(--font-icon);
				user-select: none;
			}
		}

		&-toggle {
			min-height: 1rem;
			min-width: 2rem;
			margin-top: 4px;
		}
	}

	&-control {
		width: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
		position: absolute;
		top: 4rem;
		left: 0;
		z-index: 10;

		button {
			margin: 0 4px;
			padding: 4px 4px;
			border-radius: 5px;
			background-color: rgb(77, 77, 77);
			opacity: 0.25;
			color: var(--color-complement-text);
			font-size: var(--font-s);
			text-align: center;
			transition:
				color 0.2s,
				opacity 0.2s;

			&:hover {
				opacity: 1;
				color: white;
			}
		}
	}

	&-chart,
	&-loading {
		height: 80%;
		position: relative;
		overflow-y: scroll;

		p {
			color: var(--color-border);
		}
	}

	&-loading {
		display: flex;
		align-items: center;
		justify-content: center;

		div {
			width: 2rem;
			height: 2rem;
			border-radius: 50%;
			border: solid 4px var(--color-border);
			border-top: solid 4px var(--color-highlight);
			animation: spin 0.7s ease-in-out infinite;
		}
	}
}

.checked {
	max-height: 300px;
	height: 300px;
}

.maplayer {
	height: 200px;
	max-height: 200px;
	padding-bottom: 0;
}
</style>
