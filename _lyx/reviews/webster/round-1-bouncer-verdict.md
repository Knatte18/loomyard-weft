---
verdict: APPROVED
rationale: "All five round-1 findings are fixed in 769afcb6e..e879da907 (keys= assertions, stale gitignore.Ensure line dropped, one-sentence-per-line comments, reed.json byte-identity plus kill-pane, moduleTemplate ok check); build, vet (untagged, smoke, integration) and go test ./... pass on HEAD, both Delete groups' no-caller checks hold, and a fresh pass over the eight card commits found no new defect."
---

## Review range

No `_lyx/loom/rework/` directory exists, and `_lyx/shed/operator-surface-fixes/status.json` records `product.parent: main`.
The subject is `git diff $(git merge-base main HEAD)..HEAD`, which is `812600b3a..e879da907`: the eight card commits plus the five fix commits.

## Round-1 findings, checked against HEAD

- **F1 (MEDIUM), fixed.**
  `assertOneFillLine` in `internal/configengine/config_test.go:121` and `assertFilled` in `internal/configreg/fill_test.go:122` now match `"keys="+keyPath`.
  The temp path can no longer satisfy the assertion, since it sits in the `file=` attribute.
  Each test drops exactly one key, so `keys=` holds exactly that path.
- **F2 (LOW), fixed.**
  The `gitignore.Ensure is never called` line is gone from `SpawnDriven`'s doc.
  The sentence before it already gives the reason for the exclude.
  No reference to `gitignore.Ensure` or `internal/gitignore` remains in any `.go`, `.md` or `.yaml` file outside `_lyx/`.
- **F3 (LOW), fixed.**
  I swept every added `//` line in the diff for a sentence continued across lines.
  The only hits are three breaks at an independent-clause boundary (`-- an already-seeded worktree…`, `-- an explicit literal is what…`, `, so a SaveState…`), which CONSTRAINTS.md's Comment Line-Break Convention allows.
  The other hit is the `dependsOrder` comment in `internal/vscode/config.go`, which was moved unchanged into `writeInteractiveTasks`.
  Legacy wrapping there is not a changed comment.
- **F4 (LOW), fixed.**
  `TestAddStrandUnless_LiveNamedSkips` now compares `reed.json` bytes before and after the skipped add and forbids `kill-pane`.
  The fixer reports a mutation check: adding a `SaveState` on the skip path made the test fail.
- **F5 (NIT), fixed.**
  The new `moduleTemplate` helper checks `configreg.Template`'s `ok`, and all three fill tests use it.

## Fresh pass over the card diff

- `AddStrandUnless` (`internal/reedengine/strand.go`) resolves `unlessName` through `validateNaming` before `ensureSessionLocked`, so an unformable name refuses before tmux boots.
  `validateNaming` only forms the name and does not check whether it is held, so a live `orch` does not make the skip refuse.
  The live check runs before the `IfAbsent` branch and returns with no save, reconcile or focus change.
  It matches `orchcli`'s orch strand, which is added with `NameOverride: agentname.RoleOrch` and a non-hidden default anchor, so the folder-open chain's `--unless-name orch` reaches the real strand.
- `writeVSCodeConfig` (`internal/ideengine/spawn.go`) runs the same sequence for both entry points.
  It does the tracked check, then the exclude, then `WriteConfig`, and a tracked file means no write but the launch still happens.
  `Spawn` keeps its prime workspace path after a tracked skip.
- `FillMissing` (`internal/yamlengine/fill.go`) follows card 1's rules.
  `load` fills, logs one Info line, runs the `MissingKeys` backstop without the reconcile hint, and resolves the filled bytes.
- Per-card mechanical checks:
  - Card 6's Delete group passes. `spawn_test.go` declared only `TestSpawn` and `assertSpawnedTasksStampTheReedChain`, and the helper has no caller under any build tag.
  - Card 7's Delete group passes. No Go file imports `internal/gitignore`, and no doc outside `_lyx/` names it.
  - Cards 1–5 and 8 carry no Rename or Delete group.
- Verification on HEAD: `go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go vet -tags integration ./...` and `go test ./...` all pass.
  `go test -tags integration` passes for `reedcli`, `ideengine`, `configengine`, `configreg`, `reedengine`, `vscode` and `yamlengine`.

The three `TestUnlessName_*` functions carry no doc comment.
`reedcli`'s own integration tests are mixed on this (`naming_integration_test.go` and `list_integration_test.go` carry none either), so the code around them sets no convention to break, and it is not a finding.

## Focus departures

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.
