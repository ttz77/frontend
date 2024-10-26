<!-- src/components/EndorseComponent.vue -->

<template>
  <div class="endorse-component">
    <h2>Endorse a User</h2>
    <form @submit.prevent="handleEndorse">
      <div class="form-group">
        <label for="endorseUsername">Username:</label>
        <input type="text" id="endorseUsername" v-model="endorseUsername" required placeholder="Enter the username to endorse" />
      </div>

      <div class="form-group">
        <label for="skill">Skill:</label>
        <input type="text" id="skill" v-model="skill" required placeholder="Enter the skill to endorse" />
      </div>

      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? "Endorsing..." : "Endorse" }}
      </button>
    </form>

    <div class="existing-endorsements">
      <h3>Existing Endorsements for {{ targetUsername }}</h3>
      <div v-if="isLoading">
        <p>Loading endorsements...</p>
      </div>
      <div v-else>
        <ul v-if="endorsements.length > 0">
          <li v-for="endorsement in endorsements" :key="endorsement">
            <p>{{ endorsement }}</p>
          </li>
        </ul>
        <p v-else>No endorsements yet.</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import { storeToRefs } from "pinia";
import { useEndorsementsStore } from "@/stores/endorsements";
import { useToastStore } from "@/stores/toast"; // Assuming you have a toast store
import { defineProps } from "vue";
import { useUserStore } from "@/stores/user";

// Define props
const props = defineProps<{
  username: string; // The target username whose endorsements are being viewed
}>();

// Access user store to get the current logged-in user
const userStore = useUserStore();
const { currentUsername } = storeToRefs(userStore);

// Define refs
const targetUsername = ref(props.username || ""); // Username whose endorsements are viewed
const endorseUsername = ref(""); // Username to endorse via the form
const skill = ref(""); // Skill to endorse

const isSubmitting = ref(false);
const isRemoving = ref<string[]>([]); // Tracks skills being removed

// Access endorsements store
const endorsementsStore = useEndorsementsStore();
const { endorsements, isLoading } = storeToRefs(endorsementsStore);

// Access toast store for notifications
const toastStore = useToastStore();

/**
 * Handle endorsement submission.
 */
const handleEndorse = async () => {
  if (!endorseUsername.value || !skill.value) {
    toastStore.showToast({
      message: "Please provide both username and skill.",
      style: "error",
    });
    return;
  }

  // Prevent self-endorsement
  if (endorseUsername.value === currentUsername.value) {
    toastStore.showToast({
      message: "You cannot endorse yourself.",
      style: "error",
    });
    return;
  }

  isSubmitting.value = true;
  try {
    await endorsementsStore.endorseUser(endorseUsername.value, skill.value);
    toastStore.showToast({
      message: `Successfully endorsed ${endorseUsername.value} for ${skill.value}.`,
      style: "success",
    });
    // Clear input fields after successful endorsement
    endorseUsername.value = "";
    skill.value = "";
  } catch (error: any) {
    toastStore.showToast({
      message: error.message || "Failed to endorse the user.",
      style: "error",
    });
  } finally {
    isSubmitting.value = false;
  }
};

/**
 * Handle removal of an endorsement.
 * @param skillToRemove - The skill endorsement to remove.
 */
const handleRemoveEndorsement = async (skillToRemove: string) => {
  if (!targetUsername.value) {
    toastStore.showToast({
      message: "Please provide the username first.",
      style: "error",
    });
    return;
  }

  isRemoving.value.push(skillToRemove);
  try {
    await endorsementsStore.removeEndorsement(targetUsername.value, skillToRemove);
    toastStore.showToast({
      message: `Removed endorsement for ${skillToRemove}.`,
      style: "success",
    });
  } catch (error: any) {
    toastStore.showToast({
      message: error.message || "Failed to remove endorsement.",
      style: "error",
    });
  } finally {
    isRemoving.value = isRemoving.value.filter((skill) => skill !== skillToRemove);
  }
};

/**
 * Fetch existing endorsements on component mount and whenever the targetUsername changes.
 */
onMounted(async () => {
  if (targetUsername.value) {
    await endorsementsStore.fetchEndorsements(targetUsername.value);
    console.log("Fetched endorsements on mount:", endorsements.value); // Debugging
  }
});

/**
 * Watch the 'username' prop for changes and fetch endorsements accordingly.
 */
watch(
  () => props.username,
  async (newUsername, oldUsername) => {
    if (newUsername) {
      targetUsername.value = newUsername;
      await endorsementsStore.fetchEndorsements(newUsername);
      console.log("Fetched endorsements on username change:", endorsements.value); // Debugging
    } else {
      endorsements.value = [];
      targetUsername.value = "";
      console.log("No username provided, cleared endorsements.");
    }
  },
  { immediate: true },
);
</script>

<style scoped>
.endorse-component {
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-top: 1rem;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
}

input[type="text"] {
  width: 100%;
  padding: 0.5rem;
  box-sizing: border-box;
}

button {
  padding: 0.5rem 1rem;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button[disabled] {
  background-color: #6c757d;
  cursor: not-allowed;
}

.existing-endorsements {
  margin-top: 2rem;
}

.existing-endorsements ul {
  list-style-type: none;
  padding: 0;
}

.existing-endorsements li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0;
}

.existing-endorsements button {
  background-color: #dc3545;
}

.existing-endorsements button:hover {
  background-color: #c82333;
}
</style>
