```yaml
lens: fix-affected-set
outcome: reported
```

Rechecked the sets behind F1, F2, F3, F9 and F11 to see whether each fix covers every member.

Unchecked: F4, F5, F6, F7, F8 and F10. None of them rests on a set.

**MEDIUM: card 15's exempt set misses the friction directory**
- Changed text: card 15, the sentence "The exempt paths are those loom itself writes on the weft during a row: `judgement.yaml` and everything under `RunRecordsDir`".
- Summary: loom also commits `_lym/loom/friction/` on the weft, which sits outside the run's shed directory. A resumed judge row can therefore still be stopped by loom's own commits, which is F1's premise surviving the fix.
- Evidence: `docs/overview.md` (drift paragraph area, ~line 610) says `commit-records` "commits and pushes the run's records … the status file, the review round record, friction notes and drive reports", and "Every `lym loom start` or `step` commits whatever the friction directory holds along with the seed". It places friction notes "under `_lym/loom/friction/` and drive reports under `_lym/shed/&lt;slug&gt;/drive-reports/`". Drive reports fall under `RunRecordsDir`; friction notes do not. So when a `lym loom start` resume or a driver's end-of-session `commit-records` commits a friction note after the snapshot, `WeftChangedSince` names a path outside the exempt set, and `Changes` reports a judge edit on that resume.
- Fix direction: add loom's friction directory, anchor-joined like `RunRecordsDir`, to the exempt set, with a `TestJudgeSnapshot` row for it.

**F2 (verifyFixture helpers): fully covered.**
- The three helpers are called only at `verify_test.go:226`, `:246`, `:775`, `:849`, `:871` and `:966`. All of these sit inside the helpers themselves or inside `TestDriftVerify` rows, which card 16 rewrites.
- Leaving the helpers until card 25 only leaves unused methods behind, which Go compiles.

**F3 (renamed and retired config fields): fully covered.**
- Go readers of the renamed fields:
  - `config_test.go:224` and `:265`, covered by `templateConfig` and `TestLoadConfig`.
  - `seats_test.go:53`, `:142` and `:277`, covered by `TestSeatTables` in cards 4 and 26.
  - `verify_test.go:175` and `:869`, covered by `newVerifyFixture` and `TestDriftVerify`.
- String readers:
  - `configreg_test.go:80` is in `TestModules_EveryTemplateKeyClassified`, which card 4 lists.
  - `configcli_test.go:989` is in `TestDispatchHubWideBoard`, which card 26 lists.
- `driftcli/cli.go:268` (`takeRun`) is deleted by card 22.
- The driftcli fixtures that seed `verify_max_runs` load with the key dropped once card 26 retires it, so they don't break.

**F9 (docs that state the drift row order and verbs): covered.**
- `internal/reportengine/doc.go` names Drift-Recheck only as a phase member, and card 13 already edits it.
- `contracts/stencils/drift/drift-template-baseline.md` is retired in card 24.
- The `docs/research/*` files are measurement reports and stay as written.

**F11 (`CountedCapture`): covered.** Its callers are `verify.go:170`, rewritten by card 16, and `verbs.go:258`, deleted by card 25. Its other references, `TestCountedCapture` and the `doc.go:28` line, are both removed by card 25.

Tool calls:
1. Grepped the renamed drift config keys as strings across the repo. My context held only the Go field-name matches.
2. Grepped the callers of the three `verifyFixture` helpers. My context lacked their call sites.
3. Grepped the docs and doc.go files that name the drift rows or removed verbs. My context covered only the two files the review named.
4. Read `internal/reportengine/doc.go`'s drift lines, plus a grep of cards 2x for the baseline stencil's retirement. Both files surfaced in call 3 and were not in my context.
