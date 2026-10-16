# Fixer report — discussion, round 3

## Changes

- **F1 (fixed differently)**: the premise holds on disk, and more strongly than the review states.
  D16 on disk carries no `origin/prod` sentence, and D17 has no fallback for a dropped notice at all: its only wake-time line, "deploy at your next wake after any landing", is the one D17 removes.
  D16's own fallback ("the board entry turns `done` at teardown") is named, but D17 never tells the orch to read it.
  The hub's `_board/_lyx/config/landing.yaml` sets `deploy: "./update-plugins.sh"`, and `tools/deploy/main.go` pushes `origin/prod` after each successful deploy, so `origin/prod` can only ever be a deploy baseline.
  Changes:
  - D17 gains two wake checks that read the board and do not depend on the deploy key: a running task whose board entry is `done` or whose pair is gone is treated as landed, and any ready task not yet started is started.
  - D17 gains a third, deploy-only check: while the `deploy` key is empty, run `update-plugins.sh` when `origin/main` holds a commit `origin/prod` lacks, scoped to landings into a `require_pr_to_base` branch, with a bound saying all three baselines survive a refresh, compact or handoff.
    The removed "next wake" line is now stated as replaced by these checks.
  - D16's not-retried sentence now ends at D17's board check, and its rationale names the board as the durable fallback.
  - Two acceptance rows: one pins the board checks in the orch stencil, one pins the scoped deploy check.

  Not made as proposed: the fix key's D16 step replaces an `origin/prod` sentence and narrows an existing deploy acceptance row, and neither exists on disk.
  The existing row (every landing notice goes to After landing) still holds, so it stays, and the deploy check gets its own row.
- **D11 finish line (related to F2, made because the round's second focus item asks for it)**: D11's stated reach covers every Warn site plus every Info-or-above line in a loop or polled path, but its finish line and acceptance row listed Warn sites only.
  Both now name the polled Info-or-above lines too, and the Scope bullet uses the same wording ("Info-or-above line in a loop or a polled path" in place of "log line in a polled loop").
- Support log: a Rejected-alternatives bullet for detecting the landing by `origin/prod`, and a Round 3 entry under Review rounds.

## Disputed

- **F2, premise false**: the finding says D11 exempts every Warn site outside a loop, so that the audit's reach sentence promises checks the finish line drops.
  D11 on disk audits every Warn site, everywhere:
  - its reach (decision-record.md, D11, first sentence): "Every production `logger.Warn` call is checked against the policy";
  - it gives a non-loop site a verdict and does not exempt it: "A site that sits in no loop and records a real failure stays Warn";
  - its finish line: "gives every production `logger.Warn` site a verdict";
  - its acceptance row: "Every production `logger.Warn` site has a verdict in the plan (D11)";
  - its rationale: "the board asks for the other modules' Warn calls to be checked".

  The lines the finding quotes ("every Warn line and every Info-or-above line inside a loop", "such a site is out of this audit", "keeps Warn and") do not appear in the file.
  Applying the fix would narrow the audit below the board's ask, against D11's own rationale.
  The one real gap between reach and finish line, the polled Info lines, is fixed under Changes.

## Deferred

None.
