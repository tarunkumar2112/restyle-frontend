<template>
  <div class="min-h-screen bg-white book-main">
    <div class="flex flex-col items-center gap-6  pb-16 px-4">
  <!--     
      <div class="text-center mb-2 book-heading-top">
        <h1 class="font-bold text-4xl mb-3 text-black">
          Book Your Appointment
        </h1>
        <p class="text-lg text-gray-700 max-w-2xl mx-auto leading-relaxed">
          Experience professional hair and beauty services with our expert stylists in a luxurious environment
        </p>
      </div>
  -->
      
      <div class="w-full max-w-4xl overflow-hidden department-block">
        <UStepper disabled :items="steps" v-model="currentStep" class="p-8 department-inner">
          <template #StepDepartment>
            <div class="space-y-6 department-inner-heading">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Choose Your Department</h2>
                <p class="text-gray-700">Select the service category that best fits your needs</p>
              </div>
              
              <form @submit.prevent="handleDepartmentSubmit">
                <div v-if="loadingGroups" class="space-y-4">
                  <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in 6" :key="i" />
                </div>
                <div v-else class="grid gap-4 mb-8 same-block-content">
                  <div
                    v-for="item in departmentRadioItems"
                    :key="item.value"
                    @click="selectDepartment(item.value)"
                    :class="[
                      'cursor-pointer p-6 border-1 rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm',
                      selectedDepartment === item.value
                        ? 'bg-red-50 text-black border-red-700 shadow-sm'
                        : 'bg-white text-black border-gray-200 hover:border-red-300'
                    ]"
                  >
                    <div class="flex items-center gap-4">
                      <div :class="[
                        'p-3 rounded-full',
                        selectedDepartment === item.value ? 'bg-red-100' : 'bg-gray-100'
                      ]">
                        <UIcon :name="item.icon || 'i-lucide-user-female'" :class="[
                          'text-2xl',
                          selectedDepartment === item.value ? 'text-red-700' : 'text-gray-600'
                        ]" />
                      </div>
                      <div>
                        <span class="text-xl font-semibold block text-black">{{ item.label }}</span>
                        <span class="text-sm text-gray-600">{{ item.description }}</span>
                      </div>
                    </div>
                    <div v-if="selectedDepartment === item.value" class="text-red-700">
                      <UIcon name="i-lucide-check-circle" class="text-2xl" />
                    </div>
                  </div>
                </div>
                
                <div class="flex gap-4 same-btn-prev-next">
                  <UButton
                    type="button"
                    color="gray"
                    variant="soft"
                    size="lg"
                    class="flex-1"
                    @click="goToPreviousStep"
                    :disabled="currentStep === steps[0].value"
                  >
                    <UIcon name="i-lucide-arrow-left" class="mr-2" />
                    Previous
                  </UButton>
                  <UButton
                    type="submit"
                    color="primary"
                    size="lg"
                    :disabled="!selectedDepartment || loadingGroups"
                    class="flex-1 bg-red-700 hover:bg-red-700 text-white"
                  >
                    Continue
                    <UIcon name="i-lucide-arrow-right" class="ml-2" />
                  </UButton>
                </div>
              </form>
            </div>
          </template>

          <template #StepService>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Select Your Service</h2>
                <p class="text-gray-700">Choose the specific service you'd like to book</p>
              </div>
              
              <form @submit.prevent="handleServiceSubmit">
                <div v-if="loadingServices" class="space-y-4">
                  <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in 3" :key="i" />
                </div>
                <div v-else class="grid gap-4 mb-8 same-block-content">
                  <div
                    v-for="item in serviceRadioItems"
                    :key="item.value"
                    @click="selectService(item.value)"
                    :class="[
                      'cursor-pointer p-6 border-1 rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm',
                      selectedService === item.value
                        ? 'bg-red-50 text-black border-red-700 shadow-sm'
                        : 'bg-white text-black border-gray-200 hover:border-red-300'
                    ]"
                  >
                    <div class="flex items-center gap-4">
                      <div :class="[
                        'p-3 rounded-full',
                        selectedService === item.value ? 'bg-red-100' : 'bg-gray-100'
                      ]">
                        <UIcon name="i-lucide-scissors" :class="[
                          'text-2xl',
                          selectedService === item.value ? 'text-red-700' : 'text-gray-600'
                        ]" />
                      </div>
                      <div>
                        <span class="text-xl font-semibold block text-black">{{ item.label }}</span>
                        <span class="text-sm text-gray-600">{{ item.description }}</span>
                      </div>
                    </div>
                    <div v-if="selectedService === item.value" class="text-red-700">
                      <UIcon name="i-lucide-check-circle" class="text-2xl" />
                    </div>
                  </div>
                </div>
                
                <div class="flex gap-4 same-btn-prev-next">
                  <UButton
                    type="button"
                    color="gray"
                    variant="soft"
                    size="lg"
                    class="flex-1"
                    @click="goToPreviousStep"
                  >
                    <UIcon name="i-lucide-arrow-left" class="mr-2" />
                    Previous
                  </UButton>
                  <UButton
                    type="submit"
                    color="primary"
                    size="lg"
                    :disabled="!selectedService || loadingServices"
                    class="flex-1 bg-red-700 hover:bg-red-700 text-white"
                  >
                    Continue
                    <UIcon name="i-lucide-arrow-right" class="ml-2" />
                  </UButton>
                </div>
              </form>
            </div>
          </template>

          <template #StepStaff>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Choose Your Stylist</h2>
                <p class="text-gray-700">Select your preferred stylist or let us choose the best available</p>
              </div>

              <!-- Summary cards: Service | Guests | Staff -->
              <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8 max-w-3xl mx-auto">
                <!-- Service Card -->
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200">
                  <div class="font-bold text-lg mb-1 text-black">Service</div>
                  <div class="text-gray-700">{{ selectedServiceObj?.label }}</div>
                  <div class="text-sm text-red-700 font-semibold mt-1">
                    Duration: {{ getServiceDuration(selectedService) }} mins
                  </div>
                </div>
                <!-- Guests Card with Add Guest logic -->
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200 flex flex-col items-center justify-center">
                  <div class="font-bold text-lg mb-1 text-black">Guests</div>
                  <div v-if="!showGuestInput" class="flex flex-col items-center">
                    <div class="flex items-center justify-center gap-2 mb-2">
                      <UIcon name="i-lucide-user" class="text-2xl text-red-700" />
                      <span class="font-bold text-xl text-black">1</span>
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
                <!-- Staff Card -->
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200 flex flex-col items-center justify-center">
                  <div class="font-bold text-lg mb-1 text-black">Stylist</div>
                  <div class="text-red-700 font-semibold text-center">{{ selectedStaffObj?.label || 'Any available' }}</div>
                </div>
              </div>

              <form @submit.prevent="handleStaffSubmit">
                <div v-if="loadingStaff" class="space-y-4">
                  <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in staffRadioItems.length || 3" :key="i" />
                </div>
                <div v-else class="grid gap-4 mb-8 same-block-content">
                  <div
                    v-for="item in staffRadioItems"
                    :key="item.value"
                    @click="selectStaff(item.value)"
                    :class="[
                      'cursor-pointer p-6 border-1 rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm',
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
                        <span class="text-xl font-semibold text-black">{{ item.label }}</span>
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
                
                <div class="flex gap-4 same-btn-prev-next">
                  <UButton
                    type="button"
                    color="gray"
                    variant="soft"
                    size="lg"
                    class="flex-1"
                    @click="goToPreviousStep"
                  >
                    <UIcon name="i-lucide-arrow-left" class="mr-2" />
                    Previous
                  </UButton>
                  <UButton
                    type="submit"
                    color="primary"
                    size="lg"
                    :disabled="!selectedStaff || loadingStaff"
                    class="flex-1 bg-red-700 hover:bg-red-700 text-white"
                  >
                    Continue
                    <UIcon name="i-lucide-arrow-right" class="ml-2" />
                  </UButton>
                </div>
              </form>
            </div>
          </template>
          
          <template #StepDateTime>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Select Date & Time</h2>
                <p class="text-gray-700">Choose your preferred appointment slot</p>
              </div>

              <!-- Added MST timezone indicator -->
              <div class="text-center mb-6">
                <div class="text-sm font-medium text-gray-600 uppercase tracking-wide">
                  TIME ZONE: MOUNTAIN TIME - EDMONTON (GMT-06:00)
                </div>
              </div>
              
              <!-- Updated date slider to show 3 dates and make them clickable -->
              <div class="space-y-6">
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
                    
                    <!-- Changed from grid-cols-2 to grid-cols-3 and added click handlers -->
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
                
                <!-- Updated time slots section to show only enabled slots -->
                <div class="space-y-4 times-block">
                  <div class="bg-white rounded-xl border border-gray-200 p-6 shadow-sm min-h-[400px]">
                    <div v-if="loadingSlots" class="space-y-3">
                      <div class="grid grid-cols-2 gap-3">
                        <USkeleton class="h-12 rounded-lg bg-gray-100" v-for="i in 8" :key="i" />
                      </div>
                    </div>
                    <!-- Enhanced slot display with better error handling -->
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
                      <div class="font-bold text-black text-lg">Selected Time</div>
                      <div class="text-red-700 font-semibold">{{ selectedSlot }} MST</div>
                      <div class="text-sm text-gray-600 mt-1">{{ formatDateForDisplay(selectedDateString) }}</div>
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- Navigation -->
              <div class="flex gap-4 pt-6 same-btn-prev-next">
                <UButton
                  type="button"
                  color="gray"
                  variant="soft"
                  size="lg"
                  class="flex-1"
                  @click="goToPreviousStep"
                >
                  <UIcon name="i-lucide-arrow-left" class="mr-2" />
                  Previous
                </UButton>
                <UButton
                  type="button"
                  color="primary"
                  size="lg"
                  :disabled="!selectedSlot"
                  class="flex-1 bg-red-700 hover:bg-red-700 text-white"
                  @click="goToNextStepDateTime"
                >
                  Continue
                  <UIcon name="i-lucide-arrow-right" class="ml-2" />
                </UButton>
              </div>
            </div>
          </template>

          <template #StepInformation>
            <!-- Added back arrow to booking step -->
            <div class="space-y-6">
              <div class="flex items-center mb-6">
                <UButton
                  variant="ghost"
                  size="sm"
                  @click="goToPreviousStep"
                  class="mr-4"
                >
                  <UIcon name="i-lucide-arrow-left" class="text-xl" />
                </UButton>
                <div class="text-center flex-1">
                  <h2 class="text-2xl font-bold text-black mb-2">Contact Information</h2>
                  <p class="text-gray-700">Please provide your details to complete the booking</p>
                </div>
              </div>
              
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <!-- Contact Form -->
                <div class="space-y-6">
                  <form class="space-y-6 information-depaerment-form" @submit.prevent="handleInformationSubmit">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div>
                        <UInput
                          v-model="contactForm.firstName"
                          label="First Name"
                          placeholder="Enter your first name"
                          size="lg"
                          :error="validationErrors.firstName"
                          required
                        />
                      </div>
                      <div>
                        <UInput
                          v-model="contactForm.lastName"
                          label="Last Name"
                          placeholder="Enter your last name"
                          size="lg"
                          :error="validationErrors.lastName"
                          required
                        />
                      </div>
                      <div>
                        <UInput
                          v-model="contactForm.phone"
                          label="Phone Number"
                          placeholder="(555) 123-4567"
                          size="lg"
                          :error="validationErrors.phone"
                          required
                        />
                      </div>
                      <div>
                        <UInput
                          v-model="contactForm.email"
                          label="Email Address"
                          placeholder="your@email.com"
                          type="email"
                          size="lg"
                          :error="validationErrors.email"
                          required
                        />
                      </div>
                    </div>
                    
                    <UTextarea
                      v-model="contactForm.notes"
                      label="Additional Notes (Optional)"
                      placeholder="Any special requests or information we should know?"
                      size="lg"
                      rows="3"
                    />
                    
                    <UCheckbox
                      v-model="contactForm.optIn"
                      label="I would like to receive updates and promotional offers via email"
                      size="lg"
                    />
                    
                    <UButton
                      type="submit"
                      color="primary"
                      size="xl"
                      :loading="bookingLoading"
                      :disabled="!isFormValid"
                      class="w-full font-bold bg-red-700 hover:bg-red-700 text-white justify-center"
                    >
                      <UIcon name="i-lucide-calendar-check" class="mr-2" />
                      {{ bookingLoading ? 'Booking Your Appointment...' : 'Book Appointment' }}
                    </UButton>
                  </form>
                </div>
                
                <!-- Appointment Summary -->
                <div class="space-y-4">
                  <h3 class="font-bold text-xl text-black mb-4">Appointment Summary</h3>
                  
                  <div class="space-y-4 appointment-summary-mobile">
                    <div class="p-6 rounded-xl border border-gray-200 bg-white appointment-summary-mobile-block">
                      <div class="space-y-3">
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-calendar" class="text-xl text-red-700" />
                          <span class="font-semibold text-black">
                            {{ formatCalendarDate(selectedCalendarDate) }}
                          </span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-clock" class="text-xl text-red-700" />
                          <span class="font-semibold text-black">{{ selectedSlot }} MST</span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-hourglass" class="text-xl text-red-700" />
                          <span class="text-gray-700">{{ getServiceDuration(selectedService) }} minutes</span>
                        </div>
                       
                      </div>
                    </div>
                    
                    <div class="p-6 rounded-xl border border-gray-200 bg-white appointment-summary-mobile-block">
                      <div class="space-y-3">
                        <div class="font-bold text-lg text-black">{{ selectedServiceObj?.label }}</div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-user" class="text-xl text-red-700" />
                          <span class="text-gray-700">with {{ selectedStaffObj?.label }}</span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-xl text-red-700" />
                          <span class="text-gray-700">{{ guestCount }} {{ guestCount === 1 ? 'person' : 'people' }}</span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </template>
          <template #StepSuccess>
            <div class="flex flex-col items-center justify-center min-h-[400px] text-center space-y-6 same-btn-prev-next success-last pt-4">
              <div class="relative success-last-mobile">
                <div class="w-24 h-24 h-[60px] w-[60px] bg-red-700 rounded-full flex items-center justify-center shadow-sm img-icon">
                  <UIcon name="i-lucide-check-circle" class="text-2xl text-white" />
                </div>
                <div class="absolute -top-2 -right-2 w-8 h-8 bg-red-100 rounded-full flex items-center justify-center border border-red-200">
                  <UIcon name="i-lucide-sparkles" class="text-lg text-red-700" />
                </div>
              </div>
              
              <div class="space-y-4">
                <h2 class="font-bold text-3xl text-black booking-mobile">Booking Confirmed! 🎉</h2>
                <p class="text-lg text-gray-700 max-w-md">
                  Thank you for choosing us! Your appointment has been successfully booked. 
                  We're excited to see you soon!
                </p>
              </div>
              
              <div class="p-6 bg-red-50 rounded-xl border border-red-200 max-w-md">
                <div class="space-y-2">
                  <div class="font-semibold text-black text-[20px]">What's Next?</div>
                  <div class="text-sm text-gray-700 space-y-1">
                    <p class="text-[16px] text-black font-medium">• You'll receive a confirmation email shortly</p>
                   <p class="text-[16px] text-black font-medium"> • We'll send you a reminder 24 hours before</p>
                   <p class="text-[16px] text-black font-medium"> • Feel free to call us if you need to reschedule</p>
                  </div>
                </div>
              </div>
              
              <UButton
                color="primary"
                size="lg"
                @click="resetBooking"
                class="mt-6 bg-red-700 hover:bg-red-700 text-white"
              >
                <UIcon name="i-lucide-plus" class="mr-2" />
                Book Another Appointment
              </UButton>
            </div>
          </template>
        </UStepper>
      </div>
    </div>

    <!-- Removed toast container completely -->
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { CalendarDate, today, getLocalTimeZone } from '@internationalized/date'
import { useRoute } from 'vue-router'

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
const showGuestInput = ref(false)

function resetGuestInput() {
  guestCount.value = 1
  showGuestInput.value = false
}

const selectedServiceObj = computed(() =>
  serviceRadioItems.value.find(item => item.value === selectedService.value)
)

const selectedStaff = ref('')
const staffRadioItems = ref([])
const loadingStaff = ref(false)

const selectedStaffObj = computed(() =>
  staffRadioItems.value.find(item => item.value === selectedStaff.value)
)

// Calendar and slots
const selectedCalendarDate = ref(null)
const slotsForDate = ref([])
const selectedSlot = ref('')
const loadingSlots = ref(false)

const activeSlots = ref({})
const activeDay = ref('')
const calendarId = ref('')

// Form validation
const validationErrors = ref({
  firstName: '',
  lastName: '',
  phone: '',
  email: ''
})

const contactForm = ref({
  firstName: '',
  lastName: '',
  phone: '',
  email: '',
  notes: '',
  optIn: false
})

const isFormValid = computed(() => {
  return contactForm.value.firstName.trim() && 
         contactForm.value.lastName.trim() && 
         contactForm.value.phone.trim() && 
         contactForm.value.email.trim() && 
         isValidEmail(contactForm.value.email) && 
         isValidPhone(contactForm.value.phone)
})

function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}

function isValidPhone(phone) {
  const phoneRegex = /^[\+]?[1-9][\d]{0,15}$/
  const cleanPhone = phone.replace(/[\s\-()]/g, '')
  return cleanPhone.length >= 10 && phoneRegex.test(cleanPhone)
}

function validateForm() {
  validationErrors.value = {
    firstName: '',
    lastName: '',
    phone: '',
    email: ''
  }

  if (!contactForm.value.firstName.trim()) {
    validationErrors.value.firstName = 'First name is required'
  }
  if (!contactForm.value.lastName.trim()) {
    validationErrors.value.lastName = 'Last name is required'
  }
  if (!contactForm.value.phone.trim()) {
    validationErrors.value.phone = 'Phone number is required'
  } else if (!isValidPhone(contactForm.value.phone)) {
    validationErrors.value.phone = 'Please enter a valid phone number'
  }
  if (!contactForm.value.email.trim()) {
    validationErrors.value.email = 'Email is required'
  } else if (!isValidEmail(contactForm.value.email)) {
    validationErrors.value.email = 'Please enter a valid email address'
  }

  return Object.values(validationErrors.value).every(error => !error)
}

const bookingLoading = ref(false)
const bookingResponse = ref(null)

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
    default: return 'i-lucide-user';
  }
}

// Convert CalendarDate to JavaScript Date for API calls
function calendarDateToJSDate(calendarDate) {
  if (!calendarDate) return null
  return new Date(calendarDate.year, calendarDate.month - 1, calendarDate.day)
}

// Format CalendarDate for display
function formatCalendarDate(calendarDate) {
  if (!calendarDate) return ''
  const jsDate = calendarDateToJSDate(calendarDate)
  return jsDate.toLocaleDateString('en-US', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

// Check if a time slot is in the past
function isSlotInPast(slotTime, selectedDate) {
  if (!selectedDate || !slotTime) return false
  
  const now = new Date()
  const slotDate = calendarDateToJSDate(selectedDate)
  
  // Parse slot time (e.g., "2:30 PM")
  const timeMatch = slotTime.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
  if (!timeMatch) return false
  
  let hour = parseInt(timeMatch[1])
  const minute = parseInt(timeMatch[2])
  const period = timeMatch[3].toUpperCase()
  
  if (period === 'PM' && hour !== 12) hour += 12
  if (period === 'AM' && hour === 12) hour = 0
  
  slotDate.setHours(hour, minute, 0, 0)
  
  // Convert to MST for comparison
  const mstOffset = -7 * 60 * 60 * 1000 // MST is UTC-7
  const slotMST = new Date(slotDate.getTime() + mstOffset)
  const nowMST = new Date(now.getTime() + mstOffset)
  
  return slotMST < nowMST
}

// Business hours for filtering slots
function getBusinessHours(date) {
  const dayOfWeek = date.getDay()
  
  if (dayOfWeek === 4) { // Thursday
    return { start: 11, end: 19 } // 11am to 7pm
  }
  
  return { start: 9, end: 19 } // 9am to 7pm for other days
}

// Filter slots based on business hours
function filterSlotsByBusinessHours(slots, date) {
  const businessHours = getBusinessHours(date)
  
  return slots.filter(slot => {
    const timeMatch = slot.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
    if (!timeMatch) return false
    
    let hour = parseInt(timeMatch[1])
    const minute = parseInt(timeMatch[2])
    const period = timeMatch[3].toUpperCase()
    
    if (period === 'PM' && hour !== 12) {
      hour += 12
    } else if (period === 'AM' && hour === 12) {
      hour = 0
    }
    
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

// Calendar and slots
// const selectedCalendarDate = ref(null)
// const slotsForDate = ref([])
// const selectedSlot = ref('')
// const loadingSlots = ref(false)

// const activeSlots = ref({})
// const activeDay = ref('')
// const calendarId = ref('')

async function fetchActiveSlots() {
  if (!selectedService.value) {
    console.log('No service selected for active slots fetch')
    slotsForDate.value = []
    return
  }

  console.log('Fetching active slots for service:', selectedService.value)
  
  selectedSlot.value = ''
  slotsForDate.value = []
  loadingSlots.value = true

  const serviceId = selectedService.value
  const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value

  // Build API URL for active slots
  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/getactiveslot?calendarId=${serviceId}`
  if (userId) {
    apiUrl += `&userId=${userId}`
  }

  console.log('Active Slots API URL:', apiUrl)

  try {
    const response = await fetch(apiUrl)
    const data = await response.json()
    console.log('Active Slots API response:', data)

    if (data.slots && data.activeDay && data.calendarId) {
      activeSlots.value = data.slots
      activeDay.value = data.activeDay
      calendarId.value = data.calendarId

      // Get the first available date from slots
      const availableDates = Object.keys(data.slots)
      if (availableDates.length > 0) {
        const firstAvailableDate = availableDates[0]
        
        const [year, month, day] = firstAvailableDate.split('-').map(Number)
        selectedCalendarDate.value = new CalendarDate(year, month, day)
        
        const slotsForSelectedDate = data.slots[firstAvailableDate] || []
        const slotsWithStatus = slotsForSelectedDate.map(slot => ({
          time: slot,
          isPast: isSlotInPastMST(slot, firstAvailableDate)
        }))

        slotsForDate.value = slotsWithStatus
        console.log('Active slots loaded for date:', firstAvailableDate, slotsWithStatus)
        
        if (slotsWithStatus.length === 0) {
        }
      } else {
      }
    } else {
      console.error('Invalid response format:', data)
    }
  } catch (error) {
    console.error('Error fetching active slots:', error)
    slotsForDate.value = []
  } finally {
    loadingSlots.value = false
  }
}

async function fetchSlotsForDate(dateString) {
  if (!selectedService.value || !dateString) {
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
  const serviceId = selectedService.value
  const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value
  
  // Convert date string to timestamps for the original API
  const date = new Date(dateString + 'T00:00:00')
  const start = new Date(date)
  start.setHours(0, 0, 0, 0)
  const end = new Date(date)
  end.setHours(23, 59, 59, 999)

  const startDate = start.getTime()
  const endDate = end.getTime()

  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/getAllstaffslot?calendarId=${serviceId}&startDate=${startDate}&endDate=${endDate}`
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
    
    // Filter slots by business hours
    const filteredSlots = filterSlotsByBusinessHours(allSlots, date)
    
    const slotsWithStatus = filteredSlots.map(slot => ({
      time: slot,
      isPast: isSlotInPastMST(slot, dateString)
    }))

    slotsForDate.value = slotsWithStatus
    console.log('Fallback slots loaded for date:', dateString, slotsWithStatus)
    
    if (slotsWithStatus.length === 0) {
    }
  } catch (error) {
    console.error('Error fetching slots for date:', error)
    slotsForDate.value = []
  } finally {
    loadingSlots.value = false
  }
}

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
  
  return slotMST < nowMST
}

const currentDateIndex = ref(0)
const availableDates = ref([])
const visibleDates = computed(() => {
  return availableDates.value.slice(currentDateIndex.value, currentDateIndex.value + 3)
})

function generateAvailableDates() {
  const dates = []
  const today = new Date()
  
  // Generate next 14 days
  for (let i = 0; i < 14; i++) {
    const date = new Date(today)
    date.setDate(today.getDate() + i)
    
    const dateString = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
    const dayName = date.toLocaleDateString('en-US', { weekday: 'long' })
    const dateDisplay = date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
    
    let label = ''
    if (i === 0) label = 'TODAY'
    else if (i === 1) label = 'TOMORROW'
    else if (i <= 7) label = 'THIS WEEK'
    else label = 'NEXT WEEK'
    
    dates.push({
      dateString,
      dayName,
      dateDisplay,
      label,
      date
    })
  }
  
  availableDates.value = dates
}

function navigateDate(direction) {
  const newIndex = currentDateIndex.value + direction
  if (newIndex >= 0 && newIndex <= availableDates.value.length - 3) {
    currentDateIndex.value = newIndex
    // Auto-select first visible date after navigation and fetch slots
    const firstVisibleDate = availableDates.value[newIndex]
    if (firstVisibleDate) {
      selectDate(firstVisibleDate)
    }
  }
}

function selectDate(dateInfo) {
  console.log('Selecting date:', dateInfo.dateString)
  selectedDateString.value = dateInfo.dateString
  selectedSlot.value = '' // Clear selected slot when changing date
  
  // Convert to CalendarDate for compatibility with existing logic
  const [year, month, day] = dateInfo.dateString.split('-').map(Number)
  selectedCalendarDate.value = new CalendarDate(year, month, day)
  
  // Always fetch slots when date is selected
  if (selectedService.value && selectedStaff.value) {
    fetchSlotsForDate(dateInfo.dateString)
  }
}

function selectTimeSlot(time) {
  console.log('Selecting time slot:', time, 'for date:', selectedDateString.value)
  selectedSlot.value = time
  // Auto-navigate to booking step when slot is selected
  setTimeout(() => {
    currentStep.value = 'StepInformation'
  }, 500) // Small delay for better UX
}

const enabledSlotsForDate = computed(() => {
  return slotsForDate.value.filter(slot => !slot.disabled && !slot.isPast)
})

function formatDateForDisplay(dateString) {
  if (!dateString) return ''
  const [year, month, day] = dateString.split('-').map(Number)
  const date = new Date(year, month - 1, day)
  return date.toLocaleDateString('en-US', { 
    weekday: 'long', 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric' 
  })
}

const selectedDateString = ref('')
const userTimezone = ref('')

onMounted(async () => {
  generateAvailableDates()
  userTimezone.value = 'MST'
  
  // Auto-select first available date
  if (availableDates.value.length > 0) {
    selectDate(availableDates.value[0])
  }

  // Check for ?id=... in URL (works for Nuxt or Vue Router projects)
  let idFromQuery = ''
  if (route && route.query && route.query.id) {
    idFromQuery = route.query.id
  } else if (typeof window !== 'undefined') {
    // fallback for plain Vue: parse window.location.search
    const params = new URLSearchParams(window.location.search)
    idFromQuery = params.get('id')
  }
  if (idFromQuery) {
    preselectedDepartmentId.value = idFromQuery
  }
})

watch(selectedDateString, (newDateString, oldDateString) => {
  console.log('Date changed from', oldDateString, 'to', newDateString)
  selectedSlot.value = ''
  if (newDateString && currentStep.value === 'StepDateTime' && selectedService.value && selectedStaff.value) {
    console.log('Fetching slots for new date:', newDateString)
    fetchSlotsForDate(newDateString)
  } else {
    console.log('Clearing slots - missing requirements:', {
      dateString: newDateString,
      step: currentStep.value,
      service: selectedService.value,
      staff: selectedStaff.value
    })
    slotsForDate.value = []
  }
})

watch(currentStep, (step) => {
  console.log('Step changed to:', step)
  if (step === 'StepDateTime' && selectedService.value && selectedStaff.value) {
    generateAvailableDates()
    if (availableDates.value.length > 0 && !selectedDateString.value) {
      console.log('Auto-selecting first available date')
      selectDate(availableDates.value[0])
    }
    fetchActiveSlots()
  }
})

// Previous step logic
function goToPreviousStep() {
  if (currentStep.value === 'StepService') {
    currentStep.value = 'StepDepartment'
  } else if (currentStep.value === 'StepStaff') {
    currentStep.value = 'StepService'
  } else if (currentStep.value === 'StepDateTime') {
    currentStep.value = 'StepStaff'
  } else if (currentStep.value === 'StepInformation') {
    currentStep.value = 'StepDateTime'
  }
}

function handleDepartmentSubmit() {
  if (selectedDepartment.value) {
    currentStep.value = 'StepService'
  }
}

function handleServiceSubmit() {
  if (selectedService.value) {
    currentStep.value = 'StepStaff'
  }
}

function handleStaffSubmit() {
  if (selectedStaff.value) {
    currentStep.value = 'StepDateTime'
  }
}

function getServiceDuration(serviceId) {
  const service = serviceRadioItems.value.find(s => s.value === serviceId)
  return service ? service.description.match(/Duration: (\d+) mins/)?.[1] || '' : ''
}

function goToNextStepDateTime() {
  if (selectedSlot.value) {
    currentStep.value = 'StepInformation'
  }
}

// Cache for contactId by email
const contactIdCacheKey = 'restyle_contact_ids_by_email'

// Utility to get/set contactId by email in localStorage
function getContactIdByEmail(email) {
  if (!email) return null
  try {
    const map = JSON.parse(localStorage.getItem(contactIdCacheKey) || '{}')
    return map[email.toLowerCase()] || null
  } catch {
    return null
  }
}
function setContactIdForEmail(email, contactId) {
  if (!email || !contactId) return
  try {
    const map = JSON.parse(localStorage.getItem(contactIdCacheKey) || '{}')
    map[email.toLowerCase()] = contactId
    localStorage.setItem(contactIdCacheKey, JSON.stringify(map))
  } catch {}
}

async function handleInformationSubmit() {
  if (!validateForm()) {
    return
  }

  bookingLoading.value = true

  try {
    // 1. Try to get contactId from localStorage by email
    let contactId = getContactIdByEmail(contactForm.value.email)
    if (!contactId) {
      // Create contact if not found
      const params = new URLSearchParams({
        firstName: contactForm.value.firstName,
        lastName: contactForm.value.lastName,
        email: contactForm.value.email,
        phone: contactForm.value.phone,
        notes: contactForm.value.notes || 'From landing page'
      })

      console.log('Creating contact with params:', params.toString())
      const contactRes = await fetch(`https://restyle-api.netlify.app/.netlify/functions/createContact?${params.toString()}`)
      const contactData = await contactRes.json()
      console.log('Contact creation response:', contactData)

      if (!contactData.success || !contactData.contact?.contact?.id) {
        throw new Error('Contact creation failed')
      }

      contactId = contactData.contact.contact.id
      setContactIdForEmail(contactForm.value.email, contactId)
    } else {
      console.log('Using cached contactId:', contactId)
    }

    // 2. Book appointment
    const jsDate = calendarDateToJSDate(selectedCalendarDate.value)
    // Parse selectedSlot time
    const slotMatch = selectedSlot.value.match(/(\d{1,2}):(\d{2})\s*(AM|PM)/i)
    let hour = 9, minute = 0
    if (slotMatch) {
      hour = parseInt(slotMatch[1])
      minute = parseInt(slotMatch[2])
      const period = slotMatch[3].toUpperCase()
      if (period === 'PM' && hour !== 12) hour += 12
      if (period === 'AM' && hour === 12) hour = 0
    }

    // Set the time in MST (UTC-7)
    jsDate.setHours(hour, minute, 0, 0)

    // Convert MST to UTC for API
    const mstOffset = -7 * 60 * 60 * 1000 // MST is UTC-7
    const utcStartTime = new Date(jsDate.getTime() - mstOffset)

    // Get service duration
    const duration = parseInt(getServiceDuration(selectedService.value)) || 120
    const utcEndTime = new Date(utcStartTime.getTime() + duration * 60 * 1000)

    const startTime = utcStartTime.toISOString()
    const endTime = utcEndTime.toISOString()

    // --- Assign staff logic ---
    let assignedUserId = selectedStaff.value
    // If user selected "Any available staff", pick a random real staff from staffRadioItems (excluding "any")
    if (assignedUserId === 'any' || !assignedUserId) {
      // staffRadioItems: first is "any", rest are real staff
      const realStaff = staffRadioItems.value.filter(item => item.value !== 'any')
      if (realStaff.length > 0) {
        const randomStaff = realStaff[Math.floor(Math.random() * realStaff.length)]
        assignedUserId = randomStaff.value
      } else {
        throw new Error('No staff available for this service')
      }
    }

    let bookUrl = `https://restyle-api.netlify.app/.netlify/functions/bookAppointment?contactId=${contactId}&calendarId=${selectedService.value}&startTime=${startTime}&endTime=${endTime}`
    if (assignedUserId) bookUrl += `&assignedUserId=${assignedUserId}`

    console.log('Booking URL:', bookUrl)
    const bookRes = await fetch(bookUrl)
    const bookData = await bookRes.json()
    console.log('Booking response:', bookData)

    if (!bookData.response?.id) {
      throw new Error(bookData.error || 'Booking failed')
    }

    bookingResponse.value = bookData.response
    currentStep.value = 'StepSuccess'

  } catch (err) {
    console.error('Booking error:', err)
  } finally {
    bookingLoading.value = false
  }
}

function resetBooking() {
  // Reset all form data
  currentStep.value = 'StepDepartment'
  selectedDepartment.value = ''
  selectedService.value = ''
  selectedStaff.value = ''
  selectedCalendarDate.value = null
  selectedSlot.value = ''
  guestCount.value = 1
  contactForm.value = {
    firstName: '',
    lastName: '',
    phone: '',
    email: '',
    notes: '',
    optIn: false
  }
  validationErrors.value = {
    firstName: '',
    lastName: '',
    phone: '',
    email: ''
  }
  bookingResponse.value = null
  slotsForDate.value = []
}

function selectDepartment(value) {
  selectedDepartment.value = value
  // Wait for UI update, then move to next step
  setTimeout(() => {
    handleDepartmentSubmit()
  }, 100)
}

function selectService(value) {
  selectedService.value = value
  setTimeout(() => {
    handleServiceSubmit()
  }, 100)
}

function selectStaff(value) {
  selectedStaff.value = value
  setTimeout(() => {
    handleStaffSubmit()
  }, 100)
}

// --- Add this at the top of <script setup> ---
const route = typeof useRoute === 'function' ? useRoute() : null
const preselectedDepartmentId = ref('')

// --- On mount, check for ?id=... in URL and preselect department ---
onMounted(async () => {
  // ...existing code...
  // Check for ?id=... in URL (works for Nuxt or Vue Router projects)
  let idFromQuery = ''
  if (route && route.query && route.query.id) {
    idFromQuery = route.query.id
  } else if (typeof window !== 'undefined') {
    // fallback for plain Vue: parse window.location.search
    const params = new URLSearchParams(window.location.search)
    idFromQuery = params.get('id')
  }
  if (idFromQuery) {
    preselectedDepartmentId.value = idFromQuery
  }
})

// --- After departmentRadioItems are loaded, auto-select department if preselectedDepartmentId is set ---
watch([departmentRadioItems, preselectedDepartmentId], ([items, preId]) => {
  if (preId && items.length > 0) {
    const found = items.find(item => item.value === preId)
    if (found) {
      selectedDepartment.value = preId
      // Optionally, auto-advance to next step:
      setTimeout(() => {
        handleDepartmentSubmit()
      }, 100)
    }
  }
})
</script>

<style scoped>
/* Custom animations and transitions */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* Custom scrollbar for slots */
.overflow-y-auto::-webkit-scrollbar {
  width: 6px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: #f8fafc;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #e2e8f0;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: #cbd5e1;
}
.information-depaerment-form input {
    background: transparent !important;
    width: 100%;
    border: 1px solid #751A29 !important;
    box-shadow: none !important;
}

.information-depaerment-form textarea {
    background: transparent;
    width: 100% !important;
     max-width: 100% !important;
    border: 1px solid #751A29 !important;
    box-shadow: none !important;
}
.text-highlighted {  background: transparent !important; }

.information-depaerment-form  .inline-flex.items-center {
    width: 100% !important;
    justify-content: center;
}

.information-depaerment-form input,
.information-depaerment-form textarea,
.information-depaerment-form :deep(input),
.information-depaerment-form :deep(textarea) {
  background: transparent !important;
  width: 100%;
  max-width: 100%;
  box-shadow: none !important;
  border: 1px solid #751A29 !important;
}

.information-depaerment-form :deep(.n-checkbox__label),
.information-depaerment-form :deep(.u-checkbox__label),
.information-depaerment-form :deep(label) {
  color: #364a63 !important;
}
.department-inner :deep(.text-sm),
.department-inner .text-sm {
  color: #364a63;
}
.quantity-guest :deep(input) {
  background: #fef2f2 !important;
  color: #1d293d !important;
  border-radius: 50px !important;
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px !important;
  border: 1px solid #1d293d !important;
  font-weight: 600 !important;
}

.quantity-guest :deep(.iconify) {
  background: #1d293d !important;
  color: #fff !important; 
  border-radius: 50% !important;
}

:deep(.same-btn-prev-next) {
  justify-content: space-between;
}

:deep(.same-btn-prev-next button) {
  width: max-content !important;
  max-width: max-content !important;
  opacity: 1 !important;
  color:#fff !important;
    background: #751A29;
    border: 1px solid #751A29;
}
:deep(.same-btn-prev-next button:hover) {
  background: transparent;
  color: #751A29 !important;
  cursor:pointer;
}

/* Stepper color scheme overrides */
:deep(.department-inner .u-stepper .u-stepper-item) {
  background: transparent !important;
}
:deep(.department-inner .u-stepper .u-stepper-item .u-stepper-icon) {
  background: #374151 !important; /* Inactive background */
  color: #fff !important;         /* Inactive icon color */
  border: none !important;
  transition: background 0.2s;
}
:deep(.department-inner .u-stepper .u-stepper-item.is-active .u-stepper-icon) {
  background: #751e2b !important; /* Active background */
  color: #fff !important;         /* Active icon color */
}
:deep(.department-inner .u-stepper .u-stepper-item .u-stepper-title) {
  color: #374151 !important;      /* Inactive text */
}
:deep(.department-inner .u-stepper .u-stepper-item.is-active .u-stepper-title) {
  color: #751e2b !important;      /* Active text */
  font-weight: bold;
}

/* Active tab: icon white, background your main color */
:deep(.u-tabs .u-tab.is-active .u-tab-icon),
:deep(.u-tabs .u-tab.is-active .u-icon),
:deep(.u-tabs .u-tab.is-active .iconify) {
  color: #fff !important;
  background: #751A29 !important;
  border-radius: 50% !important;
}

/* Optionally, set inactive tab icon color */
:deep(.u-tabs .u-tab .u-tab-icon),
:deep(.u-tabs .u-tab .u-icon),
:deep(.u-tabs .u-tab .iconify) {
  color: #751A29 !important;
  background: #fff !important;
  border-radius: 50% !important;
}
:deep(.department-block button.rounded-full) {
  color: #fff;
}
:deep(.calender-main[data-selected]) {
    color: #fff !important;
}
:deep(.calender-main [aria-selected] .text-lg) {
  color: #fff;
}

:deep(.same-block-content) {
  display: grid;
  grid-template-columns: 1fr 1fr;
    gap: 10px;
}

:deep(.same-block-content .cursor-pointer) {
  padding: 15px;
}

:deep(.same-block-content .cursor-pointer span) {
  font-size: 16px;
}

:deep(.same-block-content .cursor-pointer .bg-gray-100) {
 width:40px;
 height:40px;
}
:deep(.same-block-content .cursor-pointer .bg-red-500) {
  background: #751a29;
  font-size: 14px !important;
}
:deep(.information-depaerment-form .text-inverted) {
  color: #fff !important;
}


@media only screen and (max-width: 1023px) {
:deep(.appointment-summary-mobile) {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

:deep(.appointment-summary-mobile .appointment-summary-mobile-block) {
  width: 100%;
  max-width: 100%;
  margin: 0;
  padding: 15px;
}
}





@media only screen and (max-width: 767px) {
:deep(.success-last .img-icon) {
  width: 50px;
  height: 50px;
}

:deep(.success-last .img-icon .text-white) {
  width: 24px;
  height: 24px;
}
.flex.gap-4.flex-col.p-8.department-inner {
    padding: 30px 15px;
    gap:0px;
}

:deep(.department-inner .flex .group) {
  display: none !important;
}
:deep(.same-block-content .transition-all) {
  font-size: 16px !important;
  padding: 12px;
}

:deep(.same-block-content .transition-all span) {
  font-size: 16px;
}
:deep(.book-heading-top h1) {
  font-size: 26px;
}
:deep(.book-heading-top p) {
  font-size: 16px;
}
:deep(.same-block-content) {
  grid-template-columns: 1fr;
}
}
@media only screen and (max-width: 599px) {
:deep(.appointment-summary-mobile) {
  gap: 15px;
}

:deep(.appointment-summary-mobile .appointment-summary-mobile-block) {
  padding: 15px 10px;
}

:deep(.appointment-summary-mobile .appointment-summary-mobile-block span) {
  font-size: 14px;
}

:deep(.appointment-summary-mobile .appointment-summary-mobile-block .text-lg) {
  font-size: 14px;
}
:deep(.same-block-content .cursor-pointer .bg-red-500) {
  font-size: 11px !important;

}
:deep(.department-inner .booking-mobile) {
  font-size: 24px;
}

}

@media only screen and (max-width: 399px) {
  :deep(.appointment-summary-mobile) {
  grid-template-columns:1fr;
}
}
</style>
