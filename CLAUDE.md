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
- Category tags: **every category is either Home or Work** — there is no
  untagged state. Toggled two-way via a badge in the Organize Categories
  sheet, and shown as a coloured left tick and caret on the main-screen
  category bar. The two hues are gold and steel — the same pair the modes use
  — held as **fixed** values that are deliberately *not* remapped under
  `[data-mode]`, so a Home-tagged category reads gold and a Work-tagged one
  steel whichever mode is active.

  `state.categoryTags` can still be sparse (categories predating tagging, or
  old backups), so **all reads go through `tagOf(cat)`, which resolves a
  missing entry to `"work"`** — never read the map directly. That fallback is
  what keeps legacy categories reachable: the mode filter hides everything
  outside the active mode, so a genuinely untagged category would appear
  nowhere. New categories are written as `"work"` explicitly, so the tag shows
  up in an export rather than relying on the fallback.
- Work Mode / Home Mode toggle, replacing Show All / Collapse All at the top:
  **Work sits first and is the launch default** (`loadViewMode()` returns
  `"work"` when nothing is saved; a returning user gets the mode they left in).
  Selecting a mode **filters** — only that mode's categories are on screen,
  in saved order — and expands them on the way in, so the list you asked for
  is open when you arrive.

  The mode is always `"home"` or `"work"`; there is no cleared state, because
  with a filter it would render an empty screen. Tapping the mode you're
  already in is therefore a no-op, which also stops a stray tap undoing
  whatever you'd collapsed by hand. Display-only — never mutates the saved
  category order. When the active mode holds no categories the list shows a
  "Nothing in Home/Work" prompt rather than a blank screen.

  The creation sheet's category picker deliberately still lists **all**
  categories, so you can file a Home task without leaving Work mode. The task
  then won't be visible until you switch — accepted for now.
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
  steel; Home falls back to the gold on `:root`. `theme-color` tracks the mode
  so the iOS status bar follows.
- Sheets, menu and toast on the same vocabulary as the main screen: square
  corners throughout, action buttons carrying the 45° notch, and field labels
  rendered as filled tabs (the category-bar treatment — gold left tick,
  rounded right cap, `inline-flex` so the bar hugs its text). Organize rows
  reuse the hatched task-row slab. `.sheet-btn.danger` is text-only and so
  sets `clip-path: none` — a notch needs a fill to cut.
- Task sheet action row: no Cancel button — the backdrop tap is the only
  dismissal. Delete sits left at 1/3, Save right at 2/3, via
  `#sheet-delete { flex: 1 }` / `#sheet-save { flex: 2 }`, scoped by id so
  `.cat-add-btn`'s own flex isn't disturbed. Delete stays hidden when
  creating, so Save spans the full row there.
- The "Pending" tag reads **Wait** — `label: "Wait"`, badge `short: "WAIT"`
  (uppercase to match PRIO/QUICK). The stored tag id is still `pending`, so
  existing tasks and old JSON backups need no migration.
- Archive categories: categories stand in for work projects, and when a
  project finishes it comes off the main view without being deleted —
  projects come back to life, so this is archive-and-restore. A category can
  only be archived once it has **no unfinished tasks**; archiving sweeps its
  remaining (all completed) tasks into `todos.v2.archive` via the same path
  as "Archive done," then sets a flag. Shape: a sparse
  `categoryArchived: { "Project X": true }` map inside `state`, alongside
  `categoryTags`. The category never leaves `state.categories`, so restoring
  is just clearing the flag — it lands back at its original position for
  free. The "keep at least one category" guard applies to archiving too
  (counting only non-archived categories). Every iteration over
  `state.categories` that feeds the main view has to skip archived
  entries — `displayCategories()`, the creation sheet's chip picker.
- Archive category-rename drift fixed: `renameCategory` now backfills
  matching `category` fields in `todos.v2.archive`, so renaming a category
  carries its archived history with it instead of splitting it across the
  old and new names.
- Category edit sheet, opened by tapping a category bar on the main page:
  rename, switch Home/Work type, reorder (Up/Down/Top), archive, delete —
  all the category-management actions, scoped to one category, without
  leaving the main view. Tapping the bar used to toggle collapse/expand;
  that's now the caret's job alone (enlarged to a 28px tap target, since it's
  the only remaining hit zone for a gesture used every day), and the rest of
  the bar opens the edit sheet instead.

  Everything here applies immediately (rename on blur/Enter, type/archive/
  delete/move on tap) — there's no Save button, just Done to close. Up/Down/
  Top reorder relative to the category's **displayed neighbors** (same Home/
  Work tag, not archived) rather than raw array position, via a
  `swapCategories`/`categoryNeighbors` pair: `state.categories` can have the
  other tag's categories interleaved between two same-tag ones, so stepping
  by raw index could silently reorder categories in the *other* mode as a
  side effect, or produce no visible change at all. The buttons disable
  themselves at the top/bottom of that same-tag list rather than no-op
  silently. Delete's confirm-with-task-count logic lives in
  `confirmAndDeleteCategory()`.

  The same sheet also handles **creating** a category: the task sheet's
  category chip picker has a "＋ New" chip (shortened from "＋ New category")
  that opens it with `catEditIsNew` set, hiding Order/Delete/Archive (nothing
  to reorder, delete, or archive before the category exists) and defaulting
  Type to Work. The name field IS the create action — typing a name and
  blurring calls `createCategory()` instead of `renameCategory()`, tracked by
  `catEditCat` being `null` beforehand. Closing the sheet re-renders the task
  sheet's chips (if one's open behind it) so a freshly created category shows
  up as a pick right away. Stacks above the task sheet at the same elevated
  z-index tier the Organize sheet uses.

  **Gotcha that cost a debugging pass:** the Type chips' `onclick` must read
  `catEditCat` live, not close over the `cat` render-time constant — and
  `commitCategoryEditName()`'s create-success path must *not* call
  `renderCategoryEditSheet()`. Tapping a Type chip fires mousedown first,
  which blurs the name field and (on the first tap after typing) creates the
  category synchronously, *before* the chip's own click handler runs. If that
  blur handler rebuilds the chip DOM, the real click event — dispatched by
  actual touch/mouse, not a synthetic `.click()` — silently fails to land on
  the replacement element, so the tag tap is dropped with no error. Leaving
  the existing chip elements alone and reading state fresh at click time
  sidesteps the whole race.
- Organize Categories sheet — relabeled **"Restore categories"** in the
  hamburger menu — is now restore-only. Renaming, retagging, reordering,
  archiving, and deleting all moved to the category edit sheet above, so the
  only thing left here is the archived-categories list with a Restore button
  per row. No more inline rename inputs, drag-to-reorder, or an add-new-
  category row; `renderCatList()` shows "No archived categories." when empty.

## Backlog (build individually, in priority order)

1. **Reorganize categories on the home page** — reorder categories directly
   in the main single-page view (drag-to-reorder in place), rather than only
   via the separate Organize Categories sheet. Partially covered by the
   category edit sheet's Up/Down/Top buttons (tap the bar, no separate sheet
   needed) — true drag-in-place is still open if that's still wanted.
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
   3. Toast messages are invisible while a sheet is open — `.toast` is
      `z-index: 50` but `.sheet`/`.sheet-backdrop` are `61`, so a toast
      triggered from inside the create/edit or Organize Categories sheet
      (e.g. the "keep at least one category" guard) renders behind it.
      Found while building category archiving.
5. **Exercise tracker** — archiving now exists, so this is unblocked. Track
   past exercises; long-term, show a GitHub-contributions-style calendar
   checklist of exercise history.

### Parked / long-term

- Exercise tracker calendar view (the long-term half of #5 — build the
  basic tracker first, calendar view later).
- Archive viewer — a way to browse/search archived (completed) tasks in-app,
  **grouped by category and sorted by date**. The data is already there: every
  entry `archiveDone()`/`archiveCategory()` writes carries `category`,
  `createdAt`, `completedAt` and `archivedAt`, so no model change is needed,
  and category renames no longer fracture a category's history (see Current
  state). Today the archive is write-only, populated by "Archive done" and
  "Archive category," and only inspectable via JSON export. Still needs a
  decision on where the viewer lives (new sheet? filter within the existing
  category view?) before building.

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
