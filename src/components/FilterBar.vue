<template>
  <div class="config-bar">
    <!-- Search -->
    <n-input
      :value="props.filters.search"
      placeholder="Search Task"
      clearable
      @update:value="searchUpdate"
      class="search-bar"
    >
      <template #prefix>
        <n-icon :component="SearchOutline" />
      </template>
    </n-input>
    <!-- Filter By -->
    <div class="filter-item">
      <n-select
        :value="props.filters.filter"
        :options="filterOptions"
        clearable
        placeholder="Filter By"
        @update:value="selectedFilterUpdate"
        class="select-field"
      />
    </div>
  </div>
</template>

<script setup>
import { defineEmits, defineProps } from "vue";

import { SearchOutline } from "@vicons/ionicons5";
import { NIcon } from "naive-ui";

const props = defineProps({
  filters: Object,
});
const emit = defineEmits(["update:filters"]);

// Function to emit an update if search bar got input
const searchUpdate = (val) => {
  emit("update:filters", { ...props.filters, search: val });
};

// Function to emit an update if filter is selected
const selectedFilterUpdate = (val) => {
  emit("update:filters", { ...props.filters, filter: val ?? null });
};

// Filter options
const filterOptions = [
  {
    label: "Status",
    type: "group",
    key: "Status",
    children: [
      { label: "To Do", value: "To Do" },
      { label: "In Progress", value: "In Progress" },
      { label: "Done", value: "Done" },
    ],
  },
  {
    label: "Priority",
    type: "group",
    key: "Priority",
    children: [
      { label: "Low", value: "Low" },
      { label: "Medium", value: "Medium" },
      { label: "High", value: "High" },
    ],
  },
  {
    label: "Category",
    type: "group",
    key: "Category",
    children: [
      { label: "Mind", value: "Mind" },
      { label: "Soul", value: "Soul" },
      { label: "Social", value: "Social" },
      { label: "Career", value: "Career" },
    ],
  },
];
</script>

<style scoped>
.config-bar {
  display: flex;
  gap: 12px;
  width: 100%;
  flex-wrap: wrap;
  margin-bottom: var(--spacing-m);
}

.search-bar {
  flex: 3 1 300px;
}

.filter-item {
  flex: 1 1 auto;
  min-width: 160px;
}

.select-field {
  width: 100%;
}
</style>
