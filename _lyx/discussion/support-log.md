# Support log — webster-stability

## Interview

The session ran autonomously, so every question below was posed and answered by the Discussion producer after reading the code.

1. Is item 1 still open at HEAD?
   Commit 5a8be0274 already made shells expire nothing and record `EndedShells`, but `countsAtOnce` still finishes a fresh gated run at once when only unawaited shells are outstanding and the output files exist.
   That is the path the three issues describe.
   Pick: remove the at-once count for gated runs; hold on the existing deadline.
2. Hold or stop?
   lyx has no handle on a provider's background shell; strand removal is what ends it.
   Holding reuses the fork rule and the completion notification that starts Master's next turn.
   Pick: hold.
3. Why did Master fix the verify failure itself?
   The generic re-prompt reads "read it, fix every finding, and end your turn".
   The stencil's rule lost to the message.
   Pick: a per-entry re-prompt line and a way-forward line atop the findings; no new verb, since an in-session fork can only be spawned by Master's Agent tool.
4. Why did the second recovery disown the first one's files?
   `recordedSessions` lists the Master session and each batch's begin-time Master session; a recovery strand's session is nowhere in state.
   Pick: record recovery session ids on the batch, audit them, keep the contract-file evidence on Master alone.
5. Why did `recover-batch 1` exit 0 on a failed batch?
   The CLI exits non-zero only on `ErrBatchFailed` and refusals; a `stuck` or `dead` digest returns `ok: true`.
   The exact path in the quick-recipe run cannot be recovered from the code.
   Pick: non-zero for every terminal non-done record.
6. What is the recovery unsure about in item 4?
   The body states a sandbox ban on mutating `lyx fabric` verbs and a raw-`go` rule, never an allow list; the tier table that says who runs `tmux` lives in PATTERN only.
   Pick: name the allowed commands and the compile-only form for tagged tests.
7. Does quarry resolve unexported or test-file identifiers?
   Probed at HEAD: `internal/planglyph#resolveKeyFor` found, `internal/planglyph#TestCreateFindings_ResolvedTargets` found, `cmd/testtiming#rusageStats` found.
   At the time of #511, `cmd/testtiming` held `rusage_linux.go` and `rusage_other.go`, so both #511 targets were the build-constraint case.
   Pick: no resolver change; the detail carries the lookup.
8. Who decides that an ambiguous answer is one member?
   quarry is an external pinned module; its `Symbol.File` names each candidate's file.
   Pick: a classifier in planglyph over the candidate files, exact through `go/build.Context.MatchFile` over enumerated environments.
9. Which policies change?
   Edit, Delete, Rename, `Uses:`, the Create done-check, `answerSymbols` and the plan-gate member reads accept a partitioned answer; Delete-still-exists and `create-already-exists` do not.

## Rejected alternatives

- Stopping the shell from lyx and recording the discard (item 1): lyx has no process handle on a provider shell; the record already exists, and holding reads the result instead of discarding it.
- Expiring a shell after `background_shell_wait_min` again: the exact behavior #496 removed.
- Applying the hold to ungated runs: no fault seen there, and a stray shell would hold every review producer to its deadline.
- A `lyx webster` verb that spawns the verify fixer (item 2): an in-session fork is spawned only by Master's own Agent tool, so no verb can do it.
- A gate-time parent-write check failing the gate `Terminal` on a Master self-fix: the run-exit audit already catches it, and the result is the same ended run.
- A per-result re-prompt field instead of a per-entry one: the way forward does not vary by evaluation.
- Starting the second recovery from a clean tree with the first attempt's diff as reference (item 3's second option): discards nineteen files of work the run itself produced, and needs a new diff artifact.
- Attribution keyed on the shuttle run id of the batch record: the run id changes per spawn, which is the fault.
- Folding recovery writes into the Master slice: would let a recovery's `outcome.yaml` write read as Master's rewrite in the contract-file evidence.
- Exiting non-zero for `failed` only: the board named the failed case, but a reader cannot predict a rule that treats `stuck` and `dead` differently from `failed` when all three end the run stuck.
- Telling the implementer to run `tmux`-tagged tests through `lyx gate test --tags tmux`: the tier table runs that tier at Publish, and a strand compiling it is enough.
- Changing quarry to honor build constraints: an external module at a pinned version; the plan-time and done-check policies are lyx's.
- A textual disjointness rule ("every candidate has a constraint and no two are equal"): fails open on `//go:build linux` beside `//go:build amd64`.
- Treating a partitioned Create target as "does not exist yet": a declaration occupies the name whichever file holds it.
- Letting a Create reuse a name the same card deletes (#507's first option): Edit now covers per-constraint bodies, and the resolve pass reading the tree before any card runs is by design.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Answer |
|---|----------|--------|--------|
| 1 | Hold the turn end or stop the shell? | resolved, auto-pick | Hold; bounded by the run deadline and liveness (D1). |
| 2 | Apply the hold to ungated runs? | resolved, auto-pick | No (D1, scope). |
| 3 | Per-entry or per-result re-prompt text? | resolved, auto-pick | Per-entry (D2). |
| 4 | Add a mechanical guard against Master self-fix at the gate? | resolved, auto-pick | No; the run-exit audit covers it (D2). |
| 5 | Record recovery sessions or start the second recovery clean? | resolved, auto-pick | Record and audit them; contract-file evidence stays Master-only (D3). |
| 6 | Which terminal statuses exit non-zero from `recover-batch`? | resolved, auto-pick | Every terminal non-done: `failed`, `stuck`, `dead` (D3). |
| 7 | What does the implementer do with a `tmux`- or `llm`-tagged test? | resolved, auto-pick | Compile only through `lyx gate test --tags <tag> ... -- -run '^$'`; the later gates run it (D4). |
| 8 | Does the resolver skip unexported or test-file symbols? | resolved by probe | No; both resolve `found` at HEAD (D5). |
| 9 | Where does constraint partition live? | resolved, auto-pick | `internal/planglyph`, over `go/build.Context.MatchFile` (D6). |
| 10 | Exact evaluation or a textual rule? | resolved, auto-pick | Exact over enumerated environments with a tag cap (D6). |
| 11 | Does a partitioned Create target pass the Create inversion? | resolved, auto-pick | No; it stays `create-already-exists` (D6). |
| 12 | Is #507's Create-after-Delete reuse in scope? | resolved, auto-pick | No; Edit covers it (scope). |
| 13 | Why did the quick-recipe `recover-batch 1` exit 0 on a failed batch? | open, unrecoverable | The run's records are not in this worktree; D3's rule covers every path. |
