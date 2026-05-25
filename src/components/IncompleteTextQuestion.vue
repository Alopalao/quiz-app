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

function blankTone(fieldId) {
  if (!props.checked) {
    return ''
  }

  const selectedValue = props.answers[fieldId]
  const correctValue = props.question.correct_answers[fieldId]
  return selectedValue === correctValue ? 'select-correct' : 'select-incorrect'
}
</script>

<template>
  <div class="text-question">
    <div class="sentence-card">
      <p class="sentence">
        <template v-for="part in question.parts" :key="part.id ?? part.value">
          <span v-if="part.type === 'text'">{{ part.value }}</span>
          <span v-else class="blank-wrap">
            <select
              :value="answers[part.id]"
              :class="blankTone(part.id)"
              @change="onChange(part.id, $event.target.value)"
            >
              <option value="">Select</option>
              <option v-for="option in part.options" :key="option" :value="option">
                {{ option }}
              </option>
            </select>
            <span
              v-if="checked"
              class="inline-feedback"
              :class="blankTone(part.id) === 'select-correct' ? 'is-correct' : 'is-incorrect'"
            >
              <span v-if="blankTone(part.id) !== 'select-correct'">
                <strong>{{ question.correct_answers[part.id] }}</strong>
              </span>
            </span>
          </span>
        </template>
      </p>
    </div>
  </div>
</template>

<style scoped>
.sentence-card {
  background: linear-gradient(135deg, #fff9ef 0%, #f3ece3 100%);
  border: 0.0625rem solid #e5d5bc;
  border-radius: 1.375rem;
  padding: 1.5rem;
}

.sentence {
  margin: 0;
  font-size: 1.1rem;
  line-height: 2.3;
  color: #1a262b;
}

.blank-wrap {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  margin: 0 0.2rem;
  flex-wrap: wrap;
}

select {
  min-width: 9.375rem;
  border: 0.0625rem solid #c8beb0;
  border-radius: 0.625rem;
  padding: 0.5rem 0.65rem;
  background: #fff;
  color: #142126;
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

.inline-feedback {
  font-size: 0.85rem;
}

.inline-feedback.is-correct {
  color: #2a5d43;
}

.inline-feedback.is-incorrect {
  color: #7a3653;
}
</style>