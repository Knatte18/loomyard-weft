---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md:110
    summary: D4 drops a moved unit equal to the template in every ReconcileHubWideAt caller, against D3's strips-nothing rule and the declared PATTERN-sparse-hub-base text
    fix: In D4's key-move bullet, delete the clause "a moved unit equal to the template is dropped only while the marker is absent;" so a key move writes every moved unit, and add one sentence to that bullet saying the walk's one-time base strip, which runs after the key moves (D3), drops a moved unit equal to the template or a former value on an unmigrated board, so no other caller drops one.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:108
    summary: D4 keeps an emptied key-move target as a header-only base but says nothing of a source base the move empties, which moveFile.write deletes, leaving a sparse hub base absent
    fix: In D4's key-move bullet, after "a target the move leaves with no unit is written as the header-only base, never deleted;", add "and a source base the move leaves with no unit is written as the header-only base too, never deleted;". In Notes for the plan writer, change "and `moveFile.write`'s delete of an emptied target" to "and `moveFile.write`'s delete of an emptied file, source or target,". Add to the acceptance criterion on key moves (line 174) that a key move out of a hub base holding only the moved unit leaves that base present and key-less.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:50
    summary: D1's guard bound misses that Webster-Review auto-accept lets open BLOCKING findings through, leans on Merge-Review, which reviews only the merge-in and can be skipped, and omits the reviewer and fixer model downgrades
    fix: Rewrite D1's paragraph from "The loom values weaken two review guards." to the end of its Bound so it states (1) Webster-Review gets one judged round instead of three and, with no blocking_escalates on its Bouncer, accepts a harvested round at the spent budget with its open findings, BLOCKING ones included; (2) Discussion-Review gets two rounds instead of three; (3) Webster-Review's and Merge-Review's reviewer drops from sonnet[high] to sonnet[medium], and their fixer from opus[medium] to sonnet[medium]; then a Bound that names what still checks the task's code — the verify gates, PR-Gate's alignment approve or the parent — and drops Merge-Review as a bound on the task's own change, since it reviews only the task's interaction with what the parent gained and is skipped when the parent gained nothing the task touches; keep the operator-split, the pin-back with `lyx config loom --layer default --set` and the D3-only reach.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: The worktree former-value drop changes a key's resolved value where the hub base holds a tuned non-former value, breaking the worktree strip's resolves-unchanged guarantee without saying so
    fix: Add an Auto-mode assumptions bullet stating that an unmarked worktree unit equal to a former value is dropped even when the hub base holds another tuned value for that unit, so the worktree resolves to the base's value instead of the former one, since a full worktree file cannot tell a seeded copy from a tuned override; the worktree pins it again with `lyx config <module> --set`.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:167
    summary: D3 names the single-pair call (Options.Pair) as a path that drops former values whatever the board marker says, but no acceptance criterion covers it
    fix: Extend the criterion at line 167 to read "An unmarked worktree file holding a former value ends the same walk without it, and so does an unmarked pair reconciled through `hubreconcile`'s `Options.Pair` on a board that already holds the marker."
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:152
    summary: The older-binary refill risk lists the walk, lyx fabric reconcile, lyx config reconcile --apply --layer default and --set, but omits lyx fabric clone of an existing weft, which runs ReconcileHubWideAt with Apply and commits the board
    fix: In the first Open risks bullet, change "its walk, `lyx fabric reconcile` and `lyx config reconcile --apply --layer default` run the old full reconcile" to "its walk, `lyx fabric reconcile`, `lyx config reconcile --apply --layer default` and a `lyx fabric clone` of an existing weft run the old full reconcile".
---

### [BLOCKING] D4's key-move drop is a template-equal drop outside the walk

**Location:** `_lyx/discussion/decision-record.md:110`

**Issue:** D4 says "a moved unit equal to the template is dropped only while the marker is absent".
Key moves run inside `configsync.ReconcileHubWideAt` (`moveKeys` at `internal/configsync/configsync.go:558`), which `lyx fabric reconcile` (`internal/fabriccli/fabric.go:706`), `lyx config reconcile --apply --layer default` (`internal/configcli/hubwide.go:211`) and `lyx fabric clone` (`internal/fabriccli/clone.go:57`) all call.
So on an unmarked board those callers drop a unit the template holds, which contradicts D3's "Every other caller of `ReconcileHubWideAt` ... strips nothing" and the declared `PATTERN-sparse-hub-base` text, "only `hubreconcile`'s one-time walk strip drops a unit the template holds, bar `lenses`' template-equal entries".
The clause is also redundant inside the walk: D3's base strip runs "after the key moves", so the walk already drops a moved unit equal to the template or a former value on an unmarked board.
Removing the clause leaves the non-walk callers writing the moved unit, which the walk later strips, and once the marker exists it stays as a pin, as D4's previous clause already wants for a target unit.

### [MEDIUM] An emptied key-move source base is deleted

**Location:** `_lyx/discussion/decision-record.md:108`

**Issue:** `moveFile.write` (`internal/configsync/movekeys.go`) removes any file the move leaves with no keys, and the `moveKeys` doc says "A source file the move empties is deleted."
With a sparse base, a source base holding only the moved unit is the common case, and deleting it leaves the hub base absent, against D2's "The base file still exists, since strictness and the build stamp's absent-default check read its presence".
For a per-worktree module that absence makes `repoDefaultAbsent` hold, so every start verb walks until a reseed, and the reseed comes from the prime's copy.
D4 handles only "a target the move leaves with no unit", and the plan note names only "`moveFile.write`'s delete of an emptied target", so the plan writer, who reads only the record, gets no requirement for the source side.

### [MEDIUM] D1's guard bound is incomplete and leans on Merge-Review

**Location:** `_lyx/discussion/decision-record.md:50`

**Issue:** The Bound states "a harvested round's open findings accepted at Webster-Review" without saying BLOCKING findings are among them.
The Webster-Bouncer row in `contracts/recipes/loom-recipe.yaml` sets no `blocking_escalates`, so `blockingSendsToRule` returns false and an open BLOCKING finding is auto-accepted at the spent budget of one round; only Merge-Bouncer sets `blocking_escalates: true`.
The Bound then cites "Merge-Review ... still run", but the recipe describes Merge-Review as reviewing "how the task's code interacts with what the parent gained in the merge-in", and its `skip_seam: merge-overlap` skips the judge when the parent gained nothing the task touches, so it does not bound a weakened review of the task's own change.
D1 also downgrades guards the Bound leaves unnamed: `webster_reviewer_model` and `merge_reviewer_model` go from the template's `sonnet[high]` to `sonnet[medium]`, and `webster_fixer_model` and `merge_fixer_model` from `fixer_model`'s `opus[medium]` to `sonnet[medium]` (`internal/loomengine/template.yaml`).
The rubric's attack-surface question requires each downgraded guard's Decision to state what it lets through and what bounds it.

### [LOW] The worktree former-value drop can change a resolved value

**Location:** `_lyx/discussion/decision-record.md:81`

**Issue:** `hubreconcile`'s doc guarantees the worktree strip leaves "every key the file held" resolving unchanged.
D3's former-value drop removes an unmarked worktree unit equal to a former value whatever the layers below hold, so where the hub base keeps a tuned value that is neither the template nor a former value, the worktree's resolved value moves from the former value to the base's.
The outcome is likely what the hub wants, since a full worktree file holds seeded copies, but D3's Bound speaks only of the values kept and the Auto-mode assumptions cover only base units, so the resolved-value change is unstated.

### [LOW] No acceptance criterion for the `Options.Pair` former-value drop

**Location:** `_lyx/discussion/decision-record.md:167`

**Issue:** D3 bullet 2 names the walk and the single-pair call (`hubreconcile`'s `Options.Pair`, used by `internal/battencli/wire.go`) as paths that drop former values from an unmarked worktree whether or not the board marker exists.
The criteria at lines 167 and 168 cover the migrating walk, a clone's walk and a mid-merge pair's next walk, all of which go through `walkHub`; `reconcilePair` is a separate path in `internal/hubreconcile/walk.go` that no criterion reaches.

### [LOW] The older-binary refill risk omits `lyx fabric clone`

**Location:** `_lyx/discussion/decision-record.md:152`

**Issue:** The focus asks whether every way an older binary refills a sparse base is named.
`CloneAndWire` calls `configsync.ReconcileHubWideAt(..., HubWideOptions{Apply: true})` and commits and pushes the board (`internal/fabriccli/clone.go:57`), and `CloneHub` checks the board out from an existing weft `main`, so an older binary cloning a migrated hub on another machine refills its hub-wide base files with the old template.
The risk names the walk, `lyx fabric reconcile`, `lyx config reconcile --apply --layer default` and `--set`, but not the clone.
The repair the risk states, removing the board marker with a board commit, covers this refill too, so only the list needs the path.

## Focus verdicts

- Directive 1: D3 bullet 2 states the drop applies whatever the board marker says, on the walk, the mid-merge pair's next walk, `Options.Pair` and the clone's walk; F4 and F5 are the gaps.
- Directive 2: the repair once the marker exists is stated (remove the marker with a board commit), with its limit to the immediately prior template generation; F6 is the missing path.
- Directive 3: the template seed, the prime seed, `--set` and the editor scaffold agree with D2's header and D3's gating; the key-move bullet does not (F1), and a key move that empties a target is stated as header-only while an emptied source is not (F2).
- Directive 4: D1 is the only Decision that lowers a review budget or changes an accept rule; F3.
