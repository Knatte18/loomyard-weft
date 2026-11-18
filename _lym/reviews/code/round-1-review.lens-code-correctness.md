```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each new branch and error path against concrete inputs. I found no defect that blocks.

**Per function or branch (case ran → result)**

- `sendBackEdge` (`internal/loomcli/prfindings.go`):
  - No `OnReroute` → fix-here, correct.
  - Target missing from the rows → error, correct.
  - Entry gate false → fix-here, correct.
  - Entry gate error → error, correct.
  - `MaxBounces == 0` → fix-here. This is correct for review-segment rows, because `loomrecipe.applyReviewBudget` sets an explicit budget on every segment row (`internal/loomrecipe/loomrecipe.go:199`), so 0 means a zero-round segment. It would be wrong for a row that inherits a budget through `effectiveMaxBounces`, but Code-Bouncer is not such a row.
- `prFindingsVerb`:
  - Dirty tree → refusal with way forward, correct.
  - Unparsable file → refusal, correct.
  - No record → refusal, correct.
  - A mark already present → same send-back again, writes nothing, correct.
  - A repeated file → counted once by its sha256, correct.
  - Threshold `total > smallMax` with `total` including the current set only when not yet counted → correct, with 0 findings and an APPROVED verdict it stays fix-here.
  - Send-back with a crash between the findings-file write and the record update → a re-run rewrites the same `pr-findings-n.md` (correct), but see F1.
  - Failed commit → error envelope, and the re-run completes without counting twice, correct.
- `settleSendBack` / `reroutePersisted` (`internal/landingshed/prreview.go`):
  - No mark → nothing, correct.
  - Newest own entry is the reroute of the same file → one write clears the mark, count, counted sets, ask flag and session, correct.
  - Newest own entry is a Stuck, an unrelated producer, a reroute of another file, or a reroute-cap Stuck → mark and count kept, send-back repeats, correct.
  - Second send-back after a clear (mark file 2, newest entry is the reroute of file 1) → kept, correct.
  - Record or status read error → returned with `ended` true, correct.
- `sendBack`: dirty tree → halting Stuck with way forward, correct. Clean tree → Stuck with `Reroute` set and the file as output, clears nothing, correct. A `wake` that meets the mark runs the same function before any gate, correct.
- `Bouncer.reenterUnchanged` / `startNextGeneration`:
  - A full input over an unchanged subject → full generation, no amendment mark, correct.
  - A full input with `FullGeneration` set → full generation, correct.
  - Allowance-0 round-0 settle plus a later full input → generation 3, correct (covered by a test).
  - A non-full input with `FullGeneration` set → re-settle, correct.
  - Seam error → returned before the archive, correct.
  - A legacy record with a single `review_input` → read as one path, correct.
  - A full generation now keeps non-full inputs, such as the fixer report, in its record. The old code kept none. This is harmless: only the Code-Review rubric carries the marker.
- `reviewInputs` (`internal/loomcli/subjects.go`):
  - Highest-numbered findings file wins, numeric rather than lexical, so 10 beats 2, correct.
  - A non-matching name such as `pr-findings-notes.md` is ignored, correct.
  - Scratch directory absent → no input, correct.
- Drift:
  - `listFiles` with a nil `ReviewBase` → no file, correct.
  - Key = sha256 of the diff, a NUL and the sorted unique trailer lines → a trailer-only commit changes every key, correct.
  - Stored `uncaused` ruling with an unchanged range → Stuck without a judge, correct.
  - `passWithoutJudge` sets `Passed` false on an uncaused check, correct.
  - `storeRulings` / `storeCaseRulings` put the no-match reason before the uncaused one, correct.
  - `passCommand` with exit 0 and an unruled file → judge spawned with an empty `CompareLog`, so `CompareFailed` is false, correct.
  - `passCommand` with exit 0 and no unruled file → pass, correct.
  - `passCommand` with a failing compare and stored non-matching rulings → halts before any file judge. That is the old behaviour, and the file rulings wait for the resume.
  - `filesFindings` / `checkFindings`: unknown, duplicate, missing, empty-reason, none-with-checks, loosened-without-checks and caused-without-cause all refused, correct.
- `CommittedFileDiffs`: `--diff-filter=MD` with `--no-renames` lists a rename by its old path and skips added files, correct. `-z` avoids quoted paths.
- `CommitMessages`: an unresolved ref → error naming it, an empty range → empty slice, correct.
- `ReadForEngine` plain includes:
  - A missing plain fragment → error with way forward, correct.
  - Engine blocks inside the fragment are resolved, correct.
  - The fragment's own plain includes are left as they stand, correct.
  - Brief readers `darn/spec.go`, `samplerengine`, `websterengine`, `gatefixer`, `weaverengine` batch and `landingshed/prsession.go` all go through `ReadForEngine`, so the new `{{template "loom-fragment-loosened-check"}}` is resolved for them.
- `loomengine` `PRReviewSmallMax`: loaded like the other keys, correct.
- `pullrequest.Record`: new keys are omitempty and unknown keys survive, correct.

**Findings**

**F1 — LOW, design.** A send-back that crashes after `writeSendBackNote` but before `UpdateRecord` leaves a duplicate note on the re-run.
- Location: `internal/loomcli/prfindings.go:249`
- The re-run recomputes the same `n`, and `friction.NotePath` hands out a free name, so the first note stays and a second one for the same send-back appears.
- This breaks the one-note-per-send-back rule of D6.
- Fix: write the note after the record update, or reuse the `pr-review-send-back-<n>` path.

**F2 — NIT, design, unverified.** The `landing-template-fixer.md` include is not confirmed to be resolved.
- Location: `internal/landingshed/fixer.go:106,139`
- The code reads the stencil with `stencilstore.Read` and then renders it by name through the shuttle spec at `fixer.go:139`.
- Whether that route resolves the new `{{template "loom-fragment-loosened-check"}}` was not confirmed. The same applies to the `weaver-body-verify-fix` and `webster-body-verify-fix` readers.
- `websterengine` reads its verify-fix body through `ReadForEngine`. The `weaverengine/render.go:30` read path was not confirmed.
- The handler or the affected-set lens should check that each of these renders without an unresolved include action.

**Tool calls**
1. grep `ZeroRounds|zero_rounds` and `MaxBounces` in the shed packages, because my context lacked whether 0 means a zero-round segment or inherit.
2. grep `ReadForEngine(` callers and stencils with plain includes, to see which briefs resolve the new fragment.
3. grep `MaxBounces` in `loomrecipe`, `shedbuild` and the recipes, to confirm segment rows carry an explicit budget.
4. grep for `landing-template-fixer` and the verify-fix stencil readers, because I had not seen how those briefs are read.
5. grep `fixerStencilName` and `verifyFixBodyStencil` use sites, to see how the fixer and weaver stencils are rendered.
