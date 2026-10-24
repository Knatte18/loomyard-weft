# Card 15 — hubgeom-parent-of

**Create:**
- `internal/hubgeom#ParentOf`
- `internal/hubgeom_test#TestParentOf`

**Edit:**
- `internal/boardcli#Command`
- `internal/battencli#hubOwnerSeams`
- `internal/battencli#hubCancelDeps`
- `internal/adoptcli#hubDepsAs`
- `internal/loomcli#parentOwner`
- `internal/pairteardown#HubSeamOptions`
- `internal/tendercli#HubDeps`
- `internal/pairteardown#TestHubSeamOptions`
- `internal/battencli#TestRefuseStartOwnedElsewhere`
- `PATTERN.md`
- `pattern/PATTERN-owner-machine.md`

**Extends:**
- `internal/hubgeom/doc.go`
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/boardengine#ParentBeam`
- `internal/boardengine#ReadTasks`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#AddOptions`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D4: one `ParentOf` in `internal/hubgeom` is the only parent resolver every owner reader is handed.
It reads slug's origin record over the prime's location (`ReadOriginFor`), and only when no record is found reads `ParentBeam` over the local board (`ReadTasks` of the hub's board dir).
A found record with an empty parent reads as the prime and never falls through to the board; a found record's parent is its `ParentWorktree`, read as the prime when `ParentBranch` is the prime's checked-out branch or `ParentWorktree` names the prime's worktree, and never derived by matching `ParentBranch` against the pair worktrees present here.
So a member whose beam pair is absent on this machine still reads the beam as its parent and the beam's owner as its own.
Every reader that today passes `fabricengine.ParentSlug` takes it instead: `boardcli`'s `Config.ParentOf`, batten's owner seams and cancel deps, adopt's deps, loom's Finalize parent-owner read (`parentOwner`), pair teardown's `HubSeamOptions` (which also feeds fabric remove's owner read and loom's owner check) and tender's pair owners.
`TestParentOf` over a hubforge hub: a pair forked from the prime reads `""`; a pair forked from pair `b` reads `b`; a pair whose record names `b` while `b`'s pair is removed still reads `b`; a slug with no pair reads `ParentBeam` from a board the test writes (a member of beam project `b` reads `b`); a record whose `ParentWorktree` is empty reads `""` without reading the board; and a record naming the prime's branch under another worktree name reads `""`;
a pair forked from the prime whose board entry the test writes as a member of beam project `b` (a member spawned under a grouping project that later became a beam) reads `""`, the record winning over the board.
`TestRefuseStartOwnedElsewhere` gains a row: an unspawned member of beam project `b`, with no owner recorded, whose beam is owned by another machine, is refused naming that machine, through an `ownerSeams.ParentOf` that answers `b`.
`TestHubSeamOptions` gains a row whose `ParentOf` answers the beam for an unspawned member of a beam project.
PATTERN gains the declared entry, in the Fabric and git section after `PATTERN-owner-machine`:

```
- `PATTERN-pair-parent` — Picking a new pair's parent for a board entry or reading an entry's owner through its parent: only `ParentBeam` and hubgeom's one `ParentOf`, never the acting worktree.
```

`PATTERN-owner-machine`'s background says an entry with no owner recorded reads its parent's owner through that one resolver, and that adopting a beam moves the beam only.
The hubgeom and pairteardown package docs name `ParentOf` where they name `fabricengine.ParentSlug` today.

**ImpactSummary:** Seven owner readers swap their parent resolver for hubgeom's; fabricengine.ParentSlug loses every production caller and is deleted on the next card.
