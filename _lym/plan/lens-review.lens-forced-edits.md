```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. I checked what each card's changes force outside the members it names.

**BLOCKING**

1. **Card 13, `ReasonHeld` wording.** Card 13 rewords the held refusal, but a test outside its targets still asserts the old text.
   - `internal/tenderengine/resume_test.go:419` (in `internal/tenderengine#TestResume`) asserts `strings.Contains(err.Error(), "an adopt holds the hub")` on a `runfsm.ReasonHeld` verdict.
   - Card 13's new wording is "an adopt on machine <m> holds the hub", which does not contain that substring.
   - Card 13 does not list `TestResume`. Card 15 lists it later, but the test fails from card 13's commit until card 15, which breaks the stencil's rule 4 for reworded messages.
   - Fix: add `internal/tenderengine#TestResume` under card 13's `**Edit:**`.

**MEDIUM**

2. **Card 7, re-sign arrows missing.** Card 7 says "the identity travels beside it" wherever a function receives `here` as a name. It re-signs only `pairOwners` and `readTaskPlacement`. Every other function below takes `here`/`machine` as a string today and has to receive a `machinename.Identity` to call the new `OwnedElsewhere`. None carries an arrow:
   - `internal/beamcli#judgeLandOwners` (`land.go:428`, `here string`)
   - `internal/beamcli#judgeCreateEntry` (`create.go:111`)
   - `internal/beamcli#judgeParentOwner` (`create.go:129`)
   - `internal/loomcli#pawlRecheckLines` (`pawlwiring.go:375`)
   - `internal/loomcli#localOwnerVerdict` (`ownercheck.go:80`, `machine func() (string, error)`)
   - `internal/orchcli#archiveBeforeOwnerRefusal` (`start.go:558`)
   - `internal/orchcli#beamUpRefusal` (`refusal.go:52`)
   - `internal/adoptcli#pauseTargets` (`pause.go:21`), `internal/adoptcli#pauseSubject` (`pause.go:66`)
   - `internal/adoptcli#resumeSlug` (`adopt.go:490`), `internal/adoptcli#foreignParent` (`adopt.go:629`)

   Their test callers are not listed either:
   - `internal/loomcli/pawlwiring_test.go:196` calls `pawlRecheckLines(..., "here", ...)`.
   - `internal/orchcli/cli_test.go:197` calls `beamUpRefusal(tt.slug, "desktop", ...)`.
   - `internal/adoptcli/runreport_test.go:245` builds `reportInput{Machine: here, ...}`, which `runReports` compares against.

   Fix: add arrows for each re-signed member, and list the enclosing test members. The alternative is to state in the Intent that each of these takes an `elsewhere bool` or keeps its signature, and how.

**LOW**

3. **Card 23 falsifies `internal/beamcli/doc.go` but lists it under `Uses`.** The doc describes the owner pause that card 23 retires:
   - `beamcli/doc.go:307` and `:311` ("owner pauses run once"; "owner pauses and mutations")
   - `:413` ("A cancel or an owner pause of the beam's landing run")
   - `:460` ("the owner pause and the parent lock release through `tendercli` and `battencli`")

   Fix: move it to `**Edit:**`.

4. **Card 23 falsifies `internal/loomcli/doc.go:760` but lists it under `Uses`.** The doc reads "It runs over the prime's batten record of the run's slug and the owner pause." Fix: move it to `**Edit:**`.

5. **Card 23 leaves the owner-pause readers standing.** No card covers them, and once nothing writes `OwnerCaller` they read a pause that never exists:
   - `internal/pairteardown#OwnerPauseHeld` and its callers (`battencli/cancelcmd.go:224`, `battencli/owner.go:279`, `pairteardown/teardown.go:228`, `followup.go:131`)
   - `Coverage.OwnerHeld` (`pauserecord/record.go:94-96`, `:243`)
   - battenshed's lock release under an owner pause (`innerrun_parentlock.go:36`, comment `:26` and `:34`)
   - `pairteardown/doc.go:172-179`

   The build stays green, but the comments and docs that describe an owner pause become wrong. At least the battenshed `lockStep` branch and its comment ride on card 23's own retirement.

6. **Card 12 makes the `internal/dobbyengine#RunView` `Held` doc wrong.** The doc at `passes.go:53-56` says `Held` is an `AdoptCaller`/`HoldCaller` entry that "Only a hand lifts". After card 12, `bolt-hold` entries, which lift themselves, also set `Coverage.Held` (filled through `battencli/dobbyreads.go:187`). No card rewrites that comment. Card 20 already edits `RunView` and could carry it.

7. **Card 4 falsifies the machinecli package doc.** `internal/machinecli/doc.go:3` says "The package is the only writer of the machine file". From card 4 on, `boardengine.EnsureIdentity` writes it too, and the doc is only corrected at card 10. That is a stale window, not a break, so it is optional to list it earlier.

All 77 glyphs from the earlier resolve batch and the 84 card glyphs I resolved (tests, fakes, wiring members) were found. The `_test` units in cards 3, 13 and 17 match their files' package clauses (`boardengine_test`, `runfsm_test`, and the external `pairteardown_test` files).

Tool calls:
1. Grep of the re-signed functions' signatures, their test callers, the held wording, the origin-owed test references and docs naming the owner pause. My context had no call sites for these.
2. `lym quarry resolve` of every card glyph not resolved earlier, plus the package clauses of `store_test.go`, `act_test.go` and pairteardown's external test files. They had not been checked.
3. awk/grep for the test function enclosing each remaining assertion, the origin-owed and owner-pause doc mentions, and import allowlists. My context had only line numbers.
