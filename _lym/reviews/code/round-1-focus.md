---
round: 1
focus:
  - "internal/gitrepo/testmain_test.go and push_test.go (D5): TestMain dispatches child bodies from a map; settle whether a child spawned with `-test.run=^$` still reaches TestMain's dispatch before m.Run, whether a body's exit code and stdout reach the parent as TestPushCoalesced's cross-process and crashed-holder steps expect, and whether `init` registration is safe when a child variable is inherited by an unrelated nested test process."
  - "internal/fabricengine/livestate_matrix_test.go (D4): settle whether the omitted-cell tally still fails when ran or omitted disagrees with the Verbs, States and Omissions tables, and whether any comment or doc.go still describes omitted cells as skipped."
  - "Every file moved behind a _windows, _linux or `&& !windows` constraint (D1): settle for each that the tier tag is kept, exactly one tier tag plus a platform term is present, test names are unchanged, no helper the remaining files need was left behind or duplicated, and nothing is left uncompilable on the other OS (unused imports, a helper defined only in the moved file but used by the stay-behind file)."
  - "Moved tests that keep a combined check: settle that only the OS half of the skip went and the prerequisite half (root, missing binary) stays, and that no moved test lost an assertion or setup step in the move."
  - "OS-bound table rows and subtests filtered before t.Run (D2), e.g. internal/configcli/configcli_integration_test.go, internal/orchcli/rules_test.go, internal/lymcwd/lymcwd_test.go, internal/pairteardown/archive_test.go, internal/websterengine/rebaseline_test.go: settle that the filter drops the same rows the old skip skipped on the other OS and no row that ran before is now dropped."
  - "Remaining t.Skip calls near a runtime.GOOS check (D1/D2 acceptance: none outside the out-of-scope list): settle whether any test still skips on a GOOS or GOARCH check alone, including cmd/lym/crosscompile_test.go's host check, against the scope's out-of-scope list."
  - "Deletions (D3, D6): settle that TestBuildLoomShed_OutputShape and the always-skipped verify-gate subtest are gone with no orphaned helper or import, and that the board note the decision record promises for the verify-gate coverage exists in the diff or is claimed by the change description."
  - "PATTERN-test-speed (pattern/PATTERN-test-speed.md) and the tier purity allowlist in cmd/lym: settle that the new rule text matches the constraints and the allowlist follows the moved socket file test; check the repository's docs-in-same-commit rule for any doc.go or docs that name a moved or deleted test."
  - "Commit messages and landing summary: settle that each claim in the two commit bodies and any `_lym/landing/summary.md` is borne out by the diff, and that the follow-up comment-break commits change only comment line breaks and no code."
---

Focus is where to look; severity and verdicts are the reviewer's.
The Windows-only files cannot be built on this Linux run, so a reviewer reading them by eye (imports, helpers, build lines) is the only check this round has.
