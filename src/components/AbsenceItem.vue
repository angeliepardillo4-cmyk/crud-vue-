<script setup>
import { computed } from 'vue'

const props = defineProps({
  absence: Object
})

const emit = defineEmits(['delete', 'edit', 'update'])

const formattedDate = computed(() =>
  new Date(props.absence.date + 'T00:00:00').toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  })
)

// Teacher actions: change the status (UPDATE)
function setStatus(status) {
  emit('update', { ...props.absence, status })
}

function deleteAbsence() {
  if (confirm(`Delete the excuse request of ${props.absence.studentName}?`)) {
    emit('delete', props.absence.id)
  }
}
</script>

<template>
  <div class="card item">
    <div class="item-head">
      <div>
        <strong>{{ absence.studentName }}</strong>
        <span class="muted"> · {{ absence.studentId }}</span>
      </div>
      <span class="badge" :class="absence.status">{{ absence.status }}</span>
    </div>

    <p class="meta">{{ formattedDate }} · {{ absence.reason }}</p>
    <p v-if="absence.details" class="details">{{ absence.details }}</p>

    <div class="actions">
      <button
        v-if="absence.status !== 'approved'"
        class="btn success"
        @click="setStatus('approved')"
      >
        Approve
      </button>
      <button
        v-if="absence.status !== 'rejected'"
        class="btn warn"
        @click="setStatus('rejected')"
      >
        Reject
      </button>
      <button
        v-if="absence.status !== 'pending'"
        class="btn"
        @click="setStatus('pending')"
      >
        Reset
      </button>
      <button class="btn" @click="emit('edit', absence)">Edit</button>
      <button class="btn danger" @click="deleteAbsence">Delete</button>
    </div>
  </div>
</template>
