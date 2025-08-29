<template>
  <div class="min-h-screen bg-white book-main">
    <div class="flex flex-col items-center gap-6 pb-16 px-4">
      <!-- Loading state while fetching appointment data -->
      <div v-if="loadingAppointment" class="w-full max-w-4xl flex items-center justify-center py-20">
        <div class="text-center space-y-4">
          <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-red-700 mx-auto"></div>
          <p class="text-gray-600">Loading appointment details...</p>
        </div>
      </div>

      <!-- Error state -->
      <div v-else-if="appointmentError" class="w-full max-w-4xl text-center py-20">
        <div class="space-y-4">
          <UIcon name="i-lucide-alert-circle" class="text-4xl text-red-500 mx-auto" />
          <h2 class="text-2xl font-bold text-black">Appointment Not Found</h2>
          <p class="text-gray-600">{{ appointmentError }}</p>
        </div>
      </div>

      <!-- Update form -->
      <div v-else class="w-full max-w-4xl overflow-hidden department-block">
        <div class="p-8 department-inner">
          <!-- Header -->
          <div class="text-center mb-8">
            <h1 class="font-bold text-4xl mb-3 text-black">Update Your Appointment</h1>
            <p class="text-lg text-gray-700 max-w-2xl mx-auto leading-relaxed">
              Modify your appointment details below
            </p>
          </div>

          <!-- Updated progress indicator for 4 steps -->
          <div class="mb-8 hidden md:block">
            <div class="flex items-center justify-center space-x-4">
              <div v-for="(step, index) in steps" :key="index" class="flex items-center">
                <div :class="[
                  'w-10 h-10 rounded-full flex items-center justify-center text-sm font-bold transition-all duration-200',
                  currentStep >= index + 1 
                    ? 'bg-red-700 text-white' 
                    : 'bg-gray-200 text-gray-500'
                ]">
                  {{ index + 1 }}
                </div>
                <div v-if="index < steps.length - 1" :class="[
                  'w-16 h-1 mx-2 transition-all duration-200',
                  currentStep > index + 1 ? 'bg-red-700' : 'bg-gray-200'
                ]"></div>
              </div>
            </div>
            <div class="text-center mt-4">
              <h2 class="text-xl font-bold text-black">{{ steps[currentStep - 1]?.title }}</h2>
              <p class="text-gray-600 text-sm">{{ steps[currentStep - 1]?.description }}</p>
            </div>
          </div>

          <!-- Current appointment info -->
          <div class="mb-8 p-6 bg-gray-50 rounded-xl border border-gray-200">
            <h3 class="font-bold text-lg text-black mb-4">Current Appointment</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm text-dark">
              <div class="flex items-center gap-2">
                <UIcon name="i-lucide-calendar" class="text-red-700" />
                <span class="text-black font-medium">{{ formatAppointmentDate(currentAppointment.startTime) }}</span>
              </div>
              <div class="flex items-center gap-2">
                <UIcon name="i-lucide-clock" class="text-red-700" />
                <span class="text-black font-medium">{{ formatAppointmentTime(currentAppointment.startTime) }}</span>
              </div>
              <div class="flex items-center gap-2">
                <UIcon name="i-lucide-user" class="text-red-700" />
                <span class="text-black font-medium">{{ currentAppointment.assignedUserName || 'Any available staff' }}</span>
              </div>
              <div class="flex items-center gap-2">
                <UIcon name="i-lucide-scissors" class="text-red-700" />
                <span class="text-black font-medium">{{ currentAppointment.title }}</span>
              </div>
            </div>
          </div>

          <div class="space-y-8">
            <!-- Step 1: Staff Selection -->
            <div v-if="currentStep === 1" class="space-y-6">
              <div class="text-center">
                <h2 class="text-2xl font-bold text-black mb-2">Change Stylist</h2>
                <p class="text-gray-700">Select a different stylist or keep current assignment</p>
              </div>

              <div v-if="loadingStaff" class="space-y-4">
                <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in 4" :key="i" />
              </div>
              <!-- Changed to 2 columns grid for staff selection -->
            <div v-else class="grid grid-cols-1 md:grid-cols-2 w-full gap-2 text-s">
  <div
    v-for="item in staffRadioItems"
    :key="item.value"
    @click="selectStaff(item.value)"
    :class="[ 
      'cursor-pointer p-3 border rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm w-full',
      selectedStaff === item.value
        ? 'bg-red-50 text-black border-red-700 shadow-sm'
        : 'bg-white text-black border-gray-200 hover:border-red-300'
    ]"
  >
    <div class="flex items-center gap-4">
      <div :class="[
        'p-3 rounded-full',
        selectedStaff === item.value ? 'bg-red-100' : 'bg-gray-100'
      ]">
        <UIcon :name="item.icon" :class="[
          'text-2xl',
          selectedStaff === item.value ? 'text-red-700' : 'text-gray-600'
        ]" />
      </div>
      <div>
        <span class="text-l font-medium text-black">{{ item.label }}</span>
        <span v-if="item.badge" :class="[
          'ml-3 px-3 py-1 rounded-full text-xs font-medium',
          selectedStaff === item.value ? 'bg-red-100 text-red-700' : 'bg-red-500 text-white'
        ]">{{ item.badge }}</span>
      </div>
    </div>
    <div v-if="selectedStaff === item.value" class="text-red-700">
      <UIcon name="i-lucide-check-circle" class="text-2xl" />
    </div>
  </div>
</div>

            </div>

            <!-- Step 2: Date & Time Selection -->
            <div v-if="currentStep === 2" class="space-y-6">
              <div class="text-center">
                <h2 class="text-2xl font-bold text-black mb-2">Change Date & Time</h2>
                <p class="text-gray-700">Select a new appointment slot</p>
              </div>

              <!-- Timezone indicator -->
              <div class="text-center mb-6">
                <div class="text-sm font-medium text-gray-600 uppercase tracking-wide">
                  TIME ZONE: MOUNTAIN TIME - EDMONTON (GMT-06:00)
                </div>
              </div>

              <!-- Date Slider -->
              <div class="bg-white rounded-xl border border-gray-200 p-6 shadow-sm">
                <div class="flex items-center justify-between mb-6">
                  <UButton
                    variant="ghost"
                    size="sm"
                    @click="navigateDate(-1)"
                    :disabled="currentDateIndex <= 0"
                    class="p-2"
                  >
                    <UIcon name="i-lucide-chevron-left" class="text-xl" />
                  </UButton>
                  
                  <div class="flex-1 grid grid-cols-3 gap-4 mx-4">
                    <div
                      v-for="(dateInfo, index) in visibleDates"
                      :key="dateInfo.dateString"
                      @click="selectDate(dateInfo)"
                      :class="[
                        'p-4 rounded-lg border-2 transition-all duration-200 text-center cursor-pointer hover:shadow-md',
                        selectedDateString === dateInfo.dateString
                          ? 'border-red-700 bg-red-50'
                          : 'border-gray-200 bg-gray-50 hover:border-red-300'
                      ]"
                    >
                      <div class="text-xs font-medium text-gray-500 uppercase tracking-wide mb-1">
                        {{ dateInfo.label }}
                      </div>
                      <div class="font-bold text-lg text-black">
                        {{ dateInfo.dayName }}
                      </div>
                      <div class="text-sm text-gray-600">
                        {{ dateInfo.dateDisplay }}
                      </div>
                    </div>
                  </div>
                  
                  <UButton
                    variant="ghost"
                    size="sm"
                    @click="navigateDate(1)"
                    :disabled="currentDateIndex >= availableDates.length - 3"
                    class="p-2"
                  >
                    <UIcon name="i-lucide-chevron-right" class="text-xl" />
                  </UButton>
                </div>
              </div>

              <!-- Time slots -->
              <div class="space-y-4 times-block">
                <div class="bg-white rounded-xl border border-gray-200 p-6 shadow-sm min-h-[400px]">
                  <div v-if="loadingSlots" class="space-y-3">
                    <div class="grid grid-cols-2 gap-3">
                      <USkeleton class="h-12 rounded-lg bg-gray-100" v-for="i in 8" :key="i" />
                    </div>
                  </div>
                  <div v-else-if="enabledSlotsForDate.length > 0" class="space-y-4">
                    <div class="text-sm text-gray-600 mb-3">
                      Available slots for {{ selectedDateString ? formatDateForDisplay(selectedDateString) : '' }}:
                    </div>
                    <div class="grid grid-cols-2 gap-3 max-h-80 overflow-y-auto pr-2">
                      <UButton
                        v-for="slot in enabledSlotsForDate"
                        :key="slot.time"
                        :color="selectedSlot === slot.time ? 'primary' : 'gray'"
                        :variant="selectedSlot === slot.time ? 'solid' : 'soft'"
                        size="sm"
                        class="py-3 transition-all duration-200 text-black border border-gray-300 rounded-lg justify-center hover:shadow-sm"
                        @click="selectTimeSlot(slot.time)"
                        :class="[
                          selectedSlot === slot.time ? 'bg-red-700 hover:bg-red-700 text-white border-red-700 shadow-sm' : 'hover:border-red-300'
                        ]"
                      >
                        {{ slot.time }}
                      </UButton>
                    </div>
                  </div>
                  <div v-else class="flex flex-col items-center justify-center h-full text-gray-500 space-y-4">
                    <UIcon name="i-lucide-calendar-x" class="text-4xl" />
                    <div class="text-center">
                      <p class="font-medium text-black">{{ selectedDateString ? 'No slots available' : 'Select a date first' }}</p>
                      <p class="text-sm text-gray-600">{{ selectedDateString ? 'Try choosing a different date' : 'Choose a date to see available times' }}</p>
                    </div>
                  </div>
                </div>
                
                <div v-if="selectedSlot" class="p-4 bg-red-50 rounded-xl border border-red-200">
                  <div class="text-center">
                    <div class="font-bold text-black text-lg">New Selected Time</div>
                    <div class="text-red-700 font-semibold">{{ selectedSlot }} MST</div>
                    <div class="text-sm text-gray-600 mt-1">{{ formatDateForDisplay(selectedDateString) }}</div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Step 3: Review & Confirm -->
            <div v-if="currentStep === 3" class="space-y-6">
              <div class="text-center">
                <h2 class="text-2xl font-bold text-black mb-2">Review Changes</h2>
                <p class="text-gray-700">Confirm your appointment updates</p>
              </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
  <!-- Current vs New Comparison -->
  <div class="space-y-4">
    <h3 class="font-bold text-lg text-black">Current Appointment</h3>
    <div class="p-4 bg-gray-50 rounded-lg space-y-3 text-black">
      <div class="flex items-center gap-2">
        <UIcon name="i-lucide-user" class="text-black" />
        <span class="text-sm">{{ currentAppointment.assignedUserName || 'Any available staff' }}</span>
      </div>
      <div class="flex items-center gap-2">
        <UIcon name="i-lucide-calendar" class="text-black" />
        <span class="text-sm">{{ formatAppointmentDate(currentAppointment.startTime) }}</span>
      </div>
      <div class="flex items-center gap-2">
        <UIcon name="i-lucide-clock" class="text-black" />
        <span class="text-sm">{{ formatAppointmentTime(currentAppointment.startTime) }}</span>
      </div>
    </div>
  </div>

  <div class="space-y-4">
    <h3 class="font-bold text-lg text-black">New Appointment</h3>
    <div class="p-4 bg-red-50 rounded-lg space-y-3 border border-red-200 text-black">
      <div class="flex items-center gap-2">
        <UIcon name="i-lucide-user" class="text-black" />
        <span class="text-sm font-medium">{{ getSelectedStaffName() }}</span>
        <span v-if="selectedStaff !== currentAppointment.assignedUserId" class="text-xs bg-red-100 text-black px-2 py-1 rounded">Changed</span>
      </div>
      <div class="flex items-center gap-2">
        <UIcon name="i-lucide-calendar" class="text-black" />
        <span class="text-sm font-medium">{{ formatDateForDisplay(selectedDateString) }}</span>
        <span v-if="selectedDateString !== formatAppointmentDateString(currentAppointment.startTime)" class="text-xs bg-red-100 text-black px-2 py-1 rounded">Changed</span>
      </div>
      <div class="flex items-center gap-2">
        <UIcon name="i-lucide-clock" class="text-black" />
        <span class="text-sm font-medium">{{ selectedSlot }}</span>
        <span v-if="selectedSlot !== formatAppointmentTime(currentAppointment.startTime)" class="text-xs bg-red-100 text-black px-2 py-1 rounded">Changed</span>
      </div>
    </div>
  </div>
</div>


              <!-- Changes Summary -->
              <div v-if="hasChanges" class="p-4 bg-blue-50 rounded-lg border border-blue-200">
                <h4 class="font-bold text-blue-800 mb-2">Summary of Changes:</h4>
                <ul class="text-sm text-blue-700 space-y-1">
                  <li v-if="selectedStaff !== currentAppointment.assignedUserId">
                    • Stylist changed to {{ getSelectedStaffName() }}
                  </li>
                  <li v-if="selectedDateString !== formatAppointmentDateString(currentAppointment.startTime)">
                    • Date changed to {{ formatDateForDisplay(selectedDateString) }}
                  </li>
                  <li v-if="selectedSlot !== formatAppointmentTime(currentAppointment.startTime)">
                    • Time changed to {{ selectedSlot }}
                  </li>
                </ul>
              </div>

              <!-- Update button moved to step 3 -->
         <div class="flex justify-center">
  <UButton
    type="button"
    color="primary"
    size="lg"
    :loading="updateLoading"
    :disabled="!hasChanges"
    class="bg-red-700 hover:bg-red-800 text-white cursor-pointer"
    @click="updateAppointment"
  >
    <UIcon name="i-lucide-save" class="mr-2" />
    {{ updateLoading ? 'Updating...' : 'Update Appointment' }}
  </UButton>
</div>

            </div>

            <!-- Step 4: Success page with back to home button -->
            <div v-if="currentStep === 4" class="space-y-6 text-center">
              <div class="space-y-4">
                <UIcon name="i-lucide-check-circle" class="text-6xl text-green-600 mx-auto" />
                <h2 class="text-3xl font-bold text-black">Appointment Updated Successfully!</h2>
                <p class="text-lg text-gray-700">Your appointment has been updated with the new details.</p>
              </div>

              <div class="p-6 bg-green-50 rounded-xl border border-green-200">
                <h3 class="font-bold text-lg text-green-800 mb-4">Updated Appointment Details</h3>
                <div class="space-y-3 text-left max-w-md mx-auto">
                  <div class="flex items-center gap-2">
                    <UIcon name="i-lucide-user" class="text-green-700" />
                    <span class="text-sm font-medium">{{ getSelectedStaffName() }}</span>
                  </div>
                  <div class="flex items-center gap-2">
                    <UIcon name="i-lucide-calendar" class="text-green-700" />
                    <span class="text-sm font-medium">{{ formatDateForDisplay(selectedDateString) }}</span>
                  </div>
                  <div class="flex items-center gap-2">
                    <UIcon name="i-lucide-clock" class="text-green-700" />
                    <span class="text-sm font-medium">{{ selectedSlot }}</span>
                  </div>
                  <div class="flex items-center gap-2">
                    <UIcon name="i-lucide-scissors" class="text-green-700" />
                    <span class="text-sm font-medium">{{ currentAppointment.title }}</span>
                  </div>
                </div>
              </div>

              <UButton
                type="button"
                color="primary"
                size="lg"
                class="bg-red-700 hover:bg-red-700 text-white px-8"
               
              >
                <UIcon name="i-lucide-home" class="mr-2" />
                Back to Home
              </UButton>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { CalendarDate, today, getLocalTimeZone } from '@internationalized/date'
import { useRoute } from 'vue-router'

const route = useRoute()

const currentStep = ref(1)
const steps = ref([
  { title: '', description: '' },
  { title: '', description: '' },
  { title: 'Review & Confirm', description: 'Confirm your changes' },
  { title: 'Success', description: 'Appointment updated successfully' }
])

// State management
const appointmentId = ref('')
const currentAppointment = ref({})
const loadingAppointment = ref(true)
const appointmentError = ref('')

// Form state
const selectedStaff = ref('')
const staffRadioItems = ref([])
const loadingStaff = ref(false)

const selectedCalendarDate = ref(null)
const selectedDateString = ref('')
const selectedSlot = ref('')
const slotsForDate = ref([])
const loadingSlots = ref(false)

const activeSlots = ref({})
const activeDay = ref('')
const calendarId = ref('')

const updateLoading = ref(false)
const updateSuccess = ref(false)

// Date navigation
const availableDates = ref([])
const currentDateIndex = ref(0)

const visibleDates = computed(() => {
  return availableDates.value.slice(currentDateIndex.value, currentDateIndex.value + 3)
})

const enabledSlotsForDate = computed(() => {
  return slotsForDate.value.filter(slot => !slot.isPast)
})

const hasChanges = computed(() => {
  const staffChanged = selectedStaff.value !== currentAppointment.value.assignedUserId
  const timeChanged = selectedSlot.value !== formatAppointmentTime(currentAppointment.value.startTime)
  const dateChanged = selectedDateString.value !== formatAppointmentDateString(currentAppointment.value.startTime)
  
  return staffChanged || timeChanged || dateChanged
})

const canProceedToNextStep = computed(() => {
  if (currentStep.value === 1) {
    return selectedStaff.value !== ''
  }
  if (currentStep.value === 2) {
    return selectedDateString.value !== '' && selectedSlot.value !== ''
  }
  return true
})

// Initialize component
onMounted(async () => {
  appointmentId.value = route.query.id
  if (!appointmentId.value) {
    appointmentError.value = 'No appointment ID provided in URL'
    loadingAppointment.value = false
    return
  }
  
  await fetchAppointmentDetails()
  await fetchStaffOptions()
  await fetchActiveSlots()
  generateAvailableDates()
})

// Fetch appointment details
async function fetchAppointmentDetails() {
  try {
    const response = await fetch(`https://restyle-api.netlify.app/.netlify/functions/getBooking?id=${appointmentId.value}`)
    const data = await response.json()
    
    console.log('[v0] API Response:', data) // Debug log to see actual response structure
    
    let appointmentData = null
    
    // Handle different response structures
    if (data && data.appointment) {
      // Response has appointment nested under 'appointment' property
      appointmentData = data.appointment
      console.log('[v0] Found appointment in nested structure:', appointmentData)
    } else if (data && data.id) {
      // Response has appointment data directly on root
      appointmentData = data
      console.log('[v0] Found appointment in root structure:', appointmentData)
    }
    
    if (appointmentData && appointmentData.id) {
      currentAppointment.value = appointmentData
      
      // Pre-populate form with current values
      selectedStaff.value = appointmentData.assignedUserId || 'any'
      
      // Set current date and time
      const appointmentDate = new Date(appointmentData.startTime)
      const dateString = appointmentDate.toISOString().split('T')[0]
      selectedDateString.value = dateString
      
      const [year, month, day] = dateString.split('-').map(Number)
      selectedCalendarDate.value = new CalendarDate(year, month, day)
      
      selectedSlot.value = formatAppointmentTime(appointmentData.startTime)
      calendarId.value = appointmentData.calendarId
      
      console.log('[v0] Appointment loaded successfully:', appointmentData.id)
    } else {
      console.log('[v0] Invalid response structure - no appointment found:', data)
      appointmentError.value = 'Appointment not found or invalid response'
    }
  } catch (error) {
    console.error('[v0] Error fetching appointment:', error)
    appointmentError.value = 'Failed to load appointment details'
  } finally {
    loadingAppointment.value = false
  }
}

// Fetch staff options for the service
async function fetchStaffOptions() {
  if (!currentAppointment.value.calendarId) return
  
  loadingStaff.value = true
  try {
    // Fetch service details to get team members
    const serviceRes = await fetch(`https://restyle-api.netlify.app/.netlify/functions/Services?id=${currentAppointment.value.groupId || 'default'}`)
    const serviceData = await serviceRes.json()
    
    const serviceObj = (serviceData.calendars || []).find(s => s.id === currentAppointment.value.calendarId)
    const teamMembers = serviceObj?.teamMembers || []

    const items = [{
      label: 'Any available staff',
      value: 'any',
      badge: 'Recommended',
      icon: 'i-lucide-user'
    }]

    const staffPromises = teamMembers.map(async member => {
      try {
        const staffRes = await fetch(`https://restyle-api.netlify.app/.netlify/functions/Staff?id=${member.userId}`)
        const staffData = await staffRes.json()
        return {
          label: staffData.name,
          value: staffData.id,
          icon: 'i-lucide-user'
        }
      } catch (e) {
        return null
      }
    })

    const staffResults = await Promise.all(staffPromises)
    const validStaff = staffResults.filter(Boolean)
    
    staffRadioItems.value = [...items, ...validStaff]
  } catch (error) {
    console.error('Error fetching staff:', error)
    staffRadioItems.value = [{
      label: 'Any available staff',
      value: 'any',
      badge: 'Recommended',
      icon: 'i-lucide-user'
    }]
  } finally {
    loadingStaff.value = false
  }
}

// Fetch active slots
async function fetchActiveSlots() {
  if (!currentAppointment.value.calendarId) return
  
  loadingSlots.value = true
  
  const serviceId = currentAppointment.value.calendarId
  const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value

  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/Activeslots?calendarId=${serviceId}`
  if (userId) {
    apiUrl += `&userId=${userId}`
  }

  try {
    const response = await fetch(apiUrl)
    const data = await response.json()

    if (data.slots && data.activeDay && data.calendarId) {
      activeSlots.value = data.slots
      activeDay.value = data.activeDay
      
      // Update slots for current selected date
      if (selectedDateString.value && data.slots[selectedDateString.value]) {
        const slotsForSelectedDate = data.slots[selectedDateString.value] || []
        const slotsWithStatus = slotsForSelectedDate.map(slot => ({
          time: slot,
          isPast: isSlotInPastMST(slot, selectedDateString.value)
        }))
        slotsForDate.value = slotsWithStatus
      }
    }
  } catch (error) {
    console.error('Error fetching active slots:', error)
    slotsForDate.value = []
  } finally {
    loadingSlots.value = false
  }
}

// Generate available dates (next 30 days)
function generateAvailableDates() {
  const dates = []
  const todayDate = new Date()
  
  for (let i = 0; i < 30; i++) {
    const date = new Date(todayDate)
    date.setDate(todayDate.getDate() + i)
    
    const dateString = date.toISOString().split('T')[0]
    const dayNames = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']
    const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
    
    dates.push({
      dateString,
      dayName: dayNames[date.getDay()],
      dateDisplay: `${monthNames[date.getMonth()]} ${date.getDate()}`,
      label: i === 0 ? 'Today' : i === 1 ? 'Tomorrow' : dayNames[date.getDay()]
    })
  }
  
  availableDates.value = dates
}

// Event handlers
async function fetchSlotsForDate(dateString) {
  if (!currentAppointment.value.calendarId || !dateString) {
    console.log('Missing required data for date-specific slot fetch')
    slotsForDate.value = []
    return
  }

  console.log('Fetching slots for specific date:', dateString)
  
  selectedSlot.value = ''
  loadingSlots.value = true

  // Check if we already have slots for this date from active slots
  if (activeSlots.value[dateString]) {
    const slotsForSelectedDate = activeSlots.value[dateString]
    const slotsWithStatus = slotsForSelectedDate.map(slot => ({
      time: slot,
      isPast: isSlotInPastMST(slot, dateString)
    }))
    
    slotsForDate.value = slotsWithStatus
    loadingSlots.value = false
    console.log('Using cached slots for date:', dateString, slotsWithStatus)
    return
  }

  // If not in cache, fetch from the original API for the specific date
  const serviceId = currentAppointment.value.calendarId
  const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value
  
  // Convert date string to timestamps for the original API
  const date = new Date(dateString + 'T00:00:00')
  const start = new Date(date)
  start.setHours(0, 0, 0, 0)
  const end = new Date(date)
  end.setHours(23, 59, 59, 999)

  const startDate = start.getTime()
  const endDate = end.getTime()

  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/Allstaffslot?calendarId=${serviceId}&startDate=${startDate}&endDate=${endDate}`
  if (userId) {
    apiUrl += `&userId=${userId}`
  }

  console.log('Fallback API URL for date:', apiUrl)

  try {
    const response = await fetch(apiUrl)
    const data = await response.json()
    console.log('Fallback slots API response:', data)
    
    const formatted = data.formattedSlots || {}
    const key = Object.keys(formatted)[0]
    const allSlots = key ? formatted[key] : []
    
    const slotsWithStatus = allSlots.map(slot => ({
      time: slot,
      isPast: isSlotInPastMST(slot, dateString)
    }))

    slotsForDate.value = slotsWithStatus
    console.log('Fallback slots loaded for date:', dateString, slotsWithStatus)
  } catch (error) {
    console.error('Error fetching slots for date:', error)
    slotsForDate.value = []
  } finally {
    loadingSlots.value = false
  }
}

function selectStaff(staffId) {
  selectedStaff.value = staffId
  fetchActiveSlots() // Refresh slots when staff changes
  // Auto-advance to step 2
  setTimeout(() => {
    currentStep.value = 2
  }, 300)
}

function selectDate(dateInfo) {
  console.log('Selecting date:', dateInfo.dateString)
  selectedDateString.value = dateInfo.dateString
  selectedSlot.value = '' // Clear selected slot when changing date
  
  const [year, month, day] = dateInfo.dateString.split('-').map(Number)
  selectedCalendarDate.value = new CalendarDate(year, month, day)
  
  // Always fetch slots when date is selected
  if (currentAppointment.value.calendarId && selectedStaff.value) {
    fetchSlotsForDate(dateInfo.dateString)
  }
}

function selectTimeSlot(time) {
  console.log('Selecting time slot:', time, 'for date:', selectedDateString.value)
  selectedSlot.value = time
  // Auto-advance to step 3 (summary)
  setTimeout(() => {
    currentStep.value = 3
  }, 300)
}

async function updateAppointment() {
  if (!hasChanges.value) return
  
  updateLoading.value = true
  updateSuccess.value = false
  
  try {
    // Build update URL with only changed parameters
    let updateUrl = `https://restyle-api.netlify.app/.netlify/functions/updateappointment?appointmentId=${appointmentId.value}`
    
    // Add time parameters if changed
    if (selectedSlot.value !== formatAppointmentTime(currentAppointment.value.startTime) || 
        selectedDateString.value !== formatAppointmentDateString(currentAppointment.value.startTime)) {
      
      const jsDate = new Date(selectedDateString.value + 'T00:00:00')
      const slotMatch = selectedSlot.value.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
      
      if (slotMatch) {
        let hour = parseInt(slotMatch[1])
        const minute = parseInt(slotMatch[2])
        const period = slotMatch[3].toUpperCase()
        
        if (period === 'PM' && hour !== 12) hour += 12
        if (period === 'AM' && hour === 12) hour = 0
        
        jsDate.setHours(hour, minute, 0, 0)
        
        // Convert to UTC (MST is UTC-7)
        const mstOffset = -7 * 60 * 60 * 1000
        const utcStartTime = new Date(jsDate.getTime() - mstOffset)
        
        // Calculate end time (assuming same duration)
        const originalStart = new Date(currentAppointment.value.startTime)
        const originalEnd = new Date(currentAppointment.value.endTime)
        const duration = originalEnd.getTime() - originalStart.getTime()
        const utcEndTime = new Date(utcStartTime.getTime() + duration)
        
        updateUrl += `&startTime=${encodeURIComponent(utcStartTime.toISOString())}`
        updateUrl += `&endTime=${encodeURIComponent(utcEndTime.toISOString())}`
      }
    }
    
    // Add staff parameter if changed
    if (selectedStaff.value !== currentAppointment.value.assignedUserId) {
      let assignedUserId = selectedStaff.value
      if (assignedUserId === 'any') {
        // Pick a random staff member
        const realStaff = staffRadioItems.value.filter(item => item.value !== 'any')
        if (realStaff.length > 0) {
          assignedUserId = realStaff[Math.floor(Math.random() * realStaff.length)].value
        }
      }
      updateUrl += `&assignedUserId=${assignedUserId}`
    }
    
    console.log('Update URL:', updateUrl)
    
    const response = await fetch(updateUrl)
    const data = await response.json()
    
    if (data.message && data.message.includes('successfully')) {
      updateSuccess.value = true
      // Update current appointment data
      currentAppointment.value = { ...currentAppointment.value, ...data.response }
      // Auto-advance to success step
      setTimeout(() => {
        currentStep.value = 4
      }, 500)
    } else {
      throw new Error(data.error || 'Update failed')
    }
    
  } catch (error) {
    console.error('Update error:', error)
    alert('Failed to update appointment. Please try again.')
  } finally {
    updateLoading.value = false
  }
}

function goBackHome() {
  // Navigate to home page - adjust URL as needed
  window.location.href = '/'
}

function resetForm() {
  // Reset to original appointment values
  selectedStaff.value = currentAppointment.value.assignedUserId || 'any'
  
  const appointmentDate = new Date(currentAppointment.value.startTime)
  const dateString = appointmentDate.toISOString().split('T')[0]
  selectedDateString.value = dateString
  
  const [year, month, day] = dateString.split('-').map(Number)
  selectedCalendarDate.value = new CalendarDate(year, month, day)
  
  selectedSlot.value = formatAppointmentTime(currentAppointment.value.startTime)
  updateSuccess.value = false
}

watch(selectedDateString, (newDateString, oldDateString) => {
  console.log('Date changed from', oldDateString, 'to', newDateString)
  selectedSlot.value = ''
  if (newDateString && currentStep.value === 2 && currentAppointment.value.calendarId && selectedStaff.value) {
    console.log('Fetching slots for new date:', newDateString)
    fetchSlotsForDate(newDateString)
  } else {
    console.log('Clearing slots - missing requirements:', {
      dateString: newDateString,
      step: currentStep.value,
      calendarId: currentAppointment.value.calendarId,
      staff: selectedStaff.value
    })
    slotsForDate.value = []
  }
})

function isSlotInPastMST(slotTime, dateString) {
  const now = new Date()
  const slotDate = new Date(dateString + 'T00:00:00')
  
  const slotMatch = slotTime.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
  if (!slotMatch) return false
  
  let hour = parseInt(slotMatch[1])
  const minute = parseInt(slotMatch[2])
  const period = slotMatch[3].toUpperCase()
  
  if (period === 'PM' && hour !== 12) hour += 12
  if (period === 'AM' && hour === 12) hour = 0
  
  slotDate.setHours(hour, minute, 0, 0)
  
  // Add 15-minute buffer to prevent booking slots too close to current time
  const bufferTime = 15 * 60 * 1000 // 15 minutes in milliseconds
  return slotDate.getTime() < (now.getTime() + bufferTime)
}

function getSelectedStaffName() {
  const staffItem = staffRadioItems.value.find(item => item.value === selectedStaff.value)
  return staffItem ? staffItem.label : 'Any available staff'
}

function formatDateForDisplay(dateString) {
  const date = new Date(dateString)
  const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
  return `${monthNames[date.getMonth()]} ${date.getDate()}, ${date.getFullYear()}`
}

function navigateDate(direction) {
  currentDateIndex.value += direction
}

// Utility functions for date/time formatting
function formatAppointmentDate(dateString) {
  if (!dateString) return ''
  const date = new Date(dateString)
  const options = { 
    weekday: 'long', 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric',
    timeZone: 'America/Edmonton'
  }
  return date.toLocaleDateString('en-US', options)
}

function formatAppointmentTime(dateString) {
  if (!dateString) return ''
  const date = new Date(dateString)
  const options = { 
    hour: 'numeric', 
    minute: '2-digit', 
    hour12: true,
    timeZone: 'America/Edmonton'
  }
  return date.toLocaleTimeString('en-US', options)
}

function formatAppointmentDateString(dateString) {
  if (!dateString) return ''
  const date = new Date(dateString)
  // Convert to Edmonton timezone and get ISO date string
  const edmontonDate = new Date(date.toLocaleString('en-US', { timeZone: 'America/Edmonton' }))
  return edmontonDate.toISOString().split('T')[0]
}

function previousStep() {
  if (currentStep.value > 1) {
    currentStep.value -= 1
  }
}
</script>

<style scoped>
/* Reuse the same styles from the original supabase.vue */
.book-main {
  font-family: 'Inter', sans-serif;
}


.department-inner {
  background: white;
  border-radius: 16px;
}

.same-block-content {
  max-width: 600px;
  margin: 0 auto;
}

.same-btn-prev-next {
  max-width: 400px;
  margin: 0 auto;
}

.times-block {
  max-width: 800px;
  margin: 0 auto;
}

@media (max-width: 768px) {
  .department-inner {
    padding: 1.5rem;
  }
  
  .same-btn-prev-next {
    flex-direction: column;
  }
  
  .times-block .grid-cols-2 {
    grid-template-columns: 1fr;
  }
}
</style>
