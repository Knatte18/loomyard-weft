---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/01-entry-schema.md:89
    summary: The tier rule's done exemption differs by direction - direction 2 also skips done dependents, which Decision 7 never exempts, so a demotion can leave a state that a later unrelated write to the done dependent is refused for
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/04-board-json-store.md:225
    summary: Card 4 asserts a read leaves the board directory byte-identical, but its own Load reads each legacy file under that file's .swaplock, and state.ReadJSON creates the lock file
  - id: F3
    severity: LOW
    location: _lyx/plan/04-board-json-store.md:227
    summary: The slug-in-both-files refusal is tested only on a write, though the acceptance criterion says the load refuses, and no test covers a read that folds a legacy done mark over an existing board.json while persisting nothing
  - id: F4
    severity: LOW
    location: _lyx/plan/05-board-verbs.md:257
    summary: RetireLegacy must delete files after the save and before render and sync, but card 4's boardCriticalSection runs load, mutate, save, render and sync as one fixed sequence with no step between save and render
  - id: F5
    severity: LOW
    location: _lyx/plan/03-legacy-migration.md:154
    summary: Card 3 declares the legacy file-name constants in legacy.go, but board.go already declares tasksFile and notesFile and no card removes them, so card 3 either fails to compile or leaves two declarers
  - id: F6
    severity: LOW
    location: _lyx/plan/04-board-json-store.md:238
    summary: Card 4's ImpactSummary never concludes on the blast radius of the deleted Note methods, taskToUpsertFields and the changed Render and RenderToDisk signatures, or on the ideengine tests that seed a legacy tasks.json
  - id: F7
    severity: LOW
    location: _lyx/plan/06-render-roadmap.md:289
    summary: The semantic-line-break constraint covers renderer-generated markdown and every markdown file written, but neither card 6 (intro and meaning lines) nor card 10 (overview prose) carries it
  - id: F8
    severity: NIT
    location: _lyx/plan/06-render-roadmap.md:316
    summary: Card 6's ImpactSummary says list JSON order changes but does not name who consumes that order
---

### [MEDIUM] Tier rule applies the done exemption differently in each direction

**Location:** `_lyx/plan/01-entry-schema.md:89`

**Issue:**
Decision 7 exempts exactly one thing: a dependency whose status is `done`.
Card 1's direction 1 follows that: every non-`done` dependency of the incoming entry is compared, and a `done` incoming entry is still checked against its own dependencies.
Direction 2 adds a second exemption, "every non-`done` entry depending on the incoming entry", so a `done` dependent is never compared there.
The same pair is therefore judged differently depending on which side gets written.
Here is a concrete sequence.
A is tier 1 and `done`, and depends on B, which is tier 1.
`upsert` B with `tier: 3` is accepted, because direction 2 skips the `done` dependent A.
A later `upsert` of A that only fixes its title, or a `merge` or `set-deps` touching A, is then refused by direction 1, naming B, although that write changes nothing about the edge.
I traced the other write paths the focus names: upsert tier changes, `upsert-batch` (projected snapshot), `merge` (`validateWrite` over the projection, then `UpsertTask` and `SetStatus`), `set-deps`, card 2's `Promote`, and a `set-status` reopen through card 1's `SetStatus` check.
Each one reaches `validateWrite`, or the tier rule inside `SetStatus`, in both directions.
The rule's reach is correct.
Only the exemption is asymmetric.

**Fix:**
State the rule as one predicate per edge: a pair (dependent D, dependency P) is compared iff P is not `done`, and D's status never matters.
Direction 2 then reads: every entry depending on the incoming entry must have a tier ≥ the incoming entry's, unless the incoming entry is `done`.
Add a test for the sequence above: demoting a dependency of a `done` dependent is refused.

### [MEDIUM] "A read leaves the directory byte-identical" contradicts locked legacy reads

**Location:** `_lyx/plan/04-board-json-store.md:225` (and `:200`–`:202`)

**Issue:**
Card 4's `Load` reads "both legacy files (each under its own `.swaplock`, as the old binary wrote them)".
`state.ReadJSON` calls `lock.AcquireReadLock`, which is `gofrs/flock` opening the lock path with `O_CREATE`.
The repo already records this effect: `internal/fabricengine/warpbinding_reconcile_integration_test.go:403` says "a single ordinary board read left tasks.json.swaplock at the board root".
So a read on a legacy-only board that was seeded with just `tasks.json`/`notes.json` creates `tasks.json.swaplock`.
It also creates `notes.json.swaplock` even when `notes.json` is absent, because the card reads both files unconditionally once either one exists.
The `board_test.go` assertion "a read leaves the directory byte-identical" would fail as specified.
An implementer would then either bend `Load` to read the files without their lock or weaken the test on the spot.
Decision 8 and the acceptance criterion mean persisting no data ("writes nothing" next to "the first write creates `board.json`").
The lock files are gitignored machine-local scratch.

**Fix:**
Make the read assertion compare the data files: `tasks.json` and `notes.json` stay byte-identical, `board.json` stays absent, and no README or design doc is written.
Say explicitly that `*.swaplock` files are not compared.
In `Load`, read each legacy file only when that file exists, not both whenever either exists.

### [LOW] Read-path coverage for the slug refusal and the fold

**Location:** `_lyx/plan/04-board-json-store.md:227`

**Issue:**
The acceptance criterion says "A slug in both legacy files refuses the load".
Card 3 tests the pure error, and card 4 tests only that the write is refused.
A read (`GetTask`/`ListTasksBrief`) on such a board is not tested to refuse.
Card 4's `Load` produces that refusal by construction, but nothing pins it.
On the fold side, every fold test with a `board.json` present goes through a write.
No test covers the read-path half of Decision 9: a read over `board.json` plus a legacy file with a new `done` mark returns the entry `done` and leaves `board.json` unchanged.

**Fix:**
Add both cases to card 4's `board_test.go` list: the read refused with the slug named, and the fold-on-read returning `done` with `board.json` byte-identical.

### [LOW] RetireLegacy's ordering is not expressible through card 4's helper

**Location:** `_lyx/plan/05-board-verbs.md:257`

**Issue:**
Card 4 makes `boardCriticalSection` the one locked write helper: lock, load, mutate, save `board.json`, render, spawn sync.
Card 5 says each write goes through it, and it also requires RetireLegacy to save before deleting and to delete before render and sync, as Decision 10 orders.
If the deletion happens inside the mutate callback, it comes before the save, which breaks the crash-safety order the card states.
If it happens after the helper returns, it falls outside `board.lock` and after the detached sync was spawned, so that sync may commit `board.json` without the deletions.

**Fix:**
Name the mechanism.
For example, `boardCriticalSection` takes an optional after-save step that runs under the lock before render and sync, and RetireLegacy alone passes one that deletes the legacy files and their swaplocks.

### [LOW] Legacy file-name constants get a second declarer

**Location:** `_lyx/plan/03-legacy-migration.md:154`

**Issue:**
`internal/boardengine/board.go:53`–`56` already declares `tasksFile = "tasks.json"` and `notesFile = "notes.json"`.
Card 3 declares "the legacy file names" again in `legacy.go` but does not edit `board.go`.
Card 4 rewrites `board.go` without listing those constants among what it deletes.
If card 3 reuses the names, it fails to compile on its own commit.
If it picks new names, the old constants linger as a second declarer of the same literals, since Go accepts unused package-level constants.

**Fix:**
Card 3 declares distinctly named constants (`legacyTasksFile`, `legacyNotesFile`), and card 4's `board.go` list deletes `tasksFile`/`notesFile`, so `legacy.go` is the only declarer.

### [LOW] Card 4's ImpactSummary misses its largest blast radius

**Location:** `_lyx/plan/04-board-json-store.md:238`

**Issue:**
The summary says external callers keep their method names.
It does not conclude on what the card deletes or re-signs: nine `*Note*` methods, `taskToUpsertFields`, and the one-list `Render`/`RenderToDisk`.
Those have no caller outside `internal/boardengine` and `internal/boardcli`.
It also skips the `internal/ideengine` tests that seed a legacy `tasks.json` and keep passing only through migration on load.

**Fix:**
Rewrite it as a conclusion that covers those facts.

### [LOW] Semantic-line-break constraint not carried for generated markdown and overview prose

**Location:** `_lyx/plan/06-render-roadmap.md:289`; `_lyx/plan/10-overview-doc.md:426`

**Issue:**
The record's constraint "Markdown written in this task, and the markdown the renderer generates, follows semantic line breaks" is carried only by card 9, for the skill.
Card 6 fixes the README's intro and meaning lines without saying they follow the rule.
Card 10 writes new prose in `docs/overview.md` without saying it either.

**Fix:**
Card 6: the generated intro and meaning lines are each a single sentence on its own line, and the renderer never hard-wraps.
Card 10: the new overview prose follows semantic line breaks.

### [NIT] Card 6's ImpactSummary leaves the order change ungrounded

**Location:** `_lyx/plan/06-render-roadmap.md:316`

**Issue:**
"`list` JSON order changes to README order" states the change but not its reach.
`ListTasksBrief`'s only production caller is `internal/boardcli`'s `list`, and no stencil reads `lyx board list` output.

**Fix:**
Add that conclusion.

## Verdicts on the focus directives

- **Card 1 against Decision 7:** every write path named reaches the rule in both directions.
  The defect is the asymmetric exemption, F1.
- **Cards 3–5 against Decisions 8–10:**
  every read path (`GetTask`, `ListTasksBrief`, `ListTasksFull`, `Find`) goes through `Load`, which migrates and folds in memory and never writes.
  RetireLegacy's fold, save, delete, render and sync order matches Decision 10.
  The gaps are the test coverage (F3), the lock-file assertion (F2), and the mechanism for the order (F4).
- **Whole-plan fidelity:** every Decision and acceptance criterion maps to a card.
  The constraints on "Bolt", hidden-alias `Short`, and hubforge are carried by cards 9 and 7 and by the shared decisions.
  The semantic-line-break constraint is only partly carried (F7).
  No card adds unlicensed scope: the `version` refusal, the `SetStatus` reopen check, and the `find`-without-argument refusal each follow from a Decision.
- **Granularity:** not a defect.
  Card 1's parts are compile-coupled: removing `Deferred` forces the deferred rules out, and renaming `Type` forces `battencli`'s read to change.
  Card 4's facade collapse is forced by `Store` moving to the directory.
  The 2/5 split separates in-memory logic from the locked facade, and each half is testable on its own.
  Several cards edit `store.go` and `render.go` at sequential points, which the card model allows.
- **ImpactSummary:** cards 1, 2, 5, 7 and 8 carry real conclusions.
  Card 4's falls short (F6), and card 6's is thin (F8).
