```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff `1c344094a..HEAD`. I ran each changed function or branch against one concrete input.

**List (function or branch, case run, result):**
- `cmd/lyx/main.go` root registration: the root builds with `StartAliasCommand` gone. `loomcli.Command()` still registers `loom start`. Correct.
- "Available modules" help string: `start` is dropped and the list matches the registered children. Correct.
- `loomcli.StartAliasCommand` removal: no reference to it remains in the repo (grep). `newLoomCLI` is left with the one caller, `Command()`. Correct.
- `loomcli/start.go` `startCmd`: untouched apart from help text, so the flags and the hidden `--no-attach` are unchanged. Correct.
- `shortcuts.yaml`: it now holds `a`, `prime`, `p`, `r`. No non-test Go code in `shortcutcli` or `cmd/lyx` hardcodes `"attach"`, `"pause"`, `"resume"` or `"start"` (grep), so nothing else keys on the removed names. Correct.
- `pauserecord.heldLine`, input hub scope: it renders `"lyx tender resume"`. Correct.
- `pauserecord.heldLine`, input pair scope `s`: it renders `"lyx tender resume" or "lyx tender resume s"`. Correct.
- `pauserecord.CoveredError`, hub scope and pair scope: the format arguments are unchanged and only the literal text changed. Correct.
- `tenderengine.Resume`, slug covered only by a hub entry: the refusal now reads `run "lyx tender resume" to lift the hub pause`. Correct.
- `adoptcli.namedEntries`, done entry: the `%s` verbs still match their arguments, `slug, slug` and `slug`. Correct.
- `adoptcli.liftOwnerPauses`, both branches: the `Sprintf` argument counts are unchanged. Correct.
- `fabricengine` `stopWayForward`, `BranchBusy` with push lock held: it returns `` `lyx tender resume &lt;slug&gt;` ``. Correct.
- `hibernateengine` `reconcileAtWake` and `runStep`: the `Sprintf` argument counts are unchanged. Correct.
- `loomcli.loomBarePause`, input slug `""` (location nil): it yields `lyx tender pause ` with a trailing space. This is the same as before the diff, so it is not a regression.
- `tendercli.refuseCaller`, `tendercli.resumeCmd` and the help Long strings: text only, no logic change. Correct.

**Findings:** none.

**Tool calls:**
1. `grep` of the `shortcutcli` and `cmd/lyx` non-test Go files for the removed word literals, plus a repo-wide grep for `StartAliasCommand`. My context did not show whether any non-diff code keyed on the removed names or still referenced the alias.
