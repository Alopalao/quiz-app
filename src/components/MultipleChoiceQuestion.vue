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

function onChange(value) {
  emit('update-answer', { fieldId: 'selection', value })
}
</script>

<template>
  <div class="choice-question">
    <div class="choice-list">
      <label v-for="option in question.options" :key="option" class="choice-item">
        <input
          type="radio"
          name="multiple-choice"
          :value="option"
          :checked="answers.selection === option"
          @change="onChange(option)"
        />
        <span>{{ option }}</span>
      </label>
    </div>

    <div v-if="checked" class="choice-feedback">
      <p>Your answer: <strong>{{ answers.selection || 'No answer selected' }}</strong></p>
      <p>Correct answer: <strong>{{ question.correct_answers.join(', ') }}</strong></p>
    </div>
  </div>
</template>

<style scoped>
.choice-question {
  display: grid;
  gap: 1rem;
}

.choice-list {
  display: grid;
  gap: 0.85rem;
}

.choice-item {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  background: #fffaf0;
  border: 0.0625rem solid #e7d7bc;
  border-radius: 1.125rem;
  padding: 1rem 1.1rem;
  color: #1a262b;
  cursor: pointer;
}

input {
  width: 1.125rem;
  height: 1.125rem;
  accent-color: #c1661b;
}

.choice-feedback {
  background: #f7efe3;
  border: 0.0625rem solid #e2d0b1;
  border-radius: 1.125rem;
  padding: 1rem 1.1rem;
  color: #1f2a2f;
}

.choice-feedback p {
  margin: 0.2rem 0;
}
</style>