---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/tendercli/cli.go:280
    summary: The status verb's Short and Long still describe only the entries, network operations and parent locks, not the sessions and stale-process parts card 9 added
    fix: In statusCmd, extend Short and Long to also name the listing of every live session in the hub with its state and cause, and the lyx processes running a replaced binary, matching the Status section of internal/tenderengine/doc.go and the tender bullet in docs/overview.md.
  - id: F2
    severity: LOW
    class: consistency
    location: docs/overview.md:410
    summary: The tender bullet says an agent may pause or resume only its own pair, but revive follows the same caller rule and the rest of the bullet, tendercli/doc.go and the refusal spec already name all three verbs
    fix: In docs/overview.md line 410 change "An agent may pause or resume only its own pair; the orchestrator and an external caller may do both for the hub or any pair." to name revive as well, for example "An agent may pause, resume or revive only its own pair; the orchestrator and an external caller may do each for the hub or any pair."
  - id: F3
    severity: NIT
    class: consistency
    location: internal/tendercli/cli.go:63
    summary: Doc comments of Command and RunCLI still describe the pre-tender module, with Command omitting the revive verb and RunCLI calling it the pause module CLI
    fix: In internal/tendercli/cli.go change the Command doc comment to "the bare group and its pause, resume, revive and status verbs", and the RunCLI doc comment at line 320 to "the public seam for the tender module CLI".
  - id: F4
    severity: NIT
    class: consistency
    location: docs/overview.md:396
    summary: The shortcuts bullet lacks a comma between the attach/a item and the prime item
    fix: In docs/overview.md line 396 insert a comma after "(both `lyx reed attach`)" so it reads "`lyx attach` and `lyx a` (both `lyx reed attach`), `lyx prime` (...)".
---

### [LOW] status verb help omits the sessions and stale-process parts

**Location:** `internal/tendercli/cli.go:280`
**Issue:** Card 9 made `lyx tender status` list every live session and the processes running a replaced binary.
`internal/tenderengine/doc.go` and `docs/overview.md` say so, but the verb's own `--help` text (`Short` and `Long`) still describes the pre-card-9 report.
An operator reading `lyx tender status --help` is not told what the verb now shows.

### [LOW] overview.md caller rule leaves revive out

**Location:** `docs/overview.md:410`
**Issue:** `refuseCaller` applies the same agent rule to `revive` as to `pause` and `resume`, and the refusal spec row, `tendercli/doc.go` and the error text all name the three verbs.
The overview bullet that card 10 edited still names two.

### [NIT] Command and RunCLI doc comments predate the tender module

**Location:** `internal/tendercli/cli.go:63`
**Issue:** The `Command` comment lists pause, resume and status but not `revive`, and the `RunCLI` comment still says "pause module CLI".

### [NIT] Missing comma in the shortcuts bullet

**Location:** `docs/overview.md:396`
**Issue:** The list reads "`lyx attach` and `lyx a` (both `lyx reed attach`) `lyx prime` (...)" with no separator before `lyx prime`.

## Review notes

The review range is `git diff $(git merge-base main HEAD)..HEAD`: no `_lyx/loom/rework/` round exists, and `product.parent` in `_lyx/shed/tender/status.json` is `main`.

Each focus question was settled against the code and found met:

- Cards 2 and 3: `requestContext` moves a held call's deadline by exactly `held` and uses the caller's context unchanged for `held` zero.
  Every multi-request caller (`listOpenIssues`, `ListIssueComments`, `CommentAndClose`, `fetchOpenPRHeads`) derives a deadline per request, and none keeps a shared one.
- Card 4: `conditionedReading` refuses at once on an unmet state, a picker, a draft or operator activity, and plays no key.
  `keepUnlanded` stops the unlanded-text clear, and `awaitTurnStart` is bounded by one submit window and a poll cap.
- Cards 5 and 6: the notice is de-duplicated in `queueNotice` and dropped in `deliverNotice` once its state no longer reads.
  `restartSafe` re-reads state and the pause record after the tick, and the watcher stays the only writer into the orch pane.
- Cards 8 to 10: `reviveSession` runs the coverage read and the conditioned send inside `pauserecord.Update`.
  Process listing, session enumeration, the conditioned send and the notice queue are all `Deps` seams, and `tenderengine` imports no `lyxcwd`.
- Card 11: `handOver` execs on Unix and runs the unchanged child path on Windows, and `shortcutProblems` pins `reed attach` last.
- Card 12: commit `5348618b4` is a pure `git mv` plus surgical edits, git detects every file as a rename, and no `pausecli` or `pauseengine` remains outside `_lyx/` and `docs/research/`.
- Test economy: the top-level test counts match the new `test-budget.yaml` rows, and each new test covers a behavior no existing test reaches.
- `lyx gate test` over the touched packages and `cmd/lyx` passes.
