---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3 and the acceptance criterion on a failing regeneration
    summary: The criterion has the prime spawn return an unresolvable-prime failure as an error, but Spawn degrades to the bare folder before it reaches any workspace write, and D3 says it keeps returning its errors as today.
    fix: In D3, state that an unresolvable prime keeps degrading the spawn to the bare folder with its Warn, as `resolveSpawnTarget` does today, and that only failures past prime resolution (unreadable settings, lock timeout, failed write) are returned as the spawn's error; then reword the criterion's last clause to "the prime spawn returns an unreadable-settings, lock-timeout or failed-write failure as an error, and an unresolvable prime opens the bare folder".
    origin: lens:interactions
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3, D4 Bound and D5
    summary: D5's in-place write truncates before writing, so a write that fails partway leaves an empty or partial file, which contradicts D3's writes-nothing claim, D4's stale-list-only Bound and the failed-write criterion.
    fix: Keep the in-place write but make the claims match it. In D3, change "on any of them it writes nothing" to "on any failure before the write it writes nothing; a failed write may leave the file truncated or partial". In D4's Bound, add "or a truncated file, which the next regeneration rewrites because its bytes differ". In the failing-regeneration criterion, drop "failed write" from the writes-nothing list and add a criterion that a regeneration after a truncated file rewrites it whole.
    origin: lens:walkthrough
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 and the criterion that a hub with no `_board` omits `Bolt`
    summary: The lock lives under `<hub>/_board/.lyx`, so on a hub with no `_board` taking it either fails every regeneration or creates `_board` and lists `Bolt`, and D5 says neither which nor how the lock is placed and timed.
    fix: "In D5, name the lock as `<hub>/_board/.lyx/hub-workspace.lock` and state that on a hub with no `_board` the regeneration takes no lock, creates no directory and lists no Bolt; then state the wait as a fixed value, for example 5 seconds, and add a criterion that a regeneration on a hub with no `_board` leaves `_board` absent."
    origin: lens:existing-paths
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D7 Warn sentence and the criterion that after the sweep no Warn is logged
    summary: The sweep keeps foreign entries and their `_portals` directory, so a Warn keyed on `_portals` existing fires forever, naming a reconcile that cannot clear it, and D7 never defines what counts as a retired artifact existing.
    fix: In D7, replace "While either retired artifact exists" with "While a fabric-shaped portal link, judged by the same target-path rule as the sweep, or an entry at the retired workspace path exists"; then extend the Warn criterion with "a hub whose `_portals` holds only a foreign entry logs no Warn".
    origin: lens:walkthrough
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D4 last sentence and D7 foreign-entry sentence
    summary: No regenerating verb's result has a hub-level detail field, Add and Remove return zero results on the error exits D4 singles out, and ReconcileResult has only per-pair and warp-binding details, so neither D4's carried detail nor D7's foreign-entry detail has a place to land.
    fix: In D4, replace "and carried as a detail in the verb's result where the result type has one" with "and is not carried in the verb's result". In D7, replace "with a detail naming it" with "with a hub-level detail naming it, in a new `HubDetails []string` field on `ReconcileResult` that `runReconcile`'s envelope emits"; keep the foreign-entry criterion as written.
    origin: lens:existing-paths
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D2 sentence on skipping an unresolvable worktree entry
    summary: The sentence reads as a fact about the listing, but `CodeWorktrees` returns an error on an unresolvable entry today and its other callers rely on that, so the plan writer cannot tell whether it changes.
    fix: In D2, reword to "The regeneration's listing skips a worktree entry it cannot resolve, logging it at Debug, so one half-created pair never drops the others; `CodeWorktrees` keeps returning that failure for its other callers."
    origin: lens:bound-breaker
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: The Bound says the next regenerating verb or prime spawn rewrites a stale list, which holds only for a transient cause; an unresolvable prime or unreadable settings fails every later regeneration the same way, and the Warn names a reconcile that fails identically.
    fix: "Append to D4's Bound: a persistent cause, such as an unresolvable prime or unreadable prime settings, keeps the list stale and the Warn repeating until the operator fixes the cause; the Warn names the failure beside `lyx fabric reconcile`."
    origin: lens:bound-breaker
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 Reconcile sentence and D7
    summary: The record never orders Reconcile's regeneration against its sweep, so a regeneration run before the sweep warns the operator to run the very reconcile that is running.
    fix: In D7, add "Reconcile sweeps before it regenerates, so its own regeneration logs no retired-artifact Warn for anything the sweep removed."
    origin: lens:interactions
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 first sentence against D7
    summary: D6 retires the portal helpers in `portals.go` with only the name constant spared, while D7's sweep needs the portal target shape, a link-ownership check and the empty-ancestor pruning those helpers carry.
    fix: In D6, change "the portal helpers in `portals.go` ... go" to "the portal create, remove and restore helpers go; the sweep keeps what it needs of the portal target shape and ownership check, moved beside the D7 sweep".
    origin: lens:interactions
  - id: F10
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D7 second bullet and Bound
    summary: D7 covers a file or a planted link at the retired workspace path but not a directory, and the Bound's one-entry wording could be read as licensing a recursive delete there.
    fix: In D7's second bullet, add "a directory at that path is kept with a detail naming it and never removed", and add that case to the planted-symlink criterion.
    origin: lens:bound-breaker
  - id: F11
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D2 prime display name and D4 verb list
    summary: D2 names the prime after its checked-out branch, but `Checkout` on the prime changes that branch and is not a regenerating verb, so the first folder's display name goes stale with no failure and no Warn.
    fix: "Pick one and state it in D4: either add `Checkout` to the regenerating verbs, or write that a branch change in the prime leaves the prime's display name stale until the next regenerating verb or prime spawn."
    origin: handler
  - id: F12
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md acceptance criterion on an unchanged folder list
    summary: D3 compares the whole file's bytes, settings included, so an unchanged folder list with changed prime settings does rewrite, and the criterion's condition is narrower than D3's rule.
    fix: Reword the criterion to "An unchanged file, folder list and settings both, rewrites nothing and records no mutation; a changed one records `KindFileWritten`."
    origin: lens:interactions
---

### [MEDIUM] Prime spawn criterion contradicts Spawn's degrade path

**Location:** `decision-record.md` D3; the acceptance criterion "A regeneration that fails ...".

**Issue:** `Spawn` resolves the prime in `resolveSpawnTarget` before anything else (`internal/ideengine/spawn.go:36-37`, `:145-157`).
A failure there is logged at Warn and `primeResolved` comes back false.
The spawn then opens the bare folder and returns nil, so the shared regeneration function is never called.
D3's "keeps returning its errors as today" matches this.
The criterion's "the prime spawn returns the same failure as an error" requires an unresolvable prime to be returned instead.
All four lenses raised this.

### [MEDIUM] In-place write breaks "writes nothing" and the D4 Bound

**Location:** `decision-record.md` D3, D4 Bound, D5.

**Issue:** `os.Root.WriteFile` truncates before it writes (`internal/fabricengine/hubworkspace_write.go:48`).
A failed or killed write therefore leaves an empty or partial file.
That makes D3's "on any of them it writes nothing" false for a failed write.
It also makes D4's claim that only a stale folder list gets through false.
The next regeneration's byte comparison does repair the file, so in-place writing can stay; the claims and criteria have to say so.
Found by walkthrough and bound-breaker.

### [MEDIUM] Lock placement on a hub with no `_board`

**Location:** `decision-record.md` D5; the criterion "a hub with no `_board` omits `Bolt`".

**Issue:** The board's never-tracked state dir is `HubScratchDir`, `<hub>/_board/.lyx` (`internal/fabricengine/junctionnames.go:167-168`).
The existing hub lock creates its parent first (`internal/hubreconcile/walk.go:44-47`).
`lock.AcquireWriteLockWithin` creates no directories.
Either the regeneration fails on such a hub, or it creates `_board`, and then `Bolt` is listed and `looksLikeHub` sees the new directory.
D5 also leaves the lock file name and the wait value open, so the plan writer cannot place the lock or size its timeout.
Raised by every lens except walkthrough's list, which raised it too.

### [MEDIUM] The retired-artifact Warn trigger is undefined

**Location:** `decision-record.md` D7; the criterion "While a retired artifact exists ...; after the sweep it does not".

**Issue:** The sweep keeps foreign entries and their directory under `_portals`.
If "retired artifact exists" means `_portals` exists, a hub with one foreign entry warns on every regeneration forever.
The Warn names `lyx fabric reconcile`, which keeps that entry.
Only a reading keyed on fabric-shaped links, judged by the sweep's own target-path rule, meets the criterion, and D7 does not state it.

### [LOW] Detail fields that do not exist

**Location:** `decision-record.md` D4 last sentence; D7's foreign-entry sentence.

**Issue:** None of `AddResult`, `RemoveResult`, `PruneResult` or `CloneResult` has a hub-level detail field (`add.go:43-47`, `remove.go:71-88`, `prune.go:71-78`, `clone.go:72-87`).
`ReconcileResult` has only per-pair `Detail` and `WarpBindingDetail` (`reconcile.go:120-153`), and `runReconcile` hand-builds its envelope.
`Add` returns `AddResult{}` after `rollbackAdd` (`add.go:298`, `:313`), so a detail would be dropped on exactly the exits D4 names.
Both clauses leave the plan writer guessing whether to add a field.

### [LOW] D2's skip is written as a fact about `CodeWorktrees`

**Location:** `decision-record.md` D2.

**Issue:** `CodeWorktrees` returns an error on an entry it cannot resolve (`internal/fabricengine/codeworktrees.go:39-42`).
Its other callers, in `hubreconcile`, `configcli` and `reportcli`, would silently lose worktrees if that changed.
The record does not say whether the regeneration's listing is a tolerant path of its own.

### [LOW] D4 Bound overstates self-healing

**Location:** `decision-record.md` D4 Bound.

**Issue:** An unresolvable prime or unreadable prime settings fail every later regeneration the same way, Reconcile's included.
For those causes the list stays stale and the Warn repeats until the cause is fixed.

### [LOW] Reconcile's sweep and regeneration are unordered

**Location:** `decision-record.md` D4, D7.

**Issue:** If Reconcile regenerates before it sweeps, the regeneration sees the retired artifacts.
It then logs a Warn telling the operator to run the reconcile that is running.

### [LOW] D6 retires helpers D7 reuses

**Location:** `decision-record.md` D6, D7.

**Issue:** D6 deletes "the portal helpers in `portals.go`", sparing only the name constant.
D7's sweep needs the `portalTarget` shape, the `ownedWiredJunction` ownership check and `pruneEmptyAncestors`, all used by `removePortal` (`internal/fabricengine/portals.go:42-44`, `:103-122`).

### [LOW] A directory at the retired workspace path is unbounded

**Location:** `decision-record.md` D7 second bullet and Bound.

**Issue:** D7 covers a regular file and a planted link at `<hub>/_launchers/<AnchorRel>/<prime>.code-workspace`, but not a directory.
The Bound's "the one entry at the retired workspace path" does not say whether a directory there is removed, refused or kept.
Under the attack-surface rule, that is a destructive path with no stated bound.

### [LOW] A prime branch change leaves the first folder's name stale

**Location:** `decision-record.md` D2, D4.

**Issue:** D2 names the prime after its checked-out branch.
`Topology.Checkout` (`internal/fabricengine/checkout.go:39`) switches a worktree's branch, the prime's included, and is not in D4's list.
After a checkout in the prime, the file names the old branch until some other verb regenerates it.
No failure occurs and no Warn is logged, so D4's Bound does not cover the case.

### [NIT] Unchanged-folder-list criterion narrower than D3

**Location:** `decision-record.md` acceptance criterion "An unchanged folder list ...".

**Issue:** D3 rewrites when the file's bytes change, and the bytes include the spliced prime settings (`internal/vscode/workspace.go:22-26`).

## Rejected

- None of the fork findings was rejected; duplicates across lenses were merged into the single finding that carries each one's `origin`.
- Relocation check, by the completeness-before-leanness test: the Rationale paragraphs carry no deliberation narrative that belongs in `support-log.md`, and D2's note that duplicate display names are left to VS Code is a requirement the plan writer needs, so no relocation finding is raised.
- The D3 no-import-cycle claim, the weft-worktrees-never-listed premise and the local-pair and forked-pair premises were checked against the code and hold, so no finding is raised on them.
- The two declared PATTERN changes cover the new write and the retirement; no Decision contradicts a listed Constraint, so no PATTERN finding is raised.
