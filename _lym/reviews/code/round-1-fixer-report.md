---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "`Spec.NoCarryOver` makes `roleKey` return the empty key, instead of two skips in `Runner.start`. The proposed skips alone would leave the agent's record holding its name as a role key (`internal/shuttleengine/run.go`, `RoleKey: roleKey(spec)`), so a later run of a role the operator typed as the name, such as `darn`, would carry over the agent's typed turns. With the empty key the record holds none."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Fixer report, round 1

## Changes

- F1: new `Spec.NoCarryOver`, set by `operatoragent.SessionSpec`.
  Differs from the proposed change as the frontmatter says.
- F2: new `Spec.EndsWithSession`.
  Both launch paths an unawaited session can reach (`launch`, `relaunchUnderAccount`) chain the pane lines as `Runner.Open` does, through one shared `endSessionLine`.
  The strand carries the exit marker as `DoneWhen`.
  The run record still reads running until the orphan sweep removes its directory.
- F3: new `mode_rules` marker in the opening stencil, filled in `operatoragent.RenderPrompt`.
- F4, F5, F7: the proposed rows and a new `TestAgentCmd_StartsSessionFromGeometry`.
- F6: new `Runner.RefuseCovered`, called by `Runner.Open` and the agent verb; the verb's help says so.
  No test drives the agent verb under a pause; the finding is not a coverage gap.
- F8: `open`'s help uses `agentname.RoleRule`.
- F9: `_lym/landing/summary.md` names the blast radius; its changes, decisions and tests sections follow the fixes.
  Not committed.
- F10: the standalone verb list in `docs/overview.md`.

Gate repair: the gate refused five comment lines this round added, as wrapped mid-sentence or reaching 120 characters.
They are rewritten as one sentence per line in `internal/operatoragent/spec_test.go`, `internal/shuttlecli/cli_test.go`
and `internal/shuttleengine`'s `open.go`, `run.go` and `spec.go`, in a commit of their own.
Only comments changed.

Tests: `lym gate test` over the changed packages, `internal/shuttleparity`, `internal/pattern` and `cmd/lym` passed after the last code change.
It passed again over `internal/operatoragent`, `internal/shuttlecli` and `internal/shuttleengine` after the comment repair, tier0, no failures.

## Self-check

- fix-code-correctness, LOW, `paneRules` drops the sender-trust rules: fixed (same as the consistency finding 2).
  The lens's other statement, that a strand gone at startup still counts as removed, was wrong; see fix-proposal finding 1.
- fix-consistency 1, MEDIUM, done-when texts stale: fixed in the PATTERN background, the package doc and the comment in `run.go`.
- fix-consistency 2, MEDIUM, `paneRules` wording: fixed with its own wording, "Of that section, only what it says about creel mail applies to you."
- fix-consistency 3, LOW, self-removal not recorded as an exception to the stop verb: fixed in `pattern/PATTERN-shuttle-provider-seam.md`.
  This edits a PATTERN background file; the entry's line in `PATTERN.md` is unchanged.
- fix-consistency 4, LOW, `open.go` header stale: fixed.
- fix-consistency 5, LOW, overview agent entry stale: fixed.
- fix-consistency 6, LOW, summary Tests section stale: fixed, not committed.
- fix-proposal 1, BLOCKING, a provider dying at startup removes its own strand, so teardown reported a false failure and skipped the retry: fixed with the lens's first option.
  `removeStartupStrand` counts `reedengine.ErrUnknownStrand` as removed, with a test.
  Remaining limit: such a pane closes with the provider, so the saved capture is only what an earlier startup probe took.
- fix-proposal 2, LOW, a codex resume line only exits, so an interrupted session stayed a dead window: fixed.
  `endSessionLine` leaves out a line that only exits the shell, which also applies to an opened session's resume line.
- fix-affected-set: no findings.
  Its unverified note on startup removal is fix-proposal 1.

## Disputed

## Deferred

## Rulings
