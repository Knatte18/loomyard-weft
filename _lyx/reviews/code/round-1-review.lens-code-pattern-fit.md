```yaml
lens: code-pattern-fit
outcome: reported
```

Lens `code-pattern-fit`: the diff checked against `PATTERN.md`, `CLAUDE.md` and the commit trailers.

**Rules checked**
- `PATTERN-shed-stop-chain`: **broken**, see F1.
- `PATTERN-shed-driver`: held. `decideDrive` reads only `facts.ArbiterFirst`, and the Webster row name enters through `ArbiterSeam.JudgesFirst` in `drivewiring.go:155`, so the table stays recipe-blind.
- `PATTERN-sole-parsers`: held. Only `internal/discussionparser` reads the record, through `AddedDecisionTexts`, which shares `addedDecisionSpans` with `IdentityText`.
- `PATTERN-stencil-ownership`: held. Go renders `parent_decisions` into the prompt at call time, and the stencil is not embedded.
- `PATTERN-parent-directive`: held. The stencil's message rules change only the failure-ladder text, not the directive rendering.
- `PATTERN-refusal-way-forward`: **partly broken**, see F2. The new refusal in `Run` names a way forward but no test reaches it.
- `PATTERN-friction-capture`: held for this diff. The neighbouring `Run` errors, such as the `DeclaredPatternChanges` failure just above, write no friction note either, so the new refusal matches the existing code. I could not tell whether the pattern requires one.
- `PATTERN-test-speed`: held. The new behaviours are rows in `TestDecideDrive`, `TestGoDriver_Settle` and the stencil marker tests. `TestAddedDecisionTexts` is a new test, but no existing test covers a file-reading entry read.
- `PATTERN-tolerant-records`, `PATTERN-config-layers` and `PATTERN-durable-vs-ephemeral-state`: not touched.
- Declared PATTERN changes: the decision record declares none, so there is nothing to land. F1 is the consequence.
- Commit messages: neither commit carries a `Pattern-Departure:` trailer. Co-author lines are present.
- `CLAUDE.md` docs rule: held. `internal/sheddriver/doc.go` and `internal/websterengine/doc.go` are updated. `docs/overview.md` needs no change because the module table and execution stack are unchanged. The orch stencil already says the driver hands a stop to the arbiter first (`orch-template-role.md:54`), so no edit is needed.
- `CLAUDE.md` markdown rule: held. The new stencil prose is one sentence per line.
- `CLAUDE.md` board-entry rule: held. No step edits the task's own entry.

**Findings**

- **F1, BLOCKING, class `consistency`**: `internal/sheddriver/driverules.go:234` and `internal/loomcli/drivewiring.go:155`. The `PATTERN-shed-stop-chain` entry reads "rule table, tackler, arbiter, then parent, for every recipe" (`PATTERN.md:34`). The new row sends a blocked Webster stop to the arbiter and skips the tackler, so the chain is no longer the same for every recipe. The diff leaves `PATTERN.md` unchanged, and neither commit has a `Pattern-Departure:` trailer. `CLAUDE.md` requires a new cross-cutting invariant to land in `PATTERN.md` in the same commit as the code. The decision record's constraint ("stays the order for every stop the new row does not mark") admits the exception but does not make it a declared entry change. Fix: either reword the entry so a row the arbiter seam marks as judged first skips the tackler, or add a `Pattern-Departure: PATTERN-shed-stop-chain` trailer with the reason.
- **F2, MEDIUM, class `scope`**: `internal/websterengine/runlevel.go:740-743`. The new refusal ("read the parent decisions … way forward: transient, …") is reached by a non-`ErrNotExist` read error from `AddedDecisionTexts`. `PATTERN-refusal-way-forward` requires a test to reach each webster refusal's way forward. `TestAddedDecisionTexts` covers a missing record and an empty path but not a read error, and no test drives `Run` through this branch. Fix: add a row to the existing `Run` refusal test, for example a `DecisionRecordPath` that is a directory, asserting the message names the re-entry step.

**Tool calls (1)**
- Read `internal/websterengine/runlevel.go` lines 715-742 and grepped for `shed-stop-chain` references. My context did not show the code neighbouring the new refusal, so I could not judge the friction rule, and I wanted to know whether any other file cites the entry.
