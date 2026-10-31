```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour, the test that pins it, or none.

**Per-behaviour list**

- `Config.CreditBlock` (heading, LoomYard item, engines in name order, each once, blank/missing/empty-heading/empty-LoomYard items, credit off, no item): pinned by `TestConfig_CreditBlock` (`config_test.go`, 9 rows).
- `bodyWithCredit` joining body and block with a blank line: pinned for the non-empty block by `wantPRBody` in the PR-Review tests (`prreview_test.go`, `prreview_wait_test.go`) and by `finalize_pr_test.go` (`wantGitHubMerge`). The commit-message join is pinned by `finalize_integration_test.go` (`wantMsg`) and `TestFinalize_MergeOptionsCarriesComposedMessage`.
- `bodyWithCredit` with an empty block, or an empty body: none, see F1.
- `Deps.creditBlock` read-failure path at Finalize: pinned by `finalize_test.go:340`.
- `Deps.creditBlock` nil `EnginesRun`: pinned by the `NilEnginesRun` row in `finalize_test.go`.
- PR-Review Create body and Edit body carry the block: pinned by `wantCreateCall` and `wantEditSeven` in `TestPRReview_Call`, and by the `Create` and `Edit` calls in `TestPRReview_Wait`.
- PR-Review reading `EnginesRun`, with a read failure returned as an error: none, see F2. No test row in the grep sets `EnginesRun` to an error on `prReviewTestDeps`.
- PR-Review with credit off or an empty block (body is the description body alone): none, see F1.
- No `Lym-Task` or `Co-Authored-By` in the landing message: pinned by the absent-string loop in `TestFinalize_MergeOptionsCarriesComposedMessage`, and by the exact `wantMsg` in `finalize_integration_test.go`.
- `LandingMessage` with the credit block (blank line before it): pinned by the integration `wantMsg` and the `want := summary.LandingMessage(...)` comparison. The `TestLandingMessage` rows in `description_test.go:83-85` still use the old trailer text but exercise the same code path, so they are not a gap.
- `LandingMessage` with an empty block ending in the body: not pinned at Finalize with the whole `Credit` off. The empty-block row of `CreditBlock` and the `LandingMessage` empty-string row cover the two halves, so I raise no finding.
- Config template defaults (`credit` true, heading, LoomYard text, claude entry): pinned by `TestLoadConfig_TemplateDefaults`. It also pins that the template holds no retired key or `codex` entry, which covers D6 only indirectly via the exact `CreditEngines` map.
- `credit_engines` key naming no engine fails the load, naming the key: pinned by the `credit_engines.claud` row in `TestLoadConfig_InvalidKeys`.
- Retired `co_authored_by`, `trailer_text` and `engine_trailers` keys load: pinned by `TestLoadConfig_TemplateDefaults`, which also checks `CreditEngines` stays the template's.
- Migration carries no value into `credit_engines` and is unchanged on a second pass: pinned by `TestMigrateConfig` (the retired-keys row, plus the `wantLines == 0` unchanged check and the `creditEnginesKey` absence check).
- `credit_engines` is an open map (`ConfigOpenMaps`) and the four keys are cross-pair (`ConfigSpec`): the credit key categories have no direct test in the grep. I did not confirm one, so I note it without a finding. A missing category would fail the strict load.
- `credit_heading` override and `credit_engines: {}` loading: pinned by `TestLoadConfig_OptionalStringKeys`.
- `ValidateDescription` finding detail text change: no assertion on the message text in the grep (`description_test.go:29` checks the check name only). The message text is not behaviour worth a row.

**Findings**

F1. MEDIUM, design (test economy, uncovered behaviour).
- Location: `internal/landingshed/finalize_pr.go` (`bodyWithCredit`), tests in `prreview_test.go`.
- Summary: the empty-credit branches of `bodyWithCredit` have no row. These are credit off or no item line, leaving the body alone; and an empty body, leaving the block alone. D4 requires the PR body to equal the description body when the block is empty. Every PR-Review and merge-message test uses `Credit` true with a non-empty block, so breaking `if credit == ""` would pass.
- Evidence: `prReviewTestDeps` at `prreview_test.go:435-445` fixes `Credit: true`; `finalize_pr_test.go:32` uses `wantCredit` only.
- Fix: add a table row or case with credit off, asserting the Create or Edit body equals the trimmed description body with no trailing blank line.

F2. MEDIUM, design (uncovered behaviour).
- Location: `internal/landingshed/prreview.go:227` (the `creditBlock` read in `openPullRequest`).
- Summary: D4 says a PR-Review `EnginesRun` read failure is an error. No PR-Review test makes `EnginesRun` fail. Only Finalize has such a row, at `finalize_test.go:340`.
- Fix: add a row to `TestPRReview_Call` where `deps.EnginesRun` returns an error, and assert the call returns the error with no Create or Edit.

**Tool calls**
1. `grep` for `EnginesRun`, `bodyWithCredit`, `creditBlock` and `read the engines` in `internal/landingshed/*_test.go`. My context lacked the full test files, only the diff.
