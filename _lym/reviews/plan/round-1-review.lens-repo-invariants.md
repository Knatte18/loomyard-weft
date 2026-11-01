```yaml
lens: repo-invariants
outcome: reported
```

Lens repo-invariants: I checked each card for reflective seam tests and tier tags, for doc text it falsifies that no card lists, and for whether it passes its own gate after the cards before it. Four findings, all LOW consistency: each is a doc line one card falsifies with no card listing that file.

**Per card**

- **Card 1** (pauserecord): new `Coverage.Held` field has no reflective seam test over `Coverage` (held). `pauserecord/doc.go` is listed (held). No dependency on later cards (held).
- **Card 2** (dobby hold): new `RunView.Held` field. `dobbyengine`'s seam test is only an import allowlist (`seam_enforcement_test.go:39`), and `beamcli` has no `//lym:seam` marker (held). `dobbyengine/doc.go` is listed (held). Needs only card 1 (held).
- **Card 3** (board fields): the `boardengine` tests carry no `//lym:seam` marker. `replayFields` and `scopeChanged` are listed, and `TestFrontTask` is tier0 (held). Both doc files are listed (held). No dependency on later cards (held).
- **Card 4** (batten cancel): `battencli/wire_seams_test.go` is unaffected, since no wire struct gains a field (held). Doc: nothing falsified (held). Needs card 1 for its `HoldCaller` row (held).
- **Card 5** (check per subject):
  - Seams and tiers: `TestPrimeMark` is tier0 (held).
  - Doc: broken, see F1 and F2.
  - Order: the `ReadCheckView` delete and its one caller in `beamcli` sit in the same card (held).
- **Card 6** (fronttask): `fronttask_test.go` is tier0, `hub_integration_test.go` tier1, and `testmain_test.go` untagged support only (held). Doc: new package; the `lymdirs` and `fabricengine` docs are listed (held). Needs cards 3, 4 and 5 (held).
- **Card 7** (dobby front starts): new `Deps.SettleFronts`; no reflective test fills `dobbyengine.Deps` (held). Doc: `dobbyengine/doc.go`, `beamcli/doc.go` and `PATTERN.md` are listed (held). Needs card 6 (held).
- **Card 8** (darn condition stop): new `DarnDeps.ConditionStopMarkerPath`. The `envkit` NilSeams and FullEnv tests and both `TestWire_EverySeamFilled` are covered through `FullEnv` and its Uses list (held). Doc: the `loomshed` and `darnengine` docs are listed; `shedrun`, `loomcli` and `shedrecipe` docs name no stop marker (held). Order held.
- **Card 9** (`adopt_fix_tries`): no doc or test outside `dobbyengine` names the beam.yaml keys, per a search for `task_headroom` (held). Order held.
- **Card 10** (task-branch slugs): `TestTaskBranchSlugs` is tier1 (held). Doc held. Order held.
- **Card 11** (adopt holds):
  - Order: held. `world.deps` and `TestAdoptIntegration_SecondCloneTakesOver` are listed, and line 106's `PauseLifted`/`resume the run:` assertion sits inside that test.
  - Doc: broken, see F3 and F4.
- **Card 12** (green end): seams held. Doc: `adoptcli/doc.go` and `PATTERN.md` are listed (held). Needs cards 6 and 11; its `PATTERN.md` entry follows card 7's (held).
- **Card 13** (fix loop): tests sit in existing tier0 test files (held). Doc held. Needs cards 6, 9, 10 and 11 (held).
- **Card 14** (run report): tests held. Doc held. Needs card 13's `fixRunHalted` (held).

**Findings**

- **F1 — LOW — consistency — card 5 (`internal/gateslot/doc.go`).** The gate-key table there still names the check as plain `lym adopt check`. Card 5 makes the verb `lym adopt check <subject>`, run in the subject's worktree, and updates only `pattern/PATTERN-test-speed.md`'s table. Evidence: `internal/gateslot/doc.go:108`, the row `adopt  lym adopt check  adopt  all`. Fix: add the file to card 5's Edit list.
- **F2 — LOW — consistency — card 5 (`internal/adoptcli/check.go` file header).** The header says the check runs the gate "in-process in the prime". Card 5 runs it in the subject's worktree but lists only `check.go`'s members, not its file self glyph. Evidence: `internal/adoptcli/check.go:1`. Fix: add the file self glyph to card 5.
- **F3 — LOW — consistency — card 11 (`internal/adoptcli/adopt.go` and `deps.go` file headers).**
  - `adopt.go`'s header says adopt "lifts the owner pauses, resumes the runs". Card 11 makes adopt hold the runs and resume none.
  - `deps.go`'s header lists "the run resume, the child resume" among `adoptDeps`' fills. Card 11 removes the `ResumeRun`, `ChildStatus` and `ResumeChild` seams.
  - Card 11 lists only member glyphs of both files.
  - Evidence: `internal/adoptcli/adopt.go:1`, `internal/adoptcli/deps.go:1`.
  - Fix: add both file self glyphs to card 11.
- **F4 — LOW — consistency — card 11 (`internal/battenshed/doc.go`).** The doc says "`lym adopt` and `lym beam land` start a run's first pass detached". After card 11, adopt starts no first pass; card 12's green end does. No card lists the file. Evidence: `internal/battenshed/doc.go:52`. Fix: add the file to card 11, or to card 12, which takes the starts over.

One thing I could not verify: whether anything outside `adoptcli` reads the envelope's `suite` or `suite_warning` fields that card 5 renames. I had no tool calls left.

Tool calls:
1. Searched for `//lym:seam` markers and seam tests in the touched packages, and the `adoptcli` test functions: the manifest held no seam-test reads.
2. Read `loomshed/seam_enforcement_test.go` and searched docs, `PATTERN.md` and `pattern/` for adopt check and resume text: the manifest held no doc search beyond the cards' own files.
3. Narrowed that search to adopt resume, check and lift lines in the candidate docs, to find the falsified text.
4. Read the context of `gateslot/doc.go:108` and `battenshed/doc.go:52`, the `adoptcli` integration-test assertions, other readers of the beam.yaml keys, and the stop-marker lines of the `shedrun`, `loomcli` and `shedrecipe` docs: none was in the manifest.
5. Read `TestCommand_RunsFromABeam` and the assertion at `adopt_integration_test.go:95-112`, to confirm which test holds the resume assertion card 11 changes.
