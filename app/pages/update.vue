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

              <!-- Summary | Staff | Guests cards -->
              <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-2 max-w-3xl mx-auto">
                <!-- Summary Card -->
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200">
                  <div class="font-bold text-lg mb-1 text-black">Summary</div>
                  <div class="text-gray-700">{{ currentAppointment.title || 'Selected service' }}</div>
                  <div class="text-sm text-red-700 font-semibold mt-1">
                    Duration: {{ getServiceDurationMinutes() }} mins
                  </div>
                </div>
                <!-- Staff Card with Change button -->
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200 flex flex-col items-center justify-center">
                  <div class="font-bold text-lg mb-1 text-black">Staff</div>
                  <div class="text-red-700 font-semibold text-center mb-2">{{ getSelectedStaffName() }}</div>
                  <UButton
                    size="sm"
                    color="primary"
                    class="bg-red-700 hover:bg-red-700 text-white"
                    @click="currentStep = 1"
                  >
                    <UIcon name="i-lucide-user-switch" class="mr-1" /> Change Staff
                  </UButton>
                </div>
                <!-- Guests Card -->
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200 flex flex-col items-center justify-center">
                  <div class="font-bold text-lg mb-1 text-black">Guests</div>
                  <div v-if="!showGuestInput" class="flex flex-col items-center">
                    <div class="flex items-center justify-center gap-2 mb-2">
                      <UIcon name="i-lucide-user" class="text-2xl text-red-700" />
                      <span class="font-bold text-xl text-black">{{ guestCount }}</span>
                    </div>
                    <UButton
                      size="sm"
                      color="primary"
                      class="bg-red-700 hover:bg-red-700 text-white mt-2"
                      @click="showGuestInput = true"
                    >
                      <UIcon name="i-lucide-plus" class="mr-1" /> Add Guest
                    </UButton>
                  </div>
                  <div v-else class="w-full">
                    <label class="block font-semibold mb-2 text-black text-sm">Select Number of Guests</label>
                    <div class="flex justify-center quantity-guest">
                      <UInputNumber
                        v-model="guestCount"
                        :min="1"
                        :max="10"
                        size="md"
                        class="w-28"
                      />
                    </div>
                    <div class="text-center text-xs font-medium text-black bg-white rounded-lg p-2 border border-gray-200 mt-2">
                      You{{ guestCount > 1 ? ' and ' + (guestCount - 1) + ' guest' + (guestCount > 2 ? 's' : '') : '' }}
                    </div>
                    <UButton
                      size="xs"
                      color="gray"
                      variant="soft"
                      class="mt-2 border border-gray-200 w-full text-center justify-center p-[6px] text-white bg-[#751a29] cursor-pointer"
                      @click="resetGuestInput"
                    >
                      Cancel
                    </UButton>
                  </div>
                </div>
              </div>

              <!-- Timezone indicator -->
              <div class="text-center mb-6">
                <div class="text-sm font-medium text-gray-600 uppercase tracking-wide">
                  TIME ZONE: MOUNTAIN TIME - EDMONTON (GMT-06:00)
                </div>
              </div>

              <!-- Date Slider -->
              <div class="bg-white rounded-xl border border-gray-200 p-6 shadow-sm">
                <div v-if="datesLoading || visibleDates.length === 0" class="mb-6">
                  <div class="grid grid-cols-3 gap-4">
                    <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in 3" :key="i" />
                  </div>
                </div>
                <div v-else class="flex items-center justify-between mb-6">
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
                  <div v-if="loadingSlots || (!enabledSlotsForDate.length)" class="space-y-3">
                    <div class="grid grid-cols-2 gap-3">
                      <USkeleton class="h-12 rounded-lg bg-gray-100" v-for="i in 8" :key="i" />
                    </div>
                  </div>
                  <div v-else class="space-y-4">
                    <div class="text-sm text-gray-600 mb-3">
                      {{ slotStatusMessage }} for {{ selectedDateString ? formatDateForDisplay(selectedDateString) : '' }}:
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
        <span class="text-sm">{{ getCurrentStaffName() }}</span>
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
        <span v-if="selectedStaff && selectedStaff !== currentAppointment.assignedUserId" class="text-xs bg-red-100 text-black px-2 py-1 rounded">Changed</span>
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
                  <li v-if="selectedStaff && selectedStaff !== currentAppointment.assignedUserId">
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
                <h3 class="font-bold text-lg text-black mb-4">Updated Appointment Details</h3>
                <div class="space-y-3 text-left max-w-md mx-auto text-black">
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

const currentStep = ref(2)
const steps = ref([
  { title: '', description: '' },
  { title: '', description: '' },
  { title: '', description: '' }
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

// Working slots prefetched (7 working days, skipping weekends)
const workingSlots = ref({})
const workingSlotsLoaded = ref(false)

const updateLoading = ref(false)
const updateSuccess = ref(false)

// Guests (mirror logic from supabase.vue)
const guestCount = ref(1)
const showGuestInput = ref(false)
function resetGuestInput() {
  guestCount.value = 1
  showGuestInput.value = false
}

// Date navigation
const availableDates = ref([])
const currentDateIndex = ref(0)

const visibleDates = computed(() => {
  return availableDates.value.slice(currentDateIndex.value, currentDateIndex.value + 3)
})

// Loading state for dates to avoid initial empty flash
const datesLoading = ref(true)

const enabledSlotsForDate = computed(() => {
  return slotsForDate.value.filter(slot => {
    // Filter out past slots
    if (slot.isPast) return false
    
    // Filter out unavailable slots (you can add more logic here if needed)
    if (slot.isUnavailable) return false
    
    return true
  })
})

const slotStatusMessage = computed(() => {
  if (slotsForDate.value.length === 0) {
    return 'No slots found for this date'
  }
  
  const availableSlots = enabledSlotsForDate.value.length
  
  if (availableSlots === 0) {
    return 'No available slots for this date'
  }
  
  return `${availableSlots} slots available`
})

const hasChanges = computed(() => {
  const staffChanged = selectedStaff.value && selectedStaff.value !== currentAppointment.value.assignedUserId
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

// Add a new cache for slots
const slotsCache = ref({})
const slotsCacheTimestamp = ref({})

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
  await fetchWorkingSlots()
  // Generate dates after appointment details are loaded
  await generateAvailableDates()
  datesLoading.value = false
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
      
      // Default to 'any' so slots show by default on Step 2
      selectedStaff.value = 'any'
      
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

// Fetch 7 working days slots once based on next day from current booking
async function fetchWorkingSlots() {
  if (!currentAppointment.value.calendarId) return
  loadingSlots.value = true

  const serviceId = currentAppointment.value.calendarId
  // date param = next date after current booking date
  let startDateParam = ''
  try {
    const currentBookingDate = currentAppointment.value.startTime ? new Date(currentAppointment.value.startTime) : null
    if (currentBookingDate) {
      const nextDay = new Date(currentBookingDate)
      nextDay.setDate(nextDay.getDate() + 1)
      const yyyy = nextDay.getFullYear()
      const mm = String(nextDay.getMonth() + 1).padStart(2, '0')
      const dd = String(nextDay.getDate()).padStart(2, '0')
      startDateParam = `&date=${yyyy}-${mm}-${dd}`
    }
  } catch {}

  const apiUrl = `https://restyle-api.netlify.app/.netlify/functions/Workingslots?calendarId=${serviceId}${startDateParam}`

  try {
    const response = await fetch(apiUrl)
    const data = await response.json()
    if (data && data.slots) {
      workingSlots.value = data.slots
      workingSlotsLoaded.value = true
      // Prime first selected date/slots
      const dates = Object.keys(workingSlots.value).sort()
      if (dates.length > 0) {
        const first = dates[0]
        selectedDateString.value = first
        const [y, m, d] = first.split('-').map(Number)
        selectedCalendarDate.value = new CalendarDate(y, m, d)
        const slotsWithStatus = (workingSlots.value[first] || []).map(t => ({
          time: t,
          isPast: isSlotInPastMST(t, first),
          isUnavailable: false
        })).filter(s => !s.isPast)
        slotsForDate.value = slotsWithStatus
      }
    } else {
      console.error('Invalid working slots response:', data)
      workingSlots.value = {}
    }
  } catch (e) {
    console.error('Error fetching working slots:', e)
    workingSlots.value = {}
  } finally {
    loadingSlots.value = false
  }
}


async function generateAvailableDates() {
  const dates = []
  if (workingSlotsLoaded.value && Object.keys(workingSlots.value).length > 0) {
    const apiDates = Object.keys(workingSlots.value).sort()
    apiDates.forEach((dateString, idx) => {
      const [y, m, d] = dateString.split('-').map(Number)
      const date = new Date(y, m - 1, d)
      const dayName = date.toLocaleDateString('en-US', { weekday: 'long' })
      const dateDisplay = date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
      const label = idx === 0 ? 'Tomorrow' : dayName
      dates.push({ dateString, dayName, dateDisplay, label, isToday: false })
    })
  }
  availableDates.value = dates
}

// Use prefetched workingSlots for date selection
async function fetchSlotsForDate(dateString) {
  if (!dateString) {
    slotsForDate.value = []
    return
  }
  selectedSlot.value = ''
  const slotsForSelectedDate = workingSlots.value[dateString] || []
  const slotsWithStatus = slotsForSelectedDate.map(slot => ({
    time: slot,
    isPast: isSlotInPastMST(slot, dateString),
    isUnavailable: false
  })).filter(s => !s.isPast)
  slotsForDate.value = slotsWithStatus
}



// Removed old per-day API helper (using workingSlots instead)

// Function to filter slots by business hours
function filterSlotsByBusinessHours(slots, date) {
  if (!slots || slots.length === 0) return []
  
  const mountainTimeZone = 'America/Denver'
  // Determine weekday in Mountain Time
  const weekdayName = new Intl.DateTimeFormat('en-US', { weekday: 'long', timeZone: mountainTimeZone }).format(date)
  const weekdayToIndex = { Sunday: 0, Monday: 1, Tuesday: 2, Wednesday: 3, Thursday: 4, Friday: 5, Saturday: 6 }
  const dayOfWeek = weekdayToIndex[weekdayName] ?? date.getDay()
  
  // Business hours in Mountain Time: Mon-Wed/Fri 9-19, Thu 11-19, closed weekends
  let hours = null
  if (dayOfWeek === 4) {
    hours = { start: 11, end: 19 }
  } else if (dayOfWeek >= 1 && dayOfWeek <= 5) {
    hours = { start: 9, end: 19 }
  }
  if (!hours) return []
  
  return slots.filter(slot => {
    const timeMatch = slot.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
    if (!timeMatch) return false
    
    let hour = parseInt(timeMatch[1])
    const minute = parseInt(timeMatch[2])
    const period = timeMatch[3].toUpperCase()
    
    if (period === 'PM' && hour !== 12) hour += 12
    if (period === 'AM' && hour === 12) hour = 0
    
    const slotTime = hour + (minute / 60)
    return slotTime >= hours.start && slotTime < hours.end
  })
}

// Function to check if a slot is available for booking
async function checkSlotAvailability(slotTime, dateString, serviceId, userId) {
  try {
    // Convert slot time to timestamp
    const date = new Date(dateString + 'T00:00:00')
    const timeMatch = slotTime.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
    if (!timeMatch) return false
    
    let hour = parseInt(timeMatch[1])
    const minute = parseInt(timeMatch[2])
    const period = timeMatch[3].toUpperCase()
    
    if (period === 'PM' && hour !== 12) hour += 12
    if (period === 'AM' && hour === 12) hour = 0
    
    date.setHours(hour, minute, 0, 0)
    
    // Convert to UTC (MST is UTC-7)
    const mstOffset = -7 * 60 * 60 * 1000
    const utcTime = new Date(date.getTime() - mstOffset)
    
    // With workingSlots prefetch, assume availability if present in list
    const available = (workingSlots.value[dateString] || []).includes(slotTime)
    return available
  } catch (error) {
    console.error('Error checking slot availability:', error)
    return true // Assume available if check fails
  }
}

async function selectStaff(staffId) {
  selectedStaff.value = staffId
  // Slots are prefetched and independent of staff now
  if (availableDates.value.length > 0) {
    currentDateIndex.value = 0
    const firstAvailableDate = availableDates.value[0]
    selectedDateString.value = firstAvailableDate.dateString
    const [year, month, day] = firstAvailableDate.dateString.split('-').map(Number)
    selectedCalendarDate.value = new CalendarDate(year, month, day)
    await fetchSlotsForDate(firstAvailableDate.dateString)
  }
  
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
    // First, validate that the selected slot is still available
    if (selectedSlot.value !== formatAppointmentTime(currentAppointment.value.startTime) || 
        selectedDateString.value !== formatAppointmentDateString(currentAppointment.value.startTime)) {
      
      const isSlotStillAvailable = await validateSlotAvailability()
      if (!isSlotStillAvailable) {
        // Slot is no longer available, refresh slots and show error
        await refreshSlotsForCurrentDate()
        throw new Error('The selected time slot is no longer available. Please choose a different time.')
      }
    }
    
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
    if (selectedStaff.value && selectedStaff.value !== currentAppointment.value.assignedUserId) {
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
      // Handle specific error cases
      if (data.error && data.error.includes('no longer available')) {
        // Slot was taken, refresh and show better error
        await refreshSlotsForCurrentDate()
        throw new Error('The selected time slot is no longer available. Please choose a different time.')
      } else {
        throw new Error(data.error || data.details?.message || 'Update failed')
      }
    }
    
  } catch (error) {
    console.error('Update error:', error)
    
    // Show user-friendly error message
    if (error.message.includes('no longer available')) {
      alert('The selected time slot is no longer available. Please choose a different time.')
      // Go back to step 2 to let user select a new time
      currentStep.value = 2
    } else {
      alert('Failed to update appointment. Please try again.')
    }
  } finally {
    updateLoading.value = false
  }
}

// New function to validate slot availability before update
async function validateSlotAvailability() {
  if (!selectedDateString.value || !selectedSlot.value) return true
  
  try {
    // Fetch fresh slots for the selected date
    const serviceId = currentAppointment.value.calendarId
    const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value
    
    const date = new Date(selectedDateString.value + 'T00:00:00')
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

    const response = await fetch(apiUrl)
    const data = await response.json()
    
    if (data.formattedSlots) {
      const formatted = data.formattedSlots || {}
      const key = Object.keys(formatted)[0]
      const availableSlots = key ? formatted[key] : []
      
      // Check if our selected slot is still in the available slots
      return availableSlots.includes(selectedSlot.value)
    }
    
    return false
  } catch (error) {
    console.error('Error validating slot availability:', error)
    return false
  }
}

// New function to refresh slots for current date
async function refreshSlotsForCurrentDate() {
  if (selectedDateString.value) {
    // Clear cache for this date to force fresh fetch
    const serviceId = currentAppointment.value.calendarId
    const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value
    const cacheKey = `${serviceId}_${userId || 'any'}`
    
    if (slotsCache.value[cacheKey]) {
      delete slotsCache.value[cacheKey][selectedDateString.value]
    }
    
    // Fetch fresh slots
    await fetchSlotsForDate(selectedDateString.value)
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
  if (newDateString && currentStep.value === 2) {
    fetchSlotsForDate(newDateString)
  } else {
    slotsForDate.value = []
  }
})

function isSlotInPastMST(slotTime, dateString) {
  if (!dateString || !slotTime) return false
  
  // Get current time in MST (Mountain Standard Time - UTC-7)
  const now = new Date()
  const mstOffset = -7 * 60 * 60 * 1000 // MST is UTC-7
  const nowMST = new Date(now.getTime() + (now.getTimezoneOffset() * 60 * 1000) + mstOffset)
  
  // Parse the date string (YYYY-MM-DD format)
  const [year, month, day] = dateString.split('-').map(Number)
  const slotDate = new Date(year, month - 1, day) // month is 0-indexed in JS Date
  
  // Parse slot time (e.g., "2:30 PM" or "02:30 PM")
  const timeMatch = slotTime.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
  if (!timeMatch) return false
  
  let hour = parseInt(timeMatch[1])
  const minute = parseInt(timeMatch[2])
  const period = timeMatch[3].toUpperCase()
  
  if (period === 'PM' && hour !== 12) hour += 12
  if (period === 'AM' && hour === 12) hour = 0
  
  slotDate.setHours(hour, minute, 0, 0)
  
  // Convert slot time to MST for comparison
  const slotMST = new Date(slotDate.getTime() + (slotDate.getTimezoneOffset() * 60 * 1000) + mstOffset)
  
  // Get current booking time for comparison
  const currentBookingTime = currentAppointment.value.startTime ? new Date(currentAppointment.value.startTime) : null
  
  // If it's the same date as current booking, only show slots after current booking time
  if (currentBookingTime && dateString === formatAppointmentDateString(currentAppointment.value.startTime)) {
    const bookingMST = new Date(currentBookingTime.getTime() + (currentBookingTime.getTimezoneOffset() * 60 * 1000) + mstOffset)
    return slotMST.getTime() <= bookingMST.getTime()
  }
  
  // For today's date, add a buffer of 30 minutes to allow for booking time
  const today = new Date()
  const todayString = today.toISOString().split('T')[0]
  
  if (dateString === todayString) {
    const bufferTime = 30 * 60 * 1000 // 30 minutes buffer
    return slotMST.getTime() < (nowMST.getTime() - bufferTime)
  }
  
  // For other dates, compare with current time
  return slotMST < nowMST
}

// Add a function to get current staff name properly
function getCurrentStaffName() {
  if (!currentAppointment.value.assignedUserId) {
    return 'Any available staff'
  }
  
  // Find the staff name from the staff radio items
  const staffItem = staffRadioItems.value.find(item => item.value === currentAppointment.value.assignedUserId)
  return staffItem ? staffItem.label : 'Any available staff'
}

// Update the getSelectedStaffName function to handle empty selection
function getSelectedStaffName() {
  if (!selectedStaff.value) {
    return 'Please select a staff member'
  }
  
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

// Compute duration for Summary card from appointment
function getServiceDurationMinutes() {
  const start = currentAppointment.value?.startTime ? new Date(currentAppointment.value.startTime) : null
  const end = currentAppointment.value?.endTime ? new Date(currentAppointment.value.endTime) : null
  if (!start || !end) return ''
  const mins = Math.round((end.getTime() - start.getTime()) / (60 * 1000))
  return isNaN(mins) ? '' : mins
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
