<script setup>
import { ref, watch } from 'vue'

// If "absence" is passed in, the form works in Edit mode (UPDATE).
// If not, it works in Add mode (CREATE).
const props = defineProps({
  absence: Object
})

const emit = defineEmits(['save', 'cancel'])

const reasons = ['Illness', 'Family Emergency', 'Medical Appointment', 'School Activity', 'Other']

function emptyForm() {
  return {
    studentName: '',
    studentId: '',
    date: new Date().toISOString().slice(0, 10),
    reason: 'Illness',
    details: ''
  }
}

const form = ref(emptyForm())
const error = ref('')

// Fill the form when editing, reset it when adding
watch(
  () => props.absence,
  (value) => {
    form.value = value ? { ...value } : emptyForm()
    error.value = ''
  },
  { immediate: true }
)

function submit() {
  if (form.value.studentName.trim() === '' || form.value.studentId.trim() === '') {
    error.value = 'Student name and student ID are required.'
    return
  }
  if (!form.value.date) {
    error.value = 'Please choose the date of absence.'
    return
  }

  emit('save', {
    ...form.value,
    studentName: form.value.studentName.trim(),
    studentId: form.value.studentId.trim(),
    details: form.value.details.trim()
  })

  if (!props.absence) {
    form.value = emptyForm()
  }
  error.value = ''
}
</script>

<template>
  <form class="card form" @submit.prevent="submit">
    <h2>{{ absence ? 'Edit Excuse Request' : 'Submit Excuse Request' }}</h2>

    <div class="row">
      <label>
        Student Name
        <input v-model="form.studentName" type="text" placeholder="e.g. Juan Dela Cruz" />
      </label>
      <label>
        Student ID
        <input v-model="form.studentId" type="text" placeholder="e.g. 2024-00123" />
      </label>
    </div>

    <div class="row">
      <label>
        Date of Absence
        <input v-model="form.date" type="date" />
      </label>
      <label>
        Reason
        <select v-model="form.reason">
          <option v-for="r in reasons" :key="r" :value="r">{{ r }}</option>
        </select>
      </label>
    </div>

    <label>
      Details (optional)
      <textarea v-model="form.details" rows="3" placeholder="Explain briefly why you were absent"></textarea>
    </label>

    <p v-if="error" class="error">{{ error }}</p>

    <div class="actions">
      <button type="submit" class="btn primary">
        {{ absence ? 'Save Changes' : 'Submit Request' }}
      </button>
      <button v-if="absence" type="button" class="btn" @click="emit('cancel')">
        Cancel
      </button>
    </div>
  </form>
</template>
