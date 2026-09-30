---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/shedengine/transient.go:20
    summary: TransientGitHubAPI doc comment says a rate limit qualifies, contradicting githubclient.IsTransient and card 3, which exclude every rate-limit error
  - id: F2
    severity: MEDIUM
    location: internal/landingshed/publish.go:94
    summary: many new and changed Go comments across the diff are fixed-column wrapped, violating the Comment Line-Break Convention in CONSTRAINTS.md
  - id: F3
    severity: LOW
    location: internal/burlerengine/engine.go:204
    summary: card 4 commit introduced an unformatted, out-of-card whitespace edit (result.Outcome !=shuttleengine.OutcomeDone) that gofmt -l flags
  - id: F4
    severity: LOW
    location: internal/githubclient/transient.go:39
    summary: IsTransient counts every non-cancellation url.Error as transient, including a parse-op error and a TLS certificate failure, which are not transport-level and never clear on a retry
  - id: F5
    severity: LOW
    location: internal/loomcli/driverprompt_test.go:57
    summary: after 5b9d1e72d the skill parks using the records-commit command the launch prompt names, but no test pins that the parking sentence names driverRecordsCommand
  - id: F6
    severity: NIT
    location: internal/loomcli/start.go:328
    summary: the --no-attach help paragraph puts the parked-driver special case before the general rule it qualifies and leaves an overlong line in the hand-wrapped text
  - id: F7
    severity: NIT
    location: internal/loomcli/start.go:109
    summary: runDriverSpawnAndWait doc comment still describes only the spawn decision and omits the park-marker removal and resume branch it now takes
---

Review range: `product.parent` is `main` (from `_lyx/shed/ly-drive-safe-repairs/status.json`); the subject is `git diff $(git merge-base main HEAD)..HEAD` (merge base `676d9ec03`), cards 1–14 plus `5b9d1e72d`.

Substrate: `go build ./...`, `go vet ./...`, `go test ./...`, the plan's integration test set and the smoke-tag vet all pass.
`gofmt -l` over the changed Go files lists `internal/burlerengine/engine.go` alone.

### [MEDIUM] TransientGitHubAPI's doc comment names rate limits as qualifying

**Location:** internal/shedengine/transient.go:20
**Issue:** The comment reads "a network error, a 5xx, or a rate limit".
`githubclient.IsTransient` returns false for `*github.RateLimitError` and `*github.AbuseRateLimitError` on purpose, and card 3 says so explicitly ("an immediate retry hits the same limit").
This is the one declaration site of the closed class set (Transient Stop Invariant), so a reader deciding what the driver re-steps on is told the opposite of what the code does.
**Fix:** State that a rate-limit response never qualifies, beside the 4xx exclusion.

### [MEDIUM] New and changed comments are fixed-column wrapped

**Location:** internal/landingshed/publish.go:94 (representative; the pattern spans the diff)
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line for every new or changed Go comment, doc and inline, and names fixed-column wrapping as legacy, not a convention to match.
Many added comment lines break mid-sentence at about column 100 and pack two sentences onto one line.
Production sites include `landingshed/publish.go` (`Call`, `checkApproval`, `stuckOrCancelled`), `landingshed/ctx.go` (`transientFailure`), `landingshed/finalize.go` (`pushParent`), `loomcli/arm.go` and `battencli/arm.go` (the PreStep hooks), `loomcli/cli.go`, `loomcli/driverlaunch.go`, `loomcli/driverprompt.go`, `loomcli/driverresume.go`, `loomcli/driverresumeline.go`, `loomcli/start.go`, `shedverbs/buildidentity.go`, `shedverbs/steprecord.go`, `shedverbs/step.go` (`stepErrFields`), `shedrun/paths.go` (`ParkMarker`), `shedtransient/doc.go`, `shedengine/transient.go` (file header), `shuttleengine/run.go` (`Result.NotStarted`, `ErrPaneNotReady`, `paneNotReadyError`) and `burlerengine/engine.go` (`NotStarted`).
Several new test comments follow the same wrap.
**Fix:** Reflow every added or changed comment in the diff to one sentence per line, breaking long sentences only at independent-clause boundaries; leave untouched legacy lines alone.

### [LOW] Unformatted out-of-card edit in burlerengine

**Location:** internal/burlerengine/engine.go:204
**Issue:** The card 4 commit changed `result.Outcome != shuttleengine.OutcomeDone` to `result.Outcome !=shuttleengine.OutcomeDone`.
Card 4 asks only for the `NotStarted` passthrough; this edit is unrelated, and it is what makes `gofmt -l` list the file.
**Fix:** Restore the space (`gofmt -w`).

### [LOW] IsTransient treats non-transport url.Error causes as transient

**Location:** internal/githubclient/transient.go:39
**Issue:** Card 3 counts "a transport-level `*url.Error` other than a cancellation".
The implementation accepts any `*url.Error`.
go-github's `NewRequest` returns `BaseURL.Parse`'s `*url.Error` (`Op: "parse"`) unwrapped, and `bareDo` returns the HTTP client's `*url.Error` for a TLS certificate failure (an `x509` error in its chain).
Neither is transport-level in the card's sense, and neither clears on a re-step: each is a persistent misconfiguration that would be marked `github-api`, re-stepped once, and reported as transient.
The one-re-step budget bounds the cost, which keeps this LOW.
go-github's 5xx handling matches the `*github.ErrorResponse` arm: `CheckResponse` returns `*ErrorResponse` for any non-2xx status that is not a rate-limit, abuse or 2FA case.
**Fix:** Exclude a `*url.Error` whose `Op` is `parse` and one whose chain holds an `x509` certificate error, and cover both in `transient_test.go`.

### [LOW] Parking's records-commit clause is not pinned by a test

**Location:** internal/loomcli/driverprompt_test.go:57
**Issue:** `5b9d1e72d` moved `lyx loom commit-records` out of SKILL.md; the skill's parking step 2 now runs "the records-commit command the launch prompt names".
That keeps card 14's recipe-blindness, and card 11's contract still holds: the teardown command appears once and is tied to done/busy, and parking is named for every other stop.
But `TestDriverPrompt_TiesTeardownToDoneAndBusyAndParksElsewhere` checks only "leave this session open" after the park sentence.
Dropping the records-command clause from the prompt would leave every parked driver committing nothing, with no test failing.
**Fix:** Assert that `driverRecordsCommand` appears after the "At every other stop, park" sentence.

### [NIT] --no-attach help reads out of order

**Location:** internal/loomcli/start.go:328
**Issue:** The inserted sentence "For a parked ly-drive driver it returns once the delivery of the resume line is verified." precedes "It performs steps 1 through 3 and returns once the driver's readiness signal confirms it is up", the general rule it qualifies.
It also leaves one line far longer than the rest of the hand-wrapped help paragraph.
**Fix:** Move the sentence after the readiness-signal sentence and rewrap that paragraph to match its neighbours.

### [NIT] runDriverSpawnAndWait's doc comment omits the resume branch

**Location:** internal/loomcli/start.go:109
**Issue:** The function now removes a stale park marker before a spawn and resumes a live, parked driver instead of spawning, but its doc comment still describes only the probe, the spawn decision and the two spawn arms.
**Fix:** Add a sentence naming the marker removal and the resume branch.

## Focus directives settled without a finding

- burler two-consecutive-died arm: wrapping `ErrNotStarted` on the final attempt's `NotStarted` alone is card 4's own wording.
  A mid-run death followed by a never-ready retry is classed `agent-start`; since the final failure is a start failure, one re-step of the round is the right response. Not a defect.
- `binaryChanged`: card 9 defines it as a differing (Revision, Modified) pair with both revisions known, so a dirty dev rebuild at the same revision is not expected to register. Not a defect.
- gitexec `subcommand`: `GitError.Args` never includes the leading `git` (gitexec.go builds it from the caller's args), no production caller passes `--git-dir`/`--work-tree` in space form, and `transportPhrases` matches card 2's list exactly.
- `lyx loom start` resume: the marker is removed on a verified send (absent treated as success) and before every spawn; each failure return releases `bootstrapLock`; only `ErrPaneNotReady` is retried, with an attempt cap; success falls through to the unchanged handover.
- shedengine persist: every `persist` call writes an empty `Transient` through `persistTransient`, and only the healthy-context producer-error arm records a class; the CONSTRAINTS.md Transient Stop Invariant text matches the wired boundaries (`NewShed`, `loomPreStep`, `battenPreStep`).
- Per-card mechanical check: every card group is `Create` or `Edit`.
  No transient, transport or rate-limit classifier existed before the merge base.
  `internal/loomcli/statusbuild.go` has an unexported `buildIdentity` (path, size, mtime) for the status strand; card 9 chose the VCS-stamp identity in `shedverbs` explicitly, so that is not duplication the diff can fix.
- Docs lifecycle: `docs/overview.md` and `CONSTRAINTS.md` changed in the same commits as the behaviour (cards 5, 6, 7, 9, 13, 14); no touched module has a doc under `manifest/designs/`.
  `startCmd`'s step 3 help and the `--no-attach` flag text match the implemented resume behaviour (ordering aside, F6).
