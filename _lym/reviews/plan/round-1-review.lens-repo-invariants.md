```yaml
lens: repo-invariants
outcome: reported
```

Task: review the plan through the repo-invariants lens (test seams, doc text the cards falsify, card order).

**Per card: each invariant checked, held or broken**
- **Card 1:** Doc text: `PATTERN.md`, `pattern/PATTERN-lym-files.md` and the `DotLymDirName` doc are all edit targets. Held.
- **Card 2:** Doc: the new package's `doc.go` is created and `PATTERN-leaf-packages.md` gains a section. Tier: every test is tier 0. Seams: none. Held. The dobbyengine allowlist update falls on card 11, which lists it.
- **Card 3:** Doc: the "six legal" comment on `shedengine/status.go:13` is changed in the card. Import allowlist: shedengine gains no import. Seams: none. Held.
- **Card 4:** Card order: `shedrun` importing `shedengine` forms no cycle, because shedengine's tests do not reach shedrun. Held. No allowlist guards shedrun's imports. Held.
- **Card 5:** Tier 1 with hermetic git. No seams. Held.
- **Card 6:** Card order: **broken**. A pairteardown import of `battenrecipe` creates a test import cycle (finding 1).
- **Card 7:** Seams: `PublishRecord` is a func field inside the env that `envkit.NilSeams` walks. Card 7 lists `TestWire_EverySeamFilled`, `TestNilSeams_*` and `TestFullEnv_FillsEverySeam`. Held. Doc: `pattern/PATTERN-daemon-wakeups.md:58` watches the record through the path constructor, so it is still true. Held.
- **Card 8:** Seams: the envkit and wire tests are listed. Held. Doc: the `fixerattempts.go` file header is falsified (finding 4).
- **Card 9:** Doc: **broken** in `shedrun/doc.go`, `pairteardown/doc.go`, `battencli/cli.go` and `dobbyengine/doc.go` (findings 2 and 3). Seams: `cancelDeps` has no reflective test. Held.
- **Card 10:** Doc: `Teardown`'s doc is listed. Seams: `cleanupDeps.PairStands` has no reflective test. Held.
- **Card 11:** Allowlist: `dobbyengineAllowedImports` is listed. Held. Doc: `dobbyengine/doc.go` is falsified (finding 2).
- **Card 12:** Seams: `TestWire_EverySeamFilled` walks only `c.env`, so the `askAct` field needs no entry. Held.
- **Card 13:** Seams: `TestNewLoomCLI_SetsInjectedSeams` checks its fields by hand, not by reflection. Held.
- **Card 14:** Seams: no reflective test covers `tenderengine.Deps`. Held.

**Findings**

1. **BLOCKING · design · card 6, pairteardown's import of battenrecipe**
   Card 6 has `pairteardown` use `battenrecipe#RowsBeforeRunShed`, which forms an import cycle when tests build, so card 6 cannot pass its own gate.
   The path is: `battenrecipe`'s dependencies include `preflightshed`. `preflightshed`'s in-package `preflight_integration_test.go` imports `hubforge`, and `hubforge` reaches `pairteardown`.
   `internal/pairteardown/followup.go:18-25` already holds the batten row names as local constants for this reason, with `TestBattenFollowUp` pinning them.
   Fix: hold the Run-Shed row and the rows before it in pairteardown in the same way, pinned by a test, or have the caller pass them in.
   Evidence: `go list -test -deps` over battenrecipe's deps showed `preflightshed` reaching pairteardown; the import line is in `internal/preflightshed/preflight_integration_test.go`.

2. **MEDIUM · design · cards 9 and 11, the method for package docs**
   Each card lists its package's `doc.go` only under `**Uses:**`. The doc text describing the removed cancel marker therefore goes stale, and no card edits it:
   - `internal/shedrun/doc.go:51-52` describes `CancelMarker` and `CancelMarkerLock`, which card 9 deletes.
   - `internal/pairteardown/doc.go:154` says "no cancel marker of the slug stands".
   - `internal/dobbyengine/doc.go:154,286,338,385` describe the planner, `strandedEnded`, the reminders and the notices as reading the cancel marker. Card 11 replaces that with a cancelling `Phase`.

   Fix: list each `doc.go` under `**Edit:**` on card 9 (shedrun, pairteardown) and card 11 (dobbyengine), saying what the text becomes.

3. **MEDIUM · consistency · card 9, battencli's comments on the marker**
   - The file header at `internal/battencli/cli.go:23` says the cancel "takes the cancel marker: a record whose writer's liveness is a lock it holds".
   - The doc of the `preDrive` field at `internal/battencli/cli.go:282` says it "re-reads the cancel marker".

   No card targets the file's self glyph or the `battenCLI` struct for this. Card 12 edits `battenCLI` later, but its Intent does not touch this field's doc.
   Fix: add `internal/battencli/cli.go` and `internal/battencli#battenCLI` to card 9's Edit, saying the text names the cancel intent.

4. **LOW · consistency · card 8, `internal/landingshed/fixerattempts.go` file header**
   Line 2 says "The record lives in the run's scratch directory". Card 8 moves the record to the records directory but edits only `FixerAttemptsPath`'s doc, not the file header.
   Fix: add the file's self glyph to card 8's Edit.

Tool calls:
- grep of the touched packages' `doc.go` files, `battencli`, `pattern/` and `PATTERN.md` for marker, PR-record and fixer-attempt text: the manifest held none of that doc text.
- grep for reflective seam tests and import allowlists in the touched packages' tests: the manifest did not cover the seam tests.
- grep of the reflective calls in the tender and shedengine tests: needed to tell DeepEqual assertions from seam walks.
- read of `TestWire_EverySeamFilled`, the loomCLI seam test and `envkit.NilSeams`: needed to learn which structs the seam tests walk.
- `go list -test -deps` over shedengine and its in-package test imports: needed to settle whether card 4's shedrun-to-shedengine import forms a test cycle.
