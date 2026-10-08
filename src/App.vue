<script setup>
import { ref, computed, onMounted } from 'vue'
import AbsenceForm from './components/AbsenceForm.vue'
import AbsenceItem from './components/AbsenceItem.vue'

const absences = ref([])
const editingAbsence = ref(null)
const filter = ref('all')
const search = ref('')

// CREATE
function addAbsence(data) {
  absences.value.unshift({
    id: Date.now(),
    ...data,
    status: 'pending'
  })
  saveAbsences()
}

// UPDATE
function updateAbsence(updated) {
  const index = absences.value.findIndex(a => a.id === updated.id)

  if (index !== -1) {
    absences.value[index] = updated
  }

  saveAbsences()
}

// Called when the edit form is submitted
function saveEdit(data) {
  updateAbsence({ ...editingAbsence.value, ...data })
  editingAbsence.value = null
}

function startEdit(absence) {
  editingAbsence.value = absence
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// DELETE
function deleteAbsence(id) {
  absences.value = absences.value.filter(a => a.id !== id)

  if (editingAbsence.value && editingAbsence.value.id === id) {
    editingAbsence.value = null
  }

  saveAbsences()
}

// READ: filtered + searched list
const visibleAbsences = computed(() => {
  const q = search.value.trim().toLowerCase()

  return absences.value.filter(a => {
    const matchesStatus = filter.value === 'all' || a.status === filter.value
    const matchesSearch =
      q === '' ||
      a.studentName.toLowerCase().includes(q) ||
      a.studentId.toLowerCase().includes(q)
    return matchesStatus && matchesSearch
  })
})

const counts = computed(() => ({
  all: absences.value.length,
  pending: absences.value.filter(a => a.status === 'pending').length,
  approved: absences.value.filter(a => a.status === 'approved').length,
  rejected: absences.value.filter(a => a.status === 'rejected').length
}))

function saveAbsences() {
  localStorage.setItem('absences', JSON.stringify(absences.value))
}

onMounted(() => {
  const saved = localStorage.getItem('absences')

  if (saved) {
    absences.value = JSON.parse(saved)
  }
})
</script>

<template>
  <div class="container">
    <header>
      <h1>e-Excuse</h1>
      <p class="muted">Student Absence Management System</p>
    </header>

    <!-- Summary -->
    <div class="stats">
      <div class="stat"><span>{{ counts.all }}</span>Total</div>
      <div class="stat pending"><span>{{ counts.pending }}</span>Pending</div>
      <div class="stat approved"><span>{{ counts.approved }}</span>Approved</div>
      <div class="stat rejected"><span>{{ counts.rejected }}</span>Rejected</div>
    </div>

    <!-- Create / Update form -->
    <AbsenceForm
      :absence="editingAbsence"
      @save="editingAbsence ? saveEdit($event) : addAbsence($event)"
      @cancel="editingAbsence = null"
    />

    <!-- Toolbar -->
    <div class="toolbar">
      <input v-model="search" type="text" placeholder="Search by name or student ID" />
      <select v-model="filter">
        <option value="all">All</option>
        <option value="pending">Pending</option>
        <option value="approved">Approved</option>
        <option value="rejected">Rejected</option>
      </select>
    </div>

    <!-- Read: list -->
    <div class="list">
      <AbsenceItem
        v-for="absence in visibleAbsences"
        :key="absence.id"
        :absence="absence"
        @edit="startEdit"
        @update="updateAbsence"
        @delete="deleteAbsence"
      />

      <p v-if="visibleAbsences.length === 0" class="empty">
        {{ absences.length === 0 ? 'No excuse requests yet. Submit one above.' : 'No requests match your search.' }}
      </p>
    </div>
  </div>
</template>
