```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop, reset or recovery step a card adds, does its named next step succeed from the state it leaves?

**Findings**

1. **BLOCKING** (card 13, Shared Decision stop-prefixes): card 13 has the missing-baseline Stuck open with `DeferHaltPrefix` and then `DecisionHaltPrefix`, so the reason does not start with `DecisionHaltPrefix`.
   - `arbiterstop.HandsBack` (`internal/arbiterstop/arbiterstop.go:63`, per exploration) matches only a leading `DecisionHaltPrefix`. The arbiter is therefore not barred, and the halt that only the parent's `lym drift defer` can settle need not reach the parent.
   - Card 16 and the overview put `DecisionHaltPrefix` first, and card 19's `Defer` reads `DeferHaltPrefix` "after an optional decision-halt prefix". Card 13's order contradicts both.
   - Evidence: `_lym/plan/13-drift-expect-producer.md`, Intent bullet "a counted capture whose baseline directory is missing is a Stuck opening with `DeferHaltPrefix` and `shedadapters.DecisionHaltPrefix`".

2. **MEDIUM** (card 13, D6/D12): the moved-warp-head Stuck names `lym drift defer`, but card 13 never says it carries `DeferHaltPrefix`.
   - Card 19's `Defer` admits a Drift-Expect halt only by that prefix or by a failing `CheckBaseline`. A moved head fails neither, so following card 13 alone, `defer` is refused at the halt that names it.
   - Card 19's `TestDeferWaive` row "admitted at a blocked Drift-Expect whose head moved" would then fail.
   - Evidence: card 13 Intent, moved-head bullet; card 19 Intent, `Defer` paragraph.

3. **MEDIUM** (card 16, D9): the judge-change Stuck's resume is unspecified, and the snapshot guard can be lost.
   - That Stuck leaves `judge_session` open, and its way forward is restore, then `lym loom resume`.
   - On resume, step 5 goes straight to "the probe resumes a live chair … and respawns otherwise". It does not first compare the trees with the recorded snapshot, and it does not say the respawn keeps that snapshot.
   - If the respawn goes through step 10, which takes a new snapshot, the judge's unrestored edits become the new baseline and the guard is bypassed. If it keeps the snapshot, a resume without the restore spawns a judge again before re-stopping.
   - Evidence: `16-drift-verify-lines.md` steps 5 and 10, against the test row "a resume compares against the recorded snapshot".

4. **MEDIUM** (cards 16 and 15, D9): two Stucks carry no `DecisionHaltPrefix`, so the arbiter can resume past them.
   - The two Stucks are the judge-change Stuck and the snapshot-lost Stuck. Their way forward is "the parent or operator" checking or restoring both checkouts, then `lym loom resume`.
   - The stop-prefixes decision gives the prefix only to Stucks that name a parent verb.
   - The arbiter can run the resume without the restore or the check. After snapshot-lost, that resume takes a new snapshot that absorbs any judge edit.
   - Evidence: `00-overview.md`, Decision stop-prefixes; card 16 step 5 and step 10's change Stuck.

5. **MEDIUM** (cards 9, 10, 26, D7/D15): config refusals inside the current capture surface as Go errors, not Stucks, so their way forward is lost.
   - `CaptureCurrent` returns an error for `runStep`'s refusals: `missingDirRefusal`, and card 26's `{skip_file}` refusal in `renderCommand`.
   - Card 10 maps only a failed capture (`Passed` false) to `CaptureFault`. The error branch is unplanned, so `Check` returns an error.
   - Drift-Verify therefore ends in an error rather than a Stuck naming the edit, and the drift gate entry returns a gate error. The refusal's "remove it … with lym config drift" never reaches the parent as a halt.
   - No test row covers a `{skip_file}` command at Drift-Verify or the gate. The planned rows are in `TestRunCapture` and `TestRunCompare` only.
   - Evidence: card 9 Intent (`CurrentCapture.Passed`, `Reason`); card 10 step 6; card 26 `renderCommand` bullet.

6. **MEDIUM** (card 10, D8/D12): a capture failure the writer cannot fix has no way forward at the writer row.
   - `NewDriftGate` fails on `CaptureFault`, naming the log. After 3 attempts Darn-Write is Stuck, and in tapestry Merriam's run-level entry fails into the verify fixer, then Webster is Stuck.
   - Card 19's `Defer` admits a writer row only with `DeferHaltPrefix` or a failing `CheckBaseline`. A gate-exhausted writer Stuck has neither, so `lym drift defer` is refused.
   - `lym loom resume` reruns the same failing capture. For an environmental capture failure the run cannot progress past the writer.
   - Evidence: card 10's gate paragraph, and card 19's `Defer` admission.

7. **MEDIUM** (cards 11 and 21, D7/D8): in tapestry an intended but unexpected change has no clean route to the parent's amend.
   - The run-level `drift` failure goes to the verify fixer. Card 11's stencil tells it to fix the values "in code like a failing test", so it reverts intended work or burns attempts.
   - Card 10's way-forward text ("when the change is intended, leave it for the parent's `lym drift amend`") reaches Master's reprompt, but not the fixer, which is the only actor that edits code.
   - Evidence: card 11, last bullet; card 10, `NewDriftGate` paragraph.

8. **MEDIUM** (Shared Decision tapestry-fix-route, cards 16 and 11): the tapestry `FixRoute` assumes something the plan never researches or tests.
   - The route is `goto --to Webster`, then resume. It assumes Webster, re-entered with every card done, starts Merriam, which ends its turn and runs the run-level `drift` entry.
   - D7 itself says that re-entered Webster "has no card to run". If the producer returns Done without a Master session, the route loops Webster → Parent-Align → Drift-Verify → the same Stuck.
   - No card plans a row that reaches this path. It is a fact the design depends on, left to be found while the plan runs.
   - Evidence: `00-overview.md`, Decision tapestry-fix-route; card 16's `FixRoute` paragraph; the decision record's D7 tapestry sentence.

9. **LOW** (cards 19 and 22, D12): `lym drift defer` takes no slug and resolves the worktree from the cwd, unlike `amend <slug>`.
   - The parent orch, whose cwd is its own worktree, has to `cd` into the run's worktree to follow any Stuck that names `lym drift defer`.
   - No reason names that.
   - Evidence: card 19, `deferCmd` and `amendCmd` paragraphs.

10. **LOW** (card 26 against card 9): the two cards conflict on when the `{skip_file}` refusal runs.
    - Card 26 says `renderCommand` refuses "before any slot is taken", but card 9 moves `acquireSlot` ahead of `runStep`, where `renderCommand` runs.
    - The refusal therefore runs after the slot (and, for the main capture, after the lock) unless a card moves it.
    - Evidence: card 9, first bullet; card 26, `renderCommand` bullet.

11. **LOW** (card 22, in-flight runs): old Stuck reasons point at verbs card 22 removes.
    - A run parked at an old Drift-Verify decision halt carries a reason naming `lym drift accept`, `withdraw` or `waive`, which card 22 removes. Only the reason's `lym loom resume` still progresses.
    - No card notes this, though the recipe headers' in-flight text is the natural place.
    - Evidence: card 22 Intent; the current `verify.go` reason texts.

**Tool calls**

1. `grep` for an all-cards-done short-circuit in `internal/websterengine/runlevel.go` and `internal/loomshed/webster*.go`: the context did not show how Webster behaves on re-entry with every card done (finding 8). No match.
2. `grep` for `NewWebsterProducer` and re-entry text: same question; it located `internal/loomshed/webster.go` only.
3. `grep` for Done and complete handling in `loomshed/webster.go` and `Run`: same question. Inconclusive.
4. `sed` and `grep` over `runlevel.go:430-560` for early returns: same question. Only error returns were found.
5. `grep` for all-done markers across `websterengine`: same question. No match.
