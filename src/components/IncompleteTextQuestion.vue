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
</script>

<template>
  <div class="text-question">
    <div class="sentence-card">
      <p class="sentence">
        <template v-for="part in question.parts" :key="part.id ?? part.value">
          <span v-if="part.type === 'text'">{{ part.value }}</span>
          <span v-else class="blank-wrap">
            <select :value="answers[part.id]" @change="onChange(part.id, $event.target.value)">
              <option value="">Select</option>
              <option v-for="option in part.options" :key="option" :value="option">
                {{ option }}
              </option>
            </select>
            <span v-if="checked" class="inline-correct">
              Correct: <strong>{{ question.correct_answers[part.id] }}</strong>
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
  border: 1px solid #e5d5bc;
  border-radius: 22px;
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
  min-width: 150px;
  border: 1px solid #c8beb0;
  border-radius: 10px;
  padding: 0.5rem 0.65rem;
  background: #fff;
  color: #142126;
}

.inline-correct {
  font-size: 0.85rem;
  color: #2a5d43;
}
</style>