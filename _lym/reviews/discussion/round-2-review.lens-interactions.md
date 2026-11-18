```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens, which checks pairs of decisions that act on the same state.

**Pairs checked**
- D2 × D9 (`run_settings` setting hardener.yaml keys against the local layer's key classification): broken (F-A).
- D1/D3 × D9 (`tree_scope` narrows HardenFix's verify gate and can be overridden through `run_settings`): broken (F-A).
- D4 × D9 (`llm_driving` turns off the concurrent copies, `packages` narrows the live gate, both set per task): broken (F-A).
- D9 `auto_accept` × D6 (a red live gate blocks CONVERGED, "ends capped, its parent decides"): broken (F-B).
- D9 `auto_accept` × D5/D7 (the halt-time reap and the bound on the halted ledger): held. An auto-accept goes on to Finalize, which reaps.
- D3 rotation × D2 Preflight × D9 `hard_max_bounces`: held. Expansion to the hard cap, the refusal of adjacent duplicates and a wrapped first/last pair cover every round up to the hard cap, and the Bouncer never continues at the hard cap (`internal/shedadapters/bouncer.go:258`).
- D3 rotation × D6 CONVERGED (different reviewer from the previous round): held.
- D4 `live_attempts` × D6 deferral and carry-over × D9: held, apart from F-B.
- D4 live gate × `--copies` × gateslot (top-level call, all-or-nothing acquire): held. Nothing in the record nests the call in a held slot.
- D4 × D2 (Preflight checks for the `harden-live`/`-copies` keys and the tier-3 refusal): held.
- D3 `harden-burler-fix` `[0, 1]` × D4 `harden-live` `[2]`: held. A live-tier failure reaches the fixer only through the live gate.
- D5 four reaps × D6 stray report × D7 archive order: held. Each reap appends with its row and reap point, and the archive copies after the end-of-row reap.
- D6 fix-commits file × D4 live re-prompts × D7 archive: broken (F-C).
- D6 per-round files × D7 archive list: held. The list covers models, live report, fix commits, stray report, review, fixer report and ledger.
- D5 runproc.End × the PATTERN changes bullet × D5 `/proc` kill bound: held. The bound text matches the declared change.
- D5 reap × D4 tmuxkit stray check: held. They run in sequence in the row, and tmuxkit uses a private `TMUX_TMPDIR`.
- D5 Finalize/halt reap × D1 landing through Finalize: held.
- D10 order clause × D3 ring × D1 recipe rows: held. Harden-Bouncer lies on the on_done path, and `onlyRowOf` refuses a second HardenFix row (`internal/shedbuild/loomcontract.go:34,65`).
- D7 per-round archive × D6 Bouncer commit seam (only on an approved settle): held.
- D8 × the two `### Added after Discussion` entries × Open risks × support-log Q13: broken (F-D).
- D8 reinterpretation × PATTERN-config-layers (applied once, before any command): held for the hub base file.
- Constraints × D4/D5 (reed `-L` key, tmuxkit, lymtemp, CloneAndWire, the Bouncer seam, batten's list): held.
- Acceptance criteria × decisions: each D1–D12 criterion matches its decision, except the ones named in F-B and F-D.

**Findings**

**F-A** — MEDIUM, design, `_lym/discussion/decision-record.md:216` (D9 "A task's `run_settings` may override any of them"; D2 lines 49–53)
- Summary: D9 lets `run_settings` override every hardener.yaml key. But run settings reach only the local layer, which holds only `CategoryNone` keys. The keys D9 lists include ones the existing modules class as guards:
  - `auto_accept`: `internal/reviewbudget/reviewbudget.go:42`
  - the fans: `internal/loomengine/config.go:649-656`
  - `tree_scope`, which darn closes to the local layer for the very reason that it narrows the verify gate D3 runs: `internal/darnengine/config.go:59`

  D2's own per-task keys also hollow a check. `llm_driving: true` turns off D4's concurrent run, and `packages` bounds what the live gate checks. So either the load refuses them, or a run can loosen its own gates.
- Evidence: `internal/configengine/doc.go:103`, "The local layer holds only keys classified [CategoryNone]"; `internal/configreg/runsettings.go:93` (`checkLocalDocument`); `pattern/PATTERN-config-layers.md:19-25,33`.
- Fix: in D9, classify every hardener.yaml key, name which ones `run_settings` may set, and state the bound for `packages` and `llm_driving`. Drop "any of them".

**F-B** — MEDIUM, design, `_lym/discussion/decision-record.md:171` (D6 "A live gate left red by deferred failures therefore blocks CONVERGED, so such a campaign ends capped and its parent decides from the facts"), with D9 line 210 (`auto_accept`)
- Summary: with the Harden perch's `auto_accept` set, the Bouncer turns a spent budget's CONTINUE or CIRCLING round into an automatic accept and returns Done. A campaign whose live gate is still red then passes to Describe, PR-Review and Finalize and lands with no parent decision. That contradicts D6's "ends capped, its parent decides" and D4's merge bar of one green live run. The record gives `auto_accept` no default and no bound.
- Evidence: `internal/shedadapters/bouncer.go:260-261`.
- Fix: in D6/D9, either keep `auto_accept` from accepting a round whose live report is red, or have Preflight refuse it for `hardener`. Add a criterion.

**F-C** — LOW, consistency, `_lym/discussion/decision-record.md:164` (D6 "the fix commits, written by `HardenFix` from the commits its round added")
- Summary: D4 has the live gate re-prompt the fixer up to `live_attempts` times, and each re-prompt can add commits. D6 does not say when the fix-commits file is written. If it is written when the first fixer session ends, D7's archive, which copies after the gates, and the Bouncer's facts both miss the commits from the re-prompts.
- Evidence: the record itself, lines 103 and 181.
- Fix: in D6, write the fix-commits file after the row's last gate, from the commits between the row's start and its final HEAD.

**F-D** — MEDIUM, consistency, `_lym/discussion/decision-record.md:200` (D8 rationale) and `:286` (Open risks), with `_lym/discussion/support-log.md:89` (Q13)
- Summary: the operator entry "Delete treadle when Hardener is built (F10)" answers the parent question. Even so:
  - D8's rationale still carries the misquote ("kept its stencils only until Hardener is built").
  - Open risks still asks whether treadle may be deleted, and support-log Q13 still records it as open.
  - The arbiter entry's replacement text would itself call the question "open" and say the stencils are "re-derived". That contradicts the operator entry, which says the stencils are not re-derived and that its decision replaces the 2026-10-16 one.
- Fix: drop the Open risks treadle line and close Q13, citing the operator entry. Restate D8's rationale to quote the 2026-10-16 note as written and say that the operator's 2026-11-18 decision replaces it. Do not keep the arbiter text's "open parent question" and "re-derived" clauses.

**Note outside my lens:** HardenFix's verify gate is a new `verifytree.Verify` site that PATTERN-gate-slot's site list does not name, and the record declares no PATTERN change for it (`pattern/PATTERN-gate-slot.md:93-94`). This is for the handler or the PATTERN-fit check.

**Tool calls**
- Read `round-2-changes.md` and ran `git log`/`git diff` on the decision record, to find the changed hunks for the incremental-round rule. The context lacked which lines changed; `git` holds no history for `_lym/discussion`.
- Ran `git status` and listed `_lym/reviews/discussion`, to look for the record's history somewhere else. None was found, so the handler judges the changed hunks.
