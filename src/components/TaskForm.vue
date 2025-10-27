<template>
  <n-modal
    class="custom-card"
    preset="card"
    :style="bodyStyle"
    :bordered="false"
    :segmented="segmented"
    @after-leave="closeForm"
  >
    <template #header>
      {{ props.data.id ? "Modify Task" : "Create New Task" }}
    </template>
    <!-- Content -->
    <n-form
      ref="formRef"
      :model="localData"
      :rules="rules"
      :show-label="false"
      :size="'medium'"
    >
      <!-- Task Title -->
      <n-form-item path="title">
        <n-input
          v-model:value="localData.title"
          placeholder="New Task Title"
          size="large"
        />
      </n-form-item>
      <n-grid :cols="3" :x-gap="12">
        <n-form-item-gi path="status">
          <!-- Status -->
          <n-select
            v-model:value="localData.status"
            class="custom-select"
            placeholder="Status"
            :render-label="renderLabel"
            :options="statusOptions"
            clearable
          />
        </n-form-item-gi>
        <n-form-item-gi path="priority">
          <!-- Priority -->
          <n-select
            v-model:value="localData.priority"
            placeholder="Priority"
            :render-label="renderLabel"
            :options="priorityOptions"
            clearable
          />
        </n-form-item-gi>
        <n-form-item-gi path="category">
          <!-- Category -->
          <n-select
            v-model:value="localData.category"
            placeholder="Task's Category"
            :render-label="renderLabel"
            :options="categoryOptions"
            clearable
          />
        </n-form-item-gi>
      </n-grid>
      <n-grid :cols="2" :x-gap="12">
        <n-form-item-gi path="time">
          <!-- Time -->
          <n-time-picker
            v-model:formatted-value="localData.time"
            format="HH:mm"
            value-format="HH:mm"
            placeholder="Select Time"
            style="width: 100%"
          />
        </n-form-item-gi>
        <n-form-item-gi path="date">
          <!-- Date -->
          <n-date-picker
            v-model:formatted-value="localData.date"
            type="date"
            value-format="dd-MM-yyyy"
            placeholder="Select Date"
            style="width: 100%"
          />
        </n-form-item-gi>
      </n-grid>
    </n-form>

    <!-- Footer -->
    <template #footer>
      <n-space vertical :size="12">
        Note :
        <n-input
          v-model:value="localData.note"
          type="textarea"
          placeholder="Add Note..."
        />
      </n-space>
    </template>

    <!-- Action -->
    <template #action>
      <n-space justify="center">
        <n-button type="primary" @click="handleValidate">Confirm</n-button>
        <n-button @click="closeForm">Cancel</n-button>
      </n-space>
    </template>
  </n-modal>
</template>
<script setup>
import { ref, reactive, h, defineProps, defineEmits } from "vue";

import { NTag, useMessage } from "naive-ui";

const bodyStyle = {
  width: "600px",
};
const segmented = {
  content: true,
};

const props = defineProps({
  data: {
    type: Object,
    default: () => ({
      title: "",
      status: null,
      priority: null,
      category: null,
      time: null,
      date: null,
      note: "",
    }),
  },
});

// Database URL
const databaseURL = "http://localhost:4000/tasks";

// Init emit
const emit = defineEmits(["close-form", "update-tasks"]);

// Initialize ref for form component itself
const formRef = ref(null);

// Local State Data
const localData = reactive({ ...props.data });

// Form Rules
const rules = {
  title: {
    required: true,
    trigger: ["blur", "input"],
    message: "Please input Title",
  },
  status: {
    required: true,
    trigger: ["blur", "change"],
    message: "Please select Status",
  },
  priority: {
    required: true,
    trigger: ["blur", "change"],
    message: "Please select Priority",
  },
  category: {
    required: true,
    trigger: ["blur", "change"],
    message: "Please select Category",
  },
  time: {
    required: true,
    trigger: ["blur", "change"],
    message: "Please input Time",
  },
  date: {
    required: true,
    trigger: ["blur", "change"],
    message: "Please input Date",
  },
};

const statusOptions = [
  { label: "To Do", value: "To Do", type: "error" },
  { label: "In Progress", value: "In Progress", type: "info" },
  { label: "Done", value: "Done", type: "success" },
];

const priorityOptions = [
  { label: "Low", value: "Low", type: "success" },
  { label: "Medium", value: "Medium", type: "warning" },
  { label: "High", value: "High", type: "error" },
];

const categoryOptions = [
  { label: "Mind", value: "Mind", type: "info" },
  { label: "Soul", value: "Soul", type: "success" },
  { label: "Social", value: "Social", type: "warning" },
  { label: "Career", value: "Career", type: "error" },
];

const renderLabel = (option) => {
  return h(NTag, { type: option.type }, { default: () => option.label });
};

// Upload and Update API
const saveTask = async () => {
  if (localData.id) {
    await fetch(`${databaseURL}/${localData.id}`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(localData),
    });
  } else {
    await fetch(`${databaseURL}`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(localData),
    });
  }
};

// Function for Confirm Button
const popMessage = useMessage();
const handleValidate = async (e) => {
  e.preventDefault();
  try {
    // (?) is to check if formRef.value whether null // undefined or not
    // if not then validate
    await formRef.value?.validate();
    await saveTask();
    emit("update-tasks");
    popMessage.success("Task has been saved");
    closeForm();
  } catch (err) {
    popMessage.error("Invalid Input");
  }
};

// Function for Cancel Button, emit to close form
const closeForm = () => {
  emit("close-form");
  Object.assign(localData, props.data);
};
</script>
