```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, guarantee and claim about existing code in `decision-record.md`.

**Claims attacked**
- D2: one module-instrumented run holds the package view — held.
- D3: duplicate block text costs "at most one wrong hint" and an edit errs toward no finding — **broken** (F1).
- D4: a whole-record replace through `internal/state` means a reader sees either the old map or the new one — held.
- D5: dirty and locked packages are skipped before any build and do not count against `--max` — held. The outcome for a package with nothing to map is unstated — **broken** through D6 (F2).
- D5: the time bound, one `lym` build plus per package one coverage build and tests × `redundancy_test_timeout_sec` — **broken** (F3, F5).
- D5: the hash recheck after the tests discards a map built under a landing — held for Go files; the scope of the bound text is misstated (F6).
- D5: "A dependency change outside the package directory does not mark a map stale" — understated (F6).
- D6: "A dobby stop ends the child with the dobby, since `runproc.EndRole` ends every descendant" — holds today only against `EndRole`'s own documented contract (F4).
- D6: at most one refresh per hub, and in-flight detection through a live `test-map` record — held for dobby-started refreshes. A manual `lym gate testmap` in the prime can still overlap, and per-package locks bound that.
- D6: an orphaned refresh "lasts no longer than D5's time bound" — inherits F3 and F5.
- D7: the config route, where `yamlengine.FillMissing` fills `reports` into a hub-tuned `gates.&lt;key&gt;` entry — held. `gates` is a per-name map, not an open map, and `FillMissing` steps into each entry key by key (`internal/gateslot/config.go:99-107`, `internal/yamlengine/fill.go:110-130`, `internal/yamlengine/doc.go:98-99`). An older binary reading `reports` decodes non-strictly and only warns.
- D7: `reports: []` turns the step off on a key — held, since `FillMissing` never replaces a value the file holds.
- D7 step 1: merge base through `parentMergeBase`, which reads the warp, and file text through the new `fabricengine.CodeFileAt`, which reads the warp — held. No parent branch judges nothing — held.
- D7 step 1: changed paths from `impactset.ChangedPaths` stand for changed tests — **broken** for renames and deletions (F7).
- D7: "the step never changes the gate's result" — **broken** by a hang (F5) and partly by a red lint (F8).
- D7: a repeat call on a matching tree runs no build and no test — held.
- D7: a red tier records no hashes and never matches — held.
- D7 bound: a false contained finding comes from "an edit in the diff to a test helper or a non-test callee" — understated (F9).
- D8: a content match survives commits to run records and to unrelated code — held as designed.
- Constraint: `merge.go` runs `MergeStart` in the parent's warp checkout under the weft write lock — held. Before that, `Merge` also fetches and fast-forwards the warp under the same lock (`internal/fabricengine/merge.go:345-351`); D5's hash recheck covers that too.
- Concurrent git use: D5's dirt check in the prime cannot take `index.lock` from a landing, since `gitexec` sets `GIT_OPTIONAL_LOCKS=0` (`internal/gitexec/gitexec.go:243,367`) — held.

**Findings**

F1 — MEDIUM, design — D3 Bound and Rationale (`decision-record.md:60-62`).
Summary: one inserted duplicate block shifts the ordinal of every later identical block in its function, and it errs toward a false "contained" finding, not toward no finding.
Counterexample:
- A function holds several identical `return err` blocks, so each key differs only by ordinal.
- A diff inserts a new `if err != nil { return err }` before them.
- The new block takes ordinal k, which the map gave to an old block that map tests cover.
- A new test that covers only the new block then matches a covered key and is reported contained.
- Every later identical block is also shifted by one, so the cost is not "at most one wrong hint".

This contradicts both "costs at most one wrong hint" (`:62`) and "an edit errs towards no finding" (`:60`).
Evidence: the D3 key definition (`decision-record.md:49-54`). Identical error-return blocks are the common case in Go.

F2 — MEDIUM, design — D5 staleness rule and D6 capped-leftover trigger (`decision-record.md:81-86, 112`).
Summary: D5 states no outcome for a package with no test files or a failed coverage build. If no map is written, "it has no map" keeps it stale forever. Such packages lead the path order, so they can use every refresh's whole `--max` batch, and D6 then restarts a refresh on every tick without ever reaching the real packages.
Counterexample:
- `contracts/orchrole`, `contracts/recipes`, `internal/hibernateconfig`, `internal/lymdirs` and `internal/patternlint` hold no `_test.go` (the scan I ran). Any `cmd/` package without tests also sorts before them.
- `go test -c` writes no binary for a package with no test files.
- With `test_map_batch` 5, each refresh spends its batch on these, leaves stale packages capped, and D6 (`:112`) starts again on the next tick, indefinitely.
- D6's "Packages a refresh skipped never start another" (`:113`) does not cover them, since they are not skipped.

Fix direction: a package with nothing to map, or a failed build, writes a map recording its file hashes, or is excluded from staleness and from the cap count.

F3 — MEDIUM, design — D5 time bound and D6 orphan bound (`decision-record.md:106, 121, 128`).
Summary: the time bound leaves out the gate-slot wait before the first package and gives the coverage build no limit. D6's "ends on its own within D5's time bound" inherits both gaps.
Counterexample: the hub's slots are held by tier2 runs of `webster` or `finalize` gates (hub `gate.yaml`: three slots, tier2 `timeout_min` 30), so the refresh waits on the slot for a time no stated term bounds. A hung `go test -c` has no timer in the design.
Evidence: D5 says "It holds one gate slot for the whole call" (`:92`); `_bolt/_lym/config/gate.yaml` sets `slots: 3`.

F4 — MEDIUM, design (false premise) — D6 (`decision-record.md:117-119`) and the D6 acceptance criterion "a dobby stop ends a running refresh" (`:235`).
Summary: D6 registers the child under `runproc.HubOwner` with its own role `test-map`, beside the prime dobby's `RoleDobby` record under the same owner (`internal/beamcli/doc.go:353`). The documented contract of `runproc.EndRole` spares exactly such a record and everything below it.
Evidence:
- `internal/runproc/end.go:71-74`: "Every record of another role, and every process below such a record, is left alone."
- `end.go:149`: "A record that ends rejects is skipped like callerPID, with the processes below it."
- `end.go:162-166` marks the `test-map` pid as spared.
- The descendant walk in `planRecords` (`end.go:355-381`) does not consult `skip`, so today the dobby's child is still signalled.

D6's stop therefore rests on an implementation detail that contradicts `EndRole`'s stated contract. Code brought into line with its doc would leave the refresh running, and the acceptance criterion would fail.
Fix direction: the dobby's stop path ends `RoleTestMap` explicitly, or the record cites the walk's actual behaviour and pins it with a test.

F5 — MEDIUM, design — D5 per-test timeout (`decision-record.md:99-102, 106`) and D7 step 4 with the "never changes the gate's result" rule (`:148-149, 159`).
Summary: a timed-out `tier1` test can leave a grandchild holding the test binary's stdout or stderr pipe, so the verb's or step's wait never returns, which breaks both time bounds.
Counterexample:
- A `tier1` test starts `lym` or `git` and hangs.
- At `redundancy_test_timeout_sec` the test binary is ended, but its child keeps running with the inherited output pipe.
- D5 says each spawned process "is logged through `internal/logger` and waited on" (`:102`). Go's `exec.Cmd.Wait` with a captured stdout waits for the pipe copy to finish, so it blocks until that grandchild exits.
- The record states no process-group kill and no wait bound.
- In D7 the same hang stalls the `lym gate test` call itself. Its caller, an agent's Bash call or a verify site, then sees a timeout or a hang rather than the tiers' result.

F6 — LOW, consistency — D5 Bound (`decision-record.md:107`) and D4 file hashes (`:70`).
Summary: the bound says only a dependency change outside the package directory escapes staleness. A non-Go file inside the directory escapes it too, both for staleness and for the mid-package landing recheck.
Counterexample: a landing changes `internal/gateslot/template.yaml`, which that package embeds, or a `testdata/` file while the package's tests run. The hashes cover Go files only (`:70, 85, 94`), so the map is kept although it was built from a mixed tree, and the age limit is the only bound.

F7 — MEDIUM, design — D7 step 1 and step 3 (`decision-record.md:141-147`).
Summary: `impactset.ChangedPaths` lists a renamed file by both its old and new path, and lists deleted files, all without status (`internal/impactset/targets.go:101-103`). Step 1 compares tests file by file at the merge base.
Counterexample:
- A diff renames a test file holding 40 or more tests.
- At the new path every test is "absent at the merge base", so all of them are judged.
- In file-then-name order they fill `redundancy_max_tests` (default 40), so the diff's actual new tests fall past the cap and are listed as not judged.
- A test moved between files is judged the same way, as if it were new.
- A deleted `_test.go` path has no current text, and step 1 does not say what happens to it.

Fix direction: compare a test's function text by package and test name across the package's test files at the merge base, not by file.

F8 — LOW, consistency — D7 red rule and D8 match (`decision-record.md:135, 158, 183-184`).
Summary: the step runs at the lints' site. Only a red tier gives a judged-nothing report, and D8 names only a red tier.
Counterexample: the tiers pass and a `comments` or `pattern` lint fails, so the gate is red. The step still judges and records hashes, and Code-Burler-Review then shows findings from a gate that failed, with no red marker.

F9 — LOW, consistency — D7 Bound (`decision-record.md:173-175`).
Summary: the false-contained mechanism is narrowed to "an edit in the diff". The same false cover arises without any edit in the diff:
- dependency changes landed on the parent since the map was built, which never mark a map stale (D5 `:107`);
- package-level test tables or `TestMain`, which sit outside a test's function text;
- `testdata/` and embedded files.

Only for a package whose files differ does step 6 name the map's age (`:156`). A package whose hashes match but whose map is days old shows Code-Review no drift.

**Tool calls**
1. `internal/runproc/end.go` `planRecords` and `RunProcDir`: to check whether `EndRole`'s descendant walk honours the spared `test-map` record; my context held only `ender.end`.
2. `internal/runproc/end.go` descendant walk, `groupMembers` and `signal`: to settle whether the walk or a group signal reaches a spared descendant (F4).
3. A scan of the tracked Go package directories that lack a `_test.go`: to build F2's concrete counterexample in path order.
