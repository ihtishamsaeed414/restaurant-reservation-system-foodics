<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(["close", "save"]);

const selectedBranch = ref("");
const branches = ref([]);
const isLoading = ref(false);

const fetchBranches = async () => {
  isLoading.value = true;
  try {
    const response = await axios.get("/api/branches", {
      headers: {
        Accept: "application/json",
        Authorization:
          "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiI4Zjc4NjY2NC0wMDg4LTRhMDMtYmFkMi0zOTQ5OGRjNTViYjMiLCJqdGkiOiJiMjgzOGY4NmMyMjQ3NGMzMmJlMTJjY2JiZGZkYzU5YjEzODUyNzkxNGQyMWVhNWU4OTVjYjkxODQ0ZWZiM2YwNzQ0NjJlYzU3NGIwZDUxNyIsImlhdCI6MTczNzk3NDIwMC4zMzU0MjYsIm5iZiI6MTczNzk3NDIwMC4zMzU0MjYsImV4cCI6MTg5NTc0MDYwMC4yODM5ODYsInN1YiI6IjllMTE1ZDRkLTQ4YzMtNGM0YS1iYTAzLWMxMTFkNWYwZjAyYSIsInNjb3BlcyI6W10sImJ1c2luZXNzIjoiOTdhNTcxNDYtZTBhZC00ZGQ5LWE1NGUtNDk3YTIyY2I2Yzc1IiwicmVmZXJlbmNlIjoiMjg3NDM1In0.hU5wB_6dwzTf4eo-LBUE9TK6B5cuuMMlJfAI1QTJwCJwz1G67kAGP_E_hfkn7Tl0Hz6qG-Yrdd_esYIg_PkLCe1uL1Ejwptk-1QFzHhE0Dz0Fz-hQGW1MIr2FeKorQZvCSOQpBfnCpr1qEnMw1YYNFusGJc4R8Yh59cRjWCvFCID1tzIfA7ZnkSBBLJKeHvy7SuQUXN37rz6TQQyqNsXV9jVaOt0QTg6KE-rcGGxLK49IoBVzuRohg1NGQD_S01BV7UyFJq_mNCjlthQUxV3rH0c0BEgIPODYgJyIX3tLlaccGNbrLxH_TK-bKKL53YzdXfrze6OCsEvZlKITQlb-NjC0RJD3jlwLpeze3ubNXKETUz3GiR2CeyndIYwANzV5tAdUucmc8mMspYwSGDXwsqbDHpE9VLz9p_XdHnLIQ3IT9hroyAX9Fuhce6b_J3iZtmMvSaEwLJk2ubEZzHRm2KFndQQ6AiacxbX6BncfgaNXqeYM59gk91Q1kn7SA2AEo8ta0fubmA5TZ_rqBmRjj90-xom1fJFAir8vsw3Vp0AZksf8CogGUkb7tchhnK-b1gjpwLG5EUtZvwvy9FA7t16HNScveaSbLWMTT-6cItQnEe1wXAkw9zf4HkicjOCYB6iqdWXNS22qnUweFyNS7iVwOknCcIRBclUWY6kNro",
      },
    });
    branches.value = response.data.data.filter(
      (branch) => !branch.accepts_reservations
    );
  } catch (error) {
    console.error("Error fetching branches:", error);
  } finally {
    isLoading.value = false;
  }
};

onMounted(() => {
  fetchBranches();
});

const close = () => {
  emit("close");
  selectedBranch.value = "";
};

const save = async () => {
  if (selectedBranch.value) {
    try {
      isLoading.value = true;
      await axios.put(
        `/api/branches/${selectedBranch.value}`,
        {
          accepts_reservations: true,
        },
        {
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
            Authorization:
              "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiI4Zjc4NjY2NC0wMDg4LTRhMDMtYmFkMi0zOTQ5OGRjNTViYjMiLCJqdGkiOiJiMjgzOGY4NmMyMjQ3NGMzMmJlMTJjY2JiZGZkYzU5YjEzODUyNzkxNGQyMWVhNWU4OTVjYjkxODQ0ZWZiM2YwNzQ0NjJlYzU3NGIwZDUxNyIsImlhdCI6MTczNzk3NDIwMC4zMzU0MjYsIm5iZiI6MTczNzk3NDIwMC4zMzU0MjYsImV4cCI6MTg5NTc0MDYwMC4yODM5ODYsInN1YiI6IjllMTE1ZDRkLTQ4YzMtNGM0YS1iYTAzLWMxMTFkNWYwZjAyYSIsInNjb3BlcyI6W10sImJ1c2luZXNzIjoiOTdhNTcxNDYtZTBhZC00ZGQ5LWE1NGUtNDk3YTIyY2I2Yzc1IiwicmVmZXJlbmNlIjoiMjg3NDM1In0.hU5wB_6dwzTf4eo-LBUE9TK6B5cuuMMlJfAI1QTJwCJwz1G67kAGP_E_hfkn7Tl0Hz6qG-Yrdd_esYIg_PkLCe1uL1Ejwptk-1QFzHhE0Dz0Fz-hQGW1MIr2FeKorQZvCSOQpBfnCpr1qEnMw1YYNFusGJc4R8Yh59cRjWCvFCID1tzIfA7ZnkSBBLJKeHvy7SuQUXN37rz6TQQyqNsXV9jVaOt0QTg6KE-rcGGxLK49IoBVzuRohg1NGQD_S01BV7UyFJq_mNCjlthQUxV3rH0c0BEgIPODYgJyIX3tLlaccGNbrLxH_TK-bKKL53YzdXfrze6OCsEvZlKITQlb-NjC0RJD3jlwLpeze3ubNXKETUz3GiR2CeyndIYwANzV5tAdUucmc8mMspYwSGDXwsqbDHpE9VLz9p_XdHnLIQ3IT9hroyAX9Fuhce6b_J3iZtmMvSaEwLJk2ubEZzHRm2KFndQQ6AiacxbX6BncfgaNXqeYM59gk91Q1kn7SA2AEo8ta0fubmA5TZ_rqBmRjj90-xom1fJFAir8vsw3Vp0AZksf8CogGUkb7tchhnK-b1gjpwLG5EUtZvwvy9FA7t16HNScveaSbLWMTT-6cItQnEe1wXAkw9zf4HkicjOCYB6iqdWXNS22qnUweFyNS7iVwOknCcIRBclUWY6kNro",
          },
        }
      );

      emit("save", selectedBranch.value);

      await fetchBranches();

      close();
    } catch (error) {
      console.error("Error updating branch:", error);
    } finally {
      isLoading.value = false;
    }
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

          <div class="relative">
            <select
              v-model="selectedBranch"
              class="w-full px-4 py-3 bg-white border rounded-lg appearance-none pr-10 text-gray-700 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent"
              :disabled="isLoading"
            >
              <option value="" disabled>Select a branch</option>
              <option
                v-for="branch in branches"
                :key="branch.id"
                :value="branch.id"
              >
                {{ branch.name }}
              </option>
            </select>

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

          <!-- Add loading indicator -->
          <div v-if="isLoading" class="text-sm text-gray-500 mt-2">
            Loading branches...
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
          :disabled="!selectedBranch || isLoading"
          class="px-6 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700 transition-colors disabled:bg-purple-300"
        >
          {{ isLoading ? "Saving..." : "Save" }}
        </button>
      </div>
    </div>
  </div>
</template>
