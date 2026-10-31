```yaml
lens: code-affected-set
outcome: reported
```

List (code-affected-set), per change:

- **Envelope: new `message` field, `suite.tiers`, `suite.already_running`** — I found no other reader of the adopt envelope. A grep outside `internal/adoptcli` for `suite_warning` and `SuiteWarning` found nothing. The `already_running` matches in `beamcli`, `orchcli` and `shortcutcli` belong to other verbs. Covered: help text in `cli.go`, `doc.go` and `docs/overview.md:446`. Tests cover them in `adopt_test.go` and `deps_test.go`.
- **`checkDeps.Mark` seam** — the only constructors are `hubCheckDeps` (`check.go:302`, called at `:380`), the `newCheckFixture` fixture, and `check_integration_test.go:47`, which sets `Mark` at line 58. No other construction site exists, so there is no nil-`Mark` panic path. Covered.
- **`reedengine` new options `@lym_check*` and `SetCheckMark`** — documented in `bar.go`'s file header, `reedengine/doc.go` and `adoptcli/doc.go`. `PATTERN.md:53` (`PATTERN-wait-mark-display-only`) scopes itself to a pane waiting and to `@lym_wait`/`@lym_wait_start`. `SetCheckMark` marks a session, not a waiting pane, so the entry does not apply. The diff declares no PATTERN change and the rubric's directive lists none. Covered.
- **Status-bar format (`statusFormatSessions`, line 2)** — other readers of `status-format` and `@lym_prime_session` (`windowsize.go`, `ownwindow.go`, `follow.go`, `io.go`, `spawn.go`, and the non-test files in my first grep's hit list) are in `reedengine`. The only bar test the diff touched is `bar_integration_test.go`, and the diff changes no other existing test. I cannot say from the diff whether an existing test asserts the exact tail `align=right]%H:%M `, but a mismatch would fail the verify gate, which is outside this lens.
- **`suiteStart` shape (new fields)** — it is unexported and only `adoptcli` uses it. `reflect.DeepEqual` replaced `!=` in the two tests, since the struct now holds a slice. Covered.
- **Help step 8** — it now names the status bar and the `message` field. The `lym_check`/`checkMarkOf` text is documented in `adoptcli/doc.go`. Covered.
- **Commit messages and departures** — I found no `Pattern-Departure:` trailers, and none are needed.
- **Stencils and prompts** — no stencil mentions the adopt envelope, step 8 or the status bar. `PATTERN-stencil-ownership` is unaffected.

Findings: none.

Tool calls (2 used):
1. `grep` over `*.md`/`*.go`/`*.yaml`/`*.tmpl` for bar options, `SetWaitMark`, `suite_warning` and "adopt check", to find rule docs and other readers. My context lacked a repo-wide search.
2. `grep` of `PATTERN.md` for the wait-mark entry and the `@lym_*` options, and `grep` outside `adoptcli` for `suite_warning` and `already_running` readers. My context held `PATTERN.md` only through the prompt's entry list.
