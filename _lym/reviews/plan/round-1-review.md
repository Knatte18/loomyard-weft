---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-goto-ends-fixers.md
    summary: Card 9 ends every fixer role before shed's own goto checks and before every tackler or arbiter consume, so a refused goto on a running row and a target-less relaunch that resumes the same row both kill the live fixer D2 and D8 say a restarted caller attaches to
    fix: "In card 9, end the fixers only on a move that passed shed's checks. Wrap the GotoGuard that loom tells shedengine so the wrapper calls endRunFixers after the existing guard admits the target. Goto runs that guard under the run lock after its own running, done and target checks; Relaunch runs it only for a non-empty Target. Set the wrapper in loomCLI.specFor (in place of Hooks.PreGoto), in goDriverDepsFor's ConsumeTackler request (in place of the bare gotoGuard) and in loomConsumeAttempt's request (in place of the bare c.settledGuard()). Drop the calls to endRunFixers before the two consumes. Repurpose loomCLI.loomPreGoto as that guard wrapper, re-signed to the GotoGuard shape, and state in the Intent that a refused goto, a target-less relaunch and a consume that AdmitRelaunch refuses end no fixer. Change TestSpecFor_PreGotoEndsFixers to pin the wrapped guard: it ends the fixers when the inner guard admits and ends none when the inner guard refuses."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-gatefixer-core.md
    summary: Card 1 registers fixer strands with runproc under the roles gate-fixer and verify-fixer, but runproc's package doc lists its role set as RoleDriver through RoleGate, and no card updates that doc
    fix: "In card 1, add `internal/runproc/doc.go` under **Edit:**. Add an Intent bullet: runproc's doc names the record's role as one of its own constants, or the strand role of a fixer that `internal/gatefixer` declares (`gate-fixer`, `verify-fixer`), which `Registrar` registers as its own runproc role. Checked with validate-plan --overlay: no new blocking finding."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/03-webster-fixer-on-core.md
    summary: Card 3 deletes or moves the cold-fixer members, but four comments that name them stay unchanged. The file headers of render.go, strand.go and recoverbatch.go and the template_test.go comment on testReplyPath all go stale.
    fix: "In card 3's Intent, add a bullet that rewrites these comments to drop the deleted names:
      - `internal/websterengine/render.go` lines 1, 4, 25, 32 and 35, which name webster-prefix-verify-fix-cold.md and RenderVerifyFixColdPrompt;
      - `internal/websterengine/strand.go` line 1, which says the file implements StrandLive and TurnEndedAfter;
      - `internal/websterengine/recoverbatch.go` line 1, which names TurnEndedAfter and StrandLive as webster's own;
      - the `internal/websterengine/template_test.go` comment on testReplyPath at line 375, which names RenderVerifyFixColdPrompt.
      Add no target: each file already has a member target on card 3, and validate-plan --overlay reports redundant-file-target for each file self glyph added beside those members."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/07-shedrecipe-fixer-element.md
    summary: The normative recipe spec says a gates element carries only attempts and an optional pass_on_cap, which the new optional fixer element key falsifies, and no card lists the spec
    fix: "In card 7, add `contracts/specs/shed-recipe-spec.md` under **Edit:**. Add an Intent bullet: the gate-capable row paragraph names the optional `fixer` element key, a `<module>.<key>` setting holding a model-spec, read only by the Darn-Write, Plan-Write and Discussion-Write rows and refused as an unknown key on any other row. Checked with validate-plan --overlay: no new blocking finding."
---

### [MEDIUM] Card 9 ends live fixers on gotos and relaunches that do not move the run

**Location:** `_lym/plan/09-goto-ends-fixers.md`

**Issue:** `shedverbs.RunGoto` runs `Hooks.PreGoto` before `shedengine.Goto`, which only then refuses a running run, a done run or an unknown target (`internal/shedverbs/goto.go:22-26`, `internal/shedengine/goto.go`).
So `lym loom goto` on a running Plan-Write row ends the fixer that the row's closure is blocked on, even though the goto is then refused.
The wrap's `Await` then reads the strand as gone, and the writer is re-prompted with a dead note.

Card 9 also calls `endRunFixers` before every tackler and arbiter consume.
That includes a resume with no goto, where `ConsumeTacklerAttempt` relaunches with an empty `Target` and leaves the status unchanged (`internal/sheddriver/tacklerconsume.go`).
It also includes a consume that `AdmitRelaunch` then refuses.
A re-step of the same row after an interrupted step therefore loses the live fixer.
D8 says such a re-step "keeps the same set, so the `Expired` probe and the wait-only entries see the live fixer", and D2's restart path finalizes by waiting on that fixer.
With card 9 as written, the re-step instead finds a dead strand and halts gate-failed with the writer's attempts unspent.
D8 licenses ending fixers only when a goto "moves the run".

### [MEDIUM] Card 1's fixer roles fall outside the role set runproc's doc lists

**Location:** `_lym/plan/01-gatefixer-core.md`

**Issue:** `Registrar` registers the fixer's pane process with `runproc.ChildRecord(panePID, role, nil)`, where `role` is `gate-fixer` or `verify-fixer`.
`internal/runproc/doc.go` describes a record's role as one of `RoleDriver`, `RoleLoop`, `RoleBatten`, `RolePush`, `RoleSync`, `RoleDobby`, `RoleCapture`, `RoleSession` or `RoleGate`.
Every existing `Register` caller uses one of those constants.
After card 1 that doc no longer covers what runproc stores, and no card lists it.

### [MEDIUM] Card 3 leaves file headers naming the members it deletes

**Location:** `_lym/plan/03-webster-fixer-on-core.md`

**Issue:** Card 3 deletes `RenderVerifyFixColdPrompt`, the cold prefix stencil, `StrandLive`, `TurnEndRead`, `TurnEndedAfter` and `turnEndSettle` from `internal/websterengine`.
The only doc text it lists is `doc.go`.
These comments still describe the deleted members as webster's own:

- the `render.go` file header and package comments (lines 1, 4, 25, 32 and 35);
- the `strand.go` header (line 1: "webster's own strand/spawn seam helpers: StrandLive and TurnEndedAfter");
- the `recoverbatch.go` header (line 1: "over webster's own classification machinery — Classify/PollUntilTerminal/TurnEndedAfter/StrandLive");
- the `template_test.go:375` comment.

### [MEDIUM] Card 7 adds the `fixer` element key without updating the recipe spec

**Location:** `_lym/plan/07-shedrecipe-fixer-element.md`

**Issue:** `contracts/specs/shed-recipe-spec.md` says a gate-capable row's `gates` elements each carry "their own `attempts` budget (0 means off) and an optional `pass_on_cap`".
Card 7 adds a third optional key, `fixer`, on three rows, and the spec is the normative contract for recipe authors.
No card lists it.

## Focus settled

- **Fidelity.** Every Decision and acceptance criterion has a carrier:
  - D1 is carried by cards 1 to 3.
  - D2 by card 6: `Final`, `MayHold` and `Expired` are set, and the `Attempts + 1` cap is at step 3. Card 7 applies the wrap and card 8 wires it.
  - D3 by cards 4, 5 and 7.
  - D4 by cards 11 and 12.
  - D5 by card 7's `LensReviewOnly`, card 8's bodies and card 12's card body.
  - D6 by card 3.
  - D7 by cards 10 and 13.
  - D8 by cards 6 and 8 for the records, and card 9 for the goto end, with the defect in F1.
  - The `PATTERN-gate-fixer` entry is carried by card 1.
- **Card order.** No card uses a symbol, key or stencil that a later card creates.
  - Cards 3, 6, 7 and 8 use only card 1's and card 2's members.
  - Card 4's `fixer` and `FixerTimeout` come before cards 8, 11 and 12.
  - Card 5's setting keys come before card 7's recipes resolve them.
- **Reachability.** Each prescribed refusal has a reaching input and a test row. The guards that no test reaches are fail-loud wiring guards:
  - the role-set refusal in `SpawnOrAttach`;
  - the empty-path refusal in `RenderPrompt`;
  - the unknown-entry refusals in `WrapRowGate` and `fixerSpec`.
- **Attack surface.**
  - Card 4 states the bound for the gate.yaml `fixer`: the `card` key only.
  - Card 7 states the bound for the element key: three constructors, an unknown key elsewhere.
  - Card 6 states the bound for `Final`/`MayHold`: never pending, so nothing else changes.
  - Card 12 states the bound for the fixer-phase join: the same key only, and phase written before spawn and attach.
  - Card 11 states the bound for the role guard: it grants nothing a non-fixer call lacks.
- **Test economy.** Cards 2, 10 and 13 assert Go-filled values and Go-chosen fragments, and card 3 moves its tests rather than duplicating them. No card asserts a stencil's static prose.
- **Edit typing.**
  - Edit cards use member glyphs.
  - The re-signed members carry arrows: card 10's tables and directives, card 12's `runnerCmd` and `runDetached`, card 13's renderers.
  - Each ImpactSummary states a conclusion.
  - No `Extends` target is rewritten.
- **Research steps and board.** No card leaves a fact to be found while it runs, edits this task's board entry, or files a task.
- **PATTERN fit.** No card breaks an overview entry.
  - Card 9 ends processes only through `runproc.EndRole`.
  - Roles are matched through `agentname`.
  - Prompts are read through `stencilstore`.
  - Model-specs go through `NewModelChoice`.
- **Verify lines.** No card carries a `**Verify:**` line.

## Rejected

- repo-invariants finding 1 is kept at MEDIUM as F2, not BLOCKING. The overview's `PATTERN-run-process-owner` text is met: the fixer is registered in runproc and ended through `EndRole`. The background's "under a role the package names" qualifies the `SelfRecord` clause, and D1 decides the runproc role is the strand role's name. What is broken is runproc's doc text.
- repo-invariants finding 2 duplicates affected-set F1 and is merged into F3.
- guard-reach findings 1 and 2 are the same card 9 defect with one fix, merged into F1.
- guard-reach finding 3 (card 6 `Expired` "false for one this wrap spawned" is unreachable) is not shown. When a closure returns an error after a failed stop, the record stays live. A re-step in the same driver process, where the wrap instance persists, can then probe it, and the in-memory guid set decides that answer.
- guard-reach finding 4 (an empty request role cannot occur) is wrong. On a binary upgrade, an old front's `os.Executable()` path resolves to the new binary, so a new runner can read a request with no role.
- affected-set F3 (`ImplementerBodyTemplate` reads the body with the new marker) is not a defect. That function returns the raw bytes and fills nothing, and it has no production caller, so an added optional marker changes no output.
