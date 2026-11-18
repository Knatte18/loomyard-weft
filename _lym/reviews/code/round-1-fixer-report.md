---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

## What changed

- **F1.**
  Each engine resolves the lym binary once per `Prepare` through the exported `LymBinary` variable, which defaults to `selfexec.LymBinary`.
  It refuses the launch on a resolve error, before Prepare writes anything.
  `pythonAllowedCheck` is now a function of that path, shell-quoted, in both engines.
  The path reaches the rendered command through `buildSettings`, `standingDeniesFor`, `shellGuardDenies`, `renderShellGuard` and the manifest's `guardCandidates` in Claude.
  In Codex it reaches it through `toolHooks`, `guardEntries`, `guardCommand` and `launchFacts`.
  `buildDenyNotice` and the package `init` render the denies over an empty path, since they read notices and steer text alone.
  The tests run the stub at the injected path, with the no-lym row as a nonexistent path, and the settings golden holds `/stub/lym`.
  Test binaries that run a real `Prepare` point `LymBinary` at a stand-in in their `TestMain`:
  `claudeengine`, `codexengine`, `shuttleengines`, `orchcli` and `shuttleparity`,
  and the packages whose live-tier smoke tests prepare a launch: `webstercli`, `treadleengine`, `seatengine`, `burlerengine`, `creelcli`, `shuttlecli` and `loomcli`.
  The package docs and `docs/overview.md` name the launching lym by path.
- **F2.**
  The standalone test builds a target with the entries `dotgit` reads as a repository, so `ResolveWithin` succeeds, and runs the verb from a subdirectory.
  It has a row with a sibling Bolt directory holding no `_lym`, and a row with no shuttle file in the state directory that expects exit 1.
  No process is spawned, so the test stays tier 0.
- **F3.**
  A `ConfigThatDoesNotLoad` row writes an unclosed flow sequence at the hub's default layer and expects exit 1 with a JSON error.
- **F4.**
  `shuttleConfigDirs` takes the hub layers only when `preflight.BoardLymPresent` holds, and otherwise looks for the state directory.
- **F5.** `docs/overview.md` says the Codex python row is installed in every launch and asks the hidden verb on a match.
- **F6.** Both `pythonAllowedCheck` doc comments split into whole sentences.
- **F7.**
  `_lym/landing/summary.md` names the standalone fallback, rewords the fails-closed decision, replaces the "pure function of the row" decision with the by-path decision, and lists the new tests.
  I edited the file and ran no git command against it.
- **F8.** A `ProjectDirectoryCannotBeEntered` row, with a `projectDirMissing` field, expects the deny and no lym run.
- **F9.** The three `python_only` rows and the `pythonDeny` column of `TestBuildSettings_DenyToggleMatrix` are removed.

Commits: one for F2, F3 and F4, one for F5, one for F1, F6, F8 and F9, and one for the self-check fixes.
F1, F6, F8 and F9 edit the same files, so they share a commit.

A gate pass over the round's comments then broke the wrapped and over-long comment lines at sentence boundaries.
It touched the doc comments of `LymBinary` in both engines, both engine `doc.go` python paragraphs, the two live-check map comments,
the standalone test's header and doc, and one comment in the Claude hook integration test.
Only comment text changed.

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.
All four forks returned a report.

- fix-code-correctness, LOW, the claudeengine package doc still named `standingDeniesFor(cfg.RefuseRaw)`: fixed.
- fix-code-correctness, LOW, Claude `Prepare` resolved the lym binary after writing `prompt.md`: fixed, the resolution now comes before the first write.
- fix-code-correctness, no finding, the webstercli smoke helper: merged into the affected-set finding below.
- fix-consistency, LOW, the verb's `Long` help text still described only hub and standalone: fixed, it names the Bolt checkout's `_lym`.
- fix-proposal, MEDIUM, test binaries of other packages that reach a real `Prepare` had no stand-in: fixed, see F1.
- fix-proposal, LOW, the orch launch loads the hub layers whatever `BoardLymPresent` says, so for a hub damaged at `_lym` with a state directory the hook and the orch launch read different config:
  not changed.
  The fork builds the case and agrees it is narrow, since the two differ only when a state directory also exists.
  The only change that aligns the two moves orch's wiring, outside this review's scope, and reverting F4 reopens the mismatch for standalone-capable launches that F4 closes.
  I judge it a design call for the parent and name it here without a disposition of dispute or defer.
- fix-affected-set, LOW, webstercli and the other live-tier smoke packages: fixed, same change as the fix-proposal MEDIUM.

Gate: tier 0 over the touched and stand-in packages passed, and tier 1 passed over the engine, shuttlecli, shuttleengines, orchcli and shuttleparity packages.
Tier 0 ran again over the smoke packages after the last change and passed.
