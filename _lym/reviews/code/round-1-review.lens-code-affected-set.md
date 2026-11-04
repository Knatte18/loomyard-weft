```yaml
lens: code-affected-set
outcome: reported
```

This is the code-affected-set lens. I found no finding at BLOCKING, MEDIUM or LOW, and one at NIT.

**Set per change**

- **`SessionOpener.OpenLaunch(dir, spec, cfg)`**
  - Implementers: `claudeengine/command.go` is the only production one (`codexengine` has none).
  - Callers: `open.go:79` is the only production caller.
  - Test fakes: `shuttlecli/cli_test.go`, `enginedispatch_test.go` and `fakes_test.go` are updated.
  - Docs: the `SessionOpener` comment, `doc.go` (Opened sessions) and `claudeengine/doc.go` are updated.
  - Other mentions of `lym shuttle open`: `shuttlecli/open.go`, `reedcli/add.go` and the reed templates mention it only as a command, and none says hooks are missing.
  - Verdict: covered.
- **`Launch.CreelHooks` onto the strand in `Open`**
  - Consumers: `creelengine` (`seam.go`, `deliver.go:242`, `verbs.go:123`, `doc.go`) and `reedengine/state.go` take the mark as opaque.
  - The `creelengine/doc.go:22` list of pane-mail strands drops "`lym shuttle open` session" and now names a Codex session. `codexengine` has no `OpenLaunch` and no creel hooks, so a Codex session is still pane-mail. This is consistent.
  - Verdict: covered.
- **`DeliverPendingPrompt`: match on instruction, send at most once per handle, `sendMu` held for the whole call**
  - Callers: `burlerengine/fix.go:263` through `promptDeliverer`, and `hold.go`.
  - `ResumeFix` passes the same `live.Chair` handle to `seatengine.Resume`, so once-per-handle holds on that path.
  - Tests: `run_test.go`, the `enginedispatch_test.go:443` consult test and the `burlerengine` fakes still compile against the unchanged signature.
  - Docs: `shuttleengine/doc.go` (Pending prompt), the `loadSkillsThenPrompt` and `DeliverPendingPrompt` comments, and the `docs/overview.md` shuttle entry are updated.
  - `burlerengine/fix.go:241` says "when the prompt's turn never started", which is still correct.
  - Verdict: covered.
- **Hold notice suppressed when the prompt is typed, Wait entry delivery for an attached run**
  - Docs: `shuttleengine/doc.go` and the `overview.md` sentence both cover it.
  - Verdict: covered.
- **Open launch files (`settings.json`, `append-system-prompt.txt`) under `.lym/shuttle-open/<token>/`**
  - PATTERN fit: `PATTERN-durable-vs-ephemeral-state` is respected. The path comes from `lymdirs.DotLymDirName`, which `open.go` already used, and nothing is tracked.
  - `PATTERN-shuttle-provider-seam` is respected: the provider-specific files are written only under `claudeengine`, and `shuttleengine` passes only a directory.
  - `overview.md` says "every Claude session it launches with a run directory also gets the creel trust sentence". An opened session now gets it too, but the same entry covers that in the new open sentence. This is not wrong.
  - Verdict: covered.
- **Declared PATTERN changes and commit trailers**
  - The review prompt's constraints section declares no PATTERN changes, and the diff does not touch `PATTERN.md` or `pattern/`.
  - The three commits carry no `Pattern-Departure:` trailer, and none is needed.
  - Verdict: nothing to land.
- **Stencils**
  - The diff changes none.

**Finding**

1. NIT, `consistency`, `internal/shuttleengine/open.go:72-87`.
   - The session directory `.lym/shuttle-open/<token>/` is created by `writeOpenFiles` and removed only when `AddStrand` fails.
   - Nothing removes it, or the sibling `<token>.exited` marker, once the session ends. I found no cleaner in a search of the non-test Go files for `shuttle-open` and `.exited`, so the files accumulate under `.lym`.
   - The marker already behaved this way, but this diff adds files to the leak.
   - If `writeOpenFiles` fails after writing `settings.json`, the partly written directory is also left behind.
   - Evidence: `command.go:447-468`, `open.go:101`.

Tool calls:
1. Ran grep for `OpenLaunch` and for open/hooks mentions across docs and Go, to find the other implementers, callers and docs the manifest lacked.
2. Ran grep for `DeliverPendingPrompt`, `promptTurnStartedSince` and `CreelHooks`, to find the other consumers.
3. Ran grep for `shuttle open` and `shuttle-open`/`.exited`, to find any other docs and any cleanup of the session directory.
4. Read the `shuttle open` context in `shuttlecli/open.go`, `reedcli/add.go` and the reed templates, to check whether any of them claims the session has no hooks.
