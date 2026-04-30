<template>
  <div class="appointments-calendar">
    <div class="calendar-header">
      <button class="btn btn-outline-secondary btn-sm" type="button" @click="goPrevMonth">
        Prev
      </button>
      <div class="calendar-title">{{ monthLabel }}</div>
      <button class="btn btn-outline-secondary btn-sm" type="button" @click="goNextMonth">
        Next
      </button>
    </div>

    <div class="calendar-grid">
      <div v-for="dayName in dayNames" :key="dayName" class="calendar-cell calendar-day-name">
        {{ dayName }}
      </div>
      <button
        v-for="day in calendarDays"
        :key="day.key"
        class="calendar-cell calendar-day"
        :class="{
          'is-empty': !day.date,
          'is-selected': day.dateKey === selectedDate,
          'has-appointments': day.hasAppointments,
        }"
        type="button"
        :disabled="!day.date"
        @click="selectDay(day.dateKey)"
      >
        <span v-if="day.date" class="day-number">{{ day.dayNumber }}</span>
        <span v-if="day.hasAppointments" class="day-dot"></span>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

type AppointmentDay = {
  appointmentId: number
  dateTime: string
}

const props = defineProps<{
  appointments: AppointmentDay[]
  selectedDate: string
}>()

const emit = defineEmits<{
  (e: 'select', dateKey: string): void
}>()

const dayNames = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
const monthCursor = ref(new Date())

const appointmentDateKeys = computed(() => {
  const keys = new Set<string>()
  props.appointments.forEach((appt) => {
    const key = toDateKey(new Date(appt.dateTime))
    keys.add(key)
  })
  return keys
})

const monthLabel = computed(() => {
  const date = monthCursor.value
  return date.toLocaleDateString('en-US', { month: 'long', year: 'numeric' })
})

const calendarDays = computed(() => {
  const firstDay = new Date(monthCursor.value.getFullYear(), monthCursor.value.getMonth(), 1)
  const lastDay = new Date(monthCursor.value.getFullYear(), monthCursor.value.getMonth() + 1, 0)
  const startOffset = firstDay.getDay()
  const totalDays = lastDay.getDate()
  const cells: Array<{
    key: string
    date?: Date
    dateKey?: string
    dayNumber?: number
    hasAppointments: boolean
  }> = []

  for (let i = 0; i < startOffset; i += 1) {
    cells.push({ key: `empty-${i}`, hasAppointments: false })
  }

  for (let day = 1; day <= totalDays; day += 1) {
    const date = new Date(monthCursor.value.getFullYear(), monthCursor.value.getMonth(), day)
    const dateKey = toDateKey(date)
    cells.push({
      key: dateKey,
      date,
      dateKey,
      dayNumber: day,
      hasAppointments: appointmentDateKeys.value.has(dateKey),
    })
  }

  const remainder = cells.length % 7
  if (remainder !== 0) {
    const fill = 7 - remainder
    for (let i = 0; i < fill; i += 1) {
      cells.push({ key: `tail-${i}`, hasAppointments: false })
    }
  }

  return cells
})

function toDateKey(date: Date): string {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

function selectDay(dateKey?: string) {
  if (!dateKey) return
  emit('select', dateKey)
}

function goPrevMonth() {
  const date = new Date(monthCursor.value)
  date.setMonth(date.getMonth() - 1)
  monthCursor.value = date
}

function goNextMonth() {
  const date = new Date(monthCursor.value)
  date.setMonth(date.getMonth() + 1)
  monthCursor.value = date
}
</script>

<style scoped>
.appointments-calendar {
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 16px;
  background: #fff;
}

.calendar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.calendar-title {
  font-weight: 700;
  color: #1a202c;
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 6px;
}

.calendar-cell {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  min-height: 44px;
  background: #f8fafc;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.calendar-day-name {
  background: transparent;
  border: none;
  font-weight: 600;
  color: #4a5568;
  min-height: 24px;
}

.calendar-day {
  cursor: pointer;
  transition: all 0.2s ease;
}

.calendar-day:disabled {
  cursor: default;
  background: transparent;
  border: none;
}

.calendar-day.has-appointments {
  border-color: #f97316;
}

.calendar-day.is-selected {
  background: #f97316;
  color: #fff;
  border-color: #f97316;
}

.day-number {
  font-weight: 600;
}

.day-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #f97316;
  position: absolute;
  bottom: 6px;
}

.calendar-day.is-selected .day-dot {
  background: #fff;
}
</style>

