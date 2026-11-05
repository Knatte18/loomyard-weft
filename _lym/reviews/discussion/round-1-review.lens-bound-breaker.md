```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every bound, guard, guarantee and claim about existing code in the decision record.

**Claims attacked**

- D1, the `cs` keying claim (`Re = …, dP = …` keys as `Re`): held, as stated.
- D1/D5, `key#n` and `#n` identity syntax: broken (finding 8).
- D1 rationale, "shifted repeated-key cells are re-judged, letting no regression through": held. An over-narrow `key#n` gives a false Stuck, not a leak.
- D2 bound, `command` mode gives up containment and the judge sees only the log: broken. The judge also chooses which cases it rules on (finding 3).
- D3, "`partial` is recorded as `full` with one Warn": broken against D15's value set and the reinterpretation rule (finding 1).
- D3, "a run whose scope record says `off` or `full` keeps it": held. A recorded `partial` is left undefined (finding 2).
- D3, the `docs` profile's darn block can set `drift: {scope: off}`: held. `internal/runprofile/template.yaml:67-70` nests per-module blocks under `darn`.
- D4, "Drift-Verify spawns no agent that edits code": broken through D9's snapshot (finding 6).
- D4, "Done at once only at the head of the last passing check with no amendment since": held.
- D6, the forging bound (needs a hand-made weft commit before the first warp commit): held. Only the oldest `Drift-Seal` commit counts.
- D6, "every fabric weft commit carries `Warp-SHA`": held on `appendWarpSHATrailer` (`internal/fabricengine/trailer.go:91-95`).
- D6, re-entry "seals an existing valid `expected.yaml` without spawning": broken (finding 4).
- D6/D11, a failed seal check's way forward is `amend --restore`: broken for two failure shapes (finding 5).
- D7, "inheriting a held slot keeps a check run from a gate key from waiting on its own slot": broken. No gate key can run the check (finding 10).
- D7, concurrent checks serialised by the capture lock with separate scratch directories: held.
- D8, "Webster's run-level gate is fed by the Webster row's `gates` list": held (`internal/websterengine/runlevel.go:107`, `:450-452`, `:790`).
- D9, the tracked-tree snapshot catches a judge's edit: broken (finding 6).
- D10, tuple reuse and re-judging a changed tuple: held.
- D11 bound, "reaches no agent of the run": partly broken (finding 9).
- D11 bound, "only adds coverage": held. Dropping a `does not match` ruling through an amendment is stated in D11 itself.
- D12 bound, "only the parent or operator runs `defer` after the writer": held. An agent that deletes the baseline still only gets the parent offered `defer`.
- D12, a lost baseline has a way forward: broken at Drift-Expect (finding 7).
- D15, retired keys in `run_settings` are refused at seeding: held for new runs. A run already seeded is finding 2.
- D15, the reinterpretations cover the meaning changes: broken (finding 1).

**Findings**

1. **BLOCKING · design · D3, D15:** `drift.scope` changes meaning with no `configreg.Reinterpretation`.
   - Today `partial` (with `skip`) is valid and `""` means "the Drift-Baseline agent decides" (`internal/driftengine/config.go:192-200`, message at `:196`).
   - After this lands, `partial` silently means `full` (D3, decision-record.md:83) and `""` means `full` (D3:79).
   - D15 lists reinterpretations only for `compare` without `diff` and for `{skip_file}` (decision-record.md:266-269). The drift module carries only `DirBaseReinterpretation` today (`internal/configreg/configreg.go:153`).
   - D15's kept value set for `scope` is `""`, `full` or `off` (decision-record.md:257). A loader that validates against that set, as `checkScope` does today, refuses `partial` before D3's Warn path can run.
   - Counterexample: a hub base with `scope: partial, skip: [a]` either fails every drift load or turns into `full` with no flag to the prime orch.
   - This violates `PATTERN-config-reinterpretation`.

2. **MEDIUM · design · D3, D4 and the in-flight Open risks line:** a run whose recorded scope is `partial` is unbounded.
   - `ScopeRecord` holds `partial` plus `Skip` (`internal/driftengine/records.go:20-23`, `:73-74`), and that run's baseline capture wrote no file for its skipped tests (`internal/driftengine/capture.go:39-40`, `:91`).
   - After landing, D3 defines only `off` and `full`.
   - A full current capture yields an `added case` cell for every skipped test. Each is unexpected against an empty expectation, so the run is Stuck at Drift-Verify, and the `cs` capture holding `{skip_file}` is refused anyway (D15:271).
   - The Open risks line (decision-record.md:316) names acceptances and the empty expectation, but not a recorded `partial` scope or its short baseline.

3. **MEDIUM · design · D2, D9 `command` form:** the `judgement` entry cannot check completeness in `command` mode.
   - The compare is opaque, so code never knows the set of drifted cases.
   - "Every case has a valid ruling" (decision-record.md:190), and the D9 acceptance "a missing case ruling in `command` mode" (:328), can only mean every case the judge chose to list.
   - Counterexample: the compare exits 1, the judge writes no rulings or a subset, the entry passes, there is no `does not match`, and Drift-Verify is Done.
   - D2's bound (:74) states only the log's first-deviating-line limit, not this one.

4. **MEDIUM · design · D6 re-entry:** "seals an existing `expected.yaml` that validates without spawning the agent" (decision-record.md:139) cannot tell a finished expect session from an interrupted one.
   - An empty expectation is valid (D5:127), so a stub or partial file written before a pane death or crash is sealed as final.
   - Today's drift producers track an open agent session and probe it on re-entry for exactly this case (`internal/driftengine/verify.go:125-140`, `AgentSessionOpen` and `probeInterruptedAgent`).

5. **MEDIUM · design · D6, D11, against `PATTERN-refusal-way-forward`:** `lym drift amend --restore` is the one way forward for every failed seal check (decision-record.md:145), but it clears only an edited file.
   - (a) When the seal commit's `Warp-SHA` differs from the baseline head, the fault is in the commit, and restoring the file changes nothing.
   - (b) On a run past Drift-Expect with no seal commit but an `expected.yaml` present (an in-flight run where some session wrote the file), there is no sealed copy to restore. Drift-Expect never re-enters, so nothing seals it.
   - The D11 acceptance "`--restore` makes a failed seal check pass" (:330) is false for both.
   - The named way forward does not reach these cases, which the pattern forbids. The handler should judge whether that raises this to BLOCKING.

6. **MEDIUM · design · D9 snapshot, D4 "edits no code":** "snapshots the tracked tree … a tracked change the session made outside `judgement.yaml` is a Stuck" (decision-record.md:191) is broken by a judge that commits.
   - After `git commit` the tree is clean against the new HEAD.
   - A new untracked source file is not a tracked change either.
   - Agent sessions can commit in a worktree: the recent commit `fe2663d3` lets Codex sessions do so.
   - The check must also compare HEAD and the untracked paths.

7. **LOW · design · D12:** a baseline directory lost between Drift-Await and Drift-Expect has no way forward.
   - Drift-Expect needs the baseline to build the identity listing (D5:118).
   - D12 admits `defer` only at a Stuck writer, Drift-Verify or Drift-Recheck row (decision-record.md:225), so a Stuck Drift-Expect has no admitted verb.

8. **LOW · design · D1, D5 identity syntax:** `key#n` and `#n` are ambiguous when a key holds `#` or looks like `#n`.
   - Example: the 2nd occurrence of key `C#` is written `C##2`.
   - Example: key `x#2` is indistinguishable from the 2nd occurrence of `x`.
   - D5's parse of `key` against `key#n` (decision-record.md:122) has no escape rule. Default patterns allow such keys (:51-52).

9. **LOW · consistency · D11 bound:** "reaches no agent of the run" (decision-record.md:219) overstates the guard.
   - D11 adopts `lym loom decision add`'s caller check, which by its own doc "guards a mistaken call, not a session that clears or forges the variable" (`internal/loomcli/decision.go:8-11`).
   - The real bound is the strand-name check plus the shell guard that refuses setting lym's strand variables. The bound should say that.

10. **LOW · consistency · D7 rationale:** the claim that inheriting a held slot "keeps a check run from a gate key from waiting on its own slot" (decision-record.md:166) describes a path that does not exist.
    - A gate key holds only test tiers and the closed lints `comments` and `pattern` (`internal/gateslot/config.go:431-433`; `internal/gateslot/template.yaml:14-18`).
    - No gate key can run the drift check without a new mechanism, which D8 leaves to the plan.

**Tool calls**

- grep `internal/loomcli/decision.go` for its caller check: D11 copies it, and the manifest did not include it.
- grep `internal/runprofile/template.yaml` for the `docs` block shape: needed to test D3's profile claim.
- grep `internal/configreg` for the drift reinterpretations and `internal/fabricengine/trailer.go` for `Warp-SHA`: needed to test D15's reinterpretation coverage and D6's trailer claim.
