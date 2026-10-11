---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/07-twill-config-value.md, _lyx/plan/08-discussion-spec-twill.md
    summary: No card edits contracts/specs/refusal-spec.md, so the reworded unknown-producer refusal leaves its loom row stale and the new fan-resolution refusal lands with no row, against PATTERN-refusal-way-forward
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-shuttle-await-verb.md, _lyx/plan/08-discussion-spec-twill.md, _lyx/plan/01-shuttle-await-poll.md
    summary: The docs constraint's loomengine and shuttlecli package docs are not carried; no card targets a loomengine or shuttlecli package doc, cli.go's verb list goes stale, and shuttleengine/doc.go never learns the shared poll
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-discussion-template-twill-markers.md
    summary: Card 6 claims the single and chair prompts stay byte-identical, but an empty optional marker on its own line renders a blank line, the stated placement does not rule that out, and its tests pin only the absence of twill text
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/02-shuttle-await-verb.md
    summary: Card 2 declares a second defaultAwaitCap of 250 s in shuttlecli beside burlercli's, so the one cap D9 gives both verbs is spelled twice instead of living beside AwaitAny
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/08-discussion-spec-twill.md
    summary: Card 8's stated bound for the fork edge omits the claude_deny_agent_tool false case, that the hook admits a fork spawned by a fork, and that the webster fork guard stays
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/08-discussion-spec-twill.md
    summary: TestDiscussionSpec_Twill re-asserts field mappings TestDiscussionSpec already pins, and its unresolvable-fan case is a refusal belonging in TestDiscussionSpec_Refuses
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/02-shuttle-await-verb.md
    summary: TestAwaitVerb's no-argument row duplicates the arity table TestRunCLI_PositionalArgValidation already holds for every shuttle verb
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/09-discussion-marker-fresh-spawn.md
    summary: TestRemoveDiscussionReadyMarker covers removal that the wiring test card 9 extends already drives through the PrepareDiscussionSpawn closure, at the cost of a budget raise
  - id: F9
    severity: LOW
    class: scope
    location: _lyx/plan/05-twilldirective-package.md
    summary: Card 5 adds empty-field refusals and test rows for every Values field, a guard against an internal caller that D3 does not license
  - id: F10
    severity: LOW
    class: design
    location: _lyx/plan/08-discussion-spec-twill.md
    summary: DiscussionReadyMarker is a public accessor reused across packages but is bundled into the spec card, while its removal sibling sits in card 9
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/plan/07-twill-config-value.md, _lyx/plan/08-discussion-spec-twill.md
    summary: Cards 7 and 8 rewrite file headers of config.go and discussion.go while their Edit targets name only member glyphs
---

### [BLOCKING] Refusal-spec rows missing for the changed and the new loom refusal

**Location:** `_lyx/plan/07-twill-config-value.md`, `_lyx/plan/08-discussion-spec-twill.md`

**Issue:**
`contracts/specs/refusal-spec.md` line 228, in the `## loom` section, holds the row `config: unknown discussion_producer`.
Its condition reads "neither `single` nor `seats`", and its way forward reads "set `discussion_producer` to `single` or `seats`".
Card 7 rewords that refusal (`internal/loomengine/config.go:661`) so the way forward names all three values, but no card edits the row, so the row then contradicts the message.
Card 8 adds a new loom refusal: `twillBlocks` fails the Discussion-Write spawn when `burler.yaml` is unreadable or `twill-discussion` does not resolve, naming the fan and `burler.yaml`.
`PATTERN-refusal-way-forward` binds loom: "A new refusal in a bound module lands with its row in the same commit".
The decision record lists that constraint by name for both errors.
The neighbouring rows `config: unknown fan` and `config: burler.yaml unreadable` (lines 223–224) show the shape such a row takes.

**Fix:**
Add `contracts/specs/refusal-spec.md` as a `**Prosa:**` target on card 7, updating the condition and way forward of the `unknown discussion_producer` row to name `twill`.
Add it to card 8 too, with a row for the twill fan-resolution refusal, covering both an unreadable `burler.yaml` and an unresolvable `twill-discussion`, each with its way forward.
The test card 8 already plans reaches the new row.

### [MEDIUM] Docs constraint: loomengine and shuttlecli package docs not carried

**Location:** `_lyx/plan/02-shuttle-await-verb.md`, `_lyx/plan/08-discussion-spec-twill.md`, `_lyx/plan/01-shuttle-await-poll.md`

**Issue:**
The decision record's last Constraint names "`loomengine` and `shuttlecli` doc.go, the directive package's doc.go, the loom template comment, and `docs/overview.md`".
The plan carries the directive package's `doc.go` (card 5), the template comment (card 7) and `docs/overview.md` (cards 1, 2, 8).
No card targets a package doc for `internal/loomengine` or `internal/shuttlecli`.
Neither package has a `doc.go`, and both keep their module docs in file headers:

- `internal/shuttlecli/cli.go:7` enumerates the verbs ("The verbs are run, interrupt, send and the read-only state").
  Card 2 adds `await` but targets only the `internal/shuttlecli#Command` glyph, and "Its header comment lists the new verb" reads as `Command`'s doc comment, so the file header's verb list goes stale.
- Card 8 says "The file header says the spec renders the twill block" for `discussion.go` without targeting the file.
  No card records the twill design in a loomengine package doc: the spec variant (D1), fork authorization (D6) and the ready marker and its removal (D7).
- Card 1 adds the shared capped poll to `internal/shuttleengine`, which does have a `doc.go`.
  CLAUDE.md's "Docs land in the same commit" covers cross-cutting infrastructure, and D9 frames the poll as the wait every agent uses, but card 1 does not touch that file.

`PATTERN-documentation-lifecycle` puts a built design in its package doc.
The focus question is settled: only `docs/overview.md` and the twill package doc are carried, and no card among 2, 6, 7 and 8 targets a loomengine or shuttlecli package doc.

**Fix:**
On card 2, add `internal/shuttlecli/cli.go` as a target, or create `internal/shuttlecli/doc.go`, and have the package header name `await` and its contract.
On card 8 (and card 9 for the removal), add a target that carries the twill design in loomengine's package-level doc: `internal/loomengine/discussion.go` as a file target for its header, or a created `internal/loomengine/doc.go`.
On card 1, add `internal/shuttleengine/doc.go` as a target and add a short paragraph on `AwaitAny`.

### [MEDIUM] "Byte-identical" single and chair prompts are claimed, not pinned

**Location:** `_lyx/plan/06-discussion-template-twill-markers.md`

**Issue:**
D2's rationale is that optional markers keep `single`'s rendered prompt byte-identical, and the acceptance criteria say "With `single`, the rendered Discussion prompt is unchanged".
`stencil.FillWith` (`internal/stencil/stencil.go:66-79`) renders an empty optional marker as an empty string and leaves the newline after it.
So a marker added on a line of its own between two existing lines adds one blank line to every `single` and chair prompt.
Today the template has `{{.mode_rules}}`, a blank line, then `## Step 5`, and `` - `{{.support_log_path}}` ``, a blank line, then `Two narrow carve-outs`.
Card 6 places each marker "on its own line" between those, which allows the extra-line form.
Only a marker that replaces the existing blank line keeps the empty render identical.
That placement has a cost of its own, and the card leaves both open.
The twill text then abuts its neighbours, because card 5 strips the block's trailing newlines, and a paragraph right after a list bullet becomes a continuation of the bullet.
Card 6's tests pin only that the rendered prompt "carries neither marker name and no twill text".
That passes with a stray blank line, so the acceptance criterion is never tested.

**Fix:**
In card 6, name the exact placement: each marker replaces the blank line it sits on, and the blocks carry their own surrounding blank lines when non-empty.
Pin the claim: assert in `TestComposePrompt_RendersMarkers` that the empty-twill render equals the render of the template with the two marker lines removed and nothing else changed.
Alternatively, pin the exact bytes around both insertion points: the mode-rules text followed by a blank line and `## Step 5`, and the support-log bullet followed by a blank line and `Two narrow carve-outs`.
Cover the chair render in `TestDiscussionTable` the same way.

### [LOW] The 250 s default cap is spelled twice

**Location:** `_lyx/plan/02-shuttle-await-verb.md`

**Issue:**
Card 2 creates `plan:internal/shuttlecli#defaultAwaitCap` (`250 * time.Second`).
Card 1 keeps `internal/burlercli#defaultAwaitCap` (`internal/burlercli/await.go:27`, same value) and only lists it under `Uses:`.
D9 gives both verbs one default cap tied to the prompt-cache lifetime.
Two literals in two packages let them drift, for example on a cache-TTL change, and card 1 already moves the shared loop into `shuttleengine`.

**Fix:**
In card 1, create `plan:internal/shuttleengine#DefaultAwaitCap` beside `AwaitAny`.
Delete `internal/burlercli#defaultAwaitCap` and have `awaitReviewCmd` use the new constant.
In card 2, drop the shuttlecli constant and use the same one.

### [LOW] Card 8's bound for the fork edge is incomplete

**Location:** `_lyx/plan/08-discussion-spec-twill.md`

**Issue:**
Card 8 introduces the edge that authorizes fork subagents on the twill spec.
Its bound reads: "the spec authorizes fork subagents only, which the engine admits while refusing every other subagent type under the config's Agent deny".
D6's bound also states three things card 8 leaves out:

- With `claude_deny_agent_tool` false, no Agent hook is installed and the writer and its forks may spawn any subagent type.
- The hook admits a fork spawned from a fork, so only the stencil refuses that.
- The `lyx webster` fork guard stays.

The rubric's attack-surface rule asks the introducing card to say what the change lets through and what bounds it.
"Fork subagents only" is untrue under the false setting.

**Fix:**
Extend card 8's bound sentence with those three facts as D6 states them.

### [LOW] TestDiscussionSpec_Twill duplicates existing assertions

**Location:** `_lyx/plan/08-discussion-spec-twill.md`

**Issue:**
A new test is warranted for the twill prompt content and the `burler.yaml` override.
Several of its planned assertions are covered elsewhere already:

- "keeps `single`'s output files, role, segment, skills, model, effort and timeout" is the field mapping `TestDiscussionSpec` pins (`internal/loomengine/discussion_test.go:24`), whose `testtiming:keep` names exactly those fields.
- "with `single`, the prompt carries no `twill` text and `ForkSubagents` is false" is what card 6's `TestComposePrompt_RendersMarkers` rows and card 8's own `TestDiscussionSpec` edit already assert.
- The unresolvable-fan case is a refusal, and `TestDiscussionSpec_Refuses` (`discussion_test.go:89`) is the table of `DiscussionSpec` refusals.

**Fix:**
Add a producer column to `TestDiscussionSpec`'s table, with a `twill` row asserting the shared fields and `ForkSubagents`.
Add the unresolvable-fan row to `TestDiscussionSpec_Refuses`.
Keep `TestDiscussionSpec_Twill` for the rendered twill block and the `burler.yaml` override only.

### [LOW] TestAwaitVerb's no-argument row duplicates the arity table

**Location:** `_lyx/plan/02-shuttle-await-verb.md`

**Issue:**
`TestRunCLI_PositionalArgValidation` (`internal/shuttlecli/cli_test.go:175`) is the table pinning each shuttle verb's positional-argument rule through cobra's `Args`, which runs before `PersistentPreRunE`.
Card 2's `TestAwaitVerb` also plans the row "no argument is an error".

**Fix:**
Add an `await`-with-no-argument row to `TestRunCLI_PositionalArgValidation`, and drop that row from `TestAwaitVerb`.

### [LOW] TestRemoveDiscussionReadyMarker covers what the extended wiring test drives

**Location:** `_lyx/plan/09-discussion-marker-fresh-spawn.md`

**Issue:**
Card 9 extends the loomcli wiring test that asserts the discussion seams (`internal/loomcli/wiring_test.go`, beside `TestWire_DiscussionSpecEvaluatesToExpectedShape`).
The extension asserts that a marker planted at `DiscussionReadyMarker(loc)` is gone after the `PrepareDiscussionSpawn` closure runs.
That closure is `RemoveDiscussionReadyMarker`'s one caller, so the new top-level test repeats the success case through a lower surface and adds a `test-budget.yaml` raise.
`loomengine` already sits at its row of 41.

**Fix:**
Fold the absent-marker case into the wiring test's extension by running the closure a second time and expecting nil.
Drop `TestRemoveDiscussionReadyMarker` and the budget raise it needs.
If the error case must be pinned, add it to that same extension.

### [LOW] Empty-field refusals in twilldirective are unlicensed

**Location:** `_lyx/plan/05-twilldirective-package.md`

**Issue:**
Card 5 has `Directive` refuse an empty value for each of eight `Values` fields and `WriteScope` refuse an empty marker, with a test row per field.
D3 asks only for a renderer that reads at call time.
The one caller is `loomengine`, which the codebase controls.
Elsewhere, `stencil.Fill` already refuses an empty required marker and names it, which covers every scalar field.
The added refusals and their rows are scope the answer key does not license.

**Fix:**
Drop the per-field refusals and their test rows, and rely on `stencil.Fill`'s unfilled-marker error.
Keep only the stencil-read error that names the stencil.

### [LOW] DiscussionReadyMarker's card cut

**Location:** `_lyx/plan/08-discussion-spec-twill.md`

**Issue:**
`DiscussionReadyMarker` is a public loomengine accessor whose consumers are `twillBlocks` (card 8) and the `loomcli` wiring closure (card 9).
The rubric's granularity rule gives a symbol reused across packages its own card.
Its natural sibling `RemoveDiscussionReadyMarker`, together with the `TestDiscussionPaths` assertion, would form that card.
Instead the accessor rides inside the spec card and the removal inside the wiring card.

**Fix:**
Move `DiscussionReadyMarker`, `RemoveDiscussionReadyMarker` and the `TestDiscussionPaths` extension into one card placed before the spec card.
Cards 8 and 9 then list both accessors under `Uses:`.

### [NIT] File-header edits outside the cards' targets

**Location:** `_lyx/plan/07-twill-config-value.md`, `_lyx/plan/08-discussion-spec-twill.md`

**Issue:**
Card 7's Intent rewrites `config.go`'s file header ("The file header's sentence about `discussion_producer` names the three values", line 11), and card 8 rewrites `discussion.go`'s header.
Both list only member glyphs (`internal/loomengine#LoadConfig`, `internal/loomengine#DiscussionSpec`), and a file header belongs to neither member.

**Fix:**
Add `internal/loomengine/config.go` to card 7's `**Edit:**` list and `internal/loomengine/discussion.go` to card 8's.

## Verdicts on the focus entries with no finding

- D5 and D9 numbers: the 250 s cap (card 2), the 300000 ms Bash timeout (cards 2 and 4), "15 times in a row" and "at most 6 further tool calls" (card 4) match the decision record.
  The only unlicensed scope found is F9.
- D4 and D7: card 4 carries the one-pass rule and the relaunched writer's resume-and-record wording.
  Card 9's removal runs in `SingleLLMProducer`'s `prepareFreshSpawn` (`internal/shedadapters/singlellm.go:85-88,162`), which runs only after the attach probe finds no live strand.
  Reed's relaunch of a dead process happens inside a still-tracked strand, so that path is an attach and keeps the marker.
  Card 4's write-scope paragraph covers both the lead-in and the do-not-repair rule, and keeps the commit sentence about the two files.
  Not a defect.
- Card order and `Uses`: card 8's second `DiscussionSpec` edit uses only symbols from cards 3, 5 and 7.
  Card 2's `Uses` of `burlerCLI.awaitReviewCmd` follows card 1's edit.
  Not a defect.
- Verify: the section runs in the template's stated order (`loom-template-plan.md` line 179), and `./...` covers every targeted package, the integration tier included.
  Not a defect.
- Re-sign arrows: card 6's `composePrompt` carries the arrow.
  `Env` gains a struct field and takes none, and `discussionWriteEntry`, `loomCLI.wire` and `burlerCLI.awaitReviewCmd` keep their signatures.
  Not a defect.
- The decision record's acceptance criterion on a gated turn end with forks outstanding is met by existing shuttle behaviour: `expiredTurnEnd` (`internal/shuttleengine/wait.go:820-828`) never counts a gated run's waiting turn end as an arrival.
