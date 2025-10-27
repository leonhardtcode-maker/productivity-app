<template>
  <FilterBar v-model:filters="filters" name="FilterBar" />
  <n-spin v-if="isLoading" size="large" style="margin-top: 20%">
    <template #description> Fetching Data . . .</template>
  </n-spin>
  <n-space vertical v-else>
    <n-message-provider>
      <n-dialog-provider>
        <DataCard
          v-for="data in pagedData"
          :key="data.id"
          :data="data"
          @update-tasks="fetchData"
        />
      </n-dialog-provider>
    </n-message-provider>
    <n-space vertical class="bottom-content" :size="16" align="center">
      <n-pagination
        v-model:page="page"
        v-model:page-size="pageSize"
        :item-count="cleanData.length"
        :page-slot="7"
        @update:page="scrollToTop"
      />
      <n-button type="primary" size="large" @click="showTaskForm = true">
        <template #icon>
          <n-icon><AddCircleOutline /></n-icon>
        </template>
        Add New Task
      </n-button>
    </n-space>
    <n-message-provider>
      <n-modal-provider>
        <TaskForm
          v-model:show="showTaskForm"
          @close-form="showTaskForm = false"
          @update-tasks="fetchData"
        ></TaskForm>
      </n-modal-provider>
    </n-message-provider>
  </n-space>
</template>
<script setup>
import FilterBar from "./FilterBar.vue";
import DataCard from "./DataCard.vue";
import TaskForm from "./TaskForm.vue";
import { ref, computed, watch, onMounted } from "vue";

import { AddCircleOutline } from "@vicons/ionicons5";
import { useMessage } from "naive-ui";

const databaseUrl = "http://localhost:4000/tasks";
const Data = ref([]);
const isLoading = ref(true);

// Fetch Data when Main Content is in DOM
const fetchData = async () => {
  try {
    const res = await fetch(`${databaseUrl}`);
    Data.value = await res.json();
  } catch (err) {
    console.log("Error:", err.message);
  } finally {
    setTimeout(() => {
      isLoading.value = false;
    }, 500);
  }
};
onMounted(fetchData);

// Show Task Form (add task)
const showTaskForm = ref(false);

// Filters object
const filters = ref({
  search: "",
  filter: null,
});

// Filter Options
const statusOptions = ["To Do", "In Progress", "Done"];
const priorityOptions = ["Low", "Medium", "High"];
const categoryOptions = ["Mind", "Soul", "Social", "Career"];

// To change time and date into miliseconds since 1 January 1970
function parseDateTime(date, time) {
  const [day, month, year] = date.split("-").map(Number);
  const [hour, minute] = time.split(":").map(Number);

  return new Date(year, month - 1, day, hour, minute).getTime();
}
// Function to sort data closes to today by default
function defaultSortBy(data) {
  const order = { High: 1, Medium: 2, Low: 3 };
  const now = Date.now();

  return data.slice().sort((a, b) => {
    // 1. Place tasks with 'done' status at the bottom
    if (a.status === "Done" && b.status !== "Done") return 1;
    if (a.status !== "Done" && b.status === "Done") return -1;

    // 2. Convert date and time to timestamps (seconds)
    const timeA = parseDateTime(a.date, a.time);
    const timeB = parseDateTime(b.date, b.time);
    const isPastA = timeA < now;
    const isPastB = timeB < now;

    // 3. Separate tasks that are already past due from those that are upcoming
    if (isPastA && !isPastB) return 1; // isPastA = true, isPastB = false return 1 → a placed after b
    if (!isPastA && isPastB) return -1; // isPastA = false, isPastB = true return -1 → a placed before b

    // 4. Sort by closest date/time to now
    const diff = Math.abs(timeA - now) - Math.abs(timeB - now);

    // 5. If time is same → sort by priority, otherwise return diff
    if (diff !== 0) return diff;
    return order[a.priority] - order[b.priority];
  });
}

// Reset page if filter category/search changed
watch(
  () => [filters.value.search, filters.value.filter],
  () => {
    page.value = 1;
  }
);

// Function for search feature
function searchData(data, search) {
  let keyword = search.toLowerCase();
  return data.filter((data) => {
    if (!data.title.toLowerCase().includes(keyword)) return false;
    return true;
  });
}

// Function for filter by feature and search feature
function filteredData(data, filter) {
  return data.filter((data) => {
    if (!filter) return true;
    if (statusOptions.includes(filter)) return data.status === filter;
    if (priorityOptions.includes(filter)) return data.priority === filter;
    if (categoryOptions.includes(filter)) return data.category === filter;
    return true;
  });
}

// Function Combine all the filters feature
function processData(data) {
  let result = filteredData(data, filters.value.filter);
  result = searchData(result, filters.value.search);
  result = defaultSortBy(result);
  if (!isLoading.value && result.length === 0) {
    noTaskPopMessage();
  }
  return result;
}

// Clean Data after all filters conditions
const cleanData = computed(() => {
  if (!Data.value) return []; // if data is not fetched yet, return blank array
  return processData(Data.value);
});

// Page Init
const page = ref(1);
const pageSize = ref(5);

// Pagination
const pagedData = computed(() => {
  const start = (page.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return cleanData.value.slice(start, end);
});

// If we click next/prev viewport go to top = 0
const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: "smooth", // Animasi halus
  });
};

const popMessage = useMessage();
const noTaskPopMessage = () => {
  popMessage.error("No such tasks");
};
</script>
<style scoped>
.bottom-content {
  margin-bottom: var(--spacing-l);
}
</style>
