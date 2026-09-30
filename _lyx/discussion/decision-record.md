# Webster verify: rerun, compare with parent, halt only on a real regression

## Goal

A red plan-level `## verify:` at the end of a webster run halts the run only when the task itself broke a test.
A failure that goes green on one rerun is recorded as flaky and the run continues;
a failure that also fails at the task's baseline commit is recorded as pre-existing and the run continues with a warning.
The integration report names every failing test and carries the tail of its output, so neither the driver nor the hub has to read the fork transcript.

## Scope

In:

- Mechanical triage of a `FAILED` integration report inside `internal/websterengine`'s integration stage: parse failures, rerun once at head, compare against the baseline commit, classify.
- Integration fork stencil (`contracts/stencils/webster/webster-template-integration.md`): the fork captures the verify command's combined output to a told log path.
- Master stencil (`contracts/stencils/webster/webster-template-master.md`): a `FAILED` integration report no longer makes Master end `stuck`.
- Integration report (`_lyx/webster/reports/integration.yaml`): carries failures and the triage verdict.
- Bisect: on a regression, localizes the card by the regressing failures, not by the whole command's exit code.
- `summary.md` section, `RunResult` warnings, and a friction note recording flaky and pre-existing failures.
- Docs: `internal/websterengine` package doc, `docs/overview.md`'s webster row, and any module doc under `manifest/designs/` describing the integration stage.

Out:

- Per-batch or per-card verify (`**Verify:**` fields, batch fork reports) — unchanged.
- Replacing the integration fork with an in-process verify run.
- Retries beyond one rerun, quarantine lists, or any persistent flaky-test registry.
- Deflaking any specific test.

## Decisions

### D1 — Triage runs in Go, in the integration stage after Master's session ends

Decision: triage lives in `runIntegrationStage`'s unleased middle phase, before localization, running the verify command in-process the way bisect already does.
Master never runs triage and gains no new verb.
Rationale: the stage already re-runs the whole verify command in-process with no Bash-tool timeout, restores the branch after detached checkouts, and holds no state lease across long work.
Triage is deterministic, so it belongs in Go rather than in an LLM's judgment.

### D2 — Master ends `done` on either integration status; the integration verdict is webster's

Decision: the Master stencil's `status: FAILED` branch changes from `outcome: stuck` to `outcome: done`, with `summary.md` noting that the integration suite reported FAILED and that webster triages it after the session.
`runIntegrationStage` then owns the verdict: flaky or pre-existing keeps `done`;
a regression demotes the `RunResult` to `outcome: stuck` with a `StuckReason` naming the regressing tests and the localized card, returned as a result, not an error.
The existing loud error for "done over a FAILED suite" is removed;
the loud error for "done with no integration report" stays.
Rationale: demoting `done` to `stuck` is safe because a `done` run already carries a validated `summary.md` and passed the run-exit audit;
promoting a Master `stuck` to `done` would need a `summary.md` Master was never required to write.
A regression reaches the Shed adapter as `Stuck` with a reason, the same channel the old Master-stuck path used.

### D3 — Triage also runs when Master ended `stuck` over a FAILED report

Decision: whenever every batch is terminal-done and the report is `FAILED`, triage runs regardless of Master's outcome.
A non-regression verdict never promotes a Master `stuck`: the outcome stays `stuck`, and the triage section and report fields are still written.
A regression under a Master `stuck` escalates as today.
Rationale: a resumed or stale Master session following the old stencil must not bypass triage, and Master's own `stuck` may carry a reason webster cannot see.

### D4 — The fork captures output; Go parses it

Decision: the integration fork runs the verify command with its combined stdout and stderr redirected to a told log path, and reports `status` from the command's exit code exactly as today.
The log lives in webster's scratch dir under `.lyx`, never under `_lyx`.
Go parses the log; the fork never lists failures itself.
A missing or unreadable log is not an error: the first run's failure set is then unknown, and triage proceeds from the rerun.
Rationale: parsing is deterministic in Go and hallucination-free;
the full log is ephemeral diagnosis material under the Durable-vs-Ephemeral State Invariant, while the durable report carries only names and tails.

### D5 — Failure identities parsed from `go test` output

Decision: each failure gets one identity:

- **Test** — `<package>.<TopLevelTest>`, from `--- FAIL: <name>` lines attributed to the `FAIL\t<package>` line that closes their package block.
  A subtest (`TestA/case`) collapses to its top-level test.
- **Package** — `<package>`, for a `FAIL\t<package>` line with no test failures attributed to it (`[build failed]`, `[setup failed]`, a `TestMain` or init panic).
- **Opaque** — a single identity for the whole verify command, used when the command exited non-zero but no Go failure was parsed (a failing `go vet`, `go build`, or non-Go command).

Each failure carries an output tail: the indented lines following its `--- FAIL:` header for a test, and the last lines of the log for a package or opaque failure, capped at 40 lines per failure.
Parsing tolerates `\r\n` line endings.
Rationale: plan verify commands in this repo are `go test` invocations;
identities at top-level test granularity match what `go test -run` and a human rerun act on.

### D6 — Rerun the whole verify command once at head

Decision: step 1 reruns the plan's whole verify command once, in-process, at the unchanged head, capturing its output to a separate scratch log.
Green: verdict **flaky**; every identity from the first run is recorded as flaky (an opaque identity when the first log was missing or unparseable).
Red: the rerun's failure set is the head failure set;
identities that failed in the first run but not in the rerun are recorded as flaky alongside whatever step 2 decides.
Rationale: a whole-command rerun reuses the exact flags, build tags, and command chain of the verify, so it cannot report a false pass the way a reconstructed `go test -run` can when it drops a `-tags` flag and matches no test.
Plan-level verify commands are the plan's targeted suite, and bisect already pays whole-command runs.

### D7 — Compare against the baseline commit

Decision: step 2 checks out the baseline commit detached in the live worktree, runs the whole verify command once with captured output, and restores the branch afterwards, even on error, using the same `FabricBisector` surface and restore-failure handling bisect uses.
The baseline is the first parent of the first accumulated card SHA in execution order — the tree the plan started from.
Classification of each head identity:

- **Test** or **package** identity that also fails at baseline: pre-existing.
- Identity that passes, is absent, or never ran at baseline (an earlier `&&` link stopped the chain, or the test does not exist there): regression.
- **Opaque** identity: always regression, never pre-existing.

Verdict: **regression** if any head identity is a regression;
otherwise **pre-existing**.
Rationale: "fails at baseline" must be demonstrated before a failure is excused;
every ambiguous case falls to halting, which is today's behavior, so triage can only remove halts it can prove unwarranted.

### D8 — No bisector: rerun only

Decision: when `RunDeps.OpenBisector` is nil (no fabric repo), step 1 still runs.
A red rerun yields verdict regression with the existing "could not be localized" warning plus a warning that the baseline comparison was unavailable.
Rationale: without a repo there is no baseline to check out, and an unproven failure halts.

### D9 — Bisect localizes by the regressing identities

Decision: on a regression, bisect's pass predicate at each SHA becomes "none of the regressing identities fails", parsed from that SHA's captured output;
when the regressing set contains the opaque identity, the predicate is the whole command's exit code, as today.
The last-SHA-passes → `unknown` rule stays.
Rationale: with a pre-existing failure present, every SHA fails the whole command, so the exit-code predicate converges on card 1 and blames it by arithmetic.

### D10 — Integration report shape

Decision: the fork still writes `status`, `head_sha`, `deviations`.
On a `FAILED` report, Go adds, under the state-mutation lease in the stage's final phase:

```yaml
failures:            # the first run's failures; from the rerun when the first log was missing
  - id: <package>.<Test> | <package> | verify-command
    kind: test | package | opaque
    tail: |
      <up to 40 lines>
triage:
  verdict: flaky | pre-existing | regression
  rerun: passed | failed
  baseline_sha: <sha>       # omitted when step 2 did not run
  flaky: [<id>, ...]
  pre_existing: [<id>, ...]
  regressions: [<id>, ...]
```

The integration report gets its own parse type or optional fields so batch reports keep their strict shape.
Rationale: one file answers both "what failed" and "why the run did or did not halt", which is what the driver and the hub read.

### D11 — Recording flaky and pre-existing failures

Decision: a non-regression verdict produces:

- a `RunResult` warning per verdict naming the identities,
- an appended `summary.md` section (the PR text) listing flaky and pre-existing identities,
- a Go-written friction note via `friction.NotePath(deps.FrictionDir, "webster-verify-triage")` listing the same, only when `FrictionDir` is non-empty,
- a `logger.Warn` line.

A regression's `summary.md` section extends today's integration-failure section with the regressing identities and their tails.
No state record is written for a non-regression verdict; a regression writes today's reserved `-1` record.
Rationale: the Shed adapter's `Done` path forwards only the summary pointer, so `summary.md` is the channel that survives to the operator and the PR;
friction is where the hub collects environment trouble, and a pre-existing red on the baseline is exactly that.

## Constraints

- `CONSTRAINTS.md` is authoritative.
  Relevant here: Durable-vs-Ephemeral State (logs under `.lyx` scratch, report under `_lyx`), Stencil Ownership (stencils read from the told dir; the seeded default changes), Friction Leaf (websterengine imports `internal/friction`, never the reverse), Test Tier Purity (no `exec.Command` in untagged tests), Hermetic Git Test Environment, Comment Line-Break Convention, Documentation Lifecycle.
- The long triage work runs with no state-mutation lease held, matching the stage's existing three-phase split;
  report, summary, state, and friction writes happen under the lease in the final phase.
- The worktree is restored to its branch after every detached checkout, and a failed restore is surfaced, never dropped.
- The verify command still runs through the existing shell selection (`sh -c`, `cmd /C` on Windows).
- Stencil marker rules: the integration stencil's new log-path marker is required and filled like the existing required markers; no `{{if}}`/`{{range}}`.

## Auto-mode assumptions

- The baseline is the first parent of the first accumulated card SHA, not the parent branch's current head: the question is "did this task break it", and the parent branch may have moved.
- The rerun and the baseline run are whole-command runs, not reconstructed per-test `go test -run` invocations.
- An opaque failure is never classified pre-existing.
- Both flaky and pre-existing failures go to the friction note, not only flaky ones.
- The per-failure tail cap is 40 lines.
- Triage logic sits in `internal/websterengine`; the `go test` output parser is not split into its own package, since webster is its only consumer.

## Open risks

- A baseline whose `&&` chain stops on an earlier failing link never runs the later tests, so a pre-existing failure behind it is classified a regression and halts — today's behavior, not a new failure mode.
- A regression in a test that is also flaky can pass the single rerun and be recorded flaky instead of halting.
- Two extra whole-verify runs add wall-clock to every red integration stage.
- Unusual `go test` output (`-json`, custom test runners, interleaved `-p` output without `-v`) may parse only as package or opaque identities, which degrades toward halting rather than toward continuing.

## Acceptance criteria

- A plan whose integration verify fails once and passes on rerun ends `outcome: done`, with triage `verdict: flaky`, the failing identities and tails in `integration.yaml`, a flaky section in `summary.md`, a `RunResult` warning, and a friction note when `FrictionDir` is set (none when empty); bisect does not run and no `-1` state record exists.
- A failure red at head on rerun and red at the baseline for the same identity ends `outcome: done` with `verdict: pre-existing`, a pre-existing warning in `RunResult` and `summary.md`, and no bisect.
- A test that fails at head and passes at the baseline ends `outcome: stuck` with `verdict: regression`, a `StuckReason` naming the test and the localized card, the `-1` record, and `summary.md` naming the test with its tail.
- A test that fails at head and does not exist at the baseline is a regression.
- Mixed pre-existing and regressing failures yield `verdict: regression`, and the report lists both groups.
- An opaque failure red on rerun is a regression; an opaque failure green on rerun is flaky.
- With a pre-existing failure present at every card SHA, bisect localizes a regression to the card that introduced it, not to card 1.
- With no bisector, a red rerun yields `verdict: regression` with both "not localized" and "baseline unavailable" warnings; a green rerun yields flaky.
- A missing fork log does not error; failure identities come from the rerun.
- A Master `stuck` over a FAILED report still gets triage fields in the report and stays `stuck` on a non-regression verdict.
- A Master `done` with no integration report still fails loud.
- The worktree is back on its branch after the baseline run, including when the baseline verify leaves the tree dirty or errors.
- Parser unit tests (untagged) cover: top-level test failures across multiple packages, subtests collapsing to the top-level test, `[build failed]` and `[setup failed]` package failures, a panic with no `--- FAIL` line, `\r\n` input, tail capping, an all-`ok` log, and non-Go output yielding the opaque identity.
- Triage classification is unit-tested untagged behind a fake verify-runner seam; tests that spawn a real shell or git stay `integration`-tagged.
- The rendered integration fork prompt contains the log path and the redirect instruction; the rendered Master prompt's FAILED branch says `outcome: done`.
- Package doc, `docs/overview.md`'s webster row, and any `manifest/designs/` doc describing the integration stage describe triage in the same commit.

## Notes for the plan writer

- The integration stage is `runIntegrationStage` in `internal/websterengine/runlevel.go`; bisect, `runVerifyCommand`, and `LocalizeIntegrationFailure` are in `integration.go`; `AppendIntegrationFailure` is in `summary.go`.
- `runVerifyCommand` discards output today; triage and the new bisect predicate need a capturing variant.
- `ParseReport` decodes with `KnownFields(true)` and is shared with batch reports.
- Friction note paths come from `friction.NotePath`; see how `runlevel.go` derives the `webster-integration` and `webster-master` note paths.
- The Shed adapter (`internal/shedadapters/webster.go`) maps `stuck` to `Stuck` with `StuckReason` and `done` to the summary pointer; it should need no change.
