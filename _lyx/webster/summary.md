# webster-replan-state: six batches landed

All six batches reported `status: done`; the run's HEAD is `608749191`.

## Batch 01, gitexec-run-stdin (cards 1-6)

- `gitexec.RunStdin` is a checked git form that feeds git its stdin.
- `commitsNamedBy` classifies every disambiguated object in one `cat-file --batch-check` run.
- Rebaseline refusals are keyed on the caller's step and name the empty-frame cause.
- The Webster row rebaselines a changed plan on entry behind a run option, and the shed adapter sets the option and maps its errors to Stuck.
- `EditedDoneCards` reports which done cards were edited since their batch began.
- Deviations: `internal/websterengine/summary.go` and `internal/websterengine/autorebaseline.go`, new files holding the card 4 additions.

## Batch 07, plan-gate-done-card-edited (cards 7-12)

- The plan gate refuses an edit to a done card.
- The Plan-Review rubric gets webster's run-record note and the `webster_record` marker with its plumbing.
- One chain splitter serves the two verify checks, splitting at `||` and `|`.
- A card prescribing a PATTERN entry line over the cap now fails.
- Done-check refusals open with the batch and its cards.
- Deviations: `internal/testkit/envkit/envkit.go`, `internal/loomshed/publishfailure.go` and `internal/planparser/patternentry.go`.
- Card 9 put its seed and judge assertions into an existing `shedadapters` test, because that package sits at its test budget.

## Batch 13, fabric-reader-verbs (cards 13-17)

- `fabricengine.FabricReaderVerbs()` declares the reader verb names once.
- `IsReadOnlyCommand` is an allow-list for plain-word `lyx` help, fabric readers and git reads.
- A read-only fabric reference is a policy warning, through an injected classifier.
- `recover-batch` accepts all-read-only fabric references in line, with `audit_not_acceptable` on the envelope and a Master stencil rung.
- Gate and verify runs drop `LYX_STRAND_NAME` from their tests' environment.
- Deviations: `auditledger.go`, `runlevel.go` and `recordbatch.go` in `internal/websterengine`, `internal/loomcli/wiring.go`, `cmd/lyx/testdata/test-budget.yaml`, and the `doc.go` files of `gatecli` and `verifytree`.
- The fork appended one test with a shell heredoc instead of Edit.

## Batch 18, fabric-sandboxed-role-guard (cards 18-20)

- `lyx fabric` refuses its mutating verbs from a sandboxed slugged role.
- The implementer stencil states the sandbox rule.
- Every refused fabric verb's way forward names the orch, with matching `refusal-spec.md` rows.
- Deviation: the new file `internal/fabriccli/roleguard.go`.

## Batch 21, turn-end-counts-with-nothing-outstanding (cards 21-24)

- Webster's turn-end reading drops the shell expiry.
- Shuttle waits on both shell signals alike and records every shell a run ends over in `Result.EndedShells`.
- The round gate's Publish failure re-run names the top-level test only.
- Recovery spawns are counted, a dead recovery that committed is retried, and the cap is two.
- Deviations: `internal/shuttleengine/run_resume_test.go`, `config.go`, `spec.go` and `waitmark.go`, and `internal/loomshed/doc.go`.

## Batch 25, recovery-retry-on-the-verbs (cards 25-26)

- `recover-batch` and `status` carry the recovery count and retry, `recover-batch` refuses with `recovery_exhausted`, and `reset --to batch-start` clears the count.
- The Master stencil has the split dead rungs and the exhausted rung.
- Card 26 could not reproduce a failure of `TestLoomResumeReconcilesTheHubConfigBeforeArming`, alone or under CPU load, so it changed only the test's doc comment.
- Deviations: `internal/websterengine/doc.go` and `contracts/specs/refusal-spec.md`.

## Verify gate fixes

The gate's first attempt failed on five tests: `battencli.TestWire_EverySeamFilled`, `lyxcwd.TestEnforcement/tree-scan` and three `envkit` seam tests.
The fixer fork's single commit, `8898683ad` (fix), did three things:
- `shedfake.WebsterSeams` now sets `WebsterDeps.ReadOnly` to a stub that accepts nothing, which fixes the three `envkit` tests.
- `battencli` lists `WebsterDeps.ReadOnly` in `intentionallyNil`, since batten runs no webster.
- `internal/fabricengine/refscanner.go` joined `enforcementAllowlist`, because it names `--show-toplevel` as an allowed flag of the read-only `git rev-parse` form and resolves no root.

The fork reports that `battencli`, `lyxcwd`, `testkit/envkit` and `testkit/shedfake` pass under `lyx gate test --tags integration`.

## Audit

Every `record-batch` returned only informational scope-outside-plan warnings.


## Background shells waited out

Master's turn end was counted while these background shells were still running, which comes after `background_shell_wait_min` for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist; they may still be running in the session.

- `until grep -q "worktree\|FAIL" /tmp/claude-1000/-home-knatte-Code-loomyard-LYXHUB-webster-replan-state/4b72a718-bcaa-4396-ac52-cd53831c7acf/tasks/bkq8c78ik.output; do sleep 3; done; grep -v "level=WARN" /tmp/claude-1000/-home-knatte-Code-loomyard-LYXHUB-webster-replan-state/4b72a718-bcaa-4396-ac52-cd53831c7acf/tasks/bkq8c78ik.output | head -40`
- `git add internal/loomcli && git commit -q --amend --no-edit && git rev-parse --short HEAD; lyx gate test --tags integration ./internal/webstercli ./contracts/stencils ./internal/loomcli 2>&1 | tail -15`
- `lyx gate test --tags integration ./internal/battencli ./internal/lyxcwd ./internal/testkit/envkit ./internal/testkit/shedfake 2>&1 | tail -20`
