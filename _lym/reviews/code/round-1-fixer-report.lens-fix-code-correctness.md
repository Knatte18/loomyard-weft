```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F6 change set (`validateEffort`, `agentconfig.Efforts`, `checkEfforts`, and the added tests) by tracing inputs.

no findings

Traces:
- `validateEffort("turbo")`: it is not in `agentconfig.Efforts()`, so it returns an error containing "invalid effort". The existing refusal row still matches. `validateEffort("")` returns nil.
- `checkEfforts` on the template (`default: medium`, `fable: high`): both values are in `effortVocabulary`, so it passes. On `haiku: turbo`: it passes the empty check and fails the vocabulary check with `efforts.haiku`. That is a marked `ErrUnusable` wrapping `ErrInvalid`, so the new `UnknownEffort` row fails without the check.
- Other fixtures: the only `efforts:` fixtures in tests use `low`, `medium`, `high`, `xhigh` and `max`, so no other caller breaks. `validEfforts` had no remaining users after the change.
- F1 rows: without the `effortFor` call in `OpenLaunch`, `spec.Effort` stays empty and neither row finds `--effort`. With a broken `modelAlias`, `claude-fable-5-1` falls to `default` (`medium`), and the "high" row fails. Rows without `withAgents` still pass `shuttleengine.Config{}`, so "no model or effort adds neither flag" is unchanged.
- F2 rows: without `agentconfig.Load` in `LoadConfig`, the unusable row returns no error and the other rows get a zero `Config` whose `Skills` refuses darn. Every row fails, and the gate passed with the rows.
- F3 test: it fails if `wireCLI` takes the template's orch list instead of the file's. It does not cover `watch.go` passing `c.orchSkills` to `NewWatcher`; that is a one-line pass-through, and the review's fix text allowed a `wireCLI` test.
- F6: the restored line is byte-identical to the deleted one (`git show a1568d23` diff), and no test asserts the help text.

Outside my lens: `subagentEffortEnv`'s `validateEffort` call in `subagents.go` is now unreachable through `Load`. It is harmless and not made worse by the fix.

Tool calls: none; my context already held everything this lens needed.
