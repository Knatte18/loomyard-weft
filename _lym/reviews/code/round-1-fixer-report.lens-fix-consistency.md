```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F1 change agrees with its surroundings. No findings.

- **Wording:** the new `wantErr` is a substring of what `prReviewClearFailure` formats at `internal/loomcli/prreviewgoto.go:57` for the step at `:43`. The text after `could not ` is `stop the recorded PR-Review session: ` plus the cause, and the cause opens with `shuttle: stop strand guid-1: probe reed before removing it` (`removeStrandIfPresent`, `internal/shuttleengine/stop.go:229`). The stop closure's other failure, `load the shuttle and reed config to stop the session: …`, would not match, so the row now tells a built runner's failed stop from an unbuilt runner.
- **Name:** the old row name appears nowhere else in `internal/`. The `grep` found no stale reference.
- **Doc comment:** the updated lines (`gotohook_integration_test.go:21-22`) match the new row name and assertion: a runner built by the hook, a failed probe of reed, the error returned, the record kept. The row still expects `withStrand` as the record afterwards, which agrees with "keeps the record".
- **Sibling rows:** the other two rows are untouched and unaffected.

Tool calls, read-only:
1. `grep` for `prReviewClearFailure`, `could not stop the recorded` and the old row name in `internal/`: to find stale cross-references to the old name and wording.
2. `Read` of `internal/loomcli/prreviewgoto.go` lines 20-69: my context lacked this file, which holds the error format the assertion depends on.
