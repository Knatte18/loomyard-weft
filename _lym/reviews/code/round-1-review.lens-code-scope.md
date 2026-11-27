```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the goal point or added entry it builds, and checked for decisions with no change behind them.

**Per decision**

- **G1. Close the scope at the first Code-Review round, and have lym hold the line.**
  - Refusal: built by C1, C2 and C4.
  - "The refusal reaches the parent and the agent": only partly built (F1, F2).
  - The PR-agent exemption: not built (F3).
  - The Finalize exemption: built. `InClosedScope` returns false at Finalize and later, so `fixerrun.go:361` still works.
  - "The orch procedure and the parent's prompt say the same": built in `orch-role.md` and `parent-directive-parent.md`, but other text left unchanged contradicts it (F1, F2).
  - Read-first of the darn, tapestry and arras perches: done. Arras has no line, per `TestInClosedScope`.
- **G2. Item 3's tier0 budget for the Plan:** built by C5.
- **G3. Item 4, Discussion records what the parent will check:** built by C6.
- **G4. Item 5, a finding ruled in scope reaches a fixer:** built by C3, with C4 as its wording on the escalation.
- **G5. Item 6, re-check a quoted constant:** built by C7.
- **A1. Added entry: drop item 3's C# half, keep the tier0 budget:** consistent. The diff makes no dotnet or tier change, and the summary's design decisions state the drop. Its "build (a), (b) and (c) as proposed" names no proposal the record holds, so no criterion can be traced to it.

**Per change**

- **C1. Scope-line guard** (`loomrecipe/scopeline.go` and its test, `loomrecipe/doc.go`, the refusal in `decision.go` with its deps, preRun and help, `loomcli/doc.go`, the `decision_test.go` fake, `TestDecisionCalls_RefusedOnceTheScopeCloses`): traces to G1.
- **C2. Prose of the line** (`orch-role.md`, `parent-directive-parent.md`, `loom-template-arbiter.md`, `arbiterRepairVerbs`, `docs/overview.md`): traces to G1, "the orch procedure and the parent's prompt say the same".
- **C3. In-scope ruling** (`--in-scope` on circling continue, `RecordInScopeContinue`, `InScopeFindings`, the `BuildRoundContext` mark, `inScopeMark`, the `carriedFinding` and `priorEntry` changes in roundcheck, `burler-step-review.md`, the `burlerengine` and `shedadapters` doc.go files, and their tests): traces to G4.
- **C4. The `scope_closed` Bouncer key** (three recipes, `entries_bouncer.go`, `noticeWayForward`, `escalationReason`, `answerRoute`, `decideVerbs`, the decision fragment and its `answer_route` marker, the escalation template, `markers_test.go`, the pre-Motif fixture, the bouncer test): traces to G1 and G4. It is forced, because without it the escalation would name the `decision add` that the line refuses.
- **C5. Tier0 budget** (the `gateslot` config, template, `TiersValue` and its tests, the plan spec, the plan-cards block, the plan-review rubric): traces to G2.
- **C6. Acceptance criteria** (discussion spec, discussion template, discussion-review rubric, describe-review template): traces to G3. The describe-review line is a forced follow-on: it is the reader of the criteria the description states.
- **C7. Quoted figure** (code-review rubric bullet, the darn make-change fragment): traces to G5.
- **C8. Shortened `decision add --help` text** for `--amends-plan`, `--from` and `--drops-create`, and the board-copy sentence: forced by C1's added help paragraph and the help length cap (commit 5102cf8a6). Scope-wise it is no finding.

**Findings**

- **F1. MEDIUM, scope, `internal/shedadapters/burlerfix.go:105`.** G1's "the parent files a note instead of trying" is not built for the Code-Burler-Fix notice. That notice still tells the parent "Way forward: record each call with \"lym loom decision add &lt;slug&gt; --by parent\"", and it runs inside the closed Code-Review segment, where the diff's refusal now always applies. Evidence: `burlerfix.go:105`; `InClosedScope` (`scopeline.go`); the `DarnInsideTheSegment` row in `scopeline_test.go`.
- **F2. MEDIUM, scope, `contracts/orchrole/role-autonomy.md:19-23`.** G1's "the orch procedure says the same" is built in `orch-role.md` alone. This is a recurring enumeration gap, filed once.
  - The on-demand role files still send the parent to `decision add` at rows inside the closed span:
    - `role-autonomy.md:22-23`: settle a drift does-not-match stop "with a decision and a `goto` back". That stop comes from Drift-Recheck, which `scopeline_test.go`'s `DarnDriftRecheck` row marks closed.
    - `role-autonomy.md:19`: settle a design choice with `decision add`.
    - `role-landing-halts.md:22`: a landing-stop halt at Parent-Align. Parent-Align is closed once a round started, because `roundStarted` holds whatever row the run stands at.
  - Unchanged lines of `orch-role.md` do the same: 196 (a design call the child cannot make), 324 (scope correction) and 327 (`--amends-plan` with a goto).
  - The same gap holds for the agent half ("the refusal reaches … the agent"): the darn writer's asks in `darn-template-write.md:53,91,117`, `darnStopWayForward` (`loomshed/darnwrite.go:32`), `planConflictWayForward` (`shedadapters/webster.go:38`), and the sampler agent template line 38. A run sent back to the writer after a started round meets each of these.
  - Evidence: the files and lines above; the diff piece for `contracts/orchrole` changes `orch-role.md` alone.
- **F3. MEDIUM, scope, `internal/loomrecipe/scopeline.go:1227-1251`.** G1's first exemption, "feedback from a person through the PR agent", is not built.
  - The closed span runs through PR-Review (`scopeline_test.go`, row `DarnPRReview` want true), and the refusal covers `--by operator` too (`TestDecisionCalls_RefusedOnceTheScopeCloses`, the `Operator` case).
  - So a person's feedback at the pull request has no route into the record. `landingshed/prreview.go:43` confirms that no agent takes part at PR-Review.
  - The summary's "Feedback the operator gives on the pull request needs no entry in the record" and `orch-role.md`'s new "stays outside the line" assert the exemption, but no code makes it.
  - The change description does not name this as a parent correction.
  - Evidence: `scopeline.go:1247-1250`; `scopeline_test.go`, row `DarnPRReview`; `decisionreplace_test.go`, `Operator` case; `prreview.go:43`.

Tool calls: none. The inherited context held every file and line cited.
