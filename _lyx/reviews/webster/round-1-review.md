---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/pushanchored.go:1
    summary: new and changed Go comments in the diff break sentences mid-line at a fixed column, violating PATTERN-comment-line-breaks
  - id: F2
    severity: LOW
    class: consistency
    location: internal/statuscommit/statuscommit.go:21
    summary: Deps.Push and the push-warns comment say the push error names the side that failed, which is false for batten's PushAnchored and for a lock expiry
  - id: F3
    severity: LOW
    class: design
    location: internal/lock/lock.go:62
    summary: AcquireWriteLockWithin carries a dead locked-false-with-nil-error branch that flock's TryLockContext cannot produce
  - id: F4
    severity: LOW
    class: scope
    location: internal/fabricengine/pushanchored_integration_test.go:238
    summary: PushPairAnchored's unborn-HEAD skip and the retry helper's second-push-rejected-again return have no test
  - id: F5
    severity: LOW
    class: consistency
    location: internal/loomcli/cli.go:379
    summary: the lyx loom help text says commit-records pushes the run's records only, but it now pushes the task branch too
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD`, since `product.parent` is `main` and no `_lyx/loom/rework/` round exists.
`go build ./...`, `go vet` over the touched packages, `go test ./...` and the plan's integration-tag set all pass on this tree.
The five commits match their cards on the points the focus directive names, apart from the findings below.

### [BLOCKING] New and changed comments hard-wrap mid-sentence

**Location:** `internal/fabricengine/pushanchored.go:1-4`, `:19-31`, `:34-37`, `:60-63`, `:78-103`, `:130-143`; `internal/fabricengine/doc.go:582-592`; `internal/fabricengine/export_integration_test.go:68-69`; `internal/fabricengine/pushanchored_integration_test.go:3-5`, `:210-211`, `:234-238`, `:323-326`; `internal/fabricengine/coalesce_integration_test.go:150-153`; `internal/fabriccli/spawn.go:2-6`, `:11-12`; `internal/fabriccli/weft_verbs.go:14-15`, `:319-321`.
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line, with a further break only at an independent-clause boundary, for every new or changed Go comment.
Each location above holds a comment line this diff added or changed that stops in the middle of a sentence and carries the rest on the next `//` line, as in `StatusPushLockWait is the bounded push-lock wait the per-transition status pushes and` followed by `commit-records pass: ...`.
The surrounding file's older wrapped comments do not excuse these lines: the PATTERN entry calls them legacy and limits itself to new and changed comments.
The same convention is not the only constraint here: no width may be inferred from the neighbouring comments.
This is a break-placement finding, not a line-length one.
**Fix:** reflow each listed comment so every sentence starts its own line and a sentence is split only at a `, but` / `, and` / `;` clause boundary with its own subject and verb.
Leave the untouched lines of those comment blocks as they are.
In `doc.go`, the added sentences that end mid-line (`without importing gitrepo. MergeStateActive`) are split at the sentence boundary too, without rewrapping the older lines around them.

### [LOW] Deps.Push says the error names the side, which is not true for every wiring

**Location:** `internal/statuscommit/statuscommit.go:21-22` and `:44-45`.
**Issue:** `Deps.Push` is the shared contract of both `loomcli` and `battencli`, and its new line reads `Its error names the side that failed.`
Batten's `Push` runs `PushAnchored`, which returns the bare `gitrepo.ErrPushRejected` with no side in it, and a bounded lock expiry returns an error that names neither side.
The push-warns bullet repeats it as `The warning carries the entry's own error, which names the side that failed.`
Card 4 asked only for the loom entry's comments to say so, and `loomcli`'s own `Push` comment is the place where it holds for a rejection.
**Fix:** drop the side-naming sentence from `statuscommit`'s `Deps.Push` comment and from the push-warns bullet, since the core cannot promise it for either caller.
Keep the sentence in `loomcli`'s `commitStatusDeps.Push`, limited to what `PushPairAnchored` does (a failed side is named, a lock expiry names none).

### [LOW] AcquireWriteLockWithin has an unreachable branch

**Location:** `internal/lock/lock.go:62-64`.
**Issue:** After the `context.DeadlineExceeded` and other-error branches, the function returns `(nil, false, nil)` when `!locked`.
gofrs/flock's `TryLockContext` returns `true`, a non-nil error, or the context's error, so `locked == false` with a nil error cannot occur there.
The branch is code for a state the callee's own contract rules out.
**Fix:** delete the `!locked` branch and let success fall through to `&FileLock{fl}, true, nil`.

### [LOW] Two behaviors of the new push helpers have no test

**Location:** `internal/fabricengine/pushanchored_integration_test.go:238` (`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`).
**Issue:** `PushPairAnchored` skips a side whose HEAD is unborn, and card 3 states it, but no test reaches it.
The retry helper returns the bare `gitrepo.ErrPushRejected` when the second push is rejected as well, and card 3 states that too; the diverged-code-side step only reaches the `HasUnpulled` true return, so the second-push path is unreached.
`PATTERN-test-economy` and the rubric count a behavior the diff adds with no coverage as a finding.
The fetch-failure return of the retry helper is also untested, but a remote that rejects a push and then fails a fetch cannot be built on a local bare without hacks, so it is left out of this finding.
**Fix:** extend the existing scenario test with two more steps, not new test functions:
a step that moves the warp worktree onto an unborn orphan branch and asserts `PushPairAnchored` returns nil, pushes the records side and leaves the warp bare unmoved;
a step that installs an always-declining `pre-receive` hook on a bare and asserts the error satisfies `errors.Is(err, gitrepo.ErrPushRejected)` with the local HEAD unchanged.

### [LOW] The loom help text still describes commit-records as a records-only push

**Location:** `internal/loomcli/cli.go:379-380`.
**Issue:** The command's `Long` help says `"commit-records" commits and pushes the run's records (status, reviews, friction notes, drive reports)`.
The diff changes the observable behavior of that verb so that it also pushes the task branch, and updates `docs/overview.md` and the spec, but the CLI's own help is the text an operator reads at the command.
**Fix:** say in that sentence that the push carries the task branch along with the records, in the wording `docs/overview.md` uses, naming no side.
