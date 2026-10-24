# Card 36 — beam-land

**Create:**
- `plan:internal/beamcli#land` -> `func land(deps landDeps, acting, slug string) (map[string]any, error)`
- `plan:internal/beamcli#landDeps` -> `type landDeps struct`
- `plan:internal/beamcli#hubLandDeps` -> `func hubLandDeps(prime *lyxcwd.Location) (landDeps, error)`
- `plan:internal/beamcli#TestLand` -> `func TestLand(t *testing.T)`
- `plan:internal/battencli#BeamLandRefusal` -> `func BeamLandRefusal(prime *lyxcwd.Location, slug string) error`
- `plan:internal/battencli#beamLandRefusal` -> `func beamLandRefusal(slug string, members []beamMember, childPairs []string, orchLive, watcherLive bool) error`
- `plan:internal/battencli#beamMember` -> `type beamMember struct`
- `plan:internal/battencli#TestBeamLandRefusal` -> `func TestBeamLandRefusal(t *testing.T)`
- `plan:internal/battencli#TestBeamLanding_ReachesWorktreeCreateSkip` -> `func TestBeamLanding_ReachesWorktreeCreateSkip(t *testing.T)`

**Edit:**
- `plan:internal/beamcli#Command`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`

**Extends:**
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/battencli#ResumeInWindow`
- `internal/hubgeom#ParentOf`
- `internal/fabricengine#ReadOriginFor`
- `internal/orchcli#OrchLive`
- `internal/orchcli#HubOrchs`
- `internal/boardengine#ReadTasks`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D11: `lyx beam land <slug>` ends a beam with a short landing run over batten and the `beam` recipe.
`BeamLandRefusal` reads the facts and `beamLandRefusal` judges them, refusing, each with its way forward:
- any open entry whose `hubgeom.ParentOf` answer is the beam, through any depth of grouping projects, naming each entry and, for each, landing it, `lyx batten cancel <slug> --remove`, or, for an entry with no pair, moving its `project` off the beam;
- any local pair whose origin record names the beam as its parent worktree, naming each pair and `lyx batten run <slug>` to land it or `lyx batten cancel <slug> --remove`;
- the beam's orch strand live by `OrchLive`, or its watcher live (`orchengine.WatcherLive` over its paths), naming `lyx orch stop` in the beam's worktree and a re-run once `lyx orch status` there reports the watcher gone.
`land` runs from the prime or a beam (`ResolveBattenPrime`) and refuses when the acting worktree is the beam itself, by D8's rule for a verb that destroys a pair, naming the beam's parent worktree as where to run it; it then applies `BeamLandRefusal` and starts the landing with `battencli.ResumeInWindow`, the in-process `lyx batten run <slug> --window`, whose window opens in the beam's parent worktree's session (card 24).
Batten's start guard applies `BeamLandRefusal` to a project entry, and Worktree-Create applies it again after its claim, so a resume or a direct `lyx batten run` cannot skip it and a member added before the claim is caught.
The start guard's left-pair refusal and its dirt refusal give way for a project entry whose `beam` is true and whose local pair carries the beam mark, so the landing reaches Worktree-Create, which finds the pair complete and skips; a landing forks no pair, and Finalize's Stuck on a dirty parent pair guards the worktree it merges into.
A project entry whose `beam` is true and whose local pair has no readable origin record (its records worktree missing) keeps the left-pair refusal, which then names `lyx fabric reconcile` to restore the records worktree and never `lyx batten cancel <slug> --remove`.
Every other entry keeps both refusals, bar card 28's dirt exemption for a beam member.
Only a beam's landing makes a project claimable (D1), and a landing forks no pair, so the start guard refuses a run of any other project entry before the claim: a grouping project (`beam` false), naming `lyx beam create <slug>` to make it a beam, and a project whose `beam` is true with no local pair, as after a create stopped at its first step or a hand-set flag, naming `lyx beam create <slug>` to finish the create, or `lyx adopt <slug>` when the beam's branch is on origin.
Worktree-Create never calls `Add` for a project entry: with its pair absent it stops with the same `PreconditionError`, so no path forks an empty pair for a beam's landing.
A member started after the landing's claim stops at card 28's run-held precondition, which names `lyx batten cancel <member>`; the landing's after-claim refusal names the member and the same cancel, so ending the member lets `lyx batten run <slug>` resume the landing.
Batten then runs its rows over the beam as over a task: Seed-Child seeds `beam` (card 33), Land prunes the board and deploys when the parent is a deploy branch, and Worktree-Teardown removes the beam pair, whose board settle marks the project done; the orch refusals of card 19 and this card leave no orch strand or watcher of the beam behind.
Bound: a pair on another machine whose origin names the beam is not seen when its entry no longer names the beam: a pair made by hand there with `--parent <beam>` and no board entry, and a pair whose entry's `project` was changed from a machine holding no pair of it.
`TestBeamLandRefusal` reaches each refusal and the pass over told facts; `TestLand` over fake deps refuses from inside the beam, passes the refusal through, and starts the window run on a clean beam;
`TestStartGuard_Refuse` gains rows: a direct run of a beam with an open member, a child pair or a live orch is refused; a marked beam pair passes the left-pair and dirt checks; an unmarked or unreadable pair is refused, the unreadable one naming `lyx fabric reconcile`; a grouping project is refused naming `lyx beam create`, and a beam project with no local pair naming `lyx beam create` and `lyx adopt`;
`TestWire_CreateWorktreeClaimsTheBoardEntry` gains rows over its fakes: a beam project whose member was added after the start guard is refused after the claim naming the member and `lyx batten cancel`, and a beam project with no pair stops with the precondition and calls no `Add`;
`TestBeamLanding_ReachesWorktreeCreateSkip` over a hubforge hub: a beam made by create's steps and one made by promote's steps each pass the start guard with the prime holding tracked dirt and reach Worktree-Create's skip, and one whose records worktree is removed is refused naming `lyx fabric reconcile`; a sub-beam `c` of beam `b` reaches the skip while `b`'s entry is open; and `BeamLandRefusal` of `b`, with an open task whose `project` is a grouping project inside `b`, refuses naming that task.

**ImpactSummary:** Adds the land verb and one batten refusal applied at land, at the start guard and after the claim; tasks keep their start-guard rules.
