---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F6
    disposition: fixed
---

F4 and F5 are decision-class findings held by the parent; this report leaves them out.

## Changes

- F1: `steerTmuxScriptDeny` in `internal/shellguard/rows.go` now says to run the tmux-tier test through `lym gate test` with `--tier` naming the tier the hub runs tmux tests in, the package that holds the test, and after `--` a filter that selects it.
  The single-test clause is gone.
  `internal/shuttleengine/claudeengine/testdata/settings-golden.json` carries the new text in every steer and notice copy.
  `internal/shellguard/doc.go` names `--tier` too.
- F2: `darn-fragment-make-change.md` Step 3 runs the tmux-tier test through `lym gate test --tier <that tier>` and has the writer check that the report shows the test ran.
- F3: the tool-learning and no-background rules, with the single-test form including `--tier`, are in `burler-template-fixer.md`, `landing-template-fixer.md`, `webster-body-implementer.md`, `webster-prefix-verify-fix-fork.md` and `weaver-template-batch.md`.
  `webster-fragment-card-fixer.md` is a paragraph filled into `webster-body-implementer.md`, so that body's section reaches it.
  `webster-body-verify-fix.md` also serves the cold fixer behind `gatefix-frame-cold.md`, which already has the rules, so the fork-only prefix carries them instead.
  The agents of `landing-template-conflict.md` and `loom-fragment-arbiter-fix.md` run no gate, so they get the no-background rule alone.
- F6: the `DarnTable` doc in `internal/darnengine/spec.go` names the exceptions as the writer's single measuring run of one test class, for a value only a test run shows or a fact about how an external tool behaves.

Gate repair: the F6 comment line in `internal/darnengine/spec.go` reached 120 characters, so I rewrote it as two sentences, one per line, in further commits.

Gate: `lym gate test` over the changed Go packages and the stencils package passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, the verify-fix rules duplicated in the cold path: fixed by moving them to the fork prefix.
- fix-consistency, LOW, `doc.go` omits `--tier`: fixed.
- fix-proposal, LOW, same duplication as above: merged, fixed.
- fix-affected-set, MEDIUM, weaver stencils lack the rules: fixed in `weaver-template-batch.md`; `weaver-body-verify-fix.md` joins the cold frame, which carries them.
- fix-affected-set, LOW, conflict and arbiter stencils: fixed with the no-background rule.

## Disputed

None.

## Deferred

None.

## Rulings

None.
