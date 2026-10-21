---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3, second and fifth bullets and the Bound paragraph
    summary: The worktree strip's former-value drop sits under D3's board-marker gate, so an unmarked worktree stripped after the marker exists keeps old neutral values such as discussion_lenses empty as overrides over the new template, which breaks acceptance criteria 1 and 3.
    fix: In D3, replace the second bullet with a rule gated on the worktree alone - every pending strip of a worktree without its own sparse marker, in the walk and in the single-pair call (hubreconcile Options.Pair), drops a unit equal to a former value, whether or not the board marker exists; its bound is the worktree's own sparse marker, written after the strip, so each worktree is stripped of former values once. Move the opening sentence's board-marker gate so it governs only the base strip, and add an acceptance criterion - a hub cloned from a repo whose tracked worktree files are full and unmarked at the pre-change template values resolves every D1 value after the clone's walk, and a pair skipped as mid-merge in the migrating walk drops its former values in the next walk.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md Open risks, first bullet; D3, last bullet
    summary: The older-binary refill is bounded to a walk, but an older binary also refills a sparse base through lyx config --layer default --set, lyx config reconcile --apply and lyx fabric reconcile regardless of the stamp, and the named way forward only prevents a refill while nothing repairs one, since the strip is marker-gated and walk-only and the stamp fast path never reads the marker.
    fix: Rewrite the Open risks bullet to name every refill path of an older binary (the walk, lyx fabric reconcile, lyx config reconcile --apply --layer default, and lyx config <module> --layer default --set, which rewrites the module's base from the old template), and state the repair - either add to D3 that an absent board marker makes a stamp naming the running build stale, as an absent repo default does, so removing the marker by hand with a board commit makes the next start verb's walk strip again, with that recovery named in the risk; or state plainly that a refilled base is repaired only by hand-editing its files and accept it as an auto-mode assumption.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/discussion/decision-record.md D1, loom bullet
    summary: D1 downgrades the Webster-Review guard for every hub, webster_max_bounces from review_max_bounces 3 to 1 and webster_auto_accept to true, and discussion_max_bounces from 3 to 2, with no stated bound, against the spec rule that a Decision weakening a guard states what it lets through and what bounds it.
    fix: Add one paragraph to D1 stating that the new defaults give Webster-Review one judged round and accept a harvested round at the spent budget instead of the soft-cap rule, and Discussion-Review two rounds, so a change reviewed by fewer rounds can land; state the bound - the operator's split chose these values, Merge-Review, the verify gates and the plan and discussion fans still run, every value is a loom key a hub pins back with lyx config loom --layer default --set, and existing hubs reach them only through D3's strip of units equal to the former values.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md, missing PATTERN changes section; support-log.md Question ledger Q9
    summary: D4 sets a cross-cutting invariant, that every writer of a hub base writes it sparse and that only the walk's one-time strip drops base units, which a later writer breaks silently because D3 never repairs it, yet the record adds no PATTERN entry and Q9 answered only whether an existing entry states the base is full.
    fix: "Add a PATTERN changes section to decision-record.md after Acceptance criteria with the bullet - `PATTERN-sparse-hub-base` — adds: Writing a hub's base file (a hub-wide module's hub file or a per-worktree repo default, not seed-only): only the units the hub tuned, never a template fill; only hubreconcile's one-time walk strip drops a unit. Then update Q9 in support-log.md to record the add."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4, prime-seed bullet, against D3's last bullet and acceptance criterion 5
    summary: D4's prime seed drops units equal to a former value on every seed, with no marker gate, so a base reseeded from the prime after migration loses a deliberate value equal to a former value that D3 says stays.
    fix: In D4's prime-seed bullet, drop the former values from the comparison once the board marker exists - a base seeded from the prime's copy holds that copy's units that differ from the current template and, only while the board marker is absent, from the former values.
  - id: F6
    severity: LOW
    class: decision
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4, key-move bullet; Notes for the plan writer, moveKeys bullet
    summary: D4 leaves open whether a key move into a sparse base still overwrites a target holding the template default, which D2 makes a pin once migrated, drops a moved unit equal to the template even when that unit is a pin, and does not say which step writes the header-only base when the move leaves the target with no unit, since moveFile.write deletes an emptied file.
    fix: Extend D4's key-move bullet - at a base, a target unit equal to the template is overwritten by the moved value only while the board marker is absent and is kept, as a pin, once it exists; a moved unit equal to the template is dropped only while the marker is absent; a target the move leaves with no unit is written as the header-only base, never deleted.
  - id: F7
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Auto-mode assumptions
    summary: The assumptions cover a template-equal unit and the deploy-to-walk window, but not that a former value a hub set on purpose before the deploy, such as driver_kind llm or kickstart_pack true, is dropped by the migration.
    fix: Add one assumption bullet - a base unit equal to a former value is dropped at migration even when the hub set it on purpose before the deploy, since a full base cannot tell a seeded value from a tuned one; the hub pins it again with lyx config <module> --layer default --set once migrated.
  - id: F8
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3, open-map bullet; D1, batcher bullet
    summary: The batcher.profiles former value is the whole map, so a hub whose profiles hold a profile of its own or an older generation of the map, after the batcher Migrate hook, keeps budget 450000 and never receives D1's budget, and no Open risk says so.
    fix: Add an Open risks bullet - a hub whose batcher.profiles differs from both the current and the former whole map, by an own profile or an older template generation, keeps its profiles map, so the cautious budget stays at its old value; the way forward is lyx config batcher --layer default --set profiles.cautious.budget=200000 on that hub.
  - id: F9
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4, editor bullet, against D2's header rule
    summary: The editor scaffolds an absent base empty, with no header comment, so a base it creates lacks the header D2 requires, and D4's empty it by keeping the header has no header to keep.
    fix: In D4's editor bullet, say the editor scaffolds an absent hub base with D2's header comment and no unit, so the header is present and a save that keeps it leaves a key-less base.
---

### [BLOCKING] The worktree former-value drop is tied to the board marker

**Location:** `_lyx/discussion/decision-record.md` D3, second and fifth bullets and the Bound paragraph.

**Issue:** D3 opens with "The strip runs only in `hubreconcile`'s walk … and only while the board's base-sparse marker is absent", and its second bullet adds the former-value drop to "the pending strip of a worktree without the `sparse` marker" "in the same walk".
Read that way, every unmarked worktree stripped once the board marker exists keeps old neutral values as overrides, since `stripToSparse` (`internal/configsync/configsync.go:479`) compares only with the layers below, which now resolve the new template.
Three concrete paths reach it:

- `lyx fabric clone` writes the board marker in its own pass (`internal/fabriccli/clone.go:57`), and its `Ensure` walk (`clone.go:130`) then migrates an unmarked prime. A repo whose tracked worktree files are full at the old template values, such as one used standalone, keeps `discussion_lenses: ""`, and D4's prime seed leaves the repo default empty, so the new hub runs neutral, against acceptance criterion 1.
- A pair skipped as mid-merge or incomplete in the migrating walk (`internal/hubreconcile/walk.go` `reconcileWorktree`) is stripped by the next walk, as `hubreconcile`'s doc promises, but that walk sees the marker.
- The single-pair call `Options.Pair` (`internal/battencli/wire.go:310`) runs `reconcileWorktree` outside the walk.

### [MEDIUM] The older-binary refill is under-bounded and has no repair

**Location:** `_lyx/discussion/decision-record.md` Open risks, first bullet; D3, last bullet.

**Issue:** The risk names only an older binary "that walks the shared board", which happens only when that machine's stamp goes stale.
An older binary also refills a sparse base whatever the stamp says: `lyx config reconcile --apply --layer default` (`internal/configcli/hubwide.go:211`) and `lyx fabric reconcile` (`internal/fabriccli/fabric.go:706`) run the old `ReconcileHubWideAt`, which adds every template key, and `--set` on the base goes through `yamlengine.SetValues` over the template (`internal/configengine/set.go:75`).
`--set` on the base is how the orch tuned quarry.
Once refilled with the marker present, D3 never strips the base again.
`ensure` returns on a current stamp without reading the marker (`internal/hubreconcile/walk.go:34-38`), and `repoDefaultAbsent` checks only module files, so removing the marker by hand does not reach the strip either.
`update-plugins.sh` on every machine prevents a refill but repairs none.
D3 and acceptance criterion 5 rest on "marker present means the base is sparse", a state the record says can be broken.

### [MEDIUM] D1 weakens review guards with no stated bound

**Location:** `_lyx/discussion/decision-record.md` D1, loom bullet.

**Issue:** `webster_max_bounces: 1` replaces null, which takes `review_max_bounces: 3` (`internal/loomengine/template.yaml:55,60`).
`webster_auto_accept: true` makes "the Webster-Review perch accept a harvested round at a spent budget instead of running the soft-cap rule" (`template.yaml:66`).
`discussion_max_bounces: 2` lowers that perch from 3.
For every new hub, and for existing hubs through D3's strip, D1 lowers the review rounds and replaces the soft-cap rule at the spent budget.
The spec's compaction rule, "A Decision that adds an edge or weakens a guard states its bound", and the rubric's attack-surface question both require D1 to say what this now lets through and what bounds it.
The operator's split chose the values, so the finding concerns the missing statement, not the choice.

### [MEDIUM] The sparse-base write rule is a cross-cutting invariant with no PATTERN entry

**Location:** `_lyx/discussion/decision-record.md`, no `## PATTERN changes` section; `support-log.md` Q9.

**Issue:** D4 binds every writer of a hub base across `configengine`'s `Set` and `Edit`, `configsync`'s seeds and key moves, and any future writer.
D4's own rationale says a full write undoes D2 once the marker exists and D3 never repairs it.
A later writer that seeds a base from the template breaks the design silently.
`CLAUDE.md` puts a new cross-cutting invariant into PATTERN in the same commit, and the spec gives the record a `## PATTERN changes` section for it.
Q9 answered only whether an existing entry line states that the base stays full.

### [LOW] D4's prime seed drops former values after the marker

**Location:** `_lyx/discussion/decision-record.md` D4, prime-seed bullet.

**Issue:** The prime-seed rule drops a copy's units equal to a former value on every seed, the ones after migration included.
A base removed and reseeded from the prime after the marker exists loses a deliberate value, such as `driver_kind: llm`.
D3's last bullet and acceptance criterion 5 say such a value stays.
The prime copy path is `internal/configsync/configsync.go:606-613`.

### [LOW] Key moves into a sparse base leave three rules open

**Location:** `_lyx/discussion/decision-record.md` D4, key-move bullet; Notes, `moveKeys` bullet.

**Issue:** At a base, `takeKey` overwrites a target holding the template default (`internal/configsync/movekeys.go`, the `nodesEqual(current, defaultNode)` branch).
The Notes say D4 changes that branch, but the bullet states no rule, and after migration such a target is a D2 pin.
"Drops a moved unit equal to the template" also drops a pin after migration.
`moveFile.write` deletes a target the move leaves with no unit (`movekeys.go:248-254`), while D2 requires a header-only base, and no step is named to write it.

### [LOW] A deliberate pre-deploy former value is dropped without an assumption

**Location:** `_lyx/discussion/decision-record.md` Auto-mode assumptions.

**Issue:** The strip drops any base unit equal to a former value, so a hub that set `driver_kind: llm` or `kickstart_pack: true` on purpose before the deploy loses it.
The board entry endorses dropping seeded old-template entries.
Still, the assumptions name only the template-equal case and the deploy-to-walk window, and D3's rationale names `driver_kind: llm` only as a later value.

### [LOW] The whole-map former value of batcher.profiles misses many hubs

**Location:** `_lyx/discussion/decision-record.md` D3, open-map bullet; D1, batcher bullet.

**Issue:** `batcher.profiles` is one unit, so the strip drops it only on an exact whole-map match.
A hub with a profile of its own, which the template comment invites, keeps the map.
So does one seeded under an earlier generation of the map that `batcher.MigrateConfig` does not bring to the former value exactly; the template changed in `3ffc74fc`, `98c3bf05`, `c0bf19c5` and `1a8a6d0e`.
Either hub keeps `budget: 450000`, and no Open risk says so.

### [NIT] The editor's scaffolded base has no header

**Location:** `_lyx/discussion/decision-record.md` D4, editor bullet.

**Issue:** D4 says the editor scaffolds an absent base empty, while D2 says a key-less base holds a header comment.
A whitespace-only save is an abort in `EditPath` (`internal/configengine/edit.go`), so an editor-created base has no header to keep.

## Rejected

- Relocation of D3's rationale sentences on one writer and one commit (focus item 5): kept. They give the reason for the walk-only rule the bullets state, and the spec puts a Rationale on each Decision. No sentence there restates a Turn 3 finding without carrying a reason the plan writer uses.
- D1 against Scope's kept-as-loomyard's-own list (focus item 4): not a defect. D1 moves only `darn`'s reviewer and fixer keys and leaves `darn.verify`, `landing`, `gate`, `board`, `lenses` and `language` alone. The seed-only `models` file differs from the template only in indentation, and every fan, lens and model name D1 sets exists in the `lenses` and `models` templates.
- The deploy-to-first-walk window (focus item 3): stated with its bound. `lyx config` does not run `Ensure`, so the window ends at the next start verb, as the second auto-mode assumption says.
- A failed walk leaving the board marker behind: D3 states the removal as a requirement. That today's `snapshotConfig` does not cover a new board file is the plan's work, not a gap in the record.
