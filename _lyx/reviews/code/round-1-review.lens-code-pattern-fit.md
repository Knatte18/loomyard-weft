```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against the repo's rules (PATTERN entries, `CLAUDE.md`, commit message).

**Rule list**
- `PATTERN-spawn-observability`: held. `startSuite` logs the detached spawn at Info (`deps.go` startSuite), and a failed start is logged at Warn, once and not in a loop. `Start` has no `Wait`, so no teardown log is needed. It returns before spawning under `testing.Testing()`, which meets "never re-exec `os.Executable()` under `go test`".
- `PATTERN-durable-vs-ephemeral-state`: the token and the path are held, one clause is a minor deviation.
  - Held: the directory name comes only from `lyxdirs.AdoptDirName`, so no `.lyx` literal is introduced. The log sits under `HubScratchDir`, which is `BoardDir/.lyx`, so it is never tracked.
  - Deviation: the pattern says each module exposes a scratch accessor and no engine derives its own `.lyx` path. `runproc` and `failed-push` each have a `fabricengine` accessor (`junctionnames.go:191`, `failedpush.go:81`). The adopt log path is instead joined inline in the `hubDepsAs` closure. See finding 1.
- `PATTERN-run-process-owner`: held. The suite is not a process a run starts. It is a hub-level `lyx gate test`, and `gate test` holds its own slots.
- `PATTERN-gate-slot`: held. The suite runs through `lyx gate test --gate finalize --wait`, and the flags exist in `gatecli/cli.go:102,109`.
- `PATTERN-cwd-resolution`: held. The spawn dir is `prime.AnchorPath()`, and `prime` comes from `battencli.ResolveBattenPrime` (`cli.go:71`). The engine is handed absolute paths.
- `PATTERN-leaf-packages`, `PATTERN-shell-guard-table`, `PATTERN-pane-text-injection`: not touched.
- `PATTERN-test-speed`, `PATTERN-test-isolation`: held. The new rows go in the existing tier0 `TestAdopt` table with a faked seam. The one new `deps_test.go` test pins the `go test` guard and runs under `t.Parallel`.
- `PATTERN-lyx-temp-root`: held. The new test uses `t.TempDir()` like the rest of the file.
- `PATTERN-refusal-way-forward`: held. `suite_warning` names `lyx gate test --gate finalize` in the prime as the way forward.
- `CLAUDE.md` docs rule: held. `doc.go`, the long help in `cli.go` and `docs/overview.md` all land in the same commit. No new module and no new invariant, so no `PATTERN.md` edit is owed.
- Declared PATTERN changes: the directive declares none, so there is nothing to check.
- Commit message and departures: no `Pattern-Departure:` trailer is present or needed.
- Comment conventions: held. Doc comments are self-contained and have no end-of-line comments. Line wrapping is not a finding.

**Findings**

1. LOW, `consistency`, `internal/adoptcli/deps.go` (the `StartSuite` closure in `hubDepsAs`, the `filepath.Join(fabricengine.HubScratchDir(...), lyxdirs.AdoptDirName, suiteLogName)` line). The adopt log path is derived inline in `adoptcli`. The established shape for hub `.lyx` subdirectories is an accessor beside `HubScratchDir`, as for the run-process and failed-push directories. `PATTERN-durable-vs-ephemeral-state` says "No engine derives its own `.lyx` path; each module exposes a scratch accessor". Evidence: `junctionnames.go:191` and `failedpush.go:81` against the inline join in `deps.go`. Fix: add a `fabricengine` accessor such as `HubAdoptDir`, or `HubAdoptSuiteLog`, and call it from `deps.go`.

2. NIT, `scope`, `internal/adoptcli/cli.go` step 3 of the long help. The diff drops the clause "because this machine's dobby drives only runs whose parent is its own worktree" from step 3. The decision record does not ask for it. The change summary says it was trimmed to stay under the help-length cap, but the diff does not show whether the cap needs the cut. The rationale it removes is still true; `doc.go` may carry it, but I did not check. Fix: keep the clause if the cap allows, or confirm the cap forces the trim.

**Tool calls**
1. Read `pattern/PATTERN-spawn-observability.md`, `PATTERN-run-process-owner.md` and `PATTERN-durable-vs-ephemeral-state.md`. They were linked from `PATTERN.md` but not yet read into my context.
