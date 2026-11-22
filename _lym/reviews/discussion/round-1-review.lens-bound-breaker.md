```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee and existing-code claim in the decision record.

**Claims attacked**
- D1, the local layer refuses a Guard key: held (`internal/configengine/layers.go:166-168`, `:271`).
- D1, a worktree's file is inherited at fork: not broken from what I read, and left unpinned (the handler covers pins).
- D1, Finalize's notice leaves `autonomy` out: held as a design statement.
- D2, "a run's notices reach its parent worktree's nearest live orch, so the run and that orch agree on one value": **broken**.
- D2, "dobby loads `beam.yaml` strictly, so a broken file stops the dobby, which the orch hears of": **partly broken**.
- D2, a failed load reads `false`, so the operator is asked: held.
- D3, forcing `judge_interactive` off with its bound left unstated: **broken**, the judge timeout lets a ruling pass.
- D3, "a raddle filing makes draft tasks, so their release stays the operator's": held (`internal/loomshed/raddlefile.go:137`). It clashes with Scope Out (finding 10).
- D4, "only a recorded satisfied answer at the current head ends the review": **broken**, because `pr-continue` has a second use.
- D4, asks go "through the parent-review seat with an ask number, as code-review asks do": **broken**.
- D4, held turns raise hold notices, as an autonomous session's do today: held (`internal/shuttleengine/doc.go:68`).
- D4, a GitHub human-approval rule refuses the merge and is a class 4 halt: held.
- D5, "Class 5 bounds the orch's own settlements at two per row": **broken**.
- D6, "the posted drain is kept as a marker in the beam run's records, so a re-entry posts no second line": **broken or underspecified**.
- D6, "Beam-Await … sees a drained beam without a new poller": partly broken, because new seams are needed.
- D7, step 1's "when the beam lags its parent, so `lym beam pr open` would refuse": **broken, the premise is false**.
- D7, "its merge task starts released": held (`internal/beamcli/mergeparent.go:23-24`).
- D7, the bound "the go-ahead covers only these four steps": **broken** against Scope Out and the orch role.
- D8, the bound "drops only unmarked replies, only while presence is slack, the pane shows every reply": the bound holds as worded, but a forgotten marker has no backstop.
- D9, "every orch pointer … on the same path that appends the presence line": **partly broken**.
- D9, "a write through `lym config beam` mails each live orch whose value it changed": **broken by other writers**.
- Constraint, `internal/shellguard/rows.go` holds no row against the named verbs: held (grep found none).
- Constraint, Beam-Await's wakes back off to one minute: held (`internal/pawlengine/beamawait.go:27`).

**Findings**

1. **BLOCKING · design · D2 (Rationale) and D1's opt-out**
   - A notice does not always reach the parent worktree's orch. `orchcli.ParentStrand` → `orchroute.FromPair` → `From(start, live)` walks up the beam chain to the nearest *live* orch and falls back to the prime's orch.
   - Counterexample: the hub default is `true` and beam B sets `false`. B's orch is down. A member run of B reads B's `false`, but its halt goes to the prime's orch, which reads its own `true`. That orch settles and resumes B's run with no operator, so B's opt-out is bypassed.
   - The reverse case also breaks: the run is forced non-interactive while the receiving orch waits for the operator.
   - Evidence: `internal/orchroute/walk.go:107-117`, `internal/orchcli/notify.go:177-182`.

2. **BLOCKING · design · D7 step 1**
   - The premise "the beam lags its parent, so `lym beam pr open` would refuse" is false. `judgeOpenable` refuses only on an open merge task or a conflict: "A beam only behind its parent passes, since GitHub merges it".
   - Step 1 as written runs `merge-parent` on any lag. On a busy parent, each merge task's landing drains the beam. By then the parent has moved again, so the next drain files another released merge task. The loop has no bound.
   - Evidence: `internal/beamcli/pr.go:236-258`.

3. **MEDIUM · design · D5 Bound (class 5)**
   - "Class 5 bounds the orch's own settlements at two per row" counts only `--by parent` decision entries. Many settlements record no decision: a plain `lym loom resume` granting a bounce budget, the hand pass `lym batten run`, `lym loom goto` plus resume, and `circling continue`.
   - So an orch can re-settle the same row's halt indefinitely, and class 5 never fires.
   - Evidence: `contracts/orchrole/orch-role.md:55-56`, `:160-169`.

4. **MEDIUM · design · D3 (`judge_interactive` forced false)**
   - The operator's judge seat is removed, and the only stand-in is `judge_ask_parent`. Under that key, a late or missing parent answer is refused and "the judge's rulings stand as written" once `judge_parent_timeout_min` expires.
   - So a slow or dead orch lets matches rulings pass with neither the operator nor the parent. D3 states no bound for this.
   - Evidence: `contracts/orchrole/orch-role.md` (Reviewing a decision record / judge section: "a late answer is refused, and the judge's rulings stand as written"), `internal/driftengine/config.go:93-94`.

5. **MEDIUM · design · D4 (`pr-continue` refusal)**
   - `pr-continue` also clears `MailedHead`, so that the row gates a head again after a gate failure from outside the change.
   - Under D4 that re-gate needs a satisfied parent answer at the current head. Worse, the one call both re-gates and ends the review.
   - The closed-PR session also "cleans the tree under a continue flag", and D4 does not cover it.
   - The record treats `pr-continue` as the ending alone.
   - Evidence: `internal/loomcli/prcontinue.go:26-51`, `internal/landingshed/prreview.go:716`, `:317-325`.

6. **MEDIUM · design · D4 ("through the parent-review seat … as code-review asks do")**
   - `ParentReviewSeat` is a Go seat driven by a review producer or Bouncer over burler round files. Its round identity is the round-1 focus file, and it records no head.
   - No verb lets the PR-Review session's agent send the re-ask after its fixes. Today that re-ask is a free creel message.
   - Who sends ask n+1, and where its head is recorded, is unstated. It is no existing mechanism.
   - Evidence: `internal/shedadapters/parentseat.go:21-45`, `:60-62`, `internal/landingshed/prreview.go:356-393`.

7. **MEDIUM · design · D9 (change mail)**
   - Only a `lym config beam --set` write mails the orch. Other paths change the resolved value silently:
     - the editor save of `lym config beam` (`internal/configcli/doc.go:60`);
     - `lym config reconcile`;
     - a hub pull bringing another machine's default-layer change.
   - D8's hook reads autonomy at call time. So when the orch believes `false` while the hook reads `true`, every reply lacks `[operator]` and nothing reaches Slack, severe-halt relays included.
   - Evidence: `internal/configcli/doc.go:50-60`, `internal/slackcli/hook.go:162-197`.

8. **MEDIUM · design · D8 Bound**
   - Under autonomy, a severe-halt or merge-ready reply the orch forgets to mark is dropped from Slack with no backstop. The pane is no help while the operator is away.
   - The bound names this loss but does not bound it. The presence line tells the orch to also post to `<shortname>:SLACK`, but D8 does not say that this route survives as the fallback.
   - Evidence: `internal/slackcli/hook.go:162-197`, `internal/orchengine/prompt.go:29`.

9. **MEDIUM · design · D6 (drain marker "in the beam run's records")**
   - Pawl's existing markers live in the untracked `.lym/beam/` directory of the beam worktree, and Beam-Up removes an await marker an earlier run left.
   - A marker kept beside them is lost on a takeover onto another machine or a new pawl run, so a second `[beam drained]` line posts.
   - "Records" could mean the tracked store, which needs a writer; the record does not say which.
   - Evidence: `internal/hubgeom/dobbygeom.go:77-88`, `internal/pawlengine/beamup.go:58`.

10. **MEDIUM · consistency · D7 step 1 and D3 against Scope Out**
    - Scope Out says "making tasks … stay the operator's". D7 has the orch file a released merge task on its own go-ahead, and the orch role allows a step that starts work only on the operator's explicit go-ahead.
    - D3's forced `raddle_auto_approve` also makes draft tasks with no operator approval.
    - Evidence: `internal/beamcli/mergeparent.go:23-24`, `contracts/orchrole/orch-role.md:423`, `internal/loomshed/raddlefile.go:137`.

11. **LOW · consistency · D2 Rationale ("which the orch hears of")**
    - A dobby stopped by a load error posts to the orch only after a re-exec. Otherwise it only logs its own line.
    - The fallback still reads `false`, so the operator is asked, but the claim is false as stated.
    - Evidence: `internal/dobbyengine/loop.go:236-241`, `internal/dobbyengine/starts.go:95-98`.

12. **LOW · design · D6 Rationale ("Beam-Await … sees a drained beam")**
    - Beam-Await's `Deps` has no seam for member tasks, child pairs or commits beyond the parent.
    - The drain check needs new seams, not only the existing wakes.
    - Evidence: `internal/pawlengine/deps.go` `Deps` fields (lines 22-65).

13. **LOW · consistency · D9 ("on the same path that appends the presence line")**
    - `WithPresence` wraps only the start, resume and reload pointers.
    - The handoff and soft-handoff mail bodies and the `/compact` focus are rendered without it, so "the same path" does not reach every pointer a context cycle sends.
    - Evidence: `internal/orchengine/watcher.go:1173-1175`, `:1578`, `:1778`.

**Tool calls**
- `grep FromPair` in `internal/orchroute`: the routing walk behind `ParentStrand` was not in context.
- `ls` and grep of `internal/shellguard`: to confirm `rows.go` exists and check its rows, since the earlier grep showed nothing.
- grep raddle draft filing in `internal/loomshed`: whether raddle files tasks as drafts.
- grep `AwaitMarker:` wiring: where the pawl markers are rooted.
- grep of the compact and handoff render callers: whether they pass `WithPresence`.
- `sed` of `internal/hubgeom/dobbygeom.go:60-95`: the `.lym/beam/` root of `PawlPaths`.
