<script setup lang="ts">
import EventListComponent from "@/components/Post/EventListComponent.vue";
import { useUserStore } from "@/stores/user";
import { storeToRefs } from "pinia";

const { currentUsername, isLoggedIn } = storeToRefs(useUserStore());
</script>

<template>
  <main>
    <h1>Home Page</h1>
    <section>
      <h1 v-if="isLoggedIn">Welcome {{ currentUsername }}!</h1>
      <h1 v-else>Please login!</h1>
    </section>
    <router-link to="/posts" custom v-slot="{ navigate }">
      <button @click="navigate" role="link">Create an Event!</button>
    </router-link>
    <EventListComponent />
  </main>
</template>

<style scoped>
h1 {
  text-align: center;
}
</style>
