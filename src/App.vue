<script setup>
import { ref, computed, onMounted } from 'vue'
import ExpenseForm from './components/ExpenseForm.vue'
import ExpenseItem from './components/ExpenseItem.vue'

const expenses = ref([])
const editingExpense = ref(null)
const filter = ref('all')
const search = ref('')

function addExpense(data) {
  expenses.value.unshift({
    id: Date.now(),
    ...data
  })
  saveExpenses()
}

function updateExpense(updated) {
  const index = expenses.value.findIndex(e => e.id === updated.id)

  if (index !== -1) {
    expenses.value[index] = updated
  }

  saveExpenses()
}

function saveEdit(data) {
  updateExpense({ ...editingExpense.value, ...data })
  editingExpense.value = null
}

function startEdit(expense) {
  editingExpense.value = expense
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function deleteExpense(id) {
  expenses.value = expenses.value.filter(e => e.id !== id)

  if (editingExpense.value && editingExpense.value.id === id) {
    editingExpense.value = null
  }

  saveExpenses()
}

const visibleExpenses = computed(() => {
  const q = search.value.trim().toLowerCase()

  return expenses.value.filter(e => {
    const matchesCategory = filter.value === 'all' || e.category === filter.value
    const matchesSearch =
      q === '' ||
      e.description.toLowerCase().includes(q)
    return matchesCategory && matchesSearch
  })
})

const summary = computed(() => {
  const total = expenses.value.reduce((sum, e) => sum + e.amount, 0)
  const completed = expenses.value.filter(e => e.status === 'completed').reduce((sum, e) => sum + e.amount, 0)
  const pending = expenses.value.filter(e => e.status === 'pending').reduce((sum, e) => sum + e.amount, 0)
  
  return {
    total: expenses.value.length,
    totalAmount: total,
    completedAmount: completed,
    pendingAmount: pending
  }
})

function saveExpenses() {
  localStorage.setItem('expenses', JSON.stringify(expenses.value))
}

onMounted(() => {
  const saved = localStorage.getItem('expenses')

  if (saved) {
    expenses.value = JSON.parse(saved)
  }
})
</script>

<template>
  <div class="container">
    <header>
      <h1>Personal Expense Tracker</h1>
      <p class="muted">Track and manage your daily expenses</p>
    </header>

    <div class="stats">
      <div class="stat"><span>{{ summary.total }}</span>Total Expenses</div>
      <div class="stat completed"><span>${{ summary.completedAmount.toFixed(2) }}</span>Completed</div>
      <div class="stat pending"><span>${{ summary.pendingAmount.toFixed(2) }}</span>Pending</div>
      <div class="stat total-amount"><span>${{ summary.totalAmount.toFixed(2) }}</span>Total Amount</div>
    </div>

    <ExpenseForm
      :expense="editingExpense"
      @save="editingExpense ? saveEdit($event) : addExpense($event)"
      @cancel="editingExpense = null"
    />

    <div class="toolbar">
      <input v-model="search" type="text" placeholder="Search by description" />
      <select v-model="filter">
        <option value="all">All Categories</option>
        <option value="Food">Food</option>
        <option value="Transportation">Transportation</option>
        <option value="Entertainment">Entertainment</option>
        <option value="Shopping">Shopping</option>
        <option value="Bills">Bills</option>
        <option value="Other">Other</option>
      </select>
    </div>

    <div class="list">
      <ExpenseItem
        v-for="expense in visibleExpenses"
        :key="expense.id"
        :expense="expense"
        @edit="startEdit"
        @update="updateExpense"
        @delete="deleteExpense"
      />

      <p v-if="visibleExpenses.length === 0" class="empty">
        {{ expenses.length === 0 ? 'No expenses recorded yet. Add your first expense above.' : 'No expenses match your search.' }}
      </p>
    </div>
  </div>
</template>