<template>

  <h3 class="text-lg font-semibold mb-1 mt-8">DNS Leak Test</h3>


  <div class="bg-gray-900 text-white rounded p-4 mt-2 w-full max-w-2xl">

    <div v-if="loading" class="text-gray-400">Проверка...</div>
    <div v-else-if="error" class="text-red-400">Ошибка: {{ error }}</div>
    <div v-else>
      <div class="text-xs text-white bg-gray-800 rounded p-4 w-full mb-3">
        <div class="font-semibold mb-1">Клиент IP:</div>
        <div v-for="(item, i) in ipBlocks" :key="'ip'+i" class="mb-2">
          <div class="text-white">{{ item.ip }}</div>
          <div class="text-gray-400 text-xs" v-if="item.country_name">
            [{{ item.country_name }}<span v-if="item.asn">, {{ item.asn }}</span>]
          </div>
        </div>
      </div>
      <div class="text-xs text-white bg-gray-800 rounded p-4 w-full mb-3">
        <div class="font-semibold mb-1">DNS-серверы ({{ dnsServers.length }}):</div>
        <div v-for="(dns, i) in dnsServers" :key="'dns'+i">
          <span class="text-white">{{ dns.ip }}</span>
          <span class="text-gray-400 ml-2" v-if="dns.country_name">[{{ dns.country_name }}<span v-if="dns.asn">, {{ dns.asn }}</span>]</span>
        </div>
      </div>
      <div v-if="conclusion && !openbldProtected" class="mt-4 text-xs text-orange-400 bg-gray-800 rounded p-4 w-full">
        <div class="font-semibold mb-1">⚠️ Do-Tel DNS не обнаружены.</div>
<!--        <div class="text-gray-400">Вывод: {{ conclusion.ip }}</div>-->
      </div>
      <div v-if="openbldProtected" class="mt-4 text-xs text-green-400 bg-gray-800 rounded p-4 w-full">
        <div class="font-semibold mb-1">✅ Обнаружен DoTek DoNS</div>
        <div class="text-gray-400">В цепочке DNS-запросов присутствуют защищённые резолверы.</div>
      </div>
      <div v-if="dnsProofValid" class="mt-4 text-xs text-blue-400 bg-gray-800 rounded p-4 w-full">
        <div class="font-semibold mb-1">🔒 DNS подтверждён</div>
        <div class="text-gray-400">Специальные тестовые домены успешно разрешены через ваш DNS. Используется OpenBLD.</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const results = ref([])
const ipBlocks = ref([])
const dnsServers = ref([])
const conclusion = ref(null)
const loading = ref(true)
const error = ref(null)
const openbldProtected = ref(false)
const dnsProofValid = ref(false)

const API_DOMAIN = 'bash.ws'

const knownOpenBLD = [
  '95.216.215.237',
  '109.123.245.41',
  '159.69.37.239',
  '188.245.102.7',
  '2a02:c206:2106:9620::1',
  '2a01:4f8:c2c:4868::1',
  '2a01:4f9:c012:cf0a::1',
  '2a02:c206:2101:5090::1',
  '91.217.10.23',
  '2a00:5da0:1:3002::2c',
  '2606:2040:2800:111::5',
  '2606:2040:2800:111::7'
]

async function fetchText(url) {
  const response = await fetch(url)
  if (!response.ok) throw new Error(`Ошибка запроса ${url}`)
  return await response.text()
}

async function fetchJson(url) {
  const response = await fetch(url)
  if (!response.ok) throw new Error(`Ошибка запроса ${url}`)
  return await response.json()
}

async function performLeakTest() {
  try {
    const id = (await fetchText(`https://${API_DOMAIN}/id`)).trim()

    await Promise.all(
        Array.from({ length: 11 }, (_, i) =>
            fetch(`https://${i}.${id}.${API_DOMAIN}/favicon.ico`, { mode: 'no-cors', cache: 'no-store' }).catch(() => {})
        )
    )

    const result = await fetchJson(`https://${API_DOMAIN}/dnsleak/test/${id}?json`)

    results.value = result
    ipBlocks.value = result.filter(r => r.type === 'ip')
    dnsServers.value = result.filter(r => r.type === 'dns')
    conclusion.value = result.find(r => r.type === 'conclusion')

    // Check OpenBLD match
    openbldProtected.value = dnsServers.value.some(server => knownOpenBLD.includes(server.ip))

    loading.value = false
  } catch (err) {
    error.value = err.message || 'Не удалось выполнить DNS Leak Test'
    loading.value = false
  }
}

onMounted(() => {
  performLeakTest()
})

</script>

<style scoped>
</style>
