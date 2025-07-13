<template>
  <div class="calendar-container">
    <!-- Calendar Controls -->
    <div class="calendar-controls">
      <div class="view-controls">
        <button 
          v-for="view in views" 
          :key="view.value"
          @click="changeView(view.value)"
          :class="['view-btn', { active: currentView === view.value }]"
        >
          {{ view.label }}
        </button>
      </div>
      
      <div class="nav-controls">
        <button @click="goToPrev()" class="nav-btn">‹ Prev</button>
        <button @click="goToToday()" class="today-btn">Today</button>
        <button @click="goToNext()" class="nav-btn">Next ›</button>
      </div>
    </div>

    <!-- FullCalendar -->
    <div class="calendar-wrapper">
      <FullCalendar
        ref="calendar"
        :options="calendarOptions"
      />
    </div>

    <!-- Event Creation Modal -->
    <div v-if="showEventModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>{{ editingEvent ? 'Edit Event' : 'Create New Event' }}</h3>
          <button @click="closeModal" class="close-btn">×</button>
        </div>
        
        <form @submit.prevent="saveEvent" class="event-form">
          <div class="form-group">
            <label for="title">Event Title *</label>
            <input
              id="title"
              v-model="eventForm.title"
              type="text"
              placeholder="Enter event title..."
              required
            />
          </div>

          <div class="form-row">
            <div class="form-group">
              <label for="start-date">Start Date *</label>
              <input
                id="start-date"
                v-model="eventForm.startDate"
                type="date"
                required
              />
            </div>
            
            <div class="form-group">
              <label for="start-time">Start Time</label>
              <input
                id="start-time"
                v-model="eventForm.startTime"
                type="time"
              />
            </div>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label for="end-date">End Date</label>
              <input
                id="end-date"
                v-model="eventForm.endDate"
                type="date"
              />
            </div>
            
            <div class="form-group">
              <label for="end-time">End Time</label>
              <input
                id="end-time"
                v-model="eventForm.endTime"
                type="time"
              />
            </div>
          </div>

          <div class="form-group">
            <label for="description">Description</label>
            <textarea
              id="description"
              v-model="eventForm.description"
              placeholder="Event description (optional)..."
              rows="3"
            ></textarea>
          </div>

          <div class="form-group">
            <label for="color">Color</label>
            <div class="color-picker">
              <input
                id="color"
                v-model="eventForm.color"
                type="color"
              />
              <span class="color-preview" :style="{ backgroundColor: eventForm.color }"></span>
            </div>
          </div>

          <div class="form-actions">
            <button type="button" @click="closeModal" class="btn-secondary">Cancel</button>
            <button v-if="editingEvent" type="button" @click="deleteEvent" class="btn-danger">Delete</button>
            <button type="submit" class="btn-primary">
              {{ editingEvent ? 'Update Event' : 'Create Event' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import type { DateSelectArg, EventClickArg, EventDropArg, EventApi } from '@fullcalendar/core'

// Calendar instance reference
const calendar = ref()

// Calendar state
const currentView = ref('dayGridMonth')
const showEventModal = ref(false)
const editingEvent = ref<EventApi | null>(null)

// Views configuration
const views = [
  { label: 'Month', value: 'dayGridMonth' }
]

// Event form data
const eventForm = reactive({
  title: '',
  startDate: '',
  startTime: '',
  endDate: '',
  endTime: '',
  description: '',
  color: '#3788d8'
})

// Sample events data
const events = ref([
  {
    id: '1',
    title: 'Team Meeting',
    start: '2025-07-14T10:00:00',
    end: '2025-07-14T11:00:00',
    backgroundColor: '#3788d8',
    borderColor: '#3788d8'
  },
  {
    id: '2',
    title: 'Project Deadline',
    start: '2025-07-15',
    backgroundColor: '#f56565',
    borderColor: '#f56565',
    allDay: true
  },
  {
    id: '3',
    title: 'Client Call',
    start: '2025-07-16T14:00:00',
    end: '2025-07-16T15:00:00',
    backgroundColor: '#48bb78',
    borderColor: '#48bb78'
  }
])

// Calendar configuration
const calendarOptions = computed(() => ({
  plugins: [dayGridPlugin, interactionPlugin],
  headerToolbar: false,
  initialView: 'dayGridMonth',
  height: 'auto',
  events: events.value,
  selectable: true,
  selectMirror: true,
  editable: true,
  droppable: true,
  select: handleDateSelect,
  eventClick: handleEventClick,
  eventDrop: handleEventDrop,
  dayMaxEvents: true,
  weekends: true,
  eventTimeFormat: {
    hour: 'numeric',
    minute: '2-digit',
    omitZeroMinute: false,
    meridiem: 'short'
  }
}))

// Calendar controls

function goToPrev() {
  calendar.value?.getApi().prev()
}

function goToNext() {
  calendar.value?.getApi().next()
}

function goToToday() {
  calendar.value?.getApi().today()
}

// Calendar interaction handlers
function handleDateSelect(selectInfo: DateSelectArg) {
  const startDate = selectInfo.start
  const endDate = selectInfo.end
  
  // Open event creation modal with pre-filled dates
  eventForm.startDate = formatDate(startDate)
  eventForm.startTime = selectInfo.allDay ? '' : formatTime(startDate)
  eventForm.endDate = formatDate(endDate)
  eventForm.endTime = selectInfo.allDay ? '' : formatTime(endDate)
  eventForm.title = ''
  eventForm.description = ''
  eventForm.color = '#3788d8'
  
  editingEvent.value = null
  showEventModal.value = true
  
  // Clear selection
  selectInfo.view.calendar.unselect()
}

function handleEventClick(clickInfo: EventClickArg) {
  const event = clickInfo.event
  
  // Populate form with event data
  eventForm.title = event.title
  eventForm.startDate = formatDate(event.start!)
  eventForm.startTime = event.allDay ? '' : formatTime(event.start!)
  eventForm.endDate = event.end ? formatDate(event.end) : eventForm.startDate
  eventForm.endTime = event.allDay || !event.end ? '' : formatTime(event.end)
  eventForm.description = event.extendedProps.description || ''
  eventForm.color = event.backgroundColor || '#3788d8'
  
  editingEvent.value = event
  showEventModal.value = true
}

function handleEventDrop(dropInfo: EventDropArg) {
  // Event has been moved - you can add additional logic here
  console.log('Event moved:', dropInfo.event.title, 'to', dropInfo.event.start)
}

// Calendar controls
function changeView(viewName: string) {
  currentView.value = viewName
  calendar.value?.getApi().changeView(viewName)
}

// Event management
function saveEvent() {
  if (!eventForm.title.trim()) return
  
  const startDateTime = combineDateTime(eventForm.startDate, eventForm.startTime)
  const endDateTime = combineDateTime(eventForm.endDate || eventForm.startDate, eventForm.endTime)
  
  const eventData = {
    title: eventForm.title,
    start: startDateTime,
    end: endDateTime || undefined,
    backgroundColor: eventForm.color,
    borderColor: eventForm.color,
    extendedProps: {
      description: eventForm.description
    }
  }
  
  if (editingEvent.value) {
    // Update existing event
    editingEvent.value.setProp('title', eventData.title)
    editingEvent.value.setStart(eventData.start)
    if (eventData.end && eventData.end !== '') {
      editingEvent.value.setEnd(eventData.end)
    }
    editingEvent.value.setProp('backgroundColor', eventData.backgroundColor)
    editingEvent.value.setProp('borderColor', eventData.borderColor)
    editingEvent.value.setExtendedProp('description', eventData.extendedProps.description)
  } else {
    // Create new event
    calendar.value?.getApi().addEvent({
      ...eventData,
      id: Date.now().toString()
    })
  }
  
  closeModal()
}

function deleteEvent() {
  if (editingEvent.value) {
    editingEvent.value.remove()
    closeModal()
  }
}

function closeModal() {
  showEventModal.value = false
  editingEvent.value = null
  resetForm()
}

function resetForm() {
  eventForm.title = ''
  eventForm.startDate = ''
  eventForm.startTime = ''
  eventForm.endDate = ''
  eventForm.endTime = ''
  eventForm.description = ''
  eventForm.color = '#3788d8'
}

// Utility functions
function formatDate(date: Date): string {
  return date.toISOString().split('T')[0]
}

function formatTime(date: Date): string {
  return date.toTimeString().slice(0, 5)
}

function combineDateTime(date: string, time: string): string {
  if (!date) return ''
  return time ? `${date}T${time}:00` : date
}

</script>

<style scoped>
.calendar-container {
  max-width: 1200px;
  margin: 0 auto;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.calendar-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 30px;
  background: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
}

.view-controls {
  display: flex;
  gap: 8px;
}

.view-btn {
  padding: 8px 16px;
  border: 1px solid #d1d5db;
  background: white;
  color: #374151;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 14px;
  font-weight: 500;
}

.view-btn:hover {
  background: #f3f4f6;
}

.view-btn.active {
  background: #3b82f6;
  color: white;
  border-color: #3b82f6;
}

.nav-controls {
  display: flex;
  gap: 8px;
  align-items: center;
}

.nav-btn, .today-btn {
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  background: white;
  color: #374151;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 14px;
  font-weight: 500;
}

.nav-btn:hover, .today-btn:hover {
  background: #f3f4f6;
}

.today-btn {
  background: #10b981;
  color: white;
  border-color: #10b981;
}

.today-btn:hover {
  background: #059669;
}

.calendar-wrapper {
  padding: 30px;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 12px;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 30px;
  border-bottom: 1px solid #e2e8f0;
}

.modal-header h3 {
  color: #1f2937;
  font-size: 1.5rem;
  font-weight: 600;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  color: #6b7280;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: background-color 0.2s;
}

.close-btn:hover {
  background: #f3f4f6;
}

.event-form {
  padding: 30px;
}

.form-group {
  margin-bottom: 20px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  color: #374151;
  font-weight: 500;
  font-size: 14px;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.2s;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.color-picker {
  display: flex;
  align-items: center;
  gap: 10px;
}

.color-picker input[type="color"] {
  width: 40px;
  height: 40px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  cursor: pointer;
}

.color-preview {
  width: 20px;
  height: 20px;
  border-radius: 4px;
  border: 1px solid #d1d5db;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #e2e8f0;
}

.btn-primary, .btn-secondary, .btn-danger {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: #3b82f6;
  color: white;
}

.btn-primary:hover {
  background: #2563eb;
}

.btn-secondary {
  background: #6b7280;
  color: white;
}

.btn-secondary:hover {
  background: #4b5563;
}

.btn-danger {
  background: #ef4444;
  color: white;
}

.btn-danger:hover {
  background: #dc2626;
}

/* FullCalendar custom styles */
:deep(.fc-event) {
  cursor: pointer;
  border-radius: 4px;
  font-weight: 500;
}

:deep(.fc-event:hover) {
  opacity: 0.8;
}

:deep(.fc-daygrid-event) {
  padding: 2px 4px;
}

:deep(.fc-timegrid-event) {
  padding: 2px 4px;
}

:deep(.fc-day-today) {
  background-color: rgba(59, 130, 246, 0.1) !important;
}

:deep(.fc-button-primary) {
  background-color: #3b82f6;
  border-color: #3b82f6;
}

:deep(.fc-button-primary:hover) {
  background-color: #2563eb;
  border-color: #2563eb;
}
</style>
