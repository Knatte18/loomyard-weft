# Support log — twb-fta-2

## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry `twb-fta-2` and the board.
   Related entries: `arbiter-fixes` (the Webster deviations list, a separate stencil and `ScopeGuard` change, no overlap with D1–D8), `weaver` (reuses `recover-batch` and `RenderRecoveryPrompt`, which D4–D6 change compatibly), `glyph-use-audit` and `enclose-kickstart` (kick-start pack, which reads glyphs through planglyph and so sees D8's fold).
   `twb-fta-1`, `twb-ftb-*` are bench copies of this entry.
2. Explored base `800a0e892`.
   Item 1: no background shell expires at base; the first-arrival rule in `countsAtOnce` finishes a fresh gated run at once when only unawaited shells remain, and Master awaits only `lyx webster recover-batch`.
   Item 2: the re-prompt Master reads is shuttle's generic "fix every finding" text, which contradicts the stencil section.
   Item 3: `recordedSessions` lists Master sessions only, so recovery writes are never evidence; a non-`done` terminal digest exits 0.
   Item 4: the recovery prefix has no statement on gates; the plan spec leaves the `tmux` tier to Publish.
   Item 5: quarry v0.2.1 resolves unexported and same-package test-file members (`rusageStats`, `tmuxAllowed` both `found` today); the refusal hides the status.
   Item 6: `internal/proc#StartTime` resolves `ambiguous` with `proc_linux.go` and `proc_windows.go`; every planglyph read of quarry goes through `resolveTargets`.
   #511's two refusals said only "still does not resolve"; `tmuxAllowed` was the #500 member declared once per build-tag set, already restructured into one `var` at base, and `rusageStats` is one func at base, so its #511 cause is not visible.
3. Settled D1–D8 by auto-pick (see ledger).
4. Lens review, one fork per lens, all six returned:
   - premises, BLOCKING, D8 fold in `resolveTargets` would reach `lyx quarry resolve` through `planglyph.Resolve`, and that seam has no plan language: fixed, the fold is a helper each plan-reading pass applies; `Resolve` keeps the raw answer.
   - premises, LOW, D7 evidence was today's tree, `tmuxAllowed` already restructured at base: fixed, rationale states the tree checked; issue history moved here.
   - premises, LOW, name `contractevidence.go` as the caller kept clear: fixed in D4.
   - goal-scope and scenarios, MEDIUM, D8 claimed handle binding goes through `resolveTargets`: fixed, D8 states `BindHandles` binds from the delta's created ids, and an end-to-end record-batch criterion was added.
   - goal-scope, MEDIUM, D5 went past the board for retryable `dead`: fixed, D5 narrowed to terminal batches.
   - goal-scope, LOW, Create has no Plan-time flag stated as a reinterpretation: fixed in D8 and ledger Q9.
   - goal-scope, LOW, ungated-hold exclusion reason only in the log: fixed, one clause in Scope Out.
   - goal-scope, LOW, D6 why the recovery only compile-checked: no change asked, rejected as needing none.
   - pattern-fit, MEDIUM, D5 must name `refusal-spec.md`: fixed in D5 and Constraints.
   - pattern-fit and leanness, LOW, D6's `go vet -tags tmux` step runs outside the slots and was unasked: fixed, step dropped.
   - pattern-fit, LOW, D3's Go string quoting a stencil heading: fixed, the text names the verify-fix prompt path.
   - pattern-fit, LOW, glyph chokepoint and test budget: fixed in Constraints.
   - pattern-fit, LOW, D1 vs completion-signal confirmed: no change.
   - scenarios, MEDIUM, D5's `ok: false` writes a refusal friction note: fixed, D5 states it is wanted, with a criterion.
   - scenarios, MEDIUM, D4 widens a reset's own-path set unbounded: fixed, D4's bound names the run-lock and live-strand refusals.
   - scenarios, MEDIUM, D4 session list growth and resume cases: fixed, `recordedSessions` unchanged, id written with the strand, attach adds nothing; criteria added. `run --fresh` dropping the list rejected as needing no criterion: it archives the whole state, Master sessions included.
   - scenarios, LOW, D1 missed notification: fixed in R2.
   - scenarios, LOW, re-entrant terminal `recover-batch`: fixed in D5.
   - scenarios, LOW, legacy `+build` and `ignore`: fixed in D8 and a criterion.
   - scenarios, LOW, D6 package derivation for the compile step: moot, step dropped.
   - board-fit, MEDIUM, `arbiter-fixes` couples through the implementer body and `deviations`: fixed as R5.
   - board-fit, LOW, D1 relies on command labels under other engines: fixed in R1.
   - board-fit, LOW, `weaver` carry-over of the `tmux` skip: fixed as R6.
   - board-fit, LOW, `model-downgrade` and `glyph-use-audit` benches straddling this landing: rejected for the record, context for those benches only.
   - leanness, MEDIUM, D3's rejected-verb sentence: fixed, cut.
   - leanness, MEDIUM, A2–A4 restated decisions: fixed, collapsed to a pointer at the ledger.
   - leanness, LOW, D4 comparison, non-applying Constraints bullets, Goal restating Scope, R1 duplicating D1's bound, D7 history: fixed, each cut or moved here.

## Rejected alternatives

- Item 1, lyx stops the shell explicitly: lyx has no pid for a provider shell, so it cannot.
- Item 1, every outstanding shell holds a gated turn end: the first-arrival rule exists so a stray long-lived shell does not pin a finished run; only gate shells carry a result the run cares about.
- Item 1, hold ungated runs too (the recovery strand): changes shuttle's ungated rule for no reported incident; the recovery's batch is re-checked by the post-batch pass and the plan-level verify.
- Item 2, a verb that spawns the verify fixer: the fixer is an in-session fork only Master's Agent tool can spawn; a cold strand would change the fixer's context and model.
- Item 2, restate the rule only in the stencil: the stencil already says it, and the generic re-prompt outweighed it.
- Item 3, start the second recovery from a clean tree with the first diff as reference: discards valid work and needs a fabric reset, which webster's engine may not run.
- Item 3, exit non-zero on every non-`done` terminal digest: a retryable `dead` is not terminal and Master's ladder retries it, and the board asks only about a terminal batch.
- Item 4, have the recovery compile a `tmux` step with `go vet -tags tmux`: unasked, and it runs outside the gate slots.
- Item 6, teach quarry build constraints: quarry is an external module at v0.2.1, outside this repo.
- Item 6, accept ambiguous only in DoneChecks: the plan validator, delete-order and drift would still disagree with it.
- Item 6, also let a Create reuse a name the same card deletes (#507 second half): not in the board's expected outcome.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): hold or stop a gate shell? Hold via an awaited prefix `lyx gate test` (D1); stopping is impossible.
- Q2 (resolved, auto-pick): how should a still-outstanding gate shell be recorded? One clause in the existing note and warning (D2).
- Q3 (resolved, auto-pick): stencil wording or a verb for Master's verify failure? A caller-supplied re-prompt on the gate entry (D3).
- Q4 (resolved, auto-pick): attribute by batch or reset the tree? Attribute: record recovery session ids and audit them (D4).
- Q5 (resolved, auto-pick): which terminal digests exit non-zero? Those leaving the batch terminal: `stuck`, and `dead` with `recovery_retry: false`, fields kept (D5).
- Q6 (resolved, auto-pick): does the `tmux` skip extend to the implementer fork? No, recovery only, per the board (D6).
- Q7 (resolved, auto-pick): does the Create check resolve unexported or test-file members? Yes at base; only the refusal text changes (D7).
- Q8 (resolved, auto-pick): where does the build-constraint fold live? A planglyph helper every plan-reading pass applies, Go only; `lyx quarry resolve` stays raw (D8).
- Q9 (resolved, auto-pick): what is the board's "flag at Plan time"? Overlapping ambiguous Edit, Delete and `Uses:` glyphs keep refusing at Plan; a Create has no declarations before its card runs, so it has no Plan-time flag (D8).
