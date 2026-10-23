# Webster: escalate verify failures to a cold Opus-high fixer

A plan-level verify failure that the in-session fixer fork does not clear now goes to a cold fixer at the recovery role (`opus[high]` by the template) before the run halts.
Until now every fix attempt was a fork on Merriam's own model (`sonnet[medium]`), and the run halted to the driver and arbiter after the last one, neither of which fixes code.
The default `verify_gate_attempts` drops from 3 to 2, so an untuned hub runs one fork fixer, then one cold fixer.

## What changes

- The verify gate routes each failed evaluation to a fixer and records it as `fixer: fork` or `fixer: cold` in the verify-gate report.
  A failure goes cold when its failure count equals the attempt cap, when an earlier failure of the same shuttle run already went cold, or when the previous fixer added no commit above the pre-fix head.
  The report's way-forward line and the re-prompt line name the routed fixer.
- New verb `lyx webster fix-verify` spawns the cold fixer as its own strand (role `verify-fixer`, the `recovery` model key) and blocks until it ends or `recovery_timeout_min` passes, the way `recover-batch` does.
  It is re-entrant: a call that finds a live recorded fixer attaches instead of spawning.
  It reports `done` when the fixer wrote its reply file, otherwise `dead` with a reason, and a non-done status exits non-zero.
  It refuses, naming the way forward, when no verify-gate report exists.
- `State.VerifyFixers` records every cold fixer spawn.
  Each recorded session joins the run-write evidence, the run-entry reclaim stops a non-terminal fixer and marks it dead, and `reset --to start` stops every recorded fixer.
  `state.json` gains an optional key, so an older state still loads.
- The `webster-body-verify-fix` stencil keeps the findings and fixing rules both fixers share.
  New `webster-prefix-verify-fix-fork` and `webster-prefix-verify-fix-cold` carry what differs; the cold prefix is the opening stencil of the `verify-fixer` role and renders the parent, edit, PATTERN and friction directives.
- The Master template tells Merriam to run `lyx webster fix-verify` backgrounded when the way forward names it, and on its completion, whatever its exit, to rewrite its two contract files and end its turn so the gate verifies again.
- `doc.go` for `websterengine` and `webstercli`, `docs/overview.md` and the affected specs describe the new routing.

## Design decisions

- The escalation lives inside the gate because the gate already owns the attempt count and the fix-commit range; the tackler and arbiter never fix code.
- The gate, not Merriam, judges whether a fix worked, so a dead cold fixer just leaves the failure to be counted.
- No new `webster.yaml` key: the cold fixer reuses `recovery` and `recovery_timeout_min`.
- The verb counts no spawn of its own.
  The number of cold fixers is bounded by the gate messages Merriam receives, which the attempt cap limits, and by the Master template's rule to run it only when a gate message names it.
- The routing count is the gate closure's own count of failed evaluations in one shuttle run, so an attach restarts it like the attempt count.
- The cold fixer's reply file sits in Webster's scratch dir, outside `_lyx`, and is the strand's shuttle output file.
- Its commits pass the same commit check, verify, run-exit audit and Webster-Review as the fork's.
- The shared fixer body makes no run-exit audit claim a cold fixer is not under.

## To check by hand

- Nothing was run against a live tmux session: spawn, attach, timeout and stop of a real `verify-fixer` strand are covered by tests with injected seams only.
- A re-prompt the shuttle could not deliver is re-evaluated on a later tick and counts one more failure, so routing may go cold one attempt early; the cap still bounds the run.
- The cold fixer starts with only the report and the plan overview, so a failure whose cause needs the batch history may still beat it.
