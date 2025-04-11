<template>
  <div class="flex flex-col items-center space-y-4">
    <div class="relative w-40 h-40">
      <svg class="w-full h-full transform -rotate-90">
        <!-- фон круга -->
        <circle
            class="text-gray-700"
            stroke-width="12"
            stroke="currentColor"
            fill="transparent"
            r="70"
            cx="80"
            cy="80"
        />
        <!-- динамический круг -->
        <circle
            class="text-green-400 transition-all duration-500"
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
      <!-- надпись по центру -->
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

// Получаем реактивные свойства
const props = defineProps({
  blocked: Number,
  total: Number
})

// Настройки круга
const radius = 70
const circumference = 2 * Math.PI * radius

// Вычисляем процент блокировки
const score = computed(() => {
  return props.total > 0
      ? Math.round((props.blocked / props.total) * 100)
      : 0
})

// Смещение круга по мере заполнения
const offset = computed(() => {
  return circumference - (score.value / 100) * circumference
})
</script>

<style scoped>
/* ничего не нужно, всё через Tailwind */
</style>
