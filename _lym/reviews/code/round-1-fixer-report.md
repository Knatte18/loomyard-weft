---
dispositions:
  - id: F1
    disposition: deferred
    reason: "Partly deferred on the parent's ruling (creel #2683, option B): Go comments and package docs that name a template key bare stay as they are in this run, and the parent files the sweep as a follow-up note. Messages, flag help, docs, contracts and the comment sites the review lists are fixed."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "No seam in export_test.go: an unparseable loom.yaml in the pair's records fails the walk after the hub-wide commit through the real walk, so TestEnsure_MovesOldConfigKeysAndMailsTheMoves asserts the second notice with no test-only hook."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The proposed first row cannot pass: area={model: x} is not written, since the template has no leaf at area and the set is refused as an unknown key. The row asserts that refusal, and the doc comment on Set, which promised the write, now says the template judges the pair (internal/configengine/set.go)."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Code review round 1 — fixer report

## What changed

- **F1** — config keys are named `<module>.<dotted path>` in Go messages, log lines, flag help, `docs/`, `contracts/` and the comment sites the review lists.
  Most render through `configengine.KeyName` behind a per-package helper; a few help texts in packages with no helper write the name out.
  `lym config … --set` command forms keep the CLI form.
  Three strings stay bare on purpose:
  the `exceeded timeout_min` run-log marker, which `verifytree` parses back;
  recipe fields such as `max_bounces` and `verify_source`, which are no config key;
  and `shellguard`'s declared exception.
- **F2** — the reconcile help and `configcli`'s package doc say every reconcile moves or drops a renamed key;
  the docs and `preflightshed` say which verbs run the walk.
- **F3** — a walk that fails in a pair after the hub-wide commit is asserted to mail its moves.
- **F4** — one tier1 test for each of the five sites.
  `beam dobby`'s wiring closure is now the named `wireDobby`, so a test can reach it without running a dobby.
  The merge-in test uses a conflicting merge, so it fails when the reconcile runs after `MergeIn`.
- **F5** — an alias a move leaves ahead of its anchor, or without it, is written as a copy of the value.
- **F6** — both move tables are compared with literal maps written from D1.
- **F7** — two `TestSet` rows; see the disposition.
- **F8, F9** — `MoveSettings` drop rows and the two failure-path rows.
- **F10, F11** — keep marks.
- **F12** — `TestKeyName` deleted, with its `Loosened-Check` trailer.
- **F13** — `MoveKeys` and `MoveSettings` take the prefix as one key per level, so a dotted profile name moves.

Last full gate: `lym gate test --tier 0 --tier 1` over the diff scope, no failures.
Tier 2 and higher were not run.

## Gate repair

The round's gate refused comment lines that wrapped at a non-sentence break or reached 120 characters.
Each line it listed is now split into sentences or clauses, including two in `internal/shedverbs` test files.
The change touches comments only, committed under F1.
`lym gate test --tier 0 --tier 1` then passed over the packages holding those files.

## Reach repair

The reach step then listed changed functions no mapped test reaches (`.lym/gate/reach.json`).
Two had a coverage gap and now have a tier0 test, committed under F1 and passing:
`gateslot.TimeoutKeyName`, and `gatecli`'s `tierMemoryCapLine`, whose only test derived its expected line from the function.
No test was added for the others:

- Already tested, in tests the maps do not credit:
  `beamcli.wireDobby` (`TestWireDobby_StaleStampReconcilesTheHubBase`),
  `configcli.reconcileEnvelope` (`reconcile_integration_test.go`, the `moved` field),
  `driftengine.missingDirRefusal` (`capture_test.go`, `compare_test.go`),
  `gatecli`'s `testMapRun.prepare` (`testmap_integration_test.go`) and `tierMemoryCapRefusal` (`tierrun_memorycap_linux_test.go`),
  and `heddleCLI.requireOverrideOn` (`heddlecli/cli_integration_test.go`).
- Only a comment, help text or log wording changed:
  `gatecli.allInertSince`, `hardencli.commitSandboxConfig`, `heddlecli.lymOverrideName` and `ideengine.filesToOpen`.
- **Left untested:** `gatecli.newCardFixer`, whose unresolvable-model refusal had no test before this round either.
  The round changed its wording only, and reaching it needs a hub with a card gate fixer configured.

## Self-check

- fix-code-correctness 1, MEDIUM (tackler and arbiter texts could name the wrong file): fixed as proposed; both name the run's module key, else the `shed` key.
- fix-code-correctness 2, LOW (no test pins that `runDobby` passes `wireDobby`): disputed, every fix worse.
  Pinning it means running `runDobby` inside the test, which reaps the test process's inherited children and starts a dobby (`internal/beamcli/cli.go`, `runDobby` and `reapInheritedChildren`).
- fix-consistency 1, MEDIUM (messages and help naming single-word keys): fixed as proposed, every site it lists.
- fix-consistency 2, LOW (comments quoting the old landing-notice text): fixed as proposed.
- fix-consistency 3, LOW (two comments crediting only start verbs with the walk): fixed as proposed.
- fix-proposal 1, BLOCKING (same as fix-code-correctness 1): fixed as proposed.
- fix-proposal 2, LOW (reconcile help missed "outside `lym config`"): fixed as proposed.
- fix-affected-set 1, MEDIUM (messages naming keys by file): fixed as proposed, every site it lists.
  Its search and mine were pattern-based, so a further message naming a single-word key may remain.
- fix-affected-set 2, LOW (same as fix-consistency 3): fixed as proposed.

## Disputed

None among the review's findings.

## Deferred

- **F1, in part** — Go comments and package docs that name a template key bare are unchanged, apart from the sites the review lists and those quoting changed text.
  The parent ruled this (creel #2683) and files the sweep as a follow-up note.
  A search for underscored key names alone found about 525 such comment lines in 199 files before the self-check;
  single-word keys come on top.

## Rulings

None on a `decision`-class finding.
The parent's scope answer on F1 is recorded under Deferred.
