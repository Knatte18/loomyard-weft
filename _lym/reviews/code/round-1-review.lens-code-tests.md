```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**
1. `ensureRunsStrand` replaces a `loom-status` strand when no `runs` strand exists: pinned by `start_test.go` row "a loom status strand holding the overview is replaced by the runs strand". It checks the `replace` call and the sidecar GUID.
2. `ensureRunsStrand` returns the add-failure warning, and `beamUp` appends it to `report.Warnings`: pinned by the "a failing add..." row, which asserts the exact text through the `warnings` field of the envelope.
3. `ensureRunsStrand` returns warnings for its other failures: none of these are pinned. They are the build, executable and strands-read errors, the runs replace failure, the loom-status replace failure and the sidecar write failure. `runsStrandWarning` is shared, so the formatting is covered once, but the loom-status replace failure branch has its own distinct text.
4. Loop precedence: a `runs` strand wins over a `loom-status` strand, in either order in the strand list. None. Every `start_test.go` row has either a `runs` strand or a `loom-status` strand, never both. If the switch were reordered or the loop changed so `loomStatus` won, no row would fail.
5. The early return for a current `runs` strand: pinned by the existing "keeps a live current runs strand" rows at `start_test.go:304` and `:311`.
6. `statusStrandAddSpec` chooses the overview display when strands are absent or unrelated, and an own-window display with `KeepViewer` beside a `runs` strand: pinned by the three rows of `TestStatusStrandAddSpec`. The `runs` match goes through `agentname.Matches`, but the row uses only the bare name `runs`. A name carrying an agent prefix is not exercised.
7. `ensureStatusStrand` passes `statusResult.Strands` into `statusStrandAddSpec` (`sharedbootstrap.go:279`): none. The smoke tests pass `nil`. Reverting that argument to `nil` would fail nothing.
8. Reed accepts the own-window `KeepViewer` status strand beside an overview `runs` strand, and the re-entrant keep/replace path still finds it: none. No Tier 2 test places a status strand beside a runs strand.
9. `beamUpWarningLine` text and its empty case: pinned by `TestBeamUpWarningLine`.
10. `surfaceBeamUpWarnings` and the Beam-Up step closure: none. This covers three things: the closure calling `surfaceBeamUpWarnings` with `report.Warnings`, a failed `NotifyOrchAt` only logging, and the success path returning nil. Reverting to `_, err := orchcli.BeamUp(...)` would fail nothing. This is fix 2 of the task.
11. The renames (`RunsStrandName`, `LoomStatusStrandName`) and the doc edits are not behaviour.

**Findings**
- MEDIUM, design, `internal/loomcli/pawlwiring.go:99-108` and `:152-163`: nothing pins that Beam-Up surfaces the report's warnings or that a failed post never halts the row. Only the pure `beamUpWarningLine` is tested. Evidence: `pawlwiring_test.go` diff adds only `TestBeamUpWarningLine`. `NotifyOrchAt` is called directly with `location`, so there is no seam. Suggested fix: give `surfaceBeamUpWarnings` or the step a notify seam and add a row for the empty warnings, the post and the post-failure cases.
- LOW, design, `internal/orchcli/runs.go:71-78` with `start_test.go:291-335`: no row has both a `runs` strand and a `loom-status` strand, so the precedence of `runs`, and a `runs` entry after a `loom-status` entry, is unpinned. Suggested fix: add rows to `TestStart_EnsuresTheRunsStrandBeforeTheOrch` with `[loom-status, runs]` and `[runs, loom-status]`, expecting a replace of the `runs` GUID.
- LOW, design, `internal/loomcli/sharedbootstrap.go:279`: the call-site wiring of the session's strands into `statusStrandAddSpec` is unpinned, because the smoke tests pass `nil`. Suggested fix: cover it in a Tier 2 smoke row that adds a `runs` strand and then runs `ensureStatusStrand` or `statusStrandAddSpec` against it. That would also settle item 8.
- LOW, design, `internal/orchcli/runs.go:90-94`: the loom-status replace failure warning (distinct text) and the other warning branches have no row. Suggested fix: add a `replaceErr` row for the loom-status case, if the harness has one.

**Tool calls:** none. Everything came from the inherited context.
