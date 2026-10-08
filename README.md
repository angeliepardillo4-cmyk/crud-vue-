# e-Excuse: Student Absence Management System

A Vue 3 + Vite CRUD app (same structure as the To-Do reference).
Data is saved in the browser with `localStorage`.

## Run
    npm install
    npm run dev

## CRUD
- **Create**: submit an excuse request (name, ID, date, reason, details). Starts as `pending`.
- **Read**: list of requests, summary counts, search by name/ID, filter by status.
- **Update**: Edit a request, or Approve / Reject / Reset its status.
- **Delete**: remove a request (with confirmation).

## Files
- `src/App.vue`: state + CRUD functions (add, update, delete, save/load)
- `src/components/AbsenceForm.vue`: add/edit form
- `src/components/AbsenceItem.vue`: one request card with actions
