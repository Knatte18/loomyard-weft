```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens over D1–D10. I traced the existing code paths that write or read each decision's state.

**Paths traced, per decision**

- D1, `hubreconcile` and template seeding of worktree `beam.yaml`: survives as far as I could check. PATTERN-config-layers says a base file holds only tuned units, and no seeding path turned up in fabricengine or configengine.
- D1, Finalize `configChangeNotice` (`internal/landingshed/finalize.go:621`): survives. D1 names the exclusion.
- D2, `orchcli.ParentStrand` → `orchroute.FromPair` → `From` (`internal/orchroute/walk.go:92-117`): survives. The value travels on the notice.
- D3, the drift `parentAgreement` read of `cfg.JudgeAskParent` at call time (`internal/driftengine/verify.go:838`): survives the forcing.
- D3, the judge ask's end states other than timeout, `not-sent` and `no-parent` (`internal/driftengine/judgeparent.go:170`, `internal/burlerengine/parentreview.go:33-37`): breaks, F-A.
- D3, `ParentReviewWanted` loading `loomengine.LoadConfig` over the pair's layers (`internal/loomcli/landingdeps.go:468-471`): survives only through D2's run reader, as the Notes say.
- D4, the existing `parent-review answer` and `AnswerParentAsk` store (`internal/loomcli/parentreview.go:180-230`, `internal/burlerengine/parentreview.go:336-376`): breaks, F-B.
- D4, `ensureSession` re-entered on every pause stop and resume and on every judging stop and resume (`internal/landingshed/prsession.go:64-102`, `internal/landingshed/prreview.go:493-523`, `prreview.go:664-679`): breaks, F-C.
- D4, a goto against a resume on re-entry of `PRReview.Call` (`prreview.go:75-92`, `prsession.go:80`): breaks, F-C.
- D4, the row's `judge` and `sessionStopWanted` ending on a standing `Continue` flag (`prreview.go:694-695`, `prreview.go:880-885`): breaks, F-D.
- D4, `settleSendBack` clearing `ParentReviewAsked` and the session (`prreview.go:122-125`): the record is silent on whether the ask numbers and the recorded autonomy are cleared with them. This is folded into F-C.
- D4, hold notices for a non-interactive session (`internal/shuttleengine/hold.go:115`, `hold.go:336`): survives, since `Interactive` is false.
- D5, status history writes, `appendOrFold` and `foldsInto` (`internal/shedengine/run.go:706-739`), and `RecordResumer` writing no entry (`internal/shedengine/goto.go:287-307`): breaks, F-E.
- D6, the await-marker writers, `landOrRefuse` and `clearDownNotice` (`internal/pawlengine/beamawait.go:269-310`): survives. The drain marker is a separate file.
- D8, `replyHook` (`internal/slackcli/hook.go:162-197`): survives, with the filter added after the presence check.
- D9, `orchengine.WithPresence` (`internal/orchengine/prompt.go:36-48`): breaks, F-F.

**Findings**

F-A — MEDIUM, design, `_lym/discussion/decision-record.md:99-105` (D3 Bound)
D3 stops the run under autonomy only on a judge ask that times out unanswered. An ask that ends `not-sent` lets the rulings stand with neither operator nor parent, and D3 does not bound that case. `not-sent` means the cap left no room, and the gate passes it today. An ask that ends `no-parent` does the same.
Evidence: `internal/driftengine/judgeparent.go:170`, `internal/burlerengine/parentreview.go:33-37`.

F-B — BLOCKING, design, `_lym/discussion/decision-record.md:128-131` (D4)
D4 has `lym loom parent-review answer <slug> --ask <n>` record the answer against the ask in the pull request record. The existing answer verb behaves differently in three ways:
- It writes only burlerengine's never-tracked ask record.
- It answers only that record's latest ask.
- It refuses past a deadline that every ask carries, and sets the ask timeout.

D4 neither marks the verb as changed nor says how a PR-Review ask fits that record: its seed, its tag, its deadline, and whether a Code-Review ask can stand latest over it. As written, the orch's answer lands in burlerengine's record. `pr-continue` reads the pull request record, finds no answer there, and refuses every time.
Evidence: `internal/loomcli/parentreview.go:180-230`, `internal/burlerengine/parentreview.go:64-66` and `:336-376`, `internal/loomcli/prcontinue.go:31-57`.

F-C — MEDIUM, design, `_lym/discussion/decision-record.md:117-120` (D4)
D4 records autonomy "when PR-Review starts its session" and says a goto "starts a new session". In the code:
- `ensureSession` rebuilds the spec and starts or resumes the session after every pause stop and resume. It does the same after every judging stop that leaves the review going. So "at session start" re-records the live value repeatedly, and a mid-review toggle takes effect. That is what D4 and its acceptance bullet rule out.
- A goto and a `lym loom resume` reach the same `Call` → `openPullRequest` → `ensureSession` path, which resumes the recorded `SessionID`. The row cannot tell them apart and starts no new session.
- `settleSendBack` clears the session fields. D4 does not say whether the ask numbers and the recorded autonomy are cleared with them.

Evidence: `internal/landingshed/prsession.go:64-102` (line 80 resumes `rec.SessionID`), `internal/landingshed/prreview.go:493-523`, `prreview.go:664-679`, `prreview.go:122-125`.

F-D — MEDIUM, design, `_lym/discussion/decision-record.md:146-147` (D4 Bound)
D4's bound says only a recorded satisfied answer at the current head ends the review. Only `pr-continue` checks that. The row ends the review on any standing `Continue` flag once the tree is clean and the head is gated.

Counterexample:
1. The session runs `pr-continue` after a satisfied answer at head A, so `Continue` is set.
2. The agent commits B before the next wake.
3. `sessionStopWanted` returns false and the session keeps working.
4. The row gates B, then `judge` calls `endReview` at B, which no ask was sent at.

Evidence: `internal/landingshed/prreview.go:694-695`, `prreview.go:880-885`, `prreview.go:635-649`.

F-E — MEDIUM, design, `_lym/discussion/decision-record.md:177-179` (D5 class 5)
D5 counts earlier halts at a row as entries in the status history "lym writes on every halt". But `appendOrFold` folds a halt into the previous entry when both are budget-exempt Stucks with equal Producer, Output, RoutedTo and GateAttempts. A Halt is budget-exempt. `lym loom resume` writes no history entry: `RecordResumer` sets only `PendingBy`.

So a halt, its resume and the same halt again at that row leave one entry with `Repeats` counting up. An orch that counts entries never reaches two, and class 5 never fires. The record must say to count each entry plus its `Repeats`, or name the fold.
Evidence: `internal/shedengine/run.go:358-370` and `:706-739`, `internal/shedengine/status.go:98-104`, `internal/shedengine/goto.go:287-307`.

F-F — MEDIUM, design, `_lym/discussion/decision-record.md:254-256` (D9)
D9 puts the autonomy line on `orchengine.WithPresence`'s path. That function returns the pointer unchanged when the presence path is empty, the presence is unreadable, or the presence is not slack. Under a terminal presence, the usual case, no start, resume, reload or pending-resume pointer carries the autonomy line. D9's acceptance bullet asks for exactly that line.
Evidence: `internal/orchengine/prompt.go:36-48`.

**Tool calls**

- `sed internal/loomcli/landingdeps.go:460-490`: how `ParentReviewWanted` loads `pr_review_ask_parent`. The manifest showed only its comment.
- grep fabricengine and configengine for worktree `beam.yaml` seeding: whether a fork or seed writes the template's keys into a worktree file, which would mask a default-layer `true`. Nothing turned up.
- grep shuttleengine for hold notices against `Interactive`: whether a non-interactive PR-Review session raises the hold notices D4's bound relies on. The manifest lacked this.
