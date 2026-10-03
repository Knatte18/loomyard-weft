# Localize verify failures, gate Merriam once, never publish a dirty tree

A failing test now stops on the card that caused it.
The plan-level verify runs once per attempt as a gate on Merriam, and Publish and Finalize never verify or push a dirty tree.
No verify is skipped after a crash, and no agent prompt can hit the 30000-byte launch limit.
This addresses the stops refusal-audit hit: integration regressions the plan verify could not localize, a dirty tree under verify after a merge resolution, and a stale conflict report refusing a later merge-in.

No GitHub issue is linked; the work is tracked on the board as `verify-publish-hardening`.

## What changes

- **Per-card gate**: `begin-batch` renders each card's gate as `go build ./...`, the untagged `go test ./...` and `go test -tags integration` over the card's own Go package directories.
  Directories with no Go package left (Delete-only) are omitted.
  The Plan-Write stencil gains a rule that a card rewording a message greps for the old text and takes the stale assertions into its scope.
- **Webster verify gate**: a must-pass shuttle gate on Merriam's strand replaces Merriam's integration fork, the post-exit integration stage, the in-process bisect, the triage baseline and the `webster-recovery` integration-fix strand.
  The cap is the new `webster.yaml` key `verify_gate_attempts` (default 3).
  An exhausted gate ends the run `stuck`, naming the failing identities.
- **Gate failures**: Merriam reads the findings, forks one fixer from the new `webster-body-verify-fix.md` stencil and ends its turn.
  The gate report `verify-gate.yaml` replaces `integration.yaml`.
- **Waiting**: Merriam's stencil has no polling loop and no "non-interactively" framing.
  `recover-batch` blocks until a terminal digest, so `poll_wait_s` retires but old configs still load.
- **Strand name**: Merriam's strand is `<shortname>:<slug>:webster`; the `master` model key is unchanged.
- **Verified-tree record**: one function in `verifytree` serves the webster gate, a new Webster-Burler `verify` gate, Publish, Finalize and the new `lyx webster verify` verb.
  It refuses a dirty tree and skips only when the record names HEAD's tree and the same command.
  It writes the record only after a pass.
  Landing's pending-verify marker is gone.
  `CONSTRAINTS.md` gains the Verified-Tree Invariant, and a parity test pins that all sites share the function.
- **Publish and Finalize**: both check for a clean tree before merge-in, after merge-in and after verify, and are Stuck naming the paths before anything is pushed or merged into the parent.
- **mergeresolve**: tracked edits from the conflict session land in the merge commit, an untracked file aborts the merge as stuck, and each `Resolve` clears stale `conflict-resolution-r*.md` reports.
- **Prompt delivery**: `claudeengine` launches every fresh session with a fixed pointer to the `prompt.md` it already writes.
  `MasterPromptPointer` and `master.md` retire.
- **Status**: loom status shows a running verify as `verify <site> (attempt N, since HH:MM)`.
- **Docs**: the webster and landing package docs, `webster-spec.md`, `loom-plan-spec.md`, `refusal-spec.md`, `docs/overview.md` and CLAUDE.md's Merriam entry describe the result.

## Design decisions to review

- **Bisect is replaced by a hint.**
  Card SHAs do not necessarily pass the integration suite, and `begin-batch` does not enforce card order, so the first failing SHA is not reliably the culprit.
  The findings list the cards whose commits touched a failing package; Merriam judges it.
- **Shuttle's gated wait no longer counts a turn end with background work outstanding as an arrival.**
  Otherwise the turn that spawns the fixer re-arrives at the gate against a half-edited tree.
  The deadline and liveness exits still classify Done, so `master_timeout_min` bounds a fork that never finishes.
- **Fixer commits skip record-batch's done-checks, drift detection and the glyph scope guard.**
  They are bounded by a first-parent commit check at each gate arrival, the run-exit fork audit, the gate's own verify and Webster-Review.
- **The conflict session may now change any tracked file inside the merge commit.**
  Verify and PR-Gate review the result, and new files stay refused.
- **The record guard is a steering guard, not a security boundary.**
  No prompt names the record path.
- **A Webster-Burler `verify` gate** (3 attempts) means a round that changed code ends with a passing verify.
  If Webster-Bouncer approves past an exhausted gate, Publish's own verify halts the red tree before a push.
- **No migration**: a webster run in flight with an integration-fix state record or an `integration.yaml` restarts with `--fresh`; `state.json` still decodes the retired field.

## To check by hand

- The plan-level verify and the integration report passed at the final commit; nothing was run beyond that.
- The per-card integration run over a heavily tested package such as `internal/fabricengine` can take minutes per card; judge the cost on a real plan.
- Run a production deploy (`update-plugins.sh`) after landing: the stencil board copies report drift from the worktree source, and the new stencils reach production only that way.
- The pointer launch changes every role's first user message; look for any tool or test outside the suite that reads it for content.
