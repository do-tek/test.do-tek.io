<template>
  <div class="flex flex-col items-center space-y-4">
    <div class="relative w-40 h-40">
      <svg class="w-full h-full transform -rotate-90">
        <circle
            class="text-gray-700"
            stroke-width="12"
            stroke="currentColor"
            fill="transparent"
            r="70"
            cx="80"
            cy="80"
        />
        <circle
            class="text-green-400"
            :stroke-dasharray="circumference"
            :stroke-dashoffset="offset"
            stroke-width="12"
            stroke-linecap="round"
            stroke="currentColor"
            fill="transparent"
            r="70"
            cx="80"
            cy="80"
        />
      </svg>
      <div class="absolute inset-0 flex items-center justify-center">
        <span class="text-xl font-semibold">{{ score }}%</span>
      </div>
    </div>
    <p class="text-sm text-gray-400">
      Заблокировано {{ blocked }} из {{ total }} доменов
    </p>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  score: Number,
  blocked: Number,
  total: Number
})

const radius = 70
const circumference = 2 * Math.PI * radius

const offset = computed(() => {
  return circumference - (props.score / 100) * circumference
})
</script>


<style scoped>

</style>
