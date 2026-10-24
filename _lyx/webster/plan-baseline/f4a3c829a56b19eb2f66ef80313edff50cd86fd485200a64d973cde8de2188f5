# Card 20 — orch-notify-parent

**Create:**
- `plan:internal/orchcli#NotifyParent` -> `func NotifyParent(pair *lyxcwd.Location, program, line string) (posted bool, err error)`
- `plan:internal/orchcli#NotifyFrom` -> `func NotifyFrom(from *lyxcwd.Location, program, line string) (posted bool, err error)`
- `plan:internal/orchcli#ParentOrchRecorded` -> `func ParentOrchRecorded(pair *lyxcwd.Location) (bool, error)`
- `plan:internal/orchcli#OrchStrandOf` -> `func OrchStrandOf(worktree *lyxcwd.Location) (name string, recorded bool, err error)`
- `plan:internal/orchcli#parentOrchStrand` -> `func parentOrchStrand(pair *lyxcwd.Location, live func(worktree *lyxcwd.Location) (bool, error)) (name string, recorded bool, err error)`
- `plan:internal/orchcli#TestParentOrchStrand` -> `func TestParentOrchStrand(t *testing.T)`
- `plan:internal/loomcli#TestParentRecordsCommitter_CommitsTheRecordedParent` -> `func TestParentRecordsCommitter_CommitsTheRecordedParent(t *testing.T)`

**Edit:**
- `internal/orchcli#PrimeOrchStrand`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#landingDeps`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/webstercli#websterCLI.wireHub`

**Delete:**
- `internal/loomcli#primeOrchRecorded`

**Extends:**
- `internal/orchcli/doc.go`

**Uses:**
- `plan:internal/hubgeom#NearestLiveOrch`
- `internal/hubgeom#ResolveParent`
- `plan:internal/orchcli#OrchLive`
- `internal/orchcli#notifyPrime`
- `internal/loomcli#parentRecordsCommitter`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D13: a notice of a run goes to the nearest live orch up its pair's beam chain, the prime's when none is live.
`OrchStrandOf` generalises `PrimeOrchStrand` to any worktree: the strand reed's state holds for the recorded GUID, or the fixed name `<shortname>:orch` for the prime and `<shortname>:<slug>:orch` for a beam; `PrimeOrchStrand` becomes `OrchStrandOf` of the resolved prime.
`parentOrchStrand` resolves the pair's parent worktree from its origin record (`hubgeom.ResolveParent`), the prime when it names none or is gone, and the destination through `hubgeom.NearestLiveOrch(parent, live)`, answering that worktree's `OrchStrandOf`.
`NotifyParent` posts through the existing `notifyPrime` over `parentOrchStrand(pair, OrchLive)`, resolving at each call; `NotifyFrom` posts the same way with the walk starting at `from` itself, for batten's notices of a run whose pair is absent (card 25); and `ParentOrchRecorded` answers whether that destination records a strand: the watched-or-silent decision loom's driver reads in place of `primeOrchRecorded`, which is deleted.
Every caller of `orchcli.NotifyPrime` that acts on a run's pair takes `NotifyParent` over that pair's location: shuttle's hold notice set in loom's `wire`, webster's and burler's `wireHub`, loom's `NotifyParent` and Finalize's `Notify` (`landingDeps`), and the driver's hand-back `Notify` in `goDriverDepsFor`.
Callers acting on no run's pair, hibernate's notify and fabric's push notifier, keep `NotifyPrime` and so reach the prime's orch; batten's notices take the walk from their run's pair on card 25.
`TestParentOrchStrand` over a hubforge hub with a beam pair `b`, its member `m`, a seeded orch state in `b` and a fake `live`: live answers `<shortname>:b:orch`, stopped or never started answers the prime's strand, and a pair forked from the prime answers the prime's strand;
`ParentOrchRecorded` of `m` answers true with `b`'s orch live and recorded, and false when the destination is a prime whose orch state records no strand;
and `PrimeOrchStrand` resolved from `m`'s location, the lookup hibernate's notify and the push notifier keep, answers the prime's strand while `b`'s orch is live.
`TestParentRecordsCommitter_CommitsTheRecordedParent` shows the walk does not reach Finalize's parent-records commit: with member `m` forked from beam `b`, whose orch is stopped, `parentRecordsCommitter` commits a run-records file left in `b`'s records and leaves the prime's records untouched.

**ImpactSummary:** Six notice sites move to the walked destination and one loom helper is deleted; hibernate and the push notifier keep the prime.
