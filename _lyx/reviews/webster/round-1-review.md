---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/orchcli/watch.go:53
    summary: watchCmd wiring of newIndexSource to the recorded strand path has no test, and the TestWatch rows card 14 names were not added
  - id: F2
    severity: LOW
    class: design
    location: internal/orchengine/watcher.go:847
    summary: startAutoReload goes idle on a render failure but leaves compactedAt set, so every tick until the next turn end re-runs the index source and rewrites state
  - id: F3
    severity: LOW
    class: consistency
    location: internal/boardcli/payload.go:12
    summary: the board index note follows the Short with only a space, reading as a run-on sentence, and is printed under the internal audience where no payload command appears
  - id: F4
    severity: NIT
    class: consistency
    location: internal/orchengine/watcher.go:155
    summary: an index source failure is reported as role stencil failed to render, which blames the stencil for a binary or strand problem
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (no rework round exists; `product.parent` is `main`), the 17 card commits.
No Publish failure is on record.

Evidence gathered, not re-derived from the plan's own gates:

- Built `./cmd/lyx` from `HEAD` and ran it: `lyx help`, `lyx help index` (operator, `--audience role`, `--audience internal`), `lyx help index --audience bogus`, `lyx help no-such`, `lyx help board bogus`, `lyx fabric merge --help`, `lyx webster reset --help`, `lyx loom goto --help`.
  `SetHelpCommand` followed by `InitDefaultHelpCmd` works in a real execution, the refusals are JSON with the offending word, and the registration oracle reads `SetHelpCommand` arguments.
- `lyx gate test` over `./cmd/lyx`, `clihelp`, `helpcli`, `boardcli`, `boardengine`, `selfreportengine`, `orchcli`, `orchengine`, `reedengine` is green, and `go vet -tags integration` over the changed packages is clean.
- The audience values in the rendered role and internal indexes match every per-card classification the plan names (`reed switch`, the webster batch verbs, `burler` verbs, `fabric merge-stage`, `shuttle send|interrupt` as role; `watch`, `resume-context`, `watchdog`, `board sync|rerender` as internal; every `status` verb, `webster reset`, `shuttle state` as operator).
- Cards 6 to 8 changed no runtime code in `fabriccli`, `loomcli` or `webstercli` beyond gofmt realignment, so no refusal lost its way forward; the trimmed pages still carry the flag explanations the plan moved out of `Use` (`merge --continue|--abort`, `reset --to` targets, `goto --to`).
- No card carries a Rename or Delete group; the type-specific checks that apply are the `->` re-signatures, and each target matches its stated signature with every caller compiling (`resolveLookup`, `decodeIntakePayload`, `Store.SetDeps`, `Board.SetDeps`, `inboxIssue`, `issueContent`, `composePaneLaunchLine`, `RenderRoleFile`, `renderFile`, `ResumeContext`, `NewWatcher`).
- The tree walk `TestCLIForm_EveryCommand` is the one form guard; no per-package test restates it, and `TestPayloadUse_MatchesDeclaredKeys` pins a different property (Use token against parser declaration) with a proven mismatch row.
- Card 9's `set-deps` addresses by `id` through `resolveLookup`, the unknown-key refusal text is unchanged, and the lookup-contract step covers `id` success and the both-keys refusal.
- Cards 10 to 12 handle zero comments (body byte for byte as before), the filtered whole-body import pointer, a quoted pointer kept verbatim, two-page pagination, and a failed fetch leaving the board unchanged.
- Cards 13 and 14: the strand records `LyxBin` at every `launchStrandLocked` call, `Status` reports it, the three render sites call the index source, `startAutoReload` goes idle with the reason instead of failing the tick, and only the orch strand's recorded path is used by the watcher.

### [LOW] watchCmd's use of the recorded-binary index source is untested

**Location:** `internal/orchcli/watch.go:53`, `internal/orchcli/watch_test.go`
**Issue:** Card 14 asks `TestWatch` to gain a row for a strand with a recorded path, whose fake runner output lands in the role file at a reload render, and a row for a strand without one that goes idle naming `lyx orch stop`.
The diff adds `TestNewIndexSource`, which pins the source's own path selection and refusals, but nothing reaches `watchCmd` with `newIndexSource(c.strands, c.indexRunner)`; `TestWatch` is unchanged and `indexRunner` is installed by `newTestCLI` yet unused there.
Replacing that line with the in-process `clihelp.RenderIndex`, which the plan and `PATTERN-pane-binary-resolution` forbid for the watcher, would pass every test.
**Fix:** Cover the wiring in one assertion or row that drives the command (or the construction it performs) far enough to show the runner is called with the strand's recorded path.

### [LOW] a failed auto-reload render retries on every tick

**Location:** `internal/orchengine/watcher.go:847`
**Issue:** `startAutoReload` is entered from `tickIdle` whenever `w.compactedAt` is non-zero, and `compactedAt` is only reset when `startAutoReload` succeeds, at `initCursor`, or when a later turn end is read.
The new idle-with-reason branch returns through `toIdle` without clearing it, so until the next turn end every tick re-runs `probeIdle`, calls the index source again (a `<bin> help index` spawn whenever the recorded path exists but the runner fails), rewrites the state file and clears the resume mark, and `tickIdle` never reaches the cycle-request and threshold logic below it.
Before the change the same failure was a counted tick error; now nothing bounds the retries.
**Fix:** Clear `w.compactedAt` when the render fails and the watcher goes idle, so the failure is reported once and the next boundary retries it, or state in the doc comment that the retry per tick is intended.

### [LOW] the board index note runs into the Short and prints under every audience

**Location:** `internal/boardcli/payload.go:12`, `internal/boardcli/cli.go:43`
**Issue:** `RenderIndex` appends the note after one space, so the operator index reads `board: read and write the task-tracker board's tasks and notes A payload is one JSON object ...`, with no delimiter between the Short and the legend.
The note also rides on the `board` group, so the `internal` audience prints it above `rerender` and `sync`, which take no payload.
**Fix:** Start `payloadLegend` with a delimiter that reads as a separate statement (it also ends the group's `Long`, so pick one that works in both places), and consider attaching the note only where a payload command is listed.

### [NIT] an index source failure is worded as a stencil failure

**Location:** `internal/orchengine/watcher.go:155`
**Issue:** `renderRoleFile` returns `command index unavailable: ...`, and the three call sites wrap every error as `role stencil orch-template-role failed to render: ...`, so a missing orch binary reads as a stencil defect in `LastAbortReason`.
The way forward is still in the message tail, so the operator can act on it.
**Fix:** Word the reason by source (index versus stencil) at the call sites, or drop the stencil prefix for the index error.
