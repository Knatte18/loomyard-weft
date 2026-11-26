---
dispositions:
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
---

# Round 3 fixer-report: fabric-git-free-core discussion

## Changes

- F22 (fixed as proposed, taking the proposal's optional engine-level branch after the self-check):
  - D13 drops the installer's clear and the sentences built on it.
  - The required-seam bullet now states that the unset-seam refusal is an `internal/fabriccore` rule over a snapshot fact saying whether each required seam is supplied, covered by a tier0 core row.
    It also states that each required seam's setter in `internal/fabricengine` accepts nil and returns a restore, and that one fabricengine test unsets a seam through it and shows the refusal.
  - The sequential rule now covers any test that sets or unsets a process-global seam itself, the workspace builder included: it uses the seam's own setter, puts back what was there, builds its hub first and does not call `t.Parallel`.
    `internal/hubforge`'s one-time install is outside the rule.
  - D5 drops "or clears the seams through the installer's clear" and names the core row and the setter test.
  - The acceptance bullet "A test shows a verb run with a required seam unset refuses before acting" now names the core row and the engine test.
- F23 (fixed as proposed):
  - D13's `internal/stencilstore` bullet now moves `StencilSource` and `StencilBaseByStamp` to `internal/stencilsource` (new), which `internal/stencilcli` and `cmd/lym` import.
    It reads git only through `fabricengine.HeadContains` and the path-taking reads fabric adds, and it takes its directory names from `internal/fabriclayout` (new).
    Their two tests move with them under D12.
  - Every mention of the stencil ordering judge is gone: the required-seam list, the installer sentence, the codec-or-judge sentences, and the "real stencil store" adapter wording in D13 and the acceptance criteria.
  - D9 part 5 now does the move, and Scope In lists the new package.
- From the review's notes below its bar:
  - The three risks under `## Open risks` in `decision-record.md` are dropped, as both added arbiter entries direct.
    The section stays, since the record format requires it, and now says that no risk is open.
  - Q12 and Q15 in `support-log.md` are marked answered by those entries.
  - D13's list of tests that read the hub workspace file drops `internal/machinecli`, whose `start_test.go` compares path strings only.

## Self-check

- fix-proposal, MEDIUM, no test reaches the engine half of the unset-seam refusal: fixed as proposed. The setter accepts nil, a fabricengine test unsets a seam, and D5 and the acceptance criterion were updated to match.
- fix-proposal, LOW, the sequential rule serializes fabric tests that set their own path seam: fixed as proposed. D13 now says a fabric test sets a seam only where its assertion needs its own location, and otherwise takes hubforge's installed seams.
- fix-proposal, LOW, the moved `StencilBaseByStamp` needs the unexported `stencilsDirName`: fixed as proposed. The bullet now takes the bolt and stencil directory names from `internal/fabriclayout` (new).
- fix-consistency, LOW, the sequential rule reads as binding hubforge's one-time install: fixed as proposed. The rule now opens by excluding hubforge's install and covers a test that sets a seam "itself".
- fix-consistency, LOW, `support-log.md` lines 62, 73 and 111 still say "under Open risks": disputed, every fix worse. Those lines log what earlier rounds did, and they were true then, as the lens itself notes. Rewriting them would falsify the round history. The live answers sit on Q12 and Q15.
- fix-claims, no findings. Its out-of-lens note duplicates the fix-consistency LOW above.
- fix-way-forward, no findings.

## Disputed

None of the review's findings.

## Deferred

None.
The review notes below the bar that this round did not apply:
the `internal/secretpath` import against D1's closed core set,
the vacuous-pass negative assertions in `internal/ideengine` without the builder,
and the per-call `fabricengine.OriginSeams` wording.
The new sequential rule covers `internal/beamcli/dobbyleftover_integration_test.go`'s `t.Parallel`.

## Rulings

None.
