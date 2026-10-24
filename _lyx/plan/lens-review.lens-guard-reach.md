```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal and test row the plan adds can be reached from a real entry point, and whether a guarded site recurs unguarded.

1. BLOCKING — Card 24 (`windowSite`), D8 window site; also card 36's landing start and card 37's adopt resume.
   - Summary: `windowSite` opens the window in a beam's reed session whenever the beam's worktree is present, but a beam with no orch normally has no reed session, so `--window` fails there.
   - Evidence: `internal/reedengine/window.go` doc of `OpenWindow` says it "returns an error wrapping ErrNoSession when the worktree has no reed session, and creates nothing". `internal/reedengine/lifecycle.go:1299` `requireSessionLocked` refuses when `hasSession` is false.
   - Card 24's site rule reads only "the named beam's worktree is absent here". `lyx beam create` starts no session, and D9 makes the orch optional.
   - So `lyx batten run <member> --window` for a member of an orch-less beam, `lyx beam land` of a sub-beam whose parent beam has no session, and adopt's `ResumeInWindow` all refuse.
   - `TestWindowSite` has no row for a beam that is present but has no session.

2. BLOCKING — Cards 33, 36 and 9 (D1, D11): a project with no local pair is never refused at batten's start.
   - Case 1: `lyx batten run <project>` for a grouping project (`beam` false). Card 9's `writeOwners` skips the claim silently. `battencli#battenCLI.claimBoardEntry` (`internal/battencli/owner.go:168-185`) refuses only on `write.Lost`, so the run goes on. Worktree-Create then forks a pair for the grouping project, and card 33's `boardRecipe` seeds `beam` on it.
   - Case 2: the same for a project whose `beam` is true but whose pair is absent, such as a create stopped after step 1 or a hand-set flag. Card 36's start-guard changes only relax refusals when a marked pair exists, and nothing refuses a missing pair. `ParentBeam` of a top-level beam is `""`, so card 28 calls a plain `Add` from the prime. The `beam` recipe then lands an empty pair, and the teardown settle marks the project done.
   - The intended guards would be D1's "only a task is claimable, bar the landing run of a project whose beam is true" and card 36's "a landing forks no pair". No card prescribes either for this state, and no test row reaches it.

3. MEDIUM — Card 5, D6 default-branch row: the test row cannot reach the root-pair branch as written.
   - The warp default branch is checked out in the prime worktree. Card 5's own pre-flight refuses "a warp branch checked out in another worktree".
   - So `TestAdd_AdoptLocal`'s row "adopting the default branch records an empty parent branch and worktree" hits that refusal unless the row first moves the prime off the default branch. The card names no such setup.
   - The branch also needs the slug to equal the default branch's name, `BranchPrefix + slug`.

4. MEDIUM — Card 28, `beamForkPrecondition` order against the landing's Worktree-Create skip.
   - Card 28 runs the precondition "before the add", using `ParentBeam` of the entry's own `project`. Card 36 makes a beam landing reach Worktree-Create's `taskWorktreeComplete` skip.
   - For a sub-beam's landing, `ParentBeam` names the parent beam. If the parent beam's pair is absent here, or its entry is run-held, the precondition stops the landing even though the sub-beam's own pair is present and needs no fork.
   - Neither card says the precondition runs only after the completeness probe finds no pair. `TestBeamLanding_ReachesWorktreeCreateSkip` has no sub-beam row.

5. LOW — Card 35, D10 promoted-mark guard: the guarded state recurs on a path the guard does not cover.
   - The `Promoted` seam is added only to batten's start guard. After promote's step 2 or 3 the pair still holds loom's seed and records until step 4's reset.
   - So `lyx loom resume` or `lyx loom start` run inside the pair restarts the ended run with no refusal.

6. LOW — Card 7, workspace regeneration over a pair whose records worktree is missing.
   - Card 7 reads every listed pair's origin with `ReadOriginFor`. That function returns an error, not "not found", when the records worktree has a record but lacks its lock directory (`internal/fabricengine/origin.go`, the `ReadOriginFor` doc).
   - The card says only that an unrecorded parent counts as the prime's. It does not keep `hubWorkspaceFolders`' existing skip-with-debug for an unresolvable pair, so one such pair would fail the whole regeneration, beam files included.

7. LOW — Card 19, `lyx orch start` in a beam.
   - `start` still calls `reconcileHub(c.location)` (`internal/orchcli/start.go:252`), which builds `pairteardown.New` and `HubSeamOptions` over the acting location.
   - That function's doc says it runs "from the prime". In a beam the reconcile now runs from the beam's location, and card 19 does not say it resolves the prime first.

Outside my lens (way-forward): a member started while the beam's landing is claimed deadlocks. Worktree-Create's re-check stops the landing on the open member and leaves the beam run-held, and card 28 then stops the member on the run-held beam.

Tool calls:
- Grep of `internal/orchcli` for the prime-bound hook, reconcile and claim, plus `reedengine` `requireSessionLocked`: needed to check whether the orch-in-beam paths and the window site can reach a session.
- Read of `internal/orchcli/start.go:245-275` and `ownerpause.go:30-62`: to confirm the reconcile and prime claim act on the acting location.
- Read of `internal/battencli/owner.go:165-215`: to confirm `claimBoardEntry` refuses only on `Lost`, so a skipped project claim passes.
