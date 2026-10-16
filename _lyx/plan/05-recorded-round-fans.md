# Card 5 — recorded-round-fans

**Create:**
- `plan:internal/shedadapters#RoundFans` -> `type RoundFans struct`
- `plan:internal/shedadapters#roundFan` -> `func roundFan(fans []string, round int) string`
- `plan:internal/shedadapters#recordRoundFans` -> `func recordRoundFans(runDir string, round int, now func() time.Time, fan, fixFan string) error`
- `plan:internal/shedadapters#readRecordedFans` -> `func readRecordedFans(runDir string, round int) (fan, fixFan string, err error)`
- `plan:internal/shedadapters#TestBurlerReviewProducer_Call_RecordsRoundFans` -> `func TestBurlerReviewProducer_Call_RecordsRoundFans(t *testing.T)`

**Edit:**
- `internal/shedadapters#roundUsage`
- `internal/shedadapters#BurlerHalfDeps`
- `internal/shedadapters#BurlerReviewProducer`
- `internal/shedadapters#NewBurlerReviewProducer`
- `internal/shedadapters#BurlerReviewProducer.roundProfile` -> `func (p *BurlerReviewProducer) roundProfile(round int, fan string) burlerengine.Profile`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#BurlerReviewProducer.acceptedExit`
- `internal/shedadapters#BurlerFixProducer`
- `internal/shedadapters#NewBurlerFixProducer`
- `internal/shedadapters#BurlerFixProducer.Call`
- `internal/shedadapters#BurlerFixProducer.fixedExit`
- `internal/shedadapters#lensesRunUnderFan`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProfileAndOpts`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProbeAndResume`
- `internal/shedadapters#TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview`
- `internal/testkit/shedfake#BurlerHalfRunner`
- `internal/testkit/shedfake#BurlerHalfRunner.ProbeReview`
- `internal/testkit/shedfake#BurlerHalfRunner.ResumeReview`
- `internal/shedadapters#TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded`
- `internal/shedadapters#TestBurlerFixProducer_Call_FinishedFixIsRecorded`
- `internal/shedadapters#TestWriteRoundFacts_LensSection`
- `internal/shedadapters/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/shedadapters#halfProducerSpec`
- `internal/shedadapters#newReviewProducer`
- `internal/shedadapters#newFixProducer`
- `internal/shedadapters#writeJudgedRound`
- `internal/shedadapters#writeAcceptedRound`
- `internal/shedadapters#readRoundUsage`

**Intent:**
Fix a round's reviewer fan and fixer fan when its reviewer is spawned, record them in the round's usage record, and have the fix half read them from there (D2, D9).

`RoundFans` holds a segment's per-round lists, `Review` and `Fix`, entry N for round N; `roundFan` returns the round's entry, the last entry for every later round, and "" for an empty list.
`BurlerHalfDeps` gains `Fans *RoundFans`, read by the review producer, and `RequiresFans bool`, read by the fix producer.
A nil `Fans` is a row that records no names, as the Webster, Merge and Darn review rows are, and keeps today's behaviour of running its profile's own `ClusterFan`.

`roundUsage` gains `FixFan` (`fix_fan`, without `omitempty`, like `fan`), `FansRecorded` (`fans_recorded,omitempty`), `FixLenses` (`fix_lenses,omitempty`) and `FixForks` (`fix_forks,omitempty`, a pointer so a single round carries no count).
`recordRoundFans` merges `fan`, `fix_fan` and `fans_recorded: true` through `mergeRoundUsage`, the one writer that sets them; no other merge sets or clears the three keys, and a record `mergeRoundUsage` replaces because it did not decode starts without the flag.
`readRecordedFans` returns the two names, or an error for a missing record, one that does not decode, or one without the flag, since an old binary's solo record also decodes `fan` as "".

`BurlerReviewProducer.Call`, for a row with `Fans`, picks the round's names from the lists.
The probe runs under the names already recorded for the round, or under the picked names when none are recorded, so a resumed live reviewer keeps the fan it was spawned under and is audited against that fan's lens count.
On the spawn path only, after the probe finds no live reviewer, it calls `recordRoundFans` with the picked names, an empty name for a solo or single round included, before the first attempt; a failed write takes the failure exit with the round's files archived and spawns nothing.
The accepted-review hand-on path writes nothing.
`roundProfile` takes the round's fan, which sets `ClusterFan` and drives the exclusions in place of the template's fan.
`acceptedExit` stops writing `fan` from the producer's profile and merges only `Lenses` and the review half, so a hub edit between spawn and acceptance cannot overwrite the recorded name.

`BurlerFixProducer.Call`, on a row told `RequiresFans` (a Discussion or Plan fix row), reads the recorded names after the accepted-review check and before the probe.
With them it sets the profile's `FixFan` to the recorded fixer fan, so a list changed between the halves cannot reach this round.
Without them it archives the round's review, fixer report and acceptance record as the changed-review exit does and refuses with `round N's usage record holds no recorded fan names (<reason>); <wayForward>`, the `goto` of the segment's review row; the archive is what lets the review row spawn a fresh reviewer that records the names.
A row not told `RequiresFans` (Webster, Merge, Darn) reads no names, runs the single fixer and never refuses, per D2.
A recorded fixer fan that `burler.yaml` no longer defines fails the engine's `validateFix` with card 3's way forward, which the fixer's error exit carries out unchanged.
`fixedExit` merges `fix_lenses` and `fix_forks` from the `FixResult` beside the fix half when the result names lenses, and neither for a single fixer.
`lensesRunUnderFan` treats a record without `fans_recorded` as ineligible, as it treats a missing one.

`internal/testkit/shedfake`'s `BurlerHalfRunner` gains `GotProbeReviewProfiles` and `GotResumeReviewProfiles`, which `ProbeReview` and `ResumeReview` append to as the fake's other recorders do, so a test can see the fan a probe and a resume ran under.

Tests:
`TestBurlerReviewProducer_Call_RecordsRoundFans` covers the pre-spawn write alone, its `DuringReview` hook reading the record so each row proves the write happened before the spawn: a fanned entry and an empty entry (solo, names recorded empty with the flag);
a usage path that is a directory makes the write fail, so `Review` is never called, `Call` errors and the round's review and acceptance record are archived;
a nil `Fans` row writes no flag;
an already-accepted round hands on and leaves the record unwritten;
and a round whose fix half was refused, with its review, fixer report and acceptance record archived and a flagless record left, spawns a fresh reviewer and ends with the names recorded.
`TestBurlerReviewProducer_Call_ProfileAndOpts` gains per-round pick rows over `Fans` with both lists set: rounds 1 and 3 of a two-entry list take entry 1 and the last entry for both `ClusterFan` and the recorded `fix_fan`, and an empty entry runs solo;
a row whose round-2 record names the same fan without `fans_recorded` drops the round-3 focus file's exclusion, which reaches `lensesRunUnderFan`'s new check; its existing record gains `FansRecorded`.
`TestBurlerReviewProducer_Call_ProbeAndResume` gains `Fans` rows: a live reviewer whose round already recorded `fan-a` is probed and resumed under `fan-a`, read from `GotProbeReviewProfiles` and `GotResumeReviewProfiles`, though the list now says `fan-b`, and the record still names `fan-a` after acceptance;
a resumed reviewer that had already died falls through to a fresh spawn, which records and runs `fan-b`.
`TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview` gains `RequiresFans` rows: a record lacking the flag with `fan: ""`, one that does not decode and a missing one each refuse with `wayForwardToReview`, archive the review, fixer report and acceptance record and spawn nothing.
`TestBurlerFixProducer_Call_FinishedFixIsRecorded` gains rows: a `RequiresFans` row whose record names a fixer fan passes it as `FixFan`; a row without `RequiresFans` over a flagless record runs the single fixer;
a twill result naming lenses and two forks records `fix_lenses` and `fix_forks: 2`, one naming lenses and zero forks records `fix_forks: 0`, and the single rows assert neither key is set.
`TestBurlerReviewProducer_Call_AcceptedReviewIsRecorded` asserts the acceptance merge leaves `fan` as the record held it.
`TestWriteRoundFacts_LensSection` writes its usage records with `FansRecorded`, which its exclusions now need.

`refusal-spec.md` gains a loom row for the fix half's unrecorded fan names, with the archive and the `goto` way forward; a failed pre-spawn write joins the section's grouped raw I/O row.
`internal/shedadapters/doc.go` documents the recorded names, their writer and readers, and the new record keys.

**ImpactSummary:** the two half producers and the usage record in `internal/shedadapters`; the constructors keep their signatures and a nil `Fans` with no `RequiresFans` keeps every current row's behaviour until card 7 wires the lists.
