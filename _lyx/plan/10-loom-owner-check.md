# Card 10 — loom-owner-check

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli/doc.go`

**Create:**
- `internal/shedrun#StartOwnedElsewhereKind`
- `internal/loomcli#ownerVerdict`
- `internal/loomcli#runOwnerVerdict`
- `internal/loomcli#loomCLI.refuseOwnedElsewhere`
- `internal/loomcli#TestStartAndResume_RefuseARunOwnedElsewhere`
- `internal/loomcli_test#TestRunOwnerVerdict`

**Uses:**
- `internal/loomcli#seedSlug`
- `internal/loomcli#loomCLI.refuseCovered`
- `internal/fabricengine#EntryOwnedElsewhere`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#OwnedElsewhere`
- `internal/machinename#Name`
- `internal/fabricengine#ParentSlug`

**Intent:**
D3 for the shed resume verbs, `lyx loom start` and `lyx loom resume`, which serve loom and darn runs alike, and through them every InnerRun spawn of `lyx loom start --no-attach` on an existing run.
`StartOwnedElsewhereKind` is the envelope `kind` of their owner refusal, beside `StartNotParkedKind` in `shedrun`'s paths file, so InnerRun reads it (card 12) and never as a failed spawn.
`ownerVerdict` holds `Owner` and `Elsewhere`.
`runOwnerVerdict` runs a judge-only run-scope origin reconcile of `slug` from the hub's prime with `pairteardown`'s hub seams, whose `Machine` it sets to `machine`, as `runReconciler` does with `JudgeOnly` set, and reads the run's owned_elsewhere entry; with origin unreachable it reads the local Bolt's effective owner instead (`boardengine.ReadOwners`, `ParentSlug`, `machine`); a reconcile or read error is returned; it writes no owner, so a run-held task with no owner recorded reads owned here and proceeds.
`loomCLI` gains `ownerOf func(l *lyxcwd.Location, slug string) (ownerVerdict, error)`, which `newLoomCLI` sets to `runOwnerVerdict` over `machinename.Name`; a nil `ownerOf`, as a struct-literal receiver in a test has, reads owned here and calls nothing.
`refuseOwnedElsewhere` runs only when the run's status file exists, since a spawn with no status file is the fresh bootstrap under a start batten already judged; an owner elsewhere writes the refusal envelope with `kind` `owned_elsewhere` and `owner` the owner's name, "loom: this run is owned by machine %q, so this machine does not resume it; way forward: run `lyx adopt <slug>` from this machine's prime", and returns false; a verdict error refuses with its cause and the re-run of `retry`.
Both verbs call it right after `refuseCovered`, before the bootstrap lock, the seed and any spawn.
Bound: it skips nothing for a run owned here, and an unreachable origin with no local owner elsewhere proceeds as today.
Bound: with origin answering, a run whose pair has no batten run record reads owned here, since the reconcile judges only recorded pairs.
`doc.go`'s list of `lyx loom start` refusal kinds gains `owned_elsewhere`.
`TestStartAndResume_RefuseARunOwnedElsewhere` drives both verbs over a receiver with a fake `ownerOf`: owned elsewhere refuses with kind `owned_elsewhere`, the owner and `lyx adopt`, before the bootstrap lock is taken; a verdict error refuses naming the re-run; owned here goes on; an absent status file never calls the seam.
`TestRunOwnerVerdict` (integration over `hubforge`, with a fixed `machine`) seeds a batten status file at the prime for the run's pair, so its owned-elsewhere rows reach the verdict, and covers: origin's Bolt recording another owner for the run reads owned elsewhere with that owner; a run-held entry with no owner recorded reads owned here and writes nothing; with both origin URLs pointing at an address nothing answers (`http://127.0.0.1:1/x.git`), a local Bolt recording another owner reads owned elsewhere and one recording this machine reads owned here.

**ImpactSummary:** loomcli's start and resume pre-flight and one shedrun constant; batten's spawn reads the kind in card 14.

**Commit:** 10: loom: start and resume refuse a run owned elsewhere
