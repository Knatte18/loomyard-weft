```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **F1:** the `wire_seams_test.go` edit leaves no stale reference. `fmt` and `os` are still used, and the `proc` import is gone with its only use. The test still isolates HOME and creates the empty registry directory, so it never reads the real session registry. The remaining `liveness-unproven` hits are only in a dated research doc, which is a record of a past run, not a sibling that must agree.
- **F2:** the new `run.go` message matches the `AnchorHidden` and `AnchorUnknown` rows in `cli_test.go`. The `AnchorOverview` and `AnchorEmpty` rows assert only `own-window`, which the plain message still contains. The flag help text at `run.go:202` and the `shuttleengine/doc.go:79` sentence both say the same thing: hidden is refused because reed starts no command for it. `reedcli/add.go:80` still accepts `hidden`, which is a different command.

Calls made:
- grep for `liveness-unproven`, `reed starts no command` and `invalid --anchor`: to find sibling texts and twins of the changed strings.
- grep of `wire_seams_test.go` for `registry`, `os` and `fmt`: to confirm the imports and the leftover setup are still consistent.
