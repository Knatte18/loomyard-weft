```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals the fixer applied as written (F1–F4, F6–F13; F5 is decision class and left to the parent).

- **MEDIUM — F4 proposal, D7's "Every reader that compares a map's `commit` with HEAD to mean the map is current reads the planned HEAD … The audit's candidate filter … is one such reader."**
  Moving that check to the planned HEAD lets the audit re-measure a map that was never written, but the result lands in a map whose `commit` and file hashes still stand at the older commit. That contradicts D7's own statement that a map's `commit` "means the commit its content was last written at", and that every written map moves its `commit` and format commit together.
  Concrete case:
  1. A pass leaves package P's map at commit C0, while HEAD is C1.
  2. The audit picks P's oldest entry.
  3. `audit` calls `mapPackage` with `audit: true`, which is a non-whole job (`internal/gatecli/testmapaudit.go` loop; `internal/gatecli/testmappackage.go:80-87`).
  4. `mapPackage` sets `m = planned` and writes the fresh entry measured at C1 into that map. The map's `Commit` stays at C0, and its `Files`/`Linked` hashes stay at C0.
  5. The next selector diffs C0→HEAD against blocks measured at C1.

  The proposal only handled the "is this map current" check. It left open whether an audit write moves the map to HEAD with HEAD's tree (the map is then written and its `commit` moves) or keeps the old `commit`. D7 needs to say which.
  The same `mapPackage` guard (`planned.Commit != head`, testmappackage.go:82) is a second reader, which the fix's "every reader" sweeps in without naming. As things stand it would skip every audited unwritten map as "no longer stands at the planned HEAD", so the new D7 criterion ("still lets the audit sample that package's tests") cannot hold unless that guard changes too.

- **LOW — F2 proposal, D11's "imports each old `map.json` whose package has no map in the prime's `_lym/testcover/`" and "removes an old map only once the prime holds a committed map for its package."**
  Concrete case: an old bolt map belongs to a package that no longer exists at the prime's HEAD, or for which the prime's pass deliberately writes no map. The import brings it in, but if the pass drops or never commits a map for that package, the old map is never removed and is re-imported on every call. That breaks "a second call imports nothing".
  I found no code that deletes a map today (no `Remove` on map paths in `internal/gatecli/testmap*.go` or `internal/testcover`), so this only bites if a later plan adds that. The guard has no terminal rule for a package with no counterpart.

- **LOW — F1 proposal, D9's slug registration.**
  Several existing readers list the slug's registered processes: teardown's `ownedProcesses` (`internal/pairteardown/teardown.go:200-202`), cancel's `LiveRecords` (`internal/battencli/cancelcmd.go:260`) and `gateProcessLive` (`internal/battencli/wire.go:774`). The ones I read filter by role (`RoleGate`, fixer roles, `RoleDriver`), so a `RoleTestMap` record does not falsely mark a run as at work.
  I did not read whether teardown's or cancel's refusal treats any live slug record as busy. If one does, a hand pass in a task would block that pair's teardown or cancel rather than be ended by it. D9 asserts "ends it" without checking this.

Tool calls:
1. Read `internal/gatecli/testmapaudit.go` lines 60–140. My context ended at line 60, before the audit's write path.
2. Grepped map removal and every `runproc.List` caller. My context did not show whether passes delete maps, or who lists a slug's records.
3. Read `internal/battencli/wire.go` around 774 and `internal/pairteardown/teardown.go` around 200 and 754. I needed to see how those slug-record readers filter by role.
