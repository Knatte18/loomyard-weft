---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Took the review's second option with one narrowing. The walk's first worktree keeps the mail while reed still holds its orch strand, not whatever its state: `lym orch stop` and `BeamDown` remove the strand but keep it in the orch state, so an unconditional route would hold a stopped or moved beam's mail on an orch that may never start again."
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
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Fixer report, code round 1

## What changed

- **F1** (`fixed-differently`).
  Not made: routing to the beam's orch "whatever its liveness".
  Evidence: `internal/orchcli/stop.go#stopOrch` removes the reed strand and nothing clears the orch state's `Strand`.
  Made: `orchroute.Destination` keeps a notice at the worktree its walk starts at while reed holds that orch's strand, live or not;
  otherwise the nearest-live walk goes on.
  Every notice, ask and parent-name route goes through it.
  The summary's bullet that left the routing alone is replaced.
  The probe itself is unchanged: why the beam's orch read as down in the reported failure is not known.
  If that orch's strand was gone from reed's state at the time, this change does not cover the case.
- **F2.** `drift amend` and `drift defer` admit every orch above the pair, through the new `hubgeom.OrchChainAbove`, and the resolved parent.
- **F3.** The hand-back stencil names the parent and no longer says the stop reaches it.
- **F4.** `TestDecisionOrchChain` over a real hub.
- **F5.** An unreadable parent name warns and renders no parent.
- **F6.** `admissibleOrchs` admits the resolved parent when the chain cannot be read.
- **F7.** The batten tackler and arbiter deps and loom's arbiter and tackler prompt inputs are asserted.
  Loom's tackler assertion is on `tacklerPrompt()`; the closure that `goDriverDepsFor` builds over it is not reachable from a test.
- **F8, F13.** Two rows and per-row expected text in the beam orch's refusal table.
- **F9, F10.** `docs/overview.md`, the `hubgeom` package doc and the verbs' help texts state the current rules.
- **F11, F12.** `TestHandBack` checks the filled name; the missing-stencil table covers `HandBack`.
- An empty commit carries the `Root-Cause:` line, which the F1 commit holds outside its last paragraph.
  The F1 commit also ends in a stray `Claude-Session: none` line that I wrote; history is not rewritten.

Gate: tier 0 and tier 1 passed on the changed packages; tier 1 also ran `adoptcli`, `burlercli`, `reedcli` and `webstercli`.
No repository-wide run was made by the fixer.

## Repair after the verify gate

The verify command failed on `TestCLIForm_EveryCommand`: the F1 edit put the `lym creel post` help text over its 2000-byte cap.
The `--parent` paragraph is shortened and states the same rule.
The package run of `cmd/lym` that followed also failed `TestGofmt_ModuleFilesAreFormatted` on `internal/loomcli/drivewiring.go`,
left unaligned by the F7 edit; `lym loom fmt` fixed it.
`cmd/lym` was not among the packages run before the report was first written, which is how both slipped.
Tier 0 then passed on `cmd/lym`, `internal/creelcli` and `internal/loomcli`.

## Self-check

- fix-consistency 1, MEDIUM (`drift defer` help): fixed.
- fix-consistency 2, MEDIUM (`parent-review answer` help): fixed.
- fix-consistency 3, LOW (`ParentName` comments): fixed.
- fix-consistency 4, LOW ("nearest live orch" wording): fixed-differently.
  `docs/overview.md` is reworded and the `dobbyengine` package doc defines the phrase.
  The other comments that use it as the destination's name are unchanged, including the banners of the two beam stencils.
- fix-consistency 5, LOW (`loomcli` package doc lacks the fallback): fixed.
- fix-code-correctness 1, MEDIUM (a stopped beam orch holds its members' mail): fixed, as under F1.
- fix-code-correctness 2, LOW (parent-name wiring partly unpinned): fixed for batten; the loom closure stays unpinned, as under F7.
- fix-code-correctness 3, LOW (callers' packages not gated): fixed by the added tier 1 run.
- fix-proposal 1, MEDIUM (same as fix-code-correctness 1): fixed.
- fix-proposal 2, LOW ("back to its parent" still claims delivery): fixed.
- fix-affected-set 1, MEDIUM (batten wiring unpinned): fixed.
- fix-affected-set 2, LOW (`drift defer` help): fixed.
- fix-affected-set 3, LOW (sibling comments): fixed.
- fix-affected-set 4, LOW ("nearest live orch" wording): fixed-differently, as fix-consistency 4.

## Disputed

## Deferred with reason

## Rulings
