---
verdict: APPROVED
---

Round 4 checks the fix for the Publish size-cap failure (`TestOverview_RealFilePassesCheck`).
The review range is `git diff $(git merge-base main HEAD)..HEAD` (no `_lyx/loom/rework/` round exists), plus the Publish merge-in `523ec97ad`.
Attention went to the commits after the merge-in, in particular `62bc9881e` (the PATTERN tightening), with a read of `5a569cb3a` and `126a1edfb`.

## Focus question 1 — size cap and `internal/pattern` gate

- `PATTERN.md` is 16337 bytes, under `MaxOverviewBytes` (16384, `internal/pattern/check.go:21`), with 47 bytes of headroom.
- The longest entry line (`PATTERN-review-round`, untouched from main) is 319 characters; the check counts runes (`internal/pattern/check.go:111`), so it is under the 320 cap.
- Every `[background]` link the tightening touched still names `pattern/<entry>.md`, and the touched entries kept their links.
- `lyx gate test ./internal/pattern` passes.
- The `documentation-lifecycle` link text changed to `overview`; its target `docs/overview.md#documentation-lifecycle` resolves to the `## Documentation lifecycle` heading at `docs/overview.md:91`.

## Focus question 2 — trigger and requirement of each tightened entry

`62bc9881e` touches seven entries; the focus list names five. Verdict per entry:

- `PATTERN-push-both-sides`: the dropped parenthetical `(per-transition, fabric push/sync/commit)` only enumerated the prime's push verbs; the trigger, the task-pair rule and "the prime's pushes stay records-only, its code branch being the operator's" remain. Not a defect.
- `PATTERN-parent-directive`: `not stencil text` and `outside the rule` became `exempt`; the requirement, the `{{.mode_rules}}` carve-out and the orch exemption all remain. Not a defect.
- `PATTERN-shuttle-stop`, `PATTERN-review-round`, `PATTERN-batcher-registry`: byte-identical to main's entries, so the commit cannot have dropped a rule from them.
- `PATTERN-edit-directive`: same wording change as `PATTERN-parent-directive`; rule intact.
- `PATTERN-fabric-vocabulary`: dropped `in any scanned file outside the owner set`; `pattern/PATTERN-fabric-vocabulary.md` states the owner set and the scope. Rule intact.
- `PATTERN-hub-store-gc`: dropped `and it is no destructive primitive`; `pattern/PATTERN-hub-store-gc.md:11` states it. Rule intact.
- `PATTERN-quarry-cgo`: dropped `refuses a cgo-free build`; the requirement (`CGO_ENABLED=1` and a C compiler) remains and `pattern/PATTERN-quarry-cgo.md` states the refusal. Rule intact.

## Other commits read

- `5a569cb3a` only joins comment sentences onto one line; line breaks are never a finding (`PATTERN-comment-line-breaks`).
- `126a1edfb` sets `FABRIC_SKIP_PUSH=1` on the subprocess alone in `TestLoomResumeReconcilesTheHubConfigBeforeArming`, keeping `t.Parallel`; the variable is honoured by `fabricengine` (`internal/fabricengine/commit.go:107`). It extends the existing helper and test, adds no new test, and is consistent with `PATTERN-test-economy`.

No findings.
