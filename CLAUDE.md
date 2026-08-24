# Todo PWA

A single-file, offline-first todo app for daily train-commute use — organizing
the day in the morning, decompressing and reorganizing in the evening. Tasks are
grouped into categories: one category per work project, plus a **Home** category
for family responsibilities.

## Architecture

- **Single file:** all code lives in one `index.html` (HTML + CSS + JS
  inline). The only sidecar files are `manifest.json` and the three icon
  PNGs — PWA plumbing, not code; they're expected and rarely change.
- **Hosting:** GitHub Pages at
  <https://derialstrazus.github.io/iOS-todo-app/>. `git push` deploys — no
  build step.
- **Install target:** iOS Safari, added to the iPhone Home Screen as a PWA.
- **Persistence:** `localStorage` only. Data is stored as a **versioned JSON
  envelope** (forward-compatible). There is **no backend and no sync** — this is
  a deliberate constraint, not a gap to fill.
- **Theme:** dark, styled to feel native on iOS.

## Version

**Current: `v35`, bumped 2026-08-22.** Shown in the app's About sheet
(hamburger menu → About) alongside the same date, from the `APP_VERSION`/
`APP_UPDATED` consts near the top of `index.html`'s `<script>`.

A rolling integer, not a git commit hash (a hash can't be known until
after the commit that would contain it exists, which forced an awkward
separate "bump" commit). Started at 23, not 1 — that's how many prior
commits had touched `index.html` as of the commit that introduced this
counter, so it reflects the app's real edit history rather than resetting
it. On **every code edit** — not just feature work, any change to
`index.html` — increment `APP_VERSION` by 1, set `APP_UPDATED` to today,
and update the line above to match, all in the **same commit** as the edit
itself.

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
- **Sheet buttons with side effects close the sheet immediately.** When a
  button in a bottom sheet performs its action (save, archive, restore,
  delete), the sheet closes on that tap — it doesn't stay open waiting for
  a separate dismissal.

## Current state (feature-complete, in real-world trial)

Delivered and in active use. **This list is chronological — later bullets
supersede earlier ones** (e.g. the tag badge now reads WAIT, not HOLD, and
the Organize Categories sheet is now restore-only; the corrections appear
further down). When in doubt, the code wins:

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
- Category edit sheet, opened from the main page: rename, switch Home/Work
  type, reorder (Up/Down/Top), archive, delete — all the category-management
  actions, scoped to one category, without leaving the main view. (Superseded
  below: it's no longer opened by tapping the bar, but from a dedicated menu
  button.)

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
- Multi-domain backup envelope (the second Future-screens prerequisite):
  export/restore runs off a `BACKUP_DOMAINS` registry — per domain,
  `current` feeds the export payload, `normalize` validates an incoming
  section (null = invalid), `apply` swaps it in and persists, `describe`
  writes the confirm-prompt line. Sections are optional top-level envelope
  keys (`state` + `archive` today); **restore is section-scoped** — only
  domains present in the file are replaced, so a todo-only backup can never
  wipe a future domain's data. A present-but-malformed section rejects the
  whole file rather than being silently dropped (stricter than before, by
  design). Bare-state files (no envelope) still restore; a file holding
  only sections this version doesn't know is rejected as having nothing to
  restore. `BACKUP_VERSION` stays 1 — it marks the shape of existing
  sections, and adding a new optional section is deliberately not a bump.
  Adding a domain = one registry entry.
- Screen layer (the nav/dispatcher refactor from Future screens): `render()`
  is now a dispatcher over a `SCREENS` registry — each entry owns the scroll
  area's renderer, its palette (a `data-mode` value or `null` for the `:root`
  gold, mapped to a status-bar colour via `BAR_COLORS`), and a `fab` flag.
  The top bar is built once at startup by `buildNav()` from a `NAV` registry
  (Work and Home are two entries that both target the todos screen, setting
  its `viewMode` sub-state; future screens append one entry each);
  `render()` only toggles the buttons' active state, never rebuilds them.
  The active screen persists in `todos.v2.screen`, guarded like the mode,
  with unknown ids falling back to `todos`. Only the todos screen is
  registered so far, so nothing changed visually — that's the point.
- Organize Categories sheet — relabeled **"Restore categories"** in the
  hamburger menu — is now restore-only. Renaming, retagging, reordering,
  archiving, and deleting all moved to the category edit sheet above, so the
  only thing left here is the archived-categories list with a Restore button
  per row. No more inline rename inputs, drag-to-reorder, or an add-new-
  category row; `renderCatList()` shows "No archived categories." when empty.
- Category bar reworked: tapping the bar toggles collapse/expand again (the
  category edit sheet's own tap-to-open was found confusing), and a new "⋮"
  button sits between the category name and the "+" — that's what opens the
  edit sheet now. The caret is purely decorative (shrunk back down; the whole
  bar is the tap target). Category edit sheet's Delete/Archive/Done now share
  one row under a "Finalize" field-label, with Done styled as the primary
  (golden) action to match the task sheet's Save button; Delete/Archive still
  hide individually when creating a new category, rather than the whole row
  (Done has to stay visible there). Switching Home/Work mode no longer
  force-expands every category in the target mode — each category's collapse
  state now survives a mode switch, via `collapsed[cat]` no longer being
  reset in `setViewMode()`.
- Comments on tasks (backlog #2): an optional multi-line note per task,
  entered in its own "Comment (optional)" field right under Task in the
  create/edit sheet (`sheet-comment`, a `<textarea>` — unlike every other
  sheet field so far, which are single-line inputs or chip pickers). Stored
  as `task.comment`, `null` when empty, same pattern as `task.tag`; needs no
  backup-envelope change since `normalizeState()` validates shape only, not
  individual task fields. On the task row it renders as a second line under
  the task text — `.label` became a flex column (`.label-text` +
  `.label-comment`) rather than the task text sitting directly in `.label` —
  styled italic and dimmed, truncated to one line with an ellipsis
  (`white-space: nowrap` + `text-overflow: ellipsis`) so a long comment
  doesn't blow out row height; opening the row's edit sheet shows the full
  text. `.label` needs `min-width: 0` for that ellipsis to actually take
  effect, since a flex item won't shrink past its content's intrinsic width
  otherwise. Enter in the Comment field saves the sheet, same as Task —
  `sheetComment`'s `keydown` handler calls `preventDefault()` so Enter
  doesn't also insert a newline first. Deliberate trade: no multi-line entry
  via Return, chosen over a textarea that silently swallowed Enter (was
  logged as "no blue checkmark save" — expected checkmark-saves behavior
  like every other field, not textarea's default newline).
- Three iPhone bug fixes from the backlog. `closeSheet()` now blurs
  `sheetText` and `sheetComment` before closing — tapping the backdrop
  blurred the focused field for free, but saving via the keyboard's
  Return/checkmark key closed the sheet programmatically without ever
  blurring, so iOS left its keyboard up. `setViewMode()` now resets
  `scroll.scrollTop` to 0, so switching Work/Home always lands at the top of
  the new list instead of keeping whatever scroll position the previous mode
  was left at. The double-tap gap took two passes: `touch-action:
  manipulation` on `html, body` (to skip iOS Safari's built-in
  double-tap-to-zoom gesture) wasn't enough on its own — confirmed still
  reproducing on a real iPhone. The actual fix was swapping `body`'s
  `height: 100%` for `position: fixed; inset: 0`. Theory: `height: 100%`
  bakes in whatever the layout viewport's height was at layout time, and iOS
  can desync that from the true visual viewport after a double-tap even when
  the zoom gesture itself is suppressed, leaving a gap below the last row
  where `body`'s box falls short of the real screen bottom. `position: fixed`
  re-reads the actual viewport bounds instead of trusting a stale
  percentage. Doesn't change where `.menu`/`.sheet`/etc. anchor — a plain
  `position: fixed` ancestor (no `transform`) isn't a containing block for
  its own fixed descendants, so they still resolve against the viewport
  directly, same as before.
- About sheet: a new "About" item at the bottom of the hamburger menu (past
  a separator, after "Archive done") opens a sheet showing the app's
  version and last-updated date — read-only, no editable state, so it's
  just two `.about-value` rows (a plain-text variant of the input-box panel
  look) and a Done button. Shares the elevated z-index tier with the
  Organize and category-edit sheets, since all three are reachable only
  from the hamburger menu and never open at once. See the Version section
  above for what the stamp means and how it's kept current.
- FAB removed, replaced by a muted "＋ New Category" button (`.add-cat-btn`,
  see its own bullet below for the styling) at the bottom of the main scroll list —
  always rendered last, in all three of `renderTodoList()`'s branches
  (populated, "Nothing in Home/Work," and "No categories yet"), so there's
  still a way to create a category from a fully empty state. Opens the
  category edit sheet in create mode with Type defaulting to the **active
  mode** (`catCreateTag = viewMode`) rather than always Work, so the new
  category lands where you're already looking.

  Task creation now only happens from a category's own "+" button, so the
  task sheet's Category field (chip picker, plus its "+ New" chip that used
  to open the category-create flow) is gone entirely — for both create and
  edit. A task's category is fixed by whichever category's "+" opened the
  sheet, or wherever it already lives when editing; the sheet can no longer
  reassign a task to a different category. `renderSheetChips()` (renamed
  `renderSheetTagChips()`) now only builds the Tag picker.

  Category edit sheet's Done button is gone too, matching the task sheet's
  own "no Cancel, backdrop-only dismissal" pattern — nothing in that sheet
  needs an explicit Save/Done since edits already apply live. Delete and
  Archive now split the Finalize row evenly between the two of them. On the
  create flow, the whole Finalize field (label included) hides via
  `#cat-edit-finalize-field`, rather than hiding Delete/Archive individually
  and leaving an empty labeled row behind.
- First-launch seed data: `defaultState()` no longer returns a single empty
  "Home" category — it returns a small worked example (Home, Project A,
  Project B; six tasks exercising the PRIO/WAIT/QUICK tags and a comment).
  The literal is verbatim the `state` section of a JSON backup export, so
  refreshing the seed means exporting a new backup and pasting its `state`
  object in, not hand-authoring an object. `createdAt` stamps are the real
  ones from that export and stay as literals rather than being generated at
  launch — nothing in the UI surfaces them, and generating them would break
  the straight-paste property.

  Only `load()`'s final fallback reaches it, so an existing install and the
  `todos.v1` migration path are both untouched. As before, the default state
  isn't written to localStorage until something calls `save()`, so a launch
  with no edits re-derives the seed each time — indistinguishable from
  having persisted it.
- Category bar's "+" (`.sec-add`) is filled rather than outlined: it was a
  hairline `--acc-dim` ring over a 10%-opacity `--acc-wash`, and now takes
  the flat accent gradient with no border, since the fill defines the edge.
  It deliberately does **not** take the `--acc-glow` drop-shadow that the
  sheet's Save button and the active mode button carry — that halo marks a
  screen's dominant action, and this button sits inline on a category bar
  beside the collapse caret, which is the emphasis level it should match.
  (It shipped with the glow briefly in v26 and read as too loud there.)
- Scheduled tasks (backlog #1, "Future tasks"): a task can be created with a
  date and time instead of joining the list right away. It waits in
  `state.pending` — inside `state`, not a key of its own: the archive was
  split off because it accumulates thousands of rows, while this list is
  small and bounded, so keeping it here means it rides the existing backup
  domain and cascades with category renames for free. One default line in
  `load()` and one in `normalizeState()` cover old installs and old backups.

  Entry shape is `{ text, tag, comment, cat, dueAt, repeat, createdAt }` —
  the fields a task carries, plus `dueAt` and `repeat`. `repeat` is `null`
  for a one-shot, which its promotion then consumes (recurring tasks, below,
  fill it in). `promotePending()` moves every entry whose moment has arrived
  into its category, then asks `nextDueAfter(entry)` whether to re-arm it.
  `createdAt` on the promoted *task* stamps when it joined the list; the
  entry's own `createdAt` records when it was scheduled.

  Promotion runs at launch **and on `visibilitychange`** — an iOS PWA is
  resumed far more often than launched, and can sit suspended for days, so
  launch alone would miss the ordinary case. It also runs after a restore,
  since a backup can carry entries that came due while it sat in the file.
  Entries whose category is missing or archived are **held, not dropped** —
  the category may come back, and silently discarding a scheduled task is
  worse than a row that lingers. Promotion never force-expands the target
  category (that would override a deliberate collapse); the toast names the
  task instead.

  UI: the task sheet gains a **When** field (last, after Tag) with Now /
  Later chips; Later reveals an `<input type="datetime-local">` defaulting to
  tomorrow 08:00, with `min` pinned to now and `color-scheme: dark` so iOS
  renders its wheel dark. `sheetCtx.later` is tracked separately from
  `sheetCtx.when` so clearing the date field just disables Save rather than
  snapping the sheet back to Now mid-edit. The Save button renames itself to
  say what the tap does — "Schedule", or "Add now" when a scheduled task is
  switched back to Now. The field is hidden when editing a task that already
  lives in the list. `sheetCtx.mode` gained a third value, `"pending"`.

  On screen, scheduled tasks are **one collapsible "Scheduled" section below
  the categories**, spanning every category in the current mode, soonest
  first — not inline per category: a scheduled task isn't actionable today,
  and the main screen is for the day's list. Rows carry a clock where a task
  row has its checkbox, and a second line reading `Tomorrow, 08:00 · Project
  A` (near days are named; the row has to name its own category since the
  section spans them). Tapping a row opens the same sheet to edit or delete
  it. The section is skipped entirely when empty, and is also rendered in the
  "Nothing in Home/Work" branch so an entry can't become unreachable. Its
  collapse state shares the per-category `collapsed` map under the
  `SCHEDULED_KEY` sentinel.

  Archiving a category is **blocked while it has scheduled tasks** ("Delete
  its scheduled tasks first"), the same shape of guard as the existing
  unfinished-tasks one — an entry pointing at an archived category can never
  land.
- Recurring tasks: a scheduled entry whose `repeat` holds a pattern re-arms
  its own `dueAt` each time it fires, so **one entry stands in for the whole
  series** — there is no list of future instances anywhere.

  Pattern shape: `{ kind: "days", n }` | `{ kind: "weekly", weekday }` |
  `{ kind: "monthly", day }`, each carrying `until` — a plain local
  `"YYYY-MM-DD"` date, or `null` for no end (optional by decision; most
  recurring chores genuinely don't end). "Every day" and "every two days" are
  **one kind with `n = 1` or `2`**, not two kinds; the sheet offers only those
  two, but nothing in the model objects to `n = 3`.

  `weekday` and `day` are **anchors derived from the first occurrence** when
  the sheet saves — that's why Repeat is a single chip row with no
  sub-pickers, and why the chips read "Weekly (Mon)" / "Monthly (24th)",
  restating themselves when the When date moves. The trade is that "every
  Monday" requires the first occurrence to be a Monday. They're *stored*
  rather than re-read from `dueAt` on each step, and monthly is why: a task
  on the 31st must fire Feb 28 and then **March 31**, so the step has to know
  the intended day-of-month, not the clamped one it last landed on.
  `stepOccurrence()` rebuilds monthly dates from that anchor and clamps to
  the target month's length.

  All stepping is local-calendar arithmetic (`setDate`, never millisecond
  addition), so an 08:00 task stays at 08:00 across a DST boundary instead of
  drifting an hour. `nextDueAfter()` steps until it passes *now* — a daily
  task left for a week comes back as **one** task, not seven — then returns
  `null` if the result is past `until`, which ends the series and drops the
  entry. The end date is inclusive: a task due 08:00 on its `until` day still
  fires. An `until` earlier than the first occurrence needs no validation —
  the first one fires and the pattern is simply spent.

  UI: a **Repeat** chip row and an **Until (optional)** date field, both
  riding with the When picker since a pattern needs a first occurrence to
  anchor to; switching When back to Now clears them, which ends the series
  and puts the task in the list once. The `REPEATS` registry holds one entry
  per option (`label` / `build` / `matches`), so a fifth pattern is one entry
  plus a `stepOccurrence` branch. `sheetCtx.repeatKind` holds an option **id**,
  not a pattern object — the pattern is built at save time from the id plus
  the chosen date, so its anchors can't go stale while the date is still being
  edited. On a Scheduled row, a repeating entry takes a **cycle arrow** in
  place of the clock and names its pattern instead of its next date ("Every
  Monday, 09:30" — for a daily task "Tomorrow" is noise).

  Nothing about backup changed: `repeat` lives inside `state.pending`, and
  `normalizeState()` validates shape, not task fields. Entries written before
  this (a missing or null `repeat`) read as one-shots.
- Category edit sheet gains a **Save button** — `.sheet-btn.primary` (so it's
  gold in Home mode, steel in Work, matching the task sheet's Save), in its own
  full-width row below the Finalize field, reading "Add category" when creating
  and "Save category" when editing. It stays visible in the create flow, where
  the Finalize field above it is hidden. Everything else in the sheet still
  applies live; the button's only work is committing the name and closing —
  the sheet had no explicit commit-and-close action to aim at, which read as
  unfinished next to the task sheet.

  `commitCategoryEditName()` now returns a boolean so the button knows whether
  to close: it stays open on a failed create/rename (which toasts its own
  error) and toasts "Enter a name" for the one silent case, an empty name in
  the create flow. The button's own mousedown blurs the name field first, so
  the create/rename has usually already run by the time its click handler
  fires — calling it again is a no-op, since the committed name then matches
  the field. It's still called explicitly there so a save that never involved
  a blur commits too. This is safe against the Type-chip race documented above
  only because the create path still doesn't re-render the sheet.

  **Enter in the name field takes that same commit-and-close path** (shared as
  `saveCategorySheet()`), so iOS's blue Done key closes the sheet exactly like
  the task sheet's Task field. It previously only blurred, which committed the
  name but left the sheet open. `closeCategoryEditSheet()` therefore blurs the
  name field first, for the reason `closeSheet()` does: a keyboard-driven save
  closes the sheet programmatically without focus ever moving, and iOS leaves
  its keyboard up. Blurring *first* matters — the blur handler's own commit
  then still sees the committed `catEditCat`, so it's a no-op rather than a
  rename against `null` (and on the Archive/Delete paths, where the field can
  be focused when the sheet closes, it's likewise inert).

- Scheduled section demoted out of the category vocabulary. It reused
  `.sec-head` verbatim, so it read as just another category when it's the
  opposite — nothing in it is actionable today. All three cues that say
  "category" come off the header: the lighter-than-ground fill, the accent
  left tick and caret, and the full-size title. What's left is a hairline
  rule with a small caret and the word riding on its right end, in `--ink-2`
  — a section break rather than a thing you own. `order` on the flex children
  puts the rule first and pulls caret and title right, where `.sec-title`'s
  own right-alignment already sits. The box stays **34px tall** even though
  its ink is lighter than a category bar's: with no fill, height is
  invisible, so the tap target loses nothing while the visual weight does.

  The **rows** had to go with it — a muted header over full-weight task slabs
  still left the block looking like today's work. `li.sched` drops the 45°
  hatch and the panel gradient for a flat `rgba(255,255,255,0.02)` wash, and
  its text and icon drop from `--acc-dim` to `--ink-2`, since `--acc-dim` is
  still the accent and the accent is what marks the screen's live surfaces.
  Neutral white rather than an accent-derived token, so the wash stays
  colourless against both palettes. The old blanket `opacity: 0.88` is gone,
  replaced by those per-element colours.

  Four rejected alternatives are in `design/sched-variants.html` (git-ignored,
  local only): a dashed outline, a recessed bar sitting darker than the
  ground, a drained-and-faded copy of the category bar, and the header change
  alone without the row change. The last is the instructive one — it's what
  made clear the slabs were carrying as much of the "this is today's work"
  signal as the bar was.

- "＋ New Category" restyled: it now pops as a **surface**, not a louder line
  — `--acc-wash` fill, the 45° notch, `--acc-dim` text, no border. It went
  1px dashed `--row-line` (v33, too faint to find against the hatched rows) →
  2px dotted `--ink-2` (v34, which read as a placeholder drop-zone, not a
  control) → this. The lesson worth keeping: an outline can only gain
  presence by getting brighter, and a loud outline on a *secondary* control
  reads like an error state. A quiet fill raises presence without raising
  volume.

  The notch is the shape reserved for things you press, which this is; no
  border comes with it, per the notch's own mechanical rule (`clip-path` cuts
  a border away on the diagonals, so the fill has to define the edge). Note
  the text sits at `--acc-dim` over a 10% wash of the same hue — deliberately
  low contrast to keep it under the category "+", and the thing to revisit
  first if it proves hard to read on the phone.

  Rejected alternatives are in `design/addcat-variants.html` (git-ignored,
  local only): a hairline over a white wash, a dashed hairline in `--acc-dim`,
  a borderless accent-wash plate, a notched `--tab-a`/`--tab-b` button, and a
  plain solid `--ink-2` hairline.

- `.sheet-actions` stays **in flow — deliberately not sticky**. The task sheet
  at its longest (Task, Comment, Tag, When, Repeat, Until) is ~773px of
  content against the sheet's 88%-of-viewport cap (~714px on an 812pt
  iPhone), so Save sits ~59px below the fold there. v29 pinned the row to fix
  that and it was a mistake: a sticky bar paints over whatever scrolls under
  it, so the second row of Repeat chips was sliced in half by the button from
  the moment the sheet opened (reverted in v30). A sheet that scrolls is
  ordinary; a control cut in half looks broken. Note the shape of the
  problem before reaching for a pinned footer again — only the *longest* form
  overflows at all, and the common one (Repeat visible, Until hidden, because
  "Once" is selected) comes to 676px and doesn't scroll.

## Future screens (long-term plan)

Two additional screens will eventually live at the same layer as the mode
switching: an **Archive viewer**, and a **Financials tracker** (morning
review of weekly spend vs. budget, fed by a side project that
auto-categorizes transactions, with manual categorization for the rest).
Financials details are still to be talked through; this section records the
architectural decisions made so the app can grow into them without rework.

An **Exercise mode** was planned here too — a contributions-style calendar,
performance metrics, and per-discipline workout plans. It came out on
2026-08-23: it's being built as **its own separate app**, not a screen in
this one. Nothing about the decisions below depended on it (the nav registry
and the multi-domain backup envelope are both additive by design), so they
stand as written.

### Navigation: "mode" is promoted to "screen"

Home/Work today is a two-value *filter* over one screen. The new screens are
not filters — they're different renderers over different data. The agreed
shape is two-level:

- **`screen`** ∈ `todos` | `archive` | `finance` — which
  renderer owns the scroll area.
- **`viewMode`** ∈ `home` | `work` — stays exactly as it is, but scoped as a
  sub-state of the todos screen (and plausibly reusable as a filter inside
  the Archive viewer, since archived tasks carry categories that carry
  tags). The "always home or work, never cleared" rule is unchanged.

`render()` becomes a dispatcher over a screen registry
(`{ id, icon, render, palette }`); the top bar renders its buttons from
that registry. Adding a screen later is then additive. The refactor can be
done with only the todos screen registered — that's the point. Top bar goes
from 3 buttons to 5 (Work, Home, Archive, Finance, menu) — icon-only at
iPhone width is tight; check on the real device before committing them all
to the bar.

Theming already cooperates: each screen can stamp its own `data-mode` and
get its own token palette. The fixed Home/Work category-tick colours stay
fixed by design.

### Storage and backup

- One localStorage key per domain, as today: `todos.v2`, `todos.v2.archive`,
  later `finance.v1`. Rationale unchanged from the archive split: checking a
  todo checkbox must never re-serialize a larger domain's history. iOS
  Safari's ~5MB quota is the ceiling; finance stores a bounded snapshot
  (current week + a few weeks), not an ever-growing ledger.
- **Backup envelope goes multi-domain** (do this *before* any new domain
  ships): optional top-level sections — `state`/`archive` today, `finance`
  later. Old backups stay valid. Restore becomes **section-scoped**: only
  domains present in the file are replaced. Otherwise, restoring a
  pre-finance backup would silently wipe that domain's data — a data-loss
  bug designed out before it can exist.

### Per-screen notes

- **Archive viewer:** the where-does-it-live question is answered — its own
  screen. No model change needed. Sanity-check rendering ~5000 entries on
  an iPhone (grouped, collapsed by default, same trick as the main screen).
- **Financials:** external data, but within the no-backend constraint —
  *read-only pull is not sync*. Fetch on launch/screen-open when online,
  cache the latest snapshot in localStorage with an `asOf` timestamp,
  render the cached snapshot with a staleness note offline (the
  morning-train review must work without signal). Requirements on the side
  project: **HTTPS** endpoint (the PWA is on GitHub Pages over HTTPS, so
  HTTP is blocked as mixed content), **CORS** headers allowing the Pages
  origin, and — the one thing expensive to retrofit — **stable, unique
  transaction IDs**, because manual category corrections live in a local
  map keyed by transaction ID, overlaid on whatever the API returns, and
  must survive re-fetches. Email push can't be automated (a PWA can't read
  mail); paste-a-JSON-blob import is the fallback, not the primary.
  Corrections flowing *back* to the side project would be two-way sync —
  deferred; the app's overlay is authoritative for display.

### Single-file pressure

Two more screens plus recurring tasks could push the file past 5,000 lines.
Decision: **stay single-file** with strict internal layout — one
clearly-fenced section per domain (state + renderer + handlers). Splitting
would cost offline robustness (no service worker; every extra file is
another HTTP-cache miss on a train). Revisit only if it becomes painful,
together with the open `todo.js` extraction question under Testing.

### Sequencing

Only two things must precede the screens: the multi-domain backup envelope
(**done**) and the nav/dispatcher refactor (**done**) — see their bullets
in Current state. The screens are now unblocked. Archive viewer
is the natural first screen — pure UI over existing data, exercising the
new nav with zero model risk. Decisions still owed before building: whether
all four nav entries fit the top bar on the real phone, and API-vs-email for
the side project.

## Backlog (build individually, in priority order)

1. **Archive viewer** — a way to browse/search archived (completed) tasks
   in-app, **grouped by category and sorted by date**. The data is already
   there: every entry `archiveDone()`/`archiveCategory()` writes carries
   `category`, `createdAt`, `completedAt` and `archivedAt`, so no model
   change is needed, and category renames no longer fracture a category's
   history (see Current state). Today the archive is write-only, populated
   by "Archive done" and "Archive category," and only inspectable via JSON
   export. Where it lives is decided — its own screen at the nav layer (see
   Future screens); build after the nav/dispatcher refactor.

## Testing (planned)

- Unit logic: **Vitest + jsdom**.
- E2E: **Playwright**.
- Prerequisite under consideration: extract pure logic into a `todo.js` module
  so it's unit-testable. This is a single-file-constraint decision — raise it
  before doing it.

## Local dev

- Quick check: open `index.html` directly in a browser.
- Serve (closer to PWA behavior): `python -m http.server` (or `py -m
  http.server` on Windows; `python3` on macOS/Linux) then open the printed
  localhost URL.
- Deploy: commit and `git push` — GitHub Pages serves the pushed `index.html`.
