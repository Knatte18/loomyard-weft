---
format: 5
approved: true
language: go
---

# Plan: publish-recovery

A loom run that stops at Publish or Finalize recovers through one named lyx route.
Publish's verify failure names the Webster-Burler goto when its failure record was written (D1);
Publish and Finalize abort and redo their own parked merge-in before the clean check, stopping a live conflict session of the run first (D2);
`lyx loom resume` and `lyx loom start` let that leftover through (D3);
Finalize's config notice compares three points so a parent-side change never reads as a task edit (D4);
the attach probe's tables pin a `stopped` young run dir as respawn-eligible (D5);
and `lyx loom decision add` refuses a task session's label (D6).
The plan stencil's scope-addition instruction, the orch role's escalation list, the refusal-spec rows and the package docs land in the cards that change the behavior.

## Card Index

1 — midmerge-record-fields — MidMerge answers a parked record's verb, source and SHAs
2 — config-changes-three-way — ReadConfigChanges intersects the fork-point diff with the parent-tip diff; the notice names the parent tip
3 — publish-verify-failed-fix-route — a recorded Publish verify failure names the Webster-Burler goto; spec row and orch escalation item
4 — conflict-role-export — export mergeresolve's conflict role constant; agentname recognises a role's numbered forms
5 — landing-pre-merge-clear — Publish and Finalize stop their run's conflict session, abort their own parked merge-in and merge afresh; other merge state stops Stuck with its way forward
6 — resume-start-own-leftover — resume and start proceed over the row's own parked merge-in at Publish or Finalize
7 — decision-add-refuses-task-session — decision add refuses both labels from a named session other than the recorded parent; plan stencil asks the parent instead
8 — stopped-young-run-dir-rows — attach and probe tables pin a stopped young run dir as respawn-eligible

## Shared Decisions

### Decision: nil-is-absent seams

- **Decision:** every seam added to `landingshed.Deps` follows `CommitStatus`'s nil-is-absent convention: a nil seam skips the step it serves, so every existing struct-literal test runs unchanged, and `internal/loomcli`'s drift guard keeps the production wiring filled.
- **Rationale:** the package's told-value surface already works this way, and the pre-merge step must never run against a fake-free fixture.
- **Applies to:** card 5

### Decision: one merge-state answer for both consumers

- **Decision:** the parked record's verb, source and SHAs travel on `fabricengine.MidMergeState`, read through the existing `MidMerge` probe, rather than through a second probe or the resolver's fabric seam; the landing producers take the probe as a told closure, and the loom verbs call it as today.
- **Rationale:** D2 and D3 need the same two facts, verb and source, and one answer keeps the two decisions reading the same record.
- **Applies to:** cards 1, 5, 6

### Decision: every refusal lands with its row and its test

- **Decision:** each card that adds a Stuck reason or a refusal adds its refusal-spec row and the test row that reaches it on the same card, and its way forward names only verbs the agents' settings allow.
- **Rationale:** `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`.
- **Applies to:** cards 3, 5, 6, 7

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
