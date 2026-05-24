<script setup>
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

function onChange(fieldId, value) {
  emit('update-answer', { fieldId, value })
}

function dropdownTone(fieldId) {
  if (!props.checked) {
    return ''
  }

  const selectedValue = props.answers[fieldId]
  const correctValue = props.question.correct_answers[fieldId]
  return selectedValue === correctValue ? 'select-correct' : 'select-incorrect'
}
</script>

<template>
  <div class="image-question">
    <div class="image-board">
      <img :src="question.image.src" :alt="question.image.alt" class="question-image" />

      <div
        v-for="dropdown in question.dropdowns"
        :key="dropdown.id"
        class="hotspot"
        :style="{
          left: `${dropdown.x * 100}%`,
          top: `${dropdown.y * 100}%`,
          width: `${dropdown.width * 100}%`,
        }"
      >
        <select
          :id="dropdown.id"
          :aria-label="dropdown.label"
          :value="answers[dropdown.id]"
          :class="dropdownTone(dropdown.id)"
          @change="onChange(dropdown.id, $event.target.value)"
        >
          <option value="">Select</option>
          <option v-for="option in dropdown.options" :key="option" :value="option">
            {{ option }}
          </option>
        </select>
        <p
          v-if="checked"
          class="answer-feedback"
          :class="dropdownTone(dropdown.id) === 'select-correct' ? 'is-correct' : 'is-incorrect'"
        >
          <span v-if="dropdownTone(dropdown.id) === 'select-correct'">Correct selection.</span>
          <span v-else>
            Correct: <strong>{{ question.correct_answers[dropdown.id] }}</strong>
          </span>
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.image-question {
  display: grid;
}

.image-board {
  position: relative;
  background: #f4ede2;
  border: 0.0625rem solid #d9cdbb;
  border-radius: 1.375rem;
  padding: 1rem;
  overflow: auto;
}

.question-image {
  display: block;
  width: 100%;
  min-width: 42.5rem;
  border-radius: 1rem;
}

.hotspot {
  position: absolute;
  transform: translate(-50%, -50%);
  display: grid;
  gap: 0.3rem;
  background: rgba(255, 250, 240, 0.95);
  border: 0.0625rem solid rgba(39, 56, 64, 0.18);
  border-radius: 0.875rem;
  padding: 0.55rem;
  box-shadow: 0 0.875rem 1.875rem rgba(24, 44, 54, 0.16);
}

select {
  width: 100%;
  min-width: 7.5rem;
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

.answer-feedback {
  margin: 0;
  font-size: 0.72rem;
}

.answer-feedback.is-correct {
  color: #28553d;
}

.answer-feedback.is-incorrect {
  color: #7a3653;
}

@media (max-width: 60rem) {
  .question-image {
    min-width: 35rem;
  }
}
</style>