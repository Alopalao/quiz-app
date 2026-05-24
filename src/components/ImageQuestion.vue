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
        <label :for="dropdown.id">{{ dropdown.label }}</label>
        <select
          :id="dropdown.id"
          :value="answers[dropdown.id]"
          @change="onChange(dropdown.id, $event.target.value)"
        >
          <option value="">Select</option>
          <option v-for="option in dropdown.options" :key="option" :value="option">
            {{ option }}
          </option>
        </select>
        <p v-if="checked" class="answer-feedback">
          Correct: <strong>{{ question.correct_answers[dropdown.id] }}</strong>
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
  border: 1px solid #d9cdbb;
  border-radius: 22px;
  padding: 1rem;
  overflow: auto;
}

.question-image {
  display: block;
  width: 100%;
  min-width: 680px;
  border-radius: 16px;
}

.hotspot {
  position: absolute;
  transform: translate(-50%, -50%);
  display: grid;
  gap: 0.3rem;
  background: rgba(255, 250, 240, 0.95);
  border: 1px solid rgba(39, 56, 64, 0.18);
  border-radius: 14px;
  padding: 0.55rem;
  box-shadow: 0 14px 30px rgba(24, 44, 54, 0.16);
}

label {
  font-size: 0.72rem;
  font-weight: 700;
  color: #294552;
}

select {
  width: 100%;
  min-width: 120px;
  border: 1px solid #b7c6cc;
  border-radius: 10px;
  padding: 0.45rem 0.55rem;
  background: #fff;
  color: #112028;
}

.answer-feedback {
  margin: 0;
  font-size: 0.72rem;
  color: #28553d;
}

@media (max-width: 960px) {
  .question-image {
    min-width: 560px;
  }
}
</style>