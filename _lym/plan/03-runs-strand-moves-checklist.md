# Card 3 — runs-strand-moves-checklist

**Edit:**
- `internal/orchcli#orchCLI.ensureRunsStrand` -> `func (c *orchCLI) ensureRunsStrand() []string`
- `internal/orchcli#orchCLI.beamUp`
- `internal/orchcli#fakeStrands`
- `internal/orchcli#fakeStrands.Strands`
- `internal/orchcli#fakeStrands.ReplaceStrand`
- `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`
- `internal/orchcli/doc.go`
- `docs/overview.md`

**Create:**
- `plan:internal/orchcli#orchCLI.moveChecklist` -> `func (c *orchCLI) moveChecklist(guid, exe string, build statuspane.BuildIdentity) (bool, []string)`
- `plan:internal/orchcli#checklistWarning` -> `func checklistWarning(what string, cause error) string`

**Uses:**
- `plan:internal/loomengine#ChecklistStrandCmd`
- `plan:internal/loomengine#ChecklistBesideSpec`
- `plan:internal/loomengine#ChecklistSidecarPath`
- `plan:internal/loomengine#ChecklistRecipe`
- `internal/loomengine#LoomScratchDir`
- `internal/loomengine#LoomStatusStrandName`
- `internal/orchcli#runsStrandWarning`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#startHarness.run`

**Intent:**
The runs strand moves a `loom-status` checklist strand out of the overview window instead of replacing it, so a pawl run's checklist survives `lym beam start` (D2).

`ensureRunsStrand` returns every warning it raised, so one bring-up can report a failed move and a failed runs-strand add together; `beamUp` appends them all to `report.Warnings`.
Its early returns (no build identity, no executable, the strands unreadable) return their one warning, with today's text, and touch no strand.
After reading the strands it finds the runs strand and the `loom-status` strand as today, then:

1. The checklist:
   - with no runs strand and a `loom-status` strand (case 1), it calls `moveChecklist(loomStatus.GUID, exe, build)`;
     a `loom-status` strand already in its own window is relaunched the same way, since `reedengine.StrandStatus` does not tell the two apart;
   - with a runs strand and a `loom-status` strand that is not live (case 2), it calls `moveChecklist` the same way;
   - with a runs strand and a live `loom-status` strand (case 3), it leaves the checklist untouched.
2. The runs strand:
   - with a runs strand, it is kept when current and replaced when stale, as today, whatever the move did;
     a failed move in case 2 only adds its warning, since the runs strand already holds the overview;
   - with no runs strand and no `loom-status` strand, or after a successful move, it adds the runs strand in the overview window;
   - with no runs strand after a failed move, it reads the strands again: a `loom-status` strand still present is replaced by the runs strand, as today, and with none present the runs strand is added;
     when that read fails, it replaces the first read's `loom-status` GUID by the runs strand, as today, so a strand the failed move left holding the overview is still displaced, and a strand already gone makes that replace fail with today's warning.
3. A failed runs-strand add, replace or sidecar write keeps today's `runsStrandWarning` texts, which name `lym beam start`; after a successful move a failed add leaves the checklist in its own window, and `lym beam start` then finds case 1 again.

`moveChecklist` replaces the strand guid with `loomengine.ChecklistBesideSpec(loomengine.ChecklistStrandCmd(shell.ForGOOS(), exe), loomengine.ChecklistRecipe(c.location))`.
It then writes `statuspane.WriteSidecar(loomengine.ChecklistSidecarPath(c.location), strand.GUID, build)`, so the next `lym loom start`'s `resolveStatusStrandAction` keeps the strand.
It reports whether the replace succeeded, and the warnings it raised, none on success:
- a failed replace: `checklistWarning("could not relaunch the loom status strand in a window of its own, so the run's checklist may be missing", err)`;
- a failed sidecar write after a successful replace, which still reports the move a success: `checklistWarning("could not record the loom status strand's build, so the next `lym loom start` replaces it once", err)`.

`checklistWarning` logs at Warn like `runsStrandWarning` and returns `"<what> (<cause>); way forward: fix the cause it names, then run `lym loom start` in the beam's worktree"`.
That way forward progresses from every state a failure leaves: `lym loom start` runs `ensureStatusStrand` right after `reed.Up` (`internal/loomcli/start.go:553`), before the driver checks that refuse a live run, and with the runs strand holding the overview it keeps, replaces or adds `loom-status` beside it through card 2's spec.

Bound: it acts only on the strand named `loom-status` and the runs strand, every other strand is untouched, and it never fails `start`.
It selects no window: reed's rules for a viewed window that ends decide where a viewer lands, as D2 states.
The `loom-status` check is keyed on the runs strand's presence, never on a recipe name.

The doc comment of `ensureRunsStrand` states the three cases and the failures.
`internal/orchcli/doc.go`'s runs-strand sentence in "The orch in a beam", and `docs/overview.md`'s two runs-strand passages (the orch bullet and the `lym beam start` paragraph), say the runs strand moves a pawl run's `loom-status` strand into a window of its own named for the run's recipe, and that a failed move is a warning naming `lym loom start`.

`fakeStrands` gains four fields, whose zero values fail nothing and drop nothing, so every other test of the fake is unchanged:
- `replaceErrs map[string]error`, keyed by the spec's `NameOverride`, which `ReplaceStrand` returns after recording "replace" in calls but nothing in `replaced`;
- `dropOnReplaceErr bool`, which makes such a failed replace also drop the replaced guid from `strands`, as reed's failure after it removed the old strand does;
- `strandsErr error` and `strandsErrCall int`, which make `Strands` return `strandsErr` on its `strandsErrCall`-th call only, counted by the existing `strandsCalls`, so the bring-up's later strand reads are unaffected.

`TestStart_EnsuresTheRunsStrandBeforeTheOrch` (tier0) gets row fields:
- a seed recipe, written with `shedrun.WriteSeed(h.cli.location, shedrun.SelfRunID, …)` before the run, or the bytes `{not json` written at `shedrun.SeedFile(h.cli.location, shedrun.SelfRunID)`;
- `replaceErrs`, `dropOnReplaceErr`, `strandsErr` with `strandsErrCall`, and an executable error the row sets on `h.cli.executable`;
- a flag that blocks the checklist sidecar: it creates the directory `filepath.Dir(loomengine.LoomScratchDir(h.cli.location))` and writes a plain file at `loomengine.LoomScratchDir(h.cli.location)`;
- the wanted replaces as pairs of replaced guid and spec, the wanted add specs, the wanted runs sidecar guid and the wanted checklist sidecar guid;
- `wantWarnings`, the report's whole warning list, compared exactly with `slices.Equal`.
The loop asserts those instead of today's blanket check that every spec is the runs spec replacing `r1`.
When a row wants a checklist sidecar, it asserts the sidecar at `loomengine.ChecklistSidecarPath` names that guid and `h.build`, so `statuspane.StrandCurrent(true, guid, sidecar, h.build)`, the predicate `resolveStatusStrandAction` keeps on, holds; a row that wants none asserts the file is absent.
Each warning is asserted as its exact text, the fake's causes being fixed;
the blocked-sidecar warning's cause is the operating system's error naming a temporary path, so that one warning is asserted to open with its exact what-text and `(` and to close with its exact way-forward text.
The beside spec a row wants is `loomengine.ChecklistBesideSpec` over the harness's command and the row's recipe, written out as a literal `AddSpec` in the row so a builder that drops the title fails it.
The rows:
- "a loom status strand holding the overview moves to a window of its own titled by the seed's recipe": strand `s1` named `loom-status` and live, seed recipe `pawl`.
  It wants calls replace then add, `s1` replaced by the beside spec titled `pawl`, the runs spec added, the runs sidecar `added-runs` and the checklist sidecar `replaced-loom-status`.
- "with no seed the moved checklist has an empty title": the same strands with no seed; it wants the beside spec with an empty `Title`.
- "with an unreadable seed the moved checklist has an empty title": the same strands with `{not json` as the seed; it wants the beside spec with an empty `Title`.
- "a dead loom status strand beside a current runs strand is relaunched beside it": a current `r1` runs strand with its sidecar and a dead `s1`.
  It wants one replace of `s1` by the beside spec, no add, the runs sidecar still `r1` and the checklist sidecar `replaced-loom-status`.
- "a dead loom status strand beside a stale runs strand is relaunched and the runs strand replaced": `r1` with a sidecar of another build and a dead `s1`.
  It wants the replace of `s1` by the beside spec, then the replace of `r1` by the runs spec, the runs sidecar `replaced-runs` and the checklist sidecar `replaced-loom-status`.
- "a live loom status strand beside a current runs strand is left untouched": it wants no replace, no add and no checklist sidecar.
- The two "a stale runs strand is replaced when a loom status strand precedes/follows it" rows keep a live `s1` and now also assert the one replace is of `r1` by the runs spec and that no checklist sidecar is written.
- "a failed move before the old strand is removed falls back to replacing it with the runs strand": `s1` live, `replaceErrs` failing `loom-status` with `move refused`.
  It wants calls replace, replace, the second replacing `s1` by the runs spec, no checklist sidecar, and the one warning "could not relaunch the loom status strand in a window of its own, so the run's checklist may be missing (move refused); way forward: fix the cause it names, then run `lym loom start` in the beam's worktree".
- "a failed move after the old strand is removed adds the runs strand": the same with `dropOnReplaceErr`; it wants calls replace then add and the same one warning.
- "a failed move whose re-read fails replaces the first read's loom status strand with the runs strand": the same as the before-removal row with `strandsErr` on call 2.
  It wants calls replace, replace, the second replacing `s1` by the runs spec, and the same one warning.
- "a failed move beside a current runs strand only warns": a current `r1` with its sidecar, a dead `s1` and `replaceErrs` failing `loom-status`.
  It wants calls replace only, no replace of anything by the runs spec, the runs sidecar still `r1`, and the same one warning.
- "a failed runs strand add after a move leaves the checklist in its own window": `s1` live and `addErr`.
  It wants the move's replace, the checklist sidecar `replaced-loom-status`, and the one warning "could not add the runs strand, so the beam overview is missing (reed refused); way forward: fix the cause it names, then run `lym beam start`".
- "a checklist sidecar that cannot be written only warns": `s1` live and the sidecar blocked.
  It wants the move and the runs add, the runs sidecar `added-runs`, and the one warning opening "could not record the loom status strand's build, so the next `lym loom start` replaces it once (" and closing "; way forward: fix the cause it names, then run `lym loom start` in the beam's worktree".
- "an executable that cannot be located leaves every strand as it is": `s1` live and an executable error `no binary`.
  It wants no replace and no add, and the one warning "could not locate the running lym binary, so the runs strand was left as it is (no binary); way forward: fix the cause it names, then run `lym beam start`".
- The existing no-strand, kept, replaced-runs and add-failure rows keep their expectations, now stated as exact warning lists.

**ImpactSummary:** `ensureRunsStrand`'s one caller is `beamUp` in the same package, re-signed here; the fake's new fields default to today's behaviour, so only the one runs-strand test changes; `LoomStatusStrandName`'s comment changes in card 1.

**Commit:** 3: The runs strand moves a pawl run's checklist to its own window instead of replacing it
