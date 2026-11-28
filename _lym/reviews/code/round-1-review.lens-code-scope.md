```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the Goal points of the decision record.

**Decisions** (Goal points of the board entry; the record has no `Added after Discussion` entries):
- **G1**: sweep every Go comment and package doc that names a config key to `<module>.<path>`.
- **G2**: a comment-only change, no behaviour or user-visible text.
- **G3** (imported issue #265, "What to do"): add a guard test that scans Go string literals for template key names written without their module.

**Per change:**
- **C1: Comment and package-doc sweep** across the production packages, diff pieces 002–060 and the `internal/configreg/doc.go` hunk. Traces to G1.
- **C2: New guard** `internal/configreg/keynames_enforcement_test.go`. Traces to G3.
- **C3: `lym config` help literal** in `internal/configcli/configcli.go:516`, rendered through `configengine.KeyName`. Forced by C2: it is the bare literal the guard reports, so C2 cannot pass without it. The issue's "Why it is abnormal" names help texts as the target. G2's "no user-visible text" governs the comment sweep, and the guard part of the entry licenses this edit.
- **C4: `PATTERN.md` tags `PATTERN-config-key-names` as `(test)`.** Forced by C2: the rule document goes stale once a guard enforces the entry. Whether the tag fits the guard's coverage belongs to the affected-set lens and the handler.

**Per decision:**
- **G1**: built by C1 for production files only. Test files are left (see F1).
- **G2**: C1 makes no behaviour change. C3 is the one user-visible text, forced by G3.
- **G3**: built by C2.

**Findings:**

F1
- **Severity:** MEDIUM
- **Class:** scope
- **Location:** `internal/battencli/childdriverkind_test.go:16`, also `internal/landingshed/prreview_test.go:179` and `:676`
- **Summary:** G1 asks for "every Go comment", but the sweep skipped `_test.go` files.
  `childdriverkind_test.go:16` still reads "shed.yaml's driver_kind".
  `prreview_test.go:179` and `:676` still read "beam.yaml's pull_request".
  The only test file in the range is the new guard (`git diff --stat 7cb0048..HEAD -- '*_test.go'`).
  The change description narrows the claim to "Production Go comments" without the entry licensing that narrowing.
- **Evidence:** the grep output at those lines; the decision record, `_lym/discussion/decision-record.md:7`; `_lym/landing/summary.md:17`.
- **Fix:** sweep the comments of the `_test.go` files the same way.

**Tool calls:**
1. Grepped comments in the three focus-named test files, and in `_test.go` files generally, for bare key forms, plus `git diff --stat` over test files: the manifest held no test-file comment contents.
