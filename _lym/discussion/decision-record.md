## Goal

```markdown
## Seen

`harden-fabric-single-repo-verbs` round 4 drove the cases round 3 left undriven and approved, but could raise none of what it found as a finding:
its incremental scope held only round 3's gate fix (`internal/gatecli`, `internal/gateslot`, `internal/loomcli`), and an incremental round admits only BLOCKING findings on changed hunks.
It recorded them as observations; they are copied below from `round-4-review.md` in the run's archived records.

- **O1 — MEDIUM, design, CONFIRMED.**
  Location: `internal/fabriccli/fabric.go:673` (the reconcile verb's `NewBolt(boltDir).PullThenCommitWritten`), with `internal/fabricengine/clone.go:225` and `:264` ("hub already exists").
  Scenario (P8, twice): `lym fabric clone <url> --into <dir>` killed once its records store exists and before `_bolt` does.
  `fabric status` and `fabric pull` then print `not initialized here; run "lym fabric reconcile"`;
  typed, `lym fabric reconcile` fails with the raw `gitrepo: git dir of …/_bolt: dotgit: not a git repository …`;
  a plain re-clone answers `hub already exists at …` with no way forward. Only `lym fabric clone <url> --into <dir> --reset`, which no output names, rebuilds the hub.
  Fix: in the reconcile verb, before the Bolt pull, refuse a hub whose `BoltDir` holds no repository with a way forward naming `lym fabric clone <url> --into <parent of the hub> --reset`.
  Make `CloneHub`'s "hub already exists at %s" name the same `--reset` re-run.
  Add a regression test that removes `_bolt` from a hubforge one-repository hub and asserts both messages.
- **O2 — LOW, scope, CONFIRMED.**
  Location: `internal/fabricengine/mergeguards.go:57`.
  Scenario (P7): records store's `remote.origin.fetch` unset; `lym fabric merge y1` stages the merge with `ok: true` and names no layout gap,
  so the weft SHA in the merge record comes from stale tracking refs. Every other verb driven names fabric's refspec way forward.
  Fix: call `originNamespaceAt(f.records path)` before `f.records.Fetch()` in `resolveMergeSources` and log its error with the weft fetch's warning, or return it as the verb's refusal.
  The same gap stands on non-fabric paths: `pull.go:186` and `recordsgit.go:346` via `FastForwardPair`, `codetipsettle.go:21,43` via `DobbyRuns`, `bolt.go:162` via `lym machine name`.
- **O3 — LOW, design, CONFIRMED.**
  Location: `internal/fabricengine/archive.go:294` (`pushLocalArchiveTags`).
  Scenario (P3): a records commit kept under a local `archive/t4/<sha>` tag, as `DropLocal` keeps one with "the tags stay local", is pushed to origin by a later `lym fabric remove t4 --remote`.
  No verb output names the push (the job is handed off), and `lym fabric remove --help` says nothing of it.
  Fix: decide whether `DropLocal`'s tags may leave the machine.
  If yes, name in `lym fabric remove --help` and `cleanup --help` that `--remote` pushes the slug's local archive tags origin lacks, and correct `DropLocal`'s doc.
  If no, give `DropLocal`'s tags a prefix `pushLocalArchiveTags` does not list.
- **O4 — LOW, scope, CONFIRMED.**
  Location: `internal/fabricengine/cleanupremotewarp.go:302` (`remoteTagRefs`, the records-side archive listing `cleanup` reads).
  Scenario (P5): an earlier binary's plain `refs/tags/archive/t5/<sha>` on a one-repository origin is named by no `status`, `cleanup`, `prune` or `reconcile` output, before or after the branch is discarded.
  No commit is lost: the discard pushes its own namespaced tag first.
  Fix: in `cleanup`'s read of a one-repository origin, list plain `refs/tags/archive/*` and `refs/tags/archive-owner/*` refs and report each with a way forward that pushes it under `refs/lym/code-tags/` and deletes the plain ref.
- **O5 — NIT, consistency, CONFIRMED.**
  Location: `internal/fabricengine/remoteteardown.go:187` (`discardRerunWayForward`).
  Scenario (P1): a killed discard's record names "run the command that discarded the task branch again", no command an operator can type; the re-run that works is `lym batten cancel <slug> --remove`.
  Fix: carry the discarding command in the job and the pending record, as the removal and cleanup kinds name theirs, and print it.

Not driven in round 4: the owner-archive push path, the hand-over and code-tip reset paths, and a lease that loses.

## Idea

Fix O1 to O5 as each proposes; for O3, the operator decides whether a tag `DropLocal` keeps may leave the machine.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): O3: local archive tags may leave the machine

Decision: YES: keep pushLocalArchiveTags as is. Say in lym fabric remove --help and cleanup --help that --remote pushes the slug's local archive tags origin lacks, and correct DropLocal's doc.

Rationale: The smallest change: behaviour stays, and the docs name the push the code already makes.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - catchup-layout-error-misreported-unreachable (class design, severity MEDIUM, open in rounds 1)
  - cleanup-help-spells-records-namespace (class design, severity BLOCKING, open in rounds 1)
  - o2-layout-reads-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
