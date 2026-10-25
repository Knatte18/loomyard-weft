```yaml
lens: fix-affected-set
outcome: reported
```

Findings: no findings.

- **F1:** `moveWeftContent` has two callers, `moveWeft` (`anchorchange.go`) and `addanchor.go`. `moveWeft` was updated to the new `(committed bool, err error)` result. `addanchor.go` discards the first result, so it is unaffected. The integrate path never calls `moveWeftContent`. The gate tier-1 run of `fabricengine` passed.
- **F2:** Every stop built on `MoveInProgress` goes through the changed lock-held branch, so none was left on the old wording. A search of the tests found one other assertion of the old wording, a dead pid under a held lock, in `addparent_integration_test.go:149`, and the fixer already updated it. Every other `PID: 4242` record test either leaves the lock free or does not assert the held wording; the tier-1 pass confirms this. A search of `doc.go`, `PATTERN.md`, `pattern/` and `docs/` found no text that states the old wording.
- **F3:** Only `readRecordedAnchor` reads the own marker. The other `ReadFile` of the anchor marker, `clone.go:352`, reads the prime's marker for the clone path. It is not the own-marker fallback the finding describes, so I left it alone. The doc comment on `readRecordedAnchor` was updated to match.

Findings left unchecked: none.

Calls made:
1. Grep for the callers and the held-lock wording across `internal` and the docs: to enumerate the members of F1 and F2's sets, which my context lacked.
2. Grep of `doc.go`, `lyxcwd`, `PATTERN.md`, `pattern`, `docs` and non-test marker reads: to find stale prose and other own-marker readers for F2 and F3.
