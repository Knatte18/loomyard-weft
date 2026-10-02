# orch live-ready

All 13 batches reported `done`, and the plan-level integration suite reported `OK`.

- **Cards 1–4, 6:** `Spec` gained `PermissionMode`, `AllowAgentTool` and `ResumeSessionID`, realized in `claudeengine` alone, with its resume check over the transcript and Claude's session registry.
  Every generated settings file now switches prompt suggestions off.
  Card 2 reported one deviation: `claudeengine/prepare_test.go`, updated for the new `buildDenyNotice` signature.
- **Card 5:** `SessionResumer` is an optional capability that `Runner.start` checks before a resume.
- **Cards 7–10:** `orch.yaml` carries `permission_mode`, `soft_threshold_tokens` and `soft_idle_s`.
  The adopt and soft-handoff stencils exist.
  The watcher has a deferrable soft trigger with a recorded trigger and last deferral, and `lyx orch status` reports them.
- **Cards 11–12:** the orch run uses the configured permission mode with the Agent allowance and forks, and `lyx orch start --adopt <session-id>` resumes an existing session as the orch strand.
- **Card 13:** the live smoke suite was extended and run against a real Claude Code install, and its four tests passed.
  Outcomes are recorded in the orchengine package doc.
  Reported deviations: the idle probe's `isBoxRule` in `claudeengine/session.go` (and `session_test.go`) was fixed, because Claude draws the session name into the input box's top rule and the probe rejected it.
  The soft-cycle test logs the background-shell count instead of asserting it.
  A pane-squeeze finding at hub level (a visible plain run shrinks the orch pane to 3 rows and stalls the idle probe) is not fixed.

Integration verify (`go build`, the integration-tag tests, the smoke-tag vet) exited 0 at HEAD `7a6a5597d`.
