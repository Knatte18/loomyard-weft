```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's Goal points.

**Decisions** (the decision record has no plan and no `Added after Discussion` entries):
- **G1:** "A run records the digest of each stencil it may render when it starts, in weft."
- **G2:** "Resume compares them with the installed stencils and warns the orch of each that changed, naming `lym stencil diff`."

**Changes and what they trace to:**
- **C1** — `stencilstore/pins.go` `Pins`/`ChangedPins` plus the `stencilstore/doc.go` paragraph. Traces to G1 (hashing each stencil) and G2 (the compare).
- **C2** — `shedrun/stencilpins.go`, which declares the tracked `stencil-pins.json` record under `_lym/shed/<run>/`, plus the `shedrun/doc.go` paragraph. Traces to G1, "in weft".
- **C3** — `sharedbootstrap.go` step 2d: `writeStencilPins` on the first seed only, with the record added to the seed commit's `commitPaths`. Traces to G1, "when it starts".
- **C4** — the `notifyParent` seam in `cli.go`, wired to `orchcli.NotifyParent` with `loomProgram`. Traces to G2, "warns the orch".
- **C5** — `stencilpins.go` `warnChangedStencils` and `stencilsChangedNotice`, the two success paths in `resume.go` that spawn or wake a driver, the resume long help, and the `loomcli/doc.go` paragraph. Traces to G2. The notice names `lym stencil diff`.
- **C6** — the compare on `start.go`'s success envelope, plus the start long help. Traces to G2: batten sends a halted child to `lym loom start` to bring a driver back (`internal/battenshed/innerrun.go:56`, `:60`, `notice.go:317`). That makes `start` a resume path and puts it within "Resume".
- **C7** — the `stencils_changed` envelope key added through `withChangedStencils` on the start and resume success envelopes. Traces to no decision: G2 asks for a warning to the orch, not an envelope field. No `Pattern-Departure:` trailer explains it, and C5/C6 work without it.
- **C8** — the tests (`pins_test.go`, `shedrun/stencilpins_test.go`, `stencilpins_test.go`, `stencilpins_integration_test.go`) and the `StencilsChanged` field on `resume_test.go`'s envelope. These are edits that C1–C7 force.
- **C9** — commits `e02ee4e9` and `3654a3a0`: comment wrapping and the test-tier split. These are also forced edits.

**Decisions and the changes that build them:**
- G1 is built by C1, C2 and C3.
- G2 is built by C1, C4, C5 and C6.
- Every Goal point is built. There are no `Added after Discussion` entries to trace.

## Findings

1. **LOW, class `scope`.** `internal/loomcli/stencilpins.go:16` (`stencilsChangedField`) and `:71-76` (`withChangedStencils`); it is used at `internal/loomcli/start.go:628`, `internal/loomcli/resume.go:135` and `resume.go:164`.
   - **Summary:** The diff adds a `stencils_changed` key to the start and resume success envelopes, but the Goal asks only that resume "warns the orch".
   - **Evidence:** `_lym/discussion/decision-record.md:13`. The envelope key is a second, unrequested output channel, and the summary's design decisions give no reason for it.
   - **Fix:** Drop the envelope key, its help-text and doc sentences (`start.go:512-514`, `resume.go:251-253`, `doc.go:325`) and the test's `StencilsChanged` assertion. The alternative is for the parent to record it as licensed scope.

Tool calls:
1. Searched `internal/batten*` for `lym loom start` callers, to settle whether `start` is a resume path. My context did not show batten's resume fallback.
