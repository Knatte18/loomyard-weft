# Card 4 — integration-fork-log

**Edit:**
- `render.go`
- `runlevel.go`
- `template_test.go`
- `//contracts/stencils/webster/webster-template-integration.md`

**Uses:**
- `internal/websterengine#IntegrationLogPath`

**Intent:** Make the integration fork capture the verify command's combined output to a told log path, so Go can parse failures after the fork ends (D4).

`RenderIntegrationPrompt` gains a `logPath` parameter, filled into a new required marker `verify_log_path` exactly like the existing required markers (`stencil.FillOptional` requires it non-empty).
Refuse an empty `logPath` the same way an empty verify is refused.

In the integration stencil, instruct the fork to run the verify command with its combined stdout and stderr redirected to `{{.verify_log_path}}`, with the whole command grouped so the redirect covers every link of an `&&` chain (for example `( <verify> ) > {{.verify_log_path}} 2>&1`), and to take `status` from that grouped command's exit code exactly as before.
State that the fork never lists failures itself and never edits the log.
Update the stencil's header comment's marker count; add no `{{if}}`/`{{range}}`.

In `Run`, compute `IntegrationLogPath(deps.Geom.ScratchDir)` beside the integration prompt render, remove any prior log at that path (a not-exist error is ignored; any other removal error is returned), and pass the absolute path to `RenderIntegrationPrompt`.
Removing it before rendering guarantees a log present at triage time was written by this run's fork.

In `template_test.go`, update every `RenderIntegrationPrompt` call for the new parameter, assert the rendered prompt contains the log path and the `2>&1` redirect instruction, and assert an empty `logPath` is refused.

**ImpactSummary:** `RenderIntegrationPrompt` has one production caller, `Run`, plus its template tests; the seeded integration stencil gains one required marker.

**Commit:** 4: integration-fork-log
