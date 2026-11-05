```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. Result: no findings.

- **F2:** A grep for the old "Claude engine's hook" wording finds no twin left in Go or markdown files. The only remaining "claude engine" mentions are `internal/creelcli/cli_test.go:157` and `:163`. Those describe the output format a test expects, not the verb's spelling, so they are still true.
- **F4:** No `strandFlagName` or `creelTrustSentence` identifier remains outside `shuttleengine.StrandFlagNameShell` and `shuttleengine.CreelTrustSentence`. All of these use the shuttleengine names directly:
  - the claudeengine sites in `settings.go:118`, `:141`, `:463`, `:475` and `:476`;
  - the claudeengine tests;
  - the codexengine sites in `command.go:183` and `:191`, and in `codexengine.go:226`.

  The deleted `creelTrustSentence` doc comment carried the double-quote rationale, which the shuttleengine constant's own comment at `creelhooks.go:12` still states.
- **F3:** The `codex.yaml` binary and the expected `bin` in `TestPrepare_CreelHooksInAHub` now match, and its doc comment was updated to say so. No other test builds a `creelDelivery` with the default `codex` binary in this change set.

Further tool calls:
1. One `grep` over the Go and markdown files for the old comment wording and the removed identifiers, plus `git show --stat` of the last three commits. My context did not show whether any twin of the changed text survived elsewhere.
