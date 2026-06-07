<template>
  <section class="ao3-dice">
    <header class="header">
      <h1>AO3 Tag Dice</h1>
      <p>按 preset 过滤，按维度随机抽取。只需维护 JSON 即可扩展。</p>
    </header>

    <div class="controls">
      <label class="preset-select">
        <span>模式</span>
        <select v-model="activePresetKey" @change="rollAll">
          <option v-for="preset in presets" :key="preset.key" :value="preset.key">
            {{ preset.name }}
          </option>
        </select>
      </label>
      <p class="preset-description">{{ activePreset?.description }}</p>
      <div class="actions">
        <button type="button" @click="rollAll">整体 Roll</button>
        <button type="button" @click="copyResult" :disabled="!resultLines.length">
          {{ copyStatus }}
        </button>
      </div>
    </div>

    <ul class="result-list">
      <li v-for="dimension in dimensions" :key="dimension.key" class="result-card">
        <div class="card-header">
          <strong>{{ dimension.name }}</strong>
          <button type="button" @click="rerollDimension(dimension.key)">重 Roll</button>
        </div>

        <div v-if="pickedTagsByDimension[dimension.key]?.length" class="card-body">
          <div
            v-for="tag in pickedTagsByDimension[dimension.key]"
            :key="`${dimension.key}-${tag.ao3Tag}`"
            class="tag-line"
          >
            <span class="label">{{ tag.label }}</span>
            <code>{{ tag.ao3Tag }}</code>
          </div>
        </div>
        <p v-else class="empty">当前模式下该维度没有可用 tag</p>
      </li>
    </ul>
  </section>
</template>

<script setup>
import { computed, ref } from 'vue'
import tagPool from '../data/tagPool.json'

const presets = tagPool.presets ?? []
const dimensions = tagPool.dimensions ?? []

const activePresetKey = ref(presets[0]?.key ?? 'free')
const pickedTagsByDimension = ref({})
const copyStatus = ref('复制结果')

const activePreset = computed(() => presets.find((preset) => preset.key === activePresetKey.value))

const pickRandom = (tags, count = 1) => {
  if (!Array.isArray(tags) || tags.length === 0) return []
  const pool = [...tags]
  const max = Math.min(Math.max(count, 1), pool.length)
  const result = []

  for (let i = 0; i < max; i += 1) {
    const index = Math.floor(Math.random() * pool.length)
    result.push(pool[index])
    pool.splice(index, 1)
  }

  return result
}

const getFilteredTags = (dimension) => {
  const tags = Array.isArray(dimension.tags) ? dimension.tags : []
  if (activePresetKey.value === 'free') return tags

  return tags.filter((tag) => Array.isArray(tag.presets) && tag.presets.includes(activePresetKey.value))
}

const rollDimension = (dimension) => {
  const filteredTags = getFilteredTags(dimension)
  return pickRandom(filteredTags, Number(dimension.count) || 1)
}

const rollAll = () => {
  const next = {}
  dimensions.forEach((dimension) => {
    next[dimension.key] = rollDimension(dimension)
  })
  pickedTagsByDimension.value = next
  copyStatus.value = '复制结果'
}

const rerollDimension = (dimensionKey) => {
  const dimension = dimensions.find((item) => item.key === dimensionKey)
  if (!dimension) return

  pickedTagsByDimension.value = {
    ...pickedTagsByDimension.value,
    [dimensionKey]: rollDimension(dimension)
  }
  copyStatus.value = '复制结果'
}

const resultLines = computed(() => {
  return dimensions
    .flatMap((dimension) => {
      const tags = pickedTagsByDimension.value[dimension.key] ?? []
      return tags.map((tag) => `${dimension.name}: ${tag.label} | ${tag.ao3Tag}`)
    })
    .filter(Boolean)
})

const copyResult = async () => {
  if (!resultLines.value.length) return

  try {
    await navigator.clipboard.writeText(resultLines.value.join('\n'))
    copyStatus.value = '已复制'
  } catch {
    copyStatus.value = '复制失败'
  }
}

rollAll()
</script>

<style scoped>
.ao3-dice {
  max-width: 860px;
  margin: 24px auto;
  padding: 20px;
  font-family: Inter, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  color: #1f2937;
}

.header h1 {
  margin: 0;
}

.header p {
  margin-top: 8px;
  color: #4b5563;
}

.controls {
  margin-top: 18px;
  padding: 14px;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  background: #f9fafb;
}

.preset-select {
  display: grid;
  gap: 8px;
  font-weight: 600;
}

.preset-select select {
  max-width: 220px;
  padding: 8px 10px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background: #fff;
}

.preset-description {
  margin: 10px 0 0;
  color: #6b7280;
}

.actions {
  margin-top: 12px;
  display: flex;
  gap: 10px;
}

button {
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 8px;
  padding: 8px 12px;
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.result-list {
  margin: 16px 0 0;
  padding: 0;
  list-style: none;
  display: grid;
  gap: 12px;
}

.result-card {
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  padding: 12px;
  background: #fff;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}

.card-body {
  margin-top: 10px;
  display: grid;
  gap: 8px;
}

.tag-line {
  display: grid;
  gap: 3px;
}

.label {
  font-weight: 600;
}

code {
  background: #f3f4f6;
  border-radius: 6px;
  padding: 4px 6px;
  width: fit-content;
}

.empty {
  margin: 10px 0 0;
  color: #9ca3af;
}
</style>
