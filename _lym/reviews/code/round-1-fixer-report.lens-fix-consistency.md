```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens, meaning twins or siblings of the changed text left stale.

**Findings**

1. **MEDIUM** — `internal/driftcli/doc.go:33-35` (the `accept` paragraph) was not updated to match the new `acceptTest`.
   - The paragraph says `accept` "runs driftengine.CheckAccept first", then captures, then calls Accept. That order is now wrong: `acceptTest` takes the accept lock first (`holdAcceptLock`, `cli.go`).
   - It doesn't mention that a held accept lock refuses the verb.
   - It doesn't say the run status is read again after the capture. The old text implied one read.
   - Evidence: `driftcli/doc.go:33` against the `cli.go` hunk that adds `holdAcceptLock`, a second `verbStatus()` call and the "another accept holds" refusal.
   - The `acceptCmd` `Long` help text has the same gap. It says nothing about the concurrent-accept refusal, which is a new operator-visible refusal.

2. **LOW** — `docs/overview.md:615` still says "its accepted tests against the profile captured when they were accepted". F6 reworded this idea everywhere else in the code and docs, but this twin was missed.
   - F6's fix key named only `recheck.go`, `doc.go` and `recheck_test.go`.
   - The accepted profile is built from the kept outputs, and `AcceptedProfileHead` records where a compare against it last passed, so this wording is stale in the same way.

3. **LOW** — `internal/driftengine/doc.go:45` says "RunAcceptanceCapture takes no lock and writes no record either". It is still true of that function.
   - Line 21 now declares an accept lock that serializes accepts.
   - A reader of the module doc gets no sentence saying that lock is held by the verb around the capture and `Accept`, not by the engine function.
   - Line 72 of the same file also still describes only `CheckAccept` then capture then record.

4. **LOW** — Name-matching wording was not updated for the F1 change.
   - The `accepted.go` header (line 3), `doc.go:22` and the `acceptCmd` `Long` text all say "with or without its extension".
   - After F1 two shapes are carved out: a dotfile (a top-level `.gitkeep`, or a nested `group/.gitkeep`) has no extension-less name, and `t1.txt.bak` is named by `t1.txt` but not by `t1`.
   - The behaviour is pinned by the new test rows, but the prose doesn't say so.

**No finding**
- F6 wording in `recheck.go` (doc comment and Stuck text), `doc.go:25`, `doc.go:148` and the `recheck_test.go` helper comment, subtest names and assertion now agree.
- `dropAcceptedOutput` has two callers (`keepAcceptedOutput`, `Withdraw`), and both pass the new third argument.
- `Paths.AcceptLock` and its `NewPaths` entry agree with the `doc.go:21` mention.
- The new test's expected fragments match the refusal texts in `cli.go` and `accepted.go`.

**Calls**
1. `grep` for stale "profile captured", "no accepted profile", "accept lock", "takes no lock", `dropAcceptedOutput` and `path.Ext` over `internal` and `docs`, plus `accept` in `internal/driftcli/doc.go`. Reason: my context lacked `driftcli/doc.go`, `docs/` and any other twins of the changed wording.
