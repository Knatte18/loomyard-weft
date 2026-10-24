# Fixer report — plan, round 1

## Changes

- F1 (MEDIUM), card 23, fixed as proposed: `stepNonPrimeRefusal` and `stepRecordsPrimeRefusal` are on the Edit list, and the Intent says what each asserts of the new text: the task-pair refusal names the task worktree and the prime, and the records-sibling refusal is the drivable-worktree refusal wrapped in "this verb runs from the hub's prime worktree or a beam".
- F2 (MEDIUM), card 19, fixed differently: `refusalCase` is on the Edit list and asserts the shared opening "lyx orch runs from the hub's prime worktree or a beam", which both orch refusals now carry. The proposed change had `refusalCase` assert the beam verbs, but it did not make that change: `TestOrchIntegration_Refusals` runs `refusalCase` for the prime's records sibling too (`internal/orchcli/refusal_integration_test.go:31-39`), and that refusal is the pre-run's wrap of the drivable-worktree refusal (`internal/orchcli/cli.go:145`), which names no beam verb. The beam verbs are asserted by `TestRefuseNonPrime`'s non-prime row. The card also rewrites `Command`'s Long "Every verb runs from the hub's prime worktree only" (`internal/orchcli/cli.go:120`).
- F3 (MEDIUM), card 35, fixed as proposed: the Intent says the `Promoted` check runs whether or not the status file is found, unlike the left-pair check gated on `!found` (`internal/battencli/startguard.go:78`), and `TestStartGuard_Refuse` gains the marked-pair row both with no status file and with one found.
- F4 (MEDIUM), card 9, fixed with a narrower wording: the `order`/`beam` rows moved from `TestKindDependencyRules` to `TestFabricReplay` rows refused through `wantRefused`. The proposed note said "an upsert never reaches this branch". It did not keep that wording, because from card 10 the allowlist admits `order` (card 10, "`UpsertPayloadKeys` admits `project` and `order`"), so a later upsert of `order` on a task reaches `normalizeTask`. The note says no upsert reaches the branch at card 9, and that from card 11 `checkBeamPayload` refuses an upsert's `beam` first.
- F5 (MEDIUM), card 23, fixed as proposed: the `internal/battencli/cli.go` Extends entry is gone, `internal/battencli#Command` and `internal/battencli#battenCLI.cleanupCmd` are under Edit, and the Intent says their prime-only text is rewritten.
- F6 (MEDIUM), card 24, fixed as proposed: a Bound sentence says the session holds no orch strand, so `OrchLive` and the readers of `HubOrchs` (card 21's `Orchs`, one per worktree recording a strand) find no orch there. It ends at the beam's pair teardown (PATTERN-pair-teardown), and a later `lyx orch start` in the beam takes it over through `Up` (`internal/orchcli/cli.go:195`).
- F7 (MEDIUM), cards 2 and 8, fixed as proposed and then widened by the self-check: a missing records worktree names `lyx fabric reconcile`. A record file missing from a present records worktree names restoring it from the records history: `git checkout HEAD -- <record path>` while the deletion is uncommitted, else from the commit before the one that deleted it. The test rows in both cards plant a deletion and assert the restore text.
- F8 (MEDIUM), card 5, fixed differently after the self-check: the refusal and `TestAdd_AdoptLocal`'s row name, as the way forward, switching the other records worktree off the branch (back to its own pair's records branch, or `git switch --detach` when it belongs to no pair) and re-running the add. The first edit followed the proposal's example and named removing that worktree. Removing it would tear down part of a live pair outside `internal/pairteardown` (PATTERN-pair-teardown), and `git worktree remove` would fail from the warp hub.
- F9 (LOW), card 24, fixed as proposed: `battenVerbTexts` and `Command` are under Edit, and the Intent says both now name the run's parent worktree's reed session.
- F10 (LOW), card 35, fixed as proposed: there is now a refusal before step 1 for a task whose pair's origin record has an empty `ParentBranch`, naming it a root pair. It is scoped to a task, so a beam that `lyx beam create` marked over an adopt-local root pair can still finish step 6. `TestPromote` reaches it with no step run, and `ReadOriginFor` is added to Uses.
- F11 (LOW), card 4, fixed as proposed and refined by the self-check: the detached or unborn parent refusal names `lyx fabric checkout <parent branch>` in the parent's worktree. The prime-name refusal is its own bullet, naming dropping `--parent` and running the add from the prime.
- F12 (LOW), card 13, fixed as proposed: `checkNewSlugs` runs the suggestion through the seam and against the after entries' slugs, and offers none when it fails. The refusal then names choosing another slug as the way forward. `TestSlugRule` gains the collision row.
- F13 (LOW), card 11, fixed with a wording that matches card 36: the `false` refusal says the flag stays set for the beam's life, which ends when `lyx beam land <slug>` lands it and marks the project done. Landing marks the project done (card 36) and nothing clears the flag (D3), so "until the beam lands" alone would imply the land clears it. `TestUpsertFieldAllowlist` gains the row.
- F14 (LOW), card 23, fixed as proposed: the self-pair sentence says a beam reaches the refusal before its landing claims, giving `lyx batten run <beam>` from inside the beam as the example, and says the run-held refusal fires first during the landing.

`lyx loom validate-plan` reports `ok: true` after every change.

## Self-check

- fix-consistency, LOW, card 23's guard bullet does not say the refusal names the acting worktree: fixed as proposed (the bullet now names it, as today's refusal does).
- fix-consistency, LOW, card 35's Uses lacks an origin reader: fixed as proposed (`internal/fabricengine#ReadOriginFor`).
- fix-affected-set, LOW, card 23 leaves five more prime-only help texts: fixed as proposed (`lockCommand` added to Edit, and the Intent names `lockCommand`, `cancelCmd`, `overviewCmd`, and adopt's and cleanup's `Command` Long).
- fix-affected-set, LOW, card 19 leaves the integration file's prime-only comments: fixed as proposed (`TestOrchIntegration_Refusals` added to Edit, and the Intent rewrites its doc, `refusalCase`'s doc and the file header).
- fix-claims: no findings.
- fix-proposal, MEDIUM, `git checkout HEAD --` fails once a records commit has swept up the deletion: fixed as proposed (cards 2 and 8 also name restoring from the commit before the deleting one).
- fix-proposal, MEDIUM, card 5's `git worktree remove` may tear down a live pair and fails from the warp hub: fixed as proposed (switching that records worktree off the branch instead).
- fix-proposal, MEDIUM, card 4's "drop `--parent`" forks from the acting worktree when run inside a pair: fixed as proposed (the refusal also names running the add from the prime).
- fix-proposal, out-of-scope note duplicating fix-consistency's card 23 finding: merged into that finding.

## Disputed

None.

## Deferred

None.
