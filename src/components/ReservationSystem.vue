<script setup>
import { ref } from "vue";
import AddBranchesModal from "./AddBranchesModal.vue";
import BranchSettingsModal from "./BranchSettingsModal.vue";

const branches = ref([
  {
    name: "Branch 1",
    reference: "B01",
    tables: 0,
    duration: "30 Minutes",
  },
]);

const toggleReservations = () => {
  // Implementation for toggling reservations
  console.log("Toggling reservations");
};

const isModalOpen = ref(false);
const isSettingsModalOpen = ref(false);
const selectedBranch = ref({
  name: "",
  reference: "",
  tables: 0,
  duration: 30, // Changed to number to match the settings modal
});

const addBranch = () => {
  isModalOpen.value = true;
};

const handleAddBranch = (branchReference) => {
  // Add new branch to the branches array
  branches.value.push({
    name: `Branch ${branches.value.length + 1}`,
    reference: branchReference,
    tables: 0,
    duration: "30 Minutes",
  });
};

const openBranchSettings = (branch) => {
  selectedBranch.value = {
    ...branch,
    duration: parseInt(branch.duration), // Convert duration to number
  };
  isSettingsModalOpen.value = true;
};

const handleSaveSettings = (settings) => {
  // Update branch settings in your data store
  const branchIndex = branches.value.findIndex(
    (b) => b.reference === selectedBranch.value.reference
  );

  if (branchIndex !== -1) {
    branches.value[branchIndex] = {
      ...branches.value[branchIndex],
      duration: `${settings.duration} Minutes`,
      tables: settings.tables.length,
    };
  }

  console.log("Saving settings:", settings);
};
</script>

<template>
  <div>
    <!-- Modals should be at root level -->
    <AddBranchesModal
      :is-open="isModalOpen"
      @close="isModalOpen = false"
      @save="handleAddBranch"
    />

    <BranchSettingsModal
      :is-open="isSettingsModalOpen"
      :branch-data="selectedBranch"
      @close="isSettingsModalOpen = false"
      @save="handleSaveSettings"
    />

    <div class="p-6 bg-gray-50 min-h-screen">
      <div class="flex justify-between items-center mb-8">
        <h1 class="text-2xl text-gray-700 font-normal">Reservations</h1>
        <button
          class="px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors"
          @click="toggleReservations"
        >
          Disable Reservations
        </button>
      </div>

      <div class="bg-white rounded-lg shadow-sm p-6">
        <div class="flex justify-end mb-6">
          <button
            class="px-4 py-2 border border-gray-300 rounded-lg text-gray-700 hover:bg-gray-50 transition-colors"
            @click="addBranch"
          >
            Add Branches
          </button>
        </div>

        <table class="w-full">
          <thead>
            <tr class="border-b">
              <th class="text-left py-4 font-medium text-gray-900">Branch</th>
              <th class="text-left py-4 font-medium text-gray-900">
                Reference
              </th>
              <th class="text-left py-4 font-medium text-gray-900">
                Number of Tables
              </th>
              <th class="text-left py-4 font-medium text-gray-900">
                Reservation Duration
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="branch in branches"
              :key="branch.reference"
              @click="openBranchSettings(branch)"
              class="cursor-pointer hover:bg-gray-50"
            >
              <td class="py-4 text-gray-700">{{ branch.name }}</td>
              <td class="py-4 text-gray-700">{{ branch.reference }}</td>
              <td class="py-4 text-gray-700">{{ branch.tables }}</td>
              <td class="py-4 text-gray-700">{{ branch.duration }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
