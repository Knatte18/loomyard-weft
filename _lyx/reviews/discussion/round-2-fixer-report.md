# Round 2 fixer-report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed: D1 now says both hub-wide write paths of `lyx config darn` load through darn's loader and roll back a new failure.
  A D1 sentence makes `copyStagedToHub` in `internal/configcli/hubwide.go` gain the `module.CheckLoad` before-and-after check `setHubWide` runs.
  A new acceptance criterion: an editor save leaving `tree_scope_verify` set with an emptied `tree_scope`, or holding `tools` and `tools/`, is refused and leaves the board file unchanged.
  Premise checked: `copyStagedToHub` validates only through `validateHubFile`, a bare `configengine.Load`.
- F2 (MEDIUM), fixed as proposed: D2 says the read reports each path verbatim, as `-z` does, never core.quotePath's C-quoted form.
  D1's refusal list adds an empty segment (`tools//x`) and says only one trailing `/` is trimmed, so `tools//` is refused.
  The malformed-key criterion adds `tools//x` and `tools//`, and a new criterion covers a non-ASCII path under a tree.
  Premise checked: `diffNames` runs `git diff --name-only` without `-z`.
- F3 (LOW), fixed as proposed: the no-`{targets}` refusal now covers only a non-empty `tree_scope_verify`, and the `LoadConfig` criterion now says "a non-empty template".
- F4 (LOW), fixed as proposed: D1 says removing the last tree requires `lyx config darn --set tree_scope_verify=` first, or clearing `tree_scope_verify` in the same editor save.
- F5 (LOW), fixed as proposed: D5 says a scan failure that is not a guard-scan error refuses the verb before any slot with the error, and only the three content kinds name a file and line.
  The `--guards` scan-failure criterion now separates the content kinds from an unreadable directory.
  Premise checked: `FindGuardTests` returns a plain wrapped error on an `os.ReadDir` failure (`internal/impactset/guards.go:49`).
- `support-log.md`: a Round 2 entry under Review rounds.

## Self-check

- fix-claims, LOW: D1 said the rollback runs through `newLoadFailure`, which only builds the refusal; fixed, the rollback is through `rollBack`.
- fix-consistency, LOW: Scope and D6 omitted the editor-path check, which applies to every hub-wide module, and `internal/configcli/doc.go`'s `--set`-only sentence; fixed, with a Scope In bullet and a D6 sentence restating that doc line.
- fix-consistency, LOW: the no-tree refusal's way forward named only adding a tree; fixed, it also names `lyx config darn --set tree_scope_verify=`.
- fix-proposal, LOW: a failed package enumeration has no package directory to name, since `go list` enumerates; fixed, it names the `-C` module directory and the enumeration's error.
- fix-way-forward, MEDIUM: `newLoadFailure`'s message names `--set`, which D1 rules out on the editor path; fixed differently from a plain reuse, the editor refusal carries its own way forward (re-open the editor and fix the key named, or `--set tree_scope_verify=` first for the last tree).
- fix-way-forward, LOW: the non-content scan refusal named no way forward; fixed, make the directory readable or fix the module until `go list ./...` succeeds.
- fix-way-forward, LOW: a partial repair of a broken file is refused, and the editor discards the rejected edit; fixed, D1 states that a save must load or fail with the same error as before, and the refusal keeps the edit at the staging copy.

## Disputed

None.

## Deferred

None.
