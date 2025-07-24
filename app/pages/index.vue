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
              <URadioGroup
                v-else
                color="primary"
                variant="card"
                :items="departmentRadioItems"
                v-model="selectedDepartment"
                class="mb-6"
                :ui="{
                  wrapper: 'gap-4',
                  card: ({ checked }) => checked
                    ? 'bg-primary text-white border-primary'
                    : 'bg-white text-black border-gray-300',
                  label: 'text-lg font-semibold text-black',
                  description: 'text-black'
                }"
              />
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
              <URadioGroup
                v-else
                color="primary"
                variant="card"
                :items="serviceRadioItems"
                v-model="selectedService"
                class="mb-6"
                :ui="{
                  wrapper: 'gap-4',
                  card: ({ checked }) => checked
                    ? 'bg-primary text-white border-primary'
                    : 'bg-white text-black border-gray-300',
                  label: 'text-lg font-semibold text-black',
                  description: 'text-black'
                }"
              />
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
          <div>Guests step</div>
        </template>
        <template #StepStaff>
          <div>Staff selection step</div>
        </template>
        <template #StepDateTime>
          <div>Date & Time step</div>
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
import { ref, onMounted, watch } from 'vue'

const currentStep = ref('StepDepartment')
const selectedDepartment = ref('')
const departmentRadioItems = ref([])

const selectedGroup = ref('')
const groupTabs = ref([])
const loadingGroups = ref(true)

const selectedService = ref('')
const serviceRadioItems = ref([])
const loadingServices = ref(false)

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
    case 'ladies': return 'i-lucide-user-female';
    case 'gents': return 'i-lucide-user';
    case 'laser hair removal': return 'i-lucide-zap';
    case 'waxing': return 'i-lucide-droplet';
    case 'threading': return 'i-lucide-wand';
    case 'bridal': return 'i-lucide-diamond';
    case 'facials': return 'i-lucide-smile';
    default: return 'i-lucide-folder';
  }
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
    // Do not select any by default
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
    // Do not select any by default
    selectedService.value = ''
  } catch (e) {
    serviceRadioItems.value = []
  } finally {
    loadingServices.value = false
  }
})

// Previous step logic
function goToPreviousStep() {
  if (currentStep.value === 'StepService') {
    currentStep.value = 'StepDepartment'
  } else if (currentStep.value === 'StepGuests') {
    currentStep.value = 'StepService'
  }
  // Add more as needed for other steps
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
</script>

