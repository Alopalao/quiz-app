<script setup>
import { computed, reactive, ref } from 'vue'
import ImageQuestion from './components/ImageQuestion.vue'
import IncompleteTextQuestion from './components/IncompleteTextQuestion.vue'
import MultipleChoiceQuestion from './components/MultipleChoiceQuestion.vue'
import QuestionSidebar from './components/QuestionSidebar.vue'
import questionsData from '../public/questions.json'

const imageModules = import.meta.glob('../public/images/*', { eager: true, import: 'default' })

const questions = ref([])
const currentIndex = ref(0)
const answers = reactive({})
const checked = reactive({})
const loading = ref(false)
const errorMessage = ref('')

const currentQuestion = computed(() => questions.value[currentIndex.value] ?? null)

const stateByQuestion = computed(() => {
  const states = {}

  for (const question of questions.value) {
    const questionAnswers = answers[question.id] ?? {}
    const fieldIds = getFieldIds(question)
    const filledCount = fieldIds.filter((fieldId) => isFilled(questionAnswers[fieldId])).length
    const allFilled = fieldIds.length > 0 && filledCount === fieldIds.length
    const hasAnyAnswer = filledCount > 0

    let label = 'Not started'
    let tone = 'status-not-started'

    if (hasAnyAnswer && !allFilled) {
      label = 'Incomplete'
      tone = 'status-incomplete'
    } else if (allFilled && !checked[question.id]) {
      label = 'Answered'
      tone = 'status-complete'
    } else if (allFilled && checked[question.id]) {
      if (isQuestionCorrect(question)) {
        label = 'Correct'
        tone = 'status-correct'
      } else {
        label = 'Checked'
        tone = 'status-checked'
      }
    }

    states[question.id] = { label, tone }
  }

  return states
})

initializeQuestions()

function createEmptyAnswer(question) {
  if (question.type === 'image') {
    return Object.fromEntries(question.dropdowns.map((dropdown) => [dropdown.id, '']))
  }

  if (question.type === 'incomplete_text') {
    return Object.fromEntries(
      question.parts.filter((part) => part.type === 'blank').map((part) => [part.id, '']),
    )
  }

  if (question.type === 'multiple_choice') {
    return { selection: '' }
  }

  return {}
}

function initializeQuestions() {
  try {
    const rawQuestions = Array.isArray(questionsData.questions) ? questionsData.questions : []
    questions.value = rawQuestions.map((question) => normalizeQuestion(question))

    for (const question of questions.value) {
      answers[question.id] = createEmptyAnswer(question)
      checked[question.id] = false
    }
  } catch (error) {
    errorMessage.value = error instanceof Error ? error.message : 'Unable to load quiz data.'
  }
}

function normalizeQuestion(question) {
  if (question.type !== 'image' || !question.image?.src) {
    return question
  }

  return {
    ...question,
    image: {
      ...question.image,
      src: resolveImageSource(question.image.src),
    },
  }
}

function resolveImageSource(relativePath) {
  const match = Object.entries(imageModules).find(([modulePath]) => modulePath.endsWith(`/${relativePath}`))
  return match?.[1] ?? relativePath
}

function getFieldIds(question) {
  if (question.type === 'image') {
    return question.dropdowns.map((dropdown) => dropdown.id)
  }

  if (question.type === 'incomplete_text') {
    return question.parts.filter((part) => part.type === 'blank').map((part) => part.id)
  }

  if (question.type === 'multiple_choice') {
    return ['selection']
  }

  return []
}

function isFilled(value) {
  return value !== '' && value !== null && value !== undefined
}

function updateAnswer(questionId, fieldId, value) {
  answers[questionId] = {
    ...answers[questionId],
    [fieldId]: value,
  }
}

function isQuestionCorrect(question) {
  const questionAnswers = answers[question.id] ?? {}

  if (question.type === 'multiple_choice') {
    return question.correct_answers.includes(questionAnswers.selection)
  }

  return Object.entries(question.correct_answers).every(
    ([fieldId, correctValue]) => questionAnswers[fieldId] === correctValue,
  )
}

function checkCurrentQuestion() {
  if (!currentQuestion.value) {
    return
  }

  checked[currentQuestion.value.id] = true
}

function previousQuestion() {
  currentIndex.value = Math.max(0, currentIndex.value - 1)
}

function nextQuestion() {
  currentIndex.value = Math.min(questions.value.length - 1, currentIndex.value + 1)
}

function goToQuestion(index) {
  currentIndex.value = index
}
</script>

<template>
  <div class="quiz-shell">
    <QuestionSidebar
      :questions="questions"
      :current-index="currentIndex"
      :state-by-question="stateByQuestion"
      @navigate="goToQuestion"
    />

    <main class="quiz-main">
      <header class="quiz-header">
        <div>
          <p class="eyebrow">JSON-Driven Vue Quiz</p>
          <h1>Exam Workspace</h1>
        </div>
        <p v-if="currentQuestion" class="progress">
          Question {{ currentIndex + 1 }} of {{ questions.length }}
        </p>
      </header>

      <section v-if="loading" class="panel state-panel">
        <h2>Loading questions...</h2>
      </section>

      <section v-else-if="errorMessage" class="panel state-panel error">
        <h2>Unable to load quiz</h2>
        <p>{{ errorMessage }}</p>
      </section>

      <section v-else-if="currentQuestion" class="panel question-panel">
        <div class="question-head">
          <div>
            <p class="question-type">{{ currentQuestion.type.replace('_', ' ') }}</p>
            <h2>{{ currentQuestion.prompt }}</h2>
          </div>
          <div
            v-if="checked[currentQuestion.id]"
            class="result-badge"
            :class="isQuestionCorrect(currentQuestion) ? 'correct' : 'checked'"
          >
            {{ isQuestionCorrect(currentQuestion) ? 'Correct' : 'Checked' }}
          </div>
        </div>

        <ImageQuestion
          v-if="currentQuestion.type === 'image'"
          :question="currentQuestion"
          :answers="answers[currentQuestion.id]"
          :checked="checked[currentQuestion.id]"
          @update-answer="updateAnswer(currentQuestion.id, $event.fieldId, $event.value)"
        />

        <IncompleteTextQuestion
          v-else-if="currentQuestion.type === 'incomplete_text'"
          :question="currentQuestion"
          :answers="answers[currentQuestion.id]"
          :checked="checked[currentQuestion.id]"
          @update-answer="updateAnswer(currentQuestion.id, $event.fieldId, $event.value)"
        />

        <MultipleChoiceQuestion
          v-else-if="currentQuestion.type === 'multiple_choice'"
          :question="currentQuestion"
          :answers="answers[currentQuestion.id]"
          :checked="checked[currentQuestion.id]"
          @update-answer="updateAnswer(currentQuestion.id, $event.fieldId, $event.value)"
        />

        <footer class="question-actions">
          <button class="nav-button secondary" :disabled="currentIndex === 0" @click="previousQuestion">
            Previous
          </button>
          <button class="nav-button primary" @click="checkCurrentQuestion">
            Check Answer
          </button>
          <button
            class="nav-button secondary"
            :disabled="currentIndex === questions.length - 1"
            @click="nextQuestion"
          >
            Next
          </button>
        </footer>
      </section>
    </main>
  </div>
</template>
