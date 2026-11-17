# Card 22 — upgrade-first-read

**Create:**
- `plan:internal/dobbyengine#UpgradeDeps` -> `type UpgradeDeps struct`
- `plan:internal/dobbyengine#UpgradeRun` -> `type UpgradeRun struct`
- `plan:internal/dobbyengine#UpgradeReport` -> `type UpgradeReport struct`
- `plan:internal/dobbyengine#UpgradeRead` -> `func UpgradeRead(deps UpgradeDeps) (UpgradeReport, error)`
- `plan:internal/dobbyengine#TestUpgradeRead` -> `func TestUpgradeRead(t *testing.T)`

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#daemon.catchUp`
- `internal/dobbyengine#TestRun_CatchUp`
- `internal/dobbyengine#dobbyengineAllowedImports`
- `internal/beamcli#hubDobbyDeps`
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/fixtries#Read`
- `internal/boardengine#WriteHold`
- `internal/boardengine#Owners.HoldOf`
- `internal/tendercli#RecordPauseIntent`
- `internal/pauserecord#AdoptCaller`
- `internal/pauserecord#OwnerCaller`
- `internal/fabricengine#AdoptScratchDir`

**Intent:** Turn the pause record's standing `adopt`, `adopt-hold` and `owner` entries into the new model at the first read after the upgrade, run by the dobby start before its catch-up (D5, the split review's F2 route).

`UpgradeRead(deps)` takes the hub's upgrade lock (`Lock`, a path beside the pause record the wiring names through `internal/lock`), so the dobbies of one machine run it one at a time; the second finds nothing left to do.
It then reads the live entries of the pause record whose caller is `AdoptCaller`, `HoldCaller` or `OwnerCaller`; with none it does nothing.
With some, it first fetches Bolt through `PullBolt`, so the owners, the registry and the holds it judges are origin's; an unreachable origin returns `ErrOriginUnreachable`, and nothing is written.
For each `adopt` or `adopt-hold` slug entry, `Classify` returns an `UpgradeRun`: whether this machine owns the run (through the registry), whether it has a batten record, whether its state is terminal, whether it is a released `queued` task, and its adopt subject.
1. Each run owned here, with a record, not terminal, becomes a run pause intent through `PauseIntent` (pushed synchronously; an owed push is an error), so no run resumes unasked.
   A push that hands the run over (`fabricengine.ErrHandedOver`, origin holding another machine's takeover entry) reads the run as owned elsewhere: it becomes no intent and its entry is dropped at step 3.
2. The subjects of those runs and of every released `queued` task an entry names form the hold.
   With at least one, and with no hold of another machine standing on the fetched Bolt, it writes one Bolt hold through `WriteHold`, naming this machine and the subjects (each `PriorOwner` empty, since the upgrade does not know it):
   `adopt-fix` naming the fix task when `OpenFixTask` finds an open adopt fix task of one of the subjects (the front task with `on_land` `adopt-check` of that subject), each subject's `Tries` read with `fixtries.Read(filepath.Join(AdoptDir, subject))`; otherwise `adopt`.
   It posts one notice naming the hold, its machine and, as way forward, a re-run of `lym adopt <subjects>` there, then, once that adopt's check is green, `lym tender resume <slug>` there for each run that became a pause intent.
   A hold of another machine standing writes none, and the notice names that hold's machine.
   An upgrade whose entries all belong to landed runs or runs owned elsewhere writes no hold and posts no notice.
3. Only then does it drop, through `Drop` by each entry's exact ref, every `adopt`, `adopt-hold` and `owner` entry; the catch-up's step 5 then writes this machine's entries from the pause intents step 1 pushed.
A failure at step 1 or 2 returns before step 3, so a cut first read runs again from the entries left; a first read cut after its hold push whose hold a green check cleared meanwhile writes the hold again, and a re-run of `lym adopt` clears it.
No message names `--take-hold`.

`Deps` gains `Upgrade UpgradeDeps`; a zero `Upgrade` (nil `Entries` seam) skips the first read, as a hand-built `Deps` has.
`catchUp` runs `UpgradeRead` before `CatchUp` until one run of it succeeds, keeping the catch-up owed while it fails; its error takes the `origin` or `failed` wait as card 21 classifies it.
`hubDobbyDeps` fills it for every dobby, since a machine may own no prime and run only beam dobbies: the record path and the lock beside it, `PullBolt` as the catch-up's, `Classify` over `pairteardown.ReadRun` and the board, `PauseIntent` as `tendercli.RecordPauseIntent`, the holds from the fetched Bolt, `WriteHold`, `OpenFixTask` through `fronttask.FrontOf`, `AdoptDir` as `fabricengine.AdoptScratchDir(prime.HubPath)`, the entry drop through `tenderengine`, and the notice.
`dobbyengineAllowedImports` gains `internal/fixtries`.

`TestUpgradeRead`, tier0 over fakes, covers the D5 upgrade criteria:
a standing `adopt` entry becomes a pause intent and writes one `adopt` hold with one notice naming it and `lym tender resume` of the run;
an open fix task makes the hold `adopt-fix` naming it with the local try count;
an `owner` entry and a landed run's entry are dropped;
entries all of landed runs or runs owned elsewhere write no hold;
a released queued task's entry joins the hold's subjects;
another machine's standing hold writes none and the notice names it;
an unreachable origin writes nothing;
a failed intent push drops nothing, and a failed `WriteHold` drops nothing;
a push that hands the run over drops its entry with no intent;
a re-run after its hold was cleared writes the hold again;
every `adopt`, `adopt-hold` and `owner` entry is gone after a run that succeeds.
`TestRun_CatchUp` gains a row: a failing first read keeps the catch-up owed and spawns no pass, and a zero `Upgrade` runs none.
`doc.go` describes the first read.

**ImpactSummary:** One new engine function run at every dobby's start through one new Deps field; the allowlist gains fixtries and the wiring fills the seams.
