<template>
  <n-config-provider :theme="showNote ? darkTheme : lightTheme">
    <n-card
      :title="props.data.title"
      size="small"
      :segmented="{
        content: true,
      }"
      @click="toggleShowNote"
      style="margin-bottom: var(--spacing-m)"
    >
      <!-- Status -->
      <template #header-extra>
        <n-tag :type="statusTag" size="small">{{ props.data.status }}</n-tag>
      </template>
      <!-- Meta Info -->
      <n-space>
        <!-- Priority -->
        <n-tag size="small" :type="priorityTag">{{
          props.data.priority
        }}</n-tag>
        <!-- Category -->
        <n-tag size="small" :type="categoryTag">{{
          props.data.category
        }}</n-tag>
        <!-- Time -->
        <n-space :size="4">
          <n-icon :component="TimeOutline" />
          <span>{{ props.data.time }}</span>
        </n-space>
        <!-- Date -->
        <n-space :size="4">
          <n-icon :component="CalendarOutline" />
          <span>{{ props.data.date }}</span>
        </n-space>
      </n-space>
      <template #footer>
        <!-- Note -->
        Note :
        <n-collapse-transition :show="showNote">
          <div class="note-content">
            <span>{{ props.data.note }}</span>
          </div>
        </n-collapse-transition>
      </template>
      <template #action>
        <!-- Buttons Section -->
        <div @click.stop class="buttons-section">
          <n-button size="small" @click="handleEdit"> Edit </n-button>
          <n-button size="small" type="error" @click="handleDelete">
            Delete
          </n-button>
          <n-button
            v-show="isDoneVisible"
            size="small"
            type="primary"
            @click="handleMarkDone"
          >
            Mark as Done
          </n-button>
        </div>
      </template>
    </n-card>
  </n-config-provider>
  <n-message-provider>
    <n-modal-provider>
      <TaskForm
        :data="data"
        v-model:show="showTaskForm"
        @close-form="showTaskForm = false"
        @update-tasks="toggleEmitUpdate"
      />
    </n-modal-provider>
  </n-message-provider>
</template>
<script setup>
import TaskForm from "./TaskForm.vue";
import { ref, defineProps, defineEmits, computed } from "vue";

import { TimeOutline, CalendarOutline } from "@vicons/ionicons5";
import { darkTheme, lightTheme, useDialog, useMessage } from "naive-ui";

const props = defineProps({
  data: Object,
});
const emit = defineEmits(["update-tasks"]);
// To update tasks changes if any
const toggleEmitUpdate = () => {
  emit("update-tasks");
};

// To show note and (change theme + collapse transition) if Card is clicked
const showNote = ref(false);
const toggleShowNote = () => {
  showNote.value = !showNote.value;
};

// For different Priority n-tag type
const priorities = [
  { type: "success", label: "Low" },
  { type: "warning", label: "Medium" },
  { type: "error", label: "High" },
];
const priorityTag = priorities.find(
  (priority) => priority.label === props.data.priority
)?.type;

// For different Category n-tag type
const categories = [
  { type: "info", label: "Mind" },
  { type: "success", label: "Soul" },
  { type: "warning", label: "Social" },
  { type: "error", label: "Career" },
];
const categoryTag = categories.find(
  (priority) => priority.label === props.data.category
)?.type;

// // For different Status n-tag type
const statuses = [
  { type: "error", label: "To Do" },
  { type: "info", label: "In Progress" },
  { type: "success", label: "Done" },
];
const statusTag = statuses.find((tag) => tag.label === props.data.status)?.type;

// <--- BUTTONS FUNCTION --->

// -- Edit Button Funtion --
// Show Task Form (edit task)
const showTaskForm = ref(false);
const handleEdit = () => {
  showTaskForm.value = true;
};
// -- Delete Button Function --
const message = useMessage();
const dialog = useDialog();

const dataURL = `http://localhost:4000/tasks`;

const deleteTask = async () => {
  try {
    await fetch(`${dataURL}/${props.data.id}`, {
      method: "DELETE",
    });
    emit("update-tasks");
  } catch (error) {
    message.error("Failed to update task:", error);
  }
};

function handleDelete() {
  dialog.warning({
    title: "Are you sure?",
    content: "Do you want to delete this task?",
    positiveText: "Yes",
    negativeText: "Cancel",
    onPositiveClick: () => {
      deleteTask();
      message.success("Task deleted");
    },
  });
}

// -- Mark as done button Function --
const handleMarkDone = async () => {
  try {
    await fetch(`${dataURL}/${props.data.id}`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ status: "Done" }),
    });
    emit("update-tasks");
  } catch (error) {
    message.error("Failed to update task:", error);
  }
};
const isDoneVisible = computed(() => props.data.status !== "Done");
</script>
<style scoped>
.buttons-section {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}
@media (max-width: 480px) {
  .buttons-section :deep(.n-button) {
    flex: 1 1 auto;
    width: 100%;
  }
}
</style>
