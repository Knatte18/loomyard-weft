```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set searches I ran:
- **F5, other docs calling revert or rebase fabric's replay:** every occurrence now agrees with the fixed wording (`doc.go:1352`, `mergestateactive.go:13`, `recordsduringmerge.go:144-145`). The test comment at `internal/loomcli/wiring_commitstatus_integration_test.go:396` says only "records cherry-pick", which is accurate.
- **F3/F4, writers and docs for the layout cache:** the callers of `forgetLayouts` are `anchormarker.go:85`, `anchormarker.go:211` and `clone_onerepository.go:293`. The narrowed wording ("adds or retires a marker") covers the clone caller too, since a clone creates a marker. The only prose that states the invalidation contract is the `forgetLayouts` doc, the `layoutAnswers` doc and `doc.go:2493`, and the fix updated all three.

Tool calls:
1. `grep` for revert/rebase/replay wording across `*.go` and `*.md`: to find sibling sites for F5, which my context lacked.
2. `grep` for `forgetLayouts` and the kept-answer wording: to find every caller and doc site for F3/F4, which my context lacked.
3. `Read` of `wiring_commitstatus_integration_test.go:392-401`: to check the one test comment that mentions fabric's replay.
