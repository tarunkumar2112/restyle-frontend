<template>
  <div class="flex flex-col items-center gap-4 pt-16 bg-[var(--background)] text-[var(--text-color)] min-h-screen">
    <h1 class="font-bold text-3xl mb-2">Book Your appointment</h1>
    <p class="text-lg mb-4">
      Experience professional hair and beauty services with our expert stylists
    </p>
    <div class="w-full max-w-xl">
      <UStepper :items="steps" v-model="currentStep">
        <template #StepDepartment>
          <div>
            <form @submit.prevent="handleDepartmentSubmit">
              <div v-if="loadingGroups">
                <USkeleton class="h-16 mb-4 rounded-lg bg-gray-100" v-for="i in 6" :key="i" />
              </div>
              <div v-else>
                <div class="grid gap-4 mb-6">
                  <div
                    v-for="item in departmentRadioItems"
                    :key="item.value"
                    @click="selectedDepartment = item.value"
                    :class="[
                      'cursor-pointer p-4 border rounded-xl flex items-center justify-between transition',
                      selectedDepartment === item.value
                        ? 'bg-primary text-white border-primary'
                        : 'bg-white text-black border-gray-300 hover:border-primary'
                    ]"
                  >
                    <div class="flex items-center gap-3">
                      <UIcon :name="item.icon || 'i-lucide-user-female'" class="text-2xl" />
                      <span class="text-lg font-semibold">{{ item.label }}</span>
                    </div>
                    <div class="text-black text-sm">{{ item.description }}</div>
                  </div>
                </div>
              </div>
              <div class="flex gap-4">
                <UButton
                  type="button"
                  color="neutral"
                  class="text-lg font-bold flex-1"
                  @click="goToPreviousStep"
                  :disabled="currentStep === steps[0].value"
                >
                  Previous
                </UButton>
                <UButton
                  type="submit"
                  color="primary"
                  :disabled="!selectedDepartment || loadingGroups"
                  class="text-lg font-bold flex-1"
                >
                  Continue
                </UButton>
              </div>
            </form>
          </div>
        </template>

        <template #StepService>
          <div>
            <form @submit.prevent="handleServiceSubmit">
              <div v-if="loadingServices">
                <USkeleton class="h-16 mb-4 rounded-lg bg-gray-100" v-for="i in 3" :key="i" />
              </div>
              <div v-else>
                <div class="grid gap-4 mb-6">
                  <div
                    v-for="item in serviceRadioItems"
                    :key="item.value"
                    @click="selectedService = item.value"
                    :class="[
                      'cursor-pointer p-4 border rounded-xl flex items-center justify-between transition',
                      selectedService === item.value
                        ? 'bg-primary text-white border-primary'
                        : 'bg-white text-black border-gray-300 hover:border-primary'
                    ]"
                  >
                    <div class="flex items-center gap-3">
                      <span class="text-lg font-semibold">{{ item.label }}</span>
                    </div>
                    <div class="text-black text-sm">{{ item.description }}</div>
                  </div>
                </div>
              </div>
              <div class="flex gap-4">
                <UButton
                  type="button"
                  color="neutral"
                  class="text-lg font-bold flex-1"
                  @click="goToPreviousStep"
                >
                  Previous
                </UButton>
                <UButton
                  type="submit"
                  color="primary"
                  :disabled="!selectedService || loadingServices"
                  class="text-lg font-bold flex-1"
                >
                  Continue
                </UButton>
              </div>
            </form>
            <div v-if="selectedService" class="mt-2 text-base font-semibold">
              Selected Service ID: {{ selectedService }}
            </div>
          </div>
        </template>

        <template #StepGuests>
          <div>
            <div class="mb-4 p-4 rounded-xl border border-gray-200 bg-white flex items-center justify-between">
              <div>
                <div class="font-bold text-lg mb-1">
                  {{ selectedServiceObj?.label || 'No service selected' }}
                </div>
                <div class="text-black mb-2">
                  {{ selectedServiceObj?.description }}
                </div>
              </div>
              <div class="flex items-center gap-2">
                <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-2xl text-primary" />
                <span class="font-semibold text-black">{{ guestCount }}</span>
              </div>
            </div>
            <div class="mb-4">
              <label class="block font-semibold mb-2 text-black">Guests</label>
              <UInputNumber
                v-model="guestCount"
                :min="1"
                :max="10"
                :increment="{
                  color: 'neutral',
                  variant: 'solid',
                  size: 'xs'
                }"
                :decrement="{
                  color: 'neutral',
                  variant: 'solid',
                  size: 'xs'
                }"
                class="w-32"
              />
            </div>
            <div class="mt-2 text-base font-semibold text-black">
              You{{ guestCount > 1 ? ' and ' + (guestCount - 1) + ' guest' + (guestCount > 2 ? 's' : '') : '' }}
            </div>
            <div class="flex gap-4 mt-6">
              <UButton
                type="button"
                color="neutral"
                class="text-lg font-bold flex-1"
                @click="goToPreviousStep"
              >
                Previous
              </UButton>
              <UButton
                type="button"
                color="primary"
                :disabled="!guestCount"
                class="text-lg font-bold flex-1"
                @click="goToNextStep"
              >
                Continue
              </UButton>
            </div>
          </div>
        </template>

        <template #StepStaff>
          <div>
            <div class="flex justify-between items-center mb-4">
              <div class="w-1/2 pr-2">
                <div class="font-bold text-lg mb-1">Service</div>
                <div class="text-black">{{ selectedServiceObj?.label }}</div>
                <div class="text-sm text-primary font-semibold mt-1">
                  Duration: {{ getServiceDuration(selectedService) }} mins
                </div>
              </div>
              <div class="w-1/2 pl-2">
                <div class="font-bold text-lg mb-1">Guests</div>
                <div class="flex items-center gap-2">
                  <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-2xl text-primary" />
                  <span class="font-semibold text-black">{{ guestCount }}</span>
                </div>
              </div>
            </div>
            <form @submit.prevent="handleStaffSubmit">
              <div v-if="loadingStaff">
                <USkeleton class="h-16 mb-4 rounded-xl bg-gray-100" v-for="i in staffRadioItems.length || 3" :key="i" />
              </div>
              <div v-else>
                <div class="grid gap-4">
                  <div
                    v-for="item in staffRadioItems"
                    :key="item.value"
                    @click="selectedStaff = item.value"
                    :class="[
                      'cursor-pointer p-4 border rounded-xl flex items-center justify-between transition',
                      selectedStaff === item.value
                        ? 'bg-primary text-white border-primary'
                        : 'bg-white text-black border-gray-300 hover:border-primary'
                    ]"
                  >
                    <div class="flex items-center gap-3">
                      <UIcon :name="item.icon" class="text-2xl" />
                      <span class="text-lg font-semibold">{{ item.label }}</span>
                      <span v-if="item.badge" class="ml-2 bg-green-500 text-white px-2 py-1 rounded text-xs">{{ item.badge }}</span>
                    </div>
                    <div v-if="item.value !== 'any'" class="flex items-center gap-2">
                      <UIcon name="i-lucide-user" class="text-xl text-primary" />
                    </div>
                  </div>
                </div>
              </div>
              <div class="flex gap-4 mt-6">
                <UButton
                  type="button"
                  color="neutral"
                  class="text-lg font-bold flex-1"
                  @click="goToPreviousStep"
                >
                  Previous
                </UButton>
                <UButton
                  type="submit"
                  color="primary"
                  :disabled="!selectedStaff || loadingStaff"
                  class="text-lg font-bold flex-1"
                >
                  Continue
                </UButton>
              </div>
            </form>
          </div>
        </template>

        <template #StepDateTime>
          <div>
            <!-- Top summary boxes -->
            <div class="grid grid-cols-3 gap-4 mb-6">
              <div class="p-4 rounded-xl border border-gray-200 bg-white flex flex-col items-center">
                <div class="font-bold text-lg mb-1">Service</div>
                <div class="text-primary font-semibold text-center">
                  {{ selectedServiceObj?.label || 'No service selected' }}
                </div>
              </div>
              <div class="p-4 rounded-xl border border-gray-200 bg-white flex flex-col items-center">
                <div class="font-bold text-lg mb-1">Guests</div>
                <div class="flex items-center gap-2">
                  <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-2xl text-primary" />
                  <span class="font-semibold text-black text-xl">{{ guestCount }}</span>
                </div>
              </div>
              <div class="p-4 rounded-xl border border-gray-200 bg-white flex flex-col items-center">
                <div class="font-bold text-lg mb-1">Staff</div>
                <div class="flex items-center gap-2">
                  <UIcon name="i-lucide-user" class="text-2xl text-primary" />
                  <span class="font-semibold text-black text-center">{{ selectedStaffObj?.label || 'Any available staff' }}</span>
                </div>
              </div>
            </div>

            <!-- Calendar and slots row -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
              <!-- Calendar always visible on left -->
              <div>
                <h3 class="font-bold mb-4 text-lg text-black">Select Date</h3>
                <div class="bg-white rounded-lg border p-4">
                  <UCalendar
                    :model-value="selectedDate"
                    :min-value="new Date()"
                    :month-controls="true"
                    :year-controls="true"
                    :next-year-icon="'i-lucide-chevrons-right'"
                    :prev-year-icon="'i-lucide-chevrons-left'"
                    :next-month-icon="'i-lucide-chevron-right'"
                    :prev-month-icon="'i-lucide-chevron-left'"
                    color="primary"
                    size="md"
                    @update:model-value="handleDateSelect"
                  />
                </div>
              </div>
              
              <!-- Slots always visible on right -->
              <div class="flex flex-col">
                <h3 class="font-bold mb-4 text-lg text-black">Available Time Slots</h3>
                <div v-if="loadingSlots" class="space-y-2">
                  <USkeleton class="h-10 rounded bg-gray-100" v-for="i in 6" :key="i" />
                </div>
                <div v-else class="flex-1">
                  <div v-if="slotsForDate.length > 0" class="space-y-2 max-h-80 overflow-y-auto pr-2">
                    <UButton
                      v-for="slot in slotsForDate"
                      :key="slot"
                      color="primary"
                      variant="soft"
                      class="w-full"
                      @click="selectedSlot = slot"
                      :class="selectedSlot === slot ? 'ring-2 ring-primary font-bold !bg-primary !text-white' : ''"
                    >
                      {{ slot }}
                    </UButton>
                  </div>
                  <div v-else class="text-gray-500 text-center py-8">
                    {{ selectedDate ? 'No slots available for this date.' : 'Please select a date to see available slots.' }}
                  </div>
                </div>
                <div v-if="selectedSlot" class="mt-4 p-3 bg-primary/10 rounded-lg">
                  <div class="text-base font-semibold text-primary">
                    Selected Slot: {{ selectedSlot }}
                  </div>
                </div>
              </div>
            </div>

            <!-- Navigation buttons -->
            <div class="flex gap-4 mt-8">
              <UButton
                type="button"
                color="neutral"
                class="text-lg font-bold flex-1"
                @click="goToPreviousStep"
              >
                Previous
              </UButton>
              <UButton
                type="button"
                color="primary"
                :disabled="!selectedSlot"
                class="text-lg font-bold flex-1"
                @click="goToNextStepDateTime"
              >
                Continue
              </UButton>
            </div>
          </div>
        </template>

        <template #StepInformation>
          <div>Information step</div>
        </template>

        <template #StepSuccess>
          <div>Success step</div>
        </template>
      </UStepper>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'

const currentStep = ref('StepDepartment')
const selectedDepartment = ref('')
const departmentRadioItems = ref([])
const selectedGroup = ref('')
const groupTabs = ref([])
const loadingGroups = ref(true)

const selectedService = ref('')
const serviceRadioItems = ref([])
const loadingServices = ref(false)

const guestCount = ref(1)

const selectedServiceObj = computed(() =>
  serviceRadioItems.value.find(item => item.value === selectedService.value)
)

const selectedStaff = ref('')
const staffRadioItems = ref([])
const loadingStaff = ref(false)

const selectedStaffObj = computed(() =>
  staffRadioItems.value.find(item => item.value === selectedStaff.value)
)

const selectedDate = ref(new Date())
const slotsForDate = ref([])
const selectedSlot = ref('')
const loadingSlots = ref(false)

const steps = [
  {
    title: 'Department',
    icon: 'i-lucide-building',
    value: 'StepDepartment',
    slot: 'StepDepartment'
  },
  {
    title: 'Service',
    icon: 'i-lucide-scissors',
    value: 'StepService',
    slot: 'StepService'
  },
  {
    title: 'Guests',
    icon: 'i-lucide-users',
    value: 'StepGuests',
    slot: 'StepGuests'
  },
  {
    title: 'Staff',
    icon: 'i-lucide-user-check',
    value: 'StepStaff',
    slot: 'StepStaff'
  },
  {
    title: 'Date & Time',
    icon: 'i-lucide-calendar-days',
    value: 'StepDateTime',
    slot: 'StepDateTime'
  },
  {
    title: 'Information',
    icon: 'i-lucide-info',
    value: 'StepInformation',
    slot: 'StepInformation'
  },
  {
    title: 'Success',
    icon: 'i-lucide-check-circle',
    value: 'StepSuccess',
    slot: 'StepSuccess'
  }
]

// Map group names to icons
function getGroupIcon(name) {
  switch (name.toLowerCase()) {
    case 'ladies': return 'i-lucide-user';
    case 'gents': return 'i-lucide-user';
    case 'laser hair removal': return 'i-lucide-zap';
    case 'waxing': return 'i-lucide-droplet';
    case 'threading': return 'i-lucide-wand';
    case 'bridal': return 'i-lucide-diamond';
    case 'facials': return 'i-lucide-smile';
    default: return 'i-lucide-user'; // fallback to user icon
  }
}

// Business hours for filtering slots
function getBusinessHours(date) {
  const dayOfWeek = date.getDay() // 0 = Sunday, 1 = Monday, etc.
  
  // Thursday (4) has different hours: 11am-7pm
  if (dayOfWeek === 4) {
    return { start: 11, end: 19 } // 11am to 7pm
  }
  
  // All other days: 9am-7pm
  return { start: 9, end: 19 } // 9am to 7pm
}

// Filter slots based on business hours
function filterSlotsByBusinessHours(slots, date) {
  const businessHours = getBusinessHours(date)
  
  return slots.filter(slot => {
    // Extract hour from slot time (assuming format like "9:00 AM", "2:30 PM", etc.)
    const timeMatch = slot.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
    if (!timeMatch) return false
    
    let hour = parseInt(timeMatch[1])
    const minute = parseInt(timeMatch[2])
    const period = timeMatch[3].toUpperCase()
    
    // Convert to 24-hour format
    if (period === 'PM' && hour !== 12) {
      hour += 12
    } else if (period === 'AM' && hour === 12) {
      hour = 0
    }
    
    // Check if slot is within business hours
    const slotTime = hour + (minute / 60)
    return slotTime >= businessHours.start && slotTime < businessHours.end
  })
}

// Fetch groups for department radio
onMounted(async () => {
  try {
    const res = await fetch('https://restyle-api.netlify.app/.netlify/functions/getGroups')
    const data = await res.json()
    departmentRadioItems.value = data.groups.map(group => ({
      label: group.name,
      value: group.id,
      description: group.description || '',
      icon: getGroupIcon(group.name)
    }))
    selectedDepartment.value = ''
    groupTabs.value = data.groups.map(group => ({
      label: group.name,
      value: group.id,
      icon: getGroupIcon(group.name),
      content: group.description || ''
    }))
    selectedGroup.value = ''
  } catch (e) {
    departmentRadioItems.value = []
    groupTabs.value = []
  } finally {
    loadingGroups.value = false
  }
})

watch(selectedDepartment, async (groupId) => {
  if (!groupId) return
  loadingServices.value = true
  selectedService.value = ''
  try {
    const res = await fetch(`https://restyle-api.netlify.app/.netlify/functions/getDynamicService?id=${groupId}`)
    const data = await res.json()
    serviceRadioItems.value = (data.calendars || []).map(service => ({
      label: service.name,
      value: service.id,
      description: `Duration: ${service.slotDuration} mins | Staff: ${service.teamMembers?.length ?? 0}`
    }))
    selectedService.value = ''
  } catch (e) {
    serviceRadioItems.value = []
  } finally {
    loadingServices.value = false
  }
})

// Watch for selectedService change and fetch staff
watch(selectedService, async (serviceId) => {
  staffRadioItems.value = []
  selectedStaff.value = ''
  if (!serviceId) return
  loadingStaff.value = true
  try {
    const groupId = selectedDepartment.value
    const lastServiceApi = await fetch(`https://restyle-api.netlify.app/.netlify/functions/getDynamicService?id=${groupId}`)
    const lastServiceData = await lastServiceApi.json()
    const serviceObj = (lastServiceData.calendars || []).find(s => s.id === serviceId)
    const teamMembers = serviceObj?.teamMembers || []

    const items = [{
      label: 'Any available staff',
      value: 'any',
      badge: 'Recommended',
      icon: 'i-lucide-user'
    }]

    const staffPromises = teamMembers.map(async member => {
      try {
        const staffRes = await fetch(`https://restyle-api.netlify.app/.netlify/functions/getStaff?id=${member.userId}`)
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
    items.push(...staffResults.filter(Boolean))
    staffRadioItems.value = items
  } catch (e) {
    staffRadioItems.value = []
  } finally {
    loadingStaff.value = false
  }
})

// Previous step logic
function goToPreviousStep() {
  if (currentStep.value === 'StepService') {
    currentStep.value = 'StepDepartment'
  } else if (currentStep.value === 'StepGuests') {
    currentStep.value = 'StepService'
  } else if (currentStep.value === 'StepStaff') {
    currentStep.value = 'StepGuests'
  } else if (currentStep.value === 'StepDateTime') {
    currentStep.value = 'StepStaff'
  }
}

// Handle department submit
function handleDepartmentSubmit() {
  if (selectedDepartment.value) {
    currentStep.value = 'StepService'
  }
}

// Handle service submit
function handleServiceSubmit() {
  if (selectedService.value) {
    currentStep.value = 'StepGuests'
  }
}

function handleStaffSubmit() {
  if (selectedStaff.value) {
    currentStep.value = 'StepDateTime'
  }
}

function goToNextStep() {
  if (currentStep.value === 'StepGuests' && guestCount.value) {
    currentStep.value = 'StepStaff'
  }
}

function getServiceDuration(serviceId) {
  const service = serviceRadioItems.value.find(s => s.value === serviceId)
  return service ? service.description.match(/Duration: (\d+) mins/)?.[1] || '' : ''
}

function handleDateSelect(date) {
  selectedDate.value = date
  selectedSlot.value = '' // Reset selected slot when date changes
  
  // Only fetch slots if we have service and staff selected
  if (selectedService.value && selectedStaff.value && date) {
    fetchSlots(date)
  }
}

function fetchSlots(date) {
  if (!date || !selectedService.value || !selectedStaff.value) {
    slotsForDate.value = []
    return
  }

  selectedSlot.value = ''
  slotsForDate.value = []
  loadingSlots.value = true
  
  const calendarId = selectedService.value
  // If "any available staff" is selected, don't include userId in params
  const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value

  // MST is UTC-7 (Mountain Standard Time)
  const mstOffset = -7 * 60 * 60 * 1000
  const start = new Date(date)
  start.setHours(0, 0, 0, 0)
  const end = new Date(date)
  end.setHours(23, 59, 59, 999)

  const startDate = start.getTime() - mstOffset
  const endDate = end.getTime() - mstOffset

  // Build API URL - only include userId if it's not empty
  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/getAllstaffslot?calendarId=${calendarId}&startDate=${startDate}&endDate=${endDate}`
  if (userId) {
    apiUrl += `&userId=${userId}`
  }

  fetch(apiUrl)
    .then(res => res.json())
    .then(data => {
      console.log('Slots API response:', data)
      const formatted = data.formattedSlots || {}
      const key = Object.keys(formatted)[0]
      const allSlots = key ? formatted[key] : []
      
      // Filter slots by business hours
      const filteredSlots = filterSlotsByBusinessHours(allSlots, date)
      slotsForDate.value = filteredSlots
      
      console.log('Available slots (filtered by business hours):', slotsForDate.value)
    })
    .catch((error) => {
      console.error('Error fetching slots:', error)
      slotsForDate.value = []
    })
    .finally(() => {
      loadingSlots.value = false
    })
}

function goToNextStepDateTime() {
  if (selectedSlot.value) {
    currentStep.value = 'StepInformation'
  }
}
</script>