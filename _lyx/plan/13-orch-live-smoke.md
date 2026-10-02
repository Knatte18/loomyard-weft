# Card 13 — orch-live-smoke

**Edit:**
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchengine/doc.go`

**Create:**
- `internal/orchcli/smoke_live_test.go`

**Intent:**
Extend the build-tagged orch smoke suite to cover what this task changed, run it against this machine's Claude install, and record what it showed.
Every new test follows `TestSmokeOrch_OneFullCycle`'s shape: it builds `cmd/lyx` and drives it as a subprocess, skips without tmux or `claude`, and stops orch and takes reed down in cleanup.
Every `orch.yaml` the suite seeds carries the full key set, since a config missing a template key fails to load.
`TestSmokeOrch_OneFullCycle` gains `permission_mode: bypass` and a `soft_threshold_tokens` above its `threshold_tokens`, so its manual cycle stays the only trigger.

New tests in `smoke_live_test.go`, sharing the existing helpers:

- `TestSmokeOrch_BypassAgentAndIdle`: after `orch start`, the session runs a Bash command that writes a marker file, which appears with no permission dialog ever captured in the pane;
  the session spawns a subagent through the Agent tool that writes a second marker file, which appears, and the transcript holds the Agent call with no deny;
  after that turn ends, the real engine's idle probe (`Runner.SessionIdle`) passes within a bounded number of polls.
- `TestSmokeOrch_SoftCycle`: with `soft_threshold_tokens: 1` and a hard threshold out of reach, the session starts a long-running background shell and ends its turn.
  Every reading qualifies at that soft threshold, the start prompt's own turn end included, so `soft_idle_s` is set long enough (tens of seconds) for the test's background-shell turn to land before the start turn's quiet window closes;
  then that turn's end, not the start prompt's, starts the soft cycle;
  `status` reaches `cycle_count` 1 with `cycle_trigger` `soft`, the handoff file is non-empty and names the background shell, the transcript changes across `/clear`, and the resumed session launches a background shell again.
- `TestSmokeOrch_Adopt`: a plain interactive shuttle run in the prime (`bypass`) is told a random codeword and ends its turn.
  `orch start --adopt <its session id>` while that run is live refuses, naming the live holder.
  After its strand is removed and its process has exited, `orch start --adopt <id> --no-attach` succeeds with `prompt_source` `adopt`, and the adopted session answers the codeword, proving it kept its context.
  The orch strand's name in reed is `<shortname>:orch`;
  the session runs `lyx loom start --no-attach` in a hubforge pair worktree, and that run's seed records `<shortname>:orch` as `parent`;
  a second shuttle run in the prime sends a marker to `<shortname>:orch` through SendMessage, as a driver's escalation notice does, and the marker reaches the orch transcript.
  A manual `orch cycle` then completes on the adopted session.

The existing background-task and SendMessage-address observations keep logging without failing.

Run the suite with the verify command below.
Record the outcomes in the orchengine package doc: replace the "unverified, pending a smoke run" Open risks with what the run showed for a background task across `/clear` and a `SendMessage` address across `/clear`, naming the Claude Code version and the date of the run;
state whether `--resume` with a positional prompt submitted the prompt, and whether any startup dialog appeared on resume.
If `--resume` with a prompt did not submit it, record that in Open risks and stop and report rather than adding the fallback in this card, since the fallback edits production files outside this card's targets.
The report names the decision record's fallback as the follow-up fix: claudeengine's launch line resumes without a positional prompt, and orch's adopt branch sends the one-line adopt stencil once startup clears.

**ImpactSummary:** Test-only code behind the smoke tag plus orchengine's package doc; no production code changes.

**Verify:**
go test -tags smoke -run 'TestSmokeOrch' -timeout 90m -v ./internal/orchcli/
