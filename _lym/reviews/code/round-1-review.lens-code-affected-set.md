```yaml
lens: code-affected-set
outcome: reported
```

No findings from the code-affected-set lens; every change I traced is covered.

**Per change: the set found, covered or not**
- **Retired worker, record, steps and follow (`adoptrecord`, `worker.go`, `steps.go`, `steprun.go`, `runreport.go`, `pause.go`, `wouldStop`, `--keep-local`, `--drop-local`, `--discard` for adopt, `lym adopt worker`):** covered. A repo-wide search outside tests, `_lym`, `.lym` and `docs` finds no remaining reference to the removed code or flags. What remains is deliberate:
  - `pauserecord.AdoptCaller` and `HoldCaller`, kept for the legacy-entry readers and the upgrade's first read.
  - `runproc.RoleAdopt`, whose comments say "an older lym".
  - The `dobbyengine` comments about legacy entries.
  - The matches under `docs/research` are historical research notes, not rule documents.
- **Adopt record readers (dobby handover, orch watcher, `lym beam status`):** covered. `dobbyengine/handover.go`, `orchcli/watch.go` and `beamcli/adoptsection.go` were changed, with their tests.
- **Shell-guard row `adopt-take-hold`:** covered. The steer and notice spell no flag and name asking the operator. Only the pattern and `Forms` name it. `rows_integration_test.go` was extended, and the `claudeengine` settings golden and test were regenerated.
- **`--take-hold` appearing in refusals:** covered. The `boardengine/holdwrite.go:222` refusal names the flag, but it is reached only when `TakeHold` is already set, so it is operator-run. Every refusal an agent reads uses `holdWaitWayForward`, which says to ask the operator.
- **`lym adopt` CLI help (`cli.go`):** covered. The help is rewritten to the foreground flow and names `--take-hold`, as D6 asks.
- **Package docs:** covered. `adoptcli/doc.go`, `adoptrecord` (removed), `fabricengine/doc.go`, `boardengine/doc.go`, `beamcli/doc.go`, `orchcli/doc.go` and `pauserecord/doc.go` were updated.
- **`PATTERN-dobby-runs`:** the declared text landed in the `PATTERN-dobby-runs.md` intro and the "Adopt hold" section.
- **`PATTERN-daemon-wakeups`:** the `lym adopt` follow entry was removed, which matches the retired follow.
- **No other pattern entry drifted:** the diff touches only these two pattern files.
- **Commit messages:** none carries a `Pattern-Departure:` trailer, so none needs judging.
- **Stencils and specs:** the diff changes no stencil or spec file, and no spec or stencil under the repo mentions the retired adopt verbs.

**Tool calls**
1. `grep` for stale references to the retired adopt vocabulary in `.go`, `.md` and `.yaml` outside `_lym`, `.lym` and tests: my context had only the diff and a truncated first grep.
2. `grep` for the follow, detach, `AdoptRecord` and `fix-tries` vocabulary, plus `git diff --stat` over `contracts`, `specs` and `stencils`: to confirm no stencil or spec drift.
