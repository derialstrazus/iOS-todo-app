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
- Tag badges on task rows (PRIO / HOLD / QUICK, dog-eared corner).
- Hamburger menu consolidating Export, Restore, Archive done, and Organize
  categories.
- Organize-categories sheet: inline rename, drag-to-reorder, delete with
  task-count confirmation.
- "＋ New category" chip in the creation sheet.
- Full-fidelity JSON backup/restore (versioned envelope, forward-compatible).
- Touch-optimized drag-to-reorder via the `≡` handle.
- Creation sheet orders Task before Category; Show All / Collapse All live in
  the hamburger menu; Category Organizer's "Add" button disables on empty
  input; compact task-row padding.
- Add-task shortcut per category: a soft-tinted blue "+" circle on each
  category header (replacing the task count) opens the creation sheet with
  Category hidden/locked to that category.
- Category tags: categories can be tagged "Home" or "Work" (cycled via a
  badge in the Organize Categories sheet) and show as a colored left accent
  bar (green/blue) on the main-screen category section.
- Work Mode / Home Mode toggle, replacing Show All / Collapse All at the top:
  selecting a mode moves matching-tag categories to the top and expands them,
  collapses opposite-tag categories, and leaves untagged categories untouched.
  Tapping the active mode again clears it. Display-only sort — never mutates
  the saved category order.
- Timestamps: `createdAt` set once at task creation; `completedAt` set when a
  task is checked done and cleared back to `null` if unchecked. Tasks created
  before this shipped simply lack the fields until next touched.
- Archive completed tasks: hamburger menu's "Archive done" moves all
  completed tasks (across every category) out of `state` into a separate
  `todos.v2.archive` localStorage key, instead of deleting them. Keeping it
  a separate key (rather than folding archive into `state`) means routine
  saves — checking a box, dragging a row — never have to re-serialize
  accumulated history. Included in JSON export/restore for full fidelity.
  Write-only in the UI for now; no in-app way to browse it yet (see backlog).
  At ~5000 archived tasks, prompts to export a backup and trims the archive
  down to the most recent 2500.
- Styling refresh, in the visual language of Deus Ex: Human Revolution.
  Warm gold on near-black, with a god-ray wash and 45° hatch on task rows.
  The signature shape is an **opposing 45° notch** (top-right and bottom-left,
  equal offset on both axes) carried only by things you *press* — the three
  top buttons and the FAB. Task rows are square. Two mechanical rules follow
  from `clip-path`: notched elements can't use a CSS border (it's cut away on
  the diagonals, so they use a lighter fill instead), and they must use
  `filter: drop-shadow()` rather than `box-shadow`, which is also clipped.
  Category bars sit *lighter* than the ground, not darker — the list scrolls,
  so contrast can't depend on where a bar lands against the gradient. Emoji
  are gone; all icons are inline SVG. Full spec and the rejected alternatives
  live in `design/` (git-ignored, local only).
- Home / Work colour palettes: `data-mode="work"` on `<html>`, stamped from
  `viewMode` in `render()`, swaps the whole token set from warm gold to cold
  steel. Home *and* cleared-mode both fall back to the gold on `:root`, so
  those two states look identical apart from the active tab. `theme-color`
  tracks the mode so the iOS status bar follows.

## Backlog (build individually, in priority order)

1. **Reorganize categories on the home page** — reorder categories directly
   in the main single-page view (drag-to-reorder in place), rather than only
   via the separate Organize Categories sheet.
2. **Recurring tasks** — on app launch, check the recurring-tasks list and add
   any that are due; track each recurring task's last-added date in
   localStorage to determine when it's due again.
3. **Comments on tasks** — shown as italics on the main page.
4. **Bug fixes:**
   1. iPhone: extra gap at the bottom of the screen, below the last row.
      Repro: happens after double-tapping the page.
   2. iPhone: keyboard sometimes doesn't dismiss after a task is saved.
      Repro: happens when hitting the blue checkmark/return key on the
      keyboard rather than tapping away.
5. **Exercise tracker** — archiving now exists, so this is unblocked. Track
   past exercises; long-term, show a GitHub-contributions-style calendar
   checklist of exercise history.
6. **Restyle the bottom sheets** — the create/edit sheet and the Organize
   Categories sheet were rethemed to the new palette when the styling refresh
   landed, but they keep their original rounded, iOS-style geometry (18px
   corners, pill chips, circular controls). Against the angular main screen
   they now read as a different design language. Bring them onto the same
   vocabulary: square corners, 45° notches on the action buttons, and the
   filled-bar treatment for field labels.

### Parked / long-term

- Exercise tracker calendar view (the long-term half of #5 — build the
  basic tracker first, calendar view later).
- Archive viewer — a way to browse/search archived (completed) tasks in-app.
  Not yet designed; today the archive is write-only, populated by "Archive
  done" and only inspectable via JSON export. Needs a decision on where it
  lives (new sheet? filter within existing category view?) before building.

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
