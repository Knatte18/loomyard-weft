---
format: 5
approved: false
root: internal/websterengine
language: go
---

# Plan: Webster verify triage — rerun, compare with baseline, halt only on a real regression

A red plan-level `## verify:` at the end of a webster run halts the run today, even when the failure is flaky or was already red before the plan started.
This plan adds a mechanical triage step to `runIntegrationStage`: rerun the verify once at head, compare the remaining failures against the plan's starting commit, and classify flaky, pre-existing, or regression.
Only a regression demotes the run to `stuck`;
flaky and pre-existing failures are recorded in the integration report, `RunResult` warnings, `summary.md`, and a friction note, and the run continues.
The integration fork captures its output to a scratch log so Go can name every failing test and its output tail, and bisect localizes a regression by the regressing identities rather than by the whole command's exit code.

## Card Index

1 — integration-report-type — integration report parse type with optional failures and triage fields
2 — verify-output-parser — parse go test output into test, package, and opaque failure identities
3 — verify-capture — capturing verify runner seam and scratch log paths
4 — integration-fork-log — integration fork redirects verify output to a told log path
5 — triage-core — rerun and baseline classification behind a fake-runner seam
6 — bisect-by-identity — bisect pass predicate over the regressing identities
7 — summary-triage-sections — summary.md regression tails and triage section
8 — triage-recording — warnings, stuck reason, and friction note helpers
9 — stage-wiring — run triage in the integration stage and demote done on regression
10 — stencil-verdict-handoff — Master ends done over a FAILED report; Describe carries the triage section
11 — triage-docs — package doc, overview row, and webster spec describe triage

## Shared Decisions

### Decision: runner-seam

- **Decision:** triage and bisect take a `verifyRunner` function value; production passes `runVerifyCapture`, untagged tests pass a fake keyed by the fake bisector's checkout.
- **Rationale:** the Test Tier Purity Invariant bars `exec.Command` from untagged tests, and the classification and localization logic must be unit-tested untagged.
- **Applies to:** cards 3, 5, 6, 9

### Decision: identity-membership

- **Decision:** an identity "fails at a SHA" only when its `id` appears in that SHA's `parseVerifyFailures` set.
  Triage treats non-membership at baseline as a regression (it never excuses an unproven failure);
  bisect treats non-membership at a card SHA as passing (it only localizes).
- **Rationale:** D7 and D9 share one membership test and map a non-member to opposite results by design.
- **Applies to:** cards 5, 6

## verify:

go build ./... && go test ./internal/websterengine/... ./internal/webstercli/... ./internal/landingshed/... ./internal/lyxcwd/... ./contracts/... && go test -tags integration ./internal/websterengine/... ./internal/webstercli/... ./internal/landingshed/... ./internal/lyxcwd/... ./contracts/... && go vet -tags smoke ./internal/webstercli/...
