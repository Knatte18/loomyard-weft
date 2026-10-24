```yaml
lens: bound-breaker
outcome: reported
```

Claims attacked:

1. D3 hand-edit bound ("stops at Worktree-Create with D7's precondition") — broken (F3).
2. D9 "member started between steps 1 and 2 stops … and resumes" — broken (F3).
3. D9 "each step skipped when its result is already there, so a re-run finishes a stopped create" — broken (F3).
4. D5 "Merge-back, the parent lock and teardown need no change" — broken (F4).
5. D6 "the Land row already stops on an origin record without a parent branch" — held (`internal/battenshed/land.go:190`).
6. D6 bound "only a command that names the flag adopts" — held.
7. D8 self-pair refusal as the wider guard's bound — broken (F1).
8. D8 "the name `hubgeom.ResolveParent` already gives" — held (`internal/hubgeom/parent.go` decideParent).
9. D8 weft content takes no part in a landing merge — held (`internal/fabricengine/doc.go` ~l.97).
10. D10 step order "ends a running run for good" — broken (F2).
11. D10 bound "no member's weft inherits its run records" — held, but only while F2 does not occur.
12. D11 Worktree-Create re-check after claim — held (D7 run-held stop + D2 refusal of a project in a run status close the post-claim window).
13. D11 bound "a pair made by hand on another machine" — broken one hop further (F5).
14. D11 "shed writes a run failed when a row with no on_reroute returns a reroute" — held (`internal/shedengine/doc.go:96`).
15. D11 "built so that it never reroutes" — held as a requirement: darn Finalize today reroutes on conflicted/overlap merge (`internal/landingshed/finalize.go:878-884`) and on kept fixer change when `RerouteReviewed` is true (l.753); a nil `RerouteReviewed` halts; the record states the target behaviour and an acceptance line checks it.
16. D12 "every machine regenerates the same files" — broken (F7).
17. D13 every `NotifyPrime` caller gives way to "a run's resolved parent" — broken (F8).
18. D14 "none of them makes a slug" (owner setter) — broken (F6).
19. D1 "No task may depend on a project" — broken via D10 (F9).
20. D4 "Origin found with an empty parent reads as the prime and never falls through" — held.
21. D2 prune run-lock exemption "as it lets prune's depends_on rewrite through today" — held (`internal/boardengine/runlock.go:66`).
22. D1 `LoadConfig` "refuses `T`" — unverified (no `T` refusal found by grep in `internal/boardengine/config.go`; not pursued).
23. Open risk 1 deploy-before-first-project — held as a stated bound.

Findings:

- **F1 — BLOCKING — design — D8:** The wider guard's only stated bound is the self-pair refusal, which does not bound what it now admits. `RequireDrivableWorktree` admits every warp worktree, task pairs included (`internal/fabricengine/fabric.go:162`), and no shell-guard row or agent settings deny stops an agent in a task pair from running `lyx batten run/cancel`, `lyx adopt --all` or `lyx cleanup --apply` (`internal/shellguard` has no batten rows; `claudeengine/settings.go` denies only `lyx webster` in forks) — `RefuseUnlessPrime` was the barrier. Counterexample: from task pair A, `lyx batten run B --window` opens B's batten window in A's reed session (D8); when A lands, pairteardown ends A's reed session before removal (`pattern/PATTERN-pair-teardown.md`), killing B's driver. D8 states neither what the wider guard lets through nor what bounds it.
- **F2 — MEDIUM — design — D10:** Promote's restart window re-admits a start. Step 1 clears the claim and batten records; the entry stays an open unclaimed task until step 3. In that window any orch or operator `lyx batten run` starts it: Worktree-Create finds the pair present and skips, a new loom run starts in the pair; a promote stopped between steps 1 and 3 and re-run then has step 2 reset the weft under a live run, and step 3 kind-changes a run-held entry (refused by the run lock). D10 states no hold on the entry between steps.
- **F3 — MEDIUM — design — D7/D9/D3:** D7's way-forward split "no owner → `lyx beam create`, else `lyx adopt`" dead-ends both ways. D9 step 1 records this machine as owner, so a member started mid-create on the same machine is told `lyx adopt <beam>`, not to re-run the create; after D3's hand edit there is no owner, so D7 names `lyx beam create`, which D9 refuses because it admits only a project "whose `beam` is false". The same admission rule contradicts D9's own "re-run finishes a stopped create" after step 1.
- **F4 — MEDIUM — design — D5 "parent lock … need no change":** The origin reconcile guards only the prime as a landing target: prime branches move under the prime lock and only when `PrimeParentLockFree` finds no live owner (`internal/fabricengine/doc.go:87`, `originreconcilemoves.go:327-334`, `pairteardown/originseams.go:59`). A beam pair is non-live (no process while it lives), so the reconcile fast-forwards its checked-out branch via `ffWorktree` with no check of the beam branch's parent lock; a member's Finalize merging in the beam worktree under that lock can race a `lyx cleanup`/`lyx adopt` reconcile from another worktree. Not confirmed whether `LivePair` counts a live beam orch strand.
- **F5 — MEDIUM — design — D4/D11/D13:** A `project` change on an entry whose pair already exists splits the two parent readers. After `lyx batten cancel` without `--remove` the entry is not run-held, so D2 lets its `project` change; its origin still names the old beam, so `ParentOf` (origin first) gives owner reads the old beam while `ParentBeam` gives the ready notice and D11's land refusal the new one. If that pair sits on another machine, D11's refusal misses it and the beam lands and is torn down under a pair that later lands into a removed parent. D11's stated bound covers only hand-made pairs.
- **F6 — LOW — consistency — D14 vs D10:** D14 checks "a kind change to project" and says the owner setter "makes no slug", yet D10 step 3 makes that kind change through the owner setter, built without the seam per D14.
- **F7 — LOW — consistency — D12 rationale:** "every machine regenerates the same files" is false; the files list local worktrees, which differ per machine.
- **F8 — LOW — design — D13:** "one notify of a run's resolved parent" is undefined for callers with no run — `hibernatecli/deps.go:112`, `orchcli/pushnotifier.go:46` call `NotifyPrime`; D13 leaves their destination unbounded.
- **F9 — LOW — consistency — D1 vs D10:** D10 refuses only an *open* dependent; a done task whose `depends_on` names the promoted task then depends on a project, which D1 forbids, and D1 states no exemption for an unchanged `depends_on`, so later writes to that done task are ambiguous.

Tool calls:
1. `internal/battenshed/land.go` + `boardengine/config.go` — verify D6's Land-row claim and D1's `T` refusal, not in manifest.
2. `pairteardown` `PrimeParentLockFree` — one hop past D5's teardown claim.
3. `adoptcli` kind refusals + battenshed file list — locate Worktree-Create for D11's re-check.
4. `fabricengine` `PrimeParentLockFree` use — what the prime lock check guards, for F4.
5. `battenshed/create.go` — present-pair skip for D11 (no further detail found).
6. `fabricengine/doc.go` reconcile lines — how a pair branch moves, for F4.
