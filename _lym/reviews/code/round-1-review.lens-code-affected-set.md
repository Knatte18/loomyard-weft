```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I traced what it forces outside the lines it touches.

**Affected set per change**
- **`summaryparser.ValidateDescription` signature (new `forbidden` argument):** all production callers pass patterns: beamcli `deps.go` and `dobbydeps.go`, landingshed `beamdescribe.go` and `describe.go`, loomcli `validate.go`, shedrecipe `entries_gate.go`. Tests pass `nil`. `docs/overview.md:1061` ("stdlib-only") still holds, since `regexp` and `os` are stdlib. Covered.
- **`NewDescriptionGate`, `RecordBeamDescription`, `ReplaceBeamDescription` signatures:** all callers and tests updated, `parity_test.go` included. Covered.
- **New `Env.DescriptionTerms` seam:** loom `wire` and `wireLightweight`, envkit `FullEnv`, and battencli's `intentionallyNil` (trailer present). Covered.
  - The `wireLightweight` rationale comment (`wiring.go:367`, "It loads no module config") is left stale. Finding 5.
- **New landing keys (`describe_review`, `describe_review_timeout_min`, `describe_forbidden_terms`):** these places are updated:
  - `template.yaml`, `ConfigSpec` categories, `LoadConfig` validation;
  - `configreg_test` LocalKeys;
  - `docs/overview.md:1434-1439`;
  - the final-summary spec.
  - Messages naming the keys: `validate.go:126` hard-codes the full name, and `describereview.go:81` uses the short name. Findings 1 and 2.
  - The hub's own `_bolt/_lym/config/landing.yaml` sets none of them. The change description already names this as a check by hand, so I raise no finding.
- **New loom refusal in `validate-description` (forbidden terms unreadable):** no loomcli test reaches it (grep found none), and it carries no way forward of its own. Finding 3.
- **New `## Forbidden terms` section in `loom-final-summary-spec.md`:** it was inserted before the Merge danger section's closing sentence, which moved that sentence. Finding 4.
- **PR-Review sync (`prText.matches`, `syncText`, `wait` signature, `pullrequest.Status` Title/Body):**
  - callers in `prreview.go`;
  - tests in `prreview_test.go`, `prreview_wait_test.go`, `client_test.go`;
  - docs in `pullrequest/doc.go`, `landingshed/doc.go`, the PRReview type comment and spec lines 86-90.
  - Covered.
- **Finalize `setRecordedText`:** the `mergeOnGitHub` doc, the `finalize_pr_test` rows (extra `Status 7`, beam rows keep no edit), spec line 90 and `landingshed/doc.go`. Covered.
- **`MultiLLMProducer` timeout now Stuck:**
  - `shedadapters/doc.go:33` and the type and `mapOutcome` comments are updated.
  - `runAndCommit` skips the commit on an empty Path, so it is consistent.
  - `driftengine` expect and verify pass the outcome through.
  - `shedadapters/doc.go:147` is about a different producer.
  - Covered.
- **`DescribeTable` (second seat, `ReviewDir`, table gate):**
  - `requireDescribeLead` test;
  - shuttleparity catalog (`describeReviewSeat` holder plus `ReviewDir`);
  - loom wiring `ReviewDir`;
  - `loomengine.DescribeReviewDir`, which goes through `lymdirs`, per PATTERN-lym-files;
  - the stencil registered in `stencils.go`;
  - the `describeEntry` comment updated.
  - Covered.
- **Describe and darn writer stencils:** the existing rules are kept and the new rule added. No rule the entry left alone is dropped. Covered.
- **Commit trailers:** both `Loosened-Check` causes are observations from the run. The configreg one adds expected keys, which is no loosening. No `Pattern-Departure` trailers. No declared PATTERN changes. Covered.

**Findings**

1. **BLOCKING · consistency** · `internal/loomcli/validate.go:126`
   - The refusal text hard-codes `"loom: read landing.describe_forbidden_terms: "`.
   - PATTERN-config-key-names requires a message to name a key rendered by configengine's one key-name helper (`configengine.KeyName`, `config.go:106`) from named constants.
   - Evidence: `validate.go:126`; the constant `forbiddenTermsKey` is at `keynames.go:20`.

2. **BLOCKING · consistency** · `internal/landingshed/describereview.go:81`
   - The new error `fmt.Errorf("%s model-spec: %w", describeReviewKey, err)` names the key short (`describe_review`), not in full (`landing.describe_review`).
   - PATTERN-config-key-names requires the full name, via `keyName(describeReviewKey)`, which the same file already uses at line 76.
   - `fixer.go:167` follows the same old pattern, but this line is new.

3. **BLOCKING · consistency** · `internal/loomcli/validate.go:123-128`
   - The new refusal in a loom module, reachable through `lym loom validate-description`, has no test that reaches it and asserts its message.
   - It also has no trailing `way forward:` clause of its own; it relies on whatever the wrapped `LoadConfig` error carries, and a modelspec parse error carries none.
   - PATTERN-refusal-way-forward requires both.
   - Evidence: grep of `internal/loomcli/*_test.go` for `DescriptionTerms` and `describe_forbidden_terms` returned nothing.

4. **LOW · consistency** · `contracts/specs/loom-final-summary-spec.md:49-56`
   - The new `## Forbidden terms` section was inserted before the sentence "`Publish` and `Finalize` do not check it: they carry the body as it stands."
   - That sentence closed the Merge danger section and now sits under Forbidden terms. The Merge danger section loses it, and "it" now reads as the term check.
   - Evidence: the spec as the diff leaves it, lines 47-56.

5. **NIT · consistency** · `internal/loomcli/wiring.go:367`
   - The `wireLightweight` comment still says "It loads no module config".
   - The new `DescriptionTerms` closure (`wiring.go:423`) loads `landing.yaml` strictly on this path. That is the kind of config read the comment gives as the reason the lightweight path exists ("A sibling process may be rewriting a config").
   - Only line 381 was updated.

**Tool calls**
1. `sed`/`grep` of the spec section, `docs/overview.md:1458-1468` and docs/specs for stale wording: needed the spec layout around the insert and doc mentions outside the diff.
2. `sed` of the saved oversized grep output: the first call's output was truncated.
3. `grep` for model-spec error formats, the configengine `KeyName` helper, tests that reach the new seam, and assistant mentions in the overview: needed for the key-name and refusal-test checks.
4. `sed` of the `validate-description` help text and `grep` of docs naming the verb: needed to see whether the verb's docs needed the new behaviour.
5. `sed` of `wiring.go:366-384` and `docs/overview.md:1058-1066`: needed the `wireLightweight` rationale outside the diff hunk.
