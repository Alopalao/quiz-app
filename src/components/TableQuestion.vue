<script setup>
import { ref } from 'vue'

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
const activeRowHintId = ref('')

function onChange(fieldId, value) {
  emit('update-answer', { fieldId, value })
}

function toneForCell(cell, answers, question, checked) {
  if (!checked || cell.type !== 'dropdown') {
    return ''
  }

  return answers[cell.fieldId] === question.correct_answers[cell.fieldId]
    ? 'select-correct'
    : 'select-incorrect'
}

function showInlineCorrection(cell, answers, question, checked) {
  if (!checked || cell.type !== 'dropdown') {
    return false
  }

  return answers[cell.fieldId] !== question.correct_answers[cell.fieldId]
}

function hasRowHint(row) {
  return typeof row.hint === 'string' && row.hint.trim().length > 0
}

function toggleRowHint(rowId) {
  const row = props.question.rows.find((item) => item.id === rowId)

  if (!row || !hasRowHint(row)) {
    return
  }

  activeRowHintId.value = activeRowHintId.value === rowId ? '' : rowId
}
</script>

<template>
  <div class="table-question">
    <div class="table-wrap">
      <table>
        <thead>
          <tr>
            <th class="row-number-head">#</th>
            <th v-for="column in question.columns" :key="column.id">{{ column.label }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in question.rows" :key="row.id">
            <td class="row-number-cell">
              <button
                type="button"
                class="row-number"
                :class="hasRowHint(row) ? 'has-hint' : 'no-hint'"
                :disabled="!hasRowHint(row)"
                :aria-label="hasRowHint(row) ? `Show hint for row ${rowIndex + 1}` : `Row ${rowIndex + 1}`"
                @click="toggleRowHint(row.id)"
              >
                {{ rowIndex + 1 }}
              </button>
              <div v-if="activeRowHintId === row.id && hasRowHint(row)" class="row-hint-bubble">
                {{ row.hint }}
              </div>
            </td>
            <td v-for="cell in row.cells" :key="cell.columnId">
              <span v-if="cell.type === 'text'" class="cell-text">{{ cell.value }}</span>
              <div v-else-if="cell.type === 'dropdown'" class="dropdown-cell">
                <select
                  :value="answers[cell.fieldId]"
                  :class="toneForCell(cell, answers, question, checked)"
                  @change="onChange(cell.fieldId, $event.target.value)"
                >
                  <option value="">Select</option>
                  <option v-for="option in cell.options" :key="option" :value="option">{{ option }}</option>
                </select>
                <p v-if="showInlineCorrection(cell, answers, question, checked)" class="inline-correct-answer">
                  Correct: <strong>{{ question.correct_answers[cell.fieldId] }}</strong>
                </p>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.table-question {
  display: grid;
  gap: 0.75rem;
}

.table-wrap {
  overflow: auto;
  border: 0.0625rem solid #dfd4c1;
  border-radius: 0.9rem;
  background: #fffdf7;
}

table {
  width: 100%;
  border-collapse: collapse;
  min-width: 40rem;
}

th,
td {
  border-bottom: 0.0625rem solid #eadfcb;
  padding: 0.65rem;
  text-align: left;
  vertical-align: middle;
}

.row-number-head,
.row-number-cell {
  width: 3.1rem;
  text-align: center;
}

.row-number-cell {
  position: relative;
}

.row-number {
  display: inline-grid;
  place-items: center;
  width: 1.35rem;
  height: 1.35rem;
  border-radius: 999rem;
  border: 0;
  background: #1f4f63;
  color: #f1f8fb;
  font-size: 0.72rem;
  font-weight: 700;
  cursor: pointer;
}

.row-number.no-hint {
  cursor: default;
}

.row-number:disabled {
  opacity: 1;
}

.row-number.has-hint {
  box-shadow: 0 0 0 0.125rem rgba(50, 101, 122, 0.35);
}

.row-hint-bubble {
  position: absolute;
  left: calc(100% - 0.2rem);
  top: 0;
  z-index: 25;
  width: 14rem;
  text-align: left;
  background: #f2f8fb;
  border: 0.0625rem solid #c8dde7;
  border-radius: 0.6rem;
  padding: 0.45rem 0.55rem;
  font-size: 0.78rem;
  color: #1b3641;
  box-shadow: 0 0.35rem 0.9rem rgba(23, 49, 62, 0.18);
}

th {
  background: #f5ecdc;
  color: #2d434e;
  font-size: 0.84rem;
}

tbody tr:last-child td {
  border-bottom: 0;
}

.cell-text {
  color: #1c2f38;
}

.dropdown-cell {
  display: grid;
  gap: 0.25rem;
}

select {
  width: 100%;
  min-width: 10rem;
  border: 0.0625rem solid #b7c6cc;
  border-radius: 0.55rem;
  padding: 0.4rem 0.5rem;
  background: #fff;
  color: #112028;
}

select.select-correct {
  background: #e9f9e8;
  border-color: #72b57c;
}

select.select-incorrect {
  background: #ffeaf3;
  border-color: #ce6c94;
}

.inline-correct-answer {
  margin: 0;
  font-size: 0.76rem;
  color: #7a3653;
}
</style>
