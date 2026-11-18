```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- F7: the new `internal/driftengine/doc.go` lines state only driftengine's own behavior. They agree with the neighbouring bound ("still named as a change") and no longer repeat the refusal. `internal/loomcli/doc.go:166` still carries the refusal, so nothing was left without a statement.
- F8: the keep marker sits below the doc comment, directly above the `func`, after a bare `//` separator. That matches the existing markers in `internal/burlercli/cli_test.go` and the placement rule in `pattern/PATTERN-test-speed.md:117`. Its reason names the covering test the review named.

Calls made (2 of 6, one Bash command):
- `grep` of `internal/loomcli/doc.go` for the verb's refusal text, to confirm it is still stated there; my context did not hold that file.
- `grep` of `pattern/PATTERN-test-speed.md` for the marker placement rule, to confirm the marker position; my context did not hold that file.
