<script setup>
import { computed, ref, watch } from 'vue'

const props = defineProps({
  question: {
    type: Object,
    required: true,
  },
  answers: {
    type: Object,
    required: true,
  },
  checked: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['update-answer'])
const activeHintId = ref('')

function onChange(fieldId, value) {
  emit('update-answer', { fieldId, value })
}

function rowTone(checked, selected, correct) {
  if (!checked) {
    return ''
  }

  return selected === correct ? 'is-correct' : 'is-incorrect'
}

function hasHint(legend) {
  return typeof legend.hint === 'string' && legend.hint.trim().length > 0
}

function toggleHint(legendId) {
  const legend = props.question.legends.find((item) => item.id === legendId)

  if (!legend || !hasHint(legend)) {
    return
  }

  activeHintId.value = activeHintId.value === legendId ? '' : legendId
}

watch(
  () => props.question.id,
  () => {
    activeHintId.value = ''
  },
)

const imageStyle = computed(() => {
  const width = Number(props.question.image?.width)
  const height = Number(props.question.image?.height)

  if (!width || !height) {
    return null
  }

  const ratio = width / height

  return {
    width: `min(100%, calc(72vh * ${ratio}))`,
    aspectRatio: `${width} / ${height}`,
  }
})
</script>

<template>
  <div class="legend-question">
    <div class="image-wrap">
      <img :src="question.image.src" :alt="question.image.alt" class="question-image" :style="imageStyle" />
    </div>

    <section class="legend-panel">
      <div class="legend-columns">
        <div>
          <p class="legend-title">Labels</p>
          <div class="legend-list">
            <div
              v-for="legend in question.legends"
              :key="legend.id"
              class="legend-row"
              :class="rowTone(checked, answers[legend.id], question.correct_answers[legend.id])"
            >
              <button
                type="button"
                class="legend-marker marker-button"
                :class="hasHint(legend) ? 'has-hint' : 'no-hint'"
                :disabled="!hasHint(legend)"
                :aria-label="hasHint(legend) ? `Show hint for marker ${legend.marker}` : `Marker ${legend.marker}`"
                @click="toggleHint(legend.id)"
              >
                {{ legend.marker }}
              </button>
              <div v-if="activeHintId === legend.id && hasHint(legend)" class="hint-bubble">
                {{ legend.hint }}
              </div>
              <select
                :id="legend.id"
                :aria-label="legend.label"
                :value="answers[legend.id]"
                @change="onChange(legend.id, $event.target.value)"
              >
                <option value="">Select</option>
                <option v-for="option in legend.options" :key="option" :value="option">
                  {{ option }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <div class="answers-column">
          <p class="answer-title">Answer check</p>
          <ul class="answer-list">
            <li v-for="legend in question.legends" :key="`check-${legend.id}`">
              <span class="legend-marker">{{ legend.marker }}</span>
              <strong>{{ checked ? question.correct_answers[legend.id] : '—' }}</strong>
            </li>
          </ul>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.legend-question {
  display: grid;
  gap: 0.75rem;
}

.image-wrap {
  background: #f4ede2;
  border: 0.0625rem solid #d9cdbb;
  border-radius: 1.375rem;
  padding: 1rem;
  overflow: auto;
}

.question-image {
  display: block;
  width: 100%;
  max-width: 100%;
  max-height: 72vh;
  height: auto;
  object-fit: contain;
  border-radius: 1rem;
}

.legend-panel {
  width: 100%;
  background: #fffdf7;
  border: 0.0625rem solid #dfd4c1;
  border-radius: 0.75rem;
  padding: 0.65rem 0.8rem;
}

.legend-title,
.answer-title {
  margin: 0;
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: #3b4b52;
}

.legend-columns {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(13rem, 0.8fr);
  gap: 0.9rem;
  align-items: start;
}

.answers-column {
  border-left: 0.0625rem solid #e5dac8;
  padding-left: 0.85rem;
}

.legend-list {
  margin-top: 0.5rem;
  display: grid;
  gap: 0.45rem;
}

.legend-row {
  position: relative;
  display: flex;
  align-items: center;
  gap: 0.45rem;
}

.hint-bubble {
  position: absolute;
  left: 0;
  top: calc(100% + 0.35rem);
  z-index: 30;
  max-width: 14rem;
  background: #f2f8fb;
  border: 0.0625rem solid #c8dde7;
  border-radius: 0.6rem;
  padding: 0.4rem 0.5rem;
  font-size: 0.78rem;
  color: #1b3641;
  box-shadow: 0 0.35rem 0.9rem rgba(23, 49, 62, 0.18);
}

.legend-row.is-correct select {
  background: #e9f9e8;
  border-color: #72b57c;
}

.legend-row.is-incorrect select {
  background: #ffeaf3;
  border-color: #ce6c94;
}

.legend-marker {
  display: inline-grid;
  place-items: center;
  width: 1.25rem;
  height: 1.25rem;
  border-radius: 999rem;
  font-size: 0.72rem;
  font-weight: 700;
  line-height: 1;
  background: #1f4f63;
  color: #f1f8fb;
  flex: 0 0 auto;
}

.marker-button {
  border: 0;
  cursor: pointer;
}

.marker-button.no-hint {
  cursor: default;
}

.marker-button:disabled {
  opacity: 1;
}

.marker-button.has-hint {
  box-shadow: 0 0 0 0.125rem rgba(50, 101, 122, 0.35);
}

select {
  width: 100%;
  min-width: 0;
  border: 0.0625rem solid #b7c6cc;
  border-radius: 0.625rem;
  padding: 0.45rem 0.55rem;
  background: #fff;
  color: #112028;
  transition: background-color 0.2s ease, border-color 0.2s ease;
}

.answer-list {
  margin: 0.5rem 0 0;
  padding-left: 0;
  list-style: none;
  display: grid;
  gap: 0.25rem;
}

.answer-list li {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  color: #2b3c43;
}

@media (max-width: 60rem) {
  .question-image {
    max-height: 58vh;
  }

  .legend-columns {
    grid-template-columns: 1fr;
  }

  .answers-column {
    border-left: 0;
    border-top: 0.0625rem solid #e5dac8;
    padding-left: 0;
    padding-top: 0.75rem;
  }
}
</style>
