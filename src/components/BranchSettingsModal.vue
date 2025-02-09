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
    timeSlots.value[day].push("00:00 - 00:00");
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
</script>

<template>
  <div
    v-if="isOpen"
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center"
  >
    <div
      class="bg-white rounded-lg w-full max-w-2xl shadow-lg overflow-y-auto max-h-[90vh]"
    >
      <!-- Modal Header -->
      <div class="px-6 py-4 border-b">
        <h2 class="text-xl text-gray-800">
          Edit {{ branchData.name }} branch reservation settings
        </h2>
      </div>

      <!-- Modal Body -->
      <div class="px-6 py-4 space-y-6">
        <!-- Working Hours Notice -->
        <div class="bg-blue-50 p-4 text-blue-600 rounded-lg">
          Branch working hours are 00:00 - 00:00
        </div>

        <!-- Reservation Duration -->
        <div class="space-y-2">
          <label class="block text-gray-700">
            Reservation Duration (minutes) <span class="text-red-500">*</span>
          </label>
          <input
            v-model="duration"
            type="number"
            class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent"
          />
        </div>

        <!-- Tables Selection -->
        <div class="relative w-full table-dropdown">
          <label class="block text-gray-700 mb-2">Tables</label>
          <div class="relative">
            <!-- Selected Tables Display -->
            <div
              @click="toggleDropdown"
              class="w-full px-4 py-3 border rounded-lg bg-white flex items-center justify-between cursor-pointer"
            >
              <div class="flex flex-wrap gap-2 items-center">
                <template v-if="selectedTables.length === 0">
                  <span class="text-gray-500">Select tables</span>
                </template>
                <template v-else>
                  <div
                    v-for="table in selectedTables.slice(0, 2)"
                    :key="table.id"
                    class="px-3 py-1 bg-white border border-blue-400 text-blue-600 rounded-full"
                  >
                    {{ table.name }}
                  </div>
                  <span v-if="selectedTables.length > 2" class="text-gray-600">
                    +{{ selectedTables.length - 2 }} more
                  </span>
                </template>
              </div>
              <svg
                class="w-5 h-5 text-gray-400"
                :class="{ 'transform rotate-180': isDropdownOpen }"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="2"
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
                    strokeLinecap="round"
                    strokeLinejoin="round"
                    strokeWidth="2"
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
              <h3 class="text-gray-700">{{ day }}</h3>
              <button
                v-if="day === 'Saturday'"
                @click="applyToAllDays"
                class="text-purple-600 hover:text-purple-700"
              >
                Apply on all days
              </button>
            </div>

            <div class="bg-gray-50 p-4 rounded-lg">
              <div class="flex items-center justify-between">
                <div class="flex flex-wrap gap-2 flex-grow">
                  <div v-if="timeSlots[day].length === 0" class="text-gray-500">
                    Add Available Reservation Times
                  </div>

                  <div
                    v-for="(slot, index) in timeSlots[day]"
                    :key="index"
                    class="relative group"
                  >
                    <div
                      class="px-4 py-2 border border-purple-500 rounded-lg text-purple-600 flex items-center"
                    >
                      {{ slot }}
                      <button
                        @click.stop="removeTimeSlot(day, index)"
                        class="ml-2 opacity-0 group-hover:opacity-100 transition-opacity"
                      >
                        ×
                      </button>
                    </div>
                  </div>
                </div>

                <button
                  v-if="timeSlots[day].length < 3"
                  @click="addTimeSlot(day)"
                  class="ml-4 w-8 h-8 flex items-center justify-center rounded-lg border border-gray-300 hover:bg-gray-50"
                >
                  <svg
                    class="w-5 h-5 text-gray-600"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      strokeLinecap="round"
                      strokeLinejoin="round"
                      strokeWidth="2"
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
        <button class="text-red-500 hover:text-red-600">
          Disable Reservations
        </button>
        <div class="space-x-4">
          <button
            @click="close"
            class="px-6 py-2 border border-gray-300 rounded-lg text-gray-700 hover:bg-gray-50"
          >
            Close
          </button>
          <button
            @click="save"
            class="px-6 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700"
          >
            Save
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
