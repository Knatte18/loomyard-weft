---
verdict: APPROVED
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, plus the merge-in `900008660`.

## Publish failure

Both failing tests (`TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps`, `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`) failed in `.lyx/verify/verify-6.log` with `status strands ... = 1; want 0`.
Card 7 (`_lyx/plan/07-loom-checklist.md`) has `startCmd` call `ensureStatusStrand` on both drivers and deletes `applyStatusStrandSurface` and `removeStatusStrands`.
`internal/loomcli/start.go:570` now calls `c.ensureStatusStrand()` unconditionally, so an llm-seeded start leaves exactly one status strand.
The cause was two smoke tests still asserting the retired zero count.
Commits `28671a71b` and `26c7ccab5` change those assertions to exactly one, and their header and doc comments match.
The assertions now agree with `startCmd`, with the plan's card 7 text for `TestSmokeStatusStrandAcrossDriverSeeds`, and with the passing `smoke_drivingsurface_test.go` sub-test `start leaves exactly one status strand`.

The log's `FAIL` lines name only these two tests in `internal/loomcli`.
The other failure-adjacent lines are WARN logs from tests that pass.
I found no other cause in the range that could make these two fail: `ensureStatusStrand` keeps, replaces or adds one strand named `statusStrandDisplayName`, and `resolveStatusStrandAction` is idempotent across the three bootstraps (keep on a matching sidecar, otherwise replace in place, never a second add).
I could not run the tmux tier here, because raw tmux-tier go runs are refused.

## Stale references

A repo-wide search for `applyStatusStrandSurface` and `removeStatusStrands` finds no remaining use, in code or in comments.
No remaining test, file comment or doc comment in `internal/loomcli`, `docs/overview.md` or `loomcli/doc.go` describes an llm-driven start as carrying no status strand.
`docs/overview.md` (module bullet and bootstrap bullet) and `loomcli/doc.go` state the checklist verb and the strand on both drivers, as card 7 requires.

## Rest of the range

Reading card 7's `startCmd`, `mustUseLLMDriverArm`, `statusStrandAddSpec` and `start.go` header comment edits, all drop the llm arm's removal as the card says.
Test economy: the replaced removal tests are gone and the new tests cover the checklist verb and the strand spec once each; I found no duplicate or uncovered added behavior.
I did not re-derive the plan's format gates or comment line breaks, per the rubric.
