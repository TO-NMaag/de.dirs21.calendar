<template>
	<div>
		<v-sheet tile height="54" color="grey lighten-3" class="d-flex">
			<v-btn icon class="ma-2" @click="$refs.d21Calendar.prev()">
				<v-icon>mdi-chevron-left</v-icon>
			</v-btn>
			<v-select v-model="type" :items="types" dense outlined hide-details class="ma-2" label="type"></v-select>
			<v-select v-model="mode" :items="modes" dense outlined hide-details class="ma-2" label="event-overlap-mode"></v-select>
			<v-select v-model="weekday" :items="weekdays" dense outlined hide-details class="ma-2" label="weekdays"></v-select>
			<v-spacer></v-spacer>
			<v-btn icon class="ma-2" @click="$refs.d21Calendar.next()">
				<v-icon>mdi-chevron-right</v-icon>
			</v-btn>
		</v-sheet>
		<v-sheet tile height="54" color="grey lighten-3" class="d-flex">
			<v-slider v-model="interval" dense outlined hide-details thumb-label min="5" max="60" step="5" ticks class="ma-2" label="interval"></v-slider>
		</v-sheet>
		<v-sheet height="600">				
			<v-calendar ref="d21Calendar" class="d21Calendar"
			v-model="today"
			:value="today"
			:weekdays="weekday"
			:type="type"
			:events="events"
			:event-overlap-mode="mode"
			:event-overlap-threshold="60"
			:event-ripple="false"
			:locale="locale"
			:day-format="dayFormat"
			:interval-format="intervalFormat"
			:interval-minutes="interval"
			:interval-count="intervalCount"
			@chang="calendarChanged"
			@mousedown:event="startDrag"
			@mousedown:time="startTime"
			@mousemove:time="mouseMove"
			@mouseup:time="endDrag"
			@mouseleave.native="cancelDrag"
			>
			
				<template v-slot:interval="{ hour }">
				
					<!-- <div class="text-center">{{ hour }}</div> -->
				
				</template>
				
				<template v-slot:event="{ event, timeSummary, timed }">
						
						<div class="pl-1">
							
							<span class="v-event-summary">
								
								<strong>{{event.name}}</strong>
								<br>
								{{timeSummary()}}
								<br>
								{{event.availability}} Verfügarkeiten
								
							</span>
							
						</div>
						
						<div v-if="timed" class="v-event-drag-bottom" @mousedown.stop="extendBottom(event)"></div>
				
				</template>
				
				<template v-slot:day-body="{ date, week }">
				
					<div class="v-current-time" :class="{first: date === week[0].date}" :style="{ top: nowY }">
						
						<div class="v-current-time-time">
							{{ nowTime }}
						</div>
						
					</div>
				
				</template>
				
				&nbsp;
				
			</v-calendar>
		</v-sheet>
	</div>
	
</template>

<script>


  export default {
    name: 'DirsCalendar',

    data: () => ({
		
		// settings
		today: 				"2022-07-08",
		type: 				"day",
		types: 				["day", "week", "month", "4day"],
		mode: 				"column",
		modes: 				["stack", "column"],
		weekday: 			[0, 1, 2, 3, 4, 5, 6],
		weekdays: 			[
								{ text: "So - Sa", value: [0, 1, 2, 3, 4, 5, 6] },
								{ text: "Mo - So", value: [1, 2, 3, 4, 5, 6, 0] },
								{ text: "Mo - Fr", value: [1, 2, 3, 4, 5] },
								{ text: "Mo, Mi, Fr", value: [1, 3, 5] },
							],
		interval:			5,
		dragInterval:		5,
		
		// events
		events: 			[
								{
									name: 			"test event",
									start: 			"2022-07-08 09:00",
									end:			"2022-07-08 10:30",
									color:			"red",
									availability:	100,
								},
								{
									name: 		"test event",
									start: 		"2022-07-08 09:00",
									end:		"2022-07-08 10:30",
									color:		"green",
									availability:	100,
								},
								{
									name: 		"test event",
									start: 		"2022-07-08 09:00",
									end:		"2022-07-08 10:30",
									color:		"purple",
									availability:	100,
								},
								{
									name: 		"test event",
									start: 		"2022-07-08 09:00",
									end:		"2022-07-08 10:30",
									color:		"brown",
									availability:	100,
								},
								{
									name: 		"daily",
									start: 		"2022-07-08 08:00",
									end:		"2022-07-08 14:00",
									availability:	100,
								},
								{
									name: 		"Ganztägig",
									start:		"2022-07-08",
									end:		"2022-07-08",
									color:		"yellow",
									availability:	100,
								},
								{
									name: 		"Mehrere Tage",
									start:		"2022-07-05",
									end:		"2022-07-09",
									color:		"#ffaa00",
									availability:	100,
								},
								{
									name: 		"Ganzer Block",
									start:		"2022-07-08 00:00",
									end:		"2022-07-09 00:00",
									color:		"#00ffaa",
									availability:	100,
								},
							],
		lastEvent:			'',
		createEvent:		null,
		createStart:		null,
		extendOriginal:		null,
		
		// environment
		isMounted: 			false,
		locale: 			"de-DE",
		
		// Drag and drop
		dragEvent:			null,
		dragStart:			null,
		
		
    }),
	
	computed: {
		intervalCount() {
			return (24*60/this.interval);
		},
		nowY() {
			const cal = this.$refs.d21Calendar;
			if (!cal && !this.isMounted) {
				return -1;
			}
			
			return cal.timeToY(cal.times.now) + 'px';
		},
		nowTime() {
			const cal = this.$refs.d21Calendar;
			if (!cal && !this.isMounted) {
				return -1;
			}
			
			return cal.formatTime(cal.times.now);
		},
	},
	
	methods: {
		calendarChanged() {
			
		},
		dayFormat(day) {
			return day.day;
		},
		intervalFormat(interval) {
			return interval.time;
		},
		startDrag(e) {
			if (e.event && e.timed) {
				this.dragEvent = e.event;
				this.dragTime = null;
				this.extendOriginal = null;
			}
			
			this.lastEvent = "startDrag";
		},
		startTime(e) {
			const mouse = this.toDate(e);
			
			if (this.dragEvent && this.dragTime === null) {
				const start = this.toDate(this.dragEvent.start);
				
				this.dragTime = mouse.getTime() - start.getTime();
				
			} else {
				this.createStart = this.roundTime(mouse.getTime());
				this.createEvent = {
					name:		"unbenannt",
					start:		this.toTimestamp(new Date(this.createStart)),
					end:		this.toTimestamp(new Date(this.createStart)),
					color:		"#ff0000",
				};
				this.events.push(this.createEvent);
			}
			this.lastEvent = "startTime";
		},
		extendBottom(event) {
			this.createEvent = event;
			this.createStart = this.toDate(event.start).getTime();
			this.extendOriginal = event.end;
		},
		mouseMove(e) {
			if (this.dragEvent && this.dragTime != null) {
				const start = this.toDate(this.dragEvent.start);
				const end = this.toDate(this.dragEvent.end);
				const duration = end.getTime() - start.getTime();
				const mouse = this.toDate(e);
				
				const newStartTime = mouse.getTime() - this.dragTime;
				const newStart = new Date(this.roundTime(newStartTime));
				const newEnd = new Date(newStart.getTime() + duration);
				
				this.dragEvent.start = this.toTimestamp(newStart);
				this.dragEvent.end = this.toTimestamp(newEnd);
			}
			else if (this.createEvent && this.createStart != null) {
				const mouse = this.toDate(e).getTime();
				const mouseRounded = this.roundTime(mouse, false);
				const min = Math.min(mouseRounded, this.createStart);
				const max = Math.max(mouseRounded, this.createStart);
				
				this.createEvent.start = this.toTimestamp(new Date(min));
				this.createEvent.end = this.toTimestamp(new Date(max));
			}
		},
		endDrag(e) {
			this.dragTime = null;
			this.dragEvent = null;
			this.createEvent = null;
			this.createStart = null;
			this.extendOriginal = null;
			
			this.lastEvent = "endDrag";
		},
		cancelDrag(e) {
			if (this.createEvent) {
				if (this.extendOriginal) {
					this.createEvent.end = this.extendOriginal;
				} else {
					const i = this.events.indexof(this.createEvent);
					if (i !== -1) {
						this.events.splice(i, 1);
					}
				}
			}
			
			this.createEvent = null;
			this.createStart = null;
			this.dragTime = null;
			this.dragEvent = null;
			
			this.lastEvent = "cancelDrag";
		},
		roundTime(time, down = true) {
			const roundDownTime = this.interval * 60 * 1000; // 15 min, adjust later to intervals
			
			return down
				? time - time % roundDownTime
				: time + (roundDownTime - (time % roundDownTime));
		},
		toDate(tms) {
			return typeof tms === "string"
				? new Date(tms)
				: new Date(tms.year, tms.month - 1, tms.day, tms.hour, tms.minute);
		},
		toTimestamp(date) {
			return `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()} ${date.getHours()}:${date.getMinutes()}`;
		}
	},
	
	mounted() {
		const cal = this.$refs.d21Calendar;
		
		this.isMounted = true;
		
		const minutes = cal.times.now.hour * 60 + cal.times.now.minute;
		const firstTime = Math.max(0, minutes - (minutes % 30) - 30);
		cal.scrollToTime(firstTime);
		
		setInterval(() => cal.updateTimes(), 60 * 1000);
	}
  }
</script>


<style lang="less">

	.v-calendar {
		user-select: none;
		-webkit-user-select: none;
	}
	
	.v-event-drag-bottom {
		position: absolute;
		left: 0;
		right: 0;
		bottom: 4px;
		height: 4px;
		cursor: ns-resize;
	}
	
	.v-event-drag-bottom::after {
		display: none;
		position: absolute;
		left: 50%;
		height: 4px;
		border-top: 1px solid white;
		border-bottom: 1px solid white;
		width: 20px;
		margin-left: -10px;
		opacity: 0.8;
		content: "";
	}
	
	.v-event-timed:hover .v-event-drag-bottom::after {
		display: block;
	}
	
	.v-current-time {
		height: 1px;
		background-color: red;
		position: absolute;
		left: -1px;
		right: 0px;
		pointer-events: none;
		
		.first::after {
			content: "";
			position: absolute;
			background-color: red;
			width: 7px;
			height: 7px;
			border-radius: 3.5px;
			margin-top: -3px;
			margin-left: -3px;
		}
		
		.v-current-time-time {
			display: none;
		}
		
		&.first .v-current-time-time {
			display: block;
			position: absolute;
			left: -60px;
			color: red;
			font-size: 10px;
			padding: 1px;
			background-color: white;
			width: 50px;
			height: 20px;
			text-align: center;
			top: -9px;
		}
		
	}

	.d21Calendar {
		width: 100%;
	}

</style>