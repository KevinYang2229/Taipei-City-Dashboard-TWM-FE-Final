<template>
	<div class="condition">
		<div class="btnGroup">
			<button
				:class="[
					'condition-btn',
					dateTimeSelection === 'day' ? 'condition-btn--active' : '',
				]"
				@click="dateTimeClickHandler('day')"
			>
				日
			</button>
			<button
				:class="[
					'condition-btn',
					dateTimeSelection === 'month'
						? 'condition-btn--active'
						: '',
				]"
				@click="dateTimeClickHandler('month')"
			>
				月
			</button>
			<button
				:class="[
					'condition-btn',
					dateTimeSelection === 'hour' ? 'condition-btn--active' : '',
				]"
				@click="dateTimeClickHandler('hour')"
			>
				小時
			</button>
		</div>

		<div class="displayGroup">
			<div class="displayGroup-text">時間區間</div>
			<div class="displayGroup-text">{{ modelValue }}{{ unit }}</div>
		</div>

		<div class="inputGroup">
			<input
				class="condition-range"
				v-model="modelValue"
				@input="inputHandler"
				type="range"
				:min="min"
				:max="max"
				:step="step"
			/>
		</div>
	</div>
</template>

<script setup>
import { ref } from "vue";
import { useDialogStore } from "../../store/dialogStore";

const props = defineProps(["step", "unit", "min", "max"]);
const emit = defineEmits(["adjust"]);
const dialogStore = useDialogStore();

// day or month or hour
const dateTimeSelection = ref("hour");
// curren value
const modelValue = ref(props.min);

// eslint-disable-next-line no-unused-vars
const dateTimeClickHandler = (target) => {
	dialogStore.showNotification("info", "本組件預留選項，但尚未有資料符合");
	// dateTimeSelection.value = target
	// modelValue.value = 0
	// if (target === 'hour') {
	// 	maxValue.value = 24
	// 	unit.value = '小時'
	// }
	// if (target === 'day') {
	// 	maxValue.value = 30
	// 	unit.value = '日'
	// }
	// if (target === 'month') {
	// 	maxValue.value = 12
	// 	unit.value = '月'
	// }
};

const inputHandler = (e) => {
	emit("adjust", e.target.value);
};
</script>

<style lang="scss" scoped>
.condition {
	display: flex;
	flex-wrap: wrap;
	margin-top: 20px;
	margin-bottom: auto;

	&-btn {
		width: 60px;
		padding: 4px 0;
		border: 1px solid #444;
		border-radius: 6px;

		&--active {
			background: #3a69a0;
		}
	}

	&-range {
		width: 100%;
		border: 0;
	}
}

// input[type='range'] {
// 	accent-color: #dac38e;
// }

[type="range"] {
	-webkit-appearance: none;
	appearance: none;
	margin: 0;
	outline: 0;
	background-color: transparent;
}
[type="range"]::-webkit-slider-runnable-track {
	height: 4px;
	background: #3a69a0;
}
[type="range"]::-webkit-slider-container {
	height: 20px;
	overflow: hidden;
}

[type="range"]::-webkit-slider-thumb {
	position: relative;
	-webkit-appearance: none;
	appearance: none;
	width: 20px;
	height: 20px;
	border-radius: 50%;
	background-color: #a0badc;
	border: 1px solid transparent;
	margin-top: -8px;
	border-image: linear-gradient(#a0badc, #a0badc) 0 fill / 8 20 8 0 / 0px 0px
		0 2000px;
}

.inputGroup {
	width: 100%;
	margin-top: 12px;
	display: flex;
	align-items: center;
}

.btnGroup {
	width: 100%;
	display: flex;
	justify-content: center;
	gap: 0 8px;
	margin-bottom: 16px;
}

.displayGroup {
	width: 100%;
	display: flex;
	justify-content: space-between;

	&-text {
		color: #888787;
		font-size: 14px;
	}
}
</style>
