<script setup>
defineProps({
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

function onChange(fieldId, value) {
  emit('update-answer', { fieldId, value })
}

function rowTone(checked, selected, correct) {
  if (!checked) {
    return ''
  }

  return selected === correct ? 'is-correct' : 'is-incorrect'
}
</script>

<template>
  <div class="legend-question">
    <div class="image-wrap">
      <img :src="question.image.src" :alt="question.image.alt" class="question-image" />
    </div>

    <section class="legend-panel">
      <p class="legend-title">Labels</p>
      <div class="legend-list">
        <div
          v-for="legend in question.legends"
          :key="legend.id"
          class="legend-row"
          :class="rowTone(checked, answers[legend.id], question.correct_answers[legend.id])"
        >
          <span class="legend-marker">{{ legend.marker }}</span>
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
    </section>

    <section v-if="checked" class="answer-panel">
      <p class="answer-title">Answer check</p>
      <ul class="answer-list">
        <li v-for="legend in question.legends" :key="`check-${legend.id}`">
          <span class="legend-marker">{{ legend.marker }}</span>
          <strong>{{ question.correct_answers[legend.id] }}</strong>
        </li>
      </ul>
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
  width: auto;
  max-width: 100%;
  max-height: 65vh;
  height: auto;
  border-radius: 1rem;
}

.legend-panel,
.answer-panel {
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

.legend-list {
  margin-top: 0.5rem;
  display: grid;
  gap: 0.45rem;
}

.legend-row {
  display: flex;
  align-items: center;
  gap: 0.45rem;
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
    max-height: 50vh;
  }
}
</style>
