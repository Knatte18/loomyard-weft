---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/00-overview.md (Shared Decision watcher-index-binary-reading) and _lyx/plan/14-role-file-command-index.md (Intent, startAutoReload)
    summary: The plan settles the two added-after-Discussion watcher decisions by opposite precedence rules, and states only one of the two conflicts
---

### [MEDIUM] The two added watcher decisions are settled by opposite precedence rules, and only one conflict is stated

**Location:** `_lyx/plan/00-overview.md`, Shared Decision `watcher-index-binary-reading`; `_lyx/plan/14-role-file-command-index.md`, the `Intent` sentence on the three render sites.

**Issue:** The decision record carries two "Added after Discussion (parent, 2026-10-10)" entries that contradict the older sections they amend, and the plan resolves them in opposite directions.

- "Watcher index binary" says the watcher renders from its own recorded path and declines "new launch-path plumbing in this task".
  It contradicts Scope In (reed records the path), D4, the `PATTERN-pane-binary-resolution` constraint and the acceptance criterion on the watcher's render.
  The plan states this conflict in `watcher-index-binary-reading` and lets the older sections win: "four sections against one settle it".
  Cards 13 and 14 follow D4: card 13 records `Strand.LyxBin` in `launchStrandLocked`, `StrandStatus` exposes it, and card 14's `newIndexSource` reads it through `strandOps` and runs it.
- "Missing-binary bound at auto-reload" says `startAutoReload` keeps its tick-error path.
  It contradicts the same kind of older sections: D4's Bound ("the render fails and the watcher goes idle with the reason ... at all three render sites"), Scope Out's one carve-out ("except that `startAutoReload` turns a role-file render failure into idle with the reason"), and the acceptance criterion ("sends the watcher idle with the reason at each of `startClearing`, `reloadAfterCompaction` and `startAutoReload`, never a failed tick").
  Card 14 follows the added decision here ("`startAutoReload` returns the error as a failed tick ... per the record's missing-binary-bound-at-auto-reload decision"), and its test plan pins a tick error for the failing source in the auto-compaction reload test.
  No Shared Decision states this conflict, and the plan doesn't say why the precedence it gave the first added decision doesn't apply here.

So the plan's handling of the record's later Decisions is incoherent.
By the rule the plan wrote down, card 14 breaks three sections of the answer key.
By the rule it applies silently, card 13 is scope the parent's later decision declined.
The record's Open risks entry on a strand recorded before the field existed ("its watcher's renders go idle with the reason until the orch is relaunched") also stops holding at the auto-reload site, where card 14's watcher exits after the consecutive-tick-error cap instead.

**Fix:** Choose one precedence rule and apply it to both added decisions in one Shared Decision that names both conflicts.
The smallest fix keeps the stated rule: extend `watcher-index-binary-reading`, or add a sibling decision, so the D4 Bound, the Scope Out carve-out and the acceptance criterion also outrank "Missing-binary bound at auto-reload".
Card 14's `Intent` then makes a source failure at `startAutoReload` return to idle with the reason, like the other two sites.
Its auto-compaction reload test asserts idle with the reason, not a tick error.
The other option applies the opposite rule to both: the parent's later entries win.
That drops card 13 and has card 14's watcher source run the watcher's own executable, and the Shared Decision then states the D4, Scope and acceptance-criterion departures for both entries.

## Focus settlements

The directive's four questions and its closing note, each with a verdict.

1. **Invocable groups with only own persistent flags: not a defect.**
   Card 1 defines `Invocable` on `LocalNonPersistentFlags`.
   In cobra v1.10.2, that method drops every flag in the command's own `PersistentFlags()`.
   At HEAD, `burler` (`--stencils-dir`, `--target-dir`) and `webster` (`--stencils-dir`, `--plan-dir`, `--target-dir`) declare theirs only through `PersistentFlags()`, and both `Use` strings are the bare name.
   Card 1 therefore classifies them as non-invocable, and cards 4 and 8 leave them unannotated.
   Card 16's audience check calls the same `clihelp.Invocable`, so it does not fail on them.
   `config`'s `--print`/`--set` sit on `Flags()` and still qualify it as invocable.
   Every other `GroupRunE` group at HEAD has a bare `Use` and no local flag, so none is misclassified.
   Card 1's renderer test and card 16's audience subtest both pin the persistent-flag case.
2. **D9 trim sets: not a defect.**
   I built HEAD's `lyx` into the scratchpad and measured every page's `Long` (`--help --json`, plain `--help` for the `status` verbs, hidden verbs measured directly).
   The pages over 2,000 bytes are `start`/`loom start`, `fabric clone` (4,186), `webster reset`, `fabric cleanup`, the `loom` group, the `webster` group, `loom resume`, `webster validate`, `fabric remove` and `fabric merge-in`.
   This is the union of the sets named in cards 6, 7 and 8, with `fabric clone` included.
   Every other package measures under the cap, as cards 3, 4, 5 and 9 state.
   Each form card defines its trim set by measurement ("measured, not listed") and gives the list only as a snapshot.
3. **Decisions carried: one defect, F1.**
   D1→2, D2→1, D3→1, 3–9, 16, D4→13, 14, D5→15, D6/D8/D9→3–9, 16, D7→9, D10→2, 14, 16, D11→10–12, and "Root Long keeps its module list"→2, 16.
   Card 13 records the watcher's index path on `Strand` in `launchStrandLocked` and exposes it through `StrandStatus`.
   Card 14 reads it through `strandOps.Strands()`, which `Command`'s pre-run wires for `watch` as for every verb, and runs it.
   That chain is sound.
   The defect is how the two added watcher decisions are reconciled; see F1.
   Every `Short` and `Use` at HEAD that breaks D6 or D8 falls inside a card that names its rewrite: `board`→9, `ide`/`quarry`/`stencil`/`gate`/`config`→4, `webster`→8, `fabric`→6, `loom`→7, `reed remove`→5, and the root `Short`→16.
4. **Runnable group whose own annotation misses while a descendant matches: not a defect.**
   Card 1 says that group prints in group form when only a descendant prints, and not at all otherwise.
   Its printed children therefore never nest under a missing line.
   Card 1's `index_test.go` pins this with the row "a runnable group outside the audience rendered in group form above a child that is in it".

**Closing note, per-package audience assertions against card 16's tree walk: not a defect.**
The only per-package audience assertion is card 3's single added assertion in the existing `goto_test.go`.
It pins that a built verb carries the value its `VerbText` names.
Card 16's walk checks only presence and membership in the closed set, so it cannot catch a builder that writes a fixed value instead of the per-mount one.
The assertion therefore covers behavior card 16 does not, and it extends an existing test.
