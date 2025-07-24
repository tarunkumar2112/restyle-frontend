<template>
  <div class="min-h-screen bg-white">
    <div class="flex flex-col items-center gap-6 pt-12 pb-16 px-4">
      <div class="text-center mb-8">
        <h1 class="font-bold text-4xl mb-3 text-black">
          Book Your Appointment
        </h1>
        <p class="text-lg text-gray-700 max-w-2xl mx-auto leading-relaxed">
          Experience professional hair and beauty services with our expert stylists in a luxurious environment
        </p>
      </div>
      
      <div class="w-full max-w-4xl bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
        <UStepper :items="steps" v-model="currentStep" class="p-8">
          <template #StepDepartment>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Choose Your Department</h2>
                <p class="text-gray-700">Select the service category that best fits your needs</p>
              </div>
              
              <form @submit.prevent="handleDepartmentSubmit">
                <div v-if="loadingGroups" class="space-y-4">
                  <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in 6" :key="i" />
                </div>
                <div v-else class="grid gap-4 mb-8">
                  <div
                    v-for="item in departmentRadioItems"
                    :key="item.value"
                    @click="selectedDepartment = item.value"
                    :class="[
                      'cursor-pointer p-6 border-2 rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm',
                      selectedDepartment === item.value
                        ? 'bg-green-50 text-black border-green-500 shadow-sm'
                        : 'bg-white text-black border-gray-200 hover:border-green-300'
                    ]"
                  >
                    <div class="flex items-center gap-4">
                      <div :class="[
                        'p-3 rounded-full',
                        selectedDepartment === item.value ? 'bg-green-100' : 'bg-gray-100'
                      ]">
                        <UIcon :name="item.icon || 'i-lucide-user-female'" :class="[
                          'text-2xl',
                          selectedDepartment === item.value ? 'text-green-600' : 'text-gray-600'
                        ]" />
                      </div>
                      <div>
                        <span class="text-xl font-semibold block text-black">{{ item.label }}</span>
                        <span class="text-sm text-gray-600">{{ item.description }}</span>
                      </div>
                    </div>
                    <div v-if="selectedDepartment === item.value" class="text-green-600">
                      <UIcon name="i-lucide-check-circle" class="text-2xl" />
                    </div>
                  </div>
                </div>
                
                <div class="flex gap-4">
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
                    class="flex-1 bg-green-600 hover:bg-green-700 text-white"
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
                <div v-else class="grid gap-4 mb-8">
                  <div
                    v-for="item in serviceRadioItems"
                    :key="item.value"
                    @click="selectedService = item.value"
                    :class="[
                      'cursor-pointer p-6 border-2 rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm',
                      selectedService === item.value
                        ? 'bg-green-50 text-black border-green-500 shadow-sm'
                        : 'bg-white text-black border-gray-200 hover:border-green-300'
                    ]"
                  >
                    <div class="flex items-center gap-4">
                      <div :class="[
                        'p-3 rounded-full',
                        selectedService === item.value ? 'bg-green-100' : 'bg-gray-100'
                      ]">
                        <UIcon name="i-lucide-scissors" :class="[
                          'text-2xl',
                          selectedService === item.value ? 'text-green-600' : 'text-gray-600'
                        ]" />
                      </div>
                      <div>
                        <span class="text-xl font-semibold block text-black">{{ item.label }}</span>
                        <span class="text-sm text-gray-600">{{ item.description }}</span>
                      </div>
                    </div>
                    <div v-if="selectedService === item.value" class="text-green-600">
                      <UIcon name="i-lucide-check-circle" class="text-2xl" />
                    </div>
                  </div>
                </div>
                
                <div class="flex gap-4">
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
                    class="flex-1 bg-green-600 hover:bg-green-700 text-white"
                  >
                    Continue
                    <UIcon name="i-lucide-arrow-right" class="ml-2" />
                  </UButton>
                </div>
              </form>
            </div>
          </template>

          <template #StepGuests>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Number of Guests</h2>
                <p class="text-gray-700">How many people will be joining you for this service?</p>
              </div>
              
              <div class="max-w-md mx-auto">
                <div class="mb-6 p-6 rounded-xl border-2 border-gray-200 bg-white">
                  <div class="flex items-center justify-between">
                    <div>
                      <div class="font-bold text-xl mb-1 text-black">
                        {{ selectedServiceObj?.label || 'No service selected' }}
                      </div>
                      <div class="text-gray-600 text-sm">
                        {{ selectedServiceObj?.description }}
                      </div>
                    </div>
                    <div class="flex items-center gap-3 bg-green-50 rounded-full px-4 py-2 border border-green-200">
                      <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-2xl text-green-600" />
                      <span class="font-bold text-xl text-black">{{ guestCount }}</span>
                    </div>
                  </div>
                </div>
                
                <div class="text-center mb-6">
                  <label class="block font-semibold mb-4 text-lg text-black">Select Number of Guests</label>
                  <div class="flex justify-center">
                    <UInputNumber
                      v-model="guestCount"
                      :min="1"
                      :max="10"
                      size="xl"
                      class="w-48"
                    />
                  </div>
                </div>
                
                <div class="text-center text-lg font-medium text-black bg-white rounded-lg p-4 border border-gray-200">
                  You{{ guestCount > 1 ? ' and ' + (guestCount - 1) + ' guest' + (guestCount > 2 ? 's' : '') : '' }}
                </div>
              </div>
              
              <div class="flex gap-4 max-w-md mx-auto">
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
                  :disabled="!guestCount"
                  class="flex-1 bg-green-600 hover:bg-green-700 text-white"
                  @click="goToNextStep"
                >
                  Continue
                  <UIcon name="i-lucide-arrow-right" class="ml-2" />
                </UButton>
              </div>
            </div>
          </template>

          <template #StepStaff>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Choose Your Stylist</h2>
                <p class="text-gray-700">Select your preferred stylist or let us choose the best available</p>
              </div>
              
              <div class="grid grid-cols-2 gap-6 mb-8 max-w-lg mx-auto">
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200">
                  <div class="font-bold text-lg mb-1 text-black">Service</div>
                  <div class="text-gray-700">{{ selectedServiceObj?.label }}</div>
                  <div class="text-sm text-green-600 font-semibold mt-1">
                    Duration: {{ getServiceDuration(selectedService) }} mins
                  </div>
                </div>
                <div class="text-center p-4 bg-white rounded-xl border border-gray-200">
                  <div class="font-bold text-lg mb-1 text-black">Guests</div>
                  <div class="flex items-center justify-center gap-2">
                    <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-2xl text-green-600" />
                    <span class="font-bold text-xl text-black">{{ guestCount }}</span>
                  </div>
                </div>
              </div>
              
              <form @submit.prevent="handleStaffSubmit">
                <div v-if="loadingStaff" class="space-y-4">
                  <USkeleton class="h-20 rounded-xl bg-gray-100" v-for="i in staffRadioItems.length || 3" :key="i" />
                </div>
                <div v-else class="grid gap-4 mb-8">
                  <div
                    v-for="item in staffRadioItems"
                    :key="item.value"
                    @click="selectedStaff = item.value"
                    :class="[
                      'cursor-pointer p-6 border-2 rounded-xl flex items-center justify-between transition-all duration-200 hover:shadow-sm',
                      selectedStaff === item.value
                        ? 'bg-green-50 text-black border-green-500 shadow-sm'
                        : 'bg-white text-black border-gray-200 hover:border-green-300'
                    ]"
                  >
                    <div class="flex items-center gap-4">
                      <div :class="[
                        'p-3 rounded-full',
                        selectedStaff === item.value ? 'bg-green-100' : 'bg-gray-100'
                      ]">
                        <UIcon :name="item.icon" :class="[
                          'text-2xl',
                          selectedStaff === item.value ? 'text-green-600' : 'text-gray-600'
                        ]" />
                      </div>
                      <div>
                        <span class="text-xl font-semibold text-black">{{ item.label }}</span>
                        <span v-if="item.badge" :class="[
                          'ml-3 px-3 py-1 rounded-full text-xs font-medium',
                          selectedStaff === item.value ? 'bg-green-100 text-green-700' : 'bg-green-500 text-white'
                        ]">{{ item.badge }}</span>
                      </div>
                    </div>
                    <div v-if="selectedStaff === item.value" class="text-green-600">
                      <UIcon name="i-lucide-check-circle" class="text-2xl" />
                    </div>
                  </div>
                </div>
                
                <div class="flex gap-4">
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
                    class="flex-1 bg-green-600 hover:bg-green-700 text-white"
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
                <p class="text-gray-700">Choose your preferred appointment date and time</p>
              </div>
              
              <!-- Summary cards -->
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
                <div class="p-4 rounded-xl border border-gray-200 bg-white text-center">
                  <div class="font-bold text-lg mb-1 text-black">Service</div>
                  <div class="text-green-600 font-semibold">
                    {{ selectedServiceObj?.label || 'No service selected' }}
                  </div>
                </div>
                <div class="p-4 rounded-xl border border-gray-200 bg-white text-center">
                  <div class="font-bold text-lg mb-1 text-black">Guests</div>
                  <div class="flex items-center justify-center gap-2">
                    <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-2xl text-green-600" />
                    <span class="font-bold text-xl text-black">{{ guestCount }}</span>
                  </div>
                </div>
                <div class="p-4 rounded-xl border border-gray-200 bg-white text-center">
                  <div class="font-bold text-lg mb-1 text-black">Stylist</div>
                  <div class="text-green-600 font-semibold text-center">{{ selectedStaffObj?.label || 'Any available' }}</div>
                </div>
              </div>

              <!-- Timezone indicator -->
              <div class="mb-6 p-4 bg-gray-50 rounded-xl border border-gray-200">
                <div class="flex items-center justify-center gap-3">
                  <UIcon name="i-lucide-globe" class="text-gray-600 text-xl" />
                  <span class="font-medium text-black">All times shown in MST (UTC-7)</span>
                </div>
              </div>
              
              <!-- Calendar and slots -->
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <!-- Calendar -->
                <div class="space-y-4">
                  <h3 class="font-bold text-xl text-black text-center">Select Date</h3>
                  <div class="bg-white rounded-xl border border-gray-200 p-6 shadow-sm">
                    <UCalendar 
                      v-model="selectedCalendarDate"
                      size="xl"
                      :min-date="today(getLocalTimeZone())"
                    />
                  </div>
                  <div v-if="selectedCalendarDate" class="text-center p-3 bg-green-50 rounded-lg border border-green-200">
                    <span class="text-black font-semibold">{{ formatCalendarDate(selectedCalendarDate) }}</span>
                  </div>
                </div>
                
                <!-- Time slots -->
                <div class="space-y-4">
                  <h3 class="font-bold text-xl text-black text-center">Available Times (MST)</h3>
                  <div class="bg-white rounded-xl border border-gray-200 p-6 shadow-sm min-h-[400px]">
                    <div v-if="loadingSlots" class="space-y-3">
                      <div class="grid grid-cols-2 gap-3">
                        <USkeleton class="h-12 rounded-lg bg-gray-100" v-for="i in 8" :key="i" />
                      </div>
                    </div>
                    <div v-else-if="slotsForDate.length > 0" class="space-y-4">
                      <div class="grid grid-cols-2 gap-3 max-h-80 overflow-y-auto pr-2">
                        <UButton
                          v-for="slot in slotsForDate"
                          :key="slot.time"
                          :color="selectedSlot === slot.time ? 'primary' : 'gray'"
                          :variant="selectedSlot === slot.time ? 'solid' : 'soft'"
                          size="sm"
                          class="py-3 transition-all duration-200"
                          @click="selectedSlot = slot.time"
                          :class="[
                            selectedSlot === slot.time ? 'bg-green-600 hover:bg-green-700 text-white ring-2 ring-green-300 shadow-sm' : '',
                            slot.isPast ? 'opacity-40' : 'hover:shadow-sm'
                          ]"
                          :disabled="slot.isPast"
                        >
                          {{ slot.time }}
                        </UButton>
                      </div>
                    </div>
                    <div v-else class="flex flex-col items-center justify-center h-full text-gray-500 space-y-4">
                      <UIcon name="i-lucide-calendar-x" class="text-4xl" />
                      <div class="text-center">
                        <p class="font-medium text-black">{{ selectedCalendarDate ? 'No slots available' : 'Select a date first' }}</p>
                        <p class="text-sm text-gray-600">{{ selectedCalendarDate ? 'Try choosing a different date' : 'Choose a date to see available times' }}</p>
                      </div>
                    </div>
                  </div>
                  
                  <div v-if="selectedSlot" class="p-4 bg-green-50 rounded-xl border border-green-200">
                    <div class="text-center">
                      <div class="font-bold text-black text-lg">Selected Time</div>
                      <div class="text-green-600 font-semibold">{{ selectedSlot }} MST</div>
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- Navigation -->
              <div class="flex gap-4 pt-6">
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
                  class="flex-1 bg-green-600 hover:bg-green-700 text-white"
                  @click="goToNextStepDateTime"
                >
                  Continue
                  <UIcon name="i-lucide-arrow-right" class="ml-2" />
                </UButton>
              </div>
            </div>
          </template>

          <template #StepInformation>
            <div class="space-y-6">
              <div class="text-center mb-8">
                <h2 class="text-2xl font-bold text-black mb-2">Your Information</h2>
                <p class="text-gray-700">Please provide your contact details to complete the booking</p>
              </div>
              
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <!-- Contact Form -->
                <div class="space-y-6">
                  <form class="space-y-6" @submit.prevent="handleInformationSubmit">
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
                      class="w-full font-bold bg-green-600 hover:bg-green-700 text-white"
                    >
                      <UIcon name="i-lucide-calendar-check" class="mr-2" />
                      {{ bookingLoading ? 'Booking Your Appointment...' : 'Book Appointment' }}
                    </UButton>
                  </form>
                </div>
                
                <!-- Appointment Summary -->
                <div class="space-y-4">
                  <h3 class="font-bold text-xl text-black mb-4">Appointment Summary</h3>
                  
                  <div class="space-y-4">
                    <div class="p-6 rounded-xl border border-gray-200 bg-white">
                      <div class="space-y-3">
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-calendar" class="text-xl text-green-600" />
                          <span class="font-semibold text-black">
                            {{ formatCalendarDate(selectedCalendarDate) }}
                          </span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-clock" class="text-xl text-green-600" />
                          <span class="font-semibold text-black">{{ selectedSlot }} MST</span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-hourglass" class="text-xl text-green-600" />
                          <span class="text-gray-700">{{ getServiceDuration(selectedService) }} minutes</span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-globe" class="text-xl text-gray-600" />
                          <span class="text-gray-700">Mountain Standard Time</span>
                        </div>
                      </div>
                    </div>
                    
                    <div class="p-6 rounded-xl border border-gray-200 bg-white">
                      <div class="space-y-3">
                        <div class="font-bold text-lg text-black">{{ selectedServiceObj?.label }}</div>
                        <div class="flex items-center gap-3">
                          <UIcon name="i-lucide-user" class="text-xl text-green-600" />
                          <span class="text-gray-700">with {{ selectedStaffObj?.label }}</span>
                        </div>
                        <div class="flex items-center gap-3">
                          <UIcon :name="guestCount > 1 ? 'i-lucide-users' : 'i-lucide-user'" class="text-xl text-green-600" />
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
            <div class="flex flex-col items-center justify-center min-h-[400px] text-center space-y-6">
              <div class="relative">
                <div class="w-24 h-24 bg-green-600 rounded-full flex items-center justify-center shadow-sm">
                  <UIcon name="i-lucide-check-circle" class="text-4xl text-white" />
                </div>
                <div class="absolute -top-2 -right-2 w-8 h-8 bg-green-100 rounded-full flex items-center justify-center border border-green-200">
                  <UIcon name="i-lucide-sparkles" class="text-lg text-green-600" />
                </div>
              </div>
              
              <div class="space-y-4">
                <h2 class="font-bold text-3xl text-black">Booking Confirmed! 🎉</h2>
                <p class="text-lg text-gray-700 max-w-md">
                  Thank you for choosing us! Your appointment has been successfully booked. 
                  We're excited to see you soon!
                </p>
              </div>
              
              <div class="p-6 bg-green-50 rounded-xl border border-green-200 max-w-md">
                <div class="space-y-2">
                  <div class="font-semibold text-black">What's Next?</div>
                  <div class="text-sm text-gray-700 space-y-1">
                    <p>• You'll receive a confirmation email shortly</p>
                    <p>• We'll send you a reminder 24 hours before</p>
                    <p>• Feel free to call us if you need to reschedule</p>
                  </div>
                </div>
              </div>
              
              <UButton
                color="primary"
                size="lg"
                @click="resetBooking"
                class="mt-6 bg-green-600 hover:bg-green-700 text-white"
              >
                <UIcon name="i-lucide-plus" class="mr-2" />
                Book Another Appointment
              </UButton>
            </div>
          </template>
        </UStepper>
      </div>
    </div>

    <!-- Enhanced Toast Container -->
    <div class="fixed top-4 right-4 z-50 space-y-3">
      <div
        v-for="toast in toasts"
        :key="toast.id"
        :class="[
          'max-w-sm w-full bg-white shadow-lg rounded-xl pointer-events-auto flex ring-1 ring-gray-200 transform transition-all duration-300 overflow-hidden',
          toast.show ? 'translate-x-0 opacity-100 scale-100' : 'translate-x-full opacity-0 scale-95'
        ]"
      >
        <div :class="[
          'w-1 flex-shrink-0',
          toast.type === 'success' ? 'bg-green-500' : '',
          toast.type === 'error' ? 'bg-red-500' : '',
          toast.type === 'warning' ? 'bg-yellow-500' : '',
          toast.type === 'info' ? 'bg-blue-500' : ''
        ]"></div>
        <div class="flex-1 p-4">
          <div class="flex items-start">
            <div class="flex-shrink-0">
              <UIcon 
                :name="getToastIcon(toast.type)"
                :class="[
                  'h-5 w-5',
                  toast.type === 'success' ? 'text-green-500' : '',
                  toast.type === 'error' ? 'text-red-500' : '',
                  toast.type === 'warning' ? 'text-yellow-500' : '',
                  toast.type === 'info' ? 'text-blue-500' : ''
                ]"
              />
            </div>
            <div class="ml-3 w-0 flex-1">
              <p class="text-sm font-semibold text-black">{{ toast.title }}</p>
              <p class="mt-1 text-sm text-gray-600">{{ toast.message }}</p>
            </div>
            <div class="ml-4 flex-shrink-0 flex">
              <button
                @click="removeToast(toast.id)"
                class="inline-flex text-gray-400 hover:text-gray-600 focus:outline-none transition-colors duration-200"
              >
                <UIcon name="i-lucide-x" class="h-4 w-4" />
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import { CalendarDate, today, getLocalTimeZone } from '@internationalized/date'

// Custom Toast System
const toasts = ref([])
let toastId = 0

function showToast(title, message, type = 'info') {
  const id = ++toastId
  const toast = {
    id,
    title,
    message,
    type,
    show: false
  }
  
  toasts.value.push(toast)
  
  // Show toast with animation
  setTimeout(() => {
    toast.show = true
  }, 100)
  
  // Auto remove after 5 seconds
  setTimeout(() => {
    removeToast(id)
  }, 5000)
}

function removeToast(id) {
  const index = toasts.value.findIndex(t => t.id === id)
  if (index > -1) {
    toasts.value[index].show = false
    setTimeout(() => {
      toasts.value.splice(index, 1)
    }, 300)
  }
}

function getToastIcon(type) {
  switch (type) {
    case 'success': return 'i-lucide-check-circle'
    case 'error': return 'i-lucide-x-circle'
    case 'warning': return 'i-lucide-alert-triangle'
    case 'info': return 'i-lucide-info'
    default: return 'i-lucide-info'
  }
}

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

// Calendar and slots
const selectedCalendarDate = ref(null)
const slotsForDate = ref([])
const selectedSlot = ref('')
const loadingSlots = ref(false)

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
    showToast('Error', 'Failed to load departments. Please refresh the page.', 'error')
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
    showToast('Error', 'Failed to load services. Please try again.', 'error')
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
    showToast('Error', 'Failed to load staff. Please try again.', 'error')
  } finally {
    loadingStaff.value = false
  }
})

// Watch for calendar date changes
watch(selectedCalendarDate, (newDate) => {
  console.log('Calendar date changed:', newDate)
  selectedSlot.value = ''
  if (newDate && selectedService.value && selectedStaff.value) {
    const jsDate = calendarDateToJSDate(newDate)
    console.log('Converted JS Date:', jsDate)
    fetchSlots(jsDate)
  } else {
    slotsForDate.value = []
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
    showToast('Department Selected', 'Great choice! Now select your service.', 'success')
  }
}

// Handle service submit
function handleServiceSubmit() {
  if (selectedService.value) {
    currentStep.value = 'StepGuests'
    showToast('Service Selected', 'Perfect! Now specify the number of guests.', 'success')
  }
}

function handleStaffSubmit() {
  if (selectedStaff.value) {
    currentStep.value = 'StepDateTime'
    showToast('Staff Selected', 'Excellent! Now choose your preferred date and time.', 'success')
  }
}

function goToNextStep() {
  if (currentStep.value === 'StepGuests' && guestCount.value) {
    currentStep.value = 'StepStaff'
    showToast('Guest Count Set', 'Great! Now choose your preferred stylist.', 'success')
  }
}

function getServiceDuration(serviceId) {
  const service = serviceRadioItems.value.find(s => s.value === serviceId)
  return service ? service.description.match(/Duration: (\d+) mins/)?.[1] || '' : ''
}

function fetchSlots(date) {
  if (!date || !selectedService.value || !selectedStaff.value) {
    console.log('Missing required data for slot fetch:', {
      date: !!date,
      service: !!selectedService.value,
      staff: !!selectedStaff.value
    })
    slotsForDate.value = []
    return
  }

  console.log('Fetching slots for:', {
    date: date,
    dateString: date.toISOString(),
    service: selectedService.value,
    staff: selectedStaff.value
  })

  selectedSlot.value = ''
  slotsForDate.value = []
  loadingSlots.value = true

  const calendarId = selectedService.value
  const userId = selectedStaff.value === 'any' ? '' : selectedStaff.value

  // Create start and end of day in local time, then convert to UTC timestamps
  const start = new Date(date)
  start.setHours(0, 0, 0, 0)
  const end = new Date(date)
  end.setHours(23, 59, 59, 999)

  // Convert to UTC timestamps
  const startDate = start.getTime()
  const endDate = end.getTime()

  // Build API URL
  let apiUrl = `https://restyle-api.netlify.app/.netlify/functions/getAllstaffslot?calendarId=${calendarId}&startDate=${startDate}&endDate=${endDate}`
  if (userId) {
    apiUrl += `&userId=${userId}`
  }

  console.log('API URL:', apiUrl)

  fetch(apiUrl)
    .then(res => res.json())
    .then(data => {
      console.log('Slots API response:', data)
      const formatted = data.formattedSlots || {}
      const key = Object.keys(formatted)[0]
      const allSlots = key ? formatted[key] : []
      console.log('Raw slots from API:', allSlots)

      // Filter slots by business hours
      const filteredSlots = filterSlotsByBusinessHours(allSlots, date)
      
      // Convert slots to MST and add past/future status
      const slotsWithStatus = filteredSlots.map(slot => ({
        time: slot,
        isPast: isSlotInPast(slot, selectedCalendarDate.value)
      }))

      slotsForDate.value = slotsWithStatus
      console.log('Available slots (filtered and with status):', slotsForDate.value)
      
      if (slotsWithStatus.length === 0) {
        showToast('No Slots Available', 'No time slots available for this date. Please try another date.', 'warning')
      }
    })
    .catch((error) => {
      console.error('Error fetching slots:', error)
      slotsForDate.value = []
      showToast('Error', 'Failed to load available time slots. Please try again.', 'error')
    })
    .finally(() => {
      loadingSlots.value = false
    })
}

function goToNextStepDateTime() {
  if (selectedSlot.value) {
    currentStep.value = 'StepInformation'
    showToast('Time Slot Selected', 'Perfect! Now provide your contact information.', 'success')
  }
}

async function handleInformationSubmit() {
  if (!validateForm()) {
    showToast('Form Validation Error', 'Please correct the errors in the form.', 'error')
    return
  }

  bookingLoading.value = true
  
  try {
    // 1. Create contact
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

    const contactId = contactData.contact.contact.id

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

    console.log('Booking appointment with:', {
      contactId,
      calendarId: selectedService.value,
      startTime,
      endTime,
      selectedSlot: selectedSlot.value,
      duration
    })

    const calendarId = selectedService.value
    const assignedUserId = selectedStaff.value === 'any' ? '' : selectedStaff.value

    let bookUrl = `https://restyle-api.netlify.app/.netlify/functions/bookAppointment?contactId=${contactId}&calendarId=${calendarId}&startTime=${startTime}&endTime=${endTime}`
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
    
    showToast('Booking Confirmed! 🎉', 'Your appointment has been successfully booked!', 'success')
  } catch (err) {
    console.error('Booking error:', err)
    showToast('Booking Failed', err.message || 'Something went wrong. Please try again.', 'error')
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
  
  showToast('Ready for New Booking', 'You can now book another appointment.', 'info')
}
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
</style>