# Plan gate: no false caller findings, reviewer fixes the gate accepts, nested fixture modules, no plan-made parent decisions

`plan-gate-quality` · Task · bug, loom, quarry · running · Discussion-Write

## Goal

The plan gate and the plan roles agree: its checks raise no false positives, the reviewer proposes no fix the gate blocks, gates can run against nested fixture modules, and a plan agent cannot record a decision as the parent.

## From `caller-uncovered-false-positives`

Imported from [issue #461](https://github.com/Knatte18/loomyard/issues/461).

**What happened:** Adding re-sign arrows to a plan card made the `caller-uncovered` check emit 75 informational entries, all false positives.
Every file that names a type counted as a caller of a re-signed method on a different type that happens to share the name, and any `.start(` call anywhere counted as a caller of a re-signed `start` method on one specific type.
The real callers had to be separated out by grep.

**Why it is abnormal:** The matching is by name only, not by receiver type, so the noise can hide a real uncovered caller.

**What to do:** Resolve callers by receiver type (or at least qualify the match by package and receiver) in `caller-uncovered`, and rate-limit or collapse name-only matches.

Provenance: task `webster-dead-ends`, note `burler-plan-r4.md`.

## From `caller-scan-import-path`

`plan-glyph-precision` added the `caller-uncovered` plan-gate check (D5).
Its caller scan matches by package clause name, so it cannot see a caller that imports the package under an alias or a dot import, nor a caller in a file an earlier card creates.
Both are accepted bounds in that task's decision record.
Resolving imports by path would close the first gap; worth building only if planners hit it.

### Seen in webster-dead-ends, 2026-10-08

Its plan gate reported 75 informational `caller-uncovered` entries on one card, all false positives by the fixer's grep:
every file naming the `batcher.Batch` interface type was read as a caller of the re-signed concrete method `costBatcher.Batch`,
and unrelated `.start(` calls as callers of `peakAccumulator.start`, matched by method name alone.
For a method the finding is informational, so it cost no round, but the noise hides a real uncovered caller.
Resolving a method call's receiver type would remove both classes.

## From `plan-review-redundant-pairing`

Imported from [issue #460](https://github.com/Knatte18/loomyard/issues/460).

**What happened:** A plan-review fix said to keep a file target beside a member target of that file on the same card, on the premise that a file and a member of it on the same card do not overlap.
The plan gate's `redundant-file-target` check blocks exactly that pairing, so applying the fix as written failed validation, and both cards had to drop the file target again.

**Why it is abnormal:** The reviewer proposes a fix the gate rejects, which costs a wasted fix round.
A reviewer with the gate's check list in view would not suggest it.

**What to do:** Give the plan reviewer the gate's check list (at least `redundant-file-target`), or have its fixes validated against the gate before they are handed to the fixing agent.

Provenance: task `webster-dead-ends`, note `burler-plan-r4.md`.

## From `plan-gate-nested-module`

Imported from [issue #451](https://github.com/Knatte18/loomyard/issues/451).

### What happened

Two cards in one task had gates running `go test -tags integration` on package paths under a `testdata/` fixture directory.
The same card (or an earlier one) creates a `go.mod` in that directory, which makes the fixture its own module.
The main module then no longer contains those packages, so `go test` reports `main module does not contain package ... [setup failed]` and the gate's `&&` chain cannot pass as written.
The implementing agent worked around it by running `go vet` and `go test -tags integration ./...` from inside the fixture module, and wrote a friction note each time.

A related symptom in the same batch: the test-budget guard (`cmd/lyx/testdata/test-budget.yaml`) needed a package row raised on several cards, and that file was on no card's target list.

### Why it is abnormal

The plan was accepted through the plan gate although its gates were unsatisfiable by construction.
Plan validation does not check that a gate's package paths stay inside the module that the plan's own cards create or change.
Cards also do not predict the test-budget row they will cross, so each agent edits an untargeted file.

### What to do

- At plan-write or plan-review time, flag a gate command whose package path lies under a directory that a card makes a nested module (a `go.mod` in a card target), and require the gate to run inside that module.
- Consider listing `cmd/lyx/testdata/test-budget.yaml` as an implicit target, or surfacing the budget row in the card template, when a card adds tests.

Provenance: task `plan-glyph-precision`; notes `01-glyphchain-plan-gate.md`, `07-redundant-package-target.md`.

## From `plan-agent-parent-decision`

Imported from [issue #453](https://github.com/Knatte18/loomyard/issues/453).

### What happened

In the `shuttle-send` run, the Plan-Write agent recorded a decision as the parent:

```
lyx loom decision add --by parent --title "PATTERN-shuttle-stop names the loom driver exception" ...
```

The Plan role writes its own output files only; a design call outside the decided scope goes to the parent.
`lyx loom decision add` accepted `--by parent` from inside the plan agent's session, so the record reads as the hub's decision though the hub never made it.

### Expected

`decision add --by parent` refuses when it runs inside a lyx-spawned agent session other than the parent's, or the Plan stencil's deny hooks block it, so a plan agent's scope call reaches the parent as an escalation or an open question in the plan.

### Context

The decision itself was narrow and the hub would have taken it (it names loomcli's raw removal of the loom driver's strand as an exception in the new PATTERN entry).
The defect is the forged authorship, not the content.
