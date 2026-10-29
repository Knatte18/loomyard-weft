```yaml
lens: fix-proposal
outcome: reported
```

no findings

I tried three cases against the proposal:
- **The claim is true in both tests.** The first calls `logcapture.CaptureVerbose` (`spawn_test.go:556`). The second calls `logcapture.Capture` inside its subtests (`spawn_test.go:720`). Neither test could run in parallel, so the sentence "captures the process-global logger, so it does not run in parallel" states a real constraint.
- **The comment sits clear of the directive.** The new line goes above the `//` separator and the `//testtiming:keep` line. It does not enter the directive's text, so a scan that parses that directive is unaffected.
- **The wording matches the package.** Other serial tests use the same sentence: `windoworder_test.go:80`, `ensuresession_integration_test.go:151`. A scan keyed to that phrase would accept it.

I could not check the rule the review cites (`PATTERN-test-speed`, decision D3). My grep of `PATTERN.md` and `pattern/` for the serial-comment phrasing returned nothing, so I could not test the proposal against any wording that rule requires.

Tool calls:
1. `grep` over `PATTERN.md` and `pattern/` for the serial-comment rule. My context did not hold the rule's text, and the grep found none.
