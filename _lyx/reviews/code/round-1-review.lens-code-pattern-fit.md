```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit. I checked each changed file against `PATTERN.md`, `pattern/` and `CLAUDE.md`.**

**Rules checked**

- **Held:**
  - `PATTERN-cwd-resolution`: only `driftcli` imports `lyxcwd` (`wiring.go:17,31,35`); `driftengine` takes absolute paths and imports none.
  - `PATTERN-lyx-temp-root`: no `os.TempDir()` in the new non-test code.
  - `PATTERN-durable-vs-ephemeral-state`: `paths.go` builds `_lyx/drift` and `.lyx/drift` from `lyxdirs.LyxDirName` and `lyxdirs.DotLyxDirName`, with the same `drift` subpath.
  - `PATTERN-config-layers`: `LoadConfig` goes through `configengine.Load` strictly, and `capture` and `compare` are `CategoryExecutable`.
  - `PATTERN-shed-recipe-registry`: the three entries are in `registry.go`, all three rows carry `entry_gate: drift.enabled` in both recipes, and the engine is a `Deps` passthrough on `Env`.
  - `PATTERN-spawn-observability`: `spawn.go:107,127` and `steprun.go` log every spawn through `logger`.
  - `PATTERN-run-process-owner`: the capture child is registered with `runproc.RoleCapture` (`spawn.go:128`). `runproc` doc and record are updated.
  - `PATTERN-daemon-wakeups`: `wait.go` polls from 1s doubling to 30s. It is listed in `pattern/PATTERN-daemon-wakeups.md`, and `cmd/lyx/daemonwakeups_test.go` is touched.
  - `PATTERN-gate-slot`: capture and compare hold a slot (`steprun.go:acquireSlot`). The `drift-verify` key is added to the template, `goReadGates` and `envkit`. `pattern/PATTERN-gate-slot.md` and `pattern/PATTERN-test-speed.md` are updated.
  - `PATTERN-cli-cobra`: `driftcli.Command()` is a subtree registered in `main.go`.
  - `PATTERN-cliwire-sole-wiring`: `driftcli/doc.go:4` states why `cliwire` does not serve it; this is the overview plan's shared decision.
  - `PATTERN-tolerant-records`: all records go through `internal/state`.
  - `PATTERN-test-speed`: every drift test file carries a `tier0` or `tier1 && !windows` tag, except the two `testmain_test.go` files, which have none.
  - `PATTERN-gitrepo-client-boundary` and `PATTERN-fabric-git`: no raw git in the new non-test code. Git facts go through `fabricengine`.
  - The `DecisionHaltPrefix` doc is widened (`bouncerdecision.go:25`), and `arbiterstop.go:63,165` already reads that prefix, so drift halts are handed to the parent.
  - `PATTERN-pair-teardown`: teardown ends the capture's named job through `proc.TerminateNamedJob` (`teardown.go:733`).
- **CLAUDE.md "Docs land in the same commit":** held. `driftengine/doc.go`, `driftcli/doc.go`, `docs/overview.md`, the `gateslot`, `loomshed`, `loomrecipe`, `shedrecipe`, `proc`, `runproc` and `pairteardown` docs, and `docs/dotnet-hub.md` are all in the diff.
- **Declared PATTERN changes:** I found no declared-changes list in the context I hold. `PATTERN.md` is not in the diff, and the decision record says the design departs from no entry. The three `pattern/` background edits land. I raise nothing under this heading.
- **Commit messages and departures:** no `Pattern-Departure:` trailer appears in the range, so there is nothing to judge.

**Findings**

1. **LOW, consistency:** `internal/driftengine/records.go:44-47,63-64`. The `Ruling` and `RulingUnruled` docs say the ruling applies "under partial scope" and to an acceptance "a caller has not yet ruled on".
   - The code in `verbs.go:Accept` sets `Ruling` only under scope `full`. There the agent's acceptance is `unruled` and the caller's is `ruled`.
   - The decision record (D7, D6) also says `full`.
   - This breaks the "doc comments must be correct" convention from the Comment-convention rubric bullet.
   - Fix: change both docs to say scope `full`, and say `unruled` means the agent's acceptance awaits the parent's ruling.

2. **LOW, consistency:** the branch point and the code-checkout dirty paths each have two readers.
   - The row side uses `loomcli.pairMergeBase` (`darnwiring.go:58`, via `readRecordedParentBranch`) and `fabricengine.SourceDirtyPaths` (`driftwiring.go:76`).
   - The verb and capture-child side uses `ReadOrigin` plus `Fabric.MergeBase`, and `Fabric.CodeDirtyPaths` (`driftcli/wiring.go:~80-105`).
   - The row's pre-check and the child's check under the lock are meant to judge the same facts. The decision record only names `SourceDirtyPaths` for the row.
   - I read both dirty-path functions only partly. `SourceDirtyPaths` reads `trackedDirtyPaths(l.WorktreePath())` and `CodeDirtyPaths` reads `codeDirtyPaths(f)`, so I could not confirm they agree.
   - Fix: have `driftcli` use `fabricengine.SourceDirtyPaths(w.location)`, and share one branch-point reader between `driftcli` and `loomcli`. Otherwise add a test pinning that both agree.

Unverified: I did not check that every refusal and halt reaches a test (`PATTERN-refusal-way-forward`), or that `runproc.EndSparing` ends the capture. Those need the test files and `runproc` internals, which my context lacks.

**Tool calls**
1. A combined bash read of the diff stat for `PATTERN.md`, `pattern/` and `docs/`, imports greps, test build tags, the doc diffs and `Pattern-Departure` trailers. My context did not have the full stat or the test headers.
2. A bash read of `driftcli/wiring.go`, plus greps of `driftwiring.go` and `cli.go`. I needed to know how `driftcli` derives git facts and holds the job.
3. A bash read of `SourceDirtyPaths`, `CodeDirtyPaths`, `MergeBase`, `pairMergeBase` and the capture-child body, to compare the two derivations.
