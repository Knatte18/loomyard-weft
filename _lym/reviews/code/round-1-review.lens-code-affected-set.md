```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

**Per change: the set found, and whether it is covered**

1. **`reedengine.InjectBufferPrefix` exported, `injectBufferName` builds on it** (`internal/reedengine/io.go:160`).
   - Readers: `tmuxrec.injectBufferPattern` (`tmuxrec.go`) uses the constant. Covered.
   - Tests: `internal/reedengine/io_test.go:240` and `paste_integration_test.go:107` assert the `"lym-inject-"` literal. That is an expected value taken from a literal, so it is correct and not a finding.
   - No doc or stencil names the prefix.
2. **`tmuxrec.Call.Stdin`, stdin matching, `{inject-buffer}` placeholding and the two-buffer refusal** (`tmuxrec.go`, `record.go`).
   - The package doc is updated. `docs/overview.md:2113` describes tmuxrec only at a level that stays true.
   - Old recordings carry no `stdin`, and their calls pass none, so `omitempty` keeps them matching. The tier0 replay of every scenario covers this.
   - The recorder side (`record.go:57`) is updated, and so is the recorded command in `recordings_realtmux_test.go:28`. Covered.
3. **`sessionSpec` moved to the exported `tmuxrec.SessionSpec`** (`session.go`).
   - Callers are `scenarios_test.go` (updated) and the new loomcli test. `landingshed.sessionSpec` in `prsession.go:144` is a different symbol. Covered.
4. **New `IdleSession`, `replaySession` split out, and `newSession`'s shell fallback** (`session.go`).
   - Callers: `envkit.replayedSessions`. `PlaySession` callers (`landingshed/prreview_session_test.go:96`, the new loomcli test, tmuxrec's own tests) still skip off Linux before `newSession` runs.
   - `RecordSession` also inherits the fallback. It is reached only by the tier2 real-tmux test, where the shell exists.
   - Build tags: `reedengine` has a Windows template, so envkit still compiles on Windows. Covered.
5. **`envkit.LandingDeps(dir)` becomes `LandingDeps(t, dir)`, and the session seams move onto a replayed reed** (`envkit.go`).
   - Callers: `loomcli/rownotice_test.go:190`, `loomcli/wiring_test.go:507`, `loomrecipe/fixture_test.go:536`, the new test, and `FullEnv`. All are updated.
   - `FullEnv` callers in `shedbuild`, `battenrecipe`, `loomrecipe`, `shedrecipe/multillm_fixture_test.go` and `envkit_test.go` need no edit.
   - `NilSeams` still finds the seams non-nil. No test outside landingshed or loomshed drives `Sampler.Sessions` or `Landing.PullRequest.Sessions`.
   - The inventory rows `loom-landing.md:199-200` still name `noopSamplerSessions` in `envkit.go`. See finding 3.
6. **`landingshed` loses `waitWorld.mails`, and `prReviewTestDeps.MailAgent` now fails the test** (`prreview_test.go`, `prreview_wait_test.go`).
   - Every `newWaitWorld` row now inherits the failing seam. No remaining row expects a mail.
   - The inventory row `loom-landing.md:103` lists the `MailAgent` closures in `prreview_wait_test.go` with verdict `real`. See finding 3.
7. **New tier1 `loomcli/prreview_mail_integration_test.go`.**
   - Rules that apply: PATTERN-test-stand-ins ("No change adds a stand-in this rule forbids") and the inventory verdicts for `prkit` and `creelfake.Delivery`. Not covered; see findings 1 and 2.
   - PATTERN-test-speed: tier1 with no tmux spawn, and the missing `t.Parallel` is explained by `Setenv`. Covered.
8. **Commit messages and departures.**
   - No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer is on any commit of the range.
   - The commit message bodies match the diff.
   - The landingshed assertions removed with no `Loosened-Check:` trailer belong to the tests lens.
9. **Declared PATTERN changes.** The directive declares none, so there is nothing to land.

**Findings**

1. **BLOCKING — design.** `internal/loomcli/prreview_mail_integration_test.go:352-354` (`prReviewMailDeps`)
   - The new test adds a `prkit.API` stand-in for `pullrequest.API`, a repository module, scripted with a hand-written open status.
   - The inventory gives `prkit` the verdict `goes`: callers drive `pullrequest.Client` over recorded GitHub answers.
   - PATTERN-test-stand-ins says no change adds a stand-in the rule forbids.
   - Evidence: `docs/research/no-fake-other-modules/loom-landing.md:187`, `docs/research/no-fake-other-modules.md` kit table (`prkit` row), `pattern/PATTERN-test-stand-ins.md` "Standing fakes".
2. **BLOCKING — design.** `internal/loomcli/prreview_mail_integration_test.go:228` and lines 338-351
   - Through `startNoticeOrch`, the new test runs creel over `creelfake.Delivery`. The inventory gives that stand-in for `reedengine`'s strands and panes the verdict `real`: a delivery over reed and a `tmuxkit` server, at tier2.
   - The strand list that creel's "refusal of an unknown strand" reads comes from that fake (`delivery.SetStrands`). So the halt row's refusal rests on a hand-written strand table, not a real reed.
   - The test also hand-writes other modules' output:
     - `Mode` returns `"review"`, where production reads beam.yaml through `pullRequestModeSeam`.
     - `Origin` returns a fixed GitHub origin, where production reads fabric.
     - `TaskHead` returns `"head"`, where production reads fabric.
   - The test already holds a real pair (`pair`) whose `pullRequestDeps` seams it could use.
   - Evidence: `loom-landing.md:33`, `notifystamp_integration_test.go:58-59`, `landingdeps.go` (`Mode: pullRequestModeSeam(l)`, the fabric-backed `Origin`).
3. **LOW — consistency.** `docs/research/no-fake-other-modules/loom-landing.md:103,200`
   - The stand-ins inventory, which PATTERN-test-stand-ins names as the list of standing fakes, still lists the removed `MailAgent` closures in `prreview_wait_test.go` and `noopSamplerSessions` in `envkit.go`.
   - The diff does not update either row.
   - It is weak: the document says it was read from a fixed tree (`248543129`), so it may count as a dated snapshot.
   - Evidence: `loom-landing.md:103`, `loom-landing.md:200`, `envkit.go` diff (`noopSamplerSessions` removed).

Not a finding, outside my lens: `internal/shedrecipe/fixture_test.go:38-45` keeps its own `noopSamplerSessions`. It is a local builder outside envkit, which is the board entry's scope.

**Tool calls**

1. Searched for files referencing the changed symbols and the `lym-inject-` literal, to find callers and docs outside the manifest.
2. Grepped the inventory and pattern docs for the envkit, `MailAgent` and tmuxrec rows, to check the rule documents describing the changed stand-ins.
3. Grepped the remaining reference files (`overview.md`, `hermeticenv_test.go`, the shedrecipe fixture, `webster-shed.md`, `prsession.go`, the realtmux test), to see whether each reference holds after the change.
4. Read `docs/overview.md` around its tmuxrec passage, to check that the doc still describes the kit.
5. Checked build tags on `reedengine`, `tmuxrec` and `runproc`, to confirm envkit's new imports build on every platform.
