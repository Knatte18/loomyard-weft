```yaml
lens: fix-affected-set
outcome: reported
```

**Task:** run the affected-set lens over the fix, rerunning each finding's own search.

**Findings:** no findings.

- **F5, effort-vocabulary sites:** a grep for `xhigh` across `.go`, `.yaml` and `.md` shows one non-test declaration, `effortVocabulary` in `internal/agentconfig/config.go:80`. `claudeengine.validateEffort` now reads it through `agentconfig.Efforts()`. The other hits are docs, crucible agent names and test rows. `codexengine` validates against `codex.yaml`'s own efforts, not `agents.yaml`, so the new load-time check does not reach it. The `agents.yaml` fixtures in tests only use `low`, `medium`, `high`, `xhigh` and `max`, so none breaks.
- **F1, `OpenLaunch` callers:** `shuttleengine.Runner` (`open.go:94`) is the only production caller. It passes `r.cfg`, which carries `Agents` from `LoadConfig`. The other hits are fakes and the codex engine, which ignores `cfg`.
- **F4, the disputed finding:** the four `ConfigRole` assertions the review asked for are already in the tree. They are `bouncer_judge_test.go:136-138` and `:181`, `discussiontable_test.go:164` and `tidy_test.go:344`, so the dispute holds.
- **Unchecked:** F2, F3 and F6 have no set premise. F6 restores one help line.

**Calls made:**
1. Grep for `xhigh` across `.go`, `.yaml` and `.md`: to find any other effort-vocabulary declaration. My context lacked a repo-wide view.
2. Grep for `OpenLaunch(` callers outside `claudeengine`, plus docs mentioning efforts: to cover F1's callers and any stale doc wording. My context lacked both.
