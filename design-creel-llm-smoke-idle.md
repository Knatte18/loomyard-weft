# creel llm smoke test TestSmokeCreelClaudeStrand still fails with no pane capture

`creel-llm-smoke-idle` · Note · bug, creel, shuttle, claude

Imported from [issue #678](https://github.com/Knatte18/loomyard/issues/678).

Seen: `lyx gate test --tags llm ./internal/creelcli` fails with `smoke_llm_test.go:51: the adapter never read the booted session idle` after 60.7 s, the same message as before the #657 fix. In live sandbox use the #657 fix holds: a strand added with plain `lyx reed add --cmd claude` (placeholder `Try "fix lint errors"`) was reminded 0.5 s after mail was posted. The test prints no pane capture, so the cause is unknown; one candidate is the Claude folder-trust dialog in the test temp hub, which the sandbox also showed on first boot.

Expected: the smoke test passes, and on failure it prints the pane capture so the cause can be read.

Found by the creel stage 1 re-test of 2026-10-16.