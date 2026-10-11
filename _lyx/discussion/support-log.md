## Interview

The session ran autonomously.
The chair explored the code at bench base `800a0e892`, then sent two question batches to `ly:twb-seat-lead-1:discussion-advisor-1` and settled each question after its answer.

### Exploration (chair)

- Item 1: at base, `background_shell_wait_min` expires nothing. A gated turn end with its outputs present finishes Done at once while only unawaited shells are outstanding (`countsAtOnce`), and Master awaits only `lyx webster recover-batch`.
- Item 2: shuttle's generic `gateRepromptText` says "read it, fix every finding", which is what Master reads at a verify failure.
- Item 3: `loadRunWrites` audits only Master sessions. `recover-batch` returns `output.Ok` for a terminal stuck or dead digest.
- Item 5: probes with `lyx quarry resolve` showed unexported (`cmd/testtiming#rusageStats`) and `_test.go` members resolve `found`, and per-GOOS members (`internal/proc#StartTime`) resolve `ambiguous`. `rusageStats` was first written across `rusage_linux.go`/`rusage_other.go`.

### Advisor notes (`ly:twb-seat-lead-1:discussion-advisor-1`)

The advisor wrote `_lyx/discussion/advisor-1.md` and confirmed the chair's facts independently.
Its notes add that `renderVerifyGateFindings` says "Commit or remove these paths", that `AuditForks` errors on a missing transcript, and that quarry by design does not evaluate build constraints.

### Batch 1 — gate shells, Master self-fix, recovery

- Q1 (gate shell): the chair proposed a `lyx gate test` awaited prefix, segment-start matching and a gate-shell warning clause.
  Advisor: agreed. `lyx gate` has one verb and bounds its own wait, and Master is the only caller of `AwaitedShellPrefixes`. It raised leading `VAR=val` words and asked that the splitter be stated as not a parser. The clause belongs in `expiredShellWarning`, which feeds the warning, the friction note and the summary. → D1, D2, D3.
- Q2 (self-fix): the chair proposed a per-entry re-prompt.
  Advisor: agreed. There is one call site and the failing entry is in scope. Keep the findings file, because the fixer reads it. Keep the `Gate findings recorded at …` anchor. State the stencil-ownership reasoning. → D4.
- Q3 (attribution): the chair proposed recording recovery session ids and auditing them.
  Advisor: agreed, but adding the writes to `writes.Master` would let recovery writes pass as Master's in `contractevidence`, so use a third slice. Reset counting them as its own is correct. Decide what happens on a missing transcript. → D5.
- Q3b (exit code): the chair proposed `ErrFields` on stuck or dead.
  Advisor: agreed, and asked to keep `dead_reason` too, and to check the sandbox suite. → D6.

### Batch 2 — recovery gates, Create checks, build tags

- Q4: the chair proposed two sentences in the shared body.
  Advisor: agreed. Nothing at base refuses `lyx gate test --tags tmux`, so use "permitted, but never needed" wording. → D7.
- Q5: the chair proposed answering "it resolves them" and adding one detail renderer.
  Advisor: agreed. Keep `; ` out of the detail, and update the places that quote the wording. → D8.
- Q6: the chair proposed one predicate read as multipart by each reader, and no new Plan rule.
  Advisor: agreed on the predicate, with two changes. Normalize once in `resolveTargets` (copying candidates into `Symbols`) so no reader is missed, and model GOOS/GOARCH one-hot. The delta reports a per-GOOS Create as two `Created` occurrences under one ID, so `BindHandles` binds it. `handle-collision` already forces the one-card shape. Decide on the `//go:build ignore` case. → D9, D10.
  The chair kept `planglyph.Resolve` (the `lyx quarry resolve` path) verbatim, against the advisor's note that normalizing would also fix the CLI. The reason is the verb's documented verbatim contract.

## Rejected alternatives

- Item 1: lyx stops the gate shell and records it discarded. Shuttle has no pid for a shell, so this needs a new seam capability, and a discarded gate result still leaves a misleading done.
- Item 1: hold on every background shell. A stray long-running shell would pin every run to its Timeout.
- Item 1: tell the Master stencil to use bare `lyx gate …` calls instead of widening the match. Stencil discipline is weaker than matching, and forks' shells reach Master's wait too.
- Item 2: a webster verb that spawns the fixer. A fork is an Agent-tool call, and a verb could only start a cold strand, which is a different design.
- Item 2: restate the rule at the head of the findings file only. Master reads the re-prompt first, and the file's audience is the fixer fork.
- Item 3: put recovery writes into `RunWrites.Master`. `contractevidence` reads that slice as Master's own last write.
- Item 3: start the second recovery from a clean tree with the first diff as reference. This discards work through a destructive step under the fabric destruction chokepoint, and the diff is the same evidence.
- Item 3: refuse the spawn when a recovery transcript is unreadable. That turns a missing log into a blocked batch, while degrading to "not written" keeps today's safe side.
- Item 3b: keep exit 0 and add `batch_failed` to the ok envelope. The exit code itself is the fault.
- Item 4: put the sentences in the recovery prefix only. Forks read the same fabric ban.
- Item 4: "refused here" wording for the tmux tier. Nothing refuses it at base.
- Item 5: a new "resolver limitation" status. No such limitation exists at base.
- Item 6: change quarry to honour build constraints. It is an external module, and that contradicts its stated design.
- Item 6: patch each reader of `ambiguous` separately. Readers could be missed, so normalize once instead.
- Item 6: brute-force free booleans over tags. That reads `linux` and `windows` as co-true.
- Item 6: normalize `lyx quarry resolve` too. That breaks the verb's verbatim contract.
- Item 6: a new Plan-time build-constraint rule. A Create cannot be judged from quarry at Plan time, and `handle-collision` already forces the legal shape.
- #507: let a Create reuse a name the same card deletes. The detour that needed it is no longer forced.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Pick | Basis |
|---|---|---|---|---|
| 1 | Hold or discard a running gate shell at Master's turn end | resolved | hold via `lyx gate test` awaited prefix | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 2 | Prefix vs segment match for awaited labels, and `VAR=val` | resolved | segment-start on five separators, skip leading assignments, splitter not parser | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 3 | Where the gate-shell "never read" clause lives | resolved | `expiredShellWarning` | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 4 | Re-prompt vs findings file vs verb for Master self-fix | resolved | per-entry re-prompt, findings file unchanged | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 5 | Is the Go-authored re-prompt a stencil under `PATTERN-stencil-ownership` | resolved | no, a mid-run Go line like `gateRepromptText` | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 6 | Attribution by batch vs clean tree | resolved | record recovery session ids, third `Recoveries` slice | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 7 | Unreadable recovery transcript | resolved | Warn, contributes no paths, spawn proceeds | auto-pick (advisor raised it, offered no pick) |
| 8 | recover-batch exit on stuck or dead | resolved | `ErrFields`, non-zero, ladder fields kept, refusal-spec row | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 9 | Where the allowed-gates sentences go, and tmux wording | resolved | shared body, "permitted, never needed" | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 10 | Does the Create check resolve unexported or test-file members | resolved | yes, no resolver fix, the cause is item 6 | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) plus chair probes |
| 11 | Done-check detail content | resolved | one renderer naming key, lookup and answer, no `; ` | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 12 | Where the constraint-split predicate applies | resolved | normalize inside `resolveTargets` for planglyph readers | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 13 | `lyx quarry resolve` normalized too | resolved | no, stays verbatim | auto-pick (against the advisor's side note, for the verb's documented contract) |
| 14 | `//go:build ignore` candidate | resolved | not split, fails closed | auto-pick (advisor raised it, offered no pick) |
| 15 | New Plan-time rule for build-tagged Creates | resolved | none; plan stencil line, `handle-collision` already enforces the shape | advisor-informed (`ly:twb-seat-lead-1:discussion-advisor-1`) |
| 16 | #507 "Create reusing a deleted name" and #511 "goto revives a dead batch" | resolved | out of scope | auto-pick (not in the entry's expectations, and the #507 detour is no longer forced) |
| 17 | Rename of a constraint-split member (`renameDeclSource` found-only) | resolved | unchanged, out of scope | auto-pick |
