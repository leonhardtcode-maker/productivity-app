<template>
  <h1>Productivity App</h1>
  <div class="dark-mode-switch">
    <!-- <n-switch size="medium" v-model:value="darkMode"> -->
    <n-switch size="medium">
      <template #checked-icon>
        <n-icon :component="SunnyOutline" />
      </template>
      <template #checked> Light Mode </template>
      <template #unchecked-icon>
        <n-icon :component="MoonOutline" />
      </template>
      <template #unchecked> Dark Mode </template>
    </n-switch>
  </div>
</template>
<script setup>
import { inject } from "vue";

import { SunnyOutline, MoonOutline } from "@vicons/ionicons5";
import { NIcon } from "naive-ui";

// Switch Dark Mode
const darkMode = inject("darkMode");
</script>
<style scoped>
h1 {
  margin: 0;
  color: var(--bg);
}

.dark-mode-switch {
  height: 2rem;
  border-radius: 0.25rem;
  align-self: flex-end;
}
</style>
