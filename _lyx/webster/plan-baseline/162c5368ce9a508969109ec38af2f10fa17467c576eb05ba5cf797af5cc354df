# Card 18 — batten-beam-project

**Create:**
- `internal/battencli#FirstPassOptions`
- `internal/battencli#anchorParam`
- `internal/battencli#memberHoldRefusal`

**Edit:**
- `internal/battencli#StartFirstPass` -> `func StartFirstPass(location *lyxcwd.Location, slug string, opts FirstPassOptions) (map[string]any, error)`
- `internal/battencli#battenCLI.armSeed`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuseProject`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI`
- `internal/battencli#landingSeams`
- `internal/battencli#hubLandingSeams`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestStartFirstPass`
- `internal/battencli#TestWire_CreateWorktreeClaimsTheBoardEntry`
- `internal/battencli#TestWorktreeCreate_ForksFromTheBeam`
- `internal/battencli#TestBeamLanding_ReachesWorktreeCreateSkip`
- `internal/adoptcli#hubDepsAs`
- `internal/beamcli#hubLandDeps`
- `internal/battenshed/doc.go`

**Intent:**
D11: batten runs a beam project's pawl run from its first pass, and D9's member hold read at Worktree-Create.

- The start guard lets a beam project start or resume: `refuseProject` refuses a grouping project naming `lyx beam create <slug>` (reached by a `TestStartGuard_Refuse` row); a beam project with no local pair, a fresh start, gets a task's dirty-source and left-parts checks, the dirty-source check only when it forks from the prime as a task's does; a marked beam pair, a legacy beam's or a resumed run's, passes them as today; a resume past Worktree-Create passes.
  The land refusal leaves the guard (`startGuard.LandRefusal` and `landingSeams.Refusal` go) and the pawl run's own Worktree-Create, since `lyx beam land`, Beam-Await and Finalize apply it.
  The guard keeps refusing a task whose pair carries the beam mark (card 1's way forward).
- Worktree-Create, for a beam project, claims the entry, skips a complete pair as today, and otherwise makes the pair from `boardengine.ParentBeam`'s worktree through `Topology.Add` with `AddOptions{Parent: <that beam>, Anchor: <the run record's anchor>}`, unmarked, since Beam-Up marks it; a parent beam whose pair is absent here names `lyx beam create <parent>` or `lyx adopt <parent>`, as a member's fork does.
  The board's beam flag, written first by `lyx beam create`, holds a member started meanwhile at its Worktree-Create as today; the pawl run's own status there is no landing hold.
- A member's Worktree-Create reads its beam's hold again once its pair is made, and also when it finds the pair already complete, so a member that crashed between `Add` and the read and is resumed is judged too: when `boardengine.RunHolds` reads the beam's entry held, it removes the pair through `pairteardown` and refuses with `memberHoldRefusal`; so either this read or Beam-Await's re-check sees the other.
  The refused member keeps its claim, so it stays an open member and the beam's landing stops on it at the re-check or at Finalize; `memberHoldRefusal` therefore names the two steps that progress from there: `lyx batten cancel <member>` from the beam's parent worktree, which ends the member and releases its claim, then `lyx board upsert` moving its `project` off the beam, since a draft member still blocks the landing, after which the beam's next land request lands it; or, once `lyx batten status <beam>` shows the beam's run back at Beam-Await, `lyx batten run <member>`, which resumes the halted member, since dobby passes no halted run.
- A beam project skips the start guard's draft check, since a beam's cancel (card 20) leaves its entry `draft` and `lyx beam create <slug>` starts its run again from there; a task keeps the check.
  The read after `Add` goes through a `battenCLI.beamHold` field, `func(beam string) (status string, held bool, err error)`, which a nil value reads from the local board; the input that reaches the refusal is a member whose `beamForkPrecondition` passed before the beam's accept wrote Parent-Align, which `TestWorktreeCreate_ForksFromTheBeam` stages by setting `beamHold` to report the beam held at `running · Parent-Align`.
- `FirstPassOptions` holds `Anchor`; `StartFirstPass(location, slug, opts)` passes it to `armSeed`, which records it as the seed's `anchor` param on the first pass only, so it rides in batten's own run record and a resume keeps it; Worktree-Create reads it from the seed.
  Its callers pass empty options: adopt's `ResumeRun` and `lyx beam land`'s `StartLanding`, which card 22 removes.
- The `battencli` package comment, in `cli.go` beside `StartFirstPass` and `battenCLI` this card edits, and `battenshed/doc.go` describe the beam project's run.

Tests: `TestStartGuard_Refuse` gains a fresh beam project with no pair passing over a clean prime, refused over a dirty one when it forks from the prime, a marked beam pair passing over a dirty prime, a grouping project refused naming `lyx beam create`, and a beam project whose status is `draft` passing where a draft task is refused.
`TestWire_CreateWorktreeClaimsTheBoardEntry` gains a beam project whose pair is made from its parent beam, unmarked, at the seed's anchor, and a beam project whose parent beam's pair is absent here, refused naming `lyx beam create <parent>`, or `lyx adopt <parent>` when its branch is on origin.
`TestWorktreeCreate_ForksFromTheBeam` gains the member whose beam holds after its pair is made, and the member resumed over a complete pair while its beam holds: in both the pair is gone and the refusal names `lyx batten cancel <member>` and `lyx batten run <member>` with `lyx batten status <beam>`.
`TestStartFirstPass` gains a first pass with an anchor that records the seed's `anchor` param.
`TestBeamLanding_ReachesWorktreeCreateSkip` keeps its guard and skip cases without the land refusal at Worktree-Create.

What it lets through, bounded: a beam project now starts a run from no pair, bounded by the claim's owner check, the guard's dirty-source and left-parts checks and the beam flag `lyx beam create` writes first; the land refusal moves from the start to the landing, where it runs three times.
A draft beam project now starts too, past the start guard's draft check, bounded by the beam flag, which only the owner setter's `Beam` write sets and only `lyx beam create` makes; the dobby still starts tasks only (`dobbyengine`'s `released`), so only `lyx beam create`, a hand pass or adopt's `ResumeRun` of a pawl run whose record is not done starts or resumes a beam project's run, and a grouping or unflagged project still refuses at `refuseProject`.

**ImpactSummary:** batten's guard and Worktree-Create gain the beam project path and the member hold read; `StartFirstPass`'s re-sign has two production callers and one test, all listed.

**Commit:** 18: Batten runs a beam project from its first pass; a member forked under a held beam is removed
