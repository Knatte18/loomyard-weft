---
round: 2
focus:
  - "Table-driven tests that run t.Run inside a range loop and whose names did not exist on base commit 9cd12fd19, for example in internal/landingshed (finalize_test.go, publish_test.go), internal/logger, internal/orchengine, tools/sandbox and cmd/lyx/exitsweep_test.go: settle whether each calls t.Parallel at the top level and per subtest, or carries a comment naming the global state (t.Setenv, t.Chdir, a package-level stub, the process-global logger) that forbids it, as PATTERN-test-speed requires."
  - "Comment lines added or changed by the diff in the integration test files, for example internal/battencli/lifecycle_integration_test.go, internal/logger/sink_callsite_integration_test.go, internal/configcli, internal/fabriccli, internal/boardengine/sync_integration_test.go and cmd/lyx/stencilseed_integration_test.go: settle whether any new comment sentence still breaks mid-sentence, against PATTERN-comment-line-breaks. Lines byte-identical to base text are out of scope."
  - "Test helpers in internal/landingshed/prgate_test.go and internal/reedcli/smoke_test.go, and pattern/ or docs files that name a test: settle whether any helper has no caller under the default, integration, tmux and llm tags, and whether any doc or pattern background still names a top-level test that the prune deleted or turned into a subtest, such as pattern/PATTERN-batten-bookend.md."
---

The focus entries target the three mechanical sweeps the plan's prune procedure ends on, applied to the whole diff range.
