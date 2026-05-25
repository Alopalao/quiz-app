<script setup>
import { computed, ref } from 'vue'

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
const naturalSize = ref({ width: 0, height: 0 })

function onChange(fieldId, value) {
  emit('update-answer', { fieldId, value })
}

function onImageLoad(event) {
  const imageElement = event.target
  naturalSize.value = {
    width: imageElement.naturalWidth,
    height: imageElement.naturalHeight,
  }
}

function toPercent(value, axis) {
  if (value === undefined || value === null) {
    return 0
  }

  // Supports normalized values (0..1) or absolute image-pixel values.
  if (value <= 1) {
    return value * 100
  }

  const sourceSize =
    axis === 'y'
      ? props.question.image?.height ?? naturalSize.value.height
      : props.question.image?.width ?? naturalSize.value.width

  if (!sourceSize) {
    return 0
  }

  return (value / sourceSize) * 100
}

function hotspotStyle(dropdown) {
  const style = {
    left: `${toPercent(dropdown.x, 'x')}%`,
    top: `${toPercent(dropdown.y, 'y')}%`,
    width: `${toPercent(dropdown.width, 'x')}%`,
  }

  if (dropdown.height !== undefined && dropdown.height !== null) {
    style.height = `${toPercent(dropdown.height, 'y')}%`
  }

  return style
}

function dropdownHeightStyle(dropdown) {
  if (dropdown.height === undefined || dropdown.height === null) {
    return null
  }

  return { height: '100%' }
}

function dropdownTone(fieldId) {
  if (!props.checked) {
    return ''
  }

  const selectedValue = props.answers[fieldId]
  const correctValue = props.question.correct_answers[fieldId]
  return selectedValue === correctValue ? 'select-correct' : 'select-incorrect'
}

const feedbackRows = computed(() => {
  if (!props.checked) {
    return []
  }

  return props.question.dropdowns.map((dropdown, index) => {
    const selected = props.answers[dropdown.id] ?? ''
    const correct = props.question.correct_answers[dropdown.id]
    const isCorrect = selected === correct

    return {
      id: dropdown.id,
      number: index + 1,
      label: dropdown.label,
      selected,
      correct,
      isCorrect,
    }
  })
})
</script>

<template>
  <div class="image-question">
    <div class="image-board">
      <div class="image-canvas">
        <img
          :src="question.image.src"
          :alt="question.image.alt"
          class="question-image"
          @load="onImageLoad"
        />

        <div
          v-for="(dropdown, index) in question.dropdowns"
          :key="dropdown.id"
          class="hotspot"
          :style="hotspotStyle(dropdown)"
        >
          <div class="dropdown-row">
            <span class="dropdown-number">{{ index + 1 }}</span>
            <select
              :id="dropdown.id"
              :aria-label="dropdown.label"
              :value="answers[dropdown.id]"
              :class="dropdownTone(dropdown.id)"
              :style="dropdownHeightStyle(dropdown)"
              @change="onChange(dropdown.id, $event.target.value)"
            >
              <option value="">Select</option>
              <option v-for="option in dropdown.options" :key="option" :value="option">
                {{ option }}
              </option>
            </select>
          </div>
        </div>
      </div>

      <section v-if="checked" class="feedback-panel">
        <p class="feedback-title">Answer check</p>
        <ul class="feedback-list">
          <li v-for="row in feedbackRows" :key="row.id" :class="row.isCorrect ? 'is-correct' : 'is-incorrect'">
            <span class="feedback-number">{{ row.number }}</span>
            <span v-if="row.isCorrect"> correct.</span>
            <span v-else>
              <strong>{{ row.correct }}</strong>
            </span>
          </li>
        </ul>
      </section>
    </div>
  </div>
</template>

<style scoped>
.image-question {
  display: grid;
}

.image-board {
  display: grid;
  justify-items: center;
  background: #f4ede2;
  border: 0.0625rem solid #d9cdbb;
  border-radius: 1.375rem;
  padding: 1rem;
  overflow: auto;
}

.image-canvas {
  position: relative;
  width: fit-content;
  max-width: 100%;
}

.question-image {
  display: block;
  width: auto;
  max-width: 100%;
  max-height: 65vh;
  height: auto;
  border-radius: 1rem;
}

.hotspot {
  position: absolute;
  transform: translate(-50%, -50%);
  background: transparent;
  border: 0;
  border-radius: 0;
  padding: 0;
  box-shadow: none;
}

.dropdown-row {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  width: 100%;
  height: 100%;
}

.dropdown-number,
.feedback-number {
  display: inline-grid;
  place-items: center;
  width: 1.2rem;
  height: 1.2rem;
  border-radius: 999rem;
  font-size: 0.72rem;
  font-weight: 700;
  line-height: 1;
  background: #1f4f63;
  color: #f1f8fb;
  flex: 0 0 auto;
}

select {
  width: 100%;
  flex: 1 1 auto;
  min-width: 0;
  border: 0.0625rem solid #b7c6cc;
  border-radius: 0.625rem;
  padding: 0.45rem 0.55rem;
  background: #fff;
  color: #112028;
  transition: background-color 0.2s ease, border-color 0.2s ease;
}

select.select-correct {
  background: #e9f9e8;
  border-color: #72b57c;
}

select.select-incorrect {
  background: #ffeaf3;
  border-color: #ce6c94;
}

.feedback-panel {
  margin-top: 0.75rem;
  width: 100%;
  background: #fffdf7;
  border: 0.0625rem solid #dfd4c1;
  border-radius: 0.75rem;
  padding: 0.65rem 0.8rem;
}

.feedback-title {
  margin: 0;
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: #3b4b52;
}

.feedback-list {
  margin: 0.5rem 0 0;
  padding-left: 0;
  list-style: none;
  display: grid;
  gap: 0.2rem;
  font-size: 0.78rem;
}

.feedback-list li {
  display: flex;
  align-items: baseline;
  gap: 0.35rem;
}

.feedback-list li.is-correct {
  color: #28553d;
}

.feedback-list li.is-incorrect {
  color: #7a3653;
}

@media (max-width: 60rem) {
  .question-image {
    max-height: 50vh;
  }
}
</style>