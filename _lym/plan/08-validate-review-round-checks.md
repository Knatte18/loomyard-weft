# Card 8 — `lym burler validate-review` runs the round checks in a run directory

**Edit:**
- `internal/burlercli#burlerCLI.validateReviewCmd`
- `internal/burlercli/doc.go`
- `internal/burlerengine/doc.go`

**Create:**
- `internal/burlercli/validate_integration_test.go`
- `plan:internal/burlercli#TestValidateReviewVerb_RoundChecks` -> `func TestValidateReviewVerb_RoundChecks(t *testing.T)`

**Uses:**
- `plan:internal/shedadapters#BuildRoundContext`
- `internal/shedadapters#ParseRoundReviewName`
- `plan:internal/burlerengine#CheckReviewRound`
- `internal/lymcwd#ResolveWithin`
- `internal/loomengine#LoomReviewsDir`
- `internal/loomengine#DiscussionDecisionRecord`
- `internal/burlercli#TestValidateReviewVerb_AgreesWithTheReviewGate`

**Intent:**
D3: the verb a reviewer self-checks with runs the same round checks its gate runs, when the file sits in a run directory.

After the shape check passes, the verb resolves the worktree through `lymcwd.ResolveWithin` from the seam cwd alone, never from the review file's path, since `_lym` is a link out of the worktree (`PATTERN-cwd-resolution`).
It runs the round checks only when all hold: the cwd resolves; the file's base name parses through `ParseRoundReviewName`; and the file's cleaned directory lies under `loomengine.LoomReviewsDir(location)`, compared as cleaned paths without resolving links.
It then calls `BuildRoundContext(dir, round, loomengine.DiscussionDecisionRecord(location), location.AnchorPath())`, sets the context's `WorktreeRoot` to `location.WorktreePath()` when the context is non-nil, and runs `CheckReviewRound` on the file.
A refusal prints the check's error verbatim and exits 1, as a shape refusal does.
A builder error falls back to the shape checks alone, as the review producer's Warn leaves the round full: the verb prints ok with a `round_checks` field naming the builder error, never a refusal the gate would not make.
In every other case the shape checks alone decide, as today; the verb still skips the hub, mode and engine wiring of the other burler verbs.

Bound: the verb rebuilds the context from the decision record as it stands, while the gate's context was built at round start.
An entry added through `lym loom decision add` while the reviewer works is listed by the verb and not by the gate, so a `resolved` quoting only that entry passes the verb and fails the gate, whose re-prompt names the refusal and whose cap lets the review through.
The help text says the verb checks against the record as it stands.

The help text names the round checks it runs on a review in a run directory, the cases that get the shape checks alone, and no longer says the verb needs no git repository.
burlercli's doc.go line on the verb says the same, and burlerengine's doc.go sentence that the verb "needs no hub, mode or git repository" is rewritten to say it resolves the worktree from the seam cwd and runs the round checks on a review in a run directory.

`validate_integration_test.go` is tier1 (a hermetic git task worktree from `hubforge`), and holds `TestValidateReviewVerb_RoundChecks`, one table:
an incremental review in its run directory, seam cwd in the worktree, whose `prior` list misses an id of round N-1 exits 1 with the round-check refusal;
the same review written outside a run directory exits 0;
run from a seam cwd outside any git repository it exits 0;
a run directory outside the resolved worktree's `_lym` run root exits 0;
a run directory whose round N-1 snapshot record path is a directory, so the builder errors, exits 0 with `round_checks` naming the error.
`TestValidateReviewVerb_AgreesWithTheReviewGate` stays as it is, covering the shape checks.

**ImpactSummary:** One CLI verb's body and help, and two doc.go sentences; its shape-only behavior is unchanged outside a run directory, and no other verb reads it.
