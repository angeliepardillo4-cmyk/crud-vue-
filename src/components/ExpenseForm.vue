<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  expense: Object
})

const emit = defineEmits(['save', 'cancel'])

const categories = ['Food', 'Trsansportation', 'Entertainment', 'Shopping', 'Bills', 'Other']

function emptyForm() {
  return {
    description: '',
    amount: '',
    date: new Date().toISOString().slice(0, 10),
    category: 'Food',
    status: 'pending'
  }
}

const form = ref(emptyForm())
const error = ref('')

watch(
  () => props.expense,
  (value) => {
    form.value = value ? { ...value } : emptyForm()
    error.value = ''
  },
  { immediate: true }
)

function submit() {
  if (form.value.description.trim() === '') {
    error.value = 'Description is required.'
    return
  }
  if (!form.value.amount || isNaN(parseFloat(form.value.amount)) || parseFloat(form.value.amount) <= 0) {
    error.value = 'Please enter a valid amount.'
    return
  }

  emit('save', {
    ...form.value,
    description: form.value.description.trim(),
    amount: parseFloat(form.value.amount),
    category: form.value.category.trim() || 'Other'
  })

  if (!props.expense) {
    form.value = emptyForm()
  }
  error.value = ''
}
</script>

<template>
  <form class="card form" @submit.prevent="submit">
    <h2>{{ expense ? 'Edit Expense' : 'Add New Expense' }}</h2>

    <div class="row">
      <label>
        Description
        <input v-model="form.description" type="text" placeholder="e.g. Lunch at Restaurant" />
      </label>
      <label>
        Amount
        <input v-model="form.amount" type="number" step="0.01" placeholder="e.g. 25.50" />
      </label>
    </div>

    <div class="row">
      <label>
        Date
        <input v-model="form.date" type="date" />
      </label>
      <label>
        Category
        <select v-model="form.category">
          <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
        </select>
      </label>
    </div>

    <p v-if="error" class="error">{{ error }}</p>

    <div class="actions">
      <button type="submit" class="btn primary">
        {{ expense ? 'Save Changes' : 'Add Expense' }}
      </button>
      <button v-if="expense" type="button" class="btn" @click="emit('cancel')">
        Cancel
      </button>
    </div>
  </form>
</template>