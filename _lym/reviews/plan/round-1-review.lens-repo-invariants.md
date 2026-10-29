```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens on the plan cards (test seams, doc text the cards falsify, card order).

**Per-card list**
- Card 1: no seam is touched (held). The `doc.go` text names no URL shape (held). Order is fine (held).
- Card 2: `dobbyengine.Config` gains a key, which configreg classifies; that test is listed (held). The doc is listed (held).
- Card 3: `boardengine.Task` gains a field, and `replayFields` and its tests are listed (held). The boardcli doc was not checked.
- Card 4: `landingshed.Config` gains `PRReviewer`, and the template is listed (held). The gateslot doc is listed (held).
- Card 5: `Spec.Unawaited` (held for seams). Doc text is **broken**: the `wait.go` precondition is falsified, and the package doc is listed only under Uses although the card edits it (F2).
- Card 6: `Spec.DenyPublish` (held). Doc text is **broken**: the shellguard and claudeengine docs are edited but listed only under Uses (F3).
- Card 7: new test files carry tier0, and `testmain_test.go` is untagged support (held). The `overview.md` table is listed (held).
- Card 8: new test file is tier0 (held). The PATTERN entry is listed (held).
- Card 9: test goes into `commitrecords_test.go`, which is tier0 (held). Order: it uses card 7 (held).
- Card 10: the `roleOpeningStencils` scans in `parentdirective_test.go` cover the new stencil (held).
- Card 11: test seams are **broken** (F1). Doc text is **broken**: `loomrecipe/doc.go` (F4). The `overview.md` stub lines and the `shedrecipe` doc are listed (held).
- Card 12: every `pr_review` reader the grep found is covered by cards 11 and 12 (held).
- Card 13: test seams are **broken**: it adds more `PullRequestDeps` seams to the same unlisted tests (F1).
- Card 14: test seams are **broken**: `Sessions`, `SessionLive` and `RegisterSession` go to the same unlisted wire tests (F1). The marker on `TestLandingDeps_EveryFieldPopulated` arrives here, three cards after the struct (F1). The runproc doc is listed, and the pattern background lists no roles (held).
- Card 15: the PATTERN line and landingshed doc are listed (held).
- Card 16: no production file makes a mutating `.PullRequests.Create(`, `.Edit(` or `.Merge(` call today, so the new guard passes (held). `ghguard_test.go` is tier0 (held).
- Card 17: `childAtBeamAwait` has no reference outside `wire.go` (held). The battenshed doc is listed (held).

**Findings**

**F1 — BLOCKING, design, cards 11, 13 and 14: the nested `Deps.PullRequest` seams break the tier0 seam scan and the wire seam tests, and no card edits those tests.**
- `PATTERN-test-speed.md:94` says the tier0 scan refuses a test that reaches `envkit.NilSeams` when its markers miss a struct in the `Env` set. Four tests reach it with `//lym:seam internal/landingshed#Deps` and none gains a `#PullRequestDeps` marker:
  - `internal/battencli/wire_seams_test.go:124-142`
  - `internal/loomcli/wiring_darn_test.go:43-46`
  - `internal/loomcli/wiring_beam_test.go:33-36`
  - loomcli's `TestWire_EverySeamFilled`
- `wire()` leaves `Env.Landing`'s func seams nil and lists each one in `intentionallyNil` (`internal/battencli/wire_seams_test.go:73-80`, `internal/loomcli/wiring_seams_test.go:16-21`). Each test fails on any nil path missing from that map (`wire_seams_test.go:159-163`). So every new `Landing.PullRequest.*` seam needs an entry there.
- Cards 4 and 11 list these tests only under **Uses**. Card 11 lists `TestWire_BeamRecipe` under **Edit**, but only for its row list.
- `TestLandingDeps_EveryFieldPopulated` gets its `#PullRequestDeps` marker only in card 14, although card 11 adds the nested struct (`internal/loomcli/landingdeps_test.go:42`).
- As planned, card 11 cannot pass its own tier0 gate.

**F2 — MEDIUM, consistency, card 5: it falsifies shuttle's empty-OutputFiles precondition in `wait.go` without listing the file.**
- `internal/shuttleengine/wait.go:73-76` says no `*Run` with an empty `OutputFiles` ever exists, because `Spec.validate` refuses one. The same claim recurs at `wait.go:1498` and `wait.go:1529-1532`.
- `StartSession` creates exactly such a run.
- The card also says the package doc gains a paragraph, yet lists `internal/shuttleengine/doc.go` only under **Uses**.

**F3 — LOW, consistency, card 6: two docs it edits are listed only under Uses.**
- Its Intent changes `internal/shellguard/doc.go` (names the switch) and `internal/shuttleengine/claudeengine/doc.go` (the deny list).
- Both appear only under **Uses**, with no Edit or Extends target.

**F4 — MEDIUM, consistency, card 11: adding `PR-Review` to the pawl recipe falsifies the pawl row sequence in `loomrecipe/doc.go`.**
- `internal/loomrecipe/doc.go:30` reads `Beam-Up → Beam-Await → Parent-Align → Code-Review → Beam-Describe → Finalize → Beam-Down`. No card lists that file.
- `contracts/recipes/pawl-recipe.yaml:10`'s header row list also omits the row. That file is card 11's own target, but the Intent only rewrites the "stub" comments.

**Tool calls**
1. A grep across shuttleengine, stencils, battencli and the docs for `OutputFiles` claims, `//lym:seam` markers, `roleOpeningStencils` tests, `childAtBeamAwait` references and `pr_review` mentions. The manifest had none of these readers.
2. A grep for battencli's landing wiring, existing mutating `PullRequests` calls, runproc roles in the pattern file, test-file tier tags and docs naming PR-Review. These are needed for the guard, role and doc checks.
3. The battencli seam test body, the impactset seam scan and the pawl-tail docs. These confirm what the seam test fails on and which docs list the pawl sequence.
4. loomcli's seam markers, the `intentionallyNil` `Landing.*` entries and battencli's Landing wiring. These confirm the wire tests' nil handling.
