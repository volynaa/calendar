<template>
	<div class="calendar-container" ref="containerRef">
		<div class="calendar-header">
			<button @click="changeMonth('-')">&lt;</button>
			<p class="current-month-year">{{currentMonthYear}}</p>
			<button @click="changeMonth('+')">&gt;</button>
		</div>
		<div class="calendar-grid" >
			<div v-for="item in weekday" class="grid">{{item[+languageRus]}}</div>
		</div>

		<div class="calendar-day">
			<div v-for="n in firstWeekday"></div>

			<div
				v-for="(p,day) in latsDay"
				:class="{'select':selectItem(day)}"
				class="day"
				@click="newSelectDay(day+1)"
			>
				{{day+1}}
			</div>
		</div>

		<div class="change-language" @click="changeLanguage">{{ languageRus ? 'Eng' : 'Рус' }}</div>
	</div>
</template>

<script setup>
	import {computed, onMounted, onUnmounted, ref} from "vue";
	const props = defineProps({
		select: String
	});
	const emit = defineEmits(['change','close']);
	const containerRef = ref(null);
	const weekday = [['Mon','Пон'], ['Tue','Вт'], ['Wed','Ср'], ['Thu','Чт'], ['Fri','Пт'], ['Sat','Сб'], ['Sun','Вс']];
	const months = [['January','Январь'], ['February','Февраль'],['March','Март'],['April','Апрель'],['May','Май'],['June','Июнь'],
		['July','Июль'],['August','Август'],['September','Сентябрь'],['October','Октябрь'],['November','Ноябрь'],['December','Декабрь']];
	const currentDate = ref(new Date(props.select));
	const languageRus = ref(false);
	const select = ref(currentDate.value);
	const latsDay = new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0).getDate();
	const currentMonthYear = computed(()=> {
		return select.value ? months[select.value.getMonth()][+languageRus.value] + ' '+ select.value.getFullYear() : ''
	});
	const firstWeekday = computed(() => {
		if(!select.value) return 0;
		const firstDay = new Date(select.value.getFullYear(), select.value.getMonth(), 1);
		return (firstDay.getDay() + 6) % 7;
	});
	const selectItem = computed(() => (day) => {
		if (!currentDate.value) return false;
		return day === currentDate.value.getDate()-1 &&
			select.value.getMonth() === currentDate.value.getMonth() &&
			select.value.getFullYear() === currentDate.value.getFullYear();
	});

	function changeMonth(operation){
		const newDate = new Date(select.value);
		newDate.setMonth(newDate.getMonth() + (operation === '+' ? 1 : -1));
		select.value = newDate;
	}

	function newSelectDay(day){
		const date = `${select.value.getFullYear()}.${select.value.getMonth()+1}.${day}`;
		emit('change', date);
	}

	function changeLanguage(){
		languageRus.value = !languageRus.value;
		localStorage.setItem('calendar-language', languageRus.value);
	}

	const handleClickOutside = (event) => {
		if (containerRef.value && !containerRef.value.contains(event.target)) {
			emit('close');
		}
	};

	onMounted(() => {
		const savedLanguage = localStorage.getItem('calendar-language');
		if (savedLanguage) {
			languageRus.value = !!savedLanguage;
		}
		document.addEventListener('click', handleClickOutside);

	});

	onUnmounted(() => {
		document.removeEventListener('click', handleClickOutside);
	});

</script>

<style scoped>
.current-month-year{
	white-space: nowrap;
}

.calendar-container{
	border: 1px solid black;
	border-radius: 20px;
	max-width: 220px;
	padding: 5px;
}

.calendar-header{
	display: flex;
	align-items: center;
	justify-content: space-between;
	height: 20px;
}

.calendar-grid{
	display: grid;
	grid-template-columns: repeat(7, 1fr);
	gap: 4px;
	font-size: 12px;
	margin-top: 10px;
}

.grid{
	display: flex;
	align-items: center;
	justify-content: center;
}

.calendar-day{
	display: grid;
	grid-template-columns: repeat(7, 1fr);
	gap: 5px;
}

.day{
	transition: .3s ease;
	cursor: pointer;
	text-align: center;
}

.day:hover{
	background: green;
}

.select{
	border: 1px solid blue;
}

.change-language{
	background: darkgrey;
	width: fit-content;
	color: #464646;
	cursor: pointer;
	margin-top: 5px;
	margin-left: auto;
	margin-right: 5px;
	transition: .3s ease;
}

.change-language:hover{
	background: #8d8d8d;
}
</style>