---
verdict: APPROVED
---

## Range

`_lyx/shed/shuttle-hook-first/status.json` gives `product.parent: main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, plus the Publish-failure merge `6f7b77362`.

## Publish failure

The three failing tests share one cause (`.lyx/verify/verify-4.log`): the claude engine's `Prepare` returns `RecordsSessionStarts: true`, so startup waits for a session-start line in `events.jsonl` beside the ready pane.
Both stub providers printed the ready box but never recorded that line, so shuttle reported "provider never became ready".

- `integrationWriteStubDriverScript` (`integration_driverbootstrap_test.go`) now runs `stubRecordSessionStart` before it prints the ready box.
- `writeStubDriverScript` (`smoke_driverstrand_test.go`) does the same, and both share the one `stubRecordSessionStart` const.
- The helper finds the events path in the `--settings` file's recording hook and appends `claudeengine.SessionStartupPayloadFixture`, the `startup`-source payload built from the parser's own `hookEventSessionStart` constant.

I ran `lyx gate test --tags tmux ./internal/loomcli -- -count=1 -run '<the three tests>'`.
All three pass (`ok ... 8.603s`), so none of the three Publish-failure tests is a finding.

## Other fake providers

I searched the tests for the `claude` config key and `LYX_SHUTTLE_CLAUDE` pointed at a script.
Only the two stubs above set it to a script.
`smoke_helpers_test.go` points it at a nonexistent path and never launches.
The `orchcli` smoke files use a real `claude` through `llmkit.Claude` in the llm tier.
`smoke_parentreview`, `smoke_attachprobe` and `smoke_gate` use their own `shuttleengine` engine, which does not set `RecordsSessionStarts`, so they never reach the session-start wait.
No other fake provider is affected.

## Other review points

- Comment convention: the new `stubRecordSessionStart` and `SessionStartupPayloadFixture` doc comments state purpose and contract only.
- Test economy: the fix adds no new test, and the three existing tests cover the stub change.
- The exported `SessionStartupPayloadFixture` sits in production `signals.go`, as `IdleInputBoxFixture` does in `startup.go`; it is no breach of `PATTERN-shuttle-provider-seam`, since it lives under `claudeengine`.
