---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/16-status-commit-holds-archive.md
    summary: card 16 releases the status-commit hold once the highest round's working-tree record carries a class, but card 5 step 4 writes the class before the round commit, so a Failed or resume persist in that window still splits the round and PR-Rework then finishes without ever committing the new plan
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/00-overview.md
    summary: the verify section's go test -tags integration package list omits ./internal/planglyph/..., which card 7 edits and which carries //go:build integration test files
---

### [BLOCKING] Card 16's hold ends before the round commit lands

**Location:** `_lyx/plan/16-status-commit-holds-archive.md` (the `ReworkArchiveInFlight` predicate and its tests), against `_lyx/plan/05-rework-archive-generation.md` step 4.

**Issue:**
Card 5 step 4 orders the post-session work as classify, copy `coverage.md`, rewrite `record.json` with `class`, then `Commit`, then clear the rejection.
Card 16's `ReworkArchiveInFlight` reports false as soon as the highest round's working-tree `record.json` carries a class, so `statusCommitPathspec` stops holding back from the moment the class is written, not from the moment the round commit lands.
Shed persists, and so calls the status seam, on a Failed exit (`internal/shedengine/run.go:302` and `:381` both go through `persist`/`persistTransient`, which call `CommitStatus` unconditionally), and on the `running` write a resume makes after a crash.
A `Commit` seam error after the class write, or a crash between the class write and the round commit, therefore produces a status commit whose pathspec includes `_lyx/reviews` and `_lyx/loom` (`internal/loomcli/wiring.go:64-77`).
That commit lands the classed `record.json`, every `prior-generation/` copy and the review run directories' deletions, while the `_lyx/plan/` deletions, the new plan and the webster deletions stay uncommitted.
On the next step, `prRework.Call` finds the pending rejection's identity in `committedRejections` (a classed record at HEAD) and takes `finish(true)` (`internal/loomshed/prrework.go:118-120`): it clears the rejection and returns Done with no round commit.
The result breaks D14 ("the deletions and the archive land in one commit") and the acceptance criterion that a crash after the session and before the commit converges to one commit.
Worse, card 9's predicate parses the plan at HEAD, which still holds the retired generation, and the overview's `generation-round-rule` claim that the working tree and HEAD agree once Plan-Review runs no longer holds.
Card 16's own tests pin the hole: "a classed record → false" and the table case "a classed highest round ... keeps today's pathspec" both assert the release happens in the window.
Card 5's crash-resume tests run over fake seams in `prrework_test.go`, which model no status commit, so nothing in the plan exercises the window.

**Fix:**
Hold the sweep for the whole time a rejection is pending, not only while the highest round is unclassed.
`lyx loom reject` writes the pending rejection, which only PR-Rework clears, and only after its round commit has landed, so "a rejection is pending" covers the archive, the session, the class write and the commit, with no round reading at all.
Card 16 drops `ReworkArchiveInFlight` and its `generation.go`/`generation_test.go` targets.
`statusCommitPathspec` returns only the status file and the drive reports while `loomengine.LoomRejectionPath` exists, and while a stat of it fails for any reason other than not-exist.
Its tests cover a pending rejection with a classed but uncommitted round (the state a `Commit` failure after the class write leaves), and the integration test asserts HEAD gains no `round-<N>/` file and keeps the review run directory until the rejection is cleared.
The bound becomes: the hold needs a pending rejection, which only `lyx loom reject` writes and only PR-Rework clears after its round commit, and it defers the sweep of `_lyx/loom/` and `_lyx/reviews/` without dropping anything from it.
The overview's `committed-round-carries-a-class` and `one-commit-per-round` decisions and card 5's references to card 16 are reworded to the pending-rejection hold.

### [MEDIUM] Verify does not run planglyph's integration tests

**Location:** `_lyx/plan/00-overview.md`, `## verify:`.

**Issue:**
Card 7 edits `internal/planglyph/planglyph.go` and `planglyph_test.go`, and `internal/planglyph` carries `//go:build integration` test files.
The `go test -tags integration` line lists every other targeted package with tagged tests (`websterengine`, `loomcli`, `landingshed`) but omits `./internal/planglyph/...`, so those hermetic tests never run under verify.
Every other targeted package with no tagged files is listed anyway, which runs harmlessly.

**Fix:**
Add `./internal/planglyph/...` to the `go test -tags integration` line, and to the `go vet -tags smoke` line for consistency.
