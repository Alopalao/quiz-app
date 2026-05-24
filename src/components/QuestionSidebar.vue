<script setup>
defineProps({
  questions: {
    type: Array,
    default: () => [],
  },
  currentIndex: {
    type: Number,
    default: 0,
  },
  stateByQuestion: {
    type: Object,
    default: () => ({}),
  },
})

defineEmits(['navigate'])

function statusClass(questionId, stateByQuestion) {
  return stateByQuestion?.[questionId]?.tone ?? 'status-not-started'
}

function statusLabel(questionId, stateByQuestion) {
  return stateByQuestion?.[questionId]?.label ?? 'Not started'
}
</script>

<template>
  <aside class="sidebar">
    <h2>Question List</h2>
    <ul>
      <li
        v-for="(question, index) in questions"
        :key="question.id"
        class="sidebar-item"
        :class="[index === currentIndex ? 'active' : '', statusClass(question.id, stateByQuestion)]"
        @click="$emit('navigate', index)"
      >
        <span class="q-number">{{ index + 1 }}</span>
        <div class="q-meta">
          <span class="q-id">{{ question.id }}</span>
          <span class="q-type">{{ question.type }}</span>
          <span class="q-status">{{ statusLabel(question.id, stateByQuestion) }}</span>
        </div>
        <p class="q-prompt">{{ question.prompt }}</p>
      </li>
    </ul>
  </aside>
</template>

<style scoped>
.sidebar {
  width: 20rem;
  min-width: 16.25rem;
  background: linear-gradient(180deg, #13333e 0%, #0c1f28 100%);
  border-right: 0.0625rem solid rgba(255, 255, 255, 0.08);
  padding: 1rem 0.75rem;
  overflow-y: auto;
  height: 100vh;
}

.sidebar h2 {
  color: #f5f0e8;
  font-size: 1rem;
  padding: 0 0.5rem 0.75rem;
  border-bottom: 0.0625rem solid rgba(255, 255, 255, 0.08);
  margin: 0 0 0.5rem;
}

ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.sidebar-item {
  display: grid;
  grid-template-columns: 1.5rem 1fr;
  grid-template-rows: auto auto;
  gap: 0 0.5rem;
  padding: 0.6rem 0.5rem;
  border-radius: 0.75rem;
  margin-bottom: 0.25rem;
  cursor: pointer;
  border: 0.0625rem solid transparent;
  transition: background 0.15s, border-color 0.15s;
}

.sidebar-item:hover {
  background: rgba(255, 255, 255, 0.06);
}

.sidebar-item.active {
  border-color: #ffd36e;
  background: rgba(255, 211, 110, 0.12);
}

.q-number {
  grid-row: 1 / 3;
  grid-column: 1;
  font-weight: 700;
  font-size: 0.85rem;
  color: #d9e7ea;
  align-self: center;
}

.q-meta {
  grid-column: 2;
  display: flex;
  gap: 0.35rem;
  align-items: center;
  flex-wrap: wrap;
}

.q-id {
  font-size: 0.7rem;
  font-weight: 600;
  color: #87d7c8;
  background: rgba(0, 0, 0, 0.16);
  border: 0.0625rem solid rgba(255, 255, 255, 0.1);
  padding: 0.0625rem 0.3125rem;
  border-radius: 0.25rem;
}

.q-type,
.q-status {
  font-size: 0.68rem;
  padding: 0.0625rem 0.3125rem;
  border-radius: 0.25rem;
}

.q-type {
  color: #ffdba6;
  background: rgba(255, 255, 255, 0.08);
}

.q-prompt {
  grid-column: 2;
  font-size: 0.75rem;
  color: #d9e7ea;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 15rem;
  margin: 0.25rem 0 0;
}

.status-not-started .q-status {
  background: #56656a;
  color: #eaf1f2;
}

.status-not-started {
  border-left: 0.1875rem solid #8aa1a8;
}

.status-incomplete .q-status {
  background: #6d4d1f;
  color: #ffe0aa;
}

.status-incomplete {
  border-left: 0.1875rem solid #f0b35b;
}

.status-complete .q-status {
  background: #235168;
  color: #daf3fb;
}

.status-complete {
  border-left: 0.1875rem solid #7bc8e6;
}

.status-correct .q-status {
  background: #21553b;
  color: #dcf7e7;
}

.status-correct {
  border-left: 0.1875rem solid #6fdd9c;
}

.status-checked .q-status {
  background: #5e3131;
  color: #ffd6d2;
}

.status-checked {
  border-left: 0.1875rem solid #ef8b81;
}

@media (max-width: 60rem) {
  .sidebar {
    width: 100%;
    min-width: 0;
    height: auto;
    max-height: 20rem;
  }

  .q-prompt {
    max-width: none;
  }
}
</style>