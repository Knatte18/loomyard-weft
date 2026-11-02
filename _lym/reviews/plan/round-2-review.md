---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/09-goto-ends-fixers.md:34
    summary: Card 9 has endGateFixers return a pair-slug read error marked transient, which PATTERN-transient-stop forbids here and which no class or reader supports
    fix: >-
      In _lym/plan/09-goto-ends-fixers.md, rewrite the first bullet under Intent that describes endGateFixers (line 34):
      replace "A slug read error is returned marked transient; an `end` failure is returned wrapping the role and the error."
      with "A slug read error is returned wrapping the read error, and an `end` failure is returned wrapping the role and the error; neither carries a shed transient mark."
      Keep the sentence that follows it, since shedengine's guard check adds goto's re-run way forward.
      In the Tests section (line 51), replace "a slug read error is returned marked transient and calls nothing" with "a slug read error is returned and calls nothing".
      No target changes, so no overlay is needed.
  - id: F2
    severity: LOW
    class: scope
    location: _lym/plan/03-webster-fixer-on-core.md:73
    summary: Card 3 rewrites comments in render.go, strand.go, recoverbatch.go and template_test.go that name deleted members, but lists none of those sites under a target
    fix: >-
      Two changes.
      First, in _lym/plan/03-webster-fixer-on-core.md add the member glyph `internal/websterengine_test#testReplyPath` under **Edit:**, after `internal/websterengine_test#TestRenderPrompts_ParentDirective`,
      since its doc comment (template_test.go lines 375-376) names the deleted RenderVerifyFixColdPrompt; drop the "template_test.go comment on testReplyPath at line 375" bullet from the comment list, which the new target now covers.
      Second, move the three file-header comment bullets out of card 3, the render.go lines 1, 4, 25, 32 and 35 bullet, the strand.go line 1 bullet and the recoverbatch.go line 1 bullet,
      into a new `Prosa` card inserted right after card 3 with `lym loom insert-card`, its targets the file self glyphs `internal/websterengine/render.go#`, `internal/websterengine/strand.go#` and `internal/websterengine/recoverbatch.go#`,
      and its Intent stating each header's post-change wording: the shared frame and RenderPrompt in place of webster-prefix-verify-fix-cold and RenderVerifyFixColdPrompt, gatefixer's StrandLive and TurnEndedAfter in place of strand.go's own, and gatefixer's members in place of webster's in recoverbatch.go.
      The file glyphs cannot stay on card 3: `redundant-file-target` blocks each of the three beside the deleted or edited member glyphs of the same files, and a `validate-plan --overlay` run with them added reports exactly those blocking findings.
      The `testReplyPath` addition alone passes the overlay run with no new blocking finding; the new card goes through insert-card, so it is not overlaid.
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/09-goto-ends-fixers.md:28
    summary: Card 9 says fixers end once shed has admitted the move, but Relaunch runs the guard before Admit, so a refused or in-effect consume has already ended them, and the guard gains a side effect GotoGuard documents it lacks
    fix: >-
      In _lym/plan/09-goto-ends-fixers.md, reword the opening of the Intent (line 28) from "once shed has admitted the move, and before it writes it" to "once shed's own checks and the told guard have admitted the target, and before the move is written".
      In the bullet at line 44, state both cases the order leaves open: a targeted consume whose target the guard admits and `AdmitRelaunch` then refuses, and one `AdmitRelaunch` reports already in effect, have each ended the fixers without a write;
      the in-effect case ends none that are live, because the run stands paused after a goto that ended them.
      Add `internal/shedengine#GotoGuard` under **Edit:** and `internal/shedengine/doc.go` under **Uses:**, and add to the Intent that GotoGuard's doc comment (`internal/shedengine/goto.go` lines 17-20, which now says the guard only decides, moves no run and writes no file) gains one sentence:
      once it admits, a guard may end processes the move would leave running, and it still writes no file and moves no run.
      A `validate-plan --overlay` run with these targets, together with F4's `Extends`, reports no blocking finding the baseline lacks.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/09-goto-ends-fixers.md:25
    summary: Card 9 lists internal/loomcli/doc.go under Uses although its goto-guard paragraph, which names two arms, no longer describes the guard once the wrapper ends fixers
    fix: >-
      In _lym/plan/09-goto-ends-fixers.md move `internal/loomcli/doc.go` from **Uses:** to a new **Extends:** group placed before **Uses:**, and add an Intent bullet saying doc.go gains a sentence after its two-arm paragraph (doc.go lines 216-219):
      the guard is wrapped so that once both arms admit a goto, or a consume's target, every fixer role registered under the pair is ended before the move is written.
      The existing two-arm text stays true, so `Extends` is the right label and `extends-overlap` does not apply.
      The overlay run that adds this change passes with no blocking finding.
---

### [BLOCKING] Card 9 marks a slug read error transient

**Location:** `_lym/plan/09-goto-ends-fixers.md:34` and the matching test row at line 51.

**Issue:** The card has `endGateFixers` return a pair-slug read error "marked transient", and `TestEndGateFixers` asserts that mark.
`PATTERN-transient-stop` reserves the mark: it is `internal/shedengine`'s, translated by `internal/shedtransient` at a producer or step-bootstrap boundary.
A goto guard in `internal/loomcli` is neither boundary.
No class fits either: `TransientClass` is a closed set of git-transport, github-api, agent-start and host-resources failures (`internal/shedengine/transient.go`), and a failed read of the fabric origin is none of them.
Nothing would read the mark: `TransientOf` is read at `internal/shedverbs/step.go:371`, `:382` and `:405` and at `internal/shedengine/run.go:402`, none of which sees a guard error, and `Goto` and `Relaunch` return the guard's error as a refusal that sets no transient mark.
The card itself says the error "names no way forward of its own" and reaches the caller through `checkGuard`, so the mark adds nothing and cannot be asserted with any class.
Card 1's and card 3's "wrapped as transient" are the way-forward wording of webster's existing errors, not this mark; card 9 is the one place that reads as a shed mark.

### [LOW] Card 3's comment edits in four files have no target

**Location:** `_lym/plan/03-webster-fixer-on-core.md:73-77`.

**Issue:** Card 3 says these comments drop the deleted names, and lists the sites in its Intent.
Three of them are covered by a target: `doc.go` (lines 423 and 560) is under **Edit:**, the `ClassifyInputs.TurnEnded` comment sits inside `internal/websterengine#ClassifyInputs`, and the `WebsterBodyVerifyFix` doc comment is under `contracts/stencils#WebsterBodyVerifyFix`.
Four are not: `internal/websterengine/render.go` lines 1, 4, 25, 32 and 35 (all before `package`, so file-level comments, not a member's doc), `strand.go` line 1, `recoverbatch.go` line 1, and the `testReplyPath` doc comment at `template_test.go:375-376`.
`lym quarry resolve` shows `render.go#`, `strand.go#` and `recoverbatch.go#` carry these as file headers, and `internal/websterengine_test#testReplyPath` as a const member spanning lines 375-376.
The comment-only edits therefore live in the Intent alone, with no target for the dependency edges or the impact check to see.
Round 1's F3 proposed the Intent bullet and no target, on the ground that `redundant-file-target` refuses a file glyph beside the members already on card 3; that holds for the three headers, and an overlay run confirms it.
Two things differ this round.
`testReplyPath` is a member glyph that no card lists and that `redundant-file-target` does not touch, so the claim that each file already has a member target does not cover that comment.
For the three headers, the refusal applies only to card 3's own target list, and a separate `Prosa` card after card 3 can name the file glyphs without it.
The finding is LOW because the Intent already prescribes every edit and the implementer will make them; the gap is that none is visible as a target.

### [LOW] Card 9 says fixers end after shed admits, but Relaunch runs the guard first

**Location:** `_lym/plan/09-goto-ends-fixers.md:28` and `:44`.

**Issue:** The card opens by saying fixers end "once shed has admitted the move".
For `Goto` that holds: the running, done and target checks and `CheckGotoTarget` precede `checkGuard` (`internal/shedengine/goto.go:95-114`), and nothing after the guard can refuse short of the write itself.
For `Relaunch` it does not: the guard runs inside the status-lock callback before `req.Admit` (`goto.go:228-236`), and `Admit` is where `AdmitRelaunch` refuses or reports `inEffect`.
The card's bullet at line 44 accepts the refusal case and says why, which is a defensible trade, since ending after `AdmitRelaunch` would put the end between `AdmitRelaunch` and the applied mark inside `sheddriver` and `shedarbiter`, in consumes batten shares.
It does not mention the `inEffect` case, where `Relaunch` performs no goto yet the guard has run, and its opening sentence says the opposite of both.
The practical blast radius is small: a refused consume needs a live fixer on a halted run, and an in-effect one stands paused right after a goto that already ended them.
The guard also stops being a pure decision: `GotoGuard`'s doc comment says it "only decides: it moves no run and writes no file" (`goto.go:19-20`), and the wrapper now ends processes, so that contract needs one sentence.

### [LOW] Card 9 lists loomcli's doc.go under Uses although it falsifies the guard paragraph

**Location:** `_lym/plan/09-goto-ends-fixers.md:25`.

**Issue:** `internal/loomcli/doc.go` lines 216-219 say the told goto guard "has two arms, run in this order": the Finalize arm, then the settled-segment arm.
Card 9 wraps that guard so a third behavior runs after both arms admit, and lists the doc under **Uses:**, which `doc-go-unlisted` passes even when the card falsifies it.
An added sentence leaves the existing text true, so the label is `Extends`.

## Rejected

- Card 9 ends a fixer on a goto that shed refuses for a running run, a done run or an unknown target: not a defect. `Goto` checks done, unknown target, running and `CheckGotoTarget` before `checkGuard` (`internal/shedengine/goto.go:95-114`), and `Relaunch` checks the same inside its callback before the guard (`goto.go:219-235`), so `fixerEndingGuard` never runs on those.
- Card 9 ends a fixer on a target-less relaunch: not a defect. `Relaunch` runs the guard only for a non-empty `Target` (`goto.go:228`), so the tackler's resume that re-steps an interrupted step keeps the visit's live fixer, as D2 and D8 need.
- A site that hands shedengine a guard without the wrapper: not a defect. `shedverbs/goto.go:45` reads `spec.GotoGuard`, set once at `internal/loomcli/arm.go:464` and used by `driverGoto` through `c.spec`; the other two `Guard:` sites are `loomConsumeAttempt` (`arbiterconsume.go:28`) and the `ConsumeTackler` request (`drivewiring.go:281`), both named by the card. The tackler's `GotoGuard` (`drivewiring.go:208`) and `arbiterDepsFor`'s (`arbiter.go:140`) stay bare, which is right since they only check a verdict before any consume. `battencli` builds its own spec and hosts no loom fixer row.
- No test pins that each of the three sites carries the wrapper: not raised. `TestFixerEndingGuard` pins both outcomes the focus names, ends once when the inner guard admits and ends none when it refuses or errors, and the test-economy rule discourages a wiring test per site.
- `internal/runproc/doc.go` under card 1 and `contracts/specs/shed-recipe-spec.md` under card 7: not a defect. Each is listed under **Edit:**, the planned text describes the post-change state, and neither card lists a file or package glyph beside a member of it.
