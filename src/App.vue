<template>
  <n-config-provider
    :theme="darkMode ? darkTheme : lightTheme"
    class="theme-provider"
  >
    <div class="bg-fixed"></div>
    <section class="header-section">
      <Header />
    </section>
    <section class="main-content">
      <n-message-provider>
        <MainContent />
      </n-message-provider>
    </section>
  </n-config-provider>
</template>
<script setup>
import Header from "./components/Header.vue";
import MainContent from "./components/MainContent.vue";
import { ref, provide } from "vue";

import { darkTheme, lightTheme } from "naive-ui";

// Switch Dark Mode
const darkMode = ref(false);
provide("darkMode", darkMode);
</script>
<style scoped>
.main-content {
  width: 60%;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding-top: 12px;
}

.bg-fixed {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 25vh;
  z-index: -1;
  background: linear-gradient(rgba(0, 255, 21, 0.15), rgba(0, 255, 21, 0.15)),
    url("./assets/lucas-calloch-P-yzuyWFEIk-unsplash.jpg") no-repeat center;
  background-size: cover;
  background-position: 50% 25%;
}

.header-section {
  width: 60%;
  margin: 0 auto;
  padding-top: 2.5%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.container-lebar {
  height: 5rem;
  width: 100%;
  background-color: var(--bg-light);
  border-radius: 0.25rem;
}
</style>
