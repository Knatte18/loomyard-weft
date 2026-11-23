```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff I traced its callers, tests, docs and rule documents, and checked whether the diff left any of them wrong.

**Per change: the affected set I found, and whether it is covered**

- **`Runner.TurnOnRemoteControl` now returns `(bool, error)`.**
  - Set: `creelcli.sessionCommands` and `sessionCommandOps` (the embedded `*Runner`), the `recordingCommands` stub, `enginedispatch_test.go:270`, `session_test.go:954/962/975/987` and `shuttleengine/doc.go:170-171`.
  - Covered: all updated. No other production caller (grep in the manifest).
- **New `shuttleengine.RemoteControlReader`, with `Runner.RemoteControlCheckpoint` and `Runner.AwaitRemoteControl`.**
  - Set: the Claude implementation with its `var _` assertion, the `orchcli/cli.go:280-285` wiring, the claudeengine and shuttleengine `doc.go` paragraphs, and `PATTERN-shuttle-provider-seam`.
  - Covered: the provider specifics stay in `claudeengine`, and `shuttleengine` only does a type assertion.
- **New Claude transcript read, `bridgeSessionRecordedIn`.**
  - Set: `PATTERN-sole-parsers`. Claude's transcript is not a listed format, so it is covered.
  - Set: `PATTERN-code-conventions` named constants. **Not covered**: see the first finding.
- **The RC mark now means "the session recorded a connection" (`@lym_rc`).**
  - Set: `reedengine/remotecontrol.go`, `bar.go:21`, the `reedengine/doc.go:140` option list, `orchengine/doc.go:67-70`, `orchengine/template.yaml:11-12`, `orchengine/reinterpret.go:36`, `panebin.go:267-270`, the `remoteControlOnCmd` Long text and the orchcli file header.
  - Covered: none of these states that a click or a launch sets the mark, and the ones that did are updated.
- **`reedengine.MarkPaneRemoteControlOn` is removed.**
  - Set: its only caller was `remoteControlOnCmd`, now gone. `reedengine/remotecontrol_test.go:67-68` still names `TestMarkRemoteControlOn` as the public pin of the `set-option`, which is still true.
  - Covered.
- **`orchCLI.turnOnRemoteControl` lost its `mark` parameter and its `warning` field.**
  - Set: `remoteControlOnCmd` and `TestTurnOnRemoteControl`.
  - Covered: no reader of a `warning` field exists outside the removed lines.
- **`orchCLI.markLaunchedRemoteControl` has a new signature and a bounded wait.**
  - Set: `relaunchStrand` (start.go:785-792), reached from start (start.go:684, under the start lock) and from `relaunchOrch` (start.go:747); the test harness sets the new seams.
  - Partly covered: see the second finding.
- **creel `remote-control` delivery now waits for the connection before it marks the pane.**
  - Set: `hubDelivery.Command` → `creelengine` `worker.typeInject` (deliver.go:418) and its settle and requeue branches.
  - Covered for the affected-set wiring; the requeue-on-error effect belongs to code-correctness.
- **Commit messages.**
  - `6e15a5e24` carries a `Root-Cause:` trailer naming the defect the diff removes.
  - No `Pattern-Departure:` or `Loosened-Check:` trailers. The removed test rows pin behaviour the diff deliberately removes, so they are not loosened checks.
- **Declared PATTERN changes:** none declared, so nothing to check.
- **Stencils:** none changed.

**Findings**

1. **BLOCKING · consistency** — `internal/shuttleengine/claudeengine/remotecontrol.go:33` and `:52`.
   - The new code adds two more literals of Claude's transcript file name, `sessionID+".jsonl"`.
   - `PATTERN-code-conventions` says "a name or path that can go stale: one named constant, declared once and referenced everywhere, never a repeated literal".
   - The same literal already repeats at `transcriptread.go:137`, `audit.go:45`, `background.go:48` and `resume.go:119/196`, all from the manifest's grep. That is existing debt the diff adds to.
   - Fix: declare one named constant, or one transcript-path helper, in claudeengine and route the new sites through it.
2. **MEDIUM · design** — `internal/shuttleengine/remotecontrol.go:24` (`remoteControlConnectWait`), as reached from `internal/orchcli/start.go:791-792`.
   - `lym beam start` and orch relaunch now block up to a constant 20 s while they hold the start lock.
   - `PATTERN-daemon-wakeups` › Keep-warm cap reads: "Bounding a wait an agent blocks on: the cap from `shuttleengines`' one keep-warm resolver for the caller's role, never a constant".
   - This applies only when an agent runs the start. The bound is far below any keep-warm cap, so the practical risk is small, but the rule names the source of the cap, not its size.
   - Evidence: the background file `pattern/PATTERN-daemon-wakeups.md` (Keep-warm cap section), start.go:523 (lock) and start.go:684 (start path).

Tool calls:
1. I grepped the repo's `.md`, `.go`, `.yaml` and `.tmpl` files for RC button, `rc-on`, `@lym_rc` and `MarkPaneRemoteControlOn` to find docs, specs and fixtures describing the mark. The manifest held only the diff's own files.
2. I read `reedengine/doc.go:130-150` and grepped the orchcli, creelcli and creelengine docs for remote-control wording. I needed to check whether the package docs still describe the old mark semantics, and the manifest had not read them.
