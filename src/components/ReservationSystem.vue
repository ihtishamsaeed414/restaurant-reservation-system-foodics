<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import AddBranchesModal from "./AddBranchesModal.vue";
import BranchSettingsModal from "./BranchSettingsModal.vue";
import ConfirmationModal from "./ConfirmationModal.vue";
import Spinner from "./Spinner.vue";

const branches = ref([]);
const isModalOpen = ref(false);
const isSettingsModalOpen = ref(false);
const isConfirmationModalOpen = ref(false);
const isLoading = ref(false);
const selectedBranch = ref({
  name: "",
  reference: "",
  tables: 0,
  duration: 30,
  timeSlots: {
    Saturday: [],
    Sunday: [],
    Monday: [],
    Tuesday: [],
    Wednesday: [],
    Thursday: [],
    Friday: [],
  },
});

const api = axios.create({
  baseURL: "/api",
  headers: {
    Authorization:
      "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiI4Zjc4NjY2NC0wMDg4LTRhMDMtYmFkMi0zOTQ5OGRjNTViYjMiLCJqdGkiOiJiMjgzOGY4NmMyMjQ3NGMzMmJlMTJjY2JiZGZkYzU5YjEzODUyNzkxNGQyMWVhNWU4OTVjYjkxODQ0ZWZiM2YwNzQ0NjJlYzU3NGIwZDUxNyIsImlhdCI6MTczNzk3NDIwMC4zMzU0MjYsIm5iZiI6MTczNzk3NDIwMC4zMzU0MjYsImV4cCI6MTg5NTc0MDYwMC4yODM5ODYsInN1YiI6IjllMTE1ZDRkLTQ4YzMtNGM0YS1iYTAzLWMxMTFkNWYwZjAyYSIsInNjb3BlcyI6W10sImJ1c2luZXNzIjoiOTdhNTcxNDYtZTBhZC00ZGQ5LWE1NGUtNDk3YTIyY2I2Yzc1IiwicmVmZXJlbmNlIjoiMjg3NDM1In0.hU5wB_6dwzTf4eo-LBUE9TK6B5cuuMMlJfAI1QTJwCJwz1G67kAGP_E_hfkn7Tl0Hz6qG-Yrdd_esYIg_PkLCe1uL1Ejwptk-1QFzHhE0Dz0Fz-hQGW1MIr2FeKorQZvCSOQpBfnCpr1qEnMw1YYNFusGJc4R8Yh59cRjWCvFCID1tzIfA7ZnkSBBLJKeHvy7SuQUXN37rz6TQQyqNsXV9jVaOt0QTg6KE-rcGGxLK49IoBVzuRohg1NGQD_S01BV7UyFJq_mNCjlthQUxV3rH0c0BEgIPODYgJyIX3tLlaccGNbrLxH_TK-bKKL53YzdXfrze6OCsEvZlKITQlb-NjC0RJD3jlwLpeze3ubNXKETUz3GiR2CeyndIYwANzV5tAdUucmc8mMspYwSGDXwsqbDHpE9VLz9p_XdHnLIQ3IT9hroyAX9Fuhce6b_J3iZtmMvSaEwLJk2ubEZzHRm2KFndQQ6AiacxbX6BncfgaNXqeYM59gk91Q1kn7SA2AEo8ta0fubmA5TZ_rqBmRjj90-xom1fJFAir8vsw3Vp0AZksf8CogGUkb7tchhnK-b1gjpwLG5EUtZvwvy9FA7t16HNScveaSbLWMTT-6cItQnEe1wXAkw9zf4HkicjOCYB6iqdWXNS22qnUweFyNS7iVwOknCcIRBclUWY6kNro",
    "Content-Type": "application/json",
    Accept: "application/json",
  },
});

const fetchBranches = async () => {
  try {
    const response = await api.get("branches");
    // Filter branches that accept reservations and transform the data
    branches.value = response.data.data
      .filter((branch) => branch.accepts_reservations)
      .map((branch) => ({
        name: branch.name,
        reference: branch.reference,
        tables: countReservationTables(branch),
        duration: `${branch.reservation_duration} Minutes`,
        timeSlots: branch.reservation_times || {
          Saturday: [],
          Sunday: [],
          Monday: [],
          Tuesday: [],
          Wednesday: [],
          Thursday: [],
          Friday: [],
        },
      }));
  } catch (error) {
    console.error("Error fetching branches:", error);
  }
};

const countReservationTables = (branch) => {
  return (
    branch.sections?.reduce((total, section) => {
      const reservationTables =
        section.tables?.filter((table) => table.accepts_reservations) || [];
      return total + reservationTables.length;
    }, 0) || 0
  );
};

onMounted(() => {
  fetchBranches();
});

const addBranch = () => {
  isModalOpen.value = true;
};

const handleAddBranch = (branchReference) => {
  branches.value.push({
    name: `Branch ${branches.value.length + 1}`,
    reference: branchReference,
    tables: 0,
    duration: "30 Minutes",
    timeSlots: {
      Saturday: [],
      Sunday: [],
      Monday: [],
      Tuesday: [],
      Wednesday: [],
      Thursday: [],
      Friday: [],
    },
  });
};

const openBranchSettings = (branch) => {
  console.log(branch);
  selectedBranch.value = {
    ...branch,
    duration: parseInt(branch.duration),
    tables: branch.tables || 0,
    timeSlots: branch.timeSlots || {
      Saturday: [],
      Sunday: [],
      Monday: [],
      Tuesday: [],
      Wednesday: [],
      Thursday: [],
      Friday: [],
    },
  };
  isSettingsModalOpen.value = true;
};

const handleSaveSettings = (settings) => {
  const branchIndex = branches.value.findIndex(
    (b) => b.reference === selectedBranch.value.reference
  );

  if (branchIndex !== -1) {
    branches.value[branchIndex] = {
      ...branches.value[branchIndex],
      duration: `${settings.duration} Minutes`,
      tables: settings.tables.length,
      timeSlots: settings.timeSlots,
    };
  }
};

const disableAllReservations = async () => {
  try {
    // Fetching all branches first
    const fetchResponse = await api.get("branches");
    const branchesData = fetchResponse.data.data;
    const updatePromises = branchesData.map((branch) => {
      return api.put(`branches/${branch.id}`, {
        accepts_reservations: false,
      });
    });

    // Waiting for all PUT requests to complete
    await Promise.all(updatePromises);
    console.log("All reservations disabled for all branches");

    // Filtering out branches that have reservations disabled
    branches.value = branches.value.filter(
      (branch) => branch.accepts_reservations
    );
  } catch (error) {
    console.error("Error disabling all reservations:", error);
  } finally {
    isLoading.value = false;
  }
};

const onConfirmDisableReservations = async () => {
  isConfirmationModalOpen.value = false;
  isLoading.value = true;
  await disableAllReservations();
};
</script>

<template>
  <div>
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

    <ConfirmationModal
      :is-open="isConfirmationModalOpen"
      @confirm="onConfirmDisableReservations"
      @cancel="isConfirmationModalOpen = false"
    />

    <Spinner v-if="isLoading" />

    <div class="p-6 bg-gray-50 min-h-screen">
      <div class="flex justify-between items-center mb-8">
        <h1 class="text-2xl text-gray-700 font-normal">Reservations</h1>
        <button
          class="px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors"
          @click="isConfirmationModalOpen = true"
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
