---
round: 1
focus:
  - "Audit completeness (Change item 1): grep every `os.Executable` and `executablePath` use in non-test code across reedengine, reedcli, loomcli, battencli, adoptcli, orchcli and the rest. Settle whether any pin into tmux, a hook, a watchdog or a spawned process still bypasses `selfexec.LymBinary`."
  - "`internal/selfexec/binary.go`: settle whether the helper can ever return a test binary's path, including under a renamed or symlinked test binary, an injected lookup, or a PATH fallback that finds a stale `lym`."
  - "Guard coverage (Change item 3): the `reedengine` TestMain guard and its test were removed and `internal/testkit/tmuxkit` gained `reexecguard.go` plus `tmuxkit.Main`. Settle whether every package whose tests reach such a pin now runs the guard, and whether the guard exits at once on a non-test argument."
  - "`internal/gitkit/reexecguard.go` was deleted. Settle whether any former caller lost its guard and whether any reference to it remains."
  - "`internal/proc/envmarker_linux.go` and its Windows counterpart: settle correctness of the env-marker read, its failure paths, and parity between the two platforms."
  - "`internal/gatecli/tierrun.go` process cap (Change item 4): settle whether a runaway loop actually fails the gate, how the count is measured and which processes it includes, how the cap is cleaned up, and whether `processcap_test.go` covers the failure path."
  - "Rules: check the `pattern/PATTERN-*.md` edits and the constraints document for rules the diff breaks, and check the doc comments touched in `selfexec/doc.go`, `proc/doc.go` and `reedengine/doc.go`."
  - "Test economy: look for new tests that duplicate `selfexec/binary_test.go`, `tmuxkit/reexecguard_test.go` or `tmuxkit/reexecguard_integration_test.go`, and for behavior added with no coverage."
  - "Scope: settle whether any change in the range, such as the `loomcli` or `orchcli` edits, is outside the four numbered Change items. Read the commit messages for `Pattern-Departure:` trailers."
---

Seed focus drawn from the decision record's Goal and the diff stat of the review range.
