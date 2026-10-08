<script setup>
import { computed } from 'vue'

const props = defineProps({
  expense: Object
})

const emit = defineEmits(['delete', 'edit', 'update'])

const formattedDate = computed(() =>
  new Date(props.expense.date + 'T00:00:00').toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  })
)

const formattedAmount = computed(() =>
  new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(props.expense.amount)
)

function setStatus(status) {
  emit('update', { ...props.expense, status })
}

function deleteExpense() {
  const displayText = props.expense.description.length > 20
    ? props.expense.description.substring(0, 20) + '...'
    : props.expense.description
  if (confirm(`Delete the expense "${displayText}"?`)) {
    emit('delete', props.expense.id)
  }
}
</script>

<template>
  <div class="card item">
    <div class="item-head">
      <div>
        <strong>{{ expense.description }}</strong>
        <span class="muted"> · {{ formattedAmount }}</span>
      </div>
      <span class="badge" :class="expense.status">{{ expense.status }}</span>
    </div>

    <p class="meta">{{ formattedDate }} · {{ expense.category }}</p>

    <div class="actions">
      <button
        v-if="expense.status !== 'completed'"
        class="btn success"
        @click="setStatus('completed')"
      >
        Mark Complete
      </button>
      <button
        v-if="expense.status !== 'pending'"
        class="btn"
        @click="setStatus('pending')"
      >
        Reset
      </button>
      <button class="btn" @click="emit('edit', expense)">Edit</button>
      <button class="btn danger" @click="deleteExpense">Delete</button>
    </div>
  </div>
</template>