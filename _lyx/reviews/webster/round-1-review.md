---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: internal/loomcli/wiring.go:318
    summary: discussionCommitPathspec always names the parent-review directory, so CommitDiscussion fails whenever that directory holds no file (a seed with no parent, or the entry turned off with attempts 0)
  - id: F2
    severity: BLOCKING
    location: internal/loomcli/cli.go:173
    summary: with cobra.EnableTraverseRunHooks set in cmd/lyx, the loom parent's arm still runs for every review verb, reads the slug as a run-id, full-wires notify and delivered, and prints a second JSON envelope on refusal
  - id: F3
    severity: MEDIUM
    location: internal/loomcli/wiring.go:314
    summary: the two new functions sit between wire's doc comment and wire, so wire loses its doc and discussionCommitPathspec gets wire's, and the file fails gofmt at the CommitDiscussion call
  - id: F4
    severity: LOW
    location: internal/parentreview/store.go:405
    summary: RecordVerdict checks the request state under one lock acquisition and writes the verdict under another, and copies review.md outside the lock, so a verdict can land on a request the gate just expired and a refused verdict can overwrite review.md
  - id: F5
    severity: LOW
    location: internal/shedverbs/spec.go:76
    summary: several new or changed Go comments are fixed-column wrapped mid-phrase, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F6
    severity: NIT
    location: internal/shedrecipe/entries_discussionwrite_test.go:226
    summary: the doc comment on hasRound names it roundCount
---

Review range: `_lyx/shed/parent-review/status.json` gives `product.parent` = `main`; no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` with merge base `218ba2a68`, the fourteen card commits.

Evidence gathered: `go build ./...`, `go vet` over the touched packages, `go test` over every touched package, `go test -tags integration -run ParentReview ./internal/loomcli/` and `go vet -tags smoke ./internal/loomcli/` all pass.
`gofmt -l` over the diff's Go files reports `internal/loomcli/wiring.go`.
A dev build of `lyx` run in this worktree reproduced F2, and a scratch git repository reproduced F1's git refusal.

### [BLOCKING] The discussion commit fails when the parent-review directory holds no file

**Location:** internal/loomcli/wiring.go:318 (`discussionCommitPathspec`), used by `CommitDiscussion` at wiring.go:536.

**Issue:** `discussionCommitPathspec` always returns `LoomParentReviewDirRel()` beside the discussion directory.
`gitrepo.StageAndCommit` runs `git add -- <paths>` and then `git commit -m <msg> -- <paths>`.
`git add` refuses a path that does not exist (`fatal: pathspec ... did not match any files`), and `git commit` refuses a path that exists but holds no file git knows (`error: pathspec ... did not match any file(s) known to git`).
Both reproduced in a scratch repository.
`statusCommitPathspec`'s own doc comment (wiring.go:61-63) records this exact hazard and guards every directory with `holdsFile`; the new pathspec does not.
Two ordinary cases hit it:
a seed with no `Parent`, where the fresh-spawn `BeginRound` creates an empty `round-<N>/` and the gate passes without writing a file, so the commit fails on `git commit`;
and `attempts: 0` on the entry, the documented off switch, where `prepareFreshSpawn` is nil, the directory never exists, and `git add` fails.
An attach after a driver restart, or a run from before this change, also reaches the commit with no round directory.
Card 12's ImpactSummary says a seed with no parent passes the new entry; instead it fails Discussion-Write's commit.
`TestDiscussionCommitPathspec_IncludesParentReviewDir` pins the unconditional list, and the integration test only exercises a seed with a parent.

**Fix:** Take the location and add `LoomParentReviewDirRel()` only when `holdsFile(LoomParentReviewDir(location))`, as `statusCommitPathspec` does for its directories.
The round files are never deleted, so the tracked-but-removed case `reworkCommitPathspec` also handles cannot arise.
Pin both the with-file and without-file shapes in the test.

### [BLOCKING] The loom parent's arm still runs for the review verbs

**Location:** internal/loomcli/cli.go:173 (`resolvePersistentPreRun`); the claim it contradicts is review.go:4 and card 7.

**Issue:** Card 7 requires that the loom parent's `arm` never runs for the `review` group and never reads a slug as a run-id, and review.go's file comment says the group's own `PersistentPreRunE` achieves it.
It does not: `cmd/lyx/main.go:100` sets `cobra.EnableTraverseRunHooks = true`, so cobra runs every ancestor's `PersistentPreRunE`, root to leaf.
For `lyx loom review notify <slug>` the loom parent's `resolvePersistentPreRun` runs with `cmd.Name() == "notify"`, `resolveRunID` takes the slug as the run-id, and `verbUsesLightweightWiring("notify")` is false, so the full `wire()` runs: module configs, the reed engine and the seed read at run-id `<slug>`.
Reproduced with a dev build in this worktree: `lyx loom review notify nosuchslug` printed two envelopes, first the arm's `config file .../loom.yaml: missing keys: parent_review_wait_min` refusal, then the review pre-run's unknown-slug refusal; `lyx loom review delivered` refused on the loom config alone.
So a reviewer's verb fails on any unrelated full-wire failure, and a refusal emits two JSON documents on stdout.
The package's tests run without traverse hooks, which is why none caught it.

**Fix:** Make `resolvePersistentPreRun` return early for the `review` group and every command under it, the same way it skips the bare `loom` group.
Add an untagged test that calls the loom parent's `PersistentPreRunE` with each review command and asserts it writes nothing and does not abort.

### [MEDIUM] wire loses its doc comment, and wiring.go is not gofmt-clean

**Location:** internal/loomcli/wiring.go:314-347 and wiring.go:536.

**Issue:** `discussionCommitPathspec` and `newParentReviewConfig` were inserted between `wire`'s three-line doc comment and `func (c *loomCLI) wire`.
`wire` now has no doc comment, and `discussionCommitPathspec`'s doc comment opens with "wire builds the whole engine stack onto c ...".
Separately, `discussionCommitPathspec(),fmt.Sprintf(` lacks the space gofmt inserts, so `gofmt -l` flags the file.

**Fix:** Move the two new functions above `wire`'s doc comment so the comment sits on `wire` again, and gofmt the file.

### [LOW] Verdict writes are check-then-act across two lock acquisitions

**Location:** internal/parentreview/store.go:405 (`RecordVerdict`), with `MarkExpired` at store.go:327 and the gate's expiry arm at gate.go:116.

**Issue:** Each single-file write is atomic under the round's lock, but a verdict is decided over several acquisitions.
`openUnsettled` reads `request.json` under one acquisition, and the verdict is written under a later one whose mutate checks only `verdict.json`.
If the gate's `MarkExpired` lands between them, the verdict lands on an expired request: the verb reports success, and the gate, which checks `State != open` first, passes without ever reading it.
The mirror race also exists: the gate decides to expire from a read taken before a verdict that lands before its `MarkExpired`, so a reject the verb accepted is dropped.
`review.md` is also written before, and outside, the verdict's own lock, so a verdict that loses the race and is refused still overwrites the winner's `review.md`.
The window is narrow (a submit within milliseconds of the wait bound, or two concurrent submits), which is why this is LOW.

**Fix:** Under the verdict's own lock, re-read `request.json` and refuse `ErrExpired` when it is no longer open, then write `review.md` inside that same critical section, after the found check.
Under `MarkExpired`'s lock, refuse when `verdict.json` exists; the gate and `final` then re-run their own evaluation so the verdict is read.
The raw reads go through the standard library, since `state.ReadJSON` on the held lock path would deadlock.

### [LOW] Fixed-column wrapped comments

**Location:** internal/shedverbs/spec.go:76-80 (`Hooks.Waiting`), internal/shedverbs/status.go:97-98 (`waitingNote`) and 117-119 (`statusWatchLine`), internal/loomengine/config.go:6-7 (changed lines of the file comment) and 437-439, internal/loomengine/parentreview.go:1-2, contracts/stencils/stencils.go:63-64, contracts/stencils/discussiontemplate_test.go:65-66, internal/loomcli/cli_test.go:83-84, internal/shedverbs/status_test.go:552-553 and 616-617.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks with no column limit for new and changed Go comments.
These comments break mid-phrase at roughly column 100 ("adds a / waiting key", "the live Discussion-Write / session", "It returns the empty note when / the hook is nil").
Most of the diff's other comments follow the rule.

**Fix:** Rewrite each as one sentence per line, breaking further only at an independent-clause boundary.

### [NIT] Doc comment names the wrong function

**Location:** internal/shedrecipe/entries_discussionwrite_test.go:226.

**Issue:** The comment reads "roundCount reports whether the parent-review store holds a round." over `func hasRound`.

**Fix:** Name it `hasRound`.

## Focus answers (no finding unless noted)

- Pending state at a turn boundary (card 1): `gateAtBoundary` is set only on a gated Done arrival and cleared after every successful gate send, the re-prompt and the pending text alike, and the poll-tick branch runs only on `outcome == ""` with `gatePending && gateAtBoundary`.
  Liveness and the deadline sit after the events branch in the same loop iteration, so they keep firing during a pending wait.
  The pending memo is cleared after every pending result, so a poll tick and a later `finalize` read the closures afresh, and `finalize` uses `Final`.
  Both run-time refusals (pending, and `Final` on a non-`PassOnCap` entry) are in `evaluateGate`.
  Not a defect.
- Round lifecycle (cards 4-5): the verbs, the closure and status read the latest round only, `BeginRound` supersedes an unverdicted open round, an attach skips `prepareFreshSpawn`, and a reject is consumed once, re-prompts once, and the next arrival's entry is `let_through` from the in-memory budget, or `passed` from the consumed flag after a restart.
  Atomicity is F4.
- Send path: the closure records each gate prompt on disk before the loop sends, so a failed send still spends one of the three, the throttle is a minute, a failed send Warns with `SendFailedWayForward` naming `lyx loom review notify <slug>`, and the bound ends in `passed` with a Warn rather than a stall.
  Matches cards 1 and 5.
- Review verbs (card 7): every refusal row in the card and in refusal-spec.md maps to a `way forward:` message through `internal/output`, every command has a `Short`, and the exact-children test covers the subtree.
  The arm claim fails in production; that is F2.
- Waiting hook (card 8): with `Hooks.Waiting` nil, `waitingNote` returns empty, `core` gains no key, `RenderStatusHuman` takes the original branch and `statusWatchLine` appends nothing, so output is byte-identical.
  `RenderStatusHuman`'s only production caller is `statusCmd`.
- Recipe (cards 6, 12): only `parent-review` returns a non-nil `Final`, `resolveGateEntry` refuses it without `pass_on_cap: true` at construction, and a seed with no parent passes the closure without opening a request.
  The commit that follows breaks; that is F1.
- Provider wording: no `SendMessage` or Claude wording in `internal/shuttleengine`, `internal/reedengine` or `internal/parentreview`; the instruction lives only in `loom-template-parent-review-delivery.md`.
  `internal/parentreview/seam_enforcement_test.go` pins the import allowlist only, not provider words; no card asks it to, so not a defect.
- Per-card mechanical check: the cards carry only Create and Edit groups, so no Rename AST check or `assert-no-callers` applies.
  The Edit groups' blast radius matches each ImpactSummary: existing `GateEntry` literals compile unchanged, `RenderStatusHuman` has one caller, the four existing gate names resolve as before, and batten's change is confined to the running arm.
  No Create group duplicates an existing equivalent; `resolveReviewTarget` resembles battencli's `taskWorktreeLocation`, but card 7 prescribes its own refusal texts and seams.
