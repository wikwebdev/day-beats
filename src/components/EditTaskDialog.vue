<script setup lang="ts">
import { ref, computed, watch, defineEmits, type PropType } from "vue";
import type { Task } from "@/common/interfaces";
import { useTaskStore } from "@/stores/task";
import { useSegmentStore } from "@/stores/segment";
import TaskListItem from "@/components/TaskListItem.vue";
import TaskComboField from "@/components/fields/TaskComboField.vue";
import StartTimeField from "@/components/fields/StartTimeField.vue";
import { v4 as uuidV4 } from "uuid";

const emits = defineEmits(["update:modelValue"]);
const taskStore = useTaskStore();
const segmentStore = useSegmentStore();

const props = defineProps({
  modelValue: {
    type: Boolean,
    required: true,
  },
  task: {
    type: Object as PropType<Task | null>,
    default: null,
    required: false,
  },
});

const description = ref("");
const duration = ref(0);
const startTime = ref<number | undefined>(undefined);
const segmentId = ref<String | undefined>(undefined);

watch(
  () => props.modelValue,
  (opened) => {
    if (opened) {
      if (props.task) {
        description.value = String(props.task.description);
        duration.value = props.task.duration || 0;
        startTime.value = props.task.startTime;
        segmentId.value = props.task.segmentId;
      } else {
        description.value = "";
        duration.value = 0;
        startTime.value = undefined;
        segmentId.value = segmentStore.currentSegment?.id;
      }
    }
  },
  { immediate: true }
);

const updatedTask = computed(() => {
  return Object.assign({}, props.task, {
    description: description.value,
    duration: duration.value,
    startTime: startTime.value,
    segmentId: segmentId.value,
  });
});

const isExisting = computed(() => props.task && props.task.id);

function saveTask() {
  if (description.value !== "") {
    if (updatedTask.value && updatedTask.value.id) {
      taskStore.addChanges("UPDATE", updatedTask.value);
    } else {
      taskStore.addChanges(
        "INSERT",
        Object.assign({}, updatedTask.value, { id: uuidV4() })
      );
    }
    emits("update:modelValue", false);
  }
}

function deleteTask() {
  if (props.task && props.task.id) {
    taskStore.addChanges("DELETE", props.task);
    emits("update:modelValue", false);
  }
}
</script>

<template>
  <v-dialog
    :model-value="modelValue"
    @update:modelValue="$emit('update:modelValue', $event)"
    fullscreen
    :scrim="false"
    transition="slide-x-transition"
  >
    <v-card>
      <v-toolbar density="compact" color="primary">
        <v-btn icon dark @click="$emit('update:modelValue', false)">
          <v-icon>mdi-arrow-left</v-icon>
        </v-btn>
        <v-toolbar-title>Edit Task</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-toolbar-items>
          <v-btn variant="text" @click="saveTask">
            {{ isExisting ? "save" : "create" }}
          </v-btn>
        </v-toolbar-items>
      </v-toolbar>
      <v-card-text>
        <v-container>
          <v-form @submit.prevent="saveTask">
            <h2 class="text-h5">Basic Fields</h2>
            <TaskComboField
              v-model:description="description"
              v-model:duration="duration"
            />
            <div class="mt-8" />
            <h2 class="text-h5">Segment</h2>
            <v-radio-group inline v-model="segmentId">
              <v-radio
                v-for="(segment, index) in segmentStore.segments"
                :key="index"
                :label="segment.description.toString()"
                :value="segment.id"
              >
              </v-radio>
            </v-radio-group>
            <div class="mt-8" />
            <h2 class="text-h5">Start Time</h2>
            <StartTimeField v-model:modelValue="startTime" />
            <div class="mt-8" />
            <h2 class="text-h5">Preview</h2>
            <TaskListItem
              class="elevation-1"
              :modelValue="updatedTask"
              readonly
            />
            <v-card class="mt-8" variant="flat">
              <v-card-actions>
                <v-btn
                  v-if="isExisting"
                  @click="deleteTask"
                  color="error"
                  variant="text"
                  >Delete</v-btn
                >
                <v-spacer />
                <v-btn @click="$emit('update:modelValue', false)" variant="text"
                  >cancel</v-btn
                >
                <v-btn @click="saveTask" color="primary" variant="flat">
                  {{ isExisting ? "save" : "create" }}
                </v-btn>
              </v-card-actions>
            </v-card>
            <button type="submit" class="screen-reader-text">save</button>
          </v-form>
        </v-container>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>
