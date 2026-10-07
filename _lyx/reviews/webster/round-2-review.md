---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shuttleengine/rundir.go:83
    summary: Comment sentences this diff added or rewrote still break PATTERN-comment-line-breaks outside the files round 1 listed, because the fix enumerated locations instead of deriving them from the diff
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/shuttlecli/send.go:37
    summary: The behavior that lyx shuttle send appends MessageTail has no test at the CLI surface; only the WithMessageTail helper is pinned
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/wait.go:1110
    summary: finalize's message parameter and the gateLastDone memo can only ever hold an empty string now that Wait never ends on a turn end without output, so the plumbing and the test that compares two empty strings are dead
  - id: F4
    severity: LOW
    class: consistency
    location: internal/websterengine/runlevel_test.go:474
    summary: The askingMaster to diedMaster rename left its call sites as diedMaster(t, fx,"label") with no space after the comma, so the file fails gofmt
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, and no `_lyx/loom/rework/` round exists).
Evidence gathered: `go build ./...`, `go vet` under the default, `llm`, `integration` and `tmux` tags, `go test ./...` and `go test -tags integration ./...` all pass at HEAD.
Every card's mechanical check holds: no `OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking`, `MasterAskingError`, `runTriage`, `TriagePath`, `TreadleTemplateTriage`, `eventsSize`, `submitRedrawSettle` or old `clock` type use remains outside `docs/benchmarks/` and `_lyx/`.
`gofmt -l` over the diff's Go files lists `internal/websterengine/runlevel_test.go` (F4) and `internal/burlerengine/engine.go`, whose misaligned `Round:` field is on the base commit too and is not this diff's.

### [BLOCKING] Rewritten comment sentences still break PATTERN-comment-line-breaks

**Location:** `internal/shuttleengine/rundir.go:83-95` (the `RunState.Outcome` comment), `internal/shuttleengine/spec.go:140-148` (the `validate` doc's "proven live" clause), `internal/loomcli/driverspec.go:48-53`, `internal/shedadapters/singlellm.go:140-145`, `internal/burlerengine/engine.go:216-219`, `internal/treadleengine/judge.go:1-4`, `internal/treadleengine/judgeverdict.go:1-4`, `internal/treadleengine/doc.go:44-49`, and the test comments this diff rewrote in the same way under `internal/treadleengine/` (`judge_test.go`, `judgeverdict_test.go`, `template_test.go`).
**Issue:** `PATTERN-comment-line-breaks` governs new and changed comments: one sentence per line, with a further break at an independent-clause boundary.
The diff added or rewrote the sentences listed and left them on legacy fixed-column lines: a new sentence shares a line with the tail of an old one (`rundir.go:87` reads `// OutcomeDone. The legacy value "asking" is one only an older binary wrote.`), a rewritten sentence breaks mid-clause, or a deleted list item leaves a one-phrase fragment line (`treadleengine/doc.go:46` reads `// stale-artifact move-aside`).
Round 1's F1 raised this defect with an enumerated location list, and its fix reflowed exactly that list, so every location outside it was missed again.
The method that keeps missing it is listing locations by eye.
**Fix:** derive the set from the diff (`git diff -U0 <merge-base>..HEAD -- '*.go'`, added `//` lines), and reflow each sentence the diff added or rewrote to one sentence per line with no wording change.
A pure word swap or deletion inside an untouched wrapped sentence stays as it is, as round 1 decided and as the pattern's untouched-lines clause requires.

### [MEDIUM] `lyx shuttle send` appending the tail is not tested

**Location:** `internal/shuttlecli/send.go:37`
**Issue:** the diff makes `lyx shuttle send` pass its text through `shuttleengine.WithMessageTail`, a user-visible behavior of the CLI.
`internal/shuttleengine/messagetail_test.go` pins the helper alone; `cli_test.go` covers only `send`'s argument validation, and the `llm` smoke tests call `WithMessageTail` directly rather than the verb.
Nothing fails if `send.go` stops calling the helper.
**Fix:** add one test in `internal/shuttlecli` that runs the `send` verb against a started run over `shuttlefake.Reed` and `shuttlefake.Engine` and asserts the text typed into the pane ends with `MessageTail`, and that a text already ending with it is typed once with it.

### [LOW] Dead Done-message plumbing in `finalize`

**Location:** `internal/shuttleengine/wait.go:1110`, `internal/shuttleengine/run.go:284`, `internal/shuttleengine/gate_pending_test.go:104`
**Issue:** before the diff, only an asking turn end carried a message into `finalize`; a Done never did.
`Wait` now never ends on a turn end without output, so every `finalize` call passes `""`, `run.gateLastDone` is only ever set to `""`, and `Result.LastAssistantMessage` is always empty.
`gate_pending_test.go:104` compares the field with the memo, two empty strings.
**Fix:** drop `finalize`'s `message` parameter and the `gateLastDone` memo, and make the `gate_pending_test.go` assertion say the field is empty.
`Result.LastAssistantMessage` itself stays: card 5 keeps it and rewrites its doc, and removing the exported field is a plan decision for its callers.

### [LOW] `diedMaster` call sites fail gofmt

**Location:** `internal/websterengine/runlevel_test.go:474`, `:653`, `:745`, `:2207`, `:2234`, `:2254`, `:2268`, `:2296`, `:2347`, `:2358`
**Issue:** the rename of the `askingMaster` helper to `diedMaster` in card 10 left `diedMaster(t, fx,"label")` call sites with no space after the comma; the file was gofmt-clean on the base commit.
**Fix:** run `gofmt -w` on `internal/websterengine/runlevel_test.go`.
