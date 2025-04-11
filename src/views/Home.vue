// src/views/Home.vue
<template>
  <section class="min-h-screen bg-gray-950 text-white flex flex-col items-center justify-center px-4 py-10">
    <h1 class="text-2xl md:text-4xl font-bold mb-6 text-center">
      Do-Tek DNS
    </h1>

    <ProtectionScore :score="protectionScore" :blocked="blocked" :total="domains.length" />

    <div class="mt-4 text-center text-sm text-gray-400">
      Ваш IP-адрес: <span class="font-mono text-white">{{ userIP }}</span>
    </div>
<!--    <div class="text-center text-sm text-gray-400">-->
<!--      DNS-резолвер: <span class="font-mono text-white">{{ resolver }}</span>-->
<!--    </div>-->
<!--    <div v-if="blockConfirmed" class="mt-2 text-green-400 text-sm font-medium">-->
<!--      ✅ Все тесты пройдены успешно. Вы защищены!-->
<!--    </div>-->

    <div v-if="!isChecking" class="mt-2 text-green-400 text-sm font-medium">

      <div v-if="allBlocked" class="mt-2 text-green-400 text-sm font-medium">
        ✅ Все тесты пройдены успешно. Вы защищены!
      </div>
      <div v-else-if="blockConfirmed" class="mt-2 text-yellow-400 text-sm font-medium">
        ⚠️ Частично защищено: заблокировано {{ blocked }} из {{ domains.length }} доменов
      </div>
      <div v-else class="mt-2 text-red-400 text-sm font-medium">
        ❌ Вероятно используется сторонний DNS. Нет защиты.
      </div>

    </div>

    <div v-if="isChecking && !isWrong" class="mt-2 text-blue-400 text-sm font-medium">
      <svg class="animate-spin inline w-4 h-4 mr-1 text-blue-400" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 00-8 8z"></path></svg> Идёт проверка. Пожалуйста, подождите...
    </div>

    <div v-if="isWrong" class="mt-2 text-red-400 text-sm font-medium">
      ❌ Ошибка получения данных. Что то пошло не так.
    </div>

    <div class="mt-4 text-xs text-gray-500 bg-gray-800 rounded p-4 w-full max-w-2xl">
      <div class="mb-1 font-semibold">Отладка:</div>
      <div>IP клиента: <span class="text-white">{{ userIP }}</span></div>
      <div>Количество заблокированных доменов: <span class="text-white">{{ blocked }}</span></div>
      <div>Подтверждение блокировок: <span :class="blockConfirmed ? 'text-green-400' : 'text-red-400'">{{ blockConfirmed }}</span></div>
    </div>

    <div v-if="locationInfo" class="mt-4 text-xs text-gray-500 bg-gray-800 rounded p-4 w-full max-w-2xl">
      <div class="mb-1 font-semibold">Геолокация по IP:</div>
      <div>Страна: <span class="text-white">{{ locationInfo.country }}</span></div>
      <div>Город: <span class="text-white">{{ locationInfo.city }}</span></div>
      <div>Провайдер: <span class="text-white">{{ locationInfo.org }}</span></div>
    </div>

    <div class="mt-8 w-full max-w-2xl space-y-2">
      <h3 class="text-lg font-semibold mb-2">Тест доменов на блокировку</h3>

      <div
          v-for="(domain, index) in domains"
          :key="index"
          class="text-xs flex justify-between items-center px-4 py-2 rounded border border-gray-800 bg-gray-900"
      >
        <span>{{ domain.name }}</span>
        <span :class="domain.blocked ? 'text-red-400' : 'text-green-400'">
          {{ domain.blocked ? 'Заблокирован 🚫' : 'Доступен ✅' }}
        </span>
      </div>
      <div v-if="blocked === domains.length" class="mt-6 text-green-400 bg-gray-800 text-xs rounded p-4">
        ✅ Все тестовые домены успешно заблокированы. All checks passed.
      </div>
    </div>

    <DnsLeakCheck />

    <footer class="mt-10 text-center text-xs text-gray-500 w-full max-w-2xl">
      Do-Tek LLC © 2024–{{ new Date().getFullYear() }}
    </footer>

  </section>


</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import ProtectionScore from '@/components/ProtectionScore.vue'
import DnsLeakCheck from '@/components/DnsLeakCheck.vue'

const domains = ref([
      { name: 'adclick.example.test', blocked: false },
      { name: 'adclick.g.doubleclick.net', blocked: false },
      { name: 'ads.example.test', blocked: false },
      { name: 'ads.openads.io', blocked: false },
      { name: 'click.example.test', blocked: false },
      { name: 'clickbait.example.test', blocked: false },
      { name: 'popup.example.test', blocked: false },
      { name: 'popup.malvertising.local', blocked: false },
      { name: 'track.example.test', blocked: false },
      { name: 'track.doubleclick.net', blocked: false },
      { name: 'track.openads.io', blocked: false },
      { name: 'tracking.rus.miui.com', blocked: false }
    ]
)

const blocked = ref(0)
const protectionScore = ref(0)
const userIP = ref('определяется...')
const userIPv6 = ref('определяется...')
const isIPv6 = ref(false)
const resolver = ref('определяется...')
const blockConfirmed = ref(false)
const locationInfo = ref(null)
const isChecking = ref(true)
const allBlocked = computed(() => blocked.value === domains.value.length)
const isError = ref(false)
const isWrong = ref(false)

onMounted(async () => {
  isChecking.value = true
  let localBlocked = 0
  blocked.value = 0

  for (const domain of domains.value) {
    try {
      const controller = new AbortController()
      const timeout = setTimeout(() => controller.abort(), 1500)
      await fetch(`https://${domain.name}/favicon.ico`, {
        mode: 'no-cors',
        cache: 'no-store',
        signal: controller.signal,
      })
      domain.blocked = false
      clearTimeout(timeout)
    } catch (err) {
      domain.blocked = true
      localBlocked++
      blocked.value++
    }
  }
  blocked.value = localBlocked
  protectionScore.value = Math.round((blocked.value / domains.value.length) * 100)

  blockConfirmed.value = blocked.value >= 3

  try {
    const ipRes = await fetch('https://api.ipify.org?format=json')
    const ipData = await ipRes.json()
    userIP.value = ipData.ip

    const ipv6Res = await fetch('https://api6.ipify.org?format=json')
    const ipv6Data = await ipv6Res.json()
    userIPv6.value = ipv6Data.ip
    if (ipData.ip !== ipv6Data.ip) {
      isIPv6.value = true
    }


    try {
      const geoRes = await fetch(`https://ipapi.co/${ipData.ip}/json/`)
      const geoData = await geoRes.json()
      locationInfo.value = {
        country: geoData.country_name,
        city: geoData.city,
        org: geoData.org
      }

    } catch (geoErr) {
      console.error('Ошибка получения геолокации:', geoErr)
      isError.value = true
    }

  } catch (e) {
    userIP.value = 'ошибка получения'
    isError.value = true
  }

  isChecking.value = !!isError.value;

  if (isError.value) {
    setTimeout(() => {
      isWrong.value = true
    }, 5000)
  }

})

</script>

<style scoped>
</style>
