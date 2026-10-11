# Support log — dbench-omsm

## Interview

Autonomous mode: no operator answers.
The chair explored the board entry, the closed issues it merges (#505, #509, #517, #522, #511, #500, #507), the related entries `gate-test-hygiene` and `proc-child-lifetime` (no overlap), and the code at module level, then put two batches to advisor `ly:dbench-omsm:discussion-advisor-1`.

### Exploration findings

- Commit 5a8be0274 already stopped shells expiring and records ended shells; a turn end with Master's output files still ends the run, and strand removal kills the shell.
- Shuttle's shared gate re-prompt says "read it, fix every finding, and end your turn", contradicting Master's stencil, which forbids fixing and asks for one fixer fork.
- Recovery write evidence reads Master's session plus one session id per batch, overwritten by each recovery.
- quarry is external; its walk puts same-package test files in the package unit, and its own tests pin ambiguous for build-tag duplicates.
- planglyph's done check reports every non-found answer as "still does not resolve".
- `cmd/testtiming` has `rusage_linux.go` and `rusage_other.go`; `tmuxkit`'s `tmuxAllowed` was restructured to var plus init after #500.

### Batch 1 — items 1 to 4

- Q1 gate shells: chair recommended `lyx gate` as an awaited shell prefix.
  Advisor-1 agreed: `countsAtOnce` counts a gated turn end at once only when every outstanding shell is unawaited, so a prefix holds it, bounded like recover-batch;
  it flagged that the match is a label prefix (confirm claudeengine's label) and recommended a stencil line "never run `lyx gate`" as well, citing `PATTERN-gate-slot`;
  it rejected demoting to stuck as a verdict outside `PATTERN-completion-signal`.
  Settled as D1.
- Q2 Master self-fix: chair recommended a per-run re-prompt text.
  Advisor-1 agreed: the later Go message wins over the stencil;
  keep it single-line (`validateSendText`), point at the stencil section per `PATTERN-producer-pointer-rule`, read any stencil text through `stencilstore`, extend the existing re-prompt test.
  Settled as D2.
- Q3 recovery attribution: chair recommended attribution by batch and a non-zero exit on a failed digest.
  Advisor-1 agreed: append session ids instead of reusing the scalar, check `state.json` compatibility, forks need nothing since `AuditForks` covers them;
  non-zero exit is safe as the Master rungs key on digest content, provided refusals stay distinguishable.
  Settled as D3 and D4.
- Q4 recovery gates: chair recommended naming the gates in the recovery prefix only.
  Advisor-1 agreed: the ambiguity comes from the prefix's own ban;
  render the names from the card and batch gate source and check `PATTERN-no-denied-recovery`.
  Settled as D5.

### Batch 2 — items 5 and 6, scope

- Q5 Create check: chair concluded #511 was #500/#507 behind a misleading message, and recommended naming the lookup and answer in the Detail.
  Advisor-1 agreed, asked for "likely" rather than "is", reuse of `candidateList` (prints `Symbol.File`), the spec row update and a reaching test.
  Settled as D6.
- Q6 disjoint ambiguity: chair recommended a truth table over the mentioned tags.
  Advisor-1 disagreed on mechanism: independent booleans let `linux` and `windows` hold together, so per-GOOS files never read disjoint;
  it proposed enumerating contexts over mentioned GOOS/GOARCH names plus "other", cgo, and subsets of mentioned custom tags, judged by `build.Context.MatchFile`, capped at 8 custom tags and failing closed;
  candidate `File` exists but may be empty (fail closed);
  route every status reader through one helper so the status-completeness tests catch a missed site.
  Chair adopted it as D7.
- Q7 delete-then-create: chair recommended waiving create-already-exists when every declaration is deleted earlier.
  Advisor-1 agreed, with plan execution order from the existing delete-ordering logic and a Rename's old side counted as a delete.
  Settled as D8.
- Q8 scope: chair recommended leaving the dead-batch halt after `lyx loom goto` out.
  Advisor-1 agreed, noting it may be a pre-existing `PATTERN-refusal-way-forward` gap.
  Recorded under open risks.

## Rejected alternatives

- Item 1, demote the run to stuck when a gate shell is outstanding: fails a run whose work is done and adds a verdict outside the output-files-first rule.
- Item 1, stencil rule alone: an agent can still start a gate shell; the prefix is needed as the backstop.
- Item 2, a verb that spawns the verify fixer: forks are the Agent tool inside Master's session; no `lyx` verb can start one.
- Item 2, rewording only Master's stencil: the stencil already says the right thing; the contradicting Go message is what Master acts on.
- Item 3, start the second recovery from a clean tree with the first attempt's diff as reference: needs a destructive reset of uncommitted work and discards a usable starting point.
- Item 4, the gate rule in the shared implementer body: forks carry no ban that the gates could be confused with.
- Items 5 and 6, change quarry to honour build constraints: quarry is an external module needing its own release and a `go.mod` bump, out of this worktree.
- Item 6, a truth table over independent tag booleans: cannot express that exactly one GOOS holds, so per-GOOS files never read disjoint.
- Item 6, a fixed GOOS by known-tag matrix: needs a pinned platform list that drifts.
- Item 5, a resolver change for unexported or test-file members: quarry already resolves them.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 gate shell at turn end — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): awaited `lyx gate` prefix plus stencil line (D1).
- Q2 Master self-fix — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): per-run webster re-prompt text (D2).
- Q3a recovery attribution — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): append-only per-batch session ids (D3).
- Q3b recover-batch exit status — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): non-zero on stuck or dead digest (D4).
- Q4 recovery gates — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): recovery prefix only (D5).
- Q5 Create check on unexported targets — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): no resolver change, lookup named in the Detail (D6).
- Q6 disjoint build constraints — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): `MatchFile` over enumerated contexts, one status helper (D7).
- Q7 delete-then-create name reuse — resolved, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`): waiver when every declaration is deleted earlier or in the same card (D8).
- Q8 dead-batch halt after goto — resolved as out of scope, auto-pick, advisor-informed (`ly:dbench-omsm:discussion-advisor-1`); recorded as an open risk.
- Open for Plan: the exact label claudeengine records for a backgrounded Bash command, which D1's prefix match depends on.
- Open for Plan: whether `candidateList`'s `File` is repo-relative or unit-relative, which D7's `MatchFile` call depends on.
