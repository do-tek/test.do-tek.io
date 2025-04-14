// src/views/Home.vue
<template>
  <section class="min-h-screen bg-gray-950 text-white flex flex-col items-center justify-center px-4 py-10">
    <h1 class="text-2xl md:text-4xl font-bold mb-6 text-center">
      Do-Tek DNS
    </h1>

    <ProtectionScore :score="protectionScore" :blocked="blocked" :total="domains.length" />

    <!-- IPv4 -->
    <div class="mt-4 text-center text-sm text-gray-400">
      Ваш IPv4-адрес:
      <span class="font-mono text-white">{{ userIP }}</span>
      <button class="ml-2 text-xs text-blue-400 hover:underline" @click="copyToClipboard(userIP, 'ip')">копировать</button>
      <transition name="fade">
        <span v-if="showCopied === 'ip'" class="ml-2 text-green-400 text-xs flex items-center">
          <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
          </svg> Скопировано
        </span>
      </transition>
    </div>

    <!-- IPv6 -->
    <div v-if="isIPv6" class="mt-4 text-center text-sm text-gray-400">
      IPv6-адрес:
      <span class="font-mono text-white">{{ userIPv6 }}</span>
      <button class="ml-2 text-xs text-blue-400 hover:underline" @click="copyToClipboard(userIPv6, 'ipv6')">копировать</button>
      <transition name="fade">
        <span v-if="showCopied === 'ipv6'" class="ml-2 text-green-400 text-xs flex items-center">
          <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
          </svg> Скопировано
        </span>
      </transition>
    </div>

    <!-- Состояние проверки -->
    <div v-if="isChecking && !isWrong" class="mt-2 text-blue-400 text-sm font-medium">
      <svg class="animate-spin inline w-4 h-4 mr-1" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 00-8 8z" />
      </svg>
      Идёт проверка. Пожалуйста, подождите...
    </div>

    <!-- Результат защиты -->
    <div v-if="!isChecking" class="mt-2 text-green-400 text-sm font-medium">
      <div v-if="allBlocked" class="mt-2 text-green-400 text-sm font-medium">
        ✅ Тесты блокировки пройдены успешно!
      </div>
      <div v-else-if="blockConfirmed" class="mt-2 text-yellow-400 text-sm font-medium">
        ⚠️ Заблокировано {{ blocked }} из {{ domains.length }} доменов
      </div>
      <div v-else class="mt-2 text-red-400 text-sm font-medium">
        ❌ Вероятно в цепочке присутствует сторонний DNS.
      </div>
    </div>

    <!-- Отладка -->
    <div class="mt-4 text-xs text-gray-500 bg-gray-800 rounded p-4 w-full max-w-2xl">
      <div class="mb-1 font-semibold">Отладка:</div>
      <div>IP клиента: <span class="text-white">{{ userIP }}</span></div>
      <div>Количество заблокированных доменов: <span class="text-white">{{ blocked }}</span></div>
      <div>Подтверждение блокировок: <span :class="blockConfirmed ? 'text-green-400' : 'text-red-400'">{{ blockConfirmed }}</span></div>
    </div>

    <!-- Гео -->
<!--    <div v-if="locationInfo && !isWrong" class="mt-4 text-xs text-gray-500 bg-gray-800 rounded p-4 w-full max-w-2xl">-->
<!--      <div class="mb-1 font-semibold">Геолокация по IP:</div>-->
<!--      <div>Страна: <span class="text-white">{{ locationInfo.country }}</span></div>-->
<!--      <div>Город: <span class="text-white">{{ locationInfo.city }}</span></div>-->
<!--      <div>Провайдер: <span class="text-white">{{ locationInfo.org }}</span></div>-->
<!--    </div>-->
    <div v-if="locationInfo && !isWrong" class="mt-4 text-xs text-gray-500 bg-gray-800 rounded p-4 w-full max-w-2xl">
      <div class="mb-1 font-semibold">Геолокация по IP:</div>
      <div class="flex items-center gap-2">
        Страна:
        <img v-if="locationInfo.flag" :src="locationInfo.flag" alt="флаг" class="w-5 h-4 inline-block rounded-sm border" />
        <span class="text-white">{{ locationInfo.country }}</span>
      </div>
      <div>Город: <span class="text-white">{{ locationInfo.city }}</span></div>
      <div>Провайдер: <span class="text-white">{{ locationInfo.org }}</span></div>
    </div>


    <!-- Тесты блокировки -->
    <div class="mt-8 w-full max-w-2xl space-y-2">
      <h3 class="text-lg font-semibold mb-2">Тесты блокировки</h3>
      <div v-for="domain in domains" :key="domain.name" class="text-xs flex justify-between items-center px-4 py-2 rounded border border-gray-800 bg-gray-900">
        <span>{{ domain.name }}</span>
        <span :class="domain.blocked ? 'text-green-400' : 'text-red-400'">
          {{ domain.blocked ? 'Заблокирован ✅' : 'Доступен 🚫' }}
        </span>
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
      { name: 'blockcheck1.system-checks.org', blocked: false },
      { name: 'blockcheck2.system-checks.org', blocked: false },
    ]
)

const blocked = ref(0)
const protectionScore = ref(0)
const userIP = ref('определяется...')
const userIPv6 = ref('определяется...')
const isIPv6 = ref(false)
const ipData = ref('определяется...')
const blockConfirmed = ref(false)
const locationInfo = ref(null)
const isChecking = ref(true)
const allBlocked = computed(() => blocked.value === domains.value.length)
const isError = ref(false)
const isWrong = ref(false)

const showCopied = ref(null)

function copyToClipboard(value, target = 'ip') {
  if (!value) return
  navigator.clipboard.writeText(value).then(() => {
    showCopied.value = target
    setTimeout(() => (showCopied.value = null), 1500)
  }).catch(err => {
    console.warn('Ошибка копирования:', err)
  })
}

function isAllBlocked() {
  return blocked.value === domains.value.length
}

function checkDomainsWithImgFallback() {
  isChecking.value = true
  blocked.value = 0

  domains.value.forEach((domain, index) => {
    const img = new Image()
    img.onload = () => {
      domain.blocked = false
      finalize(index)
    }
    img.onerror = () => {
      domain.blocked = true
      blocked.value++
      finalize(index)
    }
    img.src = `https://${domain.name}/favicon.ico?_=${Date.now()}`
  })
}

function finalize(index) {
  protectionScore.value = Math.round((blocked.value / domains.value.length) * 100)
  blockConfirmed.value = blocked.value >= 2

  if (index === domains.value.length - 1) {
    isChecking.value = false
  }
}


onMounted(async () => {
  isChecking.value = true
  let localBlocked = 0
  blocked.value = 0

  checkDomainsWithImgFallback()

  try {

    try {
      const ipRes = await fetch('https://api.ipify.org?format=json')
      const ipData = await ipRes.json()
      userIP.value = ipData.ip
    } catch (e) {
      userIP.value = 'ошибка получения'
      isError.value = true
    }

    try {
      const ipv6Res = await fetch('https://api6.ipify.org?format=json')
      const ipv6Data = await ipv6Res.json()
      userIPv6.value = ipv6Data.ip
      if (userIPv6.value.length > 0) {
        isIPv6.value = true
        console.log(userIPv6.value)
      }

    } catch (geoErr) {
      console.error('Ошибка получения IPv6:', geoErr)
    }

    // try {
    //   const geoRes = await fetch(`https://ipapi.co/${userIP.value}/json/`)
    //   //const geoRes = await fetch(`https://ip-api.com/json/${userIP.value}`)
    //   const geoData = await geoRes.json()
    //   // console.log(geoData)
    //   locationInfo.value = {
    //     country: geoData.country,
    //     city: geoData.city,
    //     org: geoData.org
    //   }
    //
    // } catch (geoErr) {
    //   console.error('Ошибка получения геолокации:', geoErr)
    //   // isError.value = true
    // }

    try {
      const geoRes = await fetch(`https://ipwho.is/${userIP.value}`)
      const geoData = await geoRes.json()

      if (geoData.success) {
        locationInfo.value = {
          country: geoData.country,
          city: geoData.city,
          org: geoData.connection?.isp || 'n/a',
          flag: geoData.flag?.img || '',
          emoji: geoData.flag?.emoji || ''
        }
      }
    } catch (geoErr) {
      console.error('Ошибка получения геолокации:', geoErr)
    }


  } catch (e) {
    userIP.value = 'ошибка получения'
    // isError.value = true
  }

  isChecking.value = !!isError.value;

  if (isError.value) {
    setTimeout(() => {
      isWrong.value = true
    }, 3000)
  }

})

</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
