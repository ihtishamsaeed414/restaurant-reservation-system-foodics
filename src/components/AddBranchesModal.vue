<script setup>
import { ref } from "vue";
const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(["close", "save"]);

const selectedBranch = ref("");

const close = () => {
  emit("close");
  selectedBranch.value = "";
};

const save = () => {
  if (selectedBranch.value) {
    emit("save", selectedBranch.value);
    close();
  }
};
</script>

<template>
  <div
    v-if="props.isOpen"
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center"
  >
    <div class="bg-white rounded-lg w-full max-w-md shadow-lg">
      <!-- Modal Header -->
      <div class="px-6 py-4 border-b">
        <h2 class="text-xl text-gray-800">Add Branches</h2>
      </div>

      <!-- Modal Body -->
      <div class="px-6 py-4 bg-gray-50">
        <div class="space-y-4">
          <h3 class="text-lg text-gray-700">Branches</h3>

          <!-- Dropdown with custom styling -->
          <div class="relative">
            <select
              v-model="selectedBranch"
              class="w-full px-4 py-3 bg-white border rounded-lg appearance-none pr-10 text-gray-700 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent"
            >
              <option value="" disabled>Select a branch</option>
              <option value="B02">branch 2 (B02)</option>
              <option value="B03">branch 3 (B03)</option>
              <option value="B04">branch 4 (B04)</option>
              <option value="B05">branch 5 (B05)</option>
              <option value="B06">branch 6 (B06)</option>
            </select>
            <!-- Custom dropdown arrow -->
            <div
              class="absolute inset-y-0 right-0 flex items-center px-2 pointer-events-none"
            >
              <svg
                class="w-5 h-5 text-gray-400"
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
          </div>
        </div>
      </div>

      <!-- Modal Footer -->
      <div class="px-6 py-4 flex justify-end space-x-4 border-t">
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
</template>
