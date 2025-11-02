<template>
	<div class="input-container">
		<input
			v-model="displayDate"
			:class="props.inputClass"
			class="input-date"
			@input="formatDate"
			@blur="validateDate"
			maxlength="10"
		>
		<img
			src="/image/calendar.svg"
			alt="Выбрать дату"
			class="icon"
			@click.stop="showCalendar=!showCalendar"
		>
	</div>

	<Calendar
		v-if="showCalendar"
		:select="date"
		@close="closeCalendar"
		@change="changeDate"
	/>
</template>

<script setup>
	import Calendar from "@/components/calendar/Calendar.vue";
	import {ref} from "vue";
	const emit = defineEmits(['change']);
	const props = defineProps({inputClass: String});
	const showCalendar = ref(false);
	const today = new Date();
	const date = ref(`${today.getFullYear()}.${today.getMonth()+1}.${today.getDate()}`);
	const displayDate = ref(date.value);
	function formatDate(event) {
		let value = event.target.value;

		value = value.replace(/[^\d.]/g, '');

		const parts = value.split('.');
		if (parts.length > 3) {
			value = parts.slice(0, 3).join('.');
		}

		if (value.length > 4 && value.charAt(4) !== '.') {
			value = value.substring(0, 4) + '.' + value.substring(4);
		}
		if (value.length > 7 && value.charAt(7) !== '.') {
			value = value.substring(0, 7) + '.' + value.substring(7);
		}

		if (value.length > 10) {
			value = value.substring(0, 10);
		}

		displayDate.value = value;
	}

	function validateDate() {
		const parts = displayDate.value.split('.');

		if (parts.length !== 3) {
			displayDate.value = date.value;
			return;
		}

		let [year, month, day] = parts.map(Number);

		year = Math.max(1000, Math.min(year, 9999));

		month = Math.max(1, Math.min(month, 12));

		const maxDays = getDaysInMonth(year, month);
		day = Math.max(1, Math.min(day, maxDays));

		const validDate = `${year}.${month.toString().padStart(2, '0')}.${day.toString().padStart(2, '0')}`;

		if (validDate !== displayDate.value) {
			displayDate.value = validDate;
			date.value = validDate;
			emit('change', date.value);
		} else {
			date.value = displayDate.value;
			emit('change', date.value);
		}
	}

	function getDaysInMonth(year, month) {
		return new Date(year, month, 0).getDate();
	}

	function changeDate(newDate){
		date.value = newDate;
		displayDate.value = newDate;
		closeCalendar();
		emit('change', date);
	}

	function closeCalendar(){
		showCalendar.value = false;
	}

</script>

<style scoped>
.input-container{
	display: flex;
	align-items: center;
	position: relative;
	width: 177px;

}
.input-date{
	width: 100%;
}
.icon{
	position: absolute;
	right: 0;
	height: 15px;
	padding-right: 2px;
	cursor: pointer;
}
</style>