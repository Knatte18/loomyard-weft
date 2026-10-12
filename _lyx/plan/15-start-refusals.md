# Card 15 — start-refusals

**Create:**
- `internal/loomcli#loomCLI.refuseCovered`
- `internal/battencli#battenCLI.refuseCovered`
- `internal/loomcli#TestStartCmd_RefusesWhileAPauseCoversTheRun`
- `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug`

**Edit:**
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.runInWindow`
- `internal/loomcli#TestResumeVerb`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Refuse `lyx loom start`, `lyx loom resume` and `lyx batten run` on a covered slug (D4), naming `lyx resume <slug>` or `lyx resume`, so the operator's own restart cannot start agent work under a pause and a reboot under a live entry does not restart it either.

- `loomCLI.refuseCovered(retry)` reads `pauserecord.ReadCoverage` at `hubgeom.PauseGeometry(c.location)` for the run's slug and returns `pauserecord.CoveredError(entry, slug, retry)` when covered; an unreadable record refuses nothing and logs one Warn.
- `startCmd`'s RunE calls it first, before it seeds or takes the bootstrap lock, with retry `run "lyx loom start" again`; `resumeCmd`'s RunE calls it before the bootstrap lock, with retry `run "lyx loom resume" again`.
- `battenCLI.refuseCovered` reads the same record (from the prime's `hubgeom.PauseGeometry` record path) for `c.slug`; `battenPreRun` calls it first, and `runInWindow` calls it beside its `checkRunStatus`, so `--window` refuses before any window opens, as it does for a done slug.
These are the operator's restart verbs; batten's own child spawn and revive under a pause are card 17's and card 14's concern, and `lyx loom step` is driven by a live driver whose agents the shuttle start hold covers.

Reachability: a seeded run whose slug a live slug or hub entry covers passes every earlier check (there is none before the refusal) and reaches it.

`refusal-spec.md`'s `## pause` section gains three rows, each class transient: `loom start under a pause`, `loom resume under a pause` and `batten run under a pause`, triggered by the verb on a slug a live entry covers, way forward "run `lyx resume <slug>` (or `lyx resume` for a hub-wide pause), then re-run the verb".

Tests: `TestResumeVerb` gains a `covered` field and a row whose location's record holds a live entry for the slug: the envelope refuses naming `lyx resume`, and nothing is typed (`wantSent` false).
`TestStartCmd_RefusesWhileAPauseCoversTheRun` drives `startCmd` on a hand-built receiver over a temp hub whose record holds a live hub entry, asserts the refusal names `lyx resume` and that no seed file was written, then resumes the entry and asserts the same call gets past the refusal (it fails later on the fixture's missing reed, not on the pause).
`TestBattenRun_RefusesWhileAPauseCoversTheSlug` calls `battenPreRun` and `runInWindow` on a wired prime whose record holds a live entry for the slug: both refuse naming `lyx resume <slug>`, no status file is seeded and no window is opened; a slug the entry does not cover seeds as today.
The budget rows of loomcli and battencli grow by one each.

**ImpactSummary:** One coverage check at the head of three operator verbs, two of loom's and batten's run; nothing changes when no entry covers the slug.
