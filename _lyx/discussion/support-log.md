# Support log — twb-so-oh-1

## Interview

The session ran autonomously, so each turn below is a question the agent put to itself, answered from the code at base `800a0e892` and the linked issues.

1. Is the board entry's item 1 still open at base?
   Partly.
   Commit `5a8be0274` removed the ten-minute shell expiry and added `Result.EndedShells`, warnings and a friction note.
   But a gated run's waiting turn end with every output file present still finishes Done at once when every outstanding task is an unawaited shell, and only `lyx webster recover-batch` is awaited, so a gate shell is still cut off by strand removal.
2. Hold the gate shell, or stop it?
   Hold it: lyx has no pid to stop it with, and holding is the issue's first expectation.
   Labels are the command text and often start with `cd <pair> &&`, so prefix matching on the whole label misses; match per command segment.
3. Where does the segment splitter live?
   `internal/shell`, stdlib-only, which owns shell syntax; `planparser`'s chain splitter is unexported and shuttle may not import planparser.
4. Why did Master fix a verify failure itself?
   Shuttle's generic gate re-prompt reads "read it, fix every finding, and end your turn", which is what Master reads at the failure, contradicting its stencil section.
   Fix: a per-entry re-prompt clause, set by webster's verify gate.
5. Why did the second recovery see the first's files as foreign?
   `loadRunWrites` audits Master and fork sessions only; a recovery strand is its own session and is never audited, and a respawn builds a fresh `BatchState` that drops the earlier strand's record.
   Fix: record every recovery session per batch and audit them.
6. Why did `recover-batch` exit 0 over a failed batch?
   Every terminal digest, done or not, goes out through `output.Ok`; only a post-check rejection exits non-zero.
7. What confused the recovery strand about gates?
   Its prefix bans `lyx webster`, the shared body bans mutating `lyx fabric`, and neither says the gates are outside both bans; the plan spec says the `tmux` tier is compiled with `go vet -tags tmux` and run at Publish, which the body never states.
8. Do unexported or test-file identifiers fail to resolve (#511)?
   No evidence: quarry's resolve has no exported filter and keeps `_test.go` symbols.
   Both #511 targets were declared once per build-constraint set (`tmuxAllowed` in `allow_tmux.go`/`allow_none.go`; `rusageStats` beside `rusage_other.go`), so the answer was `ambiguous`, which `create-not-done` rejects without saying so.
9. Fix item 6 in quarry or in lyx?
   In lyx: quarry v0.2.1 documents in `statusForMatches` that it will not evaluate build constraints, and it is a separate module.
   planglyph normalizes an ambiguous answer of pairwise disjoint build variants to multipart at its one resolve chokepoint, so every check agrees.
10. What does "flag at Plan time" mean once variants are accepted?
    The plan gate and DoneChecks read the same normalized answer, so the plan gate accepts at Plan time exactly what record-batch accepts, and a real collision is flagged at Plan time with a detail saying why it is not a variant set.
11. How does this fit `fork-wait-noise`?
    That entry owns how forks run gates; this task owns Master's wait and the records. D1 holds Master's turn end whether or not forks change.

## Rejected alternatives

- Expire or reap a gate shell at Master's turn end: lyx has no pid for a provider shell, and expiry was removed at base on purpose.
- Await every background shell in Master's run: a stray non-gate shell would hold the run to its timeout for no gate result.
- Prefix match on the whole label with more prefixes (`cd `): matches far too much and still misses other wrappers.
- Demote a `done` run to `stuck` when a gate shell was outstanding: after D1 that case is a timeout or died outcome, which already returns an error.
- A webster verb that spawns the verify fixer: forks spawn only through the Agent tool in Master's own session, so no CLI verb can do it.
- A Go-side block on Master writes at verify-failure time: the run-exit audit already catches parent writes; the root cause was the text Master was sent.
- Second recovery from a clean tree with the first attempt's diff as reference: needs a destructive reset outside the recovery's reach and throws away good work.
- Attribute any uncommitted file to the run when a recovery ran: would let the strand commit an operator's unrelated edits.
- Make quarry honour build constraints: separate module, documented refusal, and the answer would depend on an unstated GOOS.
- Accept every ambiguous Create in DoneChecks only: Edit, Delete and the plan gate would still disagree, which is #507's failure.
- Let a Create handle reuse a name its card deletes (#507 option one): unnecessary once per-GOOS Edit and Delete work by glyph.
- Evaluate constraints by running `go list` per GOOS: slow, needs the toolchain at gate time, and still leaves free tags unenumerated.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Hold or stop a gate shell at Master's turn end? Auto-pick: hold, matched per command segment, bounded by `master_timeout_min` (D1). Resolved.
- Q2 Wording when a gate shell still ends with the run? Auto-pick: "unfinished, result never read, the run did not pass it" (D2). Resolved.
- Q3 Rule at the read site or a verb for the verify fixer? Auto-pick: rule at the read site, through a per-entry re-prompt clause (D3). Resolved.
- Q4 Attribution by batch or clean-tree restart for a second recovery? Auto-pick: attribution, recording recovery sessions per batch (D4). Resolved.
- Q5 Unreadable recovery transcript: skip or refuse? Auto-pick: refuse with a way forward, as for a Master session, so no block is rendered misattributed (D4). Resolved.
- Q6 Exit code for terminal non-done digests? Auto-pick: non-zero, every digest field kept (D5). Resolved.
- Q7 Where to state the allowed gates? Auto-pick: shared implementer body, with the recovery prefix pointing at it (D6). Resolved.
- Q8 Does the Create check fail on unexported or test-file names? Auto-pick: no evidence it does; pin with acceptance rows and fix the message (D7). Resolved.
- Q9 quarry or lyx for build variants? Auto-pick: planglyph normalization to multipart, fail closed (D8). Resolved.
- Q10 Scope of #507's create-reuse option and #511's dead-batch goto? Auto-pick: out of scope, as the board's expected outcomes do not ask for them. Resolved.
- Q11 Does a fork's leftover shell produce a Master turn on completion? Open: carried as an open risk with a mitigation for Plan.
