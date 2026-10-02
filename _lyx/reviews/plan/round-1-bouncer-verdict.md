---
verdict: BLOCKING
rationale: "Card 16's hold keys on a working-tree class, but card 5 writes the class before the round commit, so a commit-seam error or a crash in that window lets a status commit land a classed record.json at HEAD, after which committedRejections makes PR-Rework clear the rejection without ever committing the new plan; the verify section also never runs internal/planglyph's integration-tagged tests although card 7 targets that package."
---

# Plan-Review round 1 — judge verdict

The round-1 report contains no `## Focus departures` section, so there is no departure to ratify or reject.

## The report's findings

All four findings are legitimate under the rubric, and none falls under its Do-not-flag list.

- **F1 (status commit splits a round)** is correct.
  `statusCommitPathspec` (`internal/loomcli/wiring.go:64`) adds the reviews root and the loom durable directory whenever each holds a file, and shed persists on every non-Done exit.
  The fixer's card 16 closes the window between the archive step and the class write, but not the window between the class write and the round commit (see B1 below).
- **F2 (`first_card` checked outside `planparser`)** is correct against the decision record's Planparser Sole-Parser constraint.
  The fix holds: card 7 now puts `CheckFirstCard` in `internal/planparser/firstcard.go`, adds `planglyph.ValidateRework`, and documents `rework-first-card` outside the validation-checks list.
- **F3 (three wordings of the round rule)** is correct.
  The fix holds: the overview's `generation-round-rule` carries the class condition and names card 9 as authoritative, card 5 names the `class` field, and cards 12 and 13 state the same rule.
- **F4 (card 10's bound thinner than D7)** is correct.
  The fix holds: card 10's recipe comment now names the agent-written inputs, the re-classification and the seam-error fallback.

## Remaining findings

### B1 [BLOCKING] Card 16's hold ends before the round commit lands

Card 5 step 4 orders the tail of `prRework.Call` as: classify, copy `coverage.md`, rewrite `record.json` with `class`, commit, clear the rejection.
Card 16's `ReworkArchiveInFlight` reads the working tree and reports false as soon as the highest round's `record.json` carries a `class`.
So between the class write and the round commit, the hold is off while the round is still uncommitted.

Two paths persist in that window:

- the `Commit` seam returns an error, PR-Rework returns it, and shed persists Failed;
- the process dies after the rewrite and before the commit, and the resume persists `running` before re-calling the producer.

Either persist runs the status commit, which now stages `_lyx/loom/` whole (the classed `record.json`, `findings.md`, `coverage.md`, every `prior-generation/` copy) and the review run directories' deletions.
On the next call, `committedRejections` (`internal/loomshed/prrework.go:204`) finds a classed record for the pending rejection's identity at HEAD, and `Call` takes the `committed[...]` branch at line 117 straight to `finish(true)`: it clears the rejection and returns Done without running the round commit.
The new plan, the `_lyx/plan/` deletions and the webster durable directory's deletions are never committed by PR-Rework, against D14 ("the deletions and the archive land in one commit") and the crash-resume acceptance criterion for "after the session and before the commit".
Card 9's skip predicate then reads a HEAD overview that is still the retired generation's, so its `first_card` matches no round and Plan-Review runs over a plan HEAD does not hold.

The overview's `committed-round-carries-a-class` and `one-commit-per-round` decisions both rest on "classed at HEAD means the round commit landed"; card 16 as written does not guarantee it.
Card 16's tests cover a classless highest round and a classed one, but not a classed-but-uncommitted round with the rejection still pending.

Fix direction, for the writer to choose: make the hold track the round commit rather than the class, for example by holding back for as long as a rejection is pending (PR-Rework clears it only after the round commit), or by having `ReworkArchiveInFlight` read the highest round's record at HEAD through the committed-file reader.
Add a card 16 test for a classed working-tree record over a classless or absent HEAD record with the rejection pending, and a card 5 crash-resume case where the commit seam fails after the class write.

### B2 [MEDIUM] Verify never runs `internal/planglyph`'s integration-tagged tests

Card 7 targets `internal/planglyph/planglyph.go` and `internal/planglyph/planglyph_test.go`.
That package carries hermetic `//go:build integration` tests (`drift_integration_test.go`, `delta_integration_test.go`, `donecheck_integration_test.go`).
The overview's `## verify:` section runs `go test -tags integration` over a package list that omits `./internal/planglyph/...`, so those tests never run under the plan's verify.
Add the package to the integration line, and to the smoke vet line for consistency.
