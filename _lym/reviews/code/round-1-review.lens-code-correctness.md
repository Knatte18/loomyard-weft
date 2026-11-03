```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, case by case.

**Per function and branch**

| Function or branch | Case run | Result |
|---|---|---|
| `gateslot.ResolveCommandDir` (`gateslot/config.go`) | `""`, absolute, relative, `"."` | Correct. It matches the gate run path: `WithDefaultDir(AnchorRel)` then `resolveRunDir(root, dir)` gives root/`AnchorRel` for an empty dir, so it is the anchor, and `AnchorRel` `"."` gives the root. Absolute and relative dirs resolve the same on both sides. |
| `gateslot.StatCommandDir` | missing path, path that is a file, existing dir | Correct. It refuses the first two with the resolved path, and returns the resolved path on success. |
| `gateslot.CheckDirs` | load failure, tier with empty dir, `{}` build, build with a missing dir | Correct. A load error is returned as is. An empty dir stats the anchor. A build with no command is skipped. |
| `driftengine.decodeStep` | a declared step with no `dir` | Correct. `Dir` stays empty, and `ResolveCommandDir` turns that into the anchor. |
| `driftengine.CheckDirs` | undeclared step (`Command == ""`), missing dir, existing dir | Correct. Undeclared steps are skipped and a missing dir is refused under key `<step>.dir`. |
| `runStep` (`steprun.go`) | missing dir | Correct. The refusal comes after `renderCommand` and before the log directory, the slot wait and the spawn, so no slot is taken and no log is created. |
| `runStep` | existing dir | Correct. `cmd.Dir` is the resolved dir. |
| `missingDirRefusal` | `capture` and `compare` | Correct. The text carries `drift.yaml key "capture.dir"` (or `compare.dir`), the resolved path, `lym config drift --set <step>.dir=<dir> from the prime`, then the capture or resume verbs. The `%w` cause keeps the path text unchanged. |
| `RunSideCapture` passing `Name: stepCapture` | side capture with a missing dir | Mostly correct. The key and path are right. The way forward ends in "lym drift capture and lym loom resume", which is the baseline capture's recovery, not a side capture's. |
| Request builders (`driftcli` ×3, `loomcli` ×2) | each request that reaches `runStep` | Correct. All five fill `WorktreeDir` with the worktree root and `AnchorDir` with the anchor. None is swapped or left empty. The other `AnchorDir` uses (`spawn.go` child cwd, `Holder.Worktree`) are about the anchor, so they stay right. |
| `configcli.checkHubModule` and `dirsOf` | `dirs.root`, `dirs.worktree` | Correct. `baseDirOf` is `Join(WorktreePath, AnchorRel)`, so `dirs.worktree` is the anchor and `dirs.root` is the worktree root. |
| `checkHubModule` | module with `CheckDirs == nil` | Correct. It returns after `CheckLoad`. |
| `checkHubModule` | a `CheckLoad` error | Correct. `CheckDirs` is not run, so `LoadConfig` is not loaded twice on a failing file. |
| `setHubWide` and `copyStagedToHub` | file already failing with the same dir error, before and after | Correct. The same `Error()` string passes the write, and a different failure is rolled back. |
| `setChecked` and `editMachine` refactor | `newLoadFailure(module, before, module.CheckLoad(...))` | Correct. It is the same logic, and `after` is evaluated after the write as before. |
| Notice `PairDir` change | the other users of `PairDir` | Correct. It is only a rendering base (`pairRelative`, `notice.go`), and no file is located through it. |

**Findings**

1. MEDIUM, design, `internal/driftengine/seats.go:136`: `captureReading` still builds `run in `<step.Dir>` under the anchored directory`. `LoadConfig` no longer defaults `Dir` to `.`, so an unset dir now renders as run in `` (empty backticks), and a relative dir is described against the old base, which is the one this diff removed. The Drift-Baseline agent's prompt is therefore wrong, and it judges the suite's reach from that text. The diff changed no test or text here, and no test asserts this wording. Evidence: `seats.go:132-137`; `config.go:223`, where the default was removed.
2. LOW, design, `internal/driftengine/capture.go:237` (with `steprun.go:486-492`): a side capture is refused with the baseline capture's way forward ("lym drift capture and lym loom resume") because `Name` is `stepCapture` for both. The side capture is a different verb, so it should end in `lym drift side-capture` or the verb that runs it. Evidence: `capture.go:228-237`, `steprun.go:486-492`.

Tool calls:
1. One `grep` batch. It covered `baseDirOf` (the diff does not show its body), `PairDir` consumers, every `CaptureRequest`/`CompareRequest` constructor outside tests (to check `WorktreeDir` is filled everywhere), and the `Site` and step-name constants (for the refusal text).
