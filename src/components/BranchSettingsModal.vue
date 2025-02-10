<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const props = defineProps({
  isOpen: Boolean,
  branchData: {
    type: Object,
    required: true,
    default: () => ({
      name: "",
      duration: 30,
      tables: [],
    }),
  },
});

const emit = defineEmits(["close", "save"]);

// State management
const isDropdownOpen = ref(false);
const duration = ref(props.branchData.duration);
const selectedTables = ref([]);
const timeSlots = ref({
  Saturday: [],
  Sunday: [],
  Monday: [],
  Tuesday: [],
  Wednesday: [],
  Thursday: [],
  Friday: [],
});

const daysOfWeek = [
  "Saturday",
  "Sunday",
  "Monday",
  "Tuesday",
  "Wednesday",
  "Thursday",
  "Friday",
];

const availableTables = [
  { id: 1, name: "Terrace - New Table 1" },
  { id: 2, name: "Terrace - New Table 2" },
  { id: 3, name: "Terrace - New Table 3" },
  { id: 4, name: "Terrace - New Table 4" },
  { id: 5, name: "Terrace - New Table 5" },
];

// Dropdown management
const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

const toggleTableSelection = (table) => {
  const index = selectedTables.value.findIndex((t) => t.id === table.id);
  if (index === -1) {
    selectedTables.value.push(table);
  } else {
    selectedTables.value.splice(index, 1);
  }
};

const isTableSelected = (table) => {
  return selectedTables.value.some((t) => t.id === table.id);
};

// Time slot management
const addTimeSlot = (day) => {
  if (timeSlots.value[day].length < 3) {
    timeSlots.value[day].push({ start: "00:00", end: "00:00" });
  }
};

const removeTimeSlot = (day, index) => {
  timeSlots.value[day].splice(index, 1);
};

const applyToAllDays = () => {
  const saturdaySlots = [...timeSlots.value.Saturday];
  daysOfWeek.forEach((day) => {
    if (day !== "Saturday") {
      timeSlots.value[day] = [...saturdaySlots];
    }
  });
};

// Modal actions
const close = () => {
  emit("close");
};

const save = () => {
  emit("save", {
    duration: duration.value,
    tables: selectedTables.value,
    timeSlots: { ...timeSlots.value },
  });
  close();
};

// Click outside handler
const handleClickOutside = (event) => {
  if (isDropdownOpen.value && !event.target.closest(".table-dropdown")) {
    isDropdownOpen.value = false;
  }
};

onMounted(() => {
  document.addEventListener("click", handleClickOutside);
});

onUnmounted(() => {
  document.removeEventListener("click", handleClickOutside);
});

// New functionality for time range input
const activeDayInput = ref(null); // Track which day's input is active
const newTimeSlot = ref({ start: "", end: "" });

const saveTimeSlot = (day) => {
  if (newTimeSlot.value.start && newTimeSlot.value.end) {
    timeSlots.value[day].push({
      start: newTimeSlot.value.start,
      end: newTimeSlot.value.end,
    });
    newTimeSlot.value = { start: "", end: "" };
    activeDayInput.value = null; // Close the input field
  }
};

const cancelTimeSlot = (day) => {
  newTimeSlot.value = { start: "", end: "" };
  activeDayInput.value = null; // Close the input field
};
</script>

<template>
  <div
    v-if="isOpen"
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4"
    aria-modal="true"
    role="dialog"
  >
    <div
      class="bg-white rounded-lg w-full max-w-2xl shadow-lg overflow-y-auto max-h-[90vh] relative"
    >
      <!-- Modal Header -->
      <div class="px-6 py-4 border-b flex justify-between items-center">
        <h2 class="text-xl font-semibold text-gray-800">
          Edit {{ branchData.name }} branch reservation settings
        </h2>
        <button
          @click="close"
          class="text-gray-500 hover:text-gray-700 transition-colors"
          aria-label="Close modal"
        >
          <svg
            class="w-6 h-6"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </button>
      </div>

      <!-- Modal Body -->
      <div class="px-6 py-4 space-y-6">
        <!-- Working Hours Notice -->
        <div class="bg-blue-50 p-4 text-blue-600 rounded-lg text-sm">
          Branch working hours are 00:00 - 00:00
        </div>

        <!-- Reservation Duration -->
        <div class="space-y-2">
          <label class="block text-gray-700 font-medium">
            Reservation Duration (minutes) <span class="text-red-500">*</span>
          </label>
          <input
            v-model="duration"
            type="number"
            class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent transition-all"
            aria-label="Reservation duration in minutes"
          />
        </div>

        <!-- Tables Selection -->
        <div class="relative w-full table-dropdown">
          <label class="block text-gray-700 font-medium mb-2">Tables</label>
          <div class="relative">
            <!-- Selected Tables Display -->
            <div
              @click="toggleDropdown"
              class="w-full px-4 py-3 border rounded-lg bg-white flex items-center justify-between cursor-pointer hover:border-purple-500 transition-all"
              :aria-expanded="isDropdownOpen"
            >
              <div class="flex flex-wrap gap-2 items-center">
                <template v-if="selectedTables.length === 0">
                  <span class="text-gray-500">Select tables</span>
                </template>
                <template v-else>
                  <div
                    v-for="table in selectedTables.slice(0, 2)"
                    :key="table.id"
                    class="px-3 py-1 bg-white border border-purple-400 text-purple-600 rounded-full text-sm"
                  >
                    {{ table.name }}
                  </div>
                  <span v-if="selectedTables.length > 2" class="text-gray-600">
                    +{{ selectedTables.length - 2 }} more
                  </span>
                </template>
              </div>
              <svg
                class="w-5 h-5 text-gray-400 transition-transform"
                :class="{ 'transform rotate-180': isDropdownOpen }"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M19 9l-7 7-7-7"
                />
              </svg>
            </div>

            <!-- Dropdown Menu -->
            <div
              v-if="isDropdownOpen"
              class="absolute z-10 w-full mt-1 bg-white border rounded-lg shadow-lg max-h-60 overflow-auto"
            >
              <div
                v-for="table in availableTables"
                :key="table.id"
                @click="toggleTableSelection(table)"
                class="px-4 py-2 hover:bg-gray-50 cursor-pointer flex items-center justify-between"
              >
                <span>{{ table.name }}</span>
                <svg
                  v-if="isTableSelected(table)"
                  class="w-5 h-5 text-purple-600"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M5 13l4 4L19 7"
                  />
                </svg>
              </div>
            </div>
          </div>
        </div>

        <!-- Time Slots Section -->
        <div class="space-y-4">
          <div v-for="day in daysOfWeek" :key="day" class="space-y-2">
            <div class="flex justify-between items-center">
              <h3 class="text-gray-700 font-medium">{{ day }}</h3>
              <button
                v-if="day === 'Saturday'"
                @click="applyToAllDays"
                class="text-purple-600 hover:text-purple-700 text-sm"
              >
                Apply on all days
              </button>
            </div>

            <div class="bg-gray-50 p-4 rounded-lg">
              <div class="flex items-center justify-between">
                <div class="flex flex-wrap gap-2 flex-grow">
                  <!-- Display Existing Time Slots -->
                  <div
                    v-for="(slot, index) in timeSlots[day]"
                    :key="index"
                    class="relative group"
                  >
                    <div
                      class="px-4 py-2 border border-purple-500 rounded-lg text-purple-600 flex items-center text-sm"
                    >
                      {{ slot.start }} - {{ slot.end }}
                      <button
                        @click.stop="removeTimeSlot(day, index)"
                        class="ml-2 opacity-0 group-hover:opacity-100 transition-opacity text-red-600 hover:text-red-700"
                        aria-label="Remove time slot"
                      >
                        ✖️
                      </button>
                    </div>
                  </div>

                  <!-- Placeholder when no time slots are added -->
                  <div
                    v-if="timeSlots[day].length === 0 && activeDayInput !== day"
                    class="text-gray-500 italic text-sm"
                  >
                    Add available reservation Time
                  </div>

                  <!-- Input Field for New Time Slot -->
                  <div
                    v-if="activeDayInput === day"
                    class="flex items-center gap-2"
                  >
                    <input
                      v-model="newTimeSlot.start"
                      type="time"
                      class="px-4 py-2 border border-purple-500 rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent transition-all"
                      aria-label="Start time"
                    />
                    <span class="text-gray-500">to</span>
                    <input
                      v-model="newTimeSlot.end"
                      type="time"
                      class="px-4 py-2 border border-purple-500 rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent transition-all"
                      aria-label="End time"
                    />
                    <button
                      @click="saveTimeSlot(day)"
                      class="px-3 py-2 bg-gray-200 text-white rounded-lg hover:bg-green-500 transition-colors"
                      aria-label="Save time slot"
                    >
                      ✔️
                    </button>
                    <button
                      @click="cancelTimeSlot(day)"
                      class="px-3 py-2 bg-gray-200 text-gray-700 rounded-lg hover:bg-red-500 hover:text-white transition-colors"
                      aria-label="Cancel time slot"
                    >
                      ✖️
                    </button>
                  </div>
                </div>

                <!-- Add Icon -->
                <button
                  v-if="timeSlots[day].length < 3 && activeDayInput !== day"
                  @click="activeDayInput = day"
                  :disabled="timeSlots[day].length >= 3"
                  class="w-8 h-8 flex items-center justify-center rounded-lg border border-gray-300 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
                  aria-label="Add time slot"
                >
                  <svg
                    class="w-5 h-5 text-gray-600"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M12 4v16m8-8H4"
                    />
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Modal Footer -->
      <div class="px-6 py-4 flex justify-between border-t">
        <button
          class="text-red-500 hover:text-red-600 font-medium transition-colors"
        >
          Disable Reservations
        </button>
        <div class="space-x-4">
          <button
            @click="close"
            class="px-6 py-2 border border-gray-300 rounded-lg text-gray-700 hover:bg-gray-50 transition-colors"
          >
            Close
          </button>
          <button
            @click="save"
            class="px-6 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors"
          >
            Save
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Add smooth transitions for hover effects */
button,
input,
.dropdown-item {
  transition: all 0.2s ease-in-out;
}

/* Improve focus states for accessibility */
button:focus,
input:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(147, 51, 234, 0.3);
}
</style>
