<script setup>
import { computed, ref } from 'vue'
import tagPool from '../data/tagPool.json'

const presetMap = {
  全维度: ['scene', 'emotion', 'relationship', 'action', 'limit', 'intensity'],
  情绪流: ['scene', 'emotion', 'relationship', 'intensity'],
  行动流: ['scene', 'action', 'limit', 'intensity'],
}

const currentPreset = ref('全维度')
const result = ref({})
const copied = ref(false)

const activeKeys = computed(() => {
  return presetMap[currentPreset.value] || Object.keys(tagPool.dimensions)
})

function pickRandom(list) {
  return list[Math.floor(Math.random() * list.length)]
}

function roll() {
  const next = {}

  activeKeys.value.forEach((key) => {
    const group = tagPool.dimensions[key]

    if (group && Array.isArray(group.tags) && group.tags.length > 0) {
      next[key] = {
        label: group.label,
        tag: pickRandom(group.tags),
      }
    }
  })

  result.value = next
  copied.value = false
}

const resultText = computed(() => {
  return Object.values(result.value)
    .map((item) => `${item.label}：${item.tag}`)
    .join('\n')
})

async function copyResult() {
  if (!resultText.value) return

  await navigator.clipboard.writeText(resultText.value)
  copied.value = true

  setTimeout(() => {
    copied.value = false
  }, 1200)
}

roll()
</script>

<template>
  <section class="dice">
    <div class="toolbar">
      <label class="preset">
        Preset
        <select v-model="currentPreset" @change="roll">
          <option v-for="(_, name) in presetMap" :key="name" :value="name">
            {{ name }}
          </option>
        </select>
      </label>

      <button @click="roll">重新 Roll</button>
      <button :disabled="!resultText" @click="copyResult">
        {{ copied ? '已复制' : '复制结果' }}
      </button>
    </div>

    <div v-if="Object.keys(result).length" class="card">
      <div v-for="(item, key) in result" :key="key" class="tag-row">
        <span class="label">{{ item.label }}</span>
        <span class="tag">{{ item.tag }}</span>
      </div>
    </div>

    <p v-else class="empty">
      还没有读到可用 tag。检查 tagPool.json 格式。
    </p>
  </section>
</template>

<style scoped>
.dice {
  margin-top: 24px;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: center;
  margin-bottom: 20px;
}

.preset {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
}

select,
button {
  border: 1px solid #d1d5db;
  border-radius: 12px;
  background: white;
  padding: 10px 14px;
  font-size: 15px;
}

button {
  cursor: pointer;
}

button:hover {
  transform: translateY(-1px);
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.45;
}

.card {
  border: 1px solid #e5e7eb;
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.06);
}

.tag-row {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  padding: 12px 0;
  border-bottom: 1px solid #f1f5f9;
}

.tag-row:last-child {
  border-bottom: none;
}

.label {
  min-width: 90px;
  font-weight: 700;
  color: #4b5563;
}

.tag {
  text-align: right;
  font-weight: 600;
}

.empty {
  color: #6b7280;
}
</style>
