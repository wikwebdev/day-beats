<script setup lang="ts">
import { defineProps, type PropType, computed } from "vue";
import type { Task } from "@/common/interfaces";
import { getTaskStartTimeText } from "@/common/formatters";

const props = defineProps({
  modelValue: {
    type: Object as PropType<Task>,
    required: true,
  },
});
const text = computed(() => {
  const t = getTaskStartTimeText(props.modelValue);
  return t ? "@" + t : "";
});
</script>
<template>
  <span class="text-info">
    <strong>{{ text }}</strong>
  </span>
</template>
