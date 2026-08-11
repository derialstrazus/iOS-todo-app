# Todo PWA

A single-file, offline-first todo app for daily train-commute use — organizing
the day in the morning, decompressing and reorganizing in the evening. Tasks are
grouped into categories: one category per work project, plus a **Home** category
for family responsibilities.

## Architecture

- **Single file:** the entire app is one `index.html` (HTML + CSS + JS inline).
- **Hosting:** GitHub Pages. `git push` deploys — no build step.
- **Install target:** iOS Safari, added to the iPhone Home Screen as a PWA.
- **Persistence:** `localStorage` only. Data is stored as a **versioned JSON
  envelope** (forward-compatible). There is **no backend and no sync** — this is
  a deliberate constraint, not a gap to fill.
- **Theme:** dark, styled to feel native on iOS.

## Hard constraints

- **Keep it a single self-contained `index.html`.** If a change would benefit
  from splitting (e.g. extracting a `todo.js` for testing), raise it as a
  decision first — do not split silently.
- **iOS PWA reality:** `localStorage` is the only viable offline layer here.
  Don't propose API/service-based sync; it isn't practical in this architecture.
- **Backup format is JSON, not Markdown.** Backup/restore uses a versioned,
  forward-compatible JSON envelope for reliability. (Markdown/Obsidian is the
  conceptual *inspiration* for task syntax, but JSON is the persistence format.)

## Working style

- **One feature at a time.** Clean build order, quick iterative edits after each
  delivery. Do not bundle unrelated changes into a single pass.
- **Surface data-model and shared-UI decisions before building.** When a feature
  touches the data model or a shared UI surface (the creation/edit sheet, task
  rows, category view), flag the decision and lay out options *first* rather than
  assuming.
- **Sequence to avoid rework.** Prefer an order that avoids revisiting finished
  work.
- **Smoke-test before handoff.** Each delivered change should be functionally
  tested before it's considered done.
- **Nothing is locked in.** Aesthetic and UX feel matter; reversions are fine if
  something doesn't feel right in real use.

## Current state (feature-complete, in real-world trial)

Delivered and in active use:

- Collapsible single-page category view with Show All / Collapse All.
- Floating action button (FAB) opening a bottom sheet for task creation
  (category chip picker, task name, optional tag).
- Edit-via-tap reusing the same sheet, with Delete.
- Tag icons on task rows (❗ ⌛ ⚡ with colored backgrounds).
- Hamburger menu consolidating Export, Restore, Clear done, and Organize
  categories.
- Organize-categories sheet: inline rename, drag-to-reorder, delete with
  task-count confirmation.
- "＋ New category" chip in the creation sheet.
- Full-fidelity JSON backup/restore (versioned envelope, forward-compatible).
- Touch-optimized drag-to-reorder via the `≡` handle.
- Creation sheet orders Task before Category; Show All / Collapse All live in
  the hamburger menu; Category Organizer's "Add" button disables on empty
  input; compact task-row padding.

## Backlog (build individually, in priority order)

1. **Add-task shortcut per category** — a "+" button on each category box,
   replacing the task count, to add a task directly into that category.
2. **Category tags** — tag categories as "Home" or "Work".
3. **Work Mode / Home Mode toggle**, replacing Show All / Collapse All at the
   top — moves Work or Home categories to the top based on the selected mode.
   Depends on #2 (needs category tags to know which categories qualify).
4. **Recurring tasks** — on app launch, check the recurring-tasks list and add
   any that are due; track each recurring task's last-added date in
   localStorage to determine when it's due again.
5. **Timestamps** — record `createdAt` and `completedAt` on tasks.
6. **Comments on tasks** — shown as italics on the main page.
7. **Archive completed tasks.**
8. **Bug fixes:**
   1. iPhone: extra gap at the bottom of the screen (screenshot pending from user).
   2. iPhone: keyboard sometimes doesn't dismiss after a task is saved.
9. **Exercise tracker** — depends on #7 (archiving must exist first). Track
   past exercises; long-term, show a GitHub-contributions-style calendar
   checklist of exercise history.

### Parked / long-term

- Exercise tracker calendar view (the long-term half of #9 — build the
  basic tracker first, calendar view later).

## Testing (planned)

- Unit logic: **Vitest + jsdom**.
- E2E: **Playwright**.
- Prerequisite under consideration: extract pure logic into a `todo.js` module
  so it's unit-testable. This is a single-file-constraint decision — raise it
  before doing it.

## Local dev

- Quick check: open `index.html` directly in a browser.
- Serve (closer to PWA behavior): `python3 -m http.server` then open the printed
  localhost URL.
- Deploy: commit and `git push` — GitHub Pages serves the pushed `index.html`.
