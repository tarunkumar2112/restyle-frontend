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
        <div class="sm:p-8 department-inner">
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
                  
                  <div class="flex-1 grid grid-cols-1 sm:grid-cols-3 gap-4 sm:mx-4">
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
                    :disabled="currentDateIndex >= availableDates.length - (typeof window !== 'undefined' && window.innerWidth < 640 ? 1 : 3)"
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
                                         
                    <div class="grid grid-cols-2 gap-3 sm:grid-cols-4 md:grid-cols-4 max-h-80 overflow-y-auto pr-2">
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
              <div v-if="hasChanges" class="p-4 bg-red-50 rounded-lg border border-red-200">
                <h4 class=" text-[18px] font-bold text-black mb-2 sm:text-[20px]">Summary of Changes:</h4>
                <ul class="text-sm text-blue-700 space-y-1">
                  <li class="text-black text-sm font-inter" v-if="selectedStaff && selectedStaff !== currentAppointment.assignedUserId">
                    • Stylist changed to {{ getSelectedStaffName() }}
                  </li>
                  <li class="text-black text-sm font-inter" v-if="selectedDateString !== formatAppointmentDateString(currentAppointment.startTime)">
                    • Date changed to {{ formatDateForDisplay(selectedDateString) }}
                  </li>
                  <li class="text-black text-sm font-inter" v-if="selectedSlot !== formatAppointmentTime(currentAppointment.startTime)">
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
                <h2 class="text-lg font-bold text-black sm:text-3xl">Appointment Updated Successfully!</h2>
                <p class="text-lg text-gray-700">Your appointment has been updated with the new details.</p>
              </div>

              <div class="p-6 bg-green-50 rounded-xl border border-green-200 sm:w-[500px] mx-auto">
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
  // Show only 1 date on mobile, 3 on larger screens
  const isMobile = typeof window !== 'undefined' && window.innerWidth < 640
  const dateCount = isMobile ? 1 : 3
  return availableDates.value.slice(currentDateIndex.value, currentDateIndex.value + dateCount)
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

// Cache is no longer needed as we use prefetched working slots

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
      
      // By default use the current staff ID from the appointment
      selectedStaff.value = appointmentData.assignedUserId || 'any'
      
      // Set current date and time
      const appointmentDate = new Date(appointmentData.startTime)
      const dateString = appointmentDate.toISOString().split('T')[0]
      selectedDateString.value = dateString
      
      const [year, month, day] = dateString.split('-').map(Number)
      selectedCalendarDate.value = new CalendarDate(year, month, day)
      
      selectedSlot.value = formatAppointmentTime(appointmentData.startTime)
      // calendarId.value = appointmentData.calendarId // This line was removed as per new_code
      
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
  
  // By default use the current staff ID from the appointment, or selected staff if changed
  const userId = selectedStaff.value && selectedStaff.value !== 'any' 
    ? selectedStaff.value 
    : currentAppointment.value.assignedUserId || null
  
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

  // Build API URL with userId parameter for filtered slots
  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/staffSlots?calendarId=${serviceId}${startDateParam}`
  if (userId) {
    apiUrl += `&userId=${userId}`
  }

  console.log('Fetching working slots for service:', serviceId, 'and staff:', userId)

  try {
    const response = await fetch(apiUrl)
    const data = await response.json()
    if (data && data.slots) {
      workingSlots.value = data.slots
      workingSlotsLoaded.value = true
      
      // Generate available dates from working slots
      generateAvailableDates()
      
      console.log('Working slots loaded successfully:', data.slots)
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
  
  // Get today's date string for comparison (YYYY-MM-DD format)
  const today = new Date()
  const todayDateString = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`
  
  const currentBookingDate = currentAppointment.value.startTime ? new Date(currentAppointment.value.startTime) : null
  
  // If we have working slots, use them directly from API
  if (workingSlotsLoaded.value && Object.keys(workingSlots.value).length > 0) {
    const workingDates = Object.keys(workingSlots.value)
      .filter(dateString => {
        // Only include dates from today onwards (no past dates)
        return dateString >= todayDateString
      })
      .sort()
    
    workingDates.forEach((dateString, index) => {
      // Parse the date string from API to get day name and display
      const [year, month, day] = dateString.split('-').map(Number)
      const date = new Date(year, month - 1, day) // month is 0-indexed in JS Date
      
      // Get day name from the actual date (not calculated)
      const dayName = date.toLocaleDateString('en-US', { weekday: 'long' })
      const dateDisplay = date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
      
      // Determine label based on actual date difference from today
      let label = ''
      if (dateString === todayDateString) {
        label = 'TODAY'
      } else if (dateString === getTomorrowDateString()) {
        label = 'TOMORROW'
      } else if (isThisWeek(dateString)) {
        label = 'THIS WEEK'
      } else {
        label = 'NEXT WEEK'
      }
      
      dates.push({
        dateString,
        dayName,
        dateDisplay,
        label,
        date
      })
    })
  } else {
    // Fallback to generating next 7 working days if no working slots
    let workingDaysCount = 0
    let i = 0
    
    while (workingDaysCount < 7 && i < 14) {
      const date = new Date(today)
      date.setDate(today.getDate() + i)

      // Skip weekends
      const dayOfWeek = date.getDay()
      if (dayOfWeek === 0 || dayOfWeek === 6) {
        i++
        continue
      }
      
      const dateString = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
      const dayName = date.toLocaleDateString('en-US', { weekday: 'long' })
      const dateDisplay = date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
      
      let label = ''
      if (workingDaysCount === 0) label = 'TODAY'
      else if (workingDaysCount === 1) label = 'TOMORROW'
      else if (workingDaysCount <= 5) label = 'THIS WEEK'
      else label = 'NEXT WEEK'
      
      dates.push({
        dateString,
        dayName,
        dateDisplay,
        label,
        date
      })
      
      workingDaysCount++
      i++
    }
  }
  
  availableDates.value = dates
  
  if (dates.length > 0 && currentAppointment.value.calendarId && selectedStaff.value) {
    const selectedDate = dates[0]
    selectedDateString.value = selectedDate.dateString
    const [year, month, day] = selectedDate.dateString.split('-').map(Number)
    selectedCalendarDate.value = new CalendarDate(year, month, day)
    await fetchSlotsForDate(selectedDate.dateString)
  }
}

// Enhanced fetchSlotsForDate with better caching and retry logic
async function fetchSlotsForDate(dateString) {
  if (!currentAppointment.value.calendarId || !dateString) {
    console.log('Missing required data for date-specific slot fetch')
    slotsForDate.value = []
    return
  }

  console.log('Fetching slots for specific date:', dateString)
  
  selectedSlot.value = ''
  loadingSlots.value = true

  // Use working slots data if available (prefetched from single endpoint)
  if (workingSlots.value[dateString]) {
    const slotsForSelectedDate = workingSlots.value[dateString]
    const slotsWithStatus = slotsForSelectedDate.map(slot => ({
      time: slot,
      isPast: isSlotInPastMST(slot, dateString),
      isUnavailable: false
    }))
    
    // Filter out past slots and only show future/current slots
    const availableSlots = slotsWithStatus.filter(slot => !slot.isPast)
    
    slotsForDate.value = availableSlots
    loadingSlots.value = false
    console.log('Using working slots for date:', dateString, availableSlots)
    return
  }

  // If no weekly slots for this date, show empty
  slotsForDate.value = []
  loadingSlots.value = false
  console.log('No weekly slots available for date:', dateString)
}



// These functions are no longer needed as we use prefetched working slots

// This function is no longer needed as we use prefetched working slots

async function selectStaff(staffId) {
  selectedStaff.value = staffId
  // Keep spinner visible while refreshing slots for new staff
  loadingSlots.value = true
  slotsForDate.value = []
  
  // Refresh slots when staff changes
  if (selectedDateString.value) {
    await fetchSlotsForDate(selectedDateString.value)
  }
  
  // Auto-select first available date after staff selection
  if (availableDates.value.length > 0 && currentAppointment.value.calendarId) {
    // Reset slider to first visible date for update flow
    currentDateIndex.value = 0
    const firstAvailableDate = availableDates.value[0]
    
    selectedDateString.value = firstAvailableDate.dateString
    
    const [year, month, day] = firstAvailableDate.dateString.split('-').map(Number)
    selectedCalendarDate.value = new CalendarDate(year, month, day)
    
    // Fetch slots for the first available date
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
    // Always determine an assignedUserId to satisfy API requirements
    let assignedUserIdToSend = null
    if (selectedStaff.value && selectedStaff.value !== 'any') {
      assignedUserIdToSend = selectedStaff.value
    } else if (selectedStaff.value === 'any') {
      const realStaff = staffRadioItems.value.filter(item => item.value !== 'any')
      if (realStaff.length > 0) {
        assignedUserIdToSend = realStaff[0].value
      }
    }
    if (!assignedUserIdToSend && currentAppointment.value.assignedUserId) {
      assignedUserIdToSend = currentAppointment.value.assignedUserId
    }

    if (!assignedUserIdToSend) {
      throw new Error('A team member needs to be selected. assignedUserId is missing')
    }

    // Build update URL. Always include assignedUserId
    let updateUrl = `https://restyle-api.netlify.app/.netlify/functions/updateappointment?appointmentId=${appointmentId.value}&assignedUserId=${assignedUserIdToSend}`

    // Add time parameters if changed
    if (
      selectedSlot.value !== formatAppointmentTime(currentAppointment.value.startTime) ||
      selectedDateString.value !== formatAppointmentDateString(currentAppointment.value.startTime)
    ) {
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
      throw new Error(data.error || data.details?.message || 'Update failed')
    }
    
  } catch (error) {
    console.error('Update error:', error)
    alert('Failed to update appointment. Please try again.')
  } finally {
    updateLoading.value = false
  }
}

// These functions are no longer needed as we use prefetched working slots

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

// Watch for staff changes and refetch working slots with new user ID
watch(selectedStaff, async (newStaff, oldStaff) => {
  if (newStaff !== oldStaff && currentAppointment.value.calendarId) {
    console.log('Staff changed from', oldStaff, 'to', newStaff, '- refetching working slots')
    // Clear current slots and refetch with new staff
    workingSlots.value = {}
    workingSlotsLoaded.value = false
    slotsForDate.value = []
    selectedSlot.value = ''
    
    // Refetch working slots with new staff ID
    await fetchWorkingSlots()
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
  const newIndex = currentDateIndex.value + direction
  // Calculate max index based on screen size (mobile shows 1, desktop shows 3)
  const isMobile = typeof window !== 'undefined' && window.innerWidth < 640
  const dateCount = isMobile ? 1 : 3
  const maxIndex = availableDates.value.length - dateCount
  
  if (newIndex >= 0 && newIndex <= maxIndex) {
    currentDateIndex.value = newIndex
  }
}

// Helper functions for date generation
function getTomorrowDateString() {
  const tomorrow = new Date()
  tomorrow.setDate(tomorrow.getDate() + 1)
  return `${tomorrow.getFullYear()}-${String(tomorrow.getMonth() + 1).padStart(2, '0')}-${String(tomorrow.getDate()).padStart(2, '0')}`
}

function isThisWeek(dateString) {
  const today = new Date()
  const endOfWeek = new Date(today)
  endOfWeek.setDate(today.getDate() + (6 - today.getDay()))
  
  const targetDate = new Date(dateString)
  return targetDate <= endOfWeek
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
    grid-template-columns: 1fr 1fr;
  }
}
</style>
