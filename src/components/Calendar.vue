<script setup lang="ts">
import { ref } from "vue";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";

const calendarRef = ref(null);

const calendarOptions = ref({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: "dayGridMonth",
  headerToolbar: {
    left: "prev,next,today",
    center: "title",
    right: "dayGridMonth,dayGridWeek,dayGridDay",
  },
  editable: true,
  selectable: true,
  selectMirror: true,
  dayMaxEvents: true,
  weekends: true,
  events: [],
  dateClick: handleDateClick,
  eventClick: handleEventClick,
  select: handleSelect,
});

const emit = defineEmits(["date-click", "event-click", "select"]);

function handleDateClick(arg: any) {
  emit("date-click", arg);
}

function handleEventClick(arg: any) {
  emit("event-click", arg);
}

function handleSelect(arg: any) {
  emit("select", arg);
}

// Method to add transaction events to calendar
function addEvents(transactions: any) {
  const events = transactions.map((tx: any) => ({
    id: tx.id,
    title: `${tx.type}: ${tx.amount}`,
    start: tx.date,
    allDay: true,
    backgroundColor: tx.type === "Expense" ? "#f87171" : "#4ade80",
    borderColor: tx.type === "Expense" ? "#ef4444" : "#22c55e",
    textColor: "#ffffff",
    extendedProps: { transaction: tx },
  }));

  calendarOptions.value.events = events;
}

// Expose method to parent component
defineExpose({ addEvents });
</script>

<template>
  <div class="w-full h-full">
    <FullCalendar
      ref="calendarRef"
      :options="calendarOptions"
      class="w-full h-full"
    />
  </div>
</template>

<style>
/* Minimal styling for font and colors */
.fc {
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial,
    sans-serif;
  font-size: 0.9em !important;
}

/* Style the "+more" links to be centered */
.fc-daygrid-more-link {
  margin: 0 !important;
  text-align: center !important;
  background-color: rgba(0, 0, 0, 0.05) !important;
  border-radius: 3px !important;
  padding: 1px 0 !important;
  font-size: 0.65em !important;
  color: rgba(0, 0, 0, 0.6) !important;
}

/* Slightly reduce the height of event elements */
.fc-event {
  padding: 1px 2px !important;
  font-size: 0.7em !important;
  line-height: 1.2 !important;
}

/* Today's date styling */
.fc-day-today {
  background-color: rgba(234, 179, 8, 0.08) !important;
}

.fc-day-today .fc-daygrid-day-number {
  color: #b45309 !important;
  font-weight: bold !important;
}

/* Basic title styling */
.fc-toolbar-title {
  font-size: 1.2em !important;
  font-weight: 600 !important;
  color: #92400e !important;
}

/* Minimal button styling to match theme */
.fc-button {
  background-color: rgba(245, 158, 11, 0.1) !important;
  border: 1px solid rgba(245, 158, 11, 0.2) !important;
  color: #92400e !important;
  font-size: 0.8em !important;
  padding: 0.3em 0.6em !important;
  height: auto !important;
}

.fc-button:hover {
  background-color: rgba(245, 158, 11, 0.2) !important;
}

.fc-button-active {
  background-color: rgba(245, 158, 11, 0.3) !important;
}

/* Make the calendar fill the container */
.fc {
  height: 100% !important;
}

/* Reduce toolbar size a bit */
.fc-header-toolbar {
  margin-bottom: 0.8em !important;
}

/* Make the next/prev buttons narrower */
.fc-prev-button,
.fc-next-button {
  padding-left: 0.4em !important;
  padding-right: 0.4em !important;
}
</style>
