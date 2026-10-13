## Interview

Autonomous run: no operator answered; every pick below is the writer's own and is ledgered as an auto-pick.
The record faces base `800a0e892`, read through `git show` and `git grep`, as the board entry's bench section requires.

1. Board read: `twb-ftb-2` and the entries touching Webster (`arbiter-fixes`, `weaver`, `glyph-use-audit`, `enclose-kickstart`, `logging-cleanup`).
   `twb-ftb-1`, `twb-fta-1`, `twb-ftb-3`, `twb-fta-2` are bench arms of this same task, not work to coordinate with.
2. Issue 1 explored: at base no background shell expires; a gated run's waiting turn end with both output files present counts at once when every outstanding task is an unawaited shell.
   Master declares one awaited prefix, `lyx webster recover-batch`; a `lyx gate test` shell is unawaited, so the turn end counts, the gate passes and strand removal kills the shell.
   Pick: add `lyx gate test` as an awaited prefix (D1).
3. Issue 2 explored: the Master stencil already says "Do NOT localize or fix the failure yourself" under the gate-failure section, but shuttle's generic re-prompt reads "read it, fix every finding, and end your turn", and the dirty-tree findings say "Commit or remove these paths".
   Pick: a per-entry re-prompt instruction and a findings closing line (D2).
4. Issue 3 explored: `loadRunWrites` audits Master sessions and their forks only; a recovery strand is its own session, and its batch record keeps no session id, so its writes are never attributed.
   `recover-batch` returns a terminal stuck or dead digest through `output.Ok`, exit 0.
   Picks: D3, D4.
5. Issue 4 explored: the settings deny only raw `go test` with `tmux` or `llm` tags; `lyx gate test --tags tmux` is allowed, so the recovery's doubt came from the prompt.
   The plan spec says verify compiles the tmux tier and Publish runs it.
   Pick: D5, following the board entry's expected wording.
6. Issues 5 and 6 explored: quarry v0.2.1 resolves test-file members and gives no sign of filtering unexported ones; it answers `ambiguous` for build-tag duplicate pairs by design (`internal/engine/testdata/tags`), with each candidate's file.
   `internal/proc#StartTime` is declared in `proc_linux.go` and `proc_windows.go` at base.
   `DoneChecks` admits only found or multipart and reports anything else as "still does not resolve".
   The first draft took issue 5's targets for build variants; the premises lens showed that wrong (see turn 7), so issue 5 became D8.
   Picks: D6, D7, D8.
7. Lens review: six forks spawned, all six returned a report. Findings, merged across lenses, with disposition:
   - premises, BLOCKING, issue 5 root cause claimed as build variants: fixed; verified at HEAD that `tmuxAllowed` is one `var` and `rusageStats` one `func`; D8 added, the claim removed from D6.
   - goal-scope, MEDIUM, issue 5's "fix it if not" had no route: fixed in D8.
   - goal-scope, MEDIUM, D6 reads "flags" as "passes": fixed; an informational `glyph-build-variant` finding at Plan time.
   - goal-scope, MEDIUM, D3 contradicted its own open risk on the session id: fixed; D3 states the requirement and the fallback, Q8 closed.
   - goal-scope, MEDIUM, D4 assumed the incident path: fixed; every path reporting a terminal non-done batch exits non-zero, re-entrant attach included.
   - pattern-fit, BLOCKING, D4's non-zero exit lacked a way forward and a refusal row: fixed; the failure-ladder rung is the way forward and a row lands with it.
   - premises and scenarios, MEDIUM, D4 left the Master stencil unchanged: fixed; the recover-batch rungs tell a digest from a refusal by `status`.
   - scenarios, BLOCKING, an unauditable recovery transcript would fail the verb: fixed; it counts as no writes, with a warning.
   - scenarios, BLOCKING, D1 had no restart scenario: fixed; the hold is per session.
   - scenarios, MEDIUM, nothing cleared recorded recovery sessions: fixed; resets of the batch and `run --fresh` drop them.
   - scenarios and goal-scope, MEDIUM and LOW, reset widening unbounded and untested: fixed; bound and criterion added, destruction chokepoint named.
   - scenarios and leanness, MEDIUM, D6 vs `answerSymbols` and the reader list: fixed; D6 states the reader rule, the delete-ordering check stays a still-present reader, candidates become symbols.
   - pattern-fit, MEDIUM, GOOS/GOARCH list source would spawn: fixed; a fixed table in code, risk kept in Open risks.
   - pattern-fit, MEDIUM, disjointness as free booleans is wrong for GOOS: fixed; configurations are enumerated and matched with `go/build`'s rules.
   - premises, LOW, `createFindings` would lose its candidates: fixed; the detail keeps them.
   - premises, LOW, the binding path's status handling not located: fixed by noting it for the plan; criterion kept.
   - scenarios, LOW, mixed-kind candidates: fixed; same kind is required, with a criterion.
   - scenarios, LOW, the one-line check fires only at send: fixed; validated at start.
   - scenarios, LOW, empty or wrapped labels: fixed; stated as unchanged, with a criterion.
   - scenarios, LOW, `llm`-tagged steps would run: fixed; D5 skips `tmux` and `llm`.
   - pattern-fit, LOW, D5 paraphrased the plan spec: fixed; the prefix points at it.
   - pattern-fit, LOW, Go-rendered re-prompt under stencil ownership: fixed; Constraints say so.
   - board-fit, MEDIUM, `weaver` reuses the recovery path: fixed; Open risks names it.
   - board-fit, MEDIUM, agent-side quarry readers still see ambiguous: fixed; scoped in Out.
   - board-fit, LOW, D4's field list vs `arbiter-fixes`: fixed; no list, `arbiter-fixes` named.
   - board-fit, LOW, quarry candidate file reliance: fixed; a Constraints line.
   - board-fit, LOW, the gate timeout premise was unverified: fixed; the claim was dropped.
   - board-fit, LOW, the new Master stencil line is Claude-specific: rejected; the Master stencil already keys every rung on completion notifications, and `llm-agnostic-engine` owns that.
   - goal-scope, LOW, `verify_gate_attempts: 0` may make the run ungated: rejected; shuttle tests `len(gate) == 0` and the off entry is still appended, as the premises lens confirmed.
   - leanness, MEDIUM, rejected alternatives in Auto-mode assumptions: fixed; cut, they stay in Rejected alternatives.
   - leanness, LOW, rationales arguing against rejected options, D7 repeating an assumption, the open risk holding a design choice: fixed.
   - leanness, LOW, function names in the Decisions: partly fixed; names kept only where the Decision is about that seam.
   - leanness, LOW, post-base code cited as fact: fixed with the premises finding.

## Rejected alternatives

- Issue 1, lyx stops the gate shell and records its result as discarded: shuttle reports no pid, so lyx cannot stop a shell, and a discarded result still leaves a tier unrun.
- Issue 1, awaiting every background shell: would hold Master on any long-lived shell it starts by mistake; the gate prefix names the one class the fault concerns.
- Issue 2, a `lyx webster` verb that spawns the verify fixer: forks are spawned only through the Agent tool, so a CLI verb cannot spawn one; a cold strand would lose the fork's inherited context.
- Issue 2, stencil-only change: the stencil already states the rule; the contradicting message is what Master reads at the failure.
- Issue 3, a clean tree for the second recovery with the first attempt's diff as reference: cleaning destroys uncommitted work outside `PATTERN-fabric-destruction-chokepoint` and makes the recovery redo work it could keep.
- Issue 3, attributing every uncommitted path in the batch's card targets to the batch: a card target can be touched by a foreign process; session evidence is what the classing is built on.
- Issue 3, failing the verb on an unauditable recovery transcript: one lost transcript would block every later recovery of the batch.
- Issue 6, quarry honours build constraints: a cross-repo change and version bump for a policy quarry deliberately leaves to callers; its testdata pins the ambiguity.
- Issue 6, accepting any ambiguous answer in `DoneChecks`: would pass a real duplicate declaration.
- Issue 6, build-constraint satisfiability over free boolean atoms: treats `linux` and `windows` as jointly satisfiable.
- Issue 6, the GOOS/GOARCH list from `go tool dist list` at run time: a spawn per check, and a toolchain the binary may not have.
- Issue 6, a Plan-time warning for Create glyphs that may become build variants: Plan time cannot know a member the plan creates will be split.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): hold the gate shell or stop it? Hold, via an awaited prefix (D1).
- Q2 (auto-pick, resolved): rule in the message or a verb? Rule in the re-prompt and findings text (D2).
- Q3 (auto-pick, resolved): attribution by batch or clean tree? Attribution through recorded recovery sessions (D3).
- Q4 (auto-pick, resolved): which terminal statuses exit non-zero? Every status but `done`, on every path (D4).
- Q5 (auto-pick, resolved): which step kinds the recovery skips? `**Verify:**` steps tagged `tmux` or `llm` (D5).
- Q6 (auto-pick, resolved): where the build-variant reading lives? `internal/planglyph`, shared by Plan time and record-batch, with an informational Plan-time flag (D6).
- Q7 (auto-pick, resolved): what caused issue 5? Unconfirmed; the plan reproduces it first (D8).
- Q8 (auto-pick, resolved): when the recovery's session id is persisted? At the first moment shuttle knows it, with a stated fallback (D3).
