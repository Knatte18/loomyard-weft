```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop or recovery step a card adds: the next step it names, and whether that step can succeed from the state the refusal leaves.

## Per refusal: next step, progresses or not

**Fabric (cards 2–8)**
- Card 2, `MarkBeam` on a record-less pair → `lyx fabric reconcile`: **does not progress** for a record file deleted in a present records worktree (finding 2). It progresses when the whole records worktree is missing.
- Card 4, unknown parent or the prime's name → `lyx fabric pairs`: progresses for an unknown slug. For the prime it is weak: the step that works is dropping `--parent`.
- Card 4, detached or unborn parent → **no next step named** (finding 4).
- Card 4, parent on another branch → `lyx fabric checkout <branch>`: progresses.
- Card 5, warp branch checked out elsewhere → `lyx fabric checkout`: progresses.
- Card 5, records branch checked out in another records worktree → **names only the worktree, no next step** (finding 3).
- Card 6, `--local` with `--parent` or `--adopt-local` → drop the flag: progresses.
- Card 8, step-3 dirt outside the pathspec → commit or discard, then re-run `lyx beam promote`: progresses.
- Card 8, missing record → `lyx fabric reconcile`: same as card 2, **does not progress** for the planted input.
- Card 8, root pair → **no next step named, and leaves a dead end** (finding 1).

**Board (cards 9–13)**
- Card 9: forbidden project field → drop it; `depends_on` naming a project → its members or `lyx board set-deps`; a `P` type label → rename it in `board.yaml`; `lyx board promote` of a project → `lyx beam create`. All progress.
- Card 10, the membership rules → correct the slug or drop it: progresses.
- Card 10, `project` change with a local pair → `lyx batten cancel <slug> --remove`: progresses.
- Card 10, rename of a beam → `lyx board upsert` of its title: progresses.
- Card 10, merge removing a still-named project → move the named entries first: progresses.
- Card 11, a `beam: true` payload → `lyx beam create` / `lyx beam promote`: progresses.
- Card 11, a `beam: false` payload on a beam → the same verbs: **wrong direction** (finding 6).
- Card 13, slug rule → the suggested admitted slug: mostly progresses, but **the suggestion is not checked** (finding 5).

**Orch and batten (cards 19–28)**
- Card 19, `refuseNonPrime` → `lyx beam create` / `lyx beam promote`: progresses through their own refusals.
- Card 19, beam owned elsewhere → `lyx adopt <slug>`, which card 23 admits from a beam: progresses.
- Card 19, landing-held beam → `lyx batten status <slug>`: informational, adequate.
- Card 23, non-beam pair → nearest beam or the prime: progresses.
- Card 23, run-held beam → the beam's parent worktree: progresses.
- Card 23, self-pair → the parent or the prime: progresses.
- Card 27, cleanup skip → `lyx batten cancel <slug>`: progresses, including the records-HEAD case the card fixes.
- Card 28, absent beam pair → `lyx beam create <beam>` or `lyx adopt <beam>`, then `lyx batten run <slug>`: progresses.
- Card 28, run-held beam → `lyx batten status <beam>` and `lyx batten cancel <member>`: progresses; the cancel runs from the prime.

**Landing and beam verbs (cards 30–36)**
- Card 30, describe validation → `lyx batten cancel <slug>`, then `lyx board upsert`, then `lyx beam land`: progresses. The cancel lifts the run lock, and the start guard admits a marked beam pair.
- Card 34, absent entry, note, task, done project, beam owned elsewhere, absent parent beam → each named step: progresses.
- Card 35, never-started task → `lyx board upsert` plus `lyx beam create`: progresses.
- Card 35, task another machine owns → `lyx adopt`: progresses.
- Card 35, started task with no local pair → `lyx adopt <slug>`: **does not progress** when no pair was ever pushed (finding 7).
- Card 35, past parent-side merge → `lyx batten run`; open dependent → `lyx board set-deps`; end-only step 1 → re-run once origin answers; start-guard `Promoted` → `lyx beam promote`: all progress, bar the card-8 root-pair loop.
- Card 36, open member → land it, cancel `--remove`, or move `project` off: progresses. For a member with no pair while the beam is claimed, only the third option works, and it does.
- Card 36, child pair → `lyx batten run` or `lyx batten cancel --remove`: progresses (cancel tolerates an entry that is not on the board, `cancelcmd.go:438`).
- Card 36, live orch → `lyx orch stop` in the beam: progresses; card 19 refuses only `start`.
- Card 36, `land` from inside the beam → the parent worktree: progresses.
- Card 36, unreadable record → `lyx fabric reconcile`: progresses, since reconcile recreates a missing weft worktree (`fabric.go:270-273`).
- Card 36, grouping or pairless beam project → `lyx beam create` or `lyx adopt`: progresses.

## Findings

1. **MEDIUM, consistency, card 8 (`ResetRecordsToParent` step 1), root-pair refusal.** The refusal names no way forward, and inside promote it leaves a dead end.
   - On the promote path, steps 1–3 have already run when step 4 refuses: the run is ended, the mark is written and the claim is cleared.
   - A re-run of `lyx beam promote` refuses again at step 4, and card 35's start-guard `Promoted` refusal names `lyx beam promote`, so nothing advances.
   - The input that reaches it is a task pair made by `--adopt-local` over the default branch (card 5).
   - The fix: card 35 refuses a root pair before step 1, or the card-8 refusal names a step that progresses.
   - Evidence: `_lyx/plan/08-weft-reset-executor.md:20`, `_lyx/plan/35-beam-promote.md:46-49`, `_lyx/plan/05-add-adopt-local.md:25`.

2. **MEDIUM, consistency, cards 2 and 8, missing origin record.** Both refusals name `lyx fabric reconcile`, which recreates a missing weft worktree and re-points junctions but never rewrites a deleted record file in a present records worktree.
   - That deleted file is exactly the input card 8's test plants: "a pair whose origin record the test deletes".
   - The next step does not progress for that state. It needs to name restoring the file from the records branch, or the refusal has to tell the two causes apart.
   - Evidence: `_lyx/plan/02-origin-beam-mark.md:21`, `_lyx/plan/08-weft-reset-executor.md:29`, `internal/fabriccli/fabric.go:270-273`, `internal/fabricengine/reconcile.go:158-163`.

3. **MEDIUM, consistency, card 5 (`Topology.add`), records branch checked out in another records worktree.** The refusal "names that worktree" and no next step.
   - D6 and the constraints require a way forward on every new refusal.
   - `lyx fabric checkout` acts on the warp side, so its sibling refusal's step does not carry over.
   - Evidence: `_lyx/plan/05-add-adopt-local.md:24`, decision record D6 and Constraints.

4. **LOW, consistency, card 4 (`resolveAddParent`), detached or unborn parent.** The card says each refusal names a way forward, but this bullet names none; D5 groups it with `lyx fabric checkout`.
   - Separately, the refusal of the prime's own name points at `lyx fabric pairs`, while the step that works is dropping `--parent`.
   - Evidence: `_lyx/plan/04-add-from-parent.md:27-29`.

5. **LOW, consistency, card 13 (`checkNewSlugs`), suggested slug.** The suggested admitted slug (offending suffix dropped, or `-task` appended) is checked neither against the rule nor against the board.
   - A slug with the records suffix twice still fails after one suffix is dropped.
   - When the suggested slug already exists, following the suggestion with an upsert edits the existing entry instead of making a new one.
   - Evidence: `_lyx/plan/13-board-slug-seam.md:24`.

6. **LOW, consistency, card 11 (`checkBeamPayload`), `beam: false` on a beam.** A payload clearing the flag gets the same text, naming `lyx beam create` / `lyx beam promote`, which cannot clear it.
   - D3 says nothing sets the flag back, so the refusal should say the flag stays until `lyx beam land`.
   - Evidence: `_lyx/plan/11-board-beam-flag.md:17`.

7. **MEDIUM, decision, card 35 (`promote`), started task with no local pair.** The refusal names only `lyx adopt <slug>`, which adds a pair only when the branches are on origin.
   - A task that was claimed but stopped before Worktree-Create, or whose branches never reached origin, has no pair anywhere.
   - Adopt then records the owner and adds nothing, so promote keeps refusing.
   - The step that progresses is `lyx batten run <slug>` to make the pair, or a cancel and a new project through `lyx beam create`. D10 words the refusal this way too.
   - Evidence: `_lyx/plan/35-beam-promote.md:37`, `_lyx/plan/37-adopt-beam.md:7`, decision record D10.

## Tool calls

1. `grep` of the reconcile docs and `fabricengine` reconcile functions: to see whether `lyx fabric reconcile` restores a missing records worktree or origin record; the manifest had not read reconcile.
2. `grep` of `fabricengine` for restore wording and the `lyx fabric reconcile` call sites: same question, to find the verb's definition.
3. `sed` of `fabriccli/fabric.go:396-416` plus `grep` for the reconcile `Long`: to read what the verb claims to repair.
4. `sed` of `fabriccli/fabric.go:268-292` and `fabricengine/reconcile.go:140-164`: the reconcile verb's own description, which confirms it recreates missing weft worktrees only.
5. `grep` of `battencli/cancel.go` and `cancelcmd.go`, plus `adoptcli/adopt.go`: to check cancel on a pair with no board entry (card 36's child-pair step) and adopt on a task with no pair on origin (card 35's step); neither was in the manifest.
