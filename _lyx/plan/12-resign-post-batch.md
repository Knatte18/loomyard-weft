# Card 12 — resign-post-batch

**Create:**
- `plan:internal/planglyph#RebindResigns` -> `func RebindResigns(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]Finding, error)`
- `plan:internal/planglyph#index.RebindResigns` -> `func (index) RebindResigns(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]Finding, error)`
- `plan:internal/planglyph#TestRebindResigns` -> `func TestRebindResigns(t *testing.T)`

**Edit:**
- `internal/planindex#Index`
- `internal/websterengine#postBatchChecks`
- `internal/websterengine_test#TestRecordBatch_PostBatchChecks`
- `internal/websterengine_test#TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch`

**Uses:**
- `plan:internal/planglyph#nameDeclarations`
- `plan:internal/planglyph#nameResigns`
- `internal/planglyph#copyGlyphChainFixture`
- `internal/planglyph#writeLanguagePlan`
- `internal/planglyph#resignCard`
- `internal/planglyph#readCardFile`
- `internal/websterengine_test#recordScenario.restart`
- `internal/testkit/plankit#Write`
- `internal/websterengine#Fingerprint`

**Intent:** Carry a recorded C# re-sign into the plan (D7), in the post-batch pass record-batch and recover-batch share.

`RebindResigns` takes the batch's cards and, under a glyph-enabled plan, names each card's C# arrow again through `nameResigns`' declaration rule and `nameDeclarations`.
For every arrow whose minted id differs from its target it resolves the minted ids against `worktreeRoot` in one `resolveTargets` call:
- an answer other than `found` is the new blocking done check `resign-not-done` for that card, its detail naming the arrow, the answer and the way forward: make the edit match the arrow's head, or correct the head to the signature the card wrote and run `lyx webster rebaseline --card <card number>` naming that card;
  the batch's `lyx webster recover-batch` step comes from websterengine's not-done way forward, which knows the batch number;
- when every such answer is `found`, it rewrites each old glyph to its minted id plan-wide in one `plan.RewriteRefs` call, as handle binding does, so a later card naming the member by its old glyph names the minted id and resolves at its own dispatch.
A Go arrow, an arrow whose minted id equals its target, and a `none` plan do nothing.
An infrastructure error is wrapped in `ErrQuarryUnavailable`.

`planindex.Index` gains the method `RebindResigns(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]Finding, error)`, which `index.RebindResigns` forwards; the websterengine test fake embeds the real index and needs no change.

`postBatchChecks` calls it right after the done checks, on the same terms: a blocking finding is an `ErrCardNotDone` reason, so the batch is not done.
After a rewrite it runs `restampFingerprint` before the scope guard and `DetectDrift`, so the next begin-batch meets no fingerprint mismatch, then re-reads the plan through `in.Plan.Reparse()` and uses the re-read plan for everything after: handle binding, `DetectDrift`'s full plan, so no later card still names the old glyph there, and the scope guard, whose cards are the batch's cards as recorded plus their re-read form, so its target union holds both the old glyph and the minted id.
A plan with no C# re-sign rewrites nothing and runs exactly as today.

Tests:
- `TestRebindResigns` is one table over a fixture copy and a `csharp` plan with a re-sign card and a later Edit card naming the old glyph:
  the arrow `Glyphchain.Sharp#Renderer.Draw(int)` -> `public void Draw(long count)`, after the fixture's `Draw(int count)` is edited to `Draw(long count)`, returns no finding and leaves the later card naming `Glyphchain.Sharp#Renderer.Draw(long)`;
  the same plan over the unedited fixture returns one `resign-not-done` whose detail names `rebaseline --card` and rewrites no card file;
  two arrows on one card, one landed and one not, return one `resign-not-done` and rewrite no card file;
  an arrow whose minted id equals its target, such as `Glyphchain.Sharp#Renderer.Title` -> `public string Title { get; set; }`, returns nothing and rewrites nothing;
  and a `go` plan's arrow returns nothing.
- `TestRecordBatch_PostBatchChecks` gains a step whose repository's base commit adds `csharp/Renderer.cs` declaring `Draw(int count)`, and whose plan directory is written with `plankit.Write` as a `csharp` plan: card 1, batch 1, re-signs `Glyphchain.Sharp#Renderer.Draw(int)` to `public void Draw(long count)`, and pending card 2 names it under Uses.
  With the work commit changing `Draw(int count)` to `Draw(long count)`, the batch records done, card 2's file names `Glyphchain.Sharp#Renderer.Draw(long)`, `websterengine.Fingerprint` of the plan directory equals the state's `PlanFingerprint`, the fingerprint begin-batch compares, the digest's deviations hold no `Renderer.cs` entry, and the warnings hold no `later card:` warning.
  A second step whose commit leaves `Draw(int count)` fails with `ErrCardNotDone` naming `resign-not-done`.
- `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` gains the recovered form: the fixture's worktree gets `csharp/Renderer.cs` declaring `Draw(long count)`, its `Geom.PlanDir` gets the same two-card `csharp` plan written with `plankit.Write`, and its batch's cards come from that parsed plan; the recovered batch's terminal record rewrites card 2 to the minted id, and `websterengine.Fingerprint` of the plan directory equals the state's `PlanFingerprint`.

**ImpactSummary:** one new index method and one new step in the shared post-batch pass; a plan with no C# re-sign takes the pass unchanged, and the one implementation and the embedding test fake need no other edit.
